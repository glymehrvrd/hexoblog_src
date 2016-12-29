---
title: Spark Local模式可用但Standalone报错的解决方法
date: 2016-09-13 15:47:21
categories: technology
tags:
- Spark
---
使用 Spark 的 Standalone 模式部署集群遇到了很多坑，记录一下。

<!-- more -->

## java.lang.ClassCastException
这个 stackoverflow 上面也有相关问题，但也没人解决，自己鼓捣了半天后，发现问题出在 spark-submit 的 jar 包名称含有空格...文件名去掉空格后就不会报错了

## 各种ClassUndefined问题
首先检查一下有没有使用 sbt-assembly 把第三方 jar 包打包进去，或者使用 classpath 把jar包include进去。然后注意提交的 jar 包*要在所有Worker上都可见*，如果使用的posix文件名，要把jar包传到每一台Worker上，路径名称都要相同。