---
title: 量化和优化用户与 Android 设备之间的交互
date: 2017-1-5 11:20:07
tags:
categories: "Android性能"
---

### 摘要

传统性能不足以描述现代客户端设备。性能不仅仅关乎软件堆栈的稳定执行状态，而且通常采用处理器或其它子系统总吞吐量的最终得分进行报告。用户体验也不仅仅关乎着由用户输入触发的动态系统转换，而且还与用户感知到的响应能力、顺畅度、连贯性和准确性有关。传统的性能能够测量用户交互过程的每个环节，却不会从整体上来评估用户交互的全过程。因此，不能简单地将传统的性能优化方法应用于用户体验优化中。本文中，我们介绍了若干用户交互概念，还介绍了量化和优化用户与 Android 设备之间互动的方法。

### 客户端设备用户交互

在最近对市场上的若干 Android 设备进行的性能测量中，我们发现图形、媒体和浏览性能指标与设备 Y 相同的设备 X 在性能表现上却一直差于设备 Y。但是，设备 X 的用户感知体验却优于设备 Y。我们发现根本原因在于传统的性能指标或用传统方式设计的性能指标并没有真正描述用户交互，而只是测量了系统及其子系统的计算能力（如执行的指令数）或吞吐量（如处理的磁盘读取数）。

<!--more-->

下面以视频评估为例进行说明。传统的性能指标仅通过诸如 FPS（每秒帧数）或丢帧率等指标来测量视频播放性能。这种方法在评估用户体验时至少存在两个问题。第一个问题是视频播放仅仅是播放视频过程中用户交互的一部分。用户交互的典型生命周期通常至少包括以下环节：“启动播放器” → “开始播放” → “搜索进度” → “视频播放” → “返回主屏幕”。仅凭视频播放的良好性能不能描述视频播放过程中的真实用户体验。用户交互评估是传统性能评估的一个超集。

另一个问题是，利用 FPS 作为评估用户交互顺畅度的关键指标不能始终反映良好的用户体验。例如，当我们将一张图片存放到 Gallery3D 应用中，设备 Y 在滚动照片的过程中发生明显的断续，但是设备 Y 的 FPS 值却高于设备 X。为了量化两台设备之间的区别，我们收集了在设备 X 和设备 Y 上利用 Gallery3D 应用进行照片浏览操作过程中每一帧的数据，分别如图 1 和 图 2 所示。每一帧数据通过一条竖线表示，其中 X 轴代表绘制帧的时间，竖线的高度代表系统绘制该帧所花的时间。从图 1 和 图2 我们可以看出：设备 X 的 FPS 值明显低于 设备 Y，但是最大帧时间小于设备 Y，长于 30 毫秒的帧数少于设备 Y，且帧时间差异也小于设备 Y。这意味着要描述图片浏览操作的用户体验，还应当考虑诸如最大帧时间和帧时间差异等指标。

![图 1. 在设备 X 上向 Gallery3D 应用存放一次图片所需的帧时间](/images/categories/android/android-performance/quantify-figure-1.gif)
图 1. 在设备 X 上向 Gallery3D 应用存放一次图片所需的帧时间。

![图 2. 在设备 Y 上向 Gallery3D 应用存放一次图片所需的帧时间](/images/categories/android/android-performance/quantify-figure-2.gif)
图 2. 在设备 Y 上向 Gallery3D 应用存放一次图片所需的帧时间。

作为对比，图 3 显示了对设备 Y 进行优化后一次图片存放操作产生的帧数据。很明显，所有指标均得到改善且帧时间的分布更加均匀。

![图 3. 优化后在设备 Y 上向 Gallery3D 应用存放一次图片所需的帧时间](/images/categories/android/android-performance/quantify-figure-3.gif)
图 3. 优化后在设备 Y 上向 Gallery3D 应用存放一次图片所需的帧时间。

还需要注意的是用户体验是一个主观过程，仅仅考虑了观看电影或欣赏音乐时的体验。当前的学术研究使用不同的方法，如眼球追踪、心跳监控或轮询来了解用户体验。出于软件工程的目的，也为了系统地分析和优化用户交互，我们将交互场景分为四类：

  * <b>从用户、传感器和网络等输入到设备</b>。通过此类交互场景，您可以验证相关输入是否能够像预期的那样，促使设备正常（非正常）运转。对于触摸屏输入，此类交互场景可以测量出触摸速度、压力和范围等。

  * <b>设备对输入进行响应</b>。此类交互场景用于评估设备对于输入的响应速度。

  * <b>系统状态转变</b>。此类交互场景专用于评估屏幕上图形转换的顺畅度，可在设备响应了输入之后进行。

  * <b>持续控制设备</b>。操作设备的人们不仅会在设备上输入内容，有时还会对屏幕上的图形对象进行控制，例如控制游戏喷气式飞机或拖放应用图标。此类交互场景用于评估设备的可控性。

