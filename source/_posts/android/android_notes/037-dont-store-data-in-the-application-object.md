---
title: Don't Store Data in the Application Object
date: 2017-04-08 22:39:20
tags:
categories: "Android学习笔记"
---

There is always some information that is needed in many places in your app. It can be a session token, the result of an expensive computation, etc. It is often tempting to avoid the overhead of passing objects between activities or keeping those in persistent storage.

A pattern that is sometimes suggested is to dump your data in the Application object with the idea that it will be available across all activities. This solution is simple, elegant and… totally wrong.

If you assume that your data will stay there, your application will eventually crash with a NullPointerException.

### A Simple Test Case

The Application object:

```Java
// access modifiers omitted for brevity
class MyApplication extends Application {

    String name;

    String getName() {
        return name;
    }

    void setName(String name) {
        this.name = name;
    }
}
```

<!--more-->

The first activity, where we store the name of the user in the application object:

```Java
// access modifiers omitted for brevity
class WhatIsYourNameActivity extends Activity {

    void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.writing);

        // Just assume that in the real app we would really ask it!
        MyApplication app = (MyApplication) getApplication();
        app.setName("Developer Viking");
        startActivity(new Intent(this, GreetLoudlyActivity.class));

    }

}
```

The second activity, where we shout the name of the user:

```Java
// access modifiers omitted for brevity
class GreetLoudlyActivity extends Activity {

    TextView textview;

    void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.reading);
        textview = (TextView) findViewById(R.id.message);
    }

    void onResume() {
        super.onResume();

        MyApplication app = (MyApplication) getApplication();
        textview.setText("HELLO " + app.getName().toUpperCase());
    }
}
```

### The Scenario

  * The user starts the app.

  * In _WhatIsYourNameActivity_, you ask for the name of the user and you store it in _MyApplication_.

  * In _GreetLoudlyActivity_, you fetch the user’s name from the _MyApplication_ object and display it.

  * The user leaves the app using the home button.

  * A few hours later, Android silently kills the app to reclaim some memory.

So far, so good!

But here comes the crashy part…

  * The user reopens the app.

  * Android creates a new _MyApplication_ instance and restores _GreetLoudlyActivity_.

  * _GreetLoudlyActivity_ fetches the user’s name, which is now null, and crashes with a _NullPointerException_.

### Why Does it Crash?

In this sample, we crash because the Application object is brand new, so the name variable is null, leading to a _NullPointerException_ when we call _String#toUpperCase()_ on it.

Which brings us to the core of the problem: The application object will not stay in memory forever, it will get killed. Contrary to popular belief, the app won’t be restarted from scratch. Android will create a new Application object and start the activity where the user was before to give the illusion that the application was never killed in the first place.

Which means that if you expect some data to be in your application object just because your user is not supposed to be able to open activity B before activity A, you are in for a crashy surprise.

### What Are The Alternatives

There is no magic solution here, you can do one of the following:

  * Explicitly pass the data to the Activity through the intent.

  * Use one of the many ways to persist the data to disk.

  * Always do a null-check and handle it manually.

### How To Simulate The Application Being Killed

DIT: As pointed out by Daniel Lew, an easier way to kill your app is to simply use the “Stop Process” feature in DDMS. This will work on any phone as long as your application is debuggable.**

To test this, you must use an emulator or a rooted phone.

  * Exit your app using the home button.

  * In a terminal:

  ```sh
  # find the process id
  adb shell ps
  # then find the line with the package name of your app

  # Mac/Unix: save some time by using grep:
  adb shell ps | grep your.app.package

  # The result should look like:
  # USER      PID   PPID  VSIZE  RSS     WCHAN    PC         NAME
  # u0_a198   21997 160   827940 22064 ffffffff 00000000 S your.app.package

  # Kill the app by PID
  adb shell kill -9 21997

  # the app is now killed
  ```
  * Now return to the app using the task switcher.

  You are now on a new application instance.

### The Bottom Line

Storing data in the application object is error prone and can crash your app. Prefer storing your global data on disk if it is really needed later or explicitly pass it to your activity in the intent’s extras.

Also remember that while this is true for the application object, it is also true for any singleton or public static field that you might have in your app.

### References

[Don't Store Data in the Application Object](http://www.developerphil.com/dont-store-data-in-the-application-object/)
