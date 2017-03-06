---
title: Material Design Splash Screens
date: 2017-03-06 21:52:53
tags:
categories: "Android学习笔记"
---

>From google material design [documentation](https://material.io/guidelines/patterns/launch-screens.html).

>The launch screen is a user’s first experience of your application.

>Because launching your app while displaying a blank canvas increases its perceived loading time, consider using a placeholder UI or a branded launch screen.

### How to add?

I. Declare custom <font color="#f50000">drawable.xml</font> file with <font color="#f50000">items</font> for launch screen background.

```java
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">  
    <item
        android:drawable="@color/blue"/>

    <item>
        <bitmap
            android:gravity="center"
            android:src="@drawable/logo"/>
    </item>
</layer-list>  
```

<!--more-->

II. Declare custom style in your styles.xml using the new drawable as background.

```java
<style name="SplashTheme" parent="Theme.AppCompat.NoActionBar">  
    <item name="android:windowBackground">@drawable/background_splash</item>
</style>
```

>If your API Level is greater than v19, you can make Status Bar and Navigation Bar translucent setting attributes <font color="#f50000">android:windowTranslucentStatus</font> and <font color="#f50000">android:windowTranslucentNavigation</font> to <font color="#f50000">true</font>.


III. Apply this style to your splash activity via <font color="#f50000">android:theme</font> attribute in your <font color="#f50000">AndroidManifest.xml</font> file.

```java
<activity  
    android:name=".SplashActivity"
    android:theme="@style/SplashTheme">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>  
```
