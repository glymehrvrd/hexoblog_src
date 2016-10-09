---
title: 记录C++的一个调试过程
date: 2016-10-07 18:11:00
categories: technology
tags:
- c++
- debug
---
# 记录C++的一个调试过程

## Pass-By-Value

使用scala编写的斗鱼弹幕监控系统运行良好，但是java的内存消耗比较大，我的小破VPS有点承受不住。于是打算使用c++重写，但在过程中却遇到了一个bug，我调试一天的时间才解决。虽然这个bug非常的愚蠢，但是不常使用c++的人可能比较容易犯而且很难找到问题出在哪，特此记录。

我使用了一个`unordered_map<int, byte_buffer>`保存房间的缓冲(key为socket file descriptor)，缓冲`byte_buffer`是我仿照java的`ByteBuffer`写的一个类，具有`flip`,`compact`等功能。

在使用epoll循环处理消息的过程中，首先根据socket file descriptor获取对应的缓冲，然后使用`recv`将数据读取到缓冲中，循环处理收到的消息直到当前缓冲中的数据不够，最后使用compact方法清除已处理的数据并为下一次接收数据做准备。

整个程序流程和我用scala写的一样，但是总是会出现错误。根据斗鱼的协议，每条消息的前4个byte应该是消息长度，但是我处理时消息长度有时候明显不对，比如为负数或一个超大的数。

通过分析，艰苦的分析后发现，**错误只会在接收到ranklist消息后发生**，ranklist消息发送长度比较长，大于mss会分段。通过gdb调试，我发现发生错误时，`byte_buffer`中的数据明显缺少了头部信息，position为0。

这个完全不应该，但是在单元测试中，`byte_buffer`完全正常，让我百思不得其解。为什么单元测试时正确，但在整体中就不正确了呢，好像问题出在上一次`byte_buffer`中存储的数据没有存储到。

最后在gdb调试中发现，unordered_map中存储的`byte_buffer`地址和我打断点的局部变量`byte_buffer`的地址不一样，终于发现问题出在这一句上

```C++
auto buff = map_sock_buffer.at(sock_fd);
```

C++是**pass-by-value**的，而我忘记了这一点，每一次赋值相当于复制了一次`byte_buffer`，操作都发生在这个`byte_buffer`上而不是`unordered_map`中的`byte_buffer`上。改为

```C++
auto& buff = map_sock_buffer.at(sock_fd);
```
就正确了。

**问题定位**首先是gdb调试出错，通过bt查看栈，发现`packet_length`不正确，发现`byte_buffer`中的数据不正确，缺少消息头。但是检查`byte_buffer`源码并测试并没有发现错误，这个地方耽误了很久。最后打印`unordered_map`发现其中`byte_buffer`地址与局部变量的`byte_buffer`地址不同，这才找到错误。

## double free
`byte_buffer`是一个资源管理类，但我没遵守`rule of three/five`，没有定义`copy constructor`结果出现了这个错误。因为默认的`copy constructor`复制所有值包括指针，导致`destruct`时指针被重复释放，产生错误。

## unique_ptr
我想尝试一下智能指针`unique_ptr`来代替`byte_buffer`中的`raw pointer`，结果又出了问题。这次问题出在`unique_ptr`没有`copy constructor`。我使用的是这一句来插值到map中。

```C++
map.insert({123,byte_buffer(123)});
```

我认为这里的`byte_buffer`和`{123,byte_buffer(123)}`产生的都是rvalue，不会调用`copy constructor`。

但是`{123,byte_buffer(123)}`实际上并不是转换为`std::pair`，而是调用了一个`allocator`，其中会有`copy construct`的过程。改为

```C++
map.insert(std::make_pair(123,byte_buffer(123)));
```

## 总结
不得不感慨一下C++的坑之多，很多时候真的挺违背人类直觉的，用起来远不如`java`，`python`爽快。
而且C++背后编译器到底是怎么实现的，真的令人难以揣摩，很多时候只能遵照 best practice来编写。
再者写C++真的挺累人的，随时都要想着这个是传指针呢，还是传引用呢，这个对象应该怎么弄等等。
难怪C++普及度不及java。