---
title: Diving Into Android 'M' Doze
date: 2017-08-04 00:37:58
tags:
categories: "Android学习笔记"
---

### What is Doze?

The first time we saw the term "doze" used in Android, it was actually as a display state in KitKat for Wear (API 20), then later on primary devices with Lollipop. It was intended to indicate a new, low-power screen-on mode where the device shows static (non-interactive) content temporarily (think the black/white clock display on the Nexus 6 in response to significant motion, or the ambient watch display that doesn't actually update).

In the new Android 'M' Preview, Doze Mode takes on a slightly different meaning. It now refers to a sort of "forced idle" state where very little background processing is allowed to take place.

<!--more-->

### Say Hello to DeviceIdleController

_DeviceIdleController_ is the primary driver of this new feature - which I will now refer to as device idle mode instead of doze mode since that better fits the code. If you've read the official docs (linked above) you may have noticed the following in the section on testing, which developers can use to manually test their application behavior with the device in this state:

```sh
$ adb shell dumpsys battery unplug
$ adb shell dumpsys deviceidle step
```

For those not familiar, _dumpsys_ is a binary that interacts with system services (by name). It happens that _deviceidle_ is a name we hadn't seen before, because it is a new system service:

```sh
$ adb shell service list | grep deviceidle
59  deviceidle: [android.os.IDeviceIdleController]
```

This new service is registered at all times to listen for the following system events, which can trigger it into (and out of) this new idle mode:

* Screen on/off
* Charging status
* Significant motion detect

_DeviceIdleController_ maintains a state machine that contains five states:

* **ACTIVE** - Device is in use, or connected to a charge source.
* **INACTIVE** - Device has recently come out of the active state (user turned off the display or unplugged it, for example)
* **IDLE_PENDING** - Hold on to your hats, we are about to enter idle mode.
* **IDLE** - Device is idle.
* **IDLE_MAINTENANCE** - Window is open for applications to do processing.

When the device is awake and in-use, the controller is in the _ACTIVE_ state. External events (inactivity timeout, user powering off the screen, etc.) will drive the state machine into _INACTIVE_. At that point, _DeviceIdleController_ drives the process internally by setting its own alarms with _AlarmManager_:

* An alarm will be set for a pre-determined time in the future (30 minutes in the 'M' preview).
* When this alarm fires, _DeviceIdleController_ will advance to _IDLE_PENDING_ and set the same alarm again.
* On the next trigger of this alarm, the controller advances to the _IDLE_ state. This is the state where application features are fully restricted.
* Moving forward, the service will jump between _IDLE_ and _IDLE_MAINTENANCE_ periodically. The latter is the state where pending application events are triggered before returning to full lockdown.

As noted above, developers can manually advance this state machine with the step command:

```sh
$ adb shell dumpsys deviceidle step
```

We can see all the options the _deviceidle_ dump interface supports with the _-h_ flag:

```sh
$ adb shell dumpsys deviceidle -h
Device idle controller (deviceidle) dump options:
  [-h] [CMD]
  -h: print this help text.
Commands:
  step
    Immediately step to next state, without waiting for alarm.
  disable
    Completely disable device idle mode.
  enable
    Re-enable device idle mode after it had previously been disabled.
  whitelist
    Add (prefix with +) or remove (prefix with -) packages.
```

The public API of the service (represented by the _IDeviceIdleController_ interface) consists entirely of methods to access a whitelist. Applications (system or otherwise) do not have access to drive the controller state in any way.

### Are You on the List?

As you can see from the help menu above, _DeviceIdleController_ maintains a whitelist of applications. We can see the current list by dumping the service's state without any extra parameters:

```sh
$ adb shell dumpsys deviceidle
  Whitelist system apps:
    com.android.providers.downloads
    com.android.vending
    com.google.android.gms
  Whitelist app uids:
    UID=10012: true
    UID=10016: true
    UID=10026: true
  …
```

The list is broken down into two parts: system apps and user apps.

### System Applications

System applications are whitelisted by the platform maker as part of their system configuration definition. Below is an example taken from the Nexus 6, which whitelists the GMS Core (used in GCM), Play Store, and an optional app for power monitoring:

**/system/etc/sysconfig/google.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- These are configurations that must exist on all GMS devices. -->
<config>
    <allow-in-power-save package="com.google.android.gms" />

    <allow-in-power-save package="com.android.vending" />

    <allow-in-power-save package="com.google.android.volta" />
</config>
```

These values are accessible to other system services via the _SystemConfig_ instance.  _DeviceIdleController_ uses _SystemConfig.getAllowInPowerSave()_ to pull these system-defined elements into the whitelist.

### User Applications

The remainder of the whitelist is user-defined, and items can be added/removed in two ways. First, developers can use the whitelist command via the _dumpsys_ interface:

```sh
$ adb shell dumpsys deviceidle whitelist +com.example.myapplication
$ adb shell dumpsys deviceidle
  Whitelist system apps:
    com.android.providers.downloads
    com.android.vending
    com.google.android.gms
  Whitelist user apps:
    com.example.myapplication
  Whitelist app uids:
    UID=10012: true
    UID=10016: true
    UID=10026: true
    UID=10101: true
```

Second, users have control (via **Settings -> Battery -> Ignore optimizations**) to modify the whitelist.

![](/images/categories/android/android_notes/063/BatteryIgnoreMenu.png.webp)

This user control primarily exists because the whitelist is also used by the new App Standby feature of Android 'M'. The whitelist is only partially used when evaluating device idle behavior…

### System Service Behavior in Device Idle Mode

Now that we know some of the basics, let's examine how the rest of the system services react when the device is idle, and determine if that whitelist is of any real use. The following is a list of the primary services touched by _DeviceIdleController_:

#### NetworkPolicyManagerService

Lollipop introduced battery saver mode, in which only whitelisted applications could do background processing. Device idle mode piggybacks on this same logic, granting network access only to apps on the whitelist. This service treats battery saver mode and device idle mode as equivalent.

#### JobSchedulerService

Currently executing jobs in all applications (no exceptions) are canceled. No pending jobs will be started until the device exits idle mode.

#### SyncManager

Currently active syncs in all applications (no exceptions) are canceled.

#### PowerManagerService

The application whitelist is used to determine which wake locks are valid. Apps that are not on the list will have their wake locks promptly disabled.

#### AlarmManagerService

_DeviceIdleController_ registers its next wakeup alarm with a special private method (_AlarmManager.setIdleUntil()_). When _AlarmManagerService_ sees this, all standard application alarms are forced into a pending state until the next _DeviceIdleController_ alarm triggers. In this way, application alarms are batched together and run only on the _IDLE_MAINTENANCE_ periods driven by the controller.

There are three flags that allow an alarm to escape this fate:

* **FLAG_ALLOW_WHILE_IDLE** - Set by any application using the new _setAndAllowWhileIdle()_ API.
* **FLAG_WAKE_FROM_IDLE** - Set by any application using the _setAlarmClock()_ API.
* **FLAG_IDLE_UNTIL** - Reserved for the alarm used by _DeviceIdleController_ to advance its state machine.

Processes with a UID < 10000 (i.e. system services) are automatically granted _FLAG_ALLOW_WHILE_IDLE_ on every alarm they set.

### Final Thought

One thing many developers have asked about is that the docs indicate applications will be granted "brief network access" during idle if they receive high-priority messages using GCM. For my part, there is nothing I can see in the framework implementation that is linked to GCM explicitly (thank goodness, because that would be silly, right?). Since the GMS applications are closed source and obfuscated, we can only theorize how this will happen on Google devices.

As it stands, any system application could temporarily modify the app whitelist or network policy for a specific application. This would disable the networking restriction on a per-app basis without modifying the current idle state of the device. It is also a bit unclear in the current preview if there is still an open hole in the _SyncManager_. Existing syncs are canceled, but I'm not certain new syncs wouldn't be allowed. Applications may still be able to execute a new sync from a GCM trigger (which is one of the only apps allowed network access in this state).

…all of that, of course, is purely speculation.

From:[Diving Into Android 'M' Doze](https://www.protechtraining.com/blog/post/875?ncr=1)
