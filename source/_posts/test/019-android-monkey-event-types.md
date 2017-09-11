---
title: Android Monkey随机事件
date: 2017-09-11 23:02:06
tags:
categories: "测试开发"
---

本文是针对于最新的Monkey源码进行的分析，最近发现Monkey在Android 8.0的版本上改动比较大，多增加了几种随机事件，优化了Log的输出等等，本文主要是针对Monkey的随机事件类型进行分析.

通过查看[MonkeySourceRandom.java](https://android.googlesource.com/platform/development/+/master/cmds/monkey/src/com/android/commands/monkey/MonkeySourceRandom.java)中的源码：

```java

    public static final int FACTOR_TOUCH        = 0;
    public static final int FACTOR_MOTION       = 1;
    public static final int FACTOR_PINCHZOOM    = 2;
    public static final int FACTOR_TRACKBALL    = 3;
    public static final int FACTOR_ROTATION     = 4;
    public static final int FACTOR_PERMISSION   = 5;
    public static final int FACTOR_NAV          = 6;
    public static final int FACTOR_MAJORNAV     = 7;
    public static final int FACTOR_SYSOPS       = 8;
    public static final int FACTOR_APPSWITCH    = 9;
    public static final int FACTOR_FLIP         = 10;
    public static final int FACTOR_ANYTHING     = 11;
    public static final int FACTORZ_COUNT       = 12;    // should be last+1
```

可以看出主要是有12种事件。

<!--more-->

所有的事件生成都会在generateEvents中，生成后的事件保存到了一个mQ中，也就是一个队列，遵循先进先出的原则。

```java
/**
 * generate a random event based on mFactor
 */
private void generateEvents() {
    float cls = mRandom.nextFloat();
    int lastKey = 0;
    if (cls < mFactors[FACTOR_TOUCH]) {
        generatePointerEvent(mRandom, GESTURE_TAP);
        return;
    } else if (cls < mFactors[FACTOR_MOTION]) {
        generatePointerEvent(mRandom, GESTURE_DRAG);
        return;
    } else if (cls < mFactors[FACTOR_PINCHZOOM]) {
        generatePointerEvent(mRandom, GESTURE_PINCH_OR_ZOOM);
        return;
    } else if (cls < mFactors[FACTOR_TRACKBALL]) {
        generateTrackballEvent(mRandom);
        return;
    } else if (cls < mFactors[FACTOR_ROTATION]) {
        generateRotationEvent(mRandom);
        return;
    } else if (cls < mFactors[FACTOR_PERMISSION]) {
        mQ.add(mPermissionUtil.generateRandomPermissionEvent(mRandom));
        return;
    }
    // The remaining event categories are injected as key events
    for (;;) {
        if (cls < mFactors[FACTOR_NAV]) {
            lastKey = NAV_KEYS[mRandom.nextInt(NAV_KEYS.length)];
        } else if (cls < mFactors[FACTOR_MAJORNAV]) {
            lastKey = MAJOR_NAV_KEYS[mRandom.nextInt(MAJOR_NAV_KEYS.length)];
        } else if (cls < mFactors[FACTOR_SYSOPS]) {
            lastKey = SYS_KEYS[mRandom.nextInt(SYS_KEYS.length)];
        } else if (cls < mFactors[FACTOR_APPSWITCH]) {
            MonkeyActivityEvent e = new MonkeyActivityEvent(mMainApps.get(
                    mRandom.nextInt(mMainApps.size())));
            mQ.addLast(e);
            return;
        } else if (cls < mFactors[FACTOR_FLIP]) {
            MonkeyFlipEvent e = new MonkeyFlipEvent(mKeyboardOpen);
            mKeyboardOpen = !mKeyboardOpen;
            mQ.addLast(e);
            return;
        } else {
            lastKey = 1 + mRandom.nextInt(KeyEvent.getMaxKeyCode() - 1);
        }
        if (lastKey != KeyEvent.KEYCODE_POWER
                && lastKey != KeyEvent.KEYCODE_ENDCALL
                && lastKey != KeyEvent.KEYCODE_SLEEP
                && lastKey != KeyEvent.KEYCODE_SOFT_SLEEP
                && PHYSICAL_KEY_EXISTS[lastKey]) {
            break;
        }
    }
    MonkeyKeyEvent e = new MonkeyKeyEvent(KeyEvent.ACTION_DOWN, lastKey);
    mQ.addLast(e);
    e = new MonkeyKeyEvent(KeyEvent.ACTION_UP, lastKey);
    mQ.addLast(e);
}
```

* TOUCH

主要是点击事件，具体转换的KeyEvent有一个ACTION_DOWN和ACTION_UP事件。对应的方法为：generatePointerEvent(mRandom, GESTURE_TAP);

generatePointerEvent中的第二个参数指定了坐标事件的生成类型。主要有三种：

```java
//点击
private static final int GESTURE_TAP = 0;
//拖拽
private static final int GESTURE_DRAG = 1;
//捏合或者放大
private static final int GESTURE_PINCH_OR_ZOOM = 2;
```

下面看下generatePointerEvent的具体实现：

```java
/**
 * Generates a random motion event. This method counts a down, move, and up as multiple events.
 *
 * TODO:  Test & fix the selectors when non-zero percentages
 * TODO:  Longpress.
 * TODO:  Fling.
 * TODO:  Meta state
 * TODO:  More useful than the random walk here would be to pick a single random direction
 * and distance, and divvy it up into a random number of segments.  (This would serve to
 * generate fling gestures, which are important).
 *
 * @param random Random number source for positioning
 * @param gesture The gesture to perform.
 *
 */
private void generatePointerEvent(Random random, int gesture) {
    Display display = DisplayManagerGlobal.getInstance().getRealDisplay(Display.DEFAULT_DISPLAY);
    PointF p1 = randomPoint(random, display);
    PointF v1 = randomVector(random);
    long downAt = SystemClock.uptimeMillis();
    mQ.addLast(new MonkeyTouchEvent(MotionEvent.ACTION_DOWN)
            .setDownTime(downAt)
            .addPointer(0, p1.x, p1.y)
            .setIntermediateNote(false));
    // sometimes we'll move during the touch
    if (gesture == GESTURE_DRAG) {
        int count = random.nextInt(10);
        for (int i = 0; i < count; i++) {
            randomWalk(random, display, p1, v1);
            mQ.addLast(new MonkeyTouchEvent(MotionEvent.ACTION_MOVE)
                    .setDownTime(downAt)
                    .addPointer(0, p1.x, p1.y)
                    .setIntermediateNote(true));
        }
    } else if (gesture == GESTURE_PINCH_OR_ZOOM) {
        PointF p2 = randomPoint(random, display);
        PointF v2 = randomVector(random);
        randomWalk(random, display, p1, v1);
        mQ.addLast(new MonkeyTouchEvent(MotionEvent.ACTION_POINTER_DOWN
                        | (1 << MotionEvent.ACTION_POINTER_INDEX_SHIFT))
                .setDownTime(downAt)
                .addPointer(0, p1.x, p1.y).addPointer(1, p2.x, p2.y)
                .setIntermediateNote(true));
        int count = random.nextInt(10);
        for (int i = 0; i < count; i++) {
            randomWalk(random, display, p1, v1);
            randomWalk(random, display, p2, v2);
            mQ.addLast(new MonkeyTouchEvent(MotionEvent.ACTION_MOVE)
                    .setDownTime(downAt)
                    .addPointer(0, p1.x, p1.y).addPointer(1, p2.x, p2.y)
                    .setIntermediateNote(true));
        }
        randomWalk(random, display, p1, v1);
        randomWalk(random, display, p2, v2);
        mQ.addLast(new MonkeyTouchEvent(MotionEvent.ACTION_POINTER_UP
                        | (1 << MotionEvent.ACTION_POINTER_INDEX_SHIFT))
                .setDownTime(downAt)
                .addPointer(0, p1.x, p1.y).addPointer(1, p2.x, p2.y)
                .setIntermediateNote(true));
    }
    randomWalk(random, display, p1, v1);
    mQ.addLast(new MonkeyTouchEvent(MotionEvent.ACTION_UP)
            .setDownTime(downAt)
            .addPointer(0, p1.x, p1.y)
            .setIntermediateNote(false));
}

private PointF randomPoint(Random random, Display display) {
    return new PointF(random.nextInt(display.getWidth()), random.nextInt(display.getHeight()));
}

private PointF randomVector(Random random) {
    return new PointF((random.nextFloat() - 0.5f) * 50, (random.nextFloat() - 0.5f) * 50);
}

private void randomWalk(Random random, Display display, PointF point, PointF vector) {
    point.x = (float) Math.max(Math.min(point.x + random.nextFloat() * vector.x,
            display.getWidth()), 0);
    point.y = (float) Math.max(Math.min(point.y + random.nextFloat() * vector.y,
            display.getHeight()), 0);
}
```

从上可以看出GESTURE_TAP，只是随机生成一个ACTION_DOWN和ACTION_UP，而GESTURE_DRAG和GESTURE_PINCH_OR_ZOOM除了前面两个之外还会在中间生成对应动作的事件。如GESTURE_DRAG会在中间生成10个随机的ACTION_MOVE事件，而GESTURE_PINCH_OR_ZOOM会生成类似两个手指的捏合或者放大的操作，其中的移动事件（ACTION_MOVE）也有10个。

* MOTION

拖拽，Touch中有介绍

* PINCHZOOM

双指捏合或者放大，Touch中有介绍

* TRACKBALL

轨迹球事件,具体实现就是随机生成10个ACTION_MOVE事件，并且有10%的概率添加一个TOUCH事件，也就是ACTION_DOWN和ACTION_UP事件。

```java
/**
  * Generates a random trackball event. This consists of a sequence of small moves, followed by
  * an optional single click.
  *
  * TODO:  Longpress.
  * TODO:  Meta state
  * TODO:  Parameterize the % clicked
  * TODO:  More useful than the random walk here would be to pick a single random direction
  * and distance, and divvy it up into a random number of segments.  (This would serve to
  * generate fling gestures, which are important).
  *
  * @param random Random number source for positioning
  *
  */
 private void generateTrackballEvent(Random random) {
     for (int i = 0; i < 10; ++i) {
         // generate a small random step
         int dX = random.nextInt(10) - 5;
         int dY = random.nextInt(10) - 5;
         mQ.addLast(new MonkeyTrackballEvent(MotionEvent.ACTION_MOVE)
                 .addPointer(0, dX, dY)
                 .setIntermediateNote(i > 0));
     }
     // 10% of trackball moves end with a click
     if (0 == random.nextInt(10)) {
         long downAt = SystemClock.uptimeMillis();
         mQ.addLast(new MonkeyTrackballEvent(MotionEvent.ACTION_DOWN)
                 .setDownTime(downAt)
                 .addPointer(0, 0, 0)
                 .setIntermediateNote(true));
         mQ.addLast(new MonkeyTrackballEvent(MotionEvent.ACTION_UP)
                 .setDownTime(downAt)
                 .addPointer(0, 0, 0)
                 .setIntermediateNote(false));
     }
 }
 ```

 * ROTATION

 屏幕旋转事件，具体的实现：

 ```java
 /**
 * Generates a random screen rotation event.
 *
 * @param random Random number source for rotation degree.
 */
private void generateRotationEvent(Random random) {
    mQ.addLast(new MonkeyRotationEvent(
            SCREEN_ROTATION_DEGREES[random.nextInt(
                    SCREEN_ROTATION_DEGREES.length)],
            random.nextBoolean()));
}
```

MonkeyRotationEvent的第一个参数意思是随机角度，有四个

```java
/** Possible screen rotation degrees **/
private static final int[] SCREEN_ROTATION_DEGREES = {
  Surface.ROTATION_0,
  Surface.ROTATION_90,
  Surface.ROTATION_180,
  Surface.ROTATION_270,
};
```

第二个参数为随机布尔值，也就是在执行旋转后是否保持旋转后的状态，如果为true，则保持；如果false，则解除旋转。

* PERMISSION

权限事件，这个也是新加的事件类型。实现原来就是通过在测试前获取所有应用的敏感权限，然后随机生成对应包名的对应权限事件，具体的权限事件最终会根据授予的状态分别执行不同的权限动作：

```java
@Override
public int injectEvent(IWindowManager iwm, IActivityManager iam, int verbose) {
    IPackageManager pm = IPackageManager.Stub.asInterface(ServiceManager.getService("package"));
    try {
        // determine if we should grant or revoke permission
        int perm = pm.checkPermission(mPermissionInfo.name, mPkg, UserHandle.myUserId());
        boolean grant = perm == PackageManager.PERMISSION_DENIED;
        // log before calling pm in case we hit an error
        Logger.out.println(String.format(":Permission %s %s to package %s",
                grant ? "grant" : "revoke", mPermissionInfo.name, mPkg));
        if (grant) {
            pm.grantRuntimePermission(mPkg, mPermissionInfo.name, UserHandle.myUserId());
        } else {
            pm.revokeRuntimePermission(mPkg, mPermissionInfo.name, UserHandle.myUserId());
        }
        return MonkeyEvent.INJECT_SUCCESS;
    } catch (RemoteException re) {
        return MonkeyEvent.INJECT_ERROR_REMOTE_EXCEPTION;
    }
}

```

* NAV

导航事件，看下具体的定义：

```java
/** Key events that move around the UI. */
private static final int[] NAV_KEYS = {
    KeyEvent.KEYCODE_DPAD_UP, KeyEvent.KEYCODE_DPAD_DOWN,
    KeyEvent.KEYCODE_DPAD_LEFT, KeyEvent.KEYCODE_DPAD_RIGHT,
};
```

DPAD按键其实就是对应与老Android手机中的上下左右按键，现在的Android手机连接的手柄中的上下左右方向也是DPAD按键。

* MAJORNAV

MAJOR的翻译大概就是主要的，下面是具体的定义，也就是KEYCODE_MENU和KEYCODE_DPAD_CENTER（中心按键）事件：

```java
/**
 * Key events that perform major navigation options (so shouldn't be sent
 * as much).
 */
private static final int[] MAJOR_NAV_KEYS = {
    KeyEvent.KEYCODE_MENU, /*KeyEvent.KEYCODE_SOFT_RIGHT,*/
    KeyEvent.KEYCODE_DPAD_CENTER,
};
```

* SYSOPS

系统按键，通过查看源码，具体的定义如下：

```java
/** Key events that perform system operations. */
private static final int[] SYS_KEYS = {
    KeyEvent.KEYCODE_HOME, KeyEvent.KEYCODE_BACK,
    KeyEvent.KEYCODE_CALL, KeyEvent.KEYCODE_ENDCALL,
    KeyEvent.KEYCODE_VOLUME_UP, KeyEvent.KEYCODE_VOLUME_DOWN, KeyEvent.KEYCODE_VOLUME_MUTE,
    KeyEvent.KEYCODE_MUTE,
};
```

当然，在测试开始之前，Monkey会判断物理按键是否存在：

```java
/** If a physical key exists? */
private static final boolean[] PHYSICAL_KEY_EXISTS = new boolean[KeyEvent.getMaxKeyCode() + 1];
static {
    for (int i = 0; i < PHYSICAL_KEY_EXISTS.length; ++i) {
        PHYSICAL_KEY_EXISTS[i] = true;
    }
    // Only examine SYS_KEYS
    for (int i = 0; i < SYS_KEYS.length; ++i) {
        PHYSICAL_KEY_EXISTS[SYS_KEYS[i]] = KeyCharacterMap.deviceHasKey(SYS_KEYS[i]);
    }
}
```

* APPSWITCH

对应的是给定的应用中，比如指定测试三个应用会在三个应用中随机切换主Activity进行测试。

* FLIP

这个对应的是键盘的滑动事件

* ANYTHING

其对应的是随机生成的KeyCode事件，值的范围不大于KeyEvent.getMaxKeyCode()。

```java
 lastKey = 1 + mRandom.nextInt(KeyEvent.getMaxKeyCode() - 1);

```

会注意到其中有一个for循环，如果前面的事件都不满足生成，则会生成lastKey，但是lastKey也有限制：

```java
if (lastKey != KeyEvent.KEYCODE_POWER
                    && lastKey != KeyEvent.KEYCODE_ENDCALL
                    && lastKey != KeyEvent.KEYCODE_SLEEP
                    && lastKey != KeyEvent.KEYCODE_SOFT_SLEEP
                    && PHYSICAL_KEY_EXISTS[lastKey]) {
                break;
}
```

只有lastKey满足上述的状态后才会跳出循环，生成随机的KeyCode事件。

至此Monkey的事件类型分析完毕！
