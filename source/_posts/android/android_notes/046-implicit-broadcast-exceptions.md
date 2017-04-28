---
title: Implicit Broadcast Exceptions
date: 2017-4-28 08:58:31
tags:
categories: "Android O Features and APIs"
---

As part of the new O Preview Background Execution Limits, apps that target the O Developer Preview can no longer register broadcast receivers for implicit broadcasts in their manifest. However, several broadcasts are currently exempted from these limitations. Apps can continue to register listeners for the following broadcasts, no matter what API level the apps target.

**ACTION_LOCKED_BOOT_COMPLETED, ACTION_BOOT_COMPLETED**

Exempted because these broadcasts are only sent only once, at first boot, and many apps need to receive this broadcast to schedule jobs, alarms, and so forth.

**ACTION_USER_INITIALIZE, "android.intent.action.USER_ADDED", "android.intent.action.USER_REMOVED"**

These broadcasts are protected by privileged permissions, so most normal apps cannot receive them anyway.

**"android.intent.action.TIME_SET", ACTION_TIMEZONE_CHANGED**

Clock apps may need to receive these broadcasts to update alarms when the time or timezone is changed.

<!--more-->

**ACTION_LOCALE_CHANGED**

Only sent when the locale changes, which is not often. Apps might need to update their data when the locale changes.

**ACTION_USB_ACCESSORY_ATTACHED, ACTION_USB_ACCESSORY_DETACHED, ACTION_USB_DEVICE_ATTACHED, ACTION_USB_DEVICE_DETACHED**

If an app needs to know about these USB-related events, there is not currently a good alternative to registering for the broadcast.

**ACTION_HEADSET_PLUG**

Since this broadcast is only sent when the user physically connects or disconnects a plug, it's not likely to affect user experience if apps respond to this broadcast.

**ACTION_CONNECTION_STATE_CHANGED, ACTION_CONNECTION_STATE_CHANGED**

Similar to ACTION_HEADSET_PLUG, user experience is not likely to suffer if apps receive broadcasts for these Bluetooth events.

**ACTION_CARRIER_CONFIG_CHANGED, TelephonyIntents.ACTION_*_SUBSCRIPTION_CHANGED, "TelephonyIntents.SECRET_CODE_ACTION"**

OEM telephony apps may need to receive these broadcasts.

**ACTION_DEVICE_STORAGE_LOW, ACTION_DEVICE_STORAGE_OK**

Some apps may want to respond to low-storage broadcasts by deleting data. For example, photo apps might delete older photos that have already been backed up to cloud storage.

**LOGIN_ACCOUNTS_CHANGED_ACTION**

Some apps need to know about changes to login accounts so they can set up scheduled operations for the new and changed accounts.

**ACTION_PACKAGE_DATA_CLEARED**

Only sent when the user explicitly clears their data from Settings, so broadcast receivers are unlikely to significantly affect user experience.

**ACTION_PACKAGE_FULLY_REMOVED**

Some apps may need to update their stored data when another package is removed; for those apps, there is no good alternative to registering for this broadcast.

>Note: Other package-related broadcasts (such as ACTION_PACKAGE_REPLACED) are not exempted from the new restrictions. These broadcasts are common enough that there is a potential performance impact to exempting them.

**ACTION_NEW_OUTGOING_CALL**

Apps that take action in response to users placing calls need to receive this broadcast.

**ACTION_DEVICE_OWNER_CHANGED**

This broadcast is not sent very often; some apps need to receive it, so that they know that the device's security status has changed.

**ACTION_EVENT_REMINDER**

Sent by the calendar provider to post an event reminder to the calendar app. Since the calendar provider doesn't know what the calendar app is, this broadcast has to be implicit.
