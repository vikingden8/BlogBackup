---
title: 避免Android中Context引起的内存泄露
date: 2016-12-30 17:03:48
tags:
categories: "Android学习笔记"
---

Context是我们在编写Android程序经常使用到的对象，意思为上下文对象。 常用的有Activity的Context还是有Application的Context。Activity用来展示活动界面，包含了很多的视图，而视图又含有图片，文字等资源。在Android中内存泄露很容易出现，而持有很多对象内存占用的Activity更加容易出现内存泄露，开发者需要特别注意这个问题。

本文讲介绍Android中Context，更具体的说是Activity内存泄露的情况，以及如何避免Activity内存泄露，加速应用性能。  


### Drawable引起的内存泄露  
