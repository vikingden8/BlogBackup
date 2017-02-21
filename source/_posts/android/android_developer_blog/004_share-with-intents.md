---
title: Share With Intents
date: 2017-02-21 22:14:56
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

[This post is by Alexander Lucas, an Android Developer Advocate bent on saving the world 5 minutes. —Tim Bray]

Intents are awesome. They are my favorite feature of Android development. They make all sorts of stuff easier. Want to scan a barcode? In the olden platforms, if you were lucky, this involved time and effort finding and comparing barcode-scanning libraries that handled as much as possible of camera interaction, image processing, an internal database of barcode formats, and UI cues to the user of what was going on. If you weren’t lucky, it was a few months of research & haphazard coding to figure out how to do that yourself.

On Android, it’s a declaration to the system that you would like to scan a barcode.

```java
public void scanSomething() {
    // I need things done!  Do I have any volunteers?
    Intent intent = new Intent("com.google.zxing.client.android.SCAN");

    // This flag clears the called app from the activity stack, so users arrive in the expected
    // place next time this application is restarted.
    intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);

    intent.putExtra("SCAN_MODE", "QR_CODE_MODE");
    startActivityForResult(intent, 0);
}
...
public void onActivityResult(int requestCode, int resultCode, Intent intent) {
    if (requestCode == 0) {
        if (resultCode == RESULT_OK) {
            //  The Intents Fairy has delivered us some data!
            String contents = intent.getStringExtra("SCAN_RESULT");
            String format = intent.getStringExtra("SCAN_RESULT_FORMAT");
            // Handle successful scan
        } else if (resultCode == RESULT_CANCELED) {
            // Handle cancel
        }
    }
}
```
<!--more-->
See that? That’s nothing. That’s 5 minutes of coding, 3 of which were just to look up the name of the result you wanted to pull. And that was made possible because the Barcode Scanner application is designed to be able to scan barcodes for whatever other applications may need it.

More important, our app is completely decoupled from the BarcodeScanner app. There’s no integration- in fact, neither application is checking to verify that the other exists. If the user preferred, they could remove “Barcode Scanner” and replace it with a competing app. As long as that app supported the same intent, functionality would remain the same. This decoupling is important. It’s the easy way. It’s the lazy way. It’s the Android way.

### Sharing Data Using Intents

One of the most inherently useful Android intents is the Share intent. You can let the user share data to any service they want, without writing the sharing code yourself, simply by creating a share intent.

```java
Intent intent=new Intent(android.content.Intent.ACTION_SEND);
intent.setType("text/plain");
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);

// Add data to the intent, the receiving app will decide what to do with it.
intent.putExtra(Intent.EXTRA_SUBJECT, “Some Subject Line”);
intent.putExtra(Intent.EXTRA_TEXT, “Body of the message, woot!”);
```

... and starting it with a chooser:

```java
startActivity(Intent.createChooser(intent, “How do you want to share?”));
```

With these 5 lines of code, you get to bypass authenticating, credential storage/management, web API interaction via http posts, all sorts of things. Where by “bypass”, I mean “have something else take care of.” Like the barcode scanning intent, all you really had to do was declare that you have something you’d like to share, and let the user choose from a list of takers. You’re not limited to sending text, either. Here’s how you’d create an intent to share an image:

```java
private Intent createShareIntent() {
    ...
    Intent shareIntent = new Intent(Intent.ACTION_SEND);
    shareIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);
    shareIntent.setType("image/*");

    // For a file in shared storage.  For data in private storage, use a ContentProvider.
    Uri uri = Uri.fromFile(getFileStreamPath(pathToImage));
    shareIntent.putExtra(Intent.EXTRA_STREAM, uri);
    return shareIntent;
}  
```
Note that just by using setType() to set a MIME type, you’ve filtered down the list of apps to those that will know what to do with an image file.

### Intents over Integration

Think about this for a second. By making the simple assumption that any user of any service (Task Manager, Social Network, Photo sharing site) already has some app on their phone that can share to that service, you can leverage the code that they’ve already written. This has several awesome implications:

  * **Less UI** — You don’t have to clog up your UI with customized, clickable badges of services you support. Just add a “share” button. It’s okay, we’ve made sure all your users know what it does [insert smiley here].

  * **Leveraged UI** — You can bet that every high-quality web service out there has spent serious time on the UI of their Android app’s “share” activity. Don’t reinvent the wheel! Just grab a couple and go for a ride.

  * **Filtered for the user** — If I don’t have a Foo-posting app on my phone, there’s a good chance I don’t care about posting to Foo. Now I won’t see Foo icons everywhere that are useless to me.

  * **Client App Ecosystem** — Much like an email client, anyone can write a client for any service. Users will use the ones they want, uninstall the ones they don’t. Your app supports them all.

  * **Forward Compatible with new services** — If some swanky new service springs up out of nowhere with an Android Application, as long as that application knows how to receive the share intent, you already support it. You don’t spend time in meetings discussing whether or not to wedge support for the new service into your impending Next Release(tm), you don’t burn engineering resources on implementing support as fast as possible, you don’t even upload a new version of anything to Android Market. Above all, you don’t do any of that again next week, when another new service launches and the whole process threatens to repeat itself. You just hang back and let your users download an application that makes yours even more useful.

