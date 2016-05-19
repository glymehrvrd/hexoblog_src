---
title: Java concurrent包学习
date: 2016-05-09 15:47:21
categories: technology
tags:
---

JDK5.0对java.util.concurrent包做了大量的改进，吸收了大量的由 [Doug Lea](https://en.wikipedia.org/wiki/Doug_Lea) 编写的util.concurrent包中的内容。

这里记录一下我对java.util.concurrent包学习的总结。

摘要自其他文章

> JDK 5.0 中的并发改进可以分为三组:

> - JVM 级别更改。大多数现代处理器对并发对某一硬件级别提供支持，通常以compare-and-swap （CAS）指令形式。CAS 是一种低级别的、细粒度的技术，它允许多个线程更新一个内存位置，同时能够检测其他线程的冲突并进行恢复。它是许多高性能并发算法的基础。在 JDK 5.0 之前，Java 语言中用于协调线程之间的访问的惟一原语是同步，同步是更重量级和粗粒度的。公开 CAS 可以开发高度可伸缩的并发 Java 类。这些更改主要由 JDK 库类使用，而不是由开发人员使用。

> - 低级实用程序类 -- 锁定和原子类。使用 CAS 作为并发原语，ReentrantLock 类提供与 synchronized 原语相同的锁定和内存语义，然而这样可以更好地控制锁定（如计时的锁定等待、锁定轮询和可中断的锁定等待）和提供更好的可伸缩性（竞争时的高性能）。大多数开发人员将不再直接使用ReentrantLock 类，而是使用在 ReentrantLock 类上构建的高级类。

> - 高级实用程序类。这些类实现并发构建块，每个计算机科学文本中都会讲述这些类 -- 信号、互斥、闩锁、屏障、交换程序、线程池和线程安全集合类等。大部分开发人员都可以在应用程序中用这些类，来替换许多（如果不是全部）同步、wait() 和 notify() 的使用，从而提高性能、可读性和正确性。

可以看到最重要的就是引入了 CAS 这个操作，AtomicXXX，ReentrantLock 都以 CAS 来实现。

那么 CAS 是什么呢， CAS 是现代CPU支持的一条指令，操作数有三个，分别是 addr, expect, value，如果当前值与 expect 相同时，那么就该值写为 value 并返回 true，否则返回 false。由于 CAS 是一条指令，可以保证操作的原子性。

为什么需要 CAS 呢？来看一看这个例子，这个例子试图实现某个方法同时只有一个线程访问，一个简单的想法就是定义一个标志位 flag，当 flag 被设置时，说明该方法正在被访问，因此其他线程不能够访问，代码如下：
```java
1 volatile int flag=0;
2 public void method1(){
3     while(flag==1)
4         waiting...
5     flag=1;
6     dosomework;
7     flag=0;
8 }
```
问题在于，flag==1 到 flag=1 这个过程并不是原子的，假设有两个线程t1,t2。flag 初始等于0，当 t1 执行到第3行后被挂起，t2 执行到第3行时 flag 仍然为0，method1 被两个线程同时运行。虽然我们已经将 flag 申明为 volatile 保证 flag 对不同线程的可见性，但是仍然不能够实现我们的需求。所以 Java 提供了 **synchronized** 这个关键字对线程同步提供 JVM 级别的实现。

CAS 可以认为是一个乐观锁，类似于数据库中的版本控制。使用 CAS 的一般形式为：
```java
do{
    备份旧数据；  
    基于旧数据构造新数据；  
}while(!CAS( 内存地址，备份的旧数据，新数据 ))  
```

有了 CAS 后，我们就可以实现支持原子操作的数据类型，如 AtomicXXX ，还有提供线程同步的锁 Lock，因为这时的比较和赋值成为一个原子操作（3到5行），所以可以实现正确的逻辑。