---
title: Android中如何计算App的启动时间？
date: 2017-04-10 20:25:31
tags:
categories: "Android学习笔记"
---

### 应用启动场景

事实上 Android 中一个 App 的启动时间可以准确计算的.但是要分场景.也就是说要分开游戏和应用. 大家都知道,在Android中,游戏开发和应用开发是两码事.所以我们需要分开来说.

#### 应用启动

我们平时在写应用的时候,一般会指定一个 mainActivity ,用户在桌面上点击这个 Activity 的时候,系统会直接起这个 Activity. 我们知道 Activity 在启动的时候会走 onCreate/onStart/onResume .这几个回调函数.

许多书里讲过,当执行完 onResume 函数之后,应用就显示出来了…其实这是一种不准确的说法,因为从系统层面来看,一个 Activity 走完 onCreate/onStart/onResume 这几个生命周期之后,只是完成了应用自身的一些配置,比如 window 的一些属性的设置/ View 树的建立(只是建立,并没有显示,也就是说只是调用了 inflate 而已) . 后面 ViewRootImpl 还会调用两次performTraversals ,初始化 Egl 以及 measure/layout/draw. 等.
所以我们定义一个 Android 应用的启动时间, 肯定不能在 Activity 的回调函数上下手.而是以用户在手机屏幕上看到你在 onCreate 的 setContentView 中设置的 layout 完全显示为准,也就是我们常说的应用第一帧.

上面扯得有点远,不感兴趣的话可以不看,下面直接说方法.
题主说的 _adb shell am start -w packagename/activity_, 是可以完全应用的启动时间的.不过也要分场景.

<!--more-->

### 应用第一次启动

也就是我们常说的冷启动,这时候你的应用程序的进程是没有创建的. 这也是大部分应用的使用场景.用户在桌面上点击你应用的 icon 之后,首先要创建进程,然后才启动 MainActivity.
这时候adb shell am start -w packagename/MainActivity 返回的结果,就是标准的应用程序的启动时间（注意 Android 5.0 之前的手机是没有 WaitTime 这个值的）:

```sh
adb shell am start -W com.media.painter/com.media.painter.PainterMainActivity
Starting: Intent \{ act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.media.painter/.PainterMainActivity \}
Status: ok
Activity: com.media.painter/.PainterMainActivity
ThisTime: 355
TotalTime: 355
WaitTime: 365
```

总共返回了三个结果,我们以 WaitTime 为准.

关于ThisTime/TotalTime/WaitTime的区别,下面是其解释：

“_adb shell am start -W_ ”的实现在 _frameworks\base\cmds\am\src\com\android\commands\am\Am.java_ 文件中。其实就是跨Binder调用 _ActivityManagerService.startActivityAndWait()_ 接口（后面将ActivityManagerService简称为AMS），这个接口返回的结果包含上面打印的ThisTime、TotalTime时间.

  ![](/images/categories/android/android_notes/040/1.png)

  * startTime记录的刚准备调用startActivityAndWait()的时间点

  * endTime记录的是startActivityAndWait()函数调用返回的时间点

  * WaitTime = startActivityAndWait()调用耗时。

ThisTime、TotalTime 的计算在 _frameworks\base\services\core\java\com\android\server\am\ActivityRecord.java_ 文件的 _reportLaunchTimeLocked()_ 函数中。

![](/images/categories/android/android_notes/040/2.png)

我们来解释下代码里curTime、displayStartTime、mLaunchStartTime三个时间变量.

  * curTime表示该函数调用的时间点.

  * displayStartTime表示一连串启动Activity中的最后一个Activity的启动时间点.

  * mLaunchStartTime表示一连串启动Activity中第一个Activity的启动时间点.

正常情况下点击桌面图标只启动一个有界面的 Activity，此时 displayStartTime 与mLaunchStartTime 便指向同一时间点，此时 ThisTime=TotalTime。另一种情况是点击桌面图标应用会先启动一个无界面的 Activity 做逻辑处理，接着又启动一个有界面的Activity，在这种启动一连串 Activity 的情况下（知乎的启动就是属于这种情况），displayStartTime 便指向最后一个 Activity 的开始启动时间点，mLaunchStartTime 指向第一个无界面Activity的开始启动时间点，此时 ThisTime！=TotalTime。这两种情况如下图：

