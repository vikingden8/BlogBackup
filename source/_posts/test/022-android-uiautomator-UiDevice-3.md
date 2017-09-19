---
title: Android UiAutomator UiDevice（三）
date: 2017-09-19 21:03:09
tags:
categories: "测试开发"
---

上一节的UiDevice主要介绍了一些常用的按键操作，其实最终大部分都是通过调用UiAutomator框架中的[InteractionController.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/InteractionController.java)来完成的，再通过InputManager的injectInputEvent实现的KeyEvent。

本节主要介绍UiDevice中的几种动作的实现，如点击，滑动，拖拽等.

<!--more-->

* 点击

```java
/**
 * Perform a click at arbitrary coordinates specified by the user
 *
 * @param x coordinate
 * @param y coordinate
 * @return true if the click succeeded else false
 * @since API Level 16
 */
public boolean click(int x, int y) {
    Tracer.trace(x, y);
    if (x >= getDisplayWidth() || y >= getDisplayHeight()) {
        return (false);
    }
    return getAutomatorBridge().getInteractionController().clickNoSync(x, y);
}
```

首先检查要点击的区域是否再屏幕之内，然后调用[InteractionController](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/InteractionController.java)的clickNoSync(x, y)方法。

```java
/**
 * Clicks at coordinates without waiting for device idle. This may be used for operations
 * that require stressing the target.
 * @param x
 * @param y
 * @return true if the click executed successfully
 */
public boolean clickNoSync(int x, int y) {
    Log.d(LOG_TAG, "clickNoSync (" + x + ", " + y + ")");
    if (touchDown(x, y)) {
        SystemClock.sleep(REGULAR_CLICK_LENGTH);
        if (touchUp(x, y))
            return true;
    }
    return false;
}
```

clickNoSync的方法其实就是分成了两个Key事件，一个Down和一个Up。

```java
private boolean touchDown(int x, int y) {
    if (DEBUG) {
        Log.d(LOG_TAG, "touchDown (" + x + ", " + y + ")");
    }
    mDownTime = SystemClock.uptimeMillis();
    MotionEvent event = MotionEvent.obtain(
            mDownTime, mDownTime, MotionEvent.ACTION_DOWN, x, y, 1);
    event.setSource(InputDevice.SOURCE_TOUCHSCREEN);
    return injectEventSync(event);
}
private boolean touchUp(int x, int y) {
    if (DEBUG) {
        Log.d(LOG_TAG, "touchUp (" + x + ", " + y + ")");
    }
    final long eventTime = SystemClock.uptimeMillis();
    MotionEvent event = MotionEvent.obtain(
            mDownTime, eventTime, MotionEvent.ACTION_UP, x, y, 1);
    event.setSource(InputDevice.SOURCE_TOUCHSCREEN);
    mDownTime = 0;
    return injectEventSync(event);
}
```

injectEventSync的分析已经介绍过，这里不再赘述。

* 滑动

```java
/**
 * Performs a swipe from one coordinate to another using the number of steps
 * to determine smoothness and speed. Each step execution is throttled to 5ms
 * per step. So for a 100 steps, the swipe will take about 1/2 second to complete.
 *
 * @param startX
 * @param startY
 * @param endX
 * @param endY
 * @param steps is the number of move steps sent to the system
 * @return false if the operation fails or the coordinates are invalid
 * @since API Level 16
 */
public boolean swipe(int startX, int startY, int endX, int endY, int steps) {
    Tracer.trace(startX, startY, endX, endY, steps);
    return getAutomatorBridge().getInteractionController()
            .swipe(startX, startY, endX, endY, steps);
}

/**
 * Performs a swipe between points in the Point array. Each step execution is throttled
 * to 5ms per step. So for a 100 steps, the swipe will take about 1/2 second to complete
 *
 * @param segments is Point array containing at least one Point object
 * @param segmentSteps steps to inject between two Points
 * @return true on success
 * @since API Level 16
 */
public boolean swipe(Point[] segments, int segmentSteps) {
    Tracer.trace(segments, segmentSteps);
    return getAutomatorBridge().getInteractionController().swipe(segments, segmentSteps);
}
```

滑动主要有上面的两个方法，第一个给定滑动的起始和终点坐标，以及完成这个距离滑动的步数。第二个是给定一个Point的数组，Point类同样的也会包括坐标信息，所以和第一个方法差不多。

这里需要注意的是，给定了步数steps后，在执行滑动时，会根据起始和终点坐标信息算出滑动距离每一步的坐标信息，每一步执行都会有5ms的间隙，也就是如果步数是100，那么执行完这个滑动大概需要500ms的时间，步数越小，滑动速率越快；步数越大，滑动速率越慢。

看下InteractionController中的swipe方法实现：

