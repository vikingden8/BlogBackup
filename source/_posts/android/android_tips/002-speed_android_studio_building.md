---
title: 加速Gradle构建
date: 2017-04-16 22:14:41
tags:
categories: "Android设计"
---

随着项目的增大，依赖库的增多，构建速度越来越慢，现在需要好几分钟才能build一个release的安装包，在网上查找资料，发现可以通过一些配置可以加快速度，这里跟大家分享一下。

### 开启gradle单独的守护进程

在下面的目录下面创建gradle.properties文件：

```sh
/home/<username>/.gradle/ (Linux)
/Users/<username>/.gradle/ (Mac)
C:\Users\<username>\.gradle (Windows)
```

并在文件中增加：

```sh
org.gradle.daemon=true
```

<!--more-->

同时修改项目下的gradle.properties文件也可以优化：

```sh
# Project-wide Gradle settings.  

# IDE (e.g. Android Studio) users:  
# Settings specified in this file will override any Gradle settings  
# configured through the IDE.  

# For more details on how to configure your build environment visit  
# http://www.gradle.org/docs/current/userguide/build_environment.html  

# The Gradle daemon aims to improve the startup and execution time of Gradle.  
# When set to true the Gradle daemon is to run the build.  
# TODO: disable daemon on CI, since builds should be clean and reliable on servers  
org.gradle.daemon=true  

# Specifies the JVM arguments used for the daemon process.  
# The setting is particularly useful for tweaking memory settings.  
# Default value: -Xmx10248m -XX:MaxPermSize=256m  
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

# When configured, Gradle will run in incubating parallel mode.  
# This option should only be used with decoupled projects. More details, visit  
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects  
org.gradle.parallel=true  

# Enables new incubating mode that makes Gradle selective when configuring projects.   
# Only relevant projects are configured which results in faster builds for large multi-projects.  
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:configuration_on_demand  
org.gradle.configureondemand=true  
```

同时上面的这些参数也可以配置到前面的用户目录下的gradle.properties文件里，那样就不是针对一个项目生效，而是针对所有项目生效。

上面的配置文件主要就是做， 增大gradle运行的java虚拟机的大小，让gradle在编译的时候使用独立进程，让gradle可以平行的运行。

### 修改android studio配置

在android studio的配置中，开启offline模式，以及修改配置。实际上的配置和上面的一大段一样，主要是在这个地方配置的只会在ide构建的时候生效，命令行构建不会生效。

![](/images/categories/android/android_tips/002/as_gradle_offline.png)

![](/images/categories/android/android_tips/002/as_gradle_config.png)

### 命令行构建

基于上面的配置，命令行构建时在命令后面加上这个参数即可 --daemon --parallel --offline。

转载自：[加速Android Studio/Gradle构建](http://blog.isming.me/2015/03/18/android-build-speed-up/)
