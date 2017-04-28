---
title: It’s time to kiss goodbye to your implicit BroadcastReceivers
date: 2017-4-28 09:33:14
tags:
categories: "Android O Features and APIs"
---

[Android O Developer Preview is out](https://developer.android.com/preview/index.html)! It’s time to start guessing what delicious dessert the letter O will stand for.

Every Android developer should check the list of behavior changes in the upcoming OS version right now. It’s nicely sectioned, so you see the behavior changes for only apps that target Android O, and changes that affect every API level on devices running Android O.

But you can skip the Broadcast Limitations section. I’ve got you covered.

If you aren’t planning to update your app to Android O, you’re fine. But if you want to target Android O and its shiny new features AND you have registered implicit BroadcastReceivers in your AndroidManifest, keep on reading.

>NOTE: There’s a list of exceptions for broadcasts that aren’t affected. So if your app relies, for example, on ACTION_BOOT_COMPLETED or ACTION_HEADSET_PLUG, with many others, they’ll continue to work normally.

#### It’s all about saving power

Maximizing performance & battery life has been a big priority for Android for a while. In Android Marshmallow, **Doze and App Standby** were introduced to dramatically optimize previously poor battery life, a real pain point of most modern mobile phones.

While Doze restricted all apps to do battery-consuming background tasks in special maintenance windows, App Standby deferred network activities for least recently used apps. Along with Doze the JobScheduler was introduced, and Google began to urge developers to use it to schedule all background tasks.

<!--more-->

#### What does saving battery have to do with BroadcastReceivers?

With Android Nougat, the support for registering three different implicit BroadcastReceivers in your AndroidManifest was removed.

With apps targeting Android O, all implicit BroadcastReceivers registered in the manifest (except [these](https://developer.android.com/preview/features/background-broadcasts.html)) will stop working.

As it turns out, apps were registering a lot of unnecessary implicit BroadcastReceivers in their manifest, causing unwanted battery drain. For example, a lot of apps registered receivers for the _CONNECTIVITY_ACTION_ broadcast. “So what”, you might think. “What does that have to do with battery consumption?”
Imagine this: you have dozen apps that listen for connectivity change events. You leave home to go grocery shopping. The connection for the home wifi drops once you exit your apartment. Android system sends the _CONNECTIVITY_ACTION_ broadcast, and causes all of those dozen apps to wake up and react to that change.

Since the wifi is no longer available, you get connected to the mobile network. Guess what, the CONNECTIVITY_ACTION broadcast gets sent again. The same sequence happens again when you get back home and reconnect to your wifi.

magine 20 apps that each listen for network connection events, new pictures taken through camera, new app installed / uninstalled events, charger connected / disconnected events… You quickly realize the Android Team has a pretty fair point in eliminating implicit Broadcasts registered in the AndroidManifest.

Since all those apps register for these broadcasts in their AndroidManifest, they’re always woken up to receive these events. Even when they’re not in the foreground or even running at all.

#### I have registered a BroadcastReceiver in my manifest, what now?

Don’t panic. Remember that while it may be frustrating to migrate your existing code to be Android O compatible, it’s not a major change and it’s all for the greater good.

##### Determine if the BroadcastReceiver is implicit or explicit

According to the documentation, any Broadcast that’s not directly related to your app is an implicit Broadcast. Like the documentation states, _ACTION_PACKAGE_REPLACED_ is an implicit broadcast, since it notifies you of every newly installed package.

Likewise, any Broadcast that’s directly related to your app is an explicit one. So _ACTION_MY_PACKAGE_REPLACED_ is an explicit Broadcast, since it notifies you when only your app is updated.

Most of the Broadcasts you can listen to are implicit ones.

##### Check if your app is affected

Android Developer documentation provides a list of implicit Broadcasts that aren’t affected and will continue to work normally. That means if your app only registers listeners for these implicit Broadcasts in the AndroidManifest, and these only, you’re safe.

The most common unaffected ones are _ACTION_BOOT_COMPLETED_, _ACTION_HEADSET_PLUG_ and _ACTION_CONNECTION_STATE_CHANGED_ for Bluetooth state changes, with many others.

##### JobScheduler to the rescue

If you listened for network connection, charger or device idle events, the chances are you want to use _JobScheduler_ for your task instead.

```java
ComponentName myService = new ComponentName(this, MyService.class);
JobInfo myJob = new JobInfo.Builder(myService)
        .setRequiresCharging(true)
        .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)
        .setPersisted(true)
        .build();

JobScheduler jobScheduler = (JobScheduler) getSystemService(Context.JOB_SCHEDULER_SERVICE);
jobScheduler.schedule(myJob);
```

JobScheduler plays nicely with the Android system and Doze battery optimizations. It allows executing tasks based on conditions that you can define. For example, you can schedule a Job to be executed once the charger gets connected and the device has a working network connection while being in idle mode.
JobScheduler is only supported on Android Lollipop (API 21) and up. If your app has a minSdkVersion below 21, there’s libraries with similar APIs that provide backwards compatibility. The officially recommended one is [FirebaseJobDispatcher](https://github.com/firebase/firebase-jobdispatcher-android).

We at Codemate are currently having great success with Evernote’s Android-Job library. Android-Job uses _JobScheduler_ on devices that have Lollipop or greater and falls back to _GCMNetworkManager_ or _AlarmManager_ depending on if the system has _Google Play Services_ installed or not.

#### Context.registerReceiver() has always got your back

Whether your Broadcast is implicit or explicit, you can always register it by using the _Context.registerReceiver()_ method inside your Activity, Service, Fragment or even a custom View.

So, for example if your AndroidManifest had a receiver for listening charger events:

```xml
...
<receiver android:name=".ChargerReceiver">
    <intent-filter>
        <action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
    </intent-filter>
</receiver>
...
```

You would remove that receiver from your AndroidManifest.xml and the BatteryReceiver class completely, and move listening logic inside your Activity, Service, Fragment, whatever:

```java
...
public class MainActivity extends AppCompatActivity {
    private BroadcastReceiver chargerReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        chargerReceiver = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                // TODO: Awesome things
            }
        };

        registerReceiver(
                chargerReceiver,
                new IntentFilter(Intent.ACTION_POWER_CONNECTED)
        );
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(chargerReceiver);
    }
}
...
```

The difference of registering BroadcastReceivers with _Context.registerReceiver()_ is that you must eventually also unregister it, while receivers defined in you manifest will wake up your app no matter what. When to unregister depends on where you first register your receiver.

If done inside an Activity / Fragment, a good place to unregister is the onDestroy callback. Inside a Service, unregister when you don’t need to receive those events any more.

#### Conclusion

Get rid of all implicit (except these) BroadcastReceivers in your AndroidManifest. Use _JobScheduler_, _FirebaseJobDispatcher_ or alternatives for your tasks instead. If those don’t support the event you were listening to, register BroadcastReceivers in your Activity, Service, Fragment or View depending on your needs.

From:[It’s time to kiss goodbye to your implicit BroadcastReceivers](https://medium.com/@iiro.krankka/its-time-to-kiss-goodbye-to-your-implicit-broadcastreceivers-eefafd9f4f8a)