```java
/**
* Handle swipes in any direction.
* @param downX
* @param downY
* @param upX
* @param upY
* @param steps
* @return true if the swipe executed successfully
*/
public boolean swipe(int downX, int downY, int upX, int upY, int steps) {
   return swipe(downX, downY, upX, upY, steps, false /*drag*/);
}
/**
* Handle swipes/drags in any direction.
* @param downX
* @param downY
* @param upX
* @param upY
* @param steps
* @param drag when true, the swipe becomes a drag swipe
* @return true if the swipe executed successfully
*/
public boolean swipe(int downX, int downY, int upX, int upY, int steps, boolean drag) {
   boolean ret = false;
   int swipeSteps = steps;
   double xStep = 0;
   double yStep = 0;
   // avoid a divide by zero
   if(swipeSteps == 0)
       swipeSteps = 1;
   xStep = ((double)(upX - downX)) / swipeSteps;
   yStep = ((double)(upY - downY)) / swipeSteps;
   // first touch starts exactly at the point requested
   ret = touchDown(downX, downY);
   if (drag)
       SystemClock.sleep(mUiAutomatorBridge.getSystemLongPressTime());
   for(int i = 1; i < swipeSteps; i++) {
       ret &= touchMove(downX + (int)(xStep * i), downY + (int)(yStep * i));
       if(ret == false)
           break;
       // set some known constant delay between steps as without it this
       // become completely dependent on the speed of the system and results
       // may vary on different devices. This guarantees at minimum we have
       // a preset delay.
       SystemClock.sleep(MOTION_EVENT_INJECTION_DELAY_MILLIS);
   }
   if (drag)
       SystemClock.sleep(REGULAR_CLICK_LENGTH);
   ret &= touchUp(upX, upY);
   return(ret);
}
/**
* Performs a swipe between points in the Point array.
* @param segments is Point array containing at least one Point object
* @param segmentSteps steps to inject between two Points
* @return true on success
*/
public boolean swipe(Point[] segments, int segmentSteps) {
   boolean ret = false;
   int swipeSteps = segmentSteps;
   double xStep = 0;
   double yStep = 0;
   // avoid a divide by zero
   if(segmentSteps == 0)
       segmentSteps = 1;
   // must have some points
   if(segments.length == 0)
       return false;
   // first touch starts exactly at the point requested
   ret = touchDown(segments[0].x, segments[0].y);
   for(int seg = 0; seg < segments.length; seg++) {
       if(seg + 1 < segments.length) {
           xStep = ((double)(segments[seg+1].x - segments[seg].x)) / segmentSteps;
           yStep = ((double)(segments[seg+1].y - segments[seg].y)) / segmentSteps;
           for(int i = 1; i < swipeSteps; i++) {
               ret &= touchMove(segments[seg].x + (int)(xStep * i),
                       segments[seg].y + (int)(yStep * i));
               if(ret == false)
                   break;
               // set some known constant delay between steps as without it this
               // become completely dependent on the speed of the system and results
               // may vary on different devices. This guarantees at minimum we have
               // a preset delay.
               SystemClock.sleep(MOTION_EVENT_INJECTION_DELAY_MILLIS);
           }
       }
   }
   ret &= touchUp(segments[segments.length - 1].x, segments[segments.length -1].y);
   return(ret);
}
```

先来看一下swipe给定起始和终点坐标的实现方式，从代码中可以看出实现一个滑动操作的主要方式是:DOWN->MOVE->UP,这里需要注意的是，当步数大于1时，MOVE的个数等于steps-1次。根据起始坐标和终点坐标分别计算出每一步的MOVE坐标。MOTION_EVENT_INJECTION_DELAY_MILLIS的时间为5ms，REGULAR_CLICK_LENGTH为100ms。

使用Point的实现方式和给定起始和终点坐标的方式类似，DOWN的坐标为segments的第一个Point，UP为segments的最后一个，然后会根据给定的segments的长度，从0开始到segments.length-1的Point中分别计算出每两个点之间的距离，然后除以segmentSteps算出每一个步点的坐标执行MOVE事件。

* 拖拽

```java
/**
 * Performs a swipe from one coordinate to another coordinate. You can control
 * the smoothness and speed of the swipe by specifying the number of steps.
 * Each step execution is throttled to 5 milliseconds per step, so for a 100
 * steps, the swipe will take around 0.5 seconds to complete.
 *
 * @param startX X-axis value for the starting coordinate
 * @param startY Y-axis value for the starting coordinate
 * @param endX X-axis value for the ending coordinate
 * @param endY Y-axis value for the ending coordinate
 * @param steps is the number of steps for the swipe action
 * @return true if swipe is performed, false if the operation fails
 * or the coordinates are invalid
 * @since API Level 18
 */
public boolean drag(int startX, int startY, int endX, int endY, int steps) {
    Tracer.trace(startX, startY, endX, endY, steps);
    return getAutomatorBridge().getInteractionController()
            .swipe(startX, startY, endX, endY, steps, true);
}
```

其实，在分析滑动的实现时，swipe的最后一个方法参数drag就是用来区分当前操作时滑动还是拖拽的。从UiDevice的代码中可以看出，其drag方法就是通过调用InteractionController的swipe方法实现的。

拖拽和滑动的不同点就是，在第一个DOWN事件发生后要休眠一个长按的常量时间。然后在所有的MOVE事件结束后，要休眠一个点击的常量时间，那么问题来了，长按的常量时间和点击的常量时间分别时多少呢？

```java
if (drag)
    SystemClock.sleep(mUiAutomatorBridge.getSystemLongPressTime());
...
//MOVE
...
if (drag)
    SystemClock.sleep(REGULAR_CLICK_LENGTH);
```
mUiAutomatorBridge是一个抽象方法，具体的实现[InstrumentationUiAutomatorBridge.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/instrumentation/testrunner-src/com/android/uiautomator/core/InstrumentationUiAutomatorBridge.java)最终是通过ViewConfiguration.getLongPressTimeout()获取的.

REGULAR_CLICK_LENGTH在[InteractionController](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/InteractionController.java)定义成了100ms.

```java
    private static final long REGULAR_CLICK_LENGTH = 100;
```
