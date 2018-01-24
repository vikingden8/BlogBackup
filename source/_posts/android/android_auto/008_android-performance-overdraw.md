---
title: Android性能专项-过度绘制
date: 2018-01-19 22:41:57
tags:
categories: "移动性能测试"
---

### Understanding Overdraw-理解过度绘制

Overdraw refers to the system's drawing a pixel on the screen multiple times in a single frame of rendering. For example, if we have a bunch of stacked UI cards, each card hides a portion of the one below it.

多度绘制是指系统在屏幕中某个像素点在同一帧中被多次绘制。举一个例子，现有一堆UI卡片，每一张卡片都在另一张之下。

However, the system still needs to draw even the hidden portions of the cards in the stack. This is because stacked cards are rendered according to the painter's algorithm: that is, in back-to-front order. This sequence of rendering allows the system to apply proper alpha blending to translucent objects such as shadows.

然而，系统仍然会去绘制那些堆叠在最上层卡片之下的卡片。这是因为绘制的算法导致的：也就是，从最底层到顶层依次按顺序绘制。这种系列化的绘制，能让系统应用在透明对象的绘制上，比如阴影。

<!--more-->

### Finding Overdraw Problems-找出过度绘制

#### Debug GPU overdraw tool 使用GPU过度绘制调试工具

The Debug GPU Overdraw tool uses color-coding to show the number of times your app draws each pixel on the screen. The higher this count, the more likely it is that overdraw affects your app's performance.

GPU过度绘制调试工具使用颜色来显示你应用中像素点的绘制次数。若同一像素点绘制的次数越高，也就是意味着你的应用存在过度绘制，对应用的性能有影响。

To visualize overdraw on your device, proceed as follows:

查看你设备的可视化过度绘制，按下面的步骤：

* On your device, go to **Settings** and tap **Developer Options**.

* 进入设备的 **设置**，点击 **开发者选项**。

* Scroll down to the **Hardware accelerated rendering** section, and select **Debug GPU Overdraw**.

* 下滑到 **硬件加速渲染**，并选择其中的 **调试GPU过度绘制**。

* In the **Debug GPU overdraw** dialog, select **Show overdraw areas**.

* 在弹出的 **调试GPU过度绘制** 对话框中，选择 **显示过渡绘制区域**。

> 国内的ROM手机可能 **开发者选项** 在设置中的入口不一样

An app as it appears normally (left), and as it appears with GPU Overdraw enabled (right)

下图中的左边是正常情况下应用的显示，右边是开启了GPU过度绘制调试后的界面

![](/images/categories/android/android-performance/008/gpu-overdraw-before_2x.png)

Android colors UI elements to identify the amount of overdraw as follows:

Android根据颜色的不同来区分不同的过度绘制情况：

* **True color**: No overdraw

* **无颜色**：没有过度绘制

* **Blue**: Overdrawn 1 time

* **蓝色**：1X过度绘制

* **Green**: Overdrawn 2 times

* **绿色**：2X过度绘制

* **Pink**: Overdrawn 3 times

* **粉红**：3X过度绘制

* **Red**: Overdrawn 4 or more times

* **红色**：4X或超过4X过度绘制

Examples of an app with lots of overdraw (left) and much less overdraw (right)

下图的左边是存在很多的过度绘制，右边只有相对较少的过度绘制。

![](/images/categories/android/android-performance/008/gpu-overdraw-after_2x.png)

### Fixing Overdraw-过度绘制的优化

There are several strategies you can pursue to reduce or eliminate overdraw:

有以下几个策略来减少或避免过度绘制：

* Removing unneeded backgrounds in layouts.

* 在布局中移除非必要的背景。

* Flattening the view hierarchy.

* 使view的层次结构趋于平坦。

* Reducing transparency.

* 减少透明度。

#### Removing unneeded backgrounds in layouts

By default, a layout does not have a background, which means it does not render anything directly by itself. When layouts do have backgrounds, however, they may contribute to overdraw.

在默认情况下，布局是没有背景的，也就意味着它自己不会有任何渲染。然而，当布局有背景时，它们可能存在有过度绘制。

Removing unnecessary backgrounds is a quick way of improving rendering performance. An unnecessary background may never be visible because it's completely covered by everything else the app is drawing on top of that view. For example, the system may entirely cover up a parent's background when it draws child views on top of it.

提高渲染性能的一个快速方式就是移除非必要的背景。非必要的背景指的是在布局最顶层的控件完全覆盖在其之上，背景此时应是完全不可见的状态。

To find out why you're overdrawing, walk through the hierarchy in the Layout Inspector tool. As you do so, look out for any backgrounds you can eliminate because they are not visible to the user. Cases where many containers share a common background color offer another opportunity to eliminate unneeded backgrounds: You can set the window background to the main background color of your app, and leave all of the containers above it with no background values defined.

#### Flattening view hierarchy

Modern layouts make it easy to stack and layer views to produce beautiful design. However, doing so can degrade performance by resulting in overdraw, especially in scenarios where each stacked view object is opaque, requiring the drawing of both seen and unseen pixels to the screen.

流行的布局使用堆叠控件的方式来达到漂亮的设计。然而，这样做的话，可能会导致过度绘制影响性能，特别是在每个被堆叠的控件对象是不透明的情形下，需要同时在屏幕上绘制可见和不可见的像素点。

If you encounter this sort of issue, you may be able to improve performance by optimizing your view hierarchy to reduce the number of overlapping UI objects.

若你碰到类似这种问题，你可能需要通过优化布局的结构来减少UI对象堆叠，从而提高性能。

#### Reducing transparency

Rendering of transparent pixels on screen, known as alpha rendering, is a key contributor to overdraw. Unlike standard overdraw, in which the system completely hides existing drawn pixels by drawing opaque pixels on top of them, transparent objects require existing pixels to be drawn first, so that the right blending equation can occur. Visual effects like transparent animations, fade-outs, and drop shadows all involve some sort of transparency, and can therefore contribute significantly to overdraw. You can improve overdraw in these situations by reducing the number of transparent objects you render. For example, you can get gray text by drawing black text in a TextView with a translucent alpha value set on it. But you can get the same effect with far better performance by simply drawing the text in gray.

在屏幕上渲染透明的像素点，也被称作为alpha渲染，也是导致过度绘制的一个重要因素。不像标准的过度绘制，透明对象的像素点需要先被绘制。像透明动画，逐渐退出动画，阴影等视觉效果都是透明中的一种类型，会导致过度绘制。你可以在那种情形下通过减少透明对象数量的渲染来优化过度绘制。例如，你可以通过在TextView上画黑色的字，然后加上一个alpha的透明度得到一个灰色的字体。同样地，你也可以直接通过绘制灰色的字来达到最佳的性能。

### References-引用

* [Reducing Overdraw](https://developer.android.com/topic/performance/rendering/overdraw.html)

* [Inspect GPU Rendering Speed and Overdraw](https://developer.android.com/studio/profile/inspect-gpu-rendering.html)

* [Android Performance Patterns: Understanding Overdraw](https://www.youtube.com/watch?v=T52v50r-JfE)

* [Profiling GPU Rendering Codelab](https://io2015codelabs.appspot.com/codelabs/android-performance-debug-gpu-overdraw)

* [Android Performance Case Study](http://www.curious-creature.com/docs/android-performance-case-study-1.html)

* [Android性能优化之过渡绘制(一)](http://androidperformance.com/2014/10/20/android-performance-optimization-overdraw-1.html)

* [Android性能优化之过渡绘制( 二)](http://androidperformance.com/2015/01/13/android-performance-optimization-overdraw-2.html)
