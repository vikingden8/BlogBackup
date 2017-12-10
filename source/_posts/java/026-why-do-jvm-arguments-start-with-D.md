---
title: Why do JVM arguments start with “-D”?
date: 2017-12-8 23:08:02
tags:
categories: "Java学习笔记"
---

From : [Why do JVM arguments start with “-D”?](https://stackoverflow.com/questions/44745261/why-do-jvm-arguments-start-with-d)

下面主要说的是-D的D是define的意思：

"Define". The meaning is similar to a preprocessor definition in C. The -D signifies that the definition is in the context of the application, and not in the Java interpreter context like any other option before the executable name.

The usage of the letter "D" isn't specifically explained in the documentation, but the only use is to "define" a key in the system properties map.

<!--more-->

下面是介绍-，-D和-X的区别：

If you do not specify anything like -myProp="XYZ" it means it is passed as an argument to main method of the program.

-D means you can use this value using System.getProperty

-X is used for extension arguments like -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=8000

