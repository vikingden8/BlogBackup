---
title: Background Execution Limits
date: 2017-4-27 09:54:42
tags:
categories: "Android O Features and APIs"
---

Whenever an app runs in the background, it consumes some of the device's limited resources, like RAM. This can result in an impaired user experience, especially if the user is using a resource-intensive app, such as playing a game or watching video. To improve the user experience, Android O imposes limitations on what apps can do while running in the background. This document describes the changes to the operating system, and how you can update your app to work well under the new limitations.

每当一个应用在后台运行，它将会消耗设备的一部分资源，比如RAM。这将导致影响用户体验，尤其是当用户正在使用一个资源紧张的应用时，比如正在玩游戏或者观看视频。为了提高用户体验，Android O对后台运行的应用施加了限制。这个文档将描述操作系统中的改变，以及如何更新应用以便在新限制下正常运行。

### Overview 概览

Many Android apps and services can be running simultaneously. For example, a user could be playing a game in one window while browsing the web in another window, and using a third app to play music. The more apps are running at once, the more load is placed on the system. If additional apps or services are running in the background, this places additional loads on the system, which could result in a poor user experience; for example, the music app might be suddenly shut down.

多个Android应用和服务可以同时运行。比如，一用户可以在一个窗口玩游戏的同时，在另一个窗口中打开浏览器浏览网页，并且还可以使用第三个应用来播放音乐。如果同时运行更多的应用，对系统造成的负担就越大。如果额外的应用或者服务还运行在后台，这会对系统造成更大的负担，这将会导致用户体验很差；比如，音乐的应用可能会突然关闭。

To lower the chance of these problems, Android O places limitations on what apps can do while users aren't directly interacting with them. If an app targets Android O, it is restricted in two ways:

为了降低上述问题发生的几率，Android O对应用在用户不与其直接交互时可以执行的操作施加了限制。如果App应用在Android O上，将会通过下述两种方式进行约束：

  * **Background Service Limitations**: While an app is idle, there are limits to its use of background services. This does not apply to foreground services, which are more noticeable to the user.

  * **后台服务限制**: 当应用处于空闲状态时，应用可以使用的后台服务存在限制。这些限制不适用于前台服务，因为前台服务更容易引起用户注意。

  * **Broadcast Limitations: With limited exceptions**, apps cannot use their manifest to register for implicit broadcasts. They can still register for these broadcasts at runtime, and they can use the manifest to register for explicit broadcasts targeted specifically at their app.

  * **广播限制**：应用无法使用清单注册隐式广播。它们仍然可以在运行时注册这些广播，并且可以使用清单注册专门针对它们的显式广播。

In most cases, apps can work around these limitations by using _JobScheduler_ jobs. This approach lets an app arrange to perform work when the app isn't actively running, but still gives the system the leeway to schedule these jobs in a way that doesn't affect the user experience.

在大多数情况下，应用都可以使用JobScheduler作业克服这些限制。这种方式让应用安排为在未活跃运行时执行工作，不过仍能够使系统可以在不影响用户体验的情况下安排这些作业。

<!--more-->

### Background Service Limitations 后台服务限制

Services running in the background can consume device resources, potentially resulting in a worse user experience. To mitigate this problem, the system applies a number of limitations on services to apps that target Android O.

服务在后台运行消耗设备的资源，导致潜在的降低用户体验。为了缓解这一问题，系统针对Android O的应用中的服务实施了一系列限制。

>Note: These limitations apply only to apps that target Android O. Apps that target API level 25 or lower are not affected.

>注意：这些限制只应用于针对Android O的应用。应用的API级别在25或者以下的将不受影响。

The system distinguishes between foreground and background apps. (The definition of background for purposes of service limitations is distinct from the definition used by memory management; an app might be in the background as pertains to memory management, but in the foreground as pertains to its ability to launch services.) An app is considered to be in the foreground if any of the following is true:

系统区分前台和后台应用。（用于服务限制目的的后台定义与内存管理使用的定义不同；一个应用按照内存管理的定义可能处于后台，但按照能够启动服务的定义又处于前台。）如果以下的条件有一个为真，则该应用被认定为前台应用：

  * It has a visible activity, whether the activity is started or paused.

  * 具有可见 Activity（不管该 Activity 已启动还是已暂停）。

  * It has a foreground service.

  * 具有前台服务

  * Another foreground app is connected to the app, either by binding to one of its services or by making use of one of its content providers. For example, the app is in the foreground if another app binds to its:

  * 另外一个前台应用已关联到该应用，可以使通过服务绑定，或者通过使用其中一个内容提供程序。比如，如果另一个应用绑定到该应用的服务，那么该应用处于前台：

    1. IME

    2. Wallpaper service

    3. Notification listener

    4. Voice or text service

If none of those conditions is true, the app is considered to be in the background.

如果上面的条件没有一个为真，则该应用被认定为后台应用。

