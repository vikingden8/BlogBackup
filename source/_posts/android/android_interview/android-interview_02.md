---
title: Android面试一天一题（Day 2）
date: 2017-01-02 15:12:46
tags:
categories: "Android面试"
---

### 面试题：用广播来更新UI界面好吗？

做为Android四大组件之一的，广播被很多人所熟知，可算是一种非常方便的解耦组件的手段。常用的方式是直接调用Context的接口（sendBroadcast & sendOrderBroadcast）发送两类型的广播：

  * Normal broadcasts无序广播，会异步的发送给所有的Receiver，接收到广播的顺序是不确定的，有可能是同时。  

  * Ordered broadcasts有序广播，广播会先发送给优先级高(android:priority)的Receiver，而且这个Receiver有权决定是继续发送到下一个Receiver或者是直接终止广播。

当然，静态和动态的两种注册Receiver的方式也难不住面试者。只是有时为了看一下面试者是否真的全面了解广播，会问一下：

>除了上面的两种广播外，还有其他类型的广播吗？

允许我心里小小的邪恶一下。很少有人知道这种方式的，可以使用sendStickyBroadcast发送Sticky类型的广播。Sticky简单说就是，在发送广播时Reciever还没有被注册，但它注册后还是可以收到在它之前发送的那条广播。

>有时候基于数据安全考虑，我们想发送广播只有自己（本进程）能接收到，那么该如何去做呢？

在我不知道有新的API或者框架时我常常喜欢用自己现有的知识去想方案，最后再Google一下看是否有更好的。这个问题，我会先想到权限，发送广播时限定有权限（receiverPermission）的接收者才能收到。但是我们知道APK太容易被反编译，注册广播的权限也只是一个字符串，并不安全。

然后可能使用Handler，没错，往主线程的消息池（Message Queue）发送消息，只有主线程的Handler可以分发处理它，广播发送的内容是一个Intent对象，我们可以直接用Message封装一下，留一个和sendBroadcast一样的接口。在handleMessage时把Intent对象传递给已注册的Receiver。

后来在看项目组的其他同事写代码时，发现还有一个LocalBroadcastManager类，查了一下官方文档是Support V4包里的一个类，其实现方式也是使用Handler，思路也是一样的。  

>BroadcastReceiver的生命周期

有些人并不态清楚Receiver也是有生命周期的，而且很短，当它的onReceive方法执行完成后，它的生命周期就结束了。这时BroadcastReceiver已经不处于active状态，被系统杀掉的概率极高，也就是说如果你在onReceive去开线程进行异步操作或者打开Dialog都有可能在没达到你要的结果时进程就被系统杀掉。因为这个进程可能只有这个Receiver这个组件在运行，当Receiver也执行完后就是一个empty进程，是最容易被系统杀掉的。替代的方案是用Notificaiton或者Service（这种情况当然不能用bindService）。

>回到今天的面试题，使用广播来更新界面是否合适？

更新界面也分很多种情况，如果不是频繁地刷新，使用广播来做也是可以的。但对于较频繁地刷新动作，建议还是不要使用这种方式。广播的发送和接收是有一定的代价的，它的传输是通过Binder进程间通信机制来实现的（细心人会发现Intent是实现了Parcelable接口的），那么系统定会为了广播能顺利传递做一些进程间通信的准备。

除此之外，还可能有其他的因素让广播发送和到达是不准时的（或者说接收是会延时）。曾经看到有人在论坛上抱怨发几个广播都卡，Google的工程师是怎么混饭吃的。

这种情况可能吗？很可能，而且很容易发生。我们要先了解Android的ActivityManagerService有一个专门的消息队列来接收发送出来的广播，sendBroadcast执行完后就立即返回，但这时发送来的广播只是被放入到队列，并不一定马上被处理。当处理到当前广播时，又会把这个广播分发给注册的广播接收分发器ReceiverDispatcher，ReceiverDispatcher最后又把广播交给接Receiver所在的线程的消息队列去处理（就是你熟悉的UI线程的Message Queue）。

整个过程从发送--ActivityManagerService--ReceiverDispatcher进行了两次Binder进程间通信，最后还要交到UI的消息队列，如果基中有一个消息的处理阻塞了UI，当然也会延迟你的onReceive的执行。

### 总结

在项目里还是有遇到开发骨干也会在onReceive中开线程做耗时操作，很多时候他们这样做了并不会立刻就会产生问题，但是并不等于不会产生问题，当设备达到特定的临界条件时（如内存紧张），这些问题往往会在最终用户那里报出来。我们都有经验由用户报出来的随机BUG往往难于跟踪和修复，所以理解清楚一些基础的机制是很有必要的，它们能帮助我们避免一些隐藏的风险。

转载自：[Android面试一天一题（2 Day）](http://www.jianshu.com/p/df7af437e766)
