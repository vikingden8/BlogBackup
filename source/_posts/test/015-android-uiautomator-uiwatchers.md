---
title: Android UiAutomator UiWatchers
date: 2017-09-07 21:40:35
tags:
categories: "测试开发"
---

UiWatcher是一个接口，用来判定我们在用例操作的过程中，一些意外变量是否会发生，如果发生则可以做相对应的操作对规避它。本文将来说说这个接口规范，这是一个简单的接口，而对接口具体的操作则体现在UiDevice当中。

先来看看该接口的代码：

```java
/**
 * See {@link UiDevice#registerWatcher(String, UiWatcher)} on how to register a
 * a condition watcher to be called by the automation library. The automation library will
 * invoke checkForCondition() only when a regular API call is in retry mode because it is unable
 * to locate its selector yet. Only during this time, the watchers are invoked to check if there is
 * something else unexpected on the screen.
 * @since API Level 16
 */
public interface UiWatcher {

    /**
     * Custom handler that is automatically called when the testing framework is unable to
     * find a match using the {@link UiSelector}
     *
     * When the framework is in the process of matching a {@link UiSelector} and it
     * is unable to match any widget based on the specified criteria in the selector,
     * the framework will perform retries for a predetermined time, waiting for the display
     * to update and show the desired widget. While the framework is in this state, it will call
     * registered watchers' checkForCondition(). This gives the registered watchers a chance
     * to take a look at the display and see if there is a recognized condition that can be
     * handled and in doing so allowing the current test to continue.
     *
     * An example usage would be to look for dialogs popped due to other background
     * processes requesting user attention and have nothing to do with the application
     * currently under test.
     *
     * @return true to indicate a matched condition or false for nothing was matched
     * @since API Level 16
     */
    public boolean checkForCondition();
}
```

<!--more-->

从代码中，可以看到该接口中只有一个checkForCondition的方法，该方法是用来判断我们在操作过程中是否会出现意外干扰的因素，当然，我们可以提前预知这些因素的出现，使用判断来将它们规避，避免用例执行失败带来的困扰。

如何使用UiWatchers呢？首先我们应先实例化有个具体的UiWatchers对象，然后通过UiDevice的registerWatcher(String, UiWatcher)，进行注册，当我们的测试用例在结束后，应通过UiDevice的removeWatcher(String name)方法取消异常的监听，如下：

```java
import android.app.Instrumentation;
import android.support.test.InstrumentationRegistry;
import android.support.test.uiautomator.By;
import android.support.test.uiautomator.UiDevice;
import android.support.test.uiautomator.UiObject2;
import android.support.test.uiautomator.UiWatcher;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.util.regex.Pattern;

public class TestAlarm{
    public static Instrumentation instrumentation;
    public static UiDevice mDevice;

    @Before
    public void setUp(){
        instrumentation = InstrumentationRegistry.getInstrumentation();
        mDevice = UiDevice.getInstance(instrumentation);
        //register it
        mDevice.registerWatcher("alarmWatcher", alarmWatcher);
    }

    @After
    public void tearDown() {
        //remove it
        mDevice.removeWatcher("alarmWatcher");}

    @Test
    public void testAlarm()  {
        //TODO do your work
    }

    public  final UiWatcher alarmWatcher = new UiWatcher() {
        public boolean checkForCondition() {
            UiObject2 alarmWindows = mDevice.findObject(By.text(Pattern.compile(".*闹钟.*", Pattern.DOTALL)));
            if (alarmWindows != null) {
                //TODO we can handle exception work here!!!
                return true;
            } else {
                return false;
            }
        }
    };
}
```

最好的实践就是在setUp中注册UiWatchers，并且在tearDown中取消UiWatchers的注册！

下面的代码是UiDevice中的UiWatcher的注册和取消注册：

