---
title: Configure Apps with Over 64K Methods(Android项目方法数超过64K的配置)
date: 2017-3-28 10:51:02
tags:
categories: "Android学习笔记"
---

As the Android platform has continued to grow, so has the size of Android apps. When your app and the libraries it references reach a certain size, you encounter build errors that indicate your app has reached a limit of the Android app build architecture. Earlier versions of the build system report this error as follows:

随着Android平台的持续增长，Android应用包的大小也在增大。当你的应用以及引用的依赖包大小达到一定的值，你将会遇到构建错误的提示，显示你的应用大小已经达到了Android应用构建架构的限制大小。早期的构建系统将会显示类似下述的错误信息：

```sh
Conversion to Dalvik format failed:
Unable to execute dex: method ID not in [0, 0xffff]: 65536
```

More recent versions of the Android build system display a different error, which is an indication of the same problem:

最近几个新版本的Android构建系统将会显示不同的错误信息，但是是同一个问题：

```sh
trouble writing output:
Too many field references: 131000; max is 65536.
You may try using --multi-dex option.
```

Both these error conditions display a common number: 65,536. This number is significant in that it represents the total number of references that can be invoked by the code within a single Dalvik Executable (DEX) bytecode file. This page explains how to move past this limitation by enabling an app configuration known as multidex, which allows your app to build and read multiple DEX files.

上面的两个错误信息都显示了同一个数值：65536。这个数字很重要，因为它代表的是单个 Dalvik Executable (dex) 字节码文件内的代码可调用的引用总数。这篇文章将阐述在你的应用中通过配置multidex来避免上述的编译错误，让你的应用能构建和读取多个DEX文件。

<!--more-->

### About the 64K reference limit - 关于64K引用限制

Android app (APK) files contain executable bytecode files in the form of Dalvik Executable (DEX) files, which contain the compiled code used to run your app. The Dalvik Executable specification limits the total number of methods that can be referenced within a single DEX file to 65,536—including Android framework methods, library methods, and methods in your own code. In the context of computer science, the term Kilo, K, denotes 1024 (or 2^10). Because 65,536 is equal to 64 X 1024, this limit is referred to as the '64K reference limit'.

Android应用（APK）文件里面包含了.dex格式的可执行二进制文件，也就是运行你应用的已编译代码。单个DEX可执行文件对总的方法数有特殊的限制：65536。这个数值包含了Android框架的方法数，依赖库的方法数以及自己应用代码中的方法数。在计算机科学领域内，术语千（简称 K）表示 1024（或 2^10）。由于 65,536 等于 64 X 1024，因此这一限制也称为“64K 引用限制”。

### Multidex support prior to Android 5.0 - Android 5.0之前的Multidex使用