以上五类场景中，“向设备输入”和“控制设备”与“用户对设备的控制性”方面的用户体验相关。“设备对输入进行响应”和“系统状态转变”与“设备对用户的响应性”方面的用户体验相关。我们可以将用户交互生命周期的各个环节映射到上述各种场景中，然后在软件堆栈中找到每一种场景的关键指标，对用户交互进行测量和优化。

### Android 用户交互优化

如上一节所述，在测量用户体验方面不存在高招。我们针对用户交互测量设置了以下标准。

  * <b>可感知</b>。该指标必须由人类感知，否则，该指标与用户体验无关。

  * <b>可测量</b>。该指标应当由不同的组来测量。该指标不能采用只有由特定组测量的特殊基础设施。

  * <b>可重复</b>。测量结果应当能够在不同次测量中重复。不同次测量结果中如果发生较大差异，则意味着该指标不是一个好指标。

  * <b>可比较</b>。测量出来的数据应当能够在不同的系统中进行比较。软件工程师能够利用此指标来比较不同的系统。

  * <b>合理</b>。该指标应当有助于分析特定软件堆栈行为的起因。换言之，该指标应当能够映射到软件行为，并能够通过执行软件堆栈计算出来。

  * <b>可验证</b>。该指标可用于验证优化。优化前后的测量结果应当能够反映出用户体验的变化。

  * <b>自动</b>。出于软件工程的目的，我们期望该指标能够在几乎无人值守的情况下进行测量。这对于回归测试或预提交测试非常有用。不过，该指标并非硬性要求，因为它并不与用户体验分析和优化直接相关。

在测量指标的指导下，我们关注了以下几个用户体验的互补方面。

  * <b>用户对设备的控制性。这一方面主要涉及两个测量领域。

      * <b>准确性/模糊性</b>。用于评估对于来自触摸屏、传感器和其它来源设备的输入，系统支持的准确性、模糊性、分辨率和范围。例如，系统支持多少个压力级，采样的触摸事件坐标与指尖在屏幕上的移动轨迹有多接近，以及同一时间可对多少个手指进行采样等。

      * <b>一致性</b>。用于评估屏幕中指尖与拖放图形目标之间的拖放滞积距离。它还可用于评估用户操作与感应式对象之间的一致性，例如，倾斜受控的水流与设备倾斜角之间的角度差。

  * <b>设备对用户的响应性</b>。这一方面也涉及两个测量领域：

     * <b>响应性</b>。用于评估将输入提交至设备与设备作出明显响应之间的时间。它还包括完成一次操作所花费的时间。

     * <b>顺畅度</b>。这一测量领域利用最大帧时间、帧时间差异、FPS 和丢帧率等指标来评估图形转换顺畅度。如前所述，仅凭 FPS 一项指标无法判断与顺畅度有关的所有用户体验。

对于这四个测量领域，我们一旦确定采用某项具体指标，即需要理解该指标与“良好的”用户体验之间的相关性如何。由于用户体验是一个主观话题，高度依赖于人类的心理状态和个人品味，因此有关什么样的指标值构成了“良好的”用户体验这一问题并非总是存在科学的结论。因此，我们仅采用行业经验值。下表提供了几个行业经验值的示例。

![表 1. 用户体验的行业经验值示例](/images/categories/android/android-performance/quantify-figure-4.gif)
表 1. 用户体验的行业经验值示例

由于人类的本性，软件工程师在优化用户体验时需要注意以下两点。

一项指标值通常只能在一定范围内带来“良好的”用户体验。比该范围“更优”的值并不一定能够带来“更佳”的用户体验。超出该范围的一切用户均感受不到。

此处列出的值仅仅是针对普通人的常见情况的粗略标准。例如，经验丰富的游戏玩家可能对 120 fps 的动画不满意。另一方面，设计良好的漫画可能能够以 20 fps 的动画带来完美的顺畅度。

现在，我们可以建立优化用户体验的方法。该方法可总结为以下几个步骤。

  步骤 1. 从用户处接收用户体验观测信息或通过手动操作发现交互问题。

  步骤 2. 确定将用户体验问题转化成为软件症状的软件堆栈场景和指标

  步骤 3. 开发一个软件工作负载，以可测量和可重复的方式重现问题。该工作负载能够报告反映用户体验问题的指标值。

  步骤 4. 利用工作负载和相关工具来分析和优化软件堆栈。该工作负载还可对优化进行验证。

  步骤 5. 从用户处获得反馈并在更多应用上试用该优化方法，以便确认用户体验得到提升。

根据以上方法，我们已经制定了一种系统的 Android 用户体验优化工作模型。这一工作模型包含以下四个关键要素：

