---
title: Accessibility-Are You Serving All Your Users?
date: 2017-02-21 22:52:00
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

[This post is by Joe Fernandez, a technical writer for developer.android.com who cares about accessibility and usability. — Tim Bray.]

We recently published some new resources to help developers make their Android applications more accessible:

  * [Accessibility Developer Guide](https://developer.android.com/guide/topics/ui/accessibility/index.html)

  * [Implementing Accessibility Training](https://developer.android.com/training/accessibility/index.html)

“But,” you may be thinking, “What is accessibility, exactly? Why should I make it a priority? How do I do it? And most importantly, how do I spell it?” All good questions. Let’s hit some of the key points.

<!--more-->

Accessibility is about making sure that Android users who have limited vision or other physical impairments can use your application just as well as all those folks in line at the supermarket checking email on their phones. It’s also about the Mom over in the produce section whose kids are driving her to distraction, and really needs to see that critical notification your application is trying to deliver. It’s also about you, in the future; Is your eyesight getting better over time? How about that hand-eye coordination?

When it comes down to it, making an application accessible is about having a deep commitment to usability, getting the details right and delighting your users. It also means stepping into new territory and getting a different perspective on your application. Try it out: Open up an application you developed (or your all-time favorite app), then close your eyes and try to complete a task. No peeking! A little challenging, right?

### How Android Enables Accessibility

One of main ways that Android enables accessibility is by allowing users to hear spoken feedback that announces the content of user interface components as they interact with applications. This spoken feedback is provided by an accessibility service called TalkBack, which is available for free on Google Play and has become a standard component of recent Android releases.

Now enable TalkBack, and try that eyes-closed experiment again. Being able to hear your application’s interface probably makes this experiment a little easier, but it’s still challenging. This type of interaction is how many folks with limited vision use their Android devices every day. The spoken feedback works because all the user interface components provided by the Android framework are built so they can provide descriptions of themselves to accessibility services like TalkBack.

Another key element of accessibility on Android devices is the ability to use alternative navigation. Many users prefer directional controllers such as D-pads, trackballs or keyboard arrows because it allows them to make discrete, predictable movements through a user interface. You can try out directional control with your apps using the virtual keyboard in the Android emulator or by installing and enabling the Eyes-Free Keyboard on your device. Android enables this type of navigation by default, but you, as a developer, may need to take a few steps to make sure users can effectively navigate your app this way.

### How to Make Your Application Accessible

It would be great to be able to give you a standard recipe for accessibility, but the truth of the matter is that the right answer depends on the design and functionality of your application. Here are some key steps for ensuring that your application is accessible:

  * **Task flows**: Design well-defined, clear task flows with minimal navigation steps, especially for major user tasks, and make sure those tasks are navigable via focus controls (see item 4).

  * **Action target size**: Make sure buttons and selectable areas are of sufficient size for users to easily touch them, especially for critical actions. How big? We recommend that touch targets be 48dp (roughly 9mm) or larger.

  * **Label user interface controls**: Label user interface components that do not have visible text, especially ImageButton, ImageView, and EditText components. Use the android:contentDescription XML layout attribute or setContentDescription() to provide this information for accessibility services.

  * **Enable focus-based navigation**: Make sure users can navigate your screen layouts using hardware-based or software directional controls (D-pads, trackballs and keyboards). In a few cases, you may need to make UI components focusable or change the focus order to be more logical.

  * **Use framework-provided controls**: Use Android's built-in user interface controls whenever possible, as these components provide accessibility support by default.

  * **Custom view controls**: If you build custom interface controls for your application, implement accessibility interfaces for your custom views and provide text labels for the controls.

  * **Test**: Checking off the items on this list doesn’t guarantee your app is accessible. Test accessibility by attempting to navigate your application using directional controls, and also try eyes free navigation with the TalkBack service enabled.

Here’s an example of implementing some basic accessibility features for an ImageButton inside an XML layout:

```java
<ImageButton
    android:id="@+id/add_note_button"
    android:src="@drawable/add_note_image"
    android:contentDescription="@string/add_note_description"/>

```

Notice that we’ve added a content description that accessibility services can use to provide an audible explanation of the button. Users can navigate to this button and activate it with directional controls, because ImageButton objects are focusable by default (so you don’t have to include the android:focusable="true" attribute).

The good news is that, in most cases, implementing accessibility isn’t about radically restructuring your application, but rather working through the subtle details of accessibility. Making sure your application is accessible is an opportunity to look at your app from a different perspective, improve the overall quality of your app and ensure that all your users have a great experience.

### References

[Accessibility: Are You Serving All Your Users?](https://android-developers.googleblog.com/2012/04/accessibility-are-you-serving-all-your.html)
