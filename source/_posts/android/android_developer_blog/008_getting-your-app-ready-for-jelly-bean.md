---
title: Getting Your App Ready for Jelly Bean and Nexus 7
date: 2017-02-21 23:20:28
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

[This post is by Nick Butcher, an Android engineer who notices small imperfections, and they annoy him.]

We are pleased to announce that the full SDK for Android 4.1 is now available to developers and can be downloaded through your SDK Manager. You can now develop and publish applications against API level 16 using new Jelly Bean APIs. We are also releasing SDK Tools revision 20.0.1 and NDK revision 8b containing bug fixes only.

For many people, their first taste of Jelly Bean will be on the beautiful Nexus 7. While most applications will run just fine on Nexus 7, who wants their app to be just fine? Here are some tips for optimizing your application to make the most of this device.

### Screen

Giving Nexus 7 its name, is the glorious 7” screen. As developers we see this as around 600 * 960 density independent pixels and a density of tvdpi. As Dianne Hackborn has elaborated, this density might be a surprise to you but don’t panic! We actively discourage you from rushing out and creating new assets at this density; Android will scale your existing assets for you. In fact the entire Jelly Bean OS contains only a single tvdpi asset, the remainder are scaled down from hdpi assets.

<!--more-->

To be sure the system can successfully scale your hdpi assets for tvdpi, take special care that your 9-patch images are created correctly so that they can be scaled down effectively:

  * Make sure that any stretchable regions are at least 2x2 pixels in size, else they risk disappearing when scaled down.

  * Give one pixel of extra safe space in the graphics before and after stretchable regions else interpolation during scaling may cause the color at the boundaries to change.

The 7” form factor gives you more space to present your content. You can use the sw600dp resource qualifier to provide alternative layouts for this size screen. For example your application may contain a layout for your launch activity:

```sh
res/layout/activity_home.xml
```

To take advantage of the extra space on the 7” screen you might provide an alternative layout:

```sh
res/layout-sw600dp/activity_home.xml
```

The sw600dp qualifier declares that these resources are for devices that have a screen with at least 600dp available on its smallest side.

Furthermore you might even provide a different layout for 10” tablets:

```sh
res/layout-sw720dp/activity_home.xml
```

This technique allows a single application to use defined switching points to respond to a device’s configuration and present an optimized layout of your content.

Similarly if you find that your phone layout works well on a 7” screen but requires slightly larger font or image sizes then you can use a single layout but specify alternative sizes in dimensions files. For example res/values/dimens.xml may contain a font size dimension:

```sh
<dimen name="text_size">18sp</dimen>
```

but you can specify an alternative text size for 7” tablets in res/values-sw600dp/dimens.xml:

```sh
<dimen name="text_size">32sp</dimen>
```

### Hardware

Nexus 7 has different hardware features from most phones:

  * No telephony

  * A single front facing camera (apps requiring the android.hardware.camera feature will not be available on Nexus 7)

Be aware of which system features that you declare (or imply) are required to run your application or the Play Store will not make your application available to Nexus 7 users. Always declare hardware features that aren't critical to your app as required="false" then detect at runtime if the feature is present and progressively enhance functionality. For example if your app can use the camera but it isn’t essential to its operation, you would declare it with:

```sh
<uses-feature android:name="android.hardware.camera"
              android:required="false"/>
```

For more details follow Reto Meier’s [Five Steps to Hardware Happiness](http://android-developers.blogspot.co.uk/2010/10/five-steps-to-future-hardware-happiness.html).

### Conclusion

Nexus 7 ships with Jelly Bean and its updated suite of system apps are built to take advantage of new Jelly Bean APIs. These apps are the standard against which your application will be judged — make sure that you’re keeping up!

If your application shows notifications then consider utilizing the new richer notification styles. If you are displaying immersive content then control the appearance of the system UI. If you are still using the options menu then move to the Action Bar pattern.

A lot of work has gone into making Jelly Bean buttery smooth; make sure your app is as well. If you haven’t yet opted in to hardware accelerated rendering then now is the time to implement and test this.

For more information on Android 4.1 visit the Android Developers site or join us Live.

### References

[Getting Your App Ready for Jelly Bean and Nexus 7](https://android-developers.googleblog.com/2012/07/getting-your-app-ready-for-jelly-bean.html)
