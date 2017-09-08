---
title: Android UiAutomator UiObjectNotFoundException
date: 2017-09-08 22:23:10
tags:
categories: "测试开发"
---

UiObjectNotFoundException是一个再UiAutomator中常见的异常类，当给定的UiSelector在当前界面没有可匹配的情况下会抛出此异常，下面看下其具体的代码：

```java
/**
 * Generated in test runs when a {@link UiSelector} selector could not be matched
 * to any UI element displayed.
 * @since API Level 16
 */
public class UiObjectNotFoundException extends Exception {
    private static final long serialVersionUID = 1L;
    /**
     * @since API Level 16
     **/
    public UiObjectNotFoundException(String msg) {
        super(msg);
    }
    /**
     * @since API Level 16
     **/
    public UiObjectNotFoundException(String detailMessage, Throwable throwable) {
        super(detailMessage, throwable);
    }
    /**
     * @since API Level 16
     **/
    public UiObjectNotFoundException(Throwable throwable) {
        super(throwable);
    }
}
```

<!--more-->

很常见的异常定义类，复写了父类的三个方法，那具体在代码中是如何使用的呢？我们直到，如果要找到某一个控件，都需要在UiObject的构造器方法中传入具体的UiSelector，也就是匹配当前要找到的控件的一些信息特性，如果找不到，则会在程序执行时抛出此UiObjectNotFoundException异常。

同样的，我们看下UiObject中的click方法实现：

```java
/**
 * Performs a click at the center of the visible bounds of the UI element represented
 * by this UiObject.
 *
 * @return true id successful else false
 * @throws UiObjectNotFoundException
 * @since API Level 16
 */
public boolean click() throws UiObjectNotFoundException {
    Tracer.trace();
    AccessibilityNodeInfo node = findAccessibilityNodeInfo(mConfig.getWaitForSelectorTimeout());
    if(node == null) {
        throw new UiObjectNotFoundException(getSelector().toString());
    }
    Rect rect = getVisibleBounds(node);
    return getInteractionController().clickAndSync(rect.centerX(), rect.centerY(),
            mConfig.getActionAcknowledgmentTimeout());
}
```

可以看到，如果控件在当前的界面不存在，则会抛出异常。

源码路径：

* [UiObjectNotFoundException.java](https://android.googlesource.com/platform/frameworks/uiautomator/+/master/src/com/android/uiautomator/core/UiObjectNotFoundException.java)

* [UiObject.java](https://android.googlesource.com/platform/frameworks/uiautomator/+/master/src/com/android/uiautomator/core/UiObject.java)
