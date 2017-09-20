---
title: Android UiAutomator UiDevice（四）
date: 2017-09-20 20:53:47
tags:
categories: "测试开发"
---

这一节主要介绍[UiDevice](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiDevice.java)中的截屏实现。

首先看下[UiDevice](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiDevice.java)中提供给我们的截屏方法：

```java
/**
 * Take a screenshot of current window and store it as PNG
 *
 * Default scale of 1.0f (original size) and 90% quality is used
 * The screenshot is adjusted per screen rotation
 *
 * @param storePath where the PNG should be written to
 * @return true if screen shot is created successfully, false otherwise
 * @since API Level 17
 */
public boolean takeScreenshot(File storePath) {
    Tracer.trace(storePath);
    return takeScreenshot(storePath, 1.0f, 90);
}
/**
 * Take a screenshot of current window and store it as PNG
 *
 * The screenshot is adjusted per screen rotation
 *
 * @param storePath where the PNG should be written to
 * @param scale scale the screenshot down if needed; 1.0f for original size
 * @param quality quality of the PNG compression; range: 0-100
 * @return true if screen shot is created successfully, false otherwise
 * @since API Level 17
 */
public boolean takeScreenshot(File storePath, float scale, int quality) {
    Tracer.trace(storePath, scale, quality);
    return getAutomatorBridge().takeScreenshot(storePath, quality);
}
```

<!--more-->

有两个重载方法，第一个takeScreenshot只接受一个File的参数，指定当前截屏的保存路径，第二个takeScreenshot还接受一个scale缩放比例和quality图像质量的参数。可以看到scale这个参数根本没有用途，当传给[AutomatorBridge](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiAutomatorBridge.java)的takeScreenshot方法时，只有文件路径和图像质量两个参数。

我们进入AutomatorBridge的takeScreenshot查看下其实现逻辑：

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

从上面的代码可以看出，其实是通过[UiAutomation](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/UiAutomation.java)的takeScreenshot方法来获取当前屏幕的内容，然后返回一个Bitmap对象，后面使用Java的流对Bitmap进行保存处理。

```Java
/**
 * Takes a screenshot.
 *
 * @return The screenshot bitmap on success, null otherwise.
 */
public Bitmap takeScreenshot() {
    synchronized (mLock) {
        throwIfNotConnectedLocked();
    }
    Display display = DisplayManagerGlobal.getInstance()
            .getRealDisplay(Display.DEFAULT_DISPLAY);
    Point displaySize = new Point();
    display.getRealSize(displaySize);
    final int displayWidth = displaySize.x;
    final int displayHeight = displaySize.y;
    final float screenshotWidth;
    final float screenshotHeight;
    final int rotation = display.getRotation();
    switch (rotation) {
        case ROTATION_FREEZE_0: {
            screenshotWidth = displayWidth;
            screenshotHeight = displayHeight;
        } break;
        case ROTATION_FREEZE_90: {
            screenshotWidth = displayHeight;
            screenshotHeight = displayWidth;
        } break;
        case ROTATION_FREEZE_180: {
            screenshotWidth = displayWidth;
            screenshotHeight = displayHeight;
        } break;
        case ROTATION_FREEZE_270: {
            screenshotWidth = displayHeight;
            screenshotHeight = displayWidth;
        } break;
        default: {
            throw new IllegalArgumentException("Invalid rotation: "
                    + rotation);
        }
    }
    // Take the screenshot
    Bitmap screenShot = null;
    try {
        // Calling out without a lock held.
        screenShot = mUiAutomationConnection.takeScreenshot((int) screenshotWidth,
                (int) screenshotHeight);
        if (screenShot == null) {
            return null;
        }
    } catch (RemoteException re) {
        Log.e(LOG_TAG, "Error while taking screnshot!", re);
        return null;
    }
    // Rotate the screenshot to the current orientation
    if (rotation != ROTATION_FREEZE_0) {
        Bitmap unrotatedScreenShot = Bitmap.createBitmap(displayWidth, displayHeight,
                Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(unrotatedScreenShot);
        canvas.translate(unrotatedScreenShot.getWidth() / 2,
                unrotatedScreenShot.getHeight() / 2);
        canvas.rotate(getDegreesForRotation(rotation));
        canvas.translate(- screenshotWidth / 2, - screenshotHeight / 2);
        canvas.drawBitmap(screenShot, 0, 0, null);
        canvas.setBitmap(null);
        screenShot.recycle();
        screenShot = unrotatedScreenShot;
    }
    // Optimization
    screenShot.setHasAlpha(false);
    return screenShot;
}
```

首先，对屏幕的当前转向进行获取处理，因为如果是横向的屏幕，截屏的宽和高要互换。然后，如果不是正常的屏幕方法，也就是不等于0，那就要先旋转图像，然后返回。
