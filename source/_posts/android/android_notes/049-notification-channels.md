---
title: Notification Channels
date: 2017-5-3 09:29:31
tags:
categories: "Android O Features and APIs"
---

Android O introduces notification channels to provide a unified system to help users manage notifications. When you target Android O, you must implement one or more notification channels to display notifications to your users. If you don’t target Android O, your apps behave the same as they do on Android 7.0 when running on Android O devices.

Android O引入了消息通道来帮助用户提供统一的系统级消息管理。当你的应用目标是Android O时，你必须实现一个或者读个消息通道来给你的用户显示消息。如果你的应用不是针对Android O，那你的应用行为跟之前的Android 7.0保持一致。

You can create a notification channel for each distinct type of notification you need to send. You can also create notification channels to reflect choices made by users of your app. For example, you might setup separate notification channels for each conversation group created by a user in a messaging app.

你可以为需要发送的每个不同的通知类型创建一个通知通道。还可以创建通知通道来反映应用的用户做出的选择。例如，你可以为一款消息传递应用的用户创建的每个对话组建立单独的通知通道。

Users can now manage most of the settings associated with notifications using a consistent system UI. All notifications posted to a notification channel behave the same. When a user modifies the behavior for any of the following characteristics, it applies to the notification channel:

用户现在可以使用一致的系统UI管理大多数与通知有关的设置。所有发布至通知通道的通知都具有相同的行为。当用户修改任何下列特性的行为时，修改将作用于通知通道：

<!--more-->

  * Importance

  * Sound

  * Lights

  * Vibration

  * Show on lockscreen

  * Override do not disturb

Users can visit Settings, or long press a notification to change these behaviors, or even block a notification channel at any time. You can’t programmatically modify the behavior of a notification channel once it’s created and submitted to the notification manager; the user is in charge of those settings.

用户可以通过设置，或长按消息来改变那些行为，或者甚至可以随时屏蔽一个消息通道。一旦你的消息通道已创建，并且提交到了消息管理器，你不能通过在程序中改变它了，其完全由用户来管理设置。

### Notification Priority and Importance

Android O deprecates the ability to set the priority levels of individual notifications. Instead, you can now set a recommended importance level when creating a notification channel. The importance level you assign to a notification channel applies to all notification messages that you post to it. You can configure a channel with one of five importance levels that configure the amount a channel can interrupt a user, ranging from IMPORTANCE_NONE(0) to IMPORTANCE_HIGH(4). The default importance level is 3 which displays everywhere, makes noise, but doesn’t visually intrude on the user. Once you create a notification channel, only the system can modify its importance.

Android O弃用了为单个通知设置优先级的功能。现在，创建通知通道时你可以设置建议重要性级别。你为通知通道指定的重要性级别适用于你发布至该通道的所有通知消息。你可以为通道配置五个重要性级别中的一个，这些级别配置的是通道可以打断用户的程度，范围是 IMPORTANCE_NONE(0) 至 IMPORTANCE_HIGH(4)。默认重要性级别为3：在所有位置显示，发出提示音，但不对用户进行视觉干扰。创建通知通道后，只有系统可以修改其重要性。

### Creating a Notification Channel

To create a notification channel:

要创建通知通道，请执行下列操作：

  * Construct a notification channel object with an ID that’s unique within your package.

  * 构建一个在软件包内具有唯一 ID 的通知通道对象。

  * Configure the notification channel object with any desired initial settings such as an alert sound.

  * 为该通知通道对象配置所需的任何初始设置（例如提示音）。

  * Submit the notification channel object to the notification manager.

  * 将通知通道对象提交到通知管理器。

Attempting to create a notification channel when it already exists performs no operation, so it’s safe to perform the above sequence of steps when starting an app. The following code sample demonstrates creating a notification with a low importance level, and a custom vibration pattern.

如果试图创建的通知通道已存在，不会执行任何操作，因此启动应用时可以放心地执行以上步骤序列。以下示例代码演示的是如何创建具有低重要性级别和自定义振动模式的通知。

```java
NotificationManager mNotificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// The id of the channel.
String id = "my_channel_01";
// The user visible name of the channel.
CharSequence name = getString(R.string.channel_name);
int importance = NotificationManager.IMPORTANCE_LOW;
NotificationChannel mChannel = new NotificationChannel(id, name, importance);
// Configure the notification channel.
mChannel.enableLights(true);
// Sets the notification light color for notifications posted to this
// channel, if the device supports this feature.
mChannel.setLightColor(Color.RED);
mChannel.enableVibration(true);
mChannel.setVibrationPattern(new long[]{100, 200, 300, 400, 500, 400, 300, 200, 400});
mNotificationManager.createNotificationChannel(mChannel);
```

You can also create multiple notification channels in a single operation by calling _NotificationManager.createNotificationChannels()_

你还可以通过调用 _NotificationManager.createNotificationChannels()_ 一次性创建多个通知通道。

### Creating a Notification Channel Group

If your app supports multiple user accounts, you can create a notification channel group for each account. Notification channel groups allow you to manage multiple notification channels with identical names within a single app. For example, a social networking app might include support for personal, as well as business user accounts. In this scenario, each user account might require multiple notification channels with identical functions and names.

如果你的应用支持多个用户帐户，则可为每个帐户创建一个通知通道组。通知通道组用于对一款应用内的多个同名通知通道进行管理。例如，一款社交网络应用可能提供面向个人用户帐户以及企业用户帐户的支持。在此情境下，每个用户帐户可能都需要多个功能和名称相同的通知通道。

  * A personal user account including 2 notification channels:

    1. Notifications of new comments on your posts.

    2. Notifications recommending posts by your contacts.

  * A business user account including 2 notification channels:

    1. Notifications of new comments on your posts.

    2. Notifications recommending posts by your contacts.

