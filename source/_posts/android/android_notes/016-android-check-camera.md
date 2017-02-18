---
title: 检查Android是否具有摄像头
date: 2017-02-18 10:44:16
tags:
categories: "Android学习笔记"
---

通常我们进行摄像头操作，如扫描二维码需要判断是否有后置摄像头(Rear camera)，比如Nexus 7 一代就没有后置摄像头，这样在尝试使用的时候，我们需要进行判断进行一些提示或者处理。

以下代码为一系列的方法，用来判断是否有前置摄像头（Front Camera），后置摄像头。

```java
private static boolean checkCameraFacing(final int facing) {
    if (getSdkVersion() < Build.VERSION_CODES.GINGERBREAD) {
        return false;
    }
    final int cameraCount = Camera.getNumberOfCameras();
    CameraInfo info = new CameraInfo();
    for (int i = 0; i < cameraCount; i++) {
        Camera.getCameraInfo(i, info);
        if (facing == info.facing) {
            return true;
        }
    }
    return false;
}

public static boolean hasCamera(){
  return hasBackFacingCamera() || hasFrontFacingCamera() ;
}

public static boolean hasBackFacingCamera() {
    final int CAMERA_FACING_BACK = 0;
    return checkCameraFacing(CAMERA_FACING_BACK);
}

public static boolean hasFrontFacingCamera() {
    final int CAMERA_FACING_FRONT = 1;
    return checkCameraFacing(CAMERA_FACING_FRONT);
}

public static int getSdkVersion() {
    return android.os.Build.VERSION.SDK_INT;
}

```

注意：由于getNumberOfCameras以及getCameraInfo均为API 9 引入，所以方法只适用于2.3及其以上。

注意：在API-21以上,Camera类已经不被建议使用，建议使用新的Camera2 API。
