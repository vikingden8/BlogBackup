---
title: Respecting Audio Focus
date: 2017-03-15 21:41:19
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

Posted by Kristan Uccello, Google Developer Relations

It’s rude to talk during a presentation, it disrespects the speaker and annoys the audience. If your application doesn’t respect the rules of audio focus then it’s disrespecting other applications and annoying the user. If you have never heard of audio focus you should take a look at the Android developer training material.

With multiple apps potentially playing audio it's important to think about how they should interact. To avoid every music app playing at the same time, Android uses audio focus to moderate audio playback—your app should only play audio when it holds audio focus. This post provides some tips on how to handle changes in audio focus properly, to ensure the best possible experience for the user.

### Requesting audio focus

Audio focus should not be requested when your application starts (don’t get greedy), instead delay requesting it until your application is about to do something with an audio stream. By requesting audio focus through the AudioManager system service, an application can use one of the AUDIOFOCUS_GAIN* constants (see Table 1) to indicate the desired level of focus.

```java
  AudioManager am = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);

  int result = am.requestAudioFocus(mOnAudioFocusChangeListener,
     // Hint: the music stream.
     AudioManager.STREAM_MUSIC,
     // Request permanent focus.
     AudioManager.AUDIOFOCUS_GAIN);
  if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
     mState.audioFocusGranted = true;
  } else if (result == AudioManager.AUDIOFOCUS_REQUEST_FAILED) {
     mState.audioFocusGranted = false;
  }
```

<!--more-->

In line 7 above, you can see that we have requested permanent audio focus. An application could instead request transient focus using AUDIOFOCUS_GAIN_TRANSIENT which is appropriate when using the audio system for less than 45 seconds.

Alternatively, the app could use AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK, which is appropriate when the use of the audio system may be shared with another application that is currently playing audio (e.g. for playing a "keep it up" prompt in a fitness application and expecting background music to duck during the prompt). The app requesting AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK should not use the audio system for more than 15 seconds before releasing focus.

### Handling audio focus changes

In order to handle audio focus change events, an application should create an instance of OnAudioFocusChangeListener. In the listener, the application will need to handle theAUDIOFOCUS_GAIN* event and AUDIOFOCUS_LOSS* events (see Table 1). It should be noted that AUDIOFOCUS_GAIN has some nuances which are highlighted in Listing 2, below.

```java
1. mOnAudioFocusChangeListener = new AudioManager.OnAudioFocusChangeListener() {  
2.   
3. @Override
4. public void onAudioFocusChange(int focusChange) {
5.   switch (focusChange) {
6.   case AudioManager.AUDIOFOCUS_GAIN:
7.     mState.audioFocusGranted = true;
8.        
9.     if(mState.released) {
10.       initializeMediaPlayer();
11.    }
12.
13.    switch(mState.lastKnownAudioFocusState) {
14.    case UNKNOWN:
15.      if(mState.state == PlayState.PLAY && !mPlayer.isPlaying()) {
16.        mPlayer.start();
17.      }
18.      break;
19.    case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
20.      if(mState.wasPlayingWhenTransientLoss) {
21.        mPlayer.start();
22.      }
23.      break;
24.    case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
25.      restoreVolume();
26.      break;
27.       }
28.        
29.    break;
30.  case AudioManager.AUDIOFOCUS_LOSS:
31.    mState.userInitiatedState = false;
32.    mState.audioFocusGranted = false;
33.    teardown();
34.    break;
35.  case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
36.    mState.userInitiatedState = false;
37.    mState.audioFocusGranted = false;
38.    mState.wasPlayingWhenTransientLoss = mPlayer.isPlaying();
39.    mPlayer.pause();
40.    break;
41.  case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
42.   mState.userInitiatedState = false;
43.   mState.audioFocusGranted = false;
44.   lowerVolume();
45.   break;
46.  }
47.  mState.lastKnownAudioFocusState = focusChange;
48.  }
49.};
```

AUDIOFOCUS_GAIN is used in two distinct scopes of an applications code. First, it can be used when registering for audio focus as shown in Listing 1. This does NOT translate to an event for the registered OnAudioFocusChangeListener, meaning that on a successful audio focus request the listener will NOT receive an AUDIOFOCUS_GAIN event for the registration.

AUDIOFOCUS_GAIN is also used in the implementation of an OnAudioFocusChangeListener as an event condition. As stated above, the AUDIOFOCUS_GAIN event will not be triggered on audio focus requests. Instead the AUDIOFOCUS_GAIN event will occur only after an AUDIOFOCUS_LOSS* event has occurred. This is the only constant in the set shown Table 1 that is used in both scopes.

