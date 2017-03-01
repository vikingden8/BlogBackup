---
title: Native RTL support in Android 4.2
date: 2017-03-01 21:51:25
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

Posted by Fabrice Di Meglio, Android Frameworks Team
Android 4.1 (Jelly Bean) introduced limited support for bidirectional text in TextView and EditText elements, allowing apps to display and edit text in both left-to-right (LTR) and right-to-left (RTL) scripts. Android 4.2 added full native support for RTL layouts, including layout mirroring, allowing you to deliver the same great app experience to all of your users, whether their language uses a script that reads right-to-left or one that reads left-to-right.

If you do nothing, your app will not change — it will continue to appear as it currently does. However, with a few simple changes, your app will be automatically mirrored when the user switches the system language to a right-to-left script (such as Arabic, Hebrew, or Persian). For example, see the following screenshots of the Settings app:

![](/images/categories/android/android-developer-blog/010/setings-ltr.png)

<!--more-->

To take advantage of RTL layout mirroring, simply make the following changes to your app:

  * Declare in your app manifest that your app supports RTL mirroring.

  Specifically, add android:supportsRtl="true" to the <application> element in your manifest file.

  * Change all of your app's "left/right" layout properties to new "start/end" equivalents.

  If you are targeting your app to Android 4.2 (the app's targetSdkVersion or minSdkVersion is 17 or higher), then you should use “start” and “end” instead of “left” and “right”. For example, android:paddingLeft should become android:paddingStart.

  If you want your app to work with versions earlier than Android 4.2 (the app's targetSdkVersion or minSdkVersion is 16 or less), then you should add “start” and end” in addition to “left” and “right”. For example, you’d use both android:paddingLeft and android:paddingStart.

For more precise control over your app UI in both LTR and RTL mode, Android 4.2 includes the following new APIs to help manage View components:

  * android:layoutDirection — attribute for setting the direction of a component's layout.

  * android:textDirection — attribute for setting the direction of a component's text.

  * android:textAlignment — attribute for setting the alignment of a component's text.

  * getLayoutDirectionFromLocale() — method for getting the Locale-specified direction

You can even create custom versions of layout, drawables, and other resources for display when a right-to-left script is in use. Simply use the resource qualifier "ldrtl" to tag your resources, meaning “layout direction right-to-left”. To debug and optimize custom right-to-left layouts, HierarchyViewer now lets you see start/end properties, layout direction, text direction, and text alignment for all the Views in the hierarchy.

It's now easy to create beautiful Android apps for all your users, whether they use a right-to-left or left-to-right language. We look forward to seeing some great apps!

### References

[Native RTL support in Android 4.2](https://android-developers.googleblog.com/2013/03/native-rtl-support-in-android-42.html)
