---
title: Building better apps with Runtime Permissions
date: 2017-07-24 00:23:29
tags:
categories: "Android学习笔记"
---

Posted by Ian Lake, Developer Advocate

Android devices do a lot, whether it is taking pictures, getting directions or making phone calls. With all of this functionality comes a large amount of very sensitive user data including contacts, calendar appointments, current location, and more. This sensitive information is protected by permissions, which each app must have before being able to access the data. Android 6.0 Marshmallow introduces one of the largest changes to the permissions model with the addition of runtime permissions, a new permission model that replaces the existing install time permissions model when you target API 23 and the app is running on an Android 6.0+ device.

Runtime permissions give your app the ability to control when and with what context you’ll ask for permissions. This means that users installing your app from Google Play will not be required to accept a list of permissions before installing your app, making it easy for users to get directly into your app. It also means that if your app adds new permissions, app updates will not be blocked until the user accepts the new permissions. Instead, your app can ask for the newly added runtime permissions as needed.

Finding the right time to ask for runtime permissions has an important impact on your app’s user experience. We’ve gathered a number of design patterns in our new [Permission design guidelines](https://www.google.com/design/spec/patterns/permissions.html?utm_campaign=runtime-permissions-827&utm_source=dac&utm_medium=blog) including best practices around when to request permissions, how to explain why permissions are needed, and how to handle permissions being denied.

<!--more-->

![](/images/categories/android/android_notes/062/runtime-permission-promt.png)

In many cases, you can avoid permissions altogether by using the existing intents system to utilize other existing specialized apps rather than building a full experience within your app. An example of this is using _ACTION_IMAGE_CAPTURE_ to start an existing camera app the user is familiar with rather than building your own camera experience. Learn more about permissions versus intents.

However, if you do need a runtime permission, there’s a number of tools to help you. Checking for whether your app has a permission is possible with _ContextCompat.checkSelfPermission()_ (available as part of revision 23 of the support-v4 library for backward compatibility) and requesting permissions can be done with _requestPermissions()_, bringing up the system controlled permissions dialog to allow the user to grant you the requested permission(s) if you don’t already have them. Keep in mind that users can revoke permissions at any time through the system settings so you should always check permissions every time.

A special note should be made around _shouldShowRequestPermissionRationale()_. This method returns _true_ if the user has denied your permission request at least once yet have not selected the _‘Don’t ask again’_ option (which appears the second or later time the permission dialog appears). This gives you an opportunity to provide additional education around the feature and why you need the given permission. Learn more about explaining why the app needs permissions.

Read through the design guidelines and our [developer guide](https://developer.android.com/training/permissions/index.html?utm_campaign=runtime-permissions-827&utm_source=dac&utm_medium=blog) for all of the details in getting your app ready for Android 6.0 and runtime permissions. Making it easy to install your app and providing context around accessing user’s sensitive data are key changes you can make to build better apps.
