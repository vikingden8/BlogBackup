---
title: Android获取项目的方法数
date: 2017-3-30 17:24:39
tags:
categories: "Android学习笔记"
---
大家都知道Android有65535方法数的问题，也就是说App的Java代码Method总数、Field总数都不能超过65535个，那我们有什么办法能查看我们App的这个值已经有多少了呢？

### dexdump命令

在Linux上，可以执行以下命令：

```sh

#查看apk的method总数
dexdump -f [apk path] | grep method_ids_size

#查看apk的field总数
dexdump -f [apk path] | grep field_ids_size

```

<!--more-->

在Windows上，由于没有grep命令，可以使用findstr替代：

```sh
#查看apk的method总数
dexdump -f [apk path] | findstr method_ids_size

#查看apk的field总数
dexdump -f [apk path] | findstr field_ids_size
```

-f后面的参数不一定是.apk，也可以是后缀.dex、.jar、.zip的文件。

命令中的dexdump在sdk的路径中，比如我的就在下面的目录：

![](/images/categories/android/android_notes/036/01.png)

如下，在我的电脑中执行命令的结果返回：

![](/images/categories/android/android_notes/036/02.png)

### 使用Gradle的插件com.getkeepsafe.dexcount

如果使用Android Studio的话可以使用这个插件，插件的地址[在这里](https://github.com/KeepSafe/dexcount-gradle-plugin)

具体的配置，只需在你的module下的build.gradle添加如下：

```sh
buildscript {
    repositories {
        mavenCentral() // or jcenter()
    }

    dependencies {
        classpath 'com.getkeepsafe.dexcount:dexcount-gradle-plugin:0.6.3'
    }
}

// make sure this line comes *after* you apply the Android plugin
apply plugin: 'com.getkeepsafe.dexcount'
```

>注意，必须要保证　_apply plugin: 'com.getkeepsafe.dexcount'_ 在　_apply plugin: 'com.android.application'_ 之后

当你执行编译和打包命令的时候，这个插件会自动被执行，然后执行结果会被详细记录在/app/build/outputs/dexcount/debug.txt里边

但是这个插件是基于Java 8.
