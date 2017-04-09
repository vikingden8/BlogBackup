---
title: Android UI-Fixing skipped frames
date: 2017-04-09 21:02:48
tags:
categories: "Android学习笔记"
---

Anyone who begins developing android application sees this message on logcat **“Choreographer(abc): Skipped xx frames! The application may be doing too much work on its main thread.”** So what does it actually means, why should you be concerned and how to solve it.

What this means is that your code is taking long to process and frames are being skipped because of it, It maybe because of some heavy processing that you are doing at the heart of your application or DB access or any other thing which causes the thread to stop for a while. Here is a more detailed explanation –

>Choreographer lets apps to connect themselves to the vsync, and properly time things to improve performance.

>Android view animations internally uses Choreographer for the same purpose: to properly time the animations and possibly improve performance.

>Since Choreographer is told about every vsync events, I can tell if one of the Runnables passed along by the Choreographer.post* apis doesnt finish in one frame’s time, causing frames to be skipped.

>In my understanding Choreographer can only detect the frame skipping. It has no way of telling why this happens.

>The message “The application may be doing too much work on its main thread.” could be misleading.

<!--more-->

source :  [http://stackoverflow.com/questions/11266535/meaning-of-choreographer-messages-in-logcat](http://stackoverflow.com/questions/11266535/meaning-of-choreographer-messages-in-logcat)

### Why you should be concerned

When this message pops up on android emulator and the number of frames skipped are fairly small (<100) then you can take a safe bet of the emulator being slow – which happens almost all the times. But if the number of frames skipped and large and in the order of 300+ then there can be some serious trouble with your code. Android devices come in a vast array of hardware unlike ios and windows devices. The RAM and CPU varies and if you want a reasonable performance and user experience on all the devices then you need to fix this thing. When frames are skipped the UI is slow and laggy, which is not a desirable user experience.

### How to fix it

Fixing this requires identifying nodes where there is or possibly can happen long duration of processing. The best way is to do all the processing no matter how small or big in a thread separate from main UI thread. So be it accessing data form SQLite Database or doing some hardcore maths or simply sorting an array – **Do it in a different thread**

Now there is a catch here, You will create a new Thread for doing these operations and when you run your application, it will crash saying “**Only the original thread that created a view hierarchy can touch its views**“. You need to know this fact that UI in android can be changed by the main thread or the UI thread only. Any other thread which attempts to do so, fails and crashes with this error. What you need to do is create a new Runnable inside runOnUiThread and inside this runnable you should do all the operations involving the UI. Find an example [here](http://stackoverflow.com/questions/11254523/android-runonuithread-explanation).

So we have Thread and Runnable for processing data out of main Thread, what else? There is AsyncTask in android which enables doing long time processes on the UI thread. This is the most useful when you applications are data driven or web api driven or use complex UI’s like those build using Canvas. The power of AsyncTask is that is allows doing things in background and once you are done doing the processing, you can simply do the required actions on UI without causing any lagging effect. This is possible because the AsyncTask derives itself from Activity’s UI thread – all the operations you do on UI via AsyncTask are done is a different thread from the main UI thread, No hindrance to user interaction.

So this is what you need to know for making smooth android applications and as far I know every beginner gets this message on his console.

### References

[Android UI : Fixing skipped frames](https://vaibhavtolia.wordpress.com/2013/10/03/79/)
