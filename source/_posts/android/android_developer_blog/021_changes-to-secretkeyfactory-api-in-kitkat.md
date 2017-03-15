---
title: Changes to the SecretKeyFactory API in Android 4.4
date: 2017-03-15 22:25:03
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

Posted by Trevor Johns, Android Developer Relations team

random_droid
In order to encrypt data, you need two things: some data to encrypt and an encryption key. The encryption key is typically a 128- or 256-bit integer. However, most people would rather use a short passphrase instead of a remembering a 78-digit number, so Android provides a way to generate an encryption key from ASCII text inside of javax.crypto.SecretKeyFactory.

Beginning with Android 4.4 KitKat, we’ve made a subtle change to the behavior of SecretKeyFactory. This change may break some applications that use symmetric encryption and meet all of the following conditions:

  * Use SecretKeyFactory to generate symmetric keys, and

  * Use PBKDF2WithHmacSHA1 as their key generation algorithm for SecretKeyFactory, and

  * Allow Unicode input for passphrases

Specifically, PBKDF2WithHmacSHA1 only looks at the lower 8 bits of Java characters in passphrases on devices running Android 4.3 or below. Beginning with Android 4.4, we have changed this implementation to use all available bits in Unicode characters, in compliance with recommendations in PCKS #5.

Users using only ASCII characters in passphrases will see no difference. However, passphrases using higher-order Unicode characters will result in a different key being generated on devices running Android 4.4 and later.

For backward compatibility, we have added a new key generation algorithm which preserves the old behavior: PBKDF2WithHmacSHA1And8bit. Applications that need to preserve compatibility with older platform versions (pre API 19) and meet the conditions above can make use of this code:

```java
import android.os.Build;

SecretKeyFactory factory;
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
   // Use compatibility key factory -- only uses lower 8-bits of passphrase chars
   factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1And8bit");
} else {
   // Traditional key factory. Will use lower 8-bits of passphrase chars on
   // older Android versions (API level 18 and lower) and all available bits
   // on KitKat and newer (API level 19 and higher).
   factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
}
```
