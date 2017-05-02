---
title: Background Location Limits
date: 2017-5-2 09:02:46
tags:
categories: "Android O Features and APIs"
---

In an effort to reduce power consumption, Android O Developer Preview limits how frequently background apps can retrieve the user's current location, regardless of your app's target SDK version.

为了降低功耗，无论你的应用目标sdk版本是多少，Android O都会限制后台应用获取当前用户位置信息的频率。

This location retrieval behavior is particularly important to keep in mind if your app relies on real-time alerts or motion detection while running in the background.

如果你的应用在后台运行时依赖于实时提醒或运动检测，上述的位置获取行为是非常重要的，需切记在心。

>Important: As a starting point, we're allowing background apps to receive location updates only a few times each hour. We're continuing to tune the location update interval throughout the Preview based on system impact and feedback from developers.

>作为起点，我们只允许后台应用每小时接收几次位置更新。我们将在整个预览版阶段继续根据系统影响和开发者的反馈优化位置更新间隔。
<!--more-->

The system distinguishes between foreground and background apps. An app is considered to be in the foreground if any of the following is true:

系统区分前台和后台应用，如果一个应用满足以下的一个条件，则被认定为前台应用：

  * It has a visible activity, whether the activity is started or paused.

  * 它具有可见的Activity，不管这个Activity是启动还是暂停状态。

  * It has a foreground service.

  * 它有一个前台服务。

  * Another foreground app is connected to the app, either by binding to one of its services or by making use of one of its content providers.

  * 另一个前台应用正在连接这个应用，不管是通过服务绑定的方式还是通过使用它的内容提供者方式。

If none of those conditions is true, the app is considered to be in the background.

如果上述的条件没有一个满足，那么这个应用则被认定为后台应用

### Foreground app behavior is preserved

If an app is in the foreground on a device running Android O, the location update behavior is the same as on Android 7.1.1 (API level 25) and lower.

如果一个应用是前台应用，且运行在Android O系统上，其位置更新行为跟之前的Android 7.1.1以及之前的版本保持一致。

>Warning: If your app retrieves near real-time location updates over a long period of time, the device's battery life becomes significantly shorter.

>如果你的应用长时间获取实时的位置更新，设备的电池容量将会明显减少

### Tuning your app's location behavior

Consider whether your app's use cases for running in the background cannot succeed at all if your app receives infrequent location updates. If this is the case, you can retrieve location updates more frequently by performing one of the following actions:

考虑在你的应用接收位置更新不频繁的情况下其后台运行用例是否根本无法成功。如果属于这种情况，您可以通过执行下列操作之一提高位置更新的检索频率：

  * Bring your app to the foreground.

  * 把你的应用转为前台应用。

  * Use a foreground service in your app. When this service is active, your app must show an ongoing notification in the notification area.

  * 在你的应用中使用前台服务。当这个服务被启动激活时，你的应用必须在通知栏区域展现正在获取位置信息的消息。

  * Use elements of the Geofencing API, such as the GeofencingApi interface, which are optimized for minimizing power use.

  * 使用Geofencing API，比如使用已优化最少电池容量使用的GeofencingApi接口。

  * Use a passive location listener, which may receive faster location updates if there are foreground apps requesting location at a faster rate.

  * 使用被动位置侦听器，它可以在后台应用加快位置请求频率时提高位置更新的接收频率。

>Note: If your app needs access to location history that contains time-frequent updates, use the batched version of the Fused Location Provider API elements, such as the FusedLocationProviderApi interface. When your app is running in the background, this API receives the user's location more frequently than the non-batched API. Keep in mind, however, that your app still receives updates in batches only a few times each hour.

>如果你的应用需要访问的位置历史记录包含时间频繁更新，请使用批处理版本的 Fused Location Provider API 元素，例如 FusedLocationProviderApi 接口。当您的应用运行于后台时，此 API 会以高于非批处理版本 API 的频率接收用户的位置。但切记，您的应用批量接收更新的频率仍仅为每小时几次。

### Affected APIs

The changes to location retrieval behavior in background apps affect the following APIs:

对后台应用位置检索行为的更改影响下列API：

* Fused Location Provider (FLP)

  If your app is running in the background, the location system service computes a new location for your app only a few times each hour, according to the interval defined in the Android O behavior change. This is the case even when your app is requesting more frequent location updates.

  如果您的应用运行在后台，位置系统服务只会根据 Android O 行为变更中定义的间隔，按每小时几次的频率为其计算新位置。即使您的应用请求进行更频繁的位置更新，也仍是如此。

  If your app is running in the foreground, there is no change in location sampling rates compared to Android 7.1.1 (API level 25).

  如果您的应用运行在前台，与 Android 7.1.1（API 级别 25）相比，在位置采样率上不会有任何变化。

* Geofencing

  Background apps can receive geofencing transition events more frequently than updates from the Fused Location Provider.

  后台应用可以高于接收 Fused Location Provider 更新的频率接收地理围栏转换事件。

* GNSS Measurements and GNSS Navigation Message

  When your app is in the background, callbacks that are registered to receive outputs from GnssMeasurement and GnssNavigationMessage stop executing.

  当您的应用位于后台时，注册用于接收 GnssMeasurement 和 GnssNavigationMessage 输出的回调会停止执行。

* Location Manager

  Location updates are provided to background apps only a few times each hour, according to the interval defined in the Android O behavior change.

  提供给后台应用的位置更新只会根据 Android O 行为变更中定义的间隔，按每小时几次的频率提供。

  >Note: If your app is running on a device with Google Play services installed, it is highly recommended that you use the Fused Location Provider (FLP) instead.

  >注：如果运行您的应用的设备安装了 Google Play 服务，强烈建议您改用 Fused Location Provider (FLP)。