While an app is in the foreground, it can create and run both foreground and background services freely. When an app goes into the background, it has a window of several minutes in which it is still allowed to create and use services. At the end of that window, the app is considered to be idle. At this time, the system stops the app's background services, just as if the app had called the services' Service.stopSelf() methods.

当应用运行在前台，它可以自由地创建和运行前台和后台服务。当应用进入到后台时，在一个持续数分钟的时间窗内，应用仍可以创建和使用服务。在该时间窗结束后，应用将被视为处于空闲状态。此时，系统将停止应用的后台服务，就像应用已经调用服务的 _Service.stopSelf()_ 方法一样。

Under certain circumstances, a background app is placed on a temporary whitelist for several minutes. While an app is on the whitelist, it can launch services without limitation, and its background services are permitted to run. An app is placed on the whitelist when it handles a task that's visible to the user, such as:

在这些情况下，后台应用将被置于一个临时白名单中并持续数分钟。位于白名单中时，应用可以无限制地启动服务，并且其后台服务也可以运行。处理对用户可见的任务时，应用将被置于白名单中，例如：

  * Handling a high-priority Firebase Cloud Messaging (FCM) message.

  * Receiving a broadcast, such as an SMS/MMS message.

  * Executing a PendingIntent from a notification.

In many cases, your app can replace background services with _JobScheduler_ jobs. For example, _CoolPhotoApp_ needs to check whether the user has received shared photos from friends, even if the app isn't running in the foreground. Previously, the app used a background service which checked with the app's cloud storage. To migrate to Android O, the developer replaces the background service with a scheduled job, which is launched periodically, queries the server, then quits.

在很多情况下，您的应用都可以使用 _JobScheduler_ 替换后台服务。 例如，_CoolPhotoApp_ 需要检查用户是否已经从朋友那里收到共享的照片，即使该应用未在前台运行。 之前，应用使用一种会检查其云存储的后台服务。 为了迁移到 Android O，开发者使用一个计划作业替换了这种后台服务，该作业将按一定周期启动，查询服务器，然后退出。

Prior to Android O, the usual way to create a foreground service was to create a background service, then promote that service to the foreground. With Android O, this approach fails if the app is not currently able to create background services. For this reason, Android O introduces the new method _NotificationManager.startServiceInForeground()_. Calling this method is the equivalent of calling _startService()_ to create a service in the background, then immediately calling the service's _startForeground()_ method to promote it to the foreground. However, since the new service is never in the background, it is not subject to the limitations on background services.

在Android O之前，创建前台服务的方式通常是先创建一个后台服务，然后将该服务推到前台。 在 Android O中，如果应用当前无法创建后台服务，这种方式将失败。因此，Android O 引入了一种全新的方法，即 _NotificationManager.startServiceInForeground()_。 调用此方法等同于调用 _startService()_ 在后台中创建一个服务，然后立即调用该服务的 _startForeground()_ 方法将其推到前台。 不过，由于新服务在任何情况下都不会处于后台，它不适用后台服务的限制。

### Broadcast Limitations 广播限制

If an app registers to receive broadcasts, the app's receiver consumes resources every time the broadcast is sent. This can cause problems if too many apps register to receive broadcasts based on system events; a system event that triggers a broadcast can cause all of those apps to consume resources in rapid succession, impairing the user experience. To mitigate this problem, Android 7.0 (API level 25) placed limitations on broadcasts, as described in Background Optimization. The Android O makes these limitations more stringent.

如果应用注册为接收广播，则在每次发送广播时，应用的接收器都会消耗资源。如果多个应用注册为接收基于系统事件的广播，这会引发问题；触发广播的系统事件会导致所有应用快速地连续消耗资源，从而降低用户体验。为了缓解这一问题，Android 7.0（API级别25）对广播施加了一些限制，如后台优化中所述。Android O 让这些限制更为严格。

>Note: These limitations apply only to apps that target Android O. Apps that target API level 25 or lower are not affected.

>注：这些限制仅适用于针对 Android O 的应用。 针对 API 级别 25 或更低级别的应用不受影响。

  * Apps that target Android O can no longer register broadcast receivers for implicit broadcasts in their manifest. An implicit broadcast is a broadcast that does not target that app specifically. For example, _ACTION_PACKAGE_REPLACED_ is an implicit broadcast, since it is sent to all registered listeners, letting them know that some package on the device was replaced. However, _ACTION_MY_PACKAGE_REPLACED_ is not an implicit broadcast, since it is sent only to the app whose package was replaced, no matter how many other apps have registered listeners for that broadcast.

  * 针对 Android O 的应用无法继续在其清单中为隐式广播注册广播接收器。 隐式广播是一种不专门针对该应用的广播。 例如，_ACTION_PACKAGE_REPLACED_ 就是一种隐式广播，因为它将发送到注册的所有侦听器，让后者知道设备上的某些软件包已被替换。 不过，_ACTION_MY_PACKAGE_REPLACED_ 不是隐式广播，因为不管已为该广播注册侦听器的其他应用有多少，它都会只发送到软件包已被替换的应用。

  * Apps can continue to register for explicit broadcasts in their manifests.

  * 应用可以继续在它们的清单中注册显式广播。

  * Apps can use _Context.registerReceiver()_ at runtime to register a receiver for any broadcast, whether implicit or explicit.

  * 应用可以在运行时使用 _Context.registerReceiver()_ 为任意广播（不管是隐式还是显式）注册接收器。


