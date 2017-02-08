---
title: Android中的DX工具
date: 2017-2-5 11:07:39
tags:
categories: "Android学习笔记"
---

### What is dx tool?

The dx tool converts Java class files into a .dex (Dalvik Executable) file

### Where is it?

The dx.jar was original located under android-sdk/platforms/android-X/tools/lib/ before (especially in android-3 and android-4), and moved to android-sdk/platform-tools/lib/ later.

### How does it fit in Android?

The Java source files are converted to Java class files by the Java compiler.

The dx tool converts Java class files into a .dex (Dalvik Executable) file. All class files of the application are placed in this .dex file. During this conversion process redundant information in the class files are optimized in the .dex file.

For example, if the same String is found in different class files, the .dex file contains only one reference of this String.

These .dex files are therefore much smaller in size than the corresponding class files.

The .dex file and the resources of an Android project, e.g., the images and XML files, are packed into an .apk (Android Package) file.

To understand better look at the android build process：

![](/images/categories/android/android_notes/014/android_build_process.png)

**FYI:**

The program aapt (Android Asset Packaging Tool) performs apk creation. The resulting .apk file contains all necessary data to run the Android application and can be deployed to an Android device via the adb(android device bridge) tool.

You can use the dx tool from the sdk, from the command line. Something like:

```sh
dx --dex --output=dexed.jar hello.jar
```
### 资料

  * [Android dx tool](http://stackoverflow.com/questions/8487268/android-dx-tool)

  * [Android Build Process](https://developer.android.com/studio/build/index.html)

  * [Google I/O 2013 - The New Android SDK Build System](https://www.youtube.com/watch?v=LCJAgPkpmR0#t=504)
