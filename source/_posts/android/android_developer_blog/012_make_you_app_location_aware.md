---
title: Making Your App Location-Aware
date: 2017-03-11 20:35:59
tags:
categories: "Android官方博客"
---

One of the unique features of mobile applications is location awareness. Mobile users take their devices with them everywhere, and adding location awareness to your app offers users a more contextual experience. The location APIs available in Google Play services facilitate adding location awareness to your app with automated location tracking, geofencing, and activity recognition.

The [Google Play services location APIs](https://developers.google.com/android/reference/com/google/android/gms/location/package-summary) are preferred over the Android framework location APIs ([android.location](https://developer.android.com/reference/android/location/package-summary.html)) as a way of adding location awareness to your app. If you are currently using the Android framework location APIs, you are strongly encouraged to switch to the Google Play services location APIs as soon as possible.

This class shows you how to use the Google Play services location APIs in your app to get the current location, get periodic location updates, and look up addresses. The class includes sample apps and code snippets that you can use as a starting point for adding location awareness to your app.

<!--more-->

>Note: Since this class is based on the Google Play services client library, make sure you install the latest version before using the sample apps or code snippets. To learn how to set up the client library with the latest version, see [Setup](https://developers.google.com/android/guides/setup) in the Google Play services guide.

## Lessons

#### [Getting the Last Known Location]()

Learn how to retrieve the last known location of an Android device, which is usually equivalent to the user's current location.

#### [Changing Location Settings]()

Learn how to detect and apply system settings for location features.

#### [Receiving Location Updates]()

Learn how to request and receive periodic location updates.

#### [Displaying a Location Address]()

Learn how to convert a location's latitude and longitude into an address (reverse geocoding).

#### [Creating and Monitoring Geofences]()

Learn how to define one or more geographic areas as locations of interest, called geofences, and detect when the user is close to or inside a geofence.
