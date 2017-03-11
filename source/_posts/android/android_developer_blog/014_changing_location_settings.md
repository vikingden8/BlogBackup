---
title: Changing Location Settings
date: 2017-03-11 21:19:09
tags:
categories: "Android官方博客"
---

If your app needs to request location or receive permission updates, the device needs to enable the appropriate system settings, such as GPS or Wi-Fi scanning. Rather than directly enabling services such as the device's GPS, your app specifies the required level of accuracy/power consumption and desired update interval, and the device automatically makes the appropriate changes to system settings. These settings are defined by the [LocationRequest](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest) data object.

This lesson shows you how to use the [Settings API](https://developers.google.com/android/reference/com/google/android/gms/location/SettingsApi) to check which settings are enabled, and present the Location Settings dialog for the user to update their settings with a single tap.

### Connect to Location Services

In order to use the location services provided by Google Play Services and the fused location provider, connect your app using the Google API Client, then check the current location settings and prompt the user to enable the required settings if needed. For details on connecting with the Google API client, see [Getting the Last Known Location](https://developer.android.com/training/location/retrieve-current.html).

<!--more-->

Apps that use location services must request location permissions. For this lesson, coarse location detection is sufficient. Request this permission with the uses-permission element in your app manifest, as shown in the following example:

```java
  ...
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
  ...
```

If the device is running Android 6.0 or higher, and your app's target SDK is 23 or higher, the app has to list the permissions in the manifest and request those permissions at run time. For more information, see Requesting Permissions at Run Time.

### Set Up a Location Request

To store parameters for requests to the fused location provider, create a LocationRequest. The parameters determine the level of accuracy for location requests. For details of all available location request options, see the LocationRequest class reference. This lesson sets the update interval, fastest update interval, and priority, as described below:

  * **Update interval**

  _setInterval()_ - This method sets the rate in milliseconds at which your app prefers to receive location updates. Note that the location updates may be faster than this rate if another app is receiving updates at a faster rate, or slower than this rate, or there may be no updates at all (if the device has no connectivity, for example).

  * **Fastest update interval**

  _setFastestInterval()_ - This method sets the fastest rate in milliseconds at which your app can handle location updates. You need to set this rate because other apps also affect the rate at which updates are sent. The Google Play services location APIs send out updates at the fastest rate that any app has requested with setInterval(). If this rate is faster than your app can handle, you may encounter problems with UI flicker or data overflow. To prevent this, call setFastestInterval() to set an upper limit to the update rate.

  * **Priority**

  _setPriority()_ - This method sets the priority of the request, which gives the Google Play services location services a strong hint about which location sources to use. The following values are supported:

    * **PRIORITY_BALANCED_POWER_ACCURACY** - Use this setting to request location precision to within a city block, which is an accuracy of approximately 100 meters. This is considered a coarse level of accuracy, and is likely to consume less power. With this setting, the location services are likely to use WiFi and cell tower positioning. Note, however, that the choice of location provider depends on many other factors, such as which sources are available.

    * **PRIORITY_HIGH_ACCURACY** - Use this setting to request the most precise location possible. With this setting, the location services are more likely to use GPS to determine the location.

    * **PRIORITY_LOW_POWER** - Use this setting to request city-level precision, which is an accuracy of approximately 10 kilometers. This is considered a coarse level of accuracy, and is likely to consume less power.

    * **PRIORITY_NO_POWER** - Use this setting if you need negligible impact on power consumption, but want to receive location updates when available. With this setting, your app does not trigger any location updates, but receives locations triggered by other apps.

Create the location request and set the parameters as shown in this code sample:

```java
protected void createLocationRequest() {
    LocationRequest mLocationRequest = new LocationRequest();
    mLocationRequest.setInterval(10000);
    mLocationRequest.setFastestInterval(5000);
    mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
}
```

The priority of PRIORITY_HIGH_ACCURACY, combined with the ACCESS_FINE_LOCATION permission setting that you've defined in the app manifest, and a fast update interval of 5000 milliseconds (5 seconds), causes the fused location provider to return location updates that are accurate to within a few feet. This approach is appropriate for mapping apps that display the location in real time.

>Performance hint: If your app accesses the network or does other long-running work after receiving a location update, adjust the fastest interval to a slower value. This adjustment prevents your app from receiving updates it can't use. Once the long-running work is done, set the fastest interval back to a fast value.

### Get Current Location Settings

Once you have connected to Google Play services and the location services API, you can get the current location settings of a user's device. To do this, create a LocationSettingsRequest.Builder, and add one or more location requests. The following code snippet shows how to add the location request that was created in the previous step:

```java
LocationSettingsRequest.Builder builder = new LocationSettingsRequest.Builder()
     .addLocationRequest(mLocationRequest);
```

Next check whether the current location settings are satisfied:

```java
PendingResult<LocationSettingsResult> result =
         LocationServices.SettingsApi.checkLocationSettings(mGoogleClient,
                 builder.build());
```

When the _PendingResult_ returns, your app can check the location settings by looking at the status code from the _LocationSettingsResult_ object. To get even more details about the the current state of the relevant location settings, your app can call the _LocationSettingsResult_ object's _getLocationSettingsStates()_ method.

### Prompt the User to Change Location Settings

To determine whether the location settings are appropriate for the location request, check the status code from the _LocationSettingsResult_ object. A status code of RESOLUTION_REQUIRED indicates that the settings must be changed. To prompt the user for permission to modify the location settings, call _startResolutionForResult(Activity, int)_. This method brings up a dialog asking for the user's permission to modify location settings. The following code snippet shows how to check the location settings, and how to call _startResolutionForResult(Activity, int)_.

```java
result.setResultCallback(new ResultCallback<LocationSettingsResult>()) {
     @Override
     public void onResult(LocationSettingsResult result) {
         final Status status = result.getStatus();
         //final LocationSettingsStates status = result.getLocationSettingsStates();
         //above code maybe a mistake in android official documentation
         switch (status.getStatusCode()) {
             case LocationSettingsStatusCodes.SUCCESS:
                 // All location settings are satisfied. The client can
                 // initialize location requests here.
                 ...
                 break;
             case LocationSettingsStatusCodes.RESOLUTION_REQUIRED:
                 // Location settings are not satisfied, but this can be fixed
                 // by showing the user a dialog.
                 try {
                     // Show the dialog by calling startResolutionForResult(),
                     // and check the result in onActivityResult().
                     status.startResolutionForResult(
                         OuterClass.this,
                         REQUEST_CHECK_SETTINGS);
                 } catch (SendIntentException e) {
                     // Ignore the error.
                 }
                 break;
             case LocationSettingsStatusCodes.SETTINGS_CHANGE_UNAVAILABLE:
                 // Location settings are not satisfied. However, we have no way
                 // to fix the settings so we won't show the dialog.
                 ...
                 break;
         }
     }
 });
```

The next lesson, [Receiving Location Updates](https://developer.android.com/training/location/receive-location-updates.html), shows you how to receive periodic location updates.