优化方法。上文已经介绍。

  * Android 工作负载套件 (AWS)。我们已经开发出一个 AWS 工作负载套件，其中包括 Android 设备的几乎所有典型使用案例（不含通信部分）

  * Android UXtune 工具套件。我们已经开发出一个有助于在软件堆栈中分析用户交互的工具套件。与传统的性能调整工具不同，UXtune 能够将用户看到的事件与系统低级别事件关联起来。

  * 观测信息和反馈。这一要素对于软件工程团队非常重要，因为这些输入能够作为用户体验的主观方面对我们的方法进行补充。

表 2 显示了 AWS 2.0 工作负载。使用案例基于我们对移动设备行业、市场应用和用户反馈的广泛调查选出。目前，AWS 正在基于用户反馈和 Android 平台的改变而不断发展。

![表 2. Android 工作负载套件 (AWS) v2.0](/images/categories/android/android-performance/quantify-figure-5.gif)
表 2. Android 工作负载套件 (AWS) v2.0

Android UXtune 工具套件包含三类工具。

  * 产生可重复的输入，以便操作设备。当前，可使用 Android input-Gestures 工具来产生触摸手势输入，以便工程师无需手动操作设备。还有一款用于产生传感器输入的工具正在开发中。

  * 显示软件组件（如事件、帧和线程等）之间的系统交互。当前可用的此类工具是 Android UXtune 1.0。用于与 PMU（性能监控单元）事件集成的工具正在开发中。

  * 提取重要的用户体验指标。当前，用于获取系统 FPS 值、应用启动时间以及触摸压力分辨率的工具分别有 Android meter-FPS、Android app-launch 和 Android touch-pressure。
有关 AWS 和 UXtune 的详细信息将在其它白皮书中介绍。

###  Android 用户交互优化案例研究

下面，我们采用“拖放”示例来说明 Android 优化方法。

根据我们的优化方法，第一步是要发现用户交互问题。当用户在拖放屏幕中的图标时，在图标和指尖之间通常会出现滞积距离，当手指移动较快时尤其明显。

如图 4 所示， T0 时间时手指开始在 P0 位置拖放图标。当手指在 T1 时间移到 P1 位置时，图标开始移动。T2 时间时，手指移到 P2 位置，而图标仍停留在 P1 位置。我们能够看到图标与指尖之间存在的滞积距离。汇总如下：

  * T0：手指开始在 P0 位置拖放图标的时间；

  * P1：图标在 T1 时间开始移动的位置；

  * T2：图标达到 P1 位置的时间；

  * P2：手指在 T2 时间触摸的位置。

![图 4. 拖放滞积距离](/images/categories/android/android-performance/quantify-figure-6.gif)
图 4. 拖放滞积距离。

下一步是要发现软件执行过程中能够显示和描述拖放滞积距离问题的场景和指标。本例中，我们可以选择在 Android 主屏幕中拖放应用图标的场景。可以选择的指标是位置 P0 和 P1 之间的距离，即 P1 - P0。也可以选择一个时间指标作为距离指标的替代或补充指标，例如图标在拖放操作后开始移动的时间，即 T1 – T0。视具体使用的距离表示法时间指标可能比距离指标更好。这是因为物理距离（单位为英寸）不同于像素距离，因而相同的像素距离可能显示出不同的视距效果。拖放距离的另一个附加指标是距离 P2 - P1，后者也可由图标从 P0 位置移到 P1 位置的时间（即 T2 - T1）来反映。对于 T1 – T0、P1 – P0、P2 – P1 和 T2 – T1 等所有指标，均是指标越小用户体验越好。

接下来，第三步是要构建一个工作负载，用来重现症状和报告所选的指标。我们基于 Android 主屏幕来构建工作负载，并将指标计算结果映射到软件堆栈工程值。图 5 显示了工作负载的实施图。根据 Android 编程约定，大多数工作负载代码均在 onTouch() 和 onDraw() 回调函数中。回调 onTouch() 时会收到一个参数形式的触摸事件，并计算出应当能够在屏幕中显示的内容更新。回调 onDraw() 则不会利用 onTouch() 回调函数更新的内容进行真正地绘制。因此 onTouch() 主要是一个包含四个步骤的状态机器，用于处理从用户手指按压屏幕到最后一次放碰整个过程中的触摸事件输入。相应地，onDraw() 回调函数中也包含四个步骤，用于为用户提供屏幕中可视的状态转换。我们将步骤 3 中的帧绘制时间戳和触摸事件坐标记录下来，如图 5 所示。其中，手指一直在移动，屏幕一直在显示不断移动的图标。接下来，我们要计算 T1、P1、T2 和 P2 的工程值，如下所示：

  * T1 = SurfaceFlinger 绘制 F1 帧的时间

  * P1 = T1 时间时触摸事件的位置值

  * T2 = 图标位于 P1 位置时的帧时间

  * P2 = T2 时间时触摸事件的位置值