```java
/**
 * Registers a {@link UiWatcher} to run automatically when the testing framework is unable to
 * find a match using a {@link UiSelector}. See {@link #runWatchers()}
 *
 * @param name to register the UiWatcher
 * @param watcher {@link UiWatcher}
 * @since API Level 16
 */
public void registerWatcher(String name, UiWatcher watcher) {
    Tracer.trace(name, watcher);
    if (mInWatcherContext) {
        throw new IllegalStateException("Cannot register new watcher from within another");
    }
    mWatchers.put(name, watcher);
}

/**
 * Removes a previously registered {@link UiWatcher}.
 *
 * See {@link #registerWatcher(String, UiWatcher)}
 * @param name used to register the UiWatcher
 * @since API Level 16
 */
public void removeWatcher(String name) {
    Tracer.trace(name);
    if (mInWatcherContext) {
        throw new IllegalStateException("Cannot remove a watcher from within another");
    }
    mWatchers.remove(name);
}

/**
 * This method forces all registered watchers to run.
 * See {@link #registerWatcher(String, UiWatcher)}
 * @since API Level 16
 */
public void runWatchers() {
    Tracer.trace();
    if (mInWatcherContext) {
        return;
    }

    for (String watcherName : mWatchers.keySet()) {
        UiWatcher watcher = mWatchers.get(watcherName);
        if (watcher != null) {
            try {
                mInWatcherContext = true;
                if (watcher.checkForCondition()) {
                    setWatcherTriggered(watcherName);
                }
            } catch (Exception e) {
                Log.e(LOG_TAG, "Exceuting watcher: " + watcherName, e);
            } finally {
                mInWatcherContext = false;
            }
        }
    }
}

/**
 * Resets a {@link UiWatcher} that has been triggered.
 * If a UiWatcher runs and its {@link UiWatcher#checkForCondition()} call
 * returned <code>true</code>, then the UiWatcher is considered triggered.
 * See {@link #registerWatcher(String, UiWatcher)}
 * @since API Level 16
 */
public void resetWatcherTriggers() {
    Tracer.trace();
    mWatchersTriggers.clear();
}

/**
 * Checks if a specific registered  {@link UiWatcher} has triggered.
 * See {@link #registerWatcher(String, UiWatcher)}. If a UiWatcher runs and its
 * {@link UiWatcher#checkForCondition()} call returned <code>true</code>, then
 * the UiWatcher is considered triggered. This is helpful if a watcher is detecting errors
 * from ANR or crash dialogs and the test needs to know if a UiWatcher has been triggered.
 *
 * @param watcherName
 * @return true if triggered else false
 * @since API Level 16
 */
public boolean hasWatcherTriggered(String watcherName) {
    Tracer.trace(watcherName);
    return mWatchersTriggers.contains(watcherName);
}

/**
 * Checks if any registered {@link UiWatcher} have triggered.
 *
 * See {@link #registerWatcher(String, UiWatcher)}
 * See {@link #hasWatcherTriggered(String)}
 * @since API Level 16
 */
public boolean hasAnyWatcherTriggered() {
    Tracer.trace();
    return mWatchersTriggers.size() > 0;
}

/**
 * Used internally by this class to set a {@link UiWatcher} state as triggered.
 * @param watcherName
 */
private void setWatcherTriggered(String watcherName) {
    Tracer.trace(watcherName);
    if (!hasWatcherTriggered(watcherName)) {
        mWatchersTriggers.add(watcherName);
    }
}
```

可以看出，所有的UiWatcher是保存在一个集合为mWatchers的对象中：

```java
// store for registered UiWatchers
private final HashMap<String, UiWatcher> mWatchers = new HashMap<String, UiWatcher>();
private final List<String> mWatchersTriggers = new ArrayList<String>();

// remember if we're executing in the context of a UiWatcher
private boolean mInWatcherContext = false;
```

那到底是在哪里调用了UiWatcher的异常监控方法呢？其实，在UiObject创建对象之后，如果要执行动作，比如点击，都要先根据给定的控件条件去当前界面搜索是否存在：

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

findAccessibilityNodeInfo就是通过此方法去查找对应的控件是否存在，不存在的话，直接抛出UiObjectNotFoundException异常，下面关注下findAccessibilityNodeInfo的实现：

```java
/**
 * Finds a matching UI element in the accessibility hierarchy, by
 * using the selector for this UiObject.
 *
 * @param timeout in milliseconds
 * @return AccessibilityNodeInfo if found else null
 * @since API Level 16
 */
protected AccessibilityNodeInfo findAccessibilityNodeInfo(long timeout) {
    AccessibilityNodeInfo node = null;
    long startMills = SystemClock.uptimeMillis();
    long currentMills = 0;
    while (currentMills <= timeout) {
        node = getQueryController().findAccessibilityNodeInfo(getSelector());
        if (node != null) {
            break;
        } else {
            // does nothing if we're reentering another runWatchers()
            UiDevice.getInstance().runWatchers();
        }
        currentMills = SystemClock.uptimeMillis() - startMills;
        if(timeout > 0) {
            SystemClock.sleep(WAIT_FOR_SELECTOR_POLL);
        }
    }
    return node;
}
```

这个方法只有一个参数，timeout，这个是等待要操作的控件是否出现的超时设置。从上面的代码可以看出，如果要操作的控件当前没有在界面上出现，则会先调用UiDevice中的runWatchers()方法检查是否有特定的异常出现，然后再休眠WAIT_FOR_SELECTOR_POLL时间，再继续搜索，如此循环，直到timeout超时。

至此，Android UiAutomator中的UiWatcher源码已分析完毕.

源码路径：

* [UiWatcher.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiWatcher.java)

* [UiDevice.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiDevice.java)

* [UiObject.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiObject.java)
