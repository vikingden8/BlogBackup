---
title: Android视频录制－－MediaProjection
date: 2017-07-17 17:45:44
tags:
categories: "Android学习笔记"
---

Android在5.0系统之前，是没有开放视频录制的接口的，如果要录制视频，必须要先root，这部分我会在随后的博客中细讲。
在5.0，Google终于开放了视频录制的接口（其实严格来说，是屏幕采集的接口），也就是MediaProjection和MediaProjectionManager。

首先来说MediaProjectionManager，它是一个系统级的服务，类似WindowManager，AlarmManager等，你可以通过getSystemService方法来获取它的实例：

```Java
MediaProjectionManager mMediaProjectionManager = (MediaProjectionManager) getSystemService(MEDIA_PROJECTION_SERVICE);
```

获取到实例后，录像的过程如下（有点像拍照的流程）：
首先：

```Java
Intent captureIntent = mMediaProjectionManager.createScreenCaptureIntent();
startActivityForResult(captureIntent, REQUEST_CODE);
```

<!--more-->

createScreenCaptureIntent方法的注释如下：

```
/**
 * Returns an Intent that <b>must</b> passed to startActivityForResult()
 * in order to start screen capture. The activity will prompt
 * the user whether to allow screen capture.  The result of this
 * activity should be passed to getMediaProjection.
 */
大致意思是，这个方法会返回一个intent，你可以通过startActivityForResult方法来传递这个intent，为了能开始屏幕捕捉，activity会提示用户是否允许屏幕捕捉（为了防止开发者做一个木马，来捕获用户私人信息），你可以通过getMediaProjection来获取屏幕捕捉的结果。
```

createScreenCaptureIntent的代码我们可以看一下：

```Java
public Intent createScreenCaptureIntent() {
        Intent i = new Intent();
        i.setClassName("com.android.systemui", "com.android.systemui.media.MediaProjectionPermissionActivity");
        return i;
    }
```

看着很眼熟是吧，拍照的是这样：


```java
Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
startActivityForResult(intent, REQUEST_CODE_LAUNCHCAMERA);
```

所以这里是创建了一个隐式的intent，用来调用系统的录屏程序。

然后正如上面的注释所说，我们通过startActivityForResult来传递这个intent，所以我们可以通过onActivityResult来获取结果，通过getMediaProjection来取出intent中的数据：

```Java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode != PERMISSION_CODE) {
        Log.e(TAG, "Unknown request code: " + requestCode);
        return;
    }
    if (resultCode != RESULT_OK) {
        Toast.makeText(this,
                "User denied screen sharing permission", Toast.LENGTH_SHORT).show();
        return;
    }
    mMediaProjection = mProjectionManager.getMediaProjection(resultCode, data);
    mMediaProjection.registerCallback(new MediaProjectionCallback(), null);
    mVirtualDisplay = createVirtualDisplay();
}
```

我们通过getMediaProjection获取到mediaProjection，并注册了一个callback回调。
看看createVirtualDisplay做了什么：

```java
private VirtualDisplay createVirtualDisplay() {
        return mMediaProjection.createVirtualDisplay("ScreenSharingDemo",
                mDisplayWidth, mDisplayHeight, mScreenDensity,
                DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
                mSurface, null /*Callbacks*/, null /*Handler*/);
    }
```

可以看到，我们调用了MediaProjection的createVirtualDisplay方法，来创建了一个VirtualDisplay的实例，说几个createVirtualDisplay的参数含义：

>name 是生成的VirtualDisplay实例的名称；
width和height分别是生成实例的宽高，必须大于0；
dpi，生成实例的像素密度，必须大于0，一般都取1；
surface，这个比较重要，是你生成的VirtualDisplay的载体，我的理解是，VirtualDisplay的内容是一帧帧的屏幕截图（所以你看到是有宽高，像素密度等设置），所以MediaProjection获取到的其实是一帧帧的图，然后通过surface（surface你可以理解成是android的一个画布，默认它会以每秒60帧来刷新，这里我们不再展开细说），来顺序播放这些图片，形成视频。

surface我们可以这样获取到：

```java
SurfaceView mSurfaceView = (SurfaceView) findViewById(R.id.surface);
Surface mSurface = mSurfaceView.getHolder().getSurface();
```

对应的我们要在XML里面写一个SurfaceView的控件：

```xml
<SurfaceView
        android:id="@+id/surface"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerHorizontal="true"
        android:layout_alignParentTop="true"/>

```

这样，屏幕所捕获的内容，就显示在这个SurfaceView上面了

原文地址： [Android视频录制－－MediaProjection](http://blog.csdn.net/l00149133/article/details/48346107)
