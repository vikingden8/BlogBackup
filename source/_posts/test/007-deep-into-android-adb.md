---
title: android adb源码框架分析-系统
date: 2017-7-10 14:33:59
tags:
categories: "测试开发"
---

Adb模块包括adb，adbd，源代码都在system/core/adb目录中。

地址：https://android.googlesource.com/platform/system/core/+/master， 需要翻墙。

adb和adbd有很多代码是共享的，在不同的地方通过ADB_HOST编译宏隔开，ADB_HOST=1表示adb独有的代码。

![](/images/categories/test/007/01.png)

adbd是运行在Android设备上的服务程序，也称为Adb Daemon；adb则运行在PC主机上，并且有两种存在形式：Adb Server和Adb Client。另外还有一个服务的概念，服务提供具体功能，供客户端访问。服务可能存在于Adb Server、Adb Daemon中，也可能存在于adb体系以外的某个进程中。
