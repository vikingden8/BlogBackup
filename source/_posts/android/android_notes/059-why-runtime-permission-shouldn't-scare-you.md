---
title: Why runtime permissions shouldn’t scare you
date: 2017-07-23 23:44:58
tags:
categories: "Android学习笔记"
---

Marshmallow introduced Runtime Permissions, and that seems to be all anyone is talking about this summer. But that’s because prompting for a long list of permissions at install can intimidate users, and that’s not good for anyone. So apps targeting Marshmallow now have to put these permission requests into a context, by asking for them when the user is trying to use the relevant feature.

As great as this is, I’m sure that you’ve thought to yourself “that’s okay, I just won’t update to Marshmallow until I have to.” Updating your app isn’t as daunting as you might think, though. You simply need add a few lines of code that build in checks and graceful failures. So here’s a handy guide to walk you through it.

<!--more-->

**Step 1: check the platform**. If the device is running Lollipop or earlier, then the user granted permission at install time, and you’re good to go. But if the device is running Marshmallow, you can’t be so certain. Clever developers can use the support library, though, which will do this check for you. That’s one less line of code for you to add.

Which brings us to **step 2: check the permission status**. A simple call to _checkSelfPermission()_ (http://goo.gl/T7vE7b) will let you know if the permission is currently granted. The only scary thing here is that you can’t rely on assumptions here, because even if the user granted the permission in the past, they may have revoked it later on. So with one conditional statement, you’ve completed step 2.

If you don’t have permission, you may need **step 3: explain the permission**. In some instances, you’ll want to update the UI to clarify what that permission enables and why the feature needs it. This can be as simple as a toast or as complex as the fanciest layout. The cool thing here is that you don’t need to figure out what those moments are. A call to _shouldShowRequestPermissionRationale()_ (http://goo.gl/bFyfVj) will indicate whether this is one of those clarifying moments. Easy enough, right?

And now, the heart of it-- **step 4: request the permission**. The _requestPermissions()_ (http://goo.gl/yNuizg) method will prompt a dialog to the user to get their answer and then trigger your _onRequestPermissionResult()_ callback to handle the response. This is only two lines of code to add. One to make the call, and one to declare the request code, which is indicative of where the user is in your app and what they are trying to do.

Finally, **step 5: handle the response**. Here is your biggest change, and all it is is overwriting your callback with a switch statement based on that request code. Your request code will help you restore the app to the right state if the permission has been granted. If the user rejected the request, though, you’ll need to update the UI to disable the feature or indicate that it won’t be available without the permission.

So, you see, it isn’t so scary to add support for runtime permissions. Take a crack at it and  [#BuildBetterApps](https://plus.google.com/s/%23BuildBetterApps) ! And if you want some more context and implementation guidelines, check out the blog post from last week (http://goo.gl/JMnKQw).﻿

![](/images/categories/android/android_notes/059/runtime-permission-request-flow.webp)




From：[Why runtime permissions shouldn’t scare you](https://plus.google.com/+AndroidDevelopers/posts/FqgHUevqHiK)
