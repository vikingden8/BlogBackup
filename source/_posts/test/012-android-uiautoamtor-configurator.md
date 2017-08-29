---
title: Android UiAutomator Configurator
date: 2017-08-26 22:48:13
tags:
categories: "测试开发"
---

在最新的V18版本上的uiautomator终于增加了对一些常用的设置能进行配置了，也就是通过Configurator来配置。

在官方文档的介绍中，可以通过Configurator.getInstance()来得到其实例，官方建议的实际就是在设置之前，先保存原先的值，再设置新值，在测试用例执行完成之后再把之前的值设置回去。

<!--more-->

下面看一下Configurator类到底提供了哪些值提供给设置：

查看源码：

```java
public final class Configurator {
    private long mWaitForIdleTimeout = 10 * 1000;
    private long mWaitForSelector = 10 * 1000;
    private long mWaitForActionAcknowledgment = 3 * 1000;

    // The events for a scroll typically complete even before touchUp occurs.
    // This short timeout to make sure we get the very last in cases where the above isn't true.
    private long mScrollEventWaitTimeout = 200; // ms

    // Default is inject as fast as we can
    private long mKeyInjectionDelay = 0; // ms

    ...

}
```

**mWaitForIdleTimeout**:

> Sets the timeout for waiting for the user interface to go into an idle state before starting a uiautomator action.

>在开始一个uiautomator动作之前，通过设置此超时时间来等待当前的UI进入IDLE状态，也就是静止状态

>By default, all core uiautomator objects except {@link UiDevice} will perform this wait before starting to search for the widget specified by the object's {@link UiSelector}. Once the idle state is detected or the timeout elapses (whichever occurs first), the object will start to wait
for the selector to find a match.

>默认情况下，uiautomator的所有对象在通过构建的UiSelector来查找此控件之前都会先等待当前的UI处理IDLE状态，除开UiDevice的操作。只有在处于IDLE下或者已超时的情况下，才会开始通过selector来进行搜索。

**mWaitForSelector**：

>Sets the timeout for waiting for a widget to become visible in the user interface so that it can be matched by a selector.

>此超时时间是在通过selector来查找控件时，等待控件变成可见的超时设置。

>Because user interface content is dynamic, sometimes a widget may not be visible immediately and won't be detected by a selector. This timeout
allows the uiautomator framework to wait for a match to be found, up until the timeout elapses.

>因为在某些时候用户的界面是动态变化的，通过selector监测控件是否存在时有时候并不是立即可见的。这个超时时间就是为了uiautomator框架等待一个匹配的控件，直到超时。

**mScrollEventWaitTimeout**:

>Sets the timeout for waiting for an acknowledgement of an uiautomtor scroll swipe action.

>此超时时间是设置在执行uiautomator的滑动操作后的一个确认等待时长。

>The acknowledgment is an <a href="http://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html">AccessibilityEvent</a>, corresponding to the scroll action, that lets the framework determine if the scroll action was successful. Generally, this timeout should not be modified. See {@link UiScrollable}

>这个确认其实就是一个AccessibilityEvent事件，跟滑动动作相关，通过这个事件可以告知uiautomator此次的滑动是否成功。通常来说，不需要修改此值。

**mWaitForActionAcknowledgment**:

>Sets the timeout for waiting for an acknowledgment of generic uiautomator actions, such as clicks, text setting, and menu presses.

>同mScrollEventWaitTimeout，只是这个是uiautomator的点击，文本输入或者菜单点击等事件的确认超时事件设置。

>The acknowledgment is an <a href="http://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html">AccessibilityEvent</a>,
corresponding to an action, that lets the framework determine if the
action was successful. Generally, this timeout should not be modified.
See {@link UiObject}

>通过执行一个动作反馈的AccessibilityEvent事件来确认上一次的动作事件是否执行成功。通常来说，这个值也不需要修改。

**mKeyInjectionDelay**:

>Sets a delay between key presses when injecting text input.

>此超时时间指的是文本输入之间的延迟。

官方文档：[Configurator](https://developer.android.com/reference/android/support/test/uiautomator/Configurator.html)

源码: [Configurator.java](https://android.googlesource.com/platform/frameworks/uiautomator/+/android-support-test/src/main/java/android/support/test/uiautomator/Configurator.java)