In many cases, apps that previously registered for an implicit broadcast can get similar functionality by using a _JobScheduler_ job. For example, a social photo app might need to perform cleanup on its data from time to time, and prefer to do this when the device is connected to a charger. Previously, the app registered a receiver for _ACTION_POWER_CONNECTED_ in its manifest; when the app received that broadcast, it would check whether cleanup was necessary. To migrate to Android O, the app removes that receiver from its manifest. Instead, the app schedules a cleanup job that runs when the device is idle and charging.

在许多情况下，之前注册隐式广播的应用使用 _JobScheduler_ 作业可以获得类似的功能。例如，一款社交照片应用可能需要不时地执行数据清理，并且倾向于在设备连接到充电器时执行此操作。之前，应用已经在清单中为 _ACTION_POWER_CONNECTED_ 注册了一个接收器；当应用接收到该广播时，它会检查清理是否必要。为了迁移到Android O，应用将该接收器从其清单中移除。应用将清理作业安排在设备处于空闲状态和充电时运行。

>Note: A number of implicit broadcasts are currently exempted from this limitation. Apps can continue to register receivers for these broadcasts in their manifests, no matter what API level the apps are targeting. For a list of the exempted broadcasts, see [Implicit Broadcast Exceptions](https://developer.android.com/preview/features/background-broadcasts.html).

>注：很多隐式广播当前均已不受此限制所限。 应用可以继续在其清单中为这些广播注册接收器，不管应用针对哪个 API 级别。 有关已豁免广播的列表，请参阅[]隐式广播例外](https://developer.android.com/preview/features/background-broadcasts.html)。

### Migration Guide 迁移指南

These changes do not affect apps that target API level 25 or below. If your app targets Android O, you may need to update your app to comply with the new limitations.

这些变化不会影响针对 API 级别 25 或更低级别的应用。 如果您的应用针对 Android O，那么您可能需要更新应用，使其符合新限制。

Check to see how your app uses services. If your app relies on services that run in the background while your app is idle, you will need to replace them. Possible solutions include:

了解您的应用如何使用服务。 如果您的应用依赖某些在它处于空闲时于后台运行的服务，您需要替换这些服务。 可能的解决方法包括：

  * If your app needs to create a foreground service while the app is in the background, use the new _NotificationManager.startServiceInForeground()_ method instead of creating a background service and trying to promote it to the foreground.

  * 如果处于后台时您的应用需要创建一个前台服务，请使用新的 _NotificationManager.startServiceInForeground()_ 方法，而不是创建一个后台服务，然后尝试将其推到前台。

  * If the service is noticeable by the user, make it a foreground service. For example, a service that plays audio should always be a foreground service. Create the service with _NotificationManager.startServiceInForeground()_ instead of _startService()_.

  * 如果服务容易被用户注意，请将其设为前台服务。 例如，播放音频的服务始终应为前台服务。 使用 _NotificationManager.startServiceInForeground()_ 而不是 _startService()_ 创建服务。

  * Find a way to duplicate the service's functionality with a scheduled job. If the service is not doing something immediately noticeable to the user, you should generally be able to use a scheduled job instead.

  * 寻找一种使用计划作业实现服务功能的方式。 如果服务未在执行容易立即被用户注意到的操作，一般情况下，您都能够使用计划作业。

  * Use _FCM_ to selectively wake your application up when network events occur, rather than polling in the background.

  * 发生网络事件时，请使用 _FCM_ 选择性地唤醒您的应用，而不是在后台轮询。

  * Defer background work until the application is naturally in the foreground.

  * 在应用正常处于前台之前，请推迟后台工作。

Review the broadcast receivers defined in your app's manifest. If your manifest declares a receiver for an implicit broadcast, you must replace it. Possible solutions include:

检查在您应用的清单中定义的广播接收器。 如果您的清单为显式广播声明了接收器，您必须予以替换。 可能的解决方法包括：

  * Create the receiver at runtime by calling _Context.registerReceiver()_, instead of declaring the receiver in the manifest.

  * 通过调用 _Context.registerReceiver()_ 而不是在清单中声明接收器的方式在运行时创建接收器。

  * Use a scheduled job to check for the condition that would have triggered the implicit broadcast.

  * 使用计划作业检查条件是否会触发隐式广播。
