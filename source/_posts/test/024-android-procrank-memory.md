---
title: Android VSS,RSS,PSS,USS
date: 2017-09-23 20:44:24
tags:
categories: "测试开发"
---

使用adb shell procrank命令会列出当前设备中应用的内存占用大小信息：

![](/images/categories/test/024/01.png)

说明：五个参数分别为PID Vss Rss Pss Uss

一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS

* VSS - Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）

* RSS - Resident Set Size 实际使用物理内存（包含共享库占用的内存）

* PSS - Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）

* USS - Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）

<!--more-->

![](/images/categories/test/024/02.png)

![](/images/categories/test/024/03.png)

![](/images/categories/test/024/04.png)

![](/images/categories/test/024/05.png)

In general, the two numbers you want to watch are the Pss and Uss (Vss and Rss are generally worthless, because they don't accurately reflect a process's usage of pages shared with other processes.)

* **Uss** is the set of pages that are unique to a process. This is the amount of memory that would be freed if the application was terminated right now.

* **Pss** is the amount of memory shared with other processes, accounted in a way that the amount is divided evenly between the processes that share it. This is memory that would not be released if the process was terminated, but is indicative of the amount that this process is "contributing" to the overall memory load.

References

* [VSS,RSS,PSS,USS](https://yq.aliyun.com/articles/24048)

* [Using procrank to measure memory usage on embedded Linux](http://www.2net.co.uk/tutorial/procrank)

* [Android Memory Usage](http://elinux.org/Android_Memory_Usage)

* [Android分享-VSS,RSS,PSS,USS](http://www.eoeandroid.com/thread-263436-1-1.html)