There are four cases that need to be handled by the focus change listener. When the application receives an AUDIOFOCUS_LOSS this usually means it will not be getting its focus back. In this case the app should release assets associated with the audio system and stop playback. As an example, imagine a user is playing music using an app and then launches a game which takes audio focus away from the music app. There is no predictable time for when the user will exit the game. More likely, the user will navigate to the home launcher (leaving the game in the background) and launch yet another application or return to the music app causing a resume which would then request audio focus again.

However another case exists that warrants some discussion. There is a difference between losing audio focus permanently (as described above) and temporarily. When an application receives an AUDIOFOCUS_LOSS_TRANSIENT, the behavior of the app should be that it suspends its use of the audio system until it receives an AUDIOFOCUS_GAIN event. When the AUDIOFOCUS_LOSS_TRANSIENT occurs, the application should make a note that the loss is temporary, that way on audio focus gain it can reason about what the correct behavior should be (see lines 13-27 of Listing 2).

Sometimes an app loses audio focus (receives an AUDIOFOCUS_LOSS) and the interrupting application terminates or otherwise abandons audio focus. In this case the last application that had audio focus may receive an AUDIOFOCUS_GAIN event. On the subsequent AUDIOFOCUS_GAIN event the app should check and see if it is receiving the gain after a temporary loss and can thus resume use of the audio system or if recovering from an permanent loss, setup for playback.

If an application will only be using the audio capabilities for a short time (less than 45 seconds), it should use an AUDIOFOCUS_GAIN_TRANSIENT focus request and abandon focus after it has completed its playback or capture. Audio focus is handled as a stack on the system — as such the last process to request audio focus wins.

When audio focus has been gained this is the appropriate time to create a MediaPlayer or MediaRecorder instance and allocate resources. Likewise when an app receives AUDIOFOCUS_LOSS it is good practice to clean up any resources allocated. Gaining audio focus has three possibilities that also correspond to the three audio focus loss cases in Table 1. It is a good practice to always explicitly handle all the loss cases in the OnAudioFocusChangeListener.

![](/images/categories/android/android-developer-blog/019/01.png)

>Note: AUDIOFOCUS_GAIN is used in two places. When requesting audio focus it is passed in as a hint to the AudioManager and it is used as an event case in the OnAudioFocusChangeListener. The gain events highlighted in green are only used when requesting audio focus. The loss events are only used in the OnAudioFocusChangeListener.

![](/images/categories/android/android-developer-blog/019/02.png)

An app will request audio focus (see an example in the sample source code linked below) from the AudioManager (Listing 1, line 1). The three arguments it provides are an audio focus change listener object (optional), a hint as to what audio channel to use (Table 2, most apps should use STREAM_MUSIC) and the type of audio focus from Table 1, column 1. If audio focus is granted by the system (AUDIOFOCUS_REQUEST_GRANTED), only then handle any initialization (see Listing 1, line 9).

>Note: The system will not grant audio focus (AUDIOFOCUS_REQUEST_FAILED) if there is a phone call currently in process and the application will not receive AUDIOFOCUS_GAIN after the call ends.

Within an implementation of OnAudioFocusChange(), understanding what to do when an application receives an onAudioFocusChange() event is summarized in Table 3.

In the cases of losing audio focus be sure to check that the loss is in fact final. If the app receives an AUDIOFOCUS_LOSS_TRANSIENT or AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK it can hold onto the media resources it has created (don’t call release()) as there will likely be another audio focus change event very soon thereafter. The app should take note that it has received a transient loss using some sort of state flag or simple state machine.

If an application were to request permanent audio focus with AUDIOFOCUS_GAIN and then receive an AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK an appropriate action for the application would be to lower its stream volume (make sure to store the original volume state somewhere) and then raise the volume upon receiving an AUDIOFOCUS_GAIN event (see Figure 1, below).

![](/images/categories/android/android-developer-blog/019/03.png)

![](/images/categories/android/android-developer-blog/019/04.png)

### Conclusion and further reading

Understanding how to be a good audio citizen application on an Android device means respecting the system's audio focus rules and handling each case appropriately. Try to make your application behave in a consistent manner and not negatively surprise the user. There is a lot more that can be talked about within the audio system on Android and in the material below you will find some additional discussions.

  * Managing Audio Focus, an Android developer training class.

  * Allowing applications to play nice(r) with each other: Handling remote control buttons, a post on the Android Developers Blog.

Example source code is available here:

[https://android.googlesource.com/platform/development/+/master/samples/RandomMusicPlayer](https://android.googlesource.com/platform/development/+/master/samples/RandomMusicPlayer)