![](/images/categories/android/android_notes/040/3.png)

在上面的图中，我用①②③分别标注了三个时间段，在这三个时间段内分别干了什么事呢？

  * 在第①个时间段内，AMS 创建 ActivityRecord 记录块和选择合理的 Task、将当前Resume 的 Activity 进行 pause

  * 在第②个时间段内，启动进程、调用无界面 Activity 的 onCreate() 等、 pause/finish 无界面的 Activity

  * 在第③个时间段内，调用有界面 Activity 的 onCreate、onResume

看到这里应该清楚 ThisTime、TotalTime、WaitTime 三个时间的关系了吧：

  * WaitTime 就是总的耗时，包括前一个应用 Activity pause 的时间和新应用启动的时间；

  * ThisTime 表示一连串启动 Activity 的最后一个 Activity 的启动耗时；

  * TotalTime 表示新应用启动的耗时，包括新进程的启动和 Activity 的启动，但不包括前一个应用 Activity pause 的耗时。也就是说，开发者一般只要关心 TotalTime 即可，这个时间才是自己应用真正启动的耗时。

Event log中 TAG=am_activity_launch_time 中的两个值分表表示 ThisTime、TotalTime，跟通过 “adb shell am start -W ” 得到的值是一致的。

最后再说下系统根据什么来判断应用启动结束。我们知道应用启动包括进程启动、走 Activity生命周期 onCreate/onResume 等。在第一次 onResume 时添加窗口到WMS中，然后measure/layout/draw，窗口绘制完成后通知 WMS，WMS 在合适的时机控制界面开始显示(夹杂了界面切换动画逻辑)。记住是窗口界面显示出来后，WMS 才调用reportLaunchTimeLocked() 通知 AMS Activity 启动完成。

**最后总结一下，如果只关心某个应用自身启动耗时，参考TotalTime；如果关心系统启动应用耗时，参考WaitTime；如果关心应用有界面Activity启动耗时，参考ThisTime**。

#### 应用非第一次启动

如果是你按Back键，并没有将应用进程杀掉的话，那么执行上述命令就会快一些，因为不用创建进程了，只需要启动一个Activity即可。这也就是我们说的应用热启动。

### 游戏启动场景

游戏启动的话，就不适用用命令行的方法来启动了，因为从用户点击桌面图标到登录界面，既有系统的部分也有游戏自己的部分。

#### 系统部分

游戏也有一个 Activity，所以启动的时候还是会去启动这个 Activity，所以系统启动部分也就是用户点击桌面桌面响应到这个Activity启动。

#### 游戏部分

一般游戏的主 Activity 启动后，还会做一些比较耗时的事情，这时候你看到的界面是不能操作的，比如：加载游戏数据、联网更新数据、读取和更新配置文件、游戏引擎初始化等操作。从游戏开发的角度来看，到了真正用户能操作的界面才算是一个游戏真正加载完成的时间。
那么这个时间，就得使用 Log 来记录了，因为加载游戏数据、联网更新数据、读取和更新配置文件、游戏引擎初始化这些操作，都是游戏自己的逻辑，与系统无关，所以得由游戏自己定义加载完成的点。

对于游戏的启动时间，我们更倾向于计算从 **点击桌面图标** 到 **用户可以与游戏进行交互** 这个时间段作为一个游戏的启动时间。

### 总结

计算机最让人着迷的一点就是其准确性，1+1 永远等于 2，启动耗时多久就是多久，每一次可能不一样，但每一次的时间都是这一次的准确时间。

不过每个公司由于对应用的定位不同，所以对应用启动的要求也不一样。比如有的做 ROM 的公司，其内置应用的启动时间一定是要非常快的，这样给用户的第一感觉就是快、流畅；互联网公司的 App 则不是很关心启动速度，大部分互联网公司的应用都有一个启动页，用来展示广告或者功能介绍之类的，然后才会进入到主界面。需求不一样，这么做也无可厚非，不过从消费者的角度来看，越早见到主界面当然越好。

所以在做一个 Android App 的时候，一定要记得将应用的启动时间作为一个性能指标，毕竟：天下武功，唯快不破！

转载自：[Android 中如何计算 App 的启动时间？](http://androidperformance.com/2015/12/31/How-to-calculation-android-app-lunch-time.html)
