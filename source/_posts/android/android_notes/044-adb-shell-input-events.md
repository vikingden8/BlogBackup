---
title: ADB Shell Input Events
date: 2017-4-22 09:10:01
tags:
categories: "Android学习笔记"
---

Using adb shell input:

* **Insert text**:

```sh
adb shell input text "insert%syour%stext%shere"
```

(obs: %s means SPACE)

* **Event codes**:

```sh
adb shell input keyevent 82
```
(82 ---> MENU_BUTTON)

<!--more-->

* **Tap X,Y position**:

```sh
adb shell input tap 500 1450
```

To find the exact X,Y position you want to Tap go to:

>Settings > Developer Options > Check the option POINTER SLOCATION

* **Swipe X1 Y1 X2 Y2 [duration(ms)]**:

```sh
adb shell input swipe 100 500 100 1450 100
```

in this example X1=100, Y1=500, X2=100, Y2=1450, Duration = 100ms

* **LongPress X Y**:

```sh
adb shell input swipe 100 500 100 500 250
```

we utilise the same command for a swipe to emulate a long press

in this example X=100, Y=500, Duration = 250ms

The complete list of commands can be found on: [http://developer.android.com/reference/android/view/KeyEvent.html](http://developer.android.com/reference/android/view/KeyEvent.html)
