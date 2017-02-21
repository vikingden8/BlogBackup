---
title: Android Apps Break the 50MB Barrier
date: 2017-02-21 22:29:27
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

Android applications have historically been limited to a maximum size of 50MB. This works for most apps, and smaller is usually better — every megabyte you add makes it harder for your users to download and get started. However, some types of apps, like high-quality 3D interactive games, require more local resources.

So today, we’re expanding the Android app size limit to 4GB.

The size of your APK file will still be limited to 50MB to ensure secure on-device storage, but you can now attach expansion files to your APK.

Each app can have two expansion files, each one up to 2GB, in whatever format you choose.

Android Market will host the files to save you the hassle and cost of file serving.

Users will see the total size of your app and all of the downloads before they install/purchase.

On most newer devices, when users download your app from Android Market, the expansion files will be downloaded automatically, and the refund period won’t start until the expansion files are downloaded. On older devices, your app will download the expansion files the first time it runs, via a downloader library which we’ve provided below.

While you can use the two expansion files any way you wish, we recommend that one serve as the initial download and be rarely if ever updated; the second can be smaller and serve as a “patch carrier,” getting versioned with each major release.

<!--more-->