Versions of the platform prior to Android 5.0 (API level 21) use the Dalvik runtime for executing app code. By default, Dalvik limits apps to a single classes.dex bytecode file per APK. In order to get around this limitation, you can use the [multidex support library](https://developer.android.com/topic/libraries/support-library/features.html#multidex), which becomes part of the primary DEX file of your app and then manages access to the additional DEX files and the code they contain.

Android 5.0 (API 21)之前的平台版本使用的是Dalvik运行时来执行应用代码。默认情况下，Dalvik限制应用的每个APK只能使用单个classes.dex字节码文件。为了能越过这个限制，你可以使用支持库中的multidex，它会成为你应用主要DEX文件的一部分，然后管理对其他DEX文件及其所包含代码的访问。

>Note: If your project is configured for multidex with minSdkVersion 20 or lower, and you deploy to target devices running Android 4.4 (API level 20) or lower, Android Studio disables Instant Run.

>如果你的项目在配置了multidex的同时，也配置了minSdkVersion为20或者更小，当你部署机器运行的是Android 4.4（API 20）或者之前的版本时，Android Studio将会关闭Instant Run功能。

### Multidex support for Android 5.0 and higher - Android 5.0之后的Multidex使用

Android 5.0 (API level 21) and higher uses a runtime called ART which natively supports loading multiple DEX files from APK files. ART performs pre-compilation at app install time which scans for classesN.dex files and compiles them into a single .oat file for execution by the Android device. Therefore, if your minSdkVersion is 21 or higher, you do not need the multidex support library.

Android 5.0（API 21）及更高版本使用ART运行时，后者原生支持从应用APK文件加载多个dex文件。ART在应用安装时执行预编译，扫描classes(..N).dex文件，并将它们编译成单个.oat文件来给Android设备执行。

For more information on the Android 5.0 runtime, read [ART and Dalvik](https://source.android.com/devices/tech/dalvik/index.html).

如需了解有关Android 5.0运行时的详细信息，请参阅 [ART和Dalvik](https://source.android.com/devices/tech/dalvik/index.html)。

>Note: While using Instant Run, Android Studio automatically configures your app for multidex when your app's minSdkVersion is set to 21 or higher. Because Instant Run only works with the debug version of your app, you still need to configure your release build for multidex to avoid the 64K limit.

当你使用Instant Run，并且你的应用配置minSdkVersion为21或者更高时，Android Studio会自动为你的应用设置multidex支持。由于Instant Run仅适用于调试版本的应用，你仍需在发布版本构建时配置multidex，以避免64K限制。

### Avoid the 64K limit - 避免64K的限制

Before configuring your app to enable use of 64K or more method references, you should take steps to reduce the total number of references called by your app code, including methods defined by your app code or included libraries. The following strategies can help you avoid hitting the DEX reference limit:

在将你的应用配置支持使用64K或更多方法引用之前，你应该采取措施减少应用代码调用的引用总数，包括你的应用代码或依赖库定义的方法。下列策略可帮助你避免达到 DEX引用限制：

  * **Review your app's direct and transitive dependencies** - Ensure any large library dependency you include in your app is used in a manner that outweighs the amount of code being added to the app. A common anti-pattern is to include a very large library because a few utility methods were useful. Reducing your app code dependencies can often help you avoid the DEX reference limit.

  确保你在应用中使用任何庞大依赖库所带来的好处大于为应用添加大量代码所带来的弊端。一种常见的反面模式是，仅仅为了使用几个实用方法就在应用中加入非常庞大的依赖库。减少你的应用代码依赖项往往能够帮助您规避DEX引用限制。

  * **Remove unused code with ProGuard** - Enable code shrinking to run ProGuard for your release builds. Enabling shrinking ensures you are not shipping unused code with your APKs.

  在发行版本时使用ProGuard来启用压缩代码。启用压缩可确保你APK不含有未使用的代码。

Using these techniques might help you avoid the need to enable multidex in your app while also decreasing the overall size of your APK.

使用那些技术可能帮助你避免使用multidex，同时也减小了你整个应用的大小。

### Configure your app for multidex - 应用中配置multidex

Setting up your app project to use a multidex configuration requires that you make the following modifications to your app project, depending on the minimum Android version your app supports.

在你的项目中配置multidex需要以下的几个步骤，不同的配置依靠你使用的minSdkVersion。

If your minSdkVersion is set to 21 or higher, all you need to do is set multiDexEnabled to true in your module-level build.gradle file, as shown here:

如果你的应用minSdkVersion为21或更高，你只需要做一件事，就是在你module中的build.gradle文件中配置multiDexEnabled为true。

```sh
android {
    defaultConfig {
        ...
        minSdkVersion 21
        targetSdkVersion 25
        multiDexEnabled true
    }
    ...
}
```

However, if your minSdkVersion is set to 20 or lower, then you must use the multidex support library as follows:

但是，如果你的minSdkVersion设置为20或者更小，那么你必须使用支持库中的multidex。

* Modify the module-level build.gradle file to enable multidex and add the multidex library as a dependency, as shown here:

  修改mudule下的build.gradle文件，开启multidex以及添加multidex依赖库作为依赖，如下所示：

  ```sh
  android {
      defaultConfig {
          ...
          minSdkVersion 15
          targetSdkVersion 25
          multiDexEnabled true
      }
      ...
  }
  ```

  ```sh
  dependencies {
    compile 'com.android.support:multidex:1.0.1'
  }
  ```

* Depending on whether you override the Application class, perform one of the following:

  根据你是否有复写Appliction，你需要执行以下中的一种：

  1. If you do not override the Application class, edit your manifest file to set android:name in the <application> tag as follows:

  如果你没有复写Application，则只需编辑你的AndroidMinifest.xml文件，在<aplicaion>节点中设置android:name即可，如下：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="com.example.myapp">
      <application
              android:name="android.support.multidex.MultiDexApplication" >
          ...
      </application>
  </manifest>
  ```

  2. If you do override the Application class, change it to extend MultiDexApplication (if possible) as follows:

  如果你复写了Application类，改变它的继承类为MultiDexApplication。

  ```java
  public class MyApplication extends MultiDexApplication { ... }
  ```

  3. Or if you do override the Application class but it's not possible to change the base class, then you can instead override the attachBaseContext() method and call MultiDex.install(this) to enable multidex:

  如果你复写了Application类，但是你又不想改变集成的父类，那么你可以在attachBaseContext()方法中添加MultiDex.install(this)来开启multidex：

  ```java
  public class MyApplication extends SomeOtherApplication {
    @Override
    protected void attachBaseContext(Context base) {
       super.attachBaseContext(base);
       MultiDex.install(this);
    }
  }
  ```

  >Caution: Do not execute MultiDex.install() or any other code through reflection or JNI before MultiDex.install() is complete. Multidex tracing will not follow those calls, causing ClassNotFoundException or verify errors due to a bad class partition between DEX files.

Now when you build your app, the Android build tools construct a primary DEX file (classes.dex) and supporting DEX files (classes2.dex, classes3.dex, and so on) as needed. The build system then packages all DEX files into your APK.

现在你构建应用，Android构建系统工具将会创建一个主要的DEX文件(classes.dex)，以及在必要的时候创建多个DEX文件（如classes2.dex， classes3.dex)。Android构建系统将会打包所有的DEX文件到你的应用APK中。

At runtime, the multidex APIs use a special class loader to search all of the available DEX files for your methods (instead of searching only in the main classes.dex file).

在运行时，multidex的API使用一个特殊的类加载器在所有可用的DEX文件中搜索你的方法。

### References

[https://developer.android.com/studio/build/multidex.html](https://developer.android.com/studio/build/multidex.html)
