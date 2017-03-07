---
title: I don't need your permission
date: 2017-3-7 09:35:47
tags:
categories: "Android学习笔记"
---

Permissions on Android are tricky to get right from a user perspective. Usually you only want to do something minor and innocuous (pre-fill a form with a contact's info) but the actual permission you have to request gives you much more power than necessary (access to ALL contact details, ever).

It's understandable that users might be suspicious of you. If your app is closed-source then they have no way of verifying you're not downloading all their contacts to their servers. Even if you explain the permission request people may not trust you. In the past I've chosen not to implement what might be handy features just to avoid user distrust.

That said, one thing that bothers me is that **you don't always have to ask for permission to do some actions**.

Case in point: _android.permission.CALL_PHONE_. You need it to initiate phone calls from your app, right? Isn't this how you make calls?

```java
Intent intent = new Intent(Intent.ACTION_CALL);
intent.setData(Uri.parse("tel:1234567890"))
startActivity(intent);
```

Wrong! The reason you need permission is because with this code you can initiate a phone call at any time without user action! If my app had this permission I could be calling 1-900-CAT-FACTS at 3 AM every morning and you'd be none the wiser1.

The more correct way to do it is with either _ACTION_VIEW_ or _ACTION_DIAL_:

```java
Intent intent = new Intent(Intent.ACTION_DIAL);
intent.setData(Uri.parse("tel:1234567890"))
startActivity(intent);
```

<!--more-->

**The beauty of this solution is that you don't need permission**. The reason you don't is because instead of initiating a call, it launches the Dialer app with the phone number pre-filled, but still requires the user to click "dial" to initiate the call. Honestly, this feels like a better system anyways; I don't want apps to surprise me.

Another example: I wrote Quick Map for my wife when she complained that it was taking too many clicks to get navigation started on her phone. All she wanted was a list of her contacts and a way to navigate to them.

You might think that I'd need permissions for all her contacts in order to write the app: wrong again! If you look at the source code, you'll see that ACTION_PICK is used to launch another app in order to pick an address:

```java
Intent intent = new Intent(Intent.ACTION_PICK);
intent.setType(StructuredPostal.CONTENT_TYPE);
startActivityForResult(intent, 1);
```

Not only does this mean no permission request, it also means my app doesn't need any UI. It completely leverages the UX of the app you're picking the contacts from.

One of the coolest parts of Android is the Intent system because it means I don't have to write everything myself. Other apps can register themselves as experts in handling specific pieces of data, like phone numbers, text messages, or contacts. If I had to do it all myself I'd have a buggy app full of half-assed implementations because it'd be so much work.

One other advantage of this system is that I can leverage these other apps' permissions without having to ask for them on my own. That's essentially all that's happening above. While a dialer app does need the permission to be able to make calls, my Intent to open the dialer does not2. The user trusts the dialer to make calls, but not my app, which is fine; they probably would rather use the dedicated dialer app anyways!

The moral of the story is that **before you think about adding a permission, you should at least peruse the Intent documentation to see if there is a way to request another app to do it for you**. If you really want to get into the weeds, you can study the more intricate details of the permission system, which includes ways to do more fine-grained permissions.

Using fewer permissions will not only get you more user trust due to fewer permissions, but the user will have a better experience because they will get to use their preferred apps as well.

From:[I don't need your permission](http://blog.danlew.net/2014/11/26/i-dont-need-your-permission/)

More Reading:[Permissions Usage Notes](https://developer.android.com/training/permissions/usage-notes.html)
