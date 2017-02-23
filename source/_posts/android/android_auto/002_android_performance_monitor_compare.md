---
title: Android卡顿性能监测方案对比
date: 2017-02-23 21:51:57
tags:
categories: "Android自动化"
---

### 前言

近期在研究关于 Android 卡顿性能监控，分别验证了两种相对有效的监测方案：

  * Looper 字符串匹配方案

  * Choreographer 帧率检测方案

这两种方案都可以监控到应用的卡顿现象，但两种方案的适用场景却不太一样，第一种匹配字符串方案能够准确的在发生卡顿时拿到堆栈信息，但有一定的性能损耗，不适用于线上监控；第二种监测帧率的方案不一定能准确堆栈，可能会拿到无关的系统堆栈，对定位问题没有太大帮助，但能够计算出掉帧率。下面我详细介绍一下这两种方案的实现原理和监控效果。

### Looper 字符串匹配方案

这个方案相信大家使用过开源项目 BlockCanary 应该比较熟悉，具体实现如下：

![](/images/categories/android/android-performance/002/1.png)

设置自定义 Printer，根据 Tag 标记来判断是否发生卡顿。

<!--more-->

我们可以看看 Looper 的源码：

![](/images/categories/android/android-performance/002/2.png)

通过这段源码我们大概可以如果设置了 Printer，在消息分发前和后都会打印一句 Tag 用来表示事件分发的开始和结束，所以我们可以通过 Looper 打印的 Tag 的时间间隔来判断是否发生卡顿，这个就是这个方案的原理。

我们可以看看自己实现的 Printer 代码：

![](/images/categories/android/android-performance/002/3.jpg)

![](/images/categories/android/android-performance/002/4.jpg)

通过上面的代码，那就会清楚通过判断 Tag 来设置标记，用于记录字符串打印的开始和结束，那么我们怎么利用这些标记来拿到我们想要的信息呢？ 我们会实现一个定时器,这个定时器的作用就是定时去 dump 线程堆栈，通过标记我们可以判断两次打 tag 的间隔是否超过我们设定的阈值，如果超过阈值，我们会去dump当前所有的 Java 线程堆栈，然后进行数据上报。

具体实现：

![](/images/categories/android/android-performance/002/5.png)

制造一个卡顿场景，然后上报：

![](/images/categories/android/android-performance/002/6.jpg)

这种方案可以准确的拿到用户问题堆栈，但前面说了会有一定的性能损耗，主要存在与字符串拼接可能会产生比较多的临时对象，会导致内存会频繁 GC，所以这一套方案比较适用于测试阶段使用。

### Choreographer帧率检测方案

这个方案是 API 16 以上才支持，具体实现如下：

![](/images/categories/android/android-performance/002/7.png)

这个方案的原理主要是通过 Choreographer 类设置它的 FrameCallback，我们可以在每一帧被渲染的时候记录下它开始渲染的时间，这样在下一帧被处理时，我们不仅可以判断上一帧在渲染过程中是否出现掉帧，而整个过程都是实时处理的。

自定义 FrameCallback 实现：

![](/images/categories/android/android-performance/002/8.jpg)

![](/images/categories/android/android-performance/002/9.jpg)

我们知道在 Android 中系统会发生一个 VSYNC 同步信号来通知界面进行重绘、渲染，每一次同步的周期为16.6ms，代表一帧的刷新频率，一次界面渲染会回调 doFrame 方法，如果两次 doFrame 之间的间隔大于16.6ms说明发生了卡顿，我们可以保存两次 doFrame 时间进行相减然后除以刷新频率，这样算出来的结果就是两次 doFrame 的掉帧数，通过这个掉帧数就能判断出界面卡顿的严重程度，能够帮助我们定位问题。

可以看一下我们打印出来的效果：

![](/images/categories/android/android-performance/002/10.jpg)

第一个红框是 Choreographer 打印出来的 log，这个 log 在源码也可以找到：

![](/images/categories/android/android-performance/002/11.jpg)

你会发现系统打印出来的跳帧数跟我们打印的是差不多的，所以基本能保证卡顿监测的准确性；但这种方案并不能准确拿到问题堆栈，经过我测试，拿到的堆栈信息比较随机。这个方案可以放到线上环境统计 app 掉帧情况来定位整体的卡顿性能。

### 总结

上面已经对两种卡顿性能监测方案的实现原理和测试效果进行了阐述，第一种方案比较适合在发布前进行测试或者小范围灰度测试然后定位问题，第二种方案适合监控线上环境的 app 的掉帧情况来计算 app 在某些场景的流畅度然后有针对性的做性能优化。如果其他同学对上面的有任何疑问或者有更好的实现方案可以留言进行交流。
