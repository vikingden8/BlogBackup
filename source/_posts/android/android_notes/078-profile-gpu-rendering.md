---
title: Profile GPU Rendering
date: 2018-01-24 15:32:50
tags:
categories: "Android学习笔记"
---

### GPU Profile工具

渲染性能问题往往是偷取你宝贵帧数的罪魁祸首,这种问题很容易产生,很容易出现,而且在一个非常方便的工具的帮助下,也非常容易去追踪. 使用Peofile GPU Rendering tool,你可以在手机上就可以看到究竟是什么导致你的应用程序出现卡顿,变慢的情况.

这个工具在设置-开发者选项-Profile GPU rendering选项,打开后选择on screen as bars:

![](/images/categories/android/android_notes/078/20166-d43b7ab07bbdc8bd.png)

<!--more-->

然后手机屏幕上就会出现三个颜色组成的小柱状图,以及一条绿线:

![](/images/categories/android/android_notes/078/20166-18493062713b4f79.png)

这个工具会在屏幕上显示经过分析后的图形数据,最底部的图显示的是Navigation的相关信息,最上面显示的是Notification的相关信息,中间的图显示的是当前应用程序的图.

### 使用GPU Profile工具

当你的应用程序在运行时,你会看到一排柱状图在屏幕上,从左到右动态地显示,每一个垂直的柱状图代表一帧的渲染,越长的垂直柱状图表示这一帧需要渲染的时间越长.随着需要渲染的帧数越来越多,他们会堆积在一起,这样你就可以观察到这段时间帧率的变化.

#### 绿线

下图中的绿线代表16ms,要确保一秒内打到60fps,你需要确保这些帧的每一条线都在绿色的16ms标记线之下.任何时候你看到一个竖线超过了绿色的标记现,你就会看到你的动画有卡顿现象产生.

![](/images/categories/android/android_notes/078/20166-804d55a37e3c3e78.png)

#### 柱状图

每一条柱状图都由三种颜色组成: 蓝-红-黄. 这些线直接和Android的渲染流水线和他实际运行帧数的时间关联:

![](/images/categories/android/android_notes/078/20166-b5dfa43923f7f257.png)

* 蓝色代表测量绘制的时间,或者说它代表需要多长时间去创建和更新你的DisplayList.在Android中,一个视图在可以实际的进行渲染之前,它必须被转换成GPU所熟悉的格式,简单来说就是几条绘图命令,复杂点的可能是你的自定义的View嵌入了自定义的Path. 一旦完成,结果会作为一个DisplayList对象被系统送入缓存,蓝色就是记录了需要花费多长时间在屏幕上更新视图(说白了就是执行每一个View的onDraw方法,创建或者更新每一个View的Display List对象).

![](/images/categories/android/android_notes/078/20166-502c55a98a311bce.png)

当你看到蓝色的线很高的时候,有可能是因为你的一堆视图突然变得无效了(即需要重新绘制),或者你的几个自定义视图的onDraw函数过于复杂.

![](/images/categories/android/android_notes/078/20166-ef78462a31ff884a.png)

* 红色代表执行的时间,这部分是Android进行2D渲染 Display List的时间,为了绘制到屏幕上,Android需要使用OpenGl ES的API接口来绘制Display List.这些API有效地将数据发送到GPU,最终在屏幕上显示出来.

![](/images/categories/android/android_notes/078/20166-82944575512a638a.png)

记住绘制下图这样自定义的比较复杂的视图时,需要用到的OpenGl的绘制命令也会更复杂

![](/images/categories/android/android_notes/078/20166-5ca0a909a80b2d66.png)

当你看到红色的线非常高的时候,这些复杂的自定义View就是罪魁祸首:

![](/images/categories/android/android_notes/078/20166-1710efe9da88f670.png)

值得一提的是,上面图中红色线较高的一种可能性是因为重新提交了视图而导致的.这些视图并不是失效的视图,但是有些时候发生了某些事,例如视图旋转,我们需要重新清理这个区域的视图,这样可能会影响这个视图下面的视图,因为这些视图都需要进行重新的绘制操作.

* 橙色部分表示的是处理时间,或者说是CPU告诉GPU渲染一帧的地方,这是一个阻塞调用,因为CPU会一直等待GPU发出接到命令的回复,如果柱状图很高,那就意味着你给GPU太多的工作,太多的负责视图需要OpenGL命令去绘制和处理.

保持动画流畅的关键就在于让这些垂直的柱状条尽可能地保持在绿线下面,任何时候超过绿线,你就有可能丢失一帧的内容.

### 总结

GPU Profile工具能够很好地帮助你找到渲染相关的问题,但是要修复这些问题就不是那么简单了. 你需要结合代码来具体分析,找到性能的瓶颈,并进行优化.

有时候你可以以这个为工具,让负责设计这个产品的人修改他的设计,以获得良好的用户体验.

转载自: [Android性能优化典范之Profile GPU Rendering](https://www.jianshu.com/p/061bb80025c7)

视频 : [Android Performance Patterns: Tool - Profile GPU Rendering](https://www.youtube.com/watch?v=VzYkVL1n4M8)
