---
title: Android Layout Tricks
date: 2017-09-13 07:09:53
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

From : [Android Layout Tricks #2: Reusing layouts](https://android-developers.googleblog.com/2009/02/android-layout-tricks-1.html)

Android comes with a wide variety of widgets, small visual construction blocks you can glue together to present the users with complex and useful interfaces. However applications often need higher level visual components. A component can be seen as a complex widget made of several simple stock widgets. You could for instance reuse a panel containing a progress bar and a cancel button, a panel containing two buttons (positive and negative actions), a panel with an icon, a title and a description, etc. Creating new components can be done easily by writing a custom View but it can be done even more easily using only XML.

<!--more-->

In Android XML layout files, each tag is mapped to an actual class instance (the class is always a subclass of **View**.) The UI toolkit lets you also use three special tags that are not mapped to a View instance: **<requestFocus />**, **<merge />** and **<include />**. The latter, **<include />**, can be used to create pure XML visual components. (Note: I will present the **<merge />** tag in the next installment of Android Layout Tricks.)

The **<include />** does exactly what its name suggests; it includes another XML layout. Using this tag is straightforward as shown in the following example, taken straight from the source code of the Home application that currently ships with Android:

```Java
<com.android.launcher.Workspace
    android:id="@+id/workspace"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"

    launcher:defaultScreen="1">

    <include android:id="@+id/cell1" layout="@layout/workspace_screen" />
    <include android:id="@+id/cell2" layout="@layout/workspace_screen" />
    <include android:id="@+id/cell3" layout="@layout/workspace_screen" />

</com.android.launcher.Workspace>
```

In the **<include />** only the layout attribute is required. This attribute, without the android namespace prefix, is a reference to the layout file you wish to include. In this example, the same layout is included three times in a row. This tag also lets you override a few attributes of the included layout. The above example shows that you can use **android:id** to specify the id of the root view of the included layout; it will also override the id of the included layout if one is defined. Similarly, you can override all the layout parameters. This means that any android:layout_* attribute can be used with the **<include />** tag. Here is an example:

```java
<include android:layout_width="fill_parent" layout="@layout/image_holder" />
<include android:layout_width="256dip" layout="@layout/image_holder" />
```

This tag is particularly useful when you need to customize only part of your UI depending on the device's configuration. For instance, the main layout of your activity can be placed in the **layout/** directory and can include another layout which exists in two flavors, in **layout-land/** and **layout-port/**. This allows you to share most of the UI in portrait and landscape.

Like I mentioned earlier, my next post will explain the **<merge />**, which can be particularly powerful when combined with **<include />**.
