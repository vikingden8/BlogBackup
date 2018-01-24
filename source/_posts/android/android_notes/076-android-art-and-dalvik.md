---
title: Difference between ART and Dalvik
date: 2018-01-24 10:52:50
tags:
categories: "Android学习笔记"
---

Android runtime (ART) is the managed runtime used by applications and some system services on Android. ART and its predecessor Dalvik were originally created specifically for the Android project. ART as the runtime executes the Dalvik Executable format and Dex bytecode specification.

ART是Android上应用和系统服务的运行时管理器．ART和它的前身Dalvik最初是专为Android项目打造的，ART在运行时执行Dalvik可执行文件并遵循Dex字节码规范．

ART and Dalvik are compatible runtimes running Dex bytecode, so apps developed for Dalvik should work when running with ART. However, some techniques that work on Dalvik do not work on ART.

ART和Dalvik能兼容运行Dex字节码，因此针对Dalvik开发的应用应该也能在ART环境中运作。不过，Dalvik采用的一些技术并不适用于ART。

<!--more-->

### ART Features　ART特性

Here are some of the major features implemented by ART.

以下是ART实现的一些主要功能。

#### Ahead-of-time (AOT) compilation　AOT预先编译

ART introduces ahead-of-time (AOT) compilation, which can improve app performance. ART also has tighter install-time verification than Dalvik.

ART推出了能提升应用性能的AOT预先编译器．ART还具有比Dalvik更严格的安装时验证。

At install time, ART compiles apps using the on-device dex2oat tool. This utility accepts DEX files as input and generates a compiled app executable for the target device. The utility should be able to compile all valid DEX files without difficulty. However, some post-processing tools produce invalid files that may be tolerated by Dalvik but cannot be compiled by ART.

在安装时，ART使用设备自带的dex2oat工具来编译应用。该实用工具接受DEX文件作为输入，并针对目标设备生成已编译应用的可执行文件。该工具应该能够编译所有有效的DEX文件。但是，一些后处理的工具会生成一些不合法的文件，这些文件Dalvik可能会接受但是在ART上却不能被编译．

#### Improved garbage collection　优化的垃圾回收器

Garbage collection (GC) can impair an app's performance, resulting in choppy display, poor UI responsiveness, and other problems. ART improves garbage collection in several ways:

垃圾回收器(GC)会影响应用的性能，导致显示不稳定，UI的响应性差以及其他问题．ART通过以下几种方式优化垃圾回收：

* One GC pause instead of two　采用一个而非两个GC暂停

* Parallelized processing during the remaining GC pause　在GC保持暂停状态期间并行处理

* Collector with lower total GC time for the special case of cleaning up recently-allocated, short-lived objects　采用总GC时间更短的回收器清理最近分配，短生命周期对象

* Improved garbage collection ergonomics, making concurrent garbage collections more timely, which makes GC_FOR_ALLOC events extremely rare in typical use cases　优化了垃圾回收机制，这样能够更加及时地进行并行垃圾回收，而这使得GC_FOR_ALLOC事件在典型用例中更少

* Compacting GC to reduce background memory usage and fragmentation　压缩GC以减少后台内存使用和碎片

#### Development and debugging improvements　开发和调试优化

ART offers a number of features to improve app development and debugging.

ART提供了大量功能来优化应用开发和调试。

##### Support for sampling profiler　支持采样分析器

Historically, developers have used the Traceview tool (designed for tracing application execution) as a profiler. While Traceview gives useful information, its results on Dalvik have been skewed by the per-method-call overhead, and use of the tool noticeably affects run time performance.

一直以来，开发者都使用Traceview工具（旨在跟踪应用执行）作为分析器。虽然Traceview可提供有用的信息，但其根据每次方法调用开销得出的Dalvik分析结果会出现偏差，而且使用该工具明显会影响运行时性能。

ART adds support for a dedicated sampling profiler that does not have these limitations. This gives a more accurate view of app execution without significant slowdown. Sampling support was added to Traceview for Dalvik in the KitKat release.

ART增加了对没有这些限制的专用采样分析器的支持，从而更准确地了解应用执行情况，而不会明显减慢速度。KitKat版本为Dalvik的Traceview添加了采样支持。

##### Support for more debugging features　支持更多调试功能

ART supports a number of new debugging options, particularly in monitor- and garbage collection-related functionality. For example, you can:

ART 支持许多新的调试选项，特别是与监控和垃圾回收相关的功能。例如，你可以：

//　没用过　翻译起来困难--

* See what locks are held in stack traces, then jump to the thread that holds a lock.

* Ask how many live instances there are of a given class, ask to see the instances, and see what references are keeping an object live.　

* Filter events (like breakpoint) for a specific instance.

* See the value returned by a method when it exits (using “method-exit” events).

* Set field watchpoint to suspend the execution of a program when a specific field is accessed and/or modified.

##### Improved diagnostic detail in exceptions and crash reports　优化了异常和崩溃报告中的诊断详细信息

ART gives you as much context and detail as possible when runtime exceptions occur. ART provides expanded exception detail for java.lang.ClassCastException, java.lang.ClassNotFoundException, and java.lang.NullPointerException. (Later versions of Dalvik provided expanded exception detail for java.lang.ArrayIndexOutOfBoundsException and java.lang.ArrayStoreException, which now include the size of the array and the out-of-bounds offset, and ART does this as well.)

当发生运行时异常时，ART会为你提供尽可能多的上下文和详细信息。ART会提供java.lang.ClassCastException、java.lang.ClassNotFoundException和java.lang.NullPointerException的更多异常详细信息。（更高版本的Dalvik提供java.lang.ArrayIndexOutOfBoundsException和java.lang.ArrayStoreException的更多异常详细信息，这些信息现在包括数组大小和超出范围的偏移量。同样，ART也提供此类信息。）

For example, java.lang.NullPointerException now shows information about what the app was trying to do with the null pointer, such as the field the app was trying to write to, or the method it was trying to call. Here are some typical examples:

例如，java.lang.NullPointerException现在会显示有关应用尝试处理null指针所进行操作（例如应用尝试写入的字段或尝试调用的方法）的信息。一些典型常见示例如下：

```java
java.lang.NullPointerException: Attempt to write to field 'int
android.accessibilityservice.AccessibilityServiceInfo.flags' on a null object
reference
java.lang.NullPointerException: Attempt to invoke virtual method
'java.lang.String java.lang.Object.toString()' on a null object reference
```

ART also provides improved context information in app native crash reports, by including both Java and native stack information.

originally from : [ART and Dalvik](https://source.android.com/devices/tech/dalvik/)
