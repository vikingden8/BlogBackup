---
title: Android性能测试
date: 2017-02-08 21:58:37
tags:
categories: "测试开发"
---

### App性能维度分析

App类型众多，根据具体类型划分，性能指标的维度和优先级各不相同。视频类app归属于娱乐游戏型的app，因此性能测试维度优先级排序为：流畅度、crash、内存、流量、响应时长、功耗、cpu。

表征不同维度指标的量化单位如图1所示。比如流畅度是FPS（帧率），内存是兆比等等

![](/images/categories/test/03/1.png)

因为android平台底层是由Linux系统改良而来，不同维度的指标绝大部分都可以通过命令来取不同的指标（具体方法可以参加后面工具）

### App性能指标获取手段

#### CPU

CPU的测试方法分为几类

  * 使用android提供的adb shell dumpsys cpuinfo |grep packagename>/address/cpu.txt来获取

  * 使用top命令adb shell top |grep packagename>/address/cpu.txt 来获取

<!--more-->

#### 内存

内存消耗，这个测试节点的设计目标是为了让应用不占用过多的系统资源，且及时释放内存，保障整个系统的稳定性，当然关于内存测试，在这里我们需要引入几个概念：空闲状态、中等规格、满规格。

空闲状态：指打开应用后，点击home键让应用后台运行，此时应用处于的状态叫做空闲。中等规格和满规格指的是对应用的操作时间的间隔长短不一，中等规格时间较长，满规格时间较短。

接下来我们说说在内存测试中，存在很多测试子项，如下清单所示：

  * 空闲状态下的应用内存消耗情况

  * 中等规格状态下的应用内存消耗情况

  * 满规格状态下的应用内存消耗情况

  * 应用内存峰值情况

  * 应用内存泄露情况

  * 应用是否常驻内存

  * 压力测试后的内存使用情况

#### 电量

功耗测试主要从以下几个方面入手进行测试：

  * 测试手机安装目标APK前后待机功耗无明显差异。

  * 常见使用场景中能够正常进入待机，待机电流在正常范围内。

  * 长时间连续使用应用无异常耗电现象。

功耗测试的方法分为两类，一类为软件测试，一类为硬件测试。

软件测试一般分为3类：

第一种采用市场上提供的第三方工具，如金山电池管家之类的。第二种就是自写工具进行，这里一般会使用3种方法:

第一种基于android提供的PowerManager.WakeLock来进行；

第二种比较复杂一点，功耗的计算=CPU消耗+Wakelock消耗+数据传输消耗+GPS消耗+Wi-Fi连接消耗；

第三种通过 adb shell dumpsys battery来获取。

【备注：未接触过，下次补充...】

#### 启动时长

首先我们来说说启动时间。关于应用的启动时间的测试，分为三类：

  * 首次启动 --应用首次启动所花费的时间

  * 非首次启动 --应用非首次启动所花费的时间

  * 应用界面切换--应用界面内切换所花费的时间

那么如何来做启动时间的测试呢，一般我们分为2类，一类为使用软件来测试，一类为使用硬件来测试，首先我们说说软件测试的方法，可能大部分人都比较通晓使用android 提供的 DisplayManager 来获取 activity 的启动时间。通过日志过滤关键字 Displayed 来过滤所有 activity 所打印的记录日志。

#### 帧率

GPU这个词对于PC性能测试者来说并不陌生，而今3Dmax，安兔兔之类的第三方软件让GPU 在移动端性能测试领域家喻户晓，但对于app内的GPU该如何来测试呢？首先我们引入几个名词：过度绘制、帧率、帧方差。过度绘制是指界面显示的activity套接了多层导致的结果。帧率是指屏幕刷新率。帧方差是指屏幕刷新帧间隔方差。对于 GPU 的测试主要包括以下几个测试子项：界面过度绘制、屏幕滑动帧速率、屏幕滑动平滑度。

对于过度绘制的测试主要通过人工进行测试，通过打开开发者选项中的显示GPU过度绘制来进行测试（PS：只有android4.2及以上的版本才具备此功能)，验收的标准为:

  * 不允许出现黑色像素

  * 不允许存在4x过度绘

  * 不允许存在面积超过屏幕1/4区域的3x过度绘制（淡红色区域）

对于屏幕滑动帧速率主要有以下方法。

  * 手机端需打开开发者选项中的启用跟踪后，勾选 Graphics 和 View；

  * 启动SDK工具Systrace插件，勾选被测应用，点击Systrace插件，在弹出的对话框中设置持续抓取时间，在tracetaps下面勾选gfx及view选项；

  * 人滑动界面可以通过节拍来进行滑动或者扫动，帧率数据会保存到默认路径下，默认名称为 trace.html；

  * 将trace.html文件拷贝到linux系统下通过命令进行转换，生成trace.csv文件。

【备注：缺失了屏幕滑动平滑度，待补上】

#### 网络流量

性能测试的——流量，当然我所指的性能测试是针对大部分应用而言的，可能还有部分应用会关注网速、弱网之类的测试。流量测试，同样需要引入几个名词：

  * 中等负荷：应用正常操作

  * 高负荷：应用极限操作

流量测试包括以下测试项：

  * 应用首次启动流量提示

  * 应用后台连续运行2小时的流量值

  * 应用高负荷运行的流量峰值

  * 应用中等负荷运行时的流量均值流量测试一般都是用软件来进行的，这里我们一般分为2类：a、采用市场提供的第三方工具来进行测试，如流量宝之类的；b、自研工具进行测试

自研工具进行测试一般包含 2 类方法：

  * 通过tcodump抓包，再通过wireshake直接读取包信息来获得流量；

  * 首先获得被测应用的uid信息，可以通过adb shell dumpsys package来获取 然后在未操作应用之前，我们可以通过查看

  ```sh
  adb shell cat /proc/uid_stat/uid/tcp_rcv

  adb shell cat /proc/uid_stat/uid/tcp_snd
  ```

  获取到应用的起始的接收及发送的流量，然后我们再操作应用，再次通过上述2条命令可以获取到应用的结束的接收及发送的流量，通过相减及得到应用的整体流量消耗。


备注：首先吐槽下其实文章写得有点糟，好多文中说过的点，没有给出具体的方法；还有就是这排版也是个问题，看起来感觉没啥逻辑。转载过来文章有排版，后续会继续学习Android移动应用的性能测试各项指标，随时更新。

### 参考资料

转载自：[性能测试](http://mtc.baidu.com/academy/detail/article/27)