![图 5. 构建优化拖放滞积距离的工作负载](/images/categories/android/android-performance/quantify-figure-7.gif)

图 5. 构建优化拖放滞积距离的工作负载

在工作负载实施过程中，手指持续移动，因此每个手指位置均存在滞积距离。我们采用最大滞积距离值作为报告的指标结果。

接下来是第四个步骤，我们将在工具的协助下分析和优化可能的拖放滞积距离。下图清楚地显示了拖放滞积产生的根本原因，并提供了某个时间点的滞积距离计算结果。

![图 6. 拖放滞积距离分析](/images/categories/android/android-performance/quantify-figure-9.gif)
图 6. 拖放滞积距离分析


图 6 中的执行时间表由 Android UXtune 生成。我们开发这一工具套件的目的是将其用于 UX 分析。图表上的评论和箭头是我为了帮助分析而添加的。标记有起点的三个线程具有最强的相关性。它们分别是绘制线程 (surfaceFlinger)、事件线程 (inputDisptacher) 和用于处理事件的应用线程。其中，一个触摸事件（此处为事件 M-1）执行路径标记有橙色的箭头。

沿着橙色箭头，坐标为 850, 542.8 的 M-1 事件从事件线程发送至应用线程。进入应用线程后，M-1 事件在该应用的消息队列中等待处理。应用线程将该事件从队列中取出，并计算出其位于该事件位置的表面缓冲器的帧内容。随后，应用线程将该表面缓冲器传递至绘制线程。绘制线程加载该缓冲器并将其与系统的其它表面混合，然后将混合后的缓冲器传递至显示屏的帧缓冲器。

利用显示屏上绘制的帧将 M-1 事件完全处理好后，事件线程也已经将坐标为 852, 404.6（手指的当前位置）的 M+1 事件发送出去。我们可以推导出此时的拖放滞积距离为 542.8 – 404.6 = 138 像素。注意事件线程中的触摸事件时间戳并非手指触摸显示屏的确切时间。两者之间可能存在几毫秒的差别。不过，这一差别不会影响我们对拖放滞积距离的分析和优化。

要优化拖放一致性用户体验，最容易想到的方法便是减少在事件处理路径中花费的时间，包括用于处理应用线程和绘制线程的时间，以便尽可能缩小事件发送与最后的事件绘制之间的时间差。由于处理事件必然会花费时间，因此这种方法无法完全消除拖放滞积距离。

另一种补充性优化方法更加智能。应用在看到帧时会试图预测下一个手指位置，然后在该位置直接绘制图标，尽管应用开始在该位置绘制图标时手指不会仍然停留在该位置。手指位置预测的算法思想大致如下：

首先，应用根据其收到的触摸事件计算出手指的移动速度 (Speedfinger)。然后，应用计算出处理一个帧期间（Timeframe，即从开始绘制一个帧到开始绘制下一个帧之间的时间）手指能够移动的距离 (Distancefinger)。接着，应用预测出看到下一个帧时手指所在的位置 (NextPositionfinger)。预测出来的手指位置为手指的当前位置 (CurrPositionfinger) 加上处理一个帧期间手指移动的距离 (Distancefinger)。当应用尝试绘制下一帧时，会在预测的手指位置 (NextPositionicon) 处绘制该图标。计算中用到的主要公式如下所示：

```java
Speedfinger = (P1 – P0)/(T1 – T0)
Timeframe = 1/FPS
Distancefinger = Speedfinger * Timeframe
NextPositionfinger = Distancefinger + CurrPositionfinger
NextPositionicon = NextPositionfinger
```

目前，我们已经成功地为英特尔® 平台中的 Android 系统开发出了预测算法。注意，要避免图标越过手指，需要添加某种调节机制。

### 影响用户交互的技术因素

Android 设备的用户体验受到多种技术因素的影响。以下我们列出了在软件工程工作中遇到的最为重要的技术因素。此列表仅涵盖了用户体验的技术设计部分，但不包含特性部分，例如对用户体验有着重要影响的软硬件特性。
![](/images/categories/android/android-performance/quantify-figure-10gif.gif)

### 其它资源

以下在线公共网站提供了有关用户交互和体验的有用信息：

  * [http://ux.stackexchange.com/](http://ux.stackexchange.com/)
  * [http://www.useit.com/papers/responsetime.html](http://www.useit.com/papers/responsetime.html)
  * [http://www.measuringux.com/](http://www.measuringux.com/)

原文地址：[量化和优化用户与 Android* 设备之间的交互](https://software.intel.com/zh-cn/android/articles/quantify-and-optimize-the-user-interactions-with-android-devices)