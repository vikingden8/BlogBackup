---
title: Android Compatibility
date: 2017-03-24 21:34:37
tags:
categories: "Android Sources Website"
---

Android's purpose is to establish an open platform for developers to build innovative apps.

Android的目标是为开发者建立一个开放平台来创造出具有创新性的应用。

  * The Android Compatibility program defines technical details of the Android platform and provides tools for OEMs to ensure developer applications run on a variety of devices.

  Android兼容性程序定义了Android平台的技术细节，并且为OEM开发商提供工具来确保开发者的应用能在不同的机器上正常运行。

  * The Android SDK provides built-in tools for developers to clearly state the device features required by their applications.

  Android SDK为开发人员提供内置工具，以清楚地说明其应用程序所需的设备功能。

  * Google Play shows applications only to those devices that can properly run those applications.

  Google Play只会在运行正常的兼容性程序设备上提供应用市场功能。

<!--more-->

### Why build compatible Android devices?

![Figure 1. The Android ecosystem thrives with device compatibility](/images/categories/android/android-sources/001/compat-ecosystem.png)


  **Users want customizable devices**

  A mobile phone is a highly personal, always-on, always-present gateway to the Internet. We haven't met a user yet who didn't want to customize it by extending its functionality. That's why Android was designed as a robust platform for running aftermarket applications.

  **Developers outnumber us all**

  No device manufacturer can write all the software a user could conceivably need. We need third-party developers to write the apps users want, so the Android Open Source Project (AOSP) aims to make application development as easy and open as possible.

  **Everyone needs a common ecosystem**

  Every line of code developers write to work around a bug is a line of code that didn't add a new feature. The more compatible mobile devices are, the more applications we'll have to run on those devices. By building a fully compatible Android device, you benefit from the huge pool of apps written for Android while increasing the incentive for developers to build more apps.

### Android compatibility is free, and it's easy

To build an Android-compatible mobile device, follow this three-step process:

  1. Obtain the [Android software source code](http://source.android.com/source/index.html). This is the source code for the Android platform that you port to your hardware.

  2. Comply with the Android Compatibility Definition Document (CDD) ([PDF](http://static.googleusercontent.com/media/source.android.com/zh-CN//compatibility/android-cdd.pdf), [HTML](http://source.android.com/compatibility/android-cdd.html)). The CDD enumerates the software and hardware requirements of a compatible Android device.

  3. Pass the [Compatibility Test Suite (CTS)](http://source.android.com/compatibility/cts/index.html). Use the CTS as an ongoing aid to evaluate compatibility during the development process.

After complying with the CDD and passing the CTS, your device is Android compatible, meaning Android apps in the ecosystem provide a consistent experience when running on your device. For details about the Android compatibility program, see the [program overview](http://source.android.com/compatibility/overview.html).

### Licensing Google Mobile Services (GMS)

After building an Android compatible device, consider licensing Google Mobile Services (GMS), Google’s proprietary suite of apps (Google Play, YouTube, Google Maps, Gmail, and more ) that run on top of Android. GMS is not part of the Android Open Source Project and is available only through a license with Google. For information on how to request a GMS license, see [Contact Us](http://source.android.com/compatibility/contact-us.html).
