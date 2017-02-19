---
title: 为Android程序申请权限注意
date: 2017-02-19 17:35:46
tags:
categories: "Android学习笔记"
---

Android系统提供为程序提供了权限申请,即在manifest中使用uses-permission来申请即可.实现起来非常简单,但是有些问题会随之浮出水面. 常见的现象是,有时候新加一个权限,(在Google Play上)程序显示的支持的设备会减少.

#### 为什么权限越多,支持设备越少

因为有些权限隐式地需要feature,即当你显示使用uses-permission,会默认地为程序加入uses-feature.
而Android以及Google Play判断是否可以安装的依据是,设备包含的system features是否完全包含程序申请的全部features. **只有在全部满足了程序需要的feature的设备上才可以展示并安装**.

### 举个例子

我们在程序manifest加入一行申请摄像头的权限.

```java
<uses-permission android:name="android.permission.CAMERA" />
```

然后查看程序加入的feature

```sh
$ aapt dump badging PermissionDemo.apk | grep uses-feature
```

我们就会发现,这两个权限是新加的

```sh
uses-feature:'android.hardware.camera'
uses-feature:'android.hardware.camera.autofocus'
```

<!--more-->


### 解决问题:如何加权限,不减少支持设备

如果你增加的权限并且及引入的feature不是必须使用的,可以显示地将该feature设置为不需要.继续上面的例子.在manifest中加入

```java
<uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>
<uses-feature android:name="android.hardware.camera" android:required="false"/>
```

重新生成程序.再次查看需要的权限.

```sh
$ aapt dump badging PermissionDemo.apk | grep uses-feature
uses-feature-not-required:'android.hardware.camera.autofocus'
uses-feature-not-required:'android.hardware.camera'
uses-feature:'android.hardware.touchscreen'
```
就这样,可以做到增加权限,同时保证支持设备不减少.

### 延伸阅读

[http://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions)

Origin post : [为Android程序申请权限注意](http://droidyue.com/blog/2014/07/04/attentions-when-adding-permission-for-android-application/)
