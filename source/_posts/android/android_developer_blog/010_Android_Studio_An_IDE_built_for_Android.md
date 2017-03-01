---
title: Android Studio: An IDE built for Android
date: 2017-03-01 21:55:59
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

Posted by Xavier Ducrohet, Tor Norbye, Katherine Chou

Today at Google I/O we announced a new IDE that’s built with the needs of Android developers in mind. It’s called Android Studio, it’s free, and it’s available now for you to try as an early access preview.

To develop Android Studio, we cooperated with JetBrains, creators of one of the most advanced Java IDEs available today. Based on the powerful, extensible IntelliJ IDEA Community Edition, we've added features that are designed specifically for Android development, that simplify and optimize your daily workflow.

### Extensible build tools

![](/images/categories/android/android-developer-blog/011/Studio_table.png)

We know you need a build system that adapts to your project requirements but extends further to your larger development environment. Android Studio uses a new build system based on Gradle that provides flexibility, customized build flavors, dependency resolution, and much more.

This new build system allows you to build your projects in the IDE as well as on your continuous integrations servers. The combination lets you easily manage complex build configurations natively, throughout your workflow, across all of your tools. Check out the preview documentation to get a better idea of what the new build system can do.
<!--more-->
### Powerful code editing

![](/images/categories/android/android-developer-blog/011/laptop600.png)

Android Studio includes a powerful code editor. It's based on the IntelliJ IDEA editor, which supports features such as smart editing, advanced code refactoring, and deep static code analysis.

Smart editing features such as inline resource lookups make it easier to read your code, while giving you instant access to edit code the backing resources. Advanced code refactoring gives you the power to transform your code across the scope of the entire project, quickly and safely.

We added static code analysis for Android development, helping you identify bugs more quickly. On top of the hundreds of code inspections that IntelliJ IDEA provides, we’ve added custom inspections. For example, we’ve added metadata to the Android APIs, that flag which methods can return null and which can’t, which constants are allowed for which methods, and so on. Android Studio uses that data to analyze your code and find potential errors.

### Smoother and richer GUI

![](/images/categories/android/android-developer-blog/011/ide-refactor.png)

![](/images/categories/android/android-developer-blog/011/ide-smart.png)

![](/images/categories/android/android-developer-blog/011/ide-resourcelookup2.png)

Over the past year we’ve added some great drag-and-drop UI features to ADT and we’re in the process of adding them all into Android Studio. This release of Android Studio lets you preview your layouts on different device form factors, locales, and platform versions.

### Easy access to Google services within Android Tools

We wanted to make it easy for you to harness the power Google services right from your IDE. To start, we’ve made it trivial to add services such a cloud-based backend with integrated Google Cloud Messaging (GCM) to your app, directly from the IDE.

We’ve also added a new plugin called ADT Translation Manager Plugin to assist with localizing your apps. You can use the plugin to export your strings to the Google Play Developer Console for translation, then download and import your translations back into your project.

### Open source development

Starting next week we’ll be doing all of our development in the open, so you can follow along or make your own contributions. You can find the Android Studio project in AOSP at [https://android.googlesource.com/platform/tools/adt/idea/](https://android.googlesource.com/platform/tools/adt/idea/)

### Try Android Studio and give us feedback

Give Android Studio a try and send us your feedback! It's free, and the download bundle includes includes everything you need, including the IDE, the latest SDK tools, the latest Android platform, and more. .

Note: This is an early access preview intended for early adopters and testers who want to influence the direction of the Android tools. If you have a production app with a large installed base, there’s no need to migrate your development to the new tools at this time. We will continue to support Eclipse as a primary platform for development.

If you have feedback on the tools, you can send it to us using the Android Studio issue tracker.

### References

[Android Studio: An IDE built for Android](https://android-developers.googleblog.com/2013/05/android-studio-ide-built-for-android.html)
