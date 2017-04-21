---
title: Difference between View and ViewGroup in Android
date: 2017-4-21 09:13:31
tags:
categories: "Android学习笔记"
---

### UI Overview

All user interface elements in an Android app are built using _View_ and _ViewGroup_ objects. A _View_ is an object that draws something on the screen that the user can interact with. A _ViewGroup_ is an object that holds other _View_ (and _ViewGroup_) objects in order to define the layout of the interface.

>Android provides a collection of both _View_ and _ViewGroup_ subclasses that offer you common input controls (such as buttons and text fields) and various layout models (such as a linear or relative layout).

#### User Interface Layout

The user interface for each component of your app is defined using a hierarchy of _View_ and _ViewGroup_ objects, as shown in figure 1. Each view group is an invisible container that organizes child views, while the child views may be input controls or other widgets that draw some part of the UI. This hierarchy tree can be as simple or complex as you need it to be (but simplicity is best for performance).

Illustration of a view hierarchy, which defines a UI layout:

![](/images/categories/android/android_notes/043/viewgroup_2x.png)


### View

  * View objects are the basic building blocks of User Interface(UI) elements in Android.

  * View is a simple rectangle box which responds to the user's actions.

  * Examples are EditText, Button, CheckBox etc..

  * View refers to the android.view.View class, which is the base class of all UI classes.

### ViewGroup

  * ViewGroup is the invisible container. It holds View and ViewGroup

  * For example, LinearLayout is the ViewGroup that contains Button(View), and other Layouts also.

  * ViewGroup is the base class for Layouts.

### References

[Difference between View and ViewGroup in Android
](http://stackoverflow.com/questions/27352476/difference-between-view-and-viewgroup-in-android)

[UI Overview](https://developer.android.com/guide/topics/ui/overview.html)
