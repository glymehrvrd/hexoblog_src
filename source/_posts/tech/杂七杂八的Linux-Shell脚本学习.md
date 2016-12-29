---
title: 杂七杂八的Linux Shell脚本学习
date: 2016-05-09 15:46:37
categories: technology
tags:
- Linux
- Shell
---

本文为[Linux Shell Scripting Cookbook]学习记录，枚举了一些常用的命令。

<!-- more -->
## tr
`tr [options] set1 set2`  
translate 命令，用于将输入字符从set1映射到set2。
如`tr [A-Z] [a-z]`将字符全部转换为小写。

options有许多用法，`-s`可以压缩字符，`-d`可以删除字符

## sort
`sort [options] file`  
sort 命令，用于将输入按行排序。

options用法， `-r`逆序排序，`-u`相同行只输出一次

## uniq
uniq 命令，用于输出唯一行，注意uniq只对相临行作检测，也就是说
```
123
321
123
```
依然输出
```
123
321
123
```

因此uniq常常和sort联合使用。常用选项`-c`，统计出现数目。

## cut, paste
cut切分列，paste粘贴列

## sed
sed替换文本模式，删除文本模式

## awk
awk太博大精深了，参考->[linux awk 命令详解](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)

## grep
查找模式，常用选项`-o`，仅输出匹配的内容。grep也常常和sort，uniq等联合使用，比如统计词频可以这么做：
```bash
grep -o '\b\w+\b' xxxfile|sort|uniq -c
```

## tar, gzip, zip
tar创建归档文件，gzip创建压缩文件，zip可以同时完成归档和压缩功能。