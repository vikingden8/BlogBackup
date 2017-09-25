---
title: Android UiAutomator UiObject
date: 2017-09-25 22:31:48
tags:
categories: "测试开发"
---

[UiObject](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiObject.java)中是怎样实现对控件的PinchIn和PinchOut手势操作的呢？

```java
/**
  * Performs a two-pointer gesture, where each pointer moves diagonally
  * opposite across the other, from the center out towards the edges of the
  * this UiObject.
  * @param percent percentage of the object's diagonal length for the pinch gesture
  * @param steps the number of steps for the gesture. Steps are injected
  * about 5 milliseconds apart, so 100 steps may take around 0.5 seconds to complete.
  * @return <code>true</code> if all touch events for this gesture are injected successfully,
  *         <code>false</code> otherwise
  * @throws UiObjectNotFoundException
  * @since API Level 18
  */
  public boolean pinchOut(int percent, int steps) throws UiObjectNotFoundException {
     // make value between 1 and 100
     percent = (percent < 0) ? 1 : (percent > 100) ? 100 : percent;
     float percentage = percent / 100f;
     AccessibilityNodeInfo node = findAccessibilityNodeInfo(mConfig.getWaitForSelectorTimeout());
     if (node == null) {
         throw new UiObjectNotFoundException(getSelector().toString());
     }
     Rect rect = getVisibleBounds(node);
     if (rect.width() <= FINGER_TOUCH_HALF_WIDTH * 2)
         throw new IllegalStateException("Object width is too small for operation");
     // start from the same point at the center of the control
     Point startPoint1 = new Point(rect.centerX() - FINGER_TOUCH_HALF_WIDTH, rect.centerY());
     Point startPoint2 = new Point(rect.centerX() + FINGER_TOUCH_HALF_WIDTH, rect.centerY());
     // End at the top-left and bottom-right corners of the control
     Point endPoint1 = new Point(rect.centerX() - (int)((rect.width()/2) * percentage),
             rect.centerY());
     Point endPoint2 = new Point(rect.centerX() + (int)((rect.width()/2) * percentage),
             rect.centerY());
     return performTwoPointerGesture(startPoint1, startPoint2, endPoint1, endPoint2, steps);
  }

  /**
  * Performs a two-pointer gesture, where each pointer moves diagonally
  * toward the other, from the edges to the center of this UiObject .
  * @param percent percentage of the object's diagonal length for the pinch gesture
  * @param steps the number of steps for the gesture. Steps are injected
  * about 5 milliseconds apart, so 100 steps may take around 0.5 seconds to complete.
  * @return <code>true</code> if all touch events for this gesture are injected successfully,
  *         <code>false</code> otherwise
  * @throws UiObjectNotFoundException
  * @since API Level 18
  */
  public boolean pinchIn(int percent, int steps) throws UiObjectNotFoundException {
     // make value between 1 and 100
     percent = (percent < 0) ? 0 : (percent > 100) ? 100 : percent;
     float percentage = percent / 100f;
     AccessibilityNodeInfo node = findAccessibilityNodeInfo(mConfig.getWaitForSelectorTimeout());
     if (node == null) {
         throw new UiObjectNotFoundException(getSelector().toString());
     }
     Rect rect = getVisibleBounds(node);
     if (rect.width() <= FINGER_TOUCH_HALF_WIDTH * 2)
         throw new IllegalStateException("Object width is too small for operation");
     Point startPoint1 = new Point(rect.centerX() - (int)((rect.width()/2) * percentage),
             rect.centerY());
     Point startPoint2 = new Point(rect.centerX() + (int)((rect.width()/2) * percentage),
             rect.centerY());
     Point endPoint1 = new Point(rect.centerX() - FINGER_TOUCH_HALF_WIDTH, rect.centerY());
     Point endPoint2 = new Point(rect.centerX() + FINGER_TOUCH_HALF_WIDTH, rect.centerY());
     return performTwoPointerGesture(startPoint1, startPoint2, endPoint1, endPoint2, steps);
  }
```

<!--more-->

上面就是UiObject中PinchIn和PinchOut的实现，我们只需要分析其中一个就可以了，另一个就是相反的逻辑。

* 首先确保percent在介于1~100之间，如果值小于0，则为1；如果大于100，则为100，其余就是给定的percent；再转化为float类型；

* 通过findAccessibilityNodeInfo找到此控件，如果找不到控件，直接抛出异常；

* 通过getVisibleBounds获取此控件的可见区域；

* 由于PinchIn和PinchOut是两个手指的手势操作，PinchIn是两个手指从远点往近点靠拢，PinchOut则相反。拿PinchOut来说，现在的问题是先确定两个手指的起点和终点。PinchOut的两个手势的启动是控件的中心处，注意，这里的两个手势起点并没有重叠。FINGER_TOUCH_HALF_WIDTH为20，也就是第一个手势起点startPoint1距离中心点水平向左20px，而第二个手势起点startPoint2距离中心点水平向右20px。第一个手势终点endPoint1距离中心点水平向左（宽度的一半）* percent处，而第二个手势终点endPoint2距离中心点水平向右（宽度的一半）* percent处。注意：在实现的文档中说到是top-left和bottom-right，其实不是，你可以看到两个手势的起点和终点都是控件Y轴的中心处。

