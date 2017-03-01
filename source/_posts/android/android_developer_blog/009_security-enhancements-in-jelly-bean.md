---
title: Security Enhancements in Jelly Bean
date: 2017-03-01 21:45:23
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

Posted by Fred Chung, Android Developer Relations team

Android 4.2, Jelly Bean, introduced quite a few new features, and under the covers it also added a number of security enhancements to ensure a more secure environment for users and developers.

This post highlights a few of the security enhancements in Android 4.2 that are especially important for developers to be aware of and understand. Regardless whether you are targeting your app to devices running Jelly Bean or to earlier versions of Android, it's a good idea to validate these areas in order to make your app more secure and robust.

### Content Provider default access has changed

Content providers are a facility to enable data sharing amongst app and system components. Access to content providers should always be based on the principle of least privilege — that is, only grant the minimal possible access for another component to carry out the necessary tasks. You can control access to your content providers through a combination of the exported attribute in the provider declaration and app-specific permissions for reading/writing data in the provider.

In the example below, the provider ReadOnlyDataContentProvider sets the exported attribute to "true", explicitly declaring that it is readable by any external app that has acquired the READ_DATA permission, and that no other components can write to it.

```java
<provider android:name=”com.example.ReadOnlyDataContentProvider”
    android:authorities=”com.example”
    android:exported=”true”
    android:readPermission=”com.example.permission.READ_DATA” />
```
<!--more-->

Since the exported attribute is an optional field, potential ambiguity arises when the field is not explicitly declared in the manifest, and that is where the behavior has changed in Android 4.2.

Prior to Jelly Bean, the default behavior of the exported field was that, if omitted, the content provider was assumed to be "exported" and accessible from other apps (subject to permissions). For example, the content provider below would be readable and writable by other apps (subject to permissions) when running on Android 4.1 or earlier. This default behavior is undesirable for sensitive data sources.

```java
<provider android:name=”com.example.ReadOnlyDataContentProvider”
    android:authorities=”com.example” />
```

Starting in Android 4.2, the default behavior for the same provider is now “not exported”, which prevents the possibility of inadvertent data sharing when the attribute is not declared. If either the minSdkVersion or targetSdkVersion of your app is set to 17 or higher, the content provider will no longer be accessible by other apps by default.

While this change helps to avoid inadvertent data sharing, it remains the best practice to always explicitly declare the exported attribute, as well as declaring proper permissions, to avoid confusion. In addition, we strongly encourage you to make use of Android Lint, which among other things will flag any exported content providers (implicit or explicit) that aren't protected by any permissions.

### New implementation of SecureRandom

Android 4.2 includes a new default implementation of SecureRandom based on OpenSSL. In the older Bouncy Castle-based implementation, given a known seed, SecureRandom could technically (albeit incorrectly) be treated as a source of deterministic data. With the new OpenSSL-based implementation, this is no longer possible.

In general, the switch to the new SecureRandom implementation should be transparent to apps. However, if your app is relying on SecureRandom to generate deterministic data, such as keys for encrypting data, you may need to modify this area of your app. For example, if you have been using SecureRandom to retrieve keys for encrypting/decrypting content, you will need to find another means of doing that.

A recommended approach is to generate a truly random AES key upon first launch and store that key in internal storage. For more information, see the post "Using Cryptography to Store Credentials Safely".

### JavascriptInterface methods in WebViews must now be annotated

Javascript hosted in a WebView can directly invoke methods in an app through a JavaScript interface. In Android 4.1 and earlier, you could enable this by passing an object to the addJavascriptInterface() method and ensuring that the object methods intended to be accessible from JavaScript were public.

On the one hand, this was a flexible mechanism; on the other hand, any untrusted content hosted in a WebView could potentially use reflection to figure out the public methods within the JavascriptInterface object and could then make use of them.

Beginning in Android 4.2, you will now have to explicitly annotate public methods with @JavascriptInterface in order to make them accessible from hosted JavaScript. Note that this also only takes effect only if you have set your app's minSdkVersion or targetSdkVersion to 17 or higher.

```java
// Annotation is needed for SDK version 17 or above.
@JavascriptInterface
public void doSomething(String input) {
   . . .
}
```

### Secure USB debugging

Android 4.2.2 introduces a new way of protecting your apps and data on compatible devices — secure USB debugging. When enabled on a device, secure debugging ensures that only host computers authorized by the user can access the internals of a USB-connected device using the ADB tool included in the Android SDK.

Secure debugging is an extension of the ADB protocol that requires hosts to authenticate before accessing any ADB services or commands. At first launch, ADB generates an RSA key pair to uniquely identifies the host. Then, when you connect a device that requires secure debugging, the system displays an authorization dialog such as the one shown below.

![](/images/categories/android/android-developer-blog/009/adb-crop-new.png)

The user can allow USB debugging for the host for a single session or can give automatic access for all future sessions. Once a host is authorized, you can execute ADB commands for the device in the normal way. Until the device is authorized, it remains in "offline" state, as listed in the adb devices command.

For developers, the change to USB debugging should be largely transparent. If you've updated your SDK environment to include ADB version 1.0.31 (available with SDK Platform-tools r16.0.1 and higher), all you need to do is connect and authorize your device(s). If your development device appears in "offline" state, you may need to update ADB. To so so, download the latest Platform Tools release through the SDK Manager.

Secure USB debugging is enabled in the Android 4.2.2 update that is now rolling out to Nexus devices across the world. We expect many more devices to enable secure debugging in the months ahead.

### More information about security best practices

For a full list of security best practices for Android apps, make sure to take a look at the [Security Tips](https://developer.android.com/training/articles/security-tips.html) document.

### References

[Security Enhancements in Jelly Bean](https://android-developers.googleblog.com/2013/02/security-enhancements-in-jelly-bean.html)
