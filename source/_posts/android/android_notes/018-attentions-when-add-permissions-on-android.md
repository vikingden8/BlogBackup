---
title: 为Android程序申请权限注意
date: 2017-02-19 17:35:46
tags:
categories: "Android学习笔记"
---

Android系统提供为程序提供了权限申请,即在manifest中使用uses-permission来申请即可.实现起来非常简单,但是有些问题会随之浮出水面. 常见的现象是,有时候新加一个权限,(在Google Play上)程序显示的支持的设备会减少.

#### 为什么权限越多,支持设备越少

因为有些权限隐式地需要feature,即当你显示使用uses-permission,会默认地为程序加入uses-feature.
而Android以及Google Play判断是否可以安装的依据是,设备包含的system features是否完全包含程序申请的全部features. **只有在全部满足了程序需要的feature的设备上才可以展示并安装**.
