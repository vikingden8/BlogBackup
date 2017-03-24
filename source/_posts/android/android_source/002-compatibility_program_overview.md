---
title: Compatibility Program Overview
date: 2017-03-24 22:42:29
tags:
categories: "Android Sources Website"
---

The Android compatibility program makes it easy for mobile device manufacturers to develop compatible Android devices.

### Program goals

The Android compatibility program works for the benefit of the entire Android community, including **users**, **developers**, and **device manufacturers**.

Each group depends on the others. Users want a wide selection of devices and great apps; great apps come from developers motivated by a large market for their apps with many devices in users' hands; device manufacturers rely on a wide variety of great apps to increase their products' value for consumers.

<!--more-->

Our goals were designed to benefit each of these groups:

  * _Provide a consistent application and hardware environment to application developers_. Without a strong compatibility standard, devices can vary so greatly that developers must design different versions of their applications for different devices. The compatibility program provides a precise definition of what developers can expect from a compatible device in terms of APIs and capabilities. Developers can use this information to make good design decisions, and be confident that their apps will run well on any compatible device.

  * _Enable a consistent application experience for consumers_. If an application runs well on one compatible Android device, it should run well on any other device that is compatible with the same Android platform version. Android devices will differ in hardware and software capabilities, so the compatibility program also provides the tools needed for distribution systems such as Google Play to implement appropriate filtering. This means users see only the applications they can actually run.

  * _Enable device manufacturers to differentiate while being compatible._ The Android compatibility program focuses on the aspects of Android relevant to running third-party applications, which allows device manufacturers the flexibility to create unique devices that are nonetheless compatible.

  * _Minimize costs and overhead associated with compatibility_. Ensuring compatibility should be easy and inexpensive to device manufacturers. The testing tool is free, open source, and available for download. It is designed to be used for continuous self-testing during the device development process to eliminate the cost of changing your workflow or sending your device to a third party for testing. Meanwhile, there are no required certifications, and thus no corresponding costs and fees.

### Program components

The Android compatibility program consists of three key components:

  * The Android Open Source Project source code

  * The Compatilbility Definition Document (CDD), representing the "policy" aspect of compatibility

  * The Compatilbility Test Suite (CTS), representing the "mechanism" of compatibility

Just as each version of the Android platform exists in a separate branch in the source code tree, there is a separate CTS and CDD for each version as well. The CDD, CTS, and source code are -- along with your hardware and your software customizations -- everything you need to create a compatible device.
