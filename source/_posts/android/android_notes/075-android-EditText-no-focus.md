---
title: 如何让EditText不自动获取焦点
date: 2018-01-20 21:52:50
tags:
categories: "Android学习笔记"
---

转载自：[如何让EditText不自动获取焦点](http://androidperformance.com/2014/06/03/android-edittext-do-not-auto-get-focus.html)

Android中,使用EditText作为输入框很方便,但是有时候EditText会自动获取焦点,其行为:点击进入这个页面后,EditText自动获取焦点,导致软键盘直接跳出.有时候这么做很方便,但是大部分时候我们还是希望在点击EditText的时候,软键盘才弹出来.

这里有个很简单也很实用的技巧,即在EditText的父Layout中,加入下面的两个属性即可:

<!--more-->

```xml
android:focusable=”true”
android:focusableInTouchMode=”true”
```

这样做的原理是让用户进入到这个页面之后,EditText的父控件 获取焦点,这样的话EditText就获取不到焦点,软键盘也不会自动弹起.只有在点击EditText的时候,软键盘才会弹起.
