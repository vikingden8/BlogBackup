---
title: Android Compatibility Definition Document
date: 2017-03-24 22:51:43
tags:
categories: "Android Sources Website"
---

Welcome to the Android Compatibility Definition Document (CDD). This document enumerates the requirements that must be met in order for devices to be compatible with the latest version of Android. To be considered compatible with Android, device implementations MUST meet the requirements presented in this Compatibility Definition, including any documents incorporated via reference. For each release of the Android platform, a detailed CDD will be provided. The CDD represents the "policy" aspect of Android compatibility.

It is important the policy of the Android compatibility program is codified explicitly as no test suite, including CTS, can truly be comprehensive. For instance, the CTS includes a test that checks for the presence and correct behavior of OpenGL graphics APIs, but no software test can verify that the graphics actually appear correctly on the screen. More generally, it's impossible to test the presence of hardware features such as keyboards, display density, Wi-Fi, and Bluetooth.

<!--more-->

The CDD's role is to codify and clarify specific requirements, and eliminate ambiguity. The CDD does not attempt to be comprehensive. Since Android is a single corpus of open-source code, the code itself is the comprehensive "specification" of the platform and its APIs. The CDD acts as a "hub" referencing other content (such as SDK API documentation) that provides a framework in which the Android source code may be used so that the end result is a compatible system.

If you want to build a device compatible with a given Android version, start by checking out the source code for that version, and then read the corresponding CDD and stay within its guidelines. For additional details, simply examine the latest CDD.

You may view the latest CDD either as an HTML web page or an easily downloadable PDF:

  * [HTML](http://source.android.com/compatibility/android-cdd.html)

  * [PDF](http://static.googleusercontent.com/media/source.android.com/zh-CN//compatibility/android-cdd.pdf)

Find older versions of the CDD and approved release version strings here:

![](http://source.android.com/compatibility/cdd.html)
