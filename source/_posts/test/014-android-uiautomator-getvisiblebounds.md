---
title: Android UiAutomator-How to get a UiObject's visible bound
date: 2017-09-06 22:12:14
tags:
categories: "测试开发"
---

在使用Android的UiAutomatorViewer中可以看到控件的信息一栏中有获取其的Bounds，也就是控件的所占的区域。

其实我们从上一节中可以看到具体的获取是通过如下的方式：

```java
serializer.attribute("", "bounds", AccessibilityNodeInfoHelper.getVisibleBoundsInScreen(
        node, width, height).toShortString());
```

<!--more-->

也就是调用了一个帮助类AccessibilityNodeInfoHelper，通过查看其源码实现，此类只实现了这一个方法：

```java
/**
 * This class contains static helper methods to work with
 * {@link AccessibilityNodeInfo}
 */
class AccessibilityNodeInfoHelper {

    /**
     * Returns the node's bounds clipped to the size of the display
     *
     * @param node
     * @param width pixel width of the display
     * @param height pixel height of the display
     * @return null if node is null, else a Rect containing visible bounds
     */
    static Rect getVisibleBoundsInScreen(AccessibilityNodeInfo node, int width, int height) {
        if (node == null) {
            return null;
        }
        // targeted node's bounds
        Rect nodeRect = new Rect();
        node.getBoundsInScreen(nodeRect);

        Rect displayRect = new Rect();
        displayRect.top = 0;
        displayRect.left = 0;
        displayRect.right = width;
        displayRect.bottom = height;

        nodeRect.intersect(displayRect);
        return nodeRect;
    }
}
```

先通过AccessibilityNodeInfo的getBoundsInScreen方法，获取这个控件在屏幕中的坐标区域，

```java
/**
 * Gets the node bounds in screen coordinates.
 *
 * @param outBounds The output node bounds.
 */
public void getBoundsInScreen(Rect outBounds) {
    outBounds.set(mBoundsInScreen.left, mBoundsInScreen.top,
            mBoundsInScreen.right, mBoundsInScreen.bottom);
}
/**
 * Returns the actual rect containing the node bounds in screen coordinates.
 *
 * @hide Not safe to expose outside the framework.
 */
public Rect getBoundsInScreen() {
    return mBoundsInScreen;
}
/**
 * Sets the node bounds in screen coordinates.
 * <p>
 *   <strong>Note:</strong> Cannot be called from an
 *   {@link android.accessibilityservice.AccessibilityService}.
 *   This class is made immutable before being delivered to an AccessibilityService.
 * </p>
 *
 * @param bounds The node bounds.
 *
 * @throws IllegalStateException If called from an AccessibilityService.
 */
public void setBoundsInScreen(Rect bounds) {
    enforceNotSealed();
    mBoundsInScreen.set(bounds.left, bounds.top, bounds.right, bounds.bottom);
}
```

上面的代码片段就是AccessibilityNodeInfo中设置和获取当前的控件在屏幕中的坐标区域方法。

然后再通过Rect的相交方法，取当前屏幕和AccessibilityNodeInfo中获取的坐标区域（其可能会超过屏幕的范围）的重叠点。

```java
/**
 * If the rectangle specified by left,top,right,bottom intersects this
 * rectangle, return true and set this rectangle to that intersection,
 * otherwise return false and do not change this rectangle. No check is
 * performed to see if either rectangle is empty. Note: To just test for
 * intersection, use {@link #intersects(Rect, Rect)}.
 *
 * @param left The left side of the rectangle being intersected with this
 *             rectangle
 * @param top The top of the rectangle being intersected with this rectangle
 * @param right The right side of the rectangle being intersected with this
 *              rectangle.
 * @param bottom The bottom of the rectangle being intersected with this
 *             rectangle.
 * @return true if the specified rectangle and this rectangle intersect
 *              (and this rectangle is then set to that intersection) else
 *              return false and do not change this rectangle.
 */
@CheckResult
public boolean intersect(int left, int top, int right, int bottom) {
    if (this.left < right && left < this.right && this.top < bottom && top < this.bottom) {
        if (this.left < left) this.left = left;
        if (this.top < top) this.top = top;
        if (this.right > right) this.right = right;
        if (this.bottom > bottom) this.bottom = bottom;
        return true;
    }
    return false;
}

/**
 * If the specified rectangle intersects this rectangle, return true and set
 * this rectangle to that intersection, otherwise return false and do not
 * change this rectangle. No check is performed to see if either rectangle
 * is empty. To just test for intersection, use intersects()
 *
 * @param r The rectangle being intersected with this rectangle.
 * @return true if the specified rectangle and this rectangle intersect
 *              (and this rectangle is then set to that intersection) else
 *              return false and do not change this rectangle.
 */
@CheckResult
public boolean intersect(Rect r) {
    return intersect(r.left, r.top, r.right, r.bottom);
}
```

先判断是否给定的四个方向值是否和当前的Rect有交集，如果没有，则返回false；如果有交集，则会分别比对四个方向，并返回true。

### 源码路径

* [AccessibilityNodeInfoHelper.java](https://android.googlesource.com/platform/frameworks/uiautomator/+/master/src/com/android/uiautomator/core/AccessibilityNodeInfoHelper.java)

* [AccessibilityNodeInfo.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/accessibility/AccessibilityNodeInfo.java)

* [Rect.java](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/graphics/java/android/graphics/Rect.java)