Organizing the notification channels associated with each user account in this example into dedicated groups ensures that users can easily distinguish between them in Settings. Each notification channel group requires an ID that must be unique within your package, as well as a user visible name. The following snippet demonstrates how to create a notification channel group.

在本例中，将与每个用户帐户相关的通知通道组织成专用组可确保用户能在 Settings 中轻松地进行区分。每个通知通道组都必须在软件包内具有唯一 ID，并具有用户可见的名称。下面这段代码演示了如何创建通知通道组。

```java
// The id of the group.
String group = "my_group_01";
// The user visible name of the group.
CharSequence name = getString(R.string.group_name);;
NotificationManager mNotificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mNotificationManager.createNotificationChannelGroup(new NotificationChannelGroup(group, name));
```

Once you’ve created a new group you can call _NotificationChannel.setGroup()_ to associate a new channel with the group. You can only modify the association between notification channel and group before you submit the channel to the notification manager.

新建通道组后，便可调用 _NotificationChannel.setGroup()_ 将新通道与组关联起来。你只能在将通道提交给通知管理器之前修改通知通道与组之间的关联。

### Creating a Notification

To create a notification, you call _Notification.Builder.build()_, which returns a Notification object containing your specifications. To issue the notification, you pass the _NotificationManager_ object to the system by calling _NotificationManager.notify()_.

要创建通知，请调用 _Notification.Builder.build()_，它返回的 _Notification_ 对象包含你的指定值。要发出通知，请通过调用 _NotificationManager.notify()_ 将 _Notification_ 对象传递给系统。

### Required notification contents

A Notification object must contain the following:

Notification 对象必须包含以下内容：

  * A small icon, set by setSmallIcon()

  * 小图标，由 setSmallIcon() 设置

  * A title, set by setContentTitle()

  * 标题，由 setContentTitle() 设置

  * Detail text, set by setContentText()

  * 详细文本，由 setContentText() 设置

  * A valid notification channel ID, set by setChannel()

  * 有效的通知通道 ID，由 setChannel() 设置

### Posting a notification to a channel

The following snippet illustrates posting a simple notification to a notification channel. Notice that the code associates the notification with a notification channel using the channel’s ID.

下面这段代码说明如何向通知通道发布简单通知。请注意，代码利用通道的 ID 将通知与通知通道关联起来。

```java
mNotificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// Sets an ID for the notification, so it can be updated.
int notifyID = 1;
// The id of the channel.
String CHANNEL_ID = "my_channel_01";
// Create a notification and set the notification channel.
Notification notification = new Notification.Builder(MainActivity.this)
        .setContentTitle("New Message")
        .setContentText("You've received new messages.")
        .setSmallIcon(R.drawable.ic_notify_status)
        .setChannel(CHANNEL_ID)
        .build();
// Issue the notification.
mNotificationManager.notify(id, notification);
```

### Reading Notification Channel Settings

Users can modify the settings for notification channels, including behaviors such as vibration and alert sound. You can call the following two methods to discover the settings a user has applied to a notification channel:

用户可以修改通知通道的设置，包括振动和提示音等行为。你可以调用以下两个方法来发现用户对通知通道应用的设置：

  * To retrieve a single notification channel, you can call _NotificationManager.getNotificationChannel()_.

  * 要检索单个通知通道，你可以调用 _NotificationManager.getNotificationChannel()_。

  * To retrieve all notification channels belonging to your app, you can call _NotificationManager.getNotificationChannels()_.

  * 要检索归属你的应用的所有通知通道，你可以调用 _NotificationManager.getNotificationChannels()_。

### Updating Notification Channel Settings

Once you create a notification channel, the user is in charge of its settings and behavior. The following sample code describes how you can redirect a user to the settings for a notification channel by creating an intent to start an activity. In this case, the intent requires extended data including the ID of the notification channel and the package name of your app.

一旦你创建了通知通道，其设置和行为就由用户掌控。以下示例代码说明如何通过创建启动 Activity 的 Intent 将用户重定向到通知通道的设置。在本例中，Intent 要求提供扩展数据，包括通知通道的 ID 和应用的软件包名称。

```java
Intent intent = new Intent(Settings.ACTION_CHANNEL_NOTIFICATION_SETTINGS);
intent.putExtra(Settings.EXTRA_CHANNEL_ID, mChannel.getId());
intent.putExtra(Settings.EXTRA_APP_PACKAGE, getPackageName());
startActivity(intent);
```

### Deleting a Notification Channel

You can delete notification channels by calling _NotificationManager.deleteNotificationChannel()_. Deleted channels remain visible in notification settings, as a spam prevention mechanism. You can clear test channels on development devices: either by reinstalling the app, or by clearing the data associated with your copy of the app. The following sample code demonstrates how to delete a notification channel.

你可以通过调用 _NotificationManager.deleteNotificationChannel()_ 来删除通知通道。作为一个垃圾信息预防机制，删除的通道在通知设置中仍然可见。你可以通过以下任一方法清除开发设备上的测试通道：重新安装应用；清除与你的应用副本关联的数据。以下示例代码演示了如何删除通知通道。

```java
NotificationManager mNotificationManager =
        (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
// The id of the channel.
String id = "my_channel_01";
NotificationChannel mChannel = mNotificationManager.getNotificationChannel(id);
mNotificationManager.deleteNotificationChannel(mChannel);
```
