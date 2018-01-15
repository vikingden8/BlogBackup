---
title: Secure Against Screenshots
date: 2018-01-15 09:41:36
tags:
categories: "Android学习笔记"
---

From:[Secure Against Screenshots](https://commonsware.com/blog/2012/01/16/secure-against-screenshots.html)

Ice Cream Sandwich now allows the user to capture screenshots of the contents of the device screen, by simultaneously pressing the Power button and the Volume Down button.

从ICS开始，Android允许用户通过同时按住电源键和音量减键来截取设备的屏幕内容．

(which, in practice, seems to be a lot harder than you might think…)

(在实际的操作过程中可能会有一点点麻烦．．．)

This has been an oft-requested feature, so it will make many users happy.

这个功能让很多的用户点赞，因为这个功能一直被要求提供．

Note that this is not available via the SDK, so malware should not be able to screen-capture arbitrary stuff in your app. However, for the user’s benefit, there may be reasons to block screenshots from certain of your activities.

需要注意的是，这个功能并不在SDK中提供，所以恶意软件不可能在你的应用中任意的截取屏幕内容．然而，为了用户的利益，在你的应用界面中可能会有不同的原因来阻止被截取屏幕内容．

<!--more-->

To do that, use **FLAG_SECURE**:

可以使用 **FLAG_SECURE** 来实现：

```java
public class FlagSecureTestActivity extends Activity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    getWindow().setFlags(LayoutParams.FLAG_SECURE,
                         LayoutParams.FLAG_SECURE);

    setContentView(R.layout.main);
  }
}
```

Normally, when a user takes a screenshot, a **Notification** will be raised to indicate success and provide access to the screenshot in the Gallery app. With **FLAG_SECURE** in place, the Notification will be “Couldn’t capture screenshot”.

正常来说，当用户截取屏幕时，成功会有一个通知显示并可以点击通知跳到Gallery应用中查看．但是，当你的应用界面中使用了　**FLAG_SECURE**，在截取屏幕时通知栏会有类似＂不能截取屏幕＂的提示．
