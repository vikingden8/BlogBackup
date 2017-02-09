---
title: 性能测试之内存篇测试方法整理
date: 2017-02-09 21:44:08
tags:
categories: "测试开发"
---

内存测试主要是为了检测被测试应用在进行正常使用情况下，该应用消耗手机内存的情况，如果内存消耗过大就造成手机使用卡顿等现象，进而影响用户体验，甚至会影响日活数据和用户留存等情况。因此，通常情况下，移动端应用内存占用大小也是产品一个比较重要的关注点和测试重点。为了保证应用不占用过多的系统资源，且能够及时释放内存，保证整个系统的稳定性，关于内存测试需要引入几种概念：

  * 1、  空闲状态：指打开应用后，点击home键让应用后台运行，此时应用处于的状态叫做空闲

  * 2、  中等规格：对应用的操作时间的间隔长短不一，中等规格时间较长

  * 3、  满规格：对应用的操作时间的间隔长短不一，满规格时间较短

测试时，可根据用户的操作习惯进而设置应用使用等级设置。下面是对几种内存测试方法进行整理，可根据不同的测试场景和需求，选择对应测试方案以便获取相对准确的内存数据。

<!--more-->

目前存在的android的内存测试方法可以分为以下几类：

### 使用ActivityManager.MemoryInfo

使用Android自身提供的 ActivityManager.MemoryInfo() 方法获得，通过该方法获取某应用的内存信息。目前网易Emmagee，腾讯的GT等工具都是通过该方法实现某应用内存数据的获取，测试简单方便，安装app以后选中对应的应用即可开始测试，完成测试后即可在本地sd卡中保持一份性能测试的数据，可从里面获取内存信息。

### 使用dumpsys命令

使用android提供adb指令集获取内存信息即adb shell dumpsys meminfo | grep packagename or pid 来获取

#### dumpsys获取内存数据

指令：adb shell dumpsys meminfo

通过上述指令可以查看所有应用的内存消耗情况


![](/images/categories/test/04/01.jpg)

如果想查看某一应用或某一个进程的详细的内存信息，可用如下指令：

指令：adb shell dumpsys meminfo packagename or pid

![](/images/categories/test/04/02.jpg)

从上面的Heap size类别中包含Native Heap和Dalvik Heap两部分Heap，其中dalvik就是平时说的java堆，我们创建的对象都在这里分配的。其中，dalvik heap不能超过最大限制，超过最大限制就会出现OOM；

#### 查看单个应用程序最大内存限制的指令

指令：adb shell getprop | grep or findstr heapgrowthlimit

![](/images/categories/test/04/03.jpg)

上述查看到的单个内存最大限制为128MB，而meminfo里面dalvik heap size的最大值如果超过了128M就可能出现OOM。dalvik.vm.heapgrowthlimit和dalvik.vm.heapsize都是java虚拟机的最大内存限制，应用如果不想在dalvik heap达到heapgrowthlimit限制的时候出现OOM，需要在Manifest中的application标签中声明android：largeHeap=“true”，声明后，如果应用的dalvik heap 达到heapsize的时候才会出现OOM！另：设备不一样，最大内存的限制也可能不一样

C/C++申请的内存空间在native heap中，而java申请的内存空间则在dalvik heap中。这是因为Android系统对dalvik的vm heapsize作了硬性限制，当java进程申请的java空间超过阈值时，就会抛出OOM异常（这个阈值可以是48M、24M、16M等，视机型而定），可以通过adb shell getprop | grep dalvik.vm.heapgrowthlimit查看此值。

也就是说，程序发生OMM并不表示RAM不足，而是因为程序申请的java heap对象超过了dalvik vm heapgrowthlimit。也就是说，在RAM充足的情况下，也可能发生OOM。

#### 查看单个应用的内存占有量的情况，通常用如下手段查看

查看单个应用程序最大内存限制：adb shell getprop|grep heapgrowthlimit

![](/images/categories/test/04/04.jpg)

应用启动后分配的初始内存： adb shell getprop|grep dalvik.vm.heapstartsize

![](/images/categories/test/04/05.jpg)

单个java虚拟机最大的内存限制：adb shell getprop|grep dalvik.vm.heapsize

![](/images/categories/test/04/06.jpg)

### 使用android 提供的procrank获取即可

通过指令：adb shell procrank | grep packagename

![](/images/categories/test/04/07.jpg)

通过adb shell procrank指令可以获取VSS，RSS，USS，PSS

  * VSS – Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）

  * RSS – Resident Set Size 实际使用物理内存（包含共享库占用的内存）

  * PSS – Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）

  * USS – Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）

一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS

其中USS只能通过procrank获取，首先网上下载libpagemap.so, procmem, procrank，然后push到android手机中。有的root机自带这几个文件，不需要额外下载。

![](/images/categories/test/04/08.jpg)

### 通过ADT插件DDMS查看用内存MAT进行分析

利用DDMS的Heap可以很方便的查看app的内存占用情况，在app运行时，打开DDMS选项，在Devices下，可以看到正在运行的App，选择要查看内存的App，点击该条目，并选择Update Heap，如下图：

![](/images/categories/test/04/09.jpg)

在Heap中，选择Cause GC，可以查看应用的占用情况，具体如下图：

![](/images/categories/test/04/10.jpg)

### 参考资料

转载自：[性能测试之内存篇测试方法整理](http://mtc.baidu.com/academy/detail/article/104)
