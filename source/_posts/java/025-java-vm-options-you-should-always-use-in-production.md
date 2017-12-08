---
title: Java VM Options You Should Always Use in Production
date: 2017-12-8 22:36:02
tags:
categories: "Java学习笔记"
---

文章来自：[Java VM Options You Should Always Use in Production](http://blog.sokolenko.me/2014/11/javavm-options-production.html)

只做部分翻译。

This post is a cheatsheet with enumeration of options, which should be always used to configure Java Virtual Machine for 
Web-oriented server applications (i.e. Web Front-End) in production or production-like environments.

这篇文章列举出了JVM的参数清单，主要是用来配置Web生产或类生产环境服务器上的Java虚拟机（如Web前端）。

For lazy readers full listing is here (for curious detailed explanation is provided below):

下面是所有的清单（具体的解释看后面的解释）。

<!--more-->

* Java < 8

-server
-Xms<heap size>\[g|m|k] -Xmx<heap size>\[g|m|k]
-XX:PermSize=<perm gen size>\[g|m|k] -XX:MaxPermSize=<perm gen size>\[g|m|k]
-Xmn<young size>\[g|m|k]
-XX:SurvivorRatio=<ratio>
-XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled
-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=<percent>
-XX:+ScavengeBeforeFullGC -XX:+CMSScavengeBeforeRemark
-XX:+PrintGCDateStamps -verbose:gc -XX:+PrintGCDetails -Xloggc:"<path to log>"
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M
-Dsun.net.inetaddr.ttl=<TTL in seconds>
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<path to dump>`date`.hprof
-Djava.rmi.server.hostname=<external IP>
-Dcom.sun.management.jmxremote.port=<port> 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false

* Java >= 8

-server
-Xms<heap size>\[g|m|k] -Xmx<heap size>\[g|m|k]
-XX:MaxMetaspaceSize=<metaspace size>\[g|m|k]
-Xmn<young size>\[g|m|k]
-XX:SurvivorRatio=<ratio>
-XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled
-XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=<percent>
-XX:+ScavengeBeforeFullGC -XX:+CMSScavengeBeforeRemark
-XX:+PrintGCDateStamps -verbose:gc -XX:+PrintGCDetails -Xloggc:"<path to log>"
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M
-Dsun.net.inetaddr.ttl=<TTL in seconds>
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=<path to dump>`date`.hprof
-Djava.rmi.server.hostname=<external IP>
-Dcom.sun.management.jmxremote.port=<port> 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false

### Make Server a Server

**-server**

Turns Java VM features specific to server applications, such as sofisticated JIT compiler. Though this option is 
implicitely enabled for x64 virtual machines, it still makes sence to use it as according to documentation behaviour 
maybe changed in the future.

### Make your Heap Explicit

**-Xms<heap size>\[g|m|k] -Xmx<heap size>\[g|m|k]**

To avoid dynamic heap resizing and lags, which could be caused by this, we explicitely specify minimal and maximal heap 
size. Thus Java VM will spend time only once to commit on all the heap.

通过此可以显示的指定运行时的最小和最大堆大小。

**-XX:PermSize=<perm gen size>\[g|m|k] -XX:MaxPermSize=<perm gen size>\[g|m|k]**

Logic for permanent generation is the same as for overall heap — predefine sizing to avoid costs of dynamic changes. 
Not applicable to Java >= 8.

指定永久代的最小和最大值，这个参数在Java8及以上已无作用。

更多的设置，可以打开文章前面的原文章查看。