### Avoid One-Off Integrations

For each pro of the Intent approach, integrating support to post to these services one-at-a-time has a corresponding con.

  * **Bad for UI** — If your photo-sharing app has a Foo icon, what you might not immediately understand is that while you’re trying to tell the user “We post to Foo!” what you’re really saying is “We don’t post to Bar, Baz, or let you send the photo over email, sms, or bluetooth. If we did, there would be icons. In fact, we probably axed those features because of the space the icons would take on our home screen. Oh, and we’ll probably use some weird custom UI and make you authenticate through a browser, instead of the Foo client you already have installed.” I’m not going to name names, but a lot of you are guilty of this. It’s time to stop. (I mean it. Stop.)

  * **Potentially wasted effort** — Let’s say you chose one service, and integrated it into your UI perfectly. Through weeks of back-and-forth with Foo’s staff, you’ve got the API and authentication mechanisms down pat, the flow is seamless, everything’s great. Only problem is that you just wasted all that effort, because none of your user-base particularly cares for Foo. Ouch!

  * **Not forward compatible for existing services** — Any breaking changes in the API are your responsibility to fix, quickly, and that fix won’t be active until your users download a newer version of your app.

  * **Won’t detect new services** — This one really hurts. If a brand new service Baz comes out, and you’ve actually got the engineering cycles to burn, you need to get the SDK, work out the bugs, develop a sharing UI, have an artist draw up your own “edgy” (ugh) but legally distinct version of the app’s logo so you can plaster it on your home screen, work out all the bugs, and launch.

You will be judged harshly by your users. And deservedly so.

### Ice Cream Sandwich makes it even easier

With the release of ICS, a useful tool for sharing called ShareActionProvider was added to the framework, making the sharing of data across Android applications even easier. ShareActionProviders let you populate lists of custom views representing ACTION_SEND targets, facilitating (for instance) adding a “share” menu to the ActionBar, and connecting it to whatever data the user might want to send.

Doing so is pretty easy. Configure the menu items in your Activity’s onCreateOptionsMenu method, like so:

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Get the menu item.
    MenuItem menuItem = menu.findItem(R.id.menu_share);
    // Get the provider and hold onto it to set/change the share intent.
    mShareActionProvider = (ShareActionProvider) menuItem.getActionProvider();

    // Attach an intent to this ShareActionProvider.  You can update this at any time,
    // like when the user selects a new piece of data they might like to share.
    mShareActionProvider.setShareIntent(yourCreateShareIntentMethod());

    // This line chooses a custom shared history xml file. Omit the line if using
    // the default share history file is desired.
    mShareActionProvider.setShareHistoryFileName("custom_share_history.xml");
      . . .
}
```

Note that you can specify a history file, which will adapt the ordering of share targets based on past user choices. One shared history file can be used throughout an application, or different history files can be used within the same application, if you want to use a separate history based on what kind of data the user wants to share. In the above example, a custom history file is used. If you wish to use the default history for the application, you can omit that line entirely.

This will help optimize for an important feature of the ShareActionProvider: The user’s most common ways to share float to the top of the drop-down, with the least used ones disappearing below the fold of the “See More” button. The most commonly selected app will even become a shortcut right next to the dropdown, for easy one-click access!

You’ll also need to define a custom menu item in XML. Here’s an example from the ActionBar Dev Guide.

```java
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_share"
          android:title="@string/share"
          android:showAsAction="ifRoom"
          android:actionProviderClass="android.widget.ShareActionProvider" />
</menu>
```

And with that, you can have an easy sharing dropdown that will look like the screenshot here. Note that you get the nice standard three-dots-two-lines “Share” glyph for free.

### Remember: Smart and Easy

The share intent is the preferred method of sharing throughout the Android ecosystem. It’s how you share images from Gallery, links from the browser, and apps from Android Market. Intents are the easiest path to writing flexible applications that can participate in a rapidly expanding ecosystem, but they’re also the smart path to writing applications that will stay relevant to your users, letting them share their data to any service they want, no matter how often their preferences change over time. So take a step back and stop worrying about if your user wants to tweet, digg, post, email, im, mms, bluetooth, NFC, foo, bar or baz something. Just remember that they want to share it. Android can take it from there.

### References

[Share with Intents](https://android-developers.googleblog.com/2012/02/share-with-intents.html)
