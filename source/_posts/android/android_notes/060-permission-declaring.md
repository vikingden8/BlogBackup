---
title: Declaring Permissions
date: 2017-07-23 23:58:33
tags:
categories: "Android学习笔记"
---

**Every Android app runs in a limited-access sandbox. If an app needs to use resources or information outside of its own sandbox, the app has to request the appropriate permission. You declare that your app needs a permission by listing the permission in the App Manifest**.

Depending on how sensitive the permission is, the system might grant the permission automatically, or the device user might have to grant the request. For example, if your app requests permission to turn on the device's flashlight, the system grants that permission automatically. But if your app needs to read the user's contacts, the system asks the user to approve that permission. Depending on the platform version, the user grants the permission either when they install the app (on Android 5.1 and lower) or while running the app (on Android 6.0 and higher).

<!--more-->

### Determine What Permissions Your App Needs

As you develop your app, you should note when your app is using capabilities that require a permission. Typically, an app is going to need permissions whenever it uses information or resources that the app doesn't create, or performs actions that affect the behavior of the device or other apps. For example, if an app needs to access the internet, use the device camera, or turn Wi-Fi on or off, the app needs the appropriate permission. For a list of system permissions, see [Normal and Dangerous Permissions](https://developer.android.com/guide/topics/permissions/requesting.html#normal-dangerous).

Your app only needs permissions for actions that it performs directly. Your app does not need permission if it is requesting that another app perform the task or provide the information. For example, if your app needs to read the user's address book, the app needs the READ_CONTACTS permission. But if your app uses an intent to request information from the user's Contacts app, your app does not need any permissions, but the Contacts app does need to have that permission. For more information, see [Consider Using an Intent](https://developer.android.com/training/permissions/usage-notes.html#perms-vs-intents).

### Add Permissions to the Manifest

To declare that your app needs a permission, put a _<uses-permission>_ element in your app manifest, as a child of the top-level _<manifest>_ element. For example, an app that needs to send SMS messages would have this line in the manifest:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.example.snazzyapp">

    <uses-permission android:name="android.permission.SEND_SMS"/>


    <application ...>
        ...
    </application>

</manifest>
```

The system's behavior after you declare a permission depends on how sensitive the permission is. If the permission does not affect user privacy, the system grants the permission automatically. If the permission might grant access to sensitive user information, the system asks the user to approve the request.


From:[Declaring Permissions](https://developer.android.com/training/permissions/declaring.html)
