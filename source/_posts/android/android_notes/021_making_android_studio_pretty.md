---
title: Making Android Studio Pretty
date: 2017-02-25 10:05:24
tags:
categories: "Android学习笔记"
---

Today, during a glorious transition from 1.5 rc1 to 1.5 Android Studio decided to keep all of my settings completely intact, with the honorable exception of GUI and syntax themes and LogCat colors, which were just gone. Being me, I had not the slightest recollection on how the previous combo have gotten into my editor and I had to search for everything again. Not particularly enjoying the process of rediscovery, I decided to keep it stashed somewhere, both for the future me and maybe for some curious souls that happen to bump onto my blog.

### Final appearance

![](/images/categories/android/android_notes/021/prettyAS-2.png)

### GUI Theme

 manual from [here](https://github.com/ChrisRM/material-theme-jetbrains#installation)

  * [Open the Settings/Preferences](https://www.jetbrains.com/help/idea/2016.3/accessing-settings.html#openIdeSettings) dialog (OSX/Unix: ⌘+,, Windows: Ctrl+Alt+S)

  * In the left-hand pane, select **Plugins**.

  * Click **Browse repositories…** and search for Material Theme UI

  * Click **Install plugin** and confirm your intention to download and install the plugin.

  * Click **OK** in the **Settings** dialog and restart for the changes to take effect.

<!--more-->

### Editor Schema

manual from [here](https://github.com/ciscorucinski/ChroMATERIAL#installation)

#### Find plugin

Open IDE and locate **File** >> **Settings** >> **Plugins** and click **Browse Repositories…**
Search for and click ChroMATERIAL and click **Install plugin**

#### Use Color Scheme

Locate **File** >> **Settings** >> **Editor** >> **Colors & Fonts** >> **Scheme**
Choose **ChroMATERIAL** and click **Apply/OK**.

###  HOLO Logcat
<blockquote>
<table>
      <thead>
        <tr>
          <th style="text-align: right">Type</th>
          <th style="text-align: left">Color</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td style="text-align: right">verbose:</td>
          <td style="text-align: left"><kbd><b style="color:#BBB">#BBB</b></kbd></td>
        </tr>
        <tr>
          <td style="text-align: right">debug:</td>
          <td style="text-align: left"><kbd><b style="color:#33B5E5">#33B5E5</b></kbd></td>
        </tr>
        <tr>
          <td style="text-align: right">info:</td>
          <td style="text-align: left"><kbd><b style="color:#9C0">#9C0</b></kbd></td>
        </tr>
        <tr>
          <td style="text-align: right">assert:</td>
          <td style="text-align: left"><kbd><b style="color:#A6C">#A6C</b></kbd></td>
        </tr>
        <tr>
          <td style="text-align: right">error:</td>
          <td style="text-align: left"><kbd><b style="color:#F44">#F44</b></kbd></td>
        </tr>
        <tr>
          <td style="text-align: right">warning:</td>
          <td style="text-align: left"><kbd><b style="color:#FB3">#FB3</b></kbd></td>
        </tr>
      </tbody>
</table>
</blockquote>

* Go to **Preferences** → **Editor** → **Colors & Fonts** → **Android Logcat**,

* Make sure that ChroMATERIAL is selected in dropdown, and click **Save as…**,

* Choose a name ex. ChroMATERIAL + HOLO and confirm with **OK**,

* Click on each item from the list in the center and change their **Foreground** color to the one from table above,

* Click **Apply/OK**.

### Screenshots

![](/images/categories/android/android_notes/021/prettyAS-1-small.png)

![](/images/categories/android/android_notes/021/prettyAS-2-small.png)

![](/images/categories/android/android_notes/021/prettyAS-3-small.png)

![](/images/categories/android/android_notes/021/prettyAS-4-small.png)

### References

[Making Android Studio pretty](https://meedamian.com/post/deuglifying-android-studio/?hi)
