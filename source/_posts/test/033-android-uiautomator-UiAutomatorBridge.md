---
title: Android UiAutomator UiAutomatorBridge
date: 2017-11-27 22:19:48
tags:
categories: "测试开发"
---

看这个类的类名，这是一个UiAutomator的桥梁类，用来连接UiAutomation、testRunner上层的API，通过该类来进行沟通和协同合作的。

那么，我们先看看开头：

```java
/**
 * @hide
 */
public abstract class UiAutomatorBridge {
    ...
}
```

<!--more-->

可以看到该类是一个抽象类，看下其的构造方法：

```java
private final UiAutomation mUiAutomation;

private final InteractionController mInteractionController;

private final QueryController mQueryController;

UiAutomatorBridge(UiAutomation uiAutomation) {
    mUiAutomation = uiAutomation;
    mInteractionController = new InteractionController(this);
    mQueryController = new QueryController(this);
}

InteractionController getInteractionController() {
    return mInteractionController;
}

QueryController getQueryController() {
    return mQueryController;
}
```

从构造器可以看出，通过传入的UiAutomation对象实例，然后初始化InteractionController和QueryController， 这两个类在后面的控件操作和控件
查找提供实现。

接下来，又对UiAutomation的基础方法进行了二次封装

```java 
public void setOnAccessibilityEventListener(OnAccessibilityEventListener listener) {
    mUiAutomation.setOnAccessibilityEventListener(listener);
}

public AccessibilityNodeInfo getRootInActiveWindow() {
    return mUiAutomation.getRootInActiveWindow();
}

public boolean injectInputEvent(InputEvent event, boolean sync) {
    return mUiAutomation.injectInputEvent(event, sync);
}

public boolean setRotation(int rotation) {
    return mUiAutomation.setRotation(rotation);
}

public void setCompressedLayoutHierarchy(boolean compressed) {
    AccessibilityServiceInfo info = mUiAutomation.getServiceInfo();
    if (compressed)
        info.flags &= ~AccessibilityServiceInfo.FLAG_INCLUDE_NOT_IMPORTANT_VIEWS;
    else
        info.flags |= AccessibilityServiceInfo.FLAG_INCLUDE_NOT_IMPORTANT_VIEWS;
    mUiAutomation.setServiceInfo(info);
}

```

而下面，又定义了四个抽象方法

```java 
public abstract int getRotation();

public abstract boolean isScreenOn();
    
public abstract Display getDefaultDisplay();

public abstract long getSystemLongPressTime();
```

里面还有一个waitForIdle的方法：

```java 
/**
* This value has the greatest bearing on the appearance of test execution speeds.
* This value is used as the minimum time to wait before considering the UI idle after
* each action.
*/
private static final long QUIET_TIME_TO_BE_CONSIDERD_IDLE_STATE = 500;//ms

/**
* This is the maximum time the automation will wait for the UI to go idle. Execution
* will resume normally anyway. This is to prevent waiting forever on display updates
* that may be related to spinning wheels or progress updates of sorts etc...
*/
private static final long TOTAL_TIME_TO_WAIT_FOR_IDLE_STATE = 1000 * 10;//ms
    
public void waitForIdle() {
    waitForIdle(TOTAL_TIME_TO_WAIT_FOR_IDLE_STATE);
}

public void waitForIdle(long timeout) {
    try {
        mUiAutomation.waitForIdle(QUIET_TIME_TO_BE_CONSIDERD_IDLE_STATE, timeout);
    } catch (TimeoutException te) {
        Log.w(LOG_TAG, "Could not detect idle state.", te);
    }
}

```

此方法主要是用在等待当前的屏幕空闲下来，如屏幕有动态的加载进度条等，这里传给mUiAutomation的总超时时长为10s。

```java 
public AccessibilityEvent executeCommandAndWaitForAccessibilityEvent(Runnable command,
        AccessibilityEventFilter filter, long timeoutMillis) throws TimeoutException {
    return mUiAutomation.executeAndWaitForEvent(command,
            filter, timeoutMillis);
}

```

此方法用来执行特定的命令，并且等待特定的事件出现。

```java 
public boolean performGlobalAction(int action) {
    return mUiAutomation.performGlobalAction(action);
}
```

上面的performGlobalAction主要用来实现全局的动作，比如下拉通知栏，快速打开设置等。

最后一个方法为截取当前屏幕，主要是通过mUiAutomation.takeScreenshot()得到当前屏幕的一个Bitmap，然后通过流的方式写入到给定参数的文件路径中。

```java 
public boolean takeScreenshot(File storePath, int quality) {
    Bitmap screenshot = mUiAutomation.takeScreenshot();
    if (screenshot == null) {
        return false;
    }
    BufferedOutputStream bos = null;
    try {
        bos = new BufferedOutputStream(new FileOutputStream(storePath));
        if (bos != null) {
            screenshot.compress(Bitmap.CompressFormat.PNG, quality, bos);
            bos.flush();
        }
    } catch (IOException ioe) {
        Log.e(LOG_TAG, "failed to save screen shot to file", ioe);
        return false;
    } finally {
        if (bos != null) {
            try {
                bos.close();
            } catch (IOException ioe) {
                /* ignore */
            }
        }
        screenshot.recycle();
    }
    return true;
}

```

[UiAutomatorBridge.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/
core-src/com/android/uiautomator/core/UiAutomatorBridge.java)