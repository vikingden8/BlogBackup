---
title: Android Heap Size
date: 2017-6-7 14:33:18
tags:
categories: "Android学习笔记"
---

Android中给每个应用分配的堆大小是在/system/build.prop中定义的，可以通过pull命令拉取到本地

```sh
adb pull /system/build.prop /home/viking/test
```

用文本编辑器打开可以看到如下的配置（下面的是在S10上的参数配置文件）：

```sh
bgw.current3gband=0
dalvik.vm.checkjni=false
dalvik.vm.heapgrowthlimit=192m
dalvik.vm.heapsize=384m
dalvik.vm.isa.arm.features=default
dalvik.vm.isa.arm.variant=cortex-a53
dalvik.vm.isa.arm64.features=default
dalvik.vm.isa.arm64.variant=cortex-a53
dalvik.vm.mtk-stack-trace-file=/data/anr/mtk_traces.txt
dalvik.vm.stack-trace-file=/data/anr/traces.txt
drm.service.enabled=true
…
```

这个文件的内容很大，我们主要关注两个配置dalvik.vm.heapgrowthlimit和dalvik.vm.heapsize，参考文章：http://blog.csdn.net/jiaoyang623/article/details/8773445 的介绍，dalvik.vm.heapgrowthlimit其实就是系统给每一个应用在没有设置申请大堆内存的最大内存使用大小，而dalvik.vm.heapsize是在应用的AndroidMinifest.xml文件中的application节点上写上了android:largeHeap="true"时，系统给这个程序分配的能使用的最大堆内存大小。

同时，如果设置了android:largeHeap="true"， dalvik.vm.heapgrowthlimit将不会生效。

android:largeHeap在Android中的定义如下：

>**android:largeHeap**

>Whether your application's processes should be created with a large Dalvik heap. This applies to all processes created for the application. It only applies to the first application loaded into a process; if you're using a shared user ID to allow multiple applications to use a process, they all must use this option consistently or they will have unpredictable results.
Most apps should not need this and should instead focus on reducing their overall memory usage for improved performance. Enabling this also does not guarantee a fixed increase in available memory, because some devices are constrained by their total available memory.

>To query the available memory size at runtime, use the methods [getMemoryClass()](https://developer.android.com/reference/android/app/ActivityManager.html#getMemoryClass()) or [getLargeMemoryClass()](https://developer.android.com/reference/android/app/ActivityManager.html#getLargeMemoryClass()).

<!--more-->

在Android程序中，有如下两个方法可以帮助我们查看当前内存大小

* ActivityManager.getMemoryClass()获得内用正常情况下内存的大小

* ActivityManager.getLargeMemoryClass()可以获得开启largeHeap最大的内存大小

为了验证，我们可以简单的看下getMemoryClass()和getLargeMemoryClass()的实现

https://github.com/android/platform_frameworks_base/blob/master/core/java/android/app/ActivityManager.java 中搜索getMemoryClass(),可以看到如下：

```java
/**
 * Return the approximate per-application memory class of the current
 * device.  This gives you an idea of how hard a memory limit you should
 * impose on your application to let the overall system work best.  The
 * returned value is in megabytes; the baseline Android memory class is
 * 16 (which happens to be the Java heap limit of those devices); some
 * device with more memory may return 24 or even higher numbers.
 */
public int getMemoryClass() {
    return staticGetMemoryClass();
}

/** @hide */
static public int staticGetMemoryClass() {
    // Really brain dead right now -- just take this from the configured
    // vm heap size, and assume it is in megabytes and thus ends with "m".
    String vmHeapSize = SystemProperties.get("dalvik.vm.heapgrowthlimit", "");
    if (vmHeapSize != null && !"".equals(vmHeapSize)) {
        return Integer.parseInt(vmHeapSize.substring(0, vmHeapSize.length()-1));
    }
    return staticGetLargeMemoryClass();
}
```

从中可以看出ActivityManager.getMemoryClass()就是获取build.prop的dalvik.vm.heapgrowthlimit值。

同样的，https://github.com/android/platform_frameworks_base/blob/master/core/java/android/app/ActivityManager.java getLargeMemoryClass(),可以看到如下：

```java
/**
  * Return the approximate per-application memory class of the current
  * device when an application is running with a large heap.  This is the
  * space available for memory-intensive applications; most applications
  * should not need this amount of memory, and should instead stay with the
  * {@link #getMemoryClass()} limit.  The returned value is in megabytes.
  * This may be the same size as {@link #getMemoryClass()} on memory
  * constrained devices, or it may be significantly larger on devices with
  * a large amount of available RAM.
  *
  * <p>The is the size of the application's Dalvik heap if it has
  * specified <code>android:largeHeap="true"</code> in its manifest.
  */
 public int getLargeMemoryClass() {
     return staticGetLargeMemoryClass();
 }

 /** @hide */
 static public int staticGetLargeMemoryClass() {
     // Really brain dead right now -- just take this from the configured
     // vm heap size, and assume it is in megabytes and thus ends with "m".
     String vmHeapSize = SystemProperties.get("dalvik.vm.heapsize", "16m");
     return Integer.parseInt(vmHeapSize.substring(0, vmHeapSize.length() - 1));
 }
```

从中可以看出ActivityManager.getLargeMemoryClass()就是获取build.prop的dalvik.vm.heapsize值，验证完毕

从程序中验证两个值可以参考链接中的Demo：[https://github.com/androidyue/largeHeapDemo](https://github.com/androidyue/largeHeapDemo)

参考资料：

[探究android:largeHeap](http://droidyue.com/blog/2015/08/01/dive-into-android-large-heap/)
[largeHeap](https://developer.android.com/guide/topics/manifest/application-element.html#largeHeap)
[getMemoryClass()](https://developer.android.com/reference/android/app/ActivityManager.html#getMemoryClass())
[getLargeMemoryClass()](https://developer.android.com/reference/android/app/ActivityManager.html#getLargeMemoryClass())
[ActivityManager.java](https://github.com/android/platform_frameworks_base/blob/master/core/java/android/app/ActivityManager.java)
[manifest中的largeHeap是干什么用的？](http://blog.csdn.net/jiaoyang623/article/details/8773445)