* 然后交给performTwoPointerGesture函数处理。

```java
/**
   * Generates a two-pointer gesture with arbitrary starting and ending points.
   *
   * @param startPoint1 start point of pointer 1
   * @param startPoint2 start point of pointer 2
   * @param endPoint1 end point of pointer 1
   * @param endPoint2 end point of pointer 2
   * @param steps the number of steps for the gesture. Steps are injected
   * about 5 milliseconds apart, so 100 steps may take around 0.5 seconds to complete.
   * @return <code>true</code> if all touch events for this gesture are injected successfully,
   *         <code>false</code> otherwise
   * @since API Level 18
   */
  public boolean performTwoPointerGesture(Point startPoint1, Point startPoint2, Point endPoint1,
          Point endPoint2, int steps) {
      // avoid a divide by zero
      if(steps == 0)
          steps = 1;
      final float stepX1 = (endPoint1.x - startPoint1.x) / steps;
      final float stepY1 = (endPoint1.y - startPoint1.y) / steps;
      final float stepX2 = (endPoint2.x - startPoint2.x) / steps;
      final float stepY2 = (endPoint2.y - startPoint2.y) / steps;
      int eventX1, eventY1, eventX2, eventY2;
      eventX1 = startPoint1.x;
      eventY1 = startPoint1.y;
      eventX2 = startPoint2.x;
      eventY2 = startPoint2.y;
      // allocate for steps plus first down and last up
      PointerCoords[] points1 = new PointerCoords[steps + 2];
      PointerCoords[] points2 = new PointerCoords[steps + 2];
      // Include the first and last touch downs in the arrays of steps
      for (int i = 0; i < steps + 1; i++) {
          PointerCoords p1 = new PointerCoords();
          p1.x = eventX1;
          p1.y = eventY1;
          p1.pressure = 1;
          p1.size = 1;
          points1[i] = p1;
          PointerCoords p2 = new PointerCoords();
          p2.x = eventX2;
          p2.y = eventY2;
          p2.pressure = 1;
          p2.size = 1;
          points2[i] = p2;
          eventX1 += stepX1;
          eventY1 += stepY1;
          eventX2 += stepX2;
          eventY2 += stepY2;
      }
      // ending pointers coordinates
      PointerCoords p1 = new PointerCoords();
      p1.x = endPoint1.x;
      p1.y = endPoint1.y;
      p1.pressure = 1;
      p1.size = 1;
      points1[steps + 1] = p1;
      PointerCoords p2 = new PointerCoords();
      p2.x = endPoint2.x;
      p2.y = endPoint2.y;
      p2.pressure = 1;
      p2.size = 1;
      points2[steps + 1] = p2;
      return performMultiPointerGesture(points1, points2);
  }
  /**
   * Performs a multi-touch gesture. You must specify touch coordinates for
   * at least 2 pointers. Each pointer must have all of its touch steps
   * defined in an array of {@link PointerCoords}. You can use this method to
   * specify complex gestures, like circles and irregular shapes, where each
   * pointer may take a different path.
   *
   * To create a single point on a pointer's touch path:
   * <code>
   *       PointerCoords p = new PointerCoords();
   *       p.x = stepX;
   *       p.y = stepY;
   *       p.pressure = 1;
   *       p.size = 1;
   * </code>
   * @param touches represents the pointers' paths. Each {@link PointerCoords}
   * array represents a different pointer. Each {@link PointerCoords} in an
   * array element represents a touch point on a pointer's path.
   * @return <code>true</code> if all touch events for this gesture are injected successfully,
   *         <code>false</code> otherwise
   * @since API Level 18
   */
  public boolean performMultiPointerGesture(PointerCoords[] ...touches) {
      return getInteractionController().performMultiPointerGesture(touches);
  }
```

当确认了两个手势的启动和终点后，就可以根据给定steps来确定手势的轨迹坐标了，这就是performTwoPointerGesture的作用：

* 根据给定steps以及手势的起点和终点算出每一步的步数像素值stepX1，stepY1，stepX2，stepY2；

* 创建steps+2的两个PointerCoords数组points1，points2；

* 循环steps+1次，以两个手势的起点为开始，每次增加stepX1，stepY1，stepX2，stepY2；

* 以两个手势的终点分别作为points1，points2的最后一个点。

* 然后调用performMultiPointerGesture的可变参数方法。

代码转到[InteractionController](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/InteractionController.java)的performMultiPointerGesture：

