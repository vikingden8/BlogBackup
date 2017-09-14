---
title: Android Main Content Under Toolbar
date: 2017-9-14 20:09:36
tags:
categories: "Android学习笔记"
---

Many ViewGroups allow their children to overlap. These include FrameLayout, RelativeLayout, CoordinatorLayout, and DrawerLayout. One that does not allow its children to overlap is LinearLayout.

The answer to your question really depends on what the end result should be. If you are trying to just have a Toolbar that is always on screen and some content below it, then you don't need a CoordinatorLayout and AppBarLayout at all, you can just use a vertical LinearLayout with two children:

<!--more-->

```java
<LinearLayout android:orientation="vertical" ...>
    <android.support.v7.widget.Toolbar
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        ... />

    <FrameLayout 
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        ... />
</LinearLayout>
```

Note layout attributes of the FrameLayout.

If you want to do the fancy stuff where the toolbar scrolls on and off the screen as you scroll the content, then you need an AppBarLayout and you need to give your content area a special attribute like this:

```java
<android.support.design.widget.CoordinatorLayout>
    <android.support.design.widget.AppBarLayout
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       ... >

        <android.support.v7.widget.Toolbar
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:layout_scrollFlags="scroll"
            ... />
    </android.support.design.widget.AppBarLayout>

    <FrameLayout 
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        ... />
</android.support.design.widget.CoordinatorLayout>
```

From:[Display content under toolbar](https://stackoverflow.com/questions/36826829/display-content-under-toolbar)