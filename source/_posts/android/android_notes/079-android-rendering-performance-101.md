---
title: Rendering Performance 101
date: 2018-01-24 15:55:50
tags:
categories: "Android学习笔记"
---


### 设计与性能

渲染问题是你建立一个应用程序是最经常碰到的问题，一方面，设计师希望展现给用户一个超自然的体验，另一方面，这些华丽的动画和视图并不能在所有的Android手机上都流畅地运行。所以这就是问题所在。

![](/images/categories/android/android_notes/079/1.png)

<!--more-->


### 绘制原理

Android系统每16ms都会重新绘制一次你的Activity，也就是说，你的逻辑控制画面更新要保证最多16ms一帧才能每秒达到60帧（至于为什么是60帧，这个后面会有一个专题来讲解这个）。如下图，每一帧都在16ms内绘制完成，此时的世界是顺滑的。

![](/images/categories/android/android_notes/079/2.png)

但是如果你的应用程序没有在16ms内完成这一帧的绘制，假设你花费了24ms来绘制这一帧，那么就会出现我们称之为掉帧的情况，世界变得有延迟了。如下图：

>注：其实下图中的第二个24ms UPDATE，但是上面圈出来的总共时间又有34ms，不知是不是画错了.

![](/images/categories/android/android_notes/079/3.png)

系统准备将新的一帧绘制到屏幕上，但是这一帧并没有准备好，所有就不会有绘制操作，画面也就不会刷新。反馈到用户身上，就是用户盯着同一张图看了32ms而不是16ms，也就是说掉帧发生了。

### 掉帧

掉帧是用户体验中一个非常核心的问题，用户将很容易察觉到由于掉帧而产生的卡顿感，如果此时用户正在与系统进行交互，比如滑动列表，或者正在打字，那么卡顿感是非常明显的。用户会马上对你的应用进行吐槽，下一步工作肯定是回收站走你！所以弄清楚掉帧的原因是非常重要的。

不过引起掉帧发生的原因非常多，比如：

* **你花了太多的时间重新绘制你视图中的大部分东西，这样非常浪费CPU周期**

![](/images/categories/android/android_notes/079/4.png)

* **你有太多的对象堆叠到了一起，你在绘制用户看不到的对象上花费了太多的时间**

![](/images/categories/android/android_notes/079/5.png)

* **你有一大堆的动画重复了一遍又一遍，导致CPU和GPU组件的大量浪费**

![](/images/categories/android/android_notes/079/6.png)

### 检测和解决

检测和解决这些问题很大程度上依赖于你的应用程序架构，但是幸运的是，我们有很多开发者工具来协助我们发现和解决这些问题，有些工具甚至能追踪到具体出错的代码行数或者UI控件，这些工具包括但不限于：

#### Hierarchy View

![](/images/categories/android/android_notes/079/7.png)

#### On-Device Tools – Profile GPU Rendering 、Show GPU Overdraw、GPU View Updates

![](/images/categories/android/android_notes/079/8.png)

#### TraceView

![](/images/categories/android/android_notes/079/9.png)


转载自：[Android性能优化典范之Render Performance](http://androidperformance.com/2015/04/19/Android-Performance-Patterns-1.html)

视频：　[Android Performance Patterns: Rendering Performance 101](https://www.youtube.com/watch?v=HXQhu6qfTVU&t=4s)