```java
/**
   * Performs a multi-touch gesture
   *
   * Takes a series of touch coordinates for at least 2 pointers. Each pointer must have
   * all of its touch steps defined in an array of {@link PointerCoords}. By having the ability
   * to specify the touch points along the path of a pointer, the caller is able to specify
   * complex gestures like circles, irregular shapes etc, where each pointer may take a
   * different path.
   *
   * To create a single point on a pointer's touch path
   * <code>
   *       PointerCoords p = new PointerCoords();
   *       p.x = stepX;
   *       p.y = stepY;
   *       p.pressure = 1;
   *       p.size = 1;
   * </code>
   * @param touches each array of {@link PointerCoords} constitute a single pointer's touch path.
   *        Multiple {@link PointerCoords} arrays constitute multiple pointers, each with its own
   *        path. Each {@link PointerCoords} in an array constitute a point on a pointer's path.
   * @return <code>true</code> if all points on all paths are injected successfully, <code>false
   *        </code>otherwise
   * @since API Level 18
   */
  public boolean performMultiPointerGesture(PointerCoords[] ... touches) {
      // touches数组方法参数的长度必须大于等于2
      boolean ret = true;
      if (touches.length < 2) {
          throw new IllegalArgumentException("Must provide coordinates for at least 2 pointers");
      }
      // 由于是多个坐标集数组，获取最大的的步数
      // Get the pointer with the max steps to inject.
      int maxSteps = 0;
      for (int x = 0; x < touches.length; x++)
          maxSteps = (maxSteps < touches[x].length) ? touches[x].length : maxSteps;
      // 设置每一个手势都为手指，并把每个手势的第一个设置为TOUCH_DOWN
      // specify the properties for each pointer as finger touch
      PointerProperties[] properties = new PointerProperties[touches.length];
      PointerCoords[] pointerCoords = new PointerCoords[touches.length];
      for (int x = 0; x < touches.length; x++) {
          PointerProperties prop = new PointerProperties();
          prop.id = x;
          prop.toolType = MotionEvent.TOOL_TYPE_FINGER;
          properties[x] = prop;
          // for each pointer set the first coordinates for touch down
          pointerCoords[x] = touches[x][0];
      }
      // Touch down all pointers
      // 同时按下所有坐标集的
      long downTime = SystemClock.uptimeMillis();
      MotionEvent event;
      event = MotionEvent.obtain(downTime, SystemClock.uptimeMillis(), MotionEvent.ACTION_DOWN, 1,
              properties, pointerCoords, 0, 0, 1, 1, 0, 0, InputDevice.SOURCE_TOUCHSCREEN, 0);
      ret &= injectEventSync(event);
      // ACTION_POINTER_DOWN:代表用户又使用一个手指触摸到屏幕上，也就是说，在已经有一个触摸点的情况下，有新出现了一个触摸点。
      for (int x = 1; x < touches.length; x++) {
          event = MotionEvent.obtain(downTime, SystemClock.uptimeMillis(),
                  getPointerAction(MotionEvent.ACTION_POINTER_DOWN, x), x + 1, properties,
                  pointerCoords, 0, 0, 1, 1, 0, 0, InputDevice.SOURCE_TOUCHSCREEN, 0);
          ret &= injectEventSync(event);
      }
      // 同时移动所有的坐标点
      // Move all pointers
      for (int i = 1; i < maxSteps - 1; i++) {
          // for each pointer
          for (int x = 0; x < touches.length; x++) {
              // check if it has coordinates to move
              if (touches[x].length > i)
                  pointerCoords[x] = touches[x][i];
              else
                  pointerCoords[x] = touches[x][touches[x].length - 1];
          }
          event = MotionEvent.obtain(downTime, SystemClock.uptimeMillis(),
                  MotionEvent.ACTION_MOVE, touches.length, properties, pointerCoords, 0, 0, 1, 1,
                  0, 0, InputDevice.SOURCE_TOUCHSCREEN, 0);
          ret &= injectEventSync(event);
          SystemClock.sleep(MOTION_EVENT_INJECTION_DELAY_MILLIS);
      }
      // 获取每个手势的最后坐标点
      // For each pointer get the last coordinates
      for (int x = 0; x < touches.length; x++)
          pointerCoords[x] = touches[x][touches[x].length - 1];
      // 每个手势出发TOUCH_UP操作
      // touch up
      for (int x = 1; x < touches.length; x++) {
          event = MotionEvent.obtain(downTime, SystemClock.uptimeMillis(),
                  getPointerAction(MotionEvent.ACTION_POINTER_UP, x), x + 1, properties,
                  pointerCoords, 0, 0, 1, 1, 0, 0, InputDevice.SOURCE_TOUCHSCREEN, 0);
          ret &= injectEventSync(event);
      }
      Log.i(LOG_TAG, "x " + pointerCoords[0].x);
      // first to touch down is last up
      event = MotionEvent.obtain(downTime, SystemClock.uptimeMillis(), MotionEvent.ACTION_UP, 1,
              properties, pointerCoords, 0, 0, 1, 1, 0, 0, InputDevice.SOURCE_TOUCHSCREEN, 0);
      ret &= injectEventSync(event);
      return ret;
  }

```
