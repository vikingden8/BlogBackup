---
title: Android export属性
date: 2016-12-28 09:49:20
tags:
categories: "Android学习笔记"
---

android:exported是Android中四大组件Activity、Service、Broadcast Receiver和Content Provider都会有的一个属性。  

总的来说，主要作用就是：是否支持其他应用调用当前的组件。  

默认值：如果当前的组件在AndroidMinifest.xml配置了intent-filter，则默认值为true；没有配置的话，默认值为false。  

下面以Activity为例，看下官方的语法说明：  

```java
<activity
  ......
  android:exported=["true" | "false"]
  ......
```
<!--more-->  

![语法说明](/images/categories/android/android_notes/android_exported/android_exported_doc.png)  

android:exported是个可选属性，那么，Android系统是如何设置exported属性的值呢？  

通过阅读系统源码，AndroidMinifest.xml是在：  

>frameword/base/core/java/android/content/pm/PackageParse.java  

中解析，找到其中解析activity项的源码，在parseActivity函数：  

```java
private Activity parseActivity(Package owner, Resources res,
            XmlPullParser parser, AttributeSet attrs, int flags, String[] outError,
            boolean receiver, boolean hardwareAccelerated)

        ...

        boolean setExported = sa.hasValue(R.styleable.AndroidManifestActivity_exported);
        if (setExported) {
            a.info.exported = sa.getBoolean(R.styleable.AndroidManifestActivity_exported, false);
        }

        ...

        if (!setExported) {
            a.info.exported = a.intents.size() > 0;
        }
        return a;
    }
```  

这里，我们只保留了与exported属性解析相关的源码。从源码中，我们可以得出：  

  - 如果显式exported属性，不管这个activity有没有设置intent-filter,那么exported的值就是显式设置的值  

  - 如果没有设置exported属性，那么exported属性取决于这个activity是否有intent-filter   

    * 如有intent-filter,那么exported的值就是true  

    * 如没有intent-filter,那么exported的值就是false  
