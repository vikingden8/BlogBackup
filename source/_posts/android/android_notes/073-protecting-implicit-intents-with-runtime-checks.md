---
title: Protecting Implicit Intents with Runtime Checks
date: 2017-07-19 23:39:50
tags:
categories: "Android学习笔记"
---

Implicit intents (https://developer.android.com/guide/components/intents-filters.html#ExampleSend) could cause an _ActivityNotFoundException_ if there’s no activity to handle the Intent you’ve created, whether it is from the user uninstalling or disabling the other apps, restricted profiles, or device administrator policies. Make sure you protect your implicit intents with a simple runtime check.

**Implicit intents [0]** are great for when you want to start an activity for an action, such as show a map, capture a photo, or share some content. But the new **Restricted Profiles [1]** feature available with Android 4.3 introduces risk for such intents because **it’s possible that a restricted profile does not have access to some of the apps that you’ve previously assumed are always available**, such as web browsers, email or SMS apps, or cameras.

<!--more-->

Before you call _startActivity()_, you need to be certain that an app is available to handle the intent. You can check whether there’s an app to handle your intent by calling _PackageManager.resolveActivity() [2]_. As long as the return value is not null, it’s safe to start the intent. If _resolveActivity()_ does return null, you should display a message to the user explaining why the action cannot be performed. For example:

```java
// Create a “share” intent with a photo with a text message
Intent intent = new Intent();
intent.setAction(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_STREAM, uriForPhoto);
...

// Verify that the intent will resolve to an activity
if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
} else {
    Toast.makeText(context, R.string.app_not_available, Toast.LENGTH_LONG).show();
}
```

Better yet, **you should run this check when your activity starts so you can disable the feature completely** instead of failing when the user attempts to perform the action.

[0] Implicit intents: http://developer.android.com/guide/components/intents-filters.html#ires
[1] Restricted profiles: http://developer.android.com/about/versions/android-4.3.html#RestrictedProfiles
[2] resolveActivity: http://goo.gl/msJUfZ﻿


**References**

[Protecting Implicit Intents with Runtime Checks (Android Development Patterns Ep 1)](https://www.youtube.com/watch?v=HGElAW224dE&list=PLRMikyO0Ly72nbjk6ssHYPcnfgiKAIVVB&index=10)

[https://plus.google.com/+AndroidDevelopers/posts/JUGR3bXGDoM](https://plus.google.com/+AndroidDevelopers/posts/JUGR3bXGDoM)
