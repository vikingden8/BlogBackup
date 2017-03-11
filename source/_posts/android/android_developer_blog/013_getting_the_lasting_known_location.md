---
title: Getting the Last Known Location
date: 2017-03-11 20:51:48
tags:
categories: "Android官方博客"
---

Using the Google Play services location APIs, your app can request the last known location of the user's device. In most cases, you are interested in the user's current location, which is usually equivalent to the last known location of the device.

Specifically, use the [fused location provider](https://developer.android.com/reference/com/google/android/gms/location/FusedLocationProviderApi.html) to retrieve the device's last known location. The fused location provider is one of the location APIs in Google Play services. It manages the underlying location technology and provides a simple API so that you can specify requirements at a high level, like high accuracy or low power. It also optimizes the device's use of battery power.

This lesson shows you how to make a single request for the location of a device using the [getLastLocation()](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderApi#getLastLocation(com.google.android.gms.common.api.GoogleApiClient)) method in the fused location provider.

### Set Up Google Play Services

To access the fused location provider, your app's development project must include Google Play services. Download and install the Google Play services component via the SDK Manager and add the library to your project. For details, see the guide to [Setting Up Google Play Services](https://developers.google.com/android/guides/setup).

<!--more-->

### Specify App Permissions

Apps that use location services must request location permissions. Android offers two location permissions: _ACCESS_COARSE_LOCATION_ and _ACCESS_FINE_LOCATION_. The permission you choose determines the accuracy of the location returned by the API. If you specify _ACCESS_COARSE_LOCATION_, the API returns a location with an accuracy approximately equivalent to a city block.

This lesson requires only coarse location. Request this permission with the uses-permission element in your app manifest, as the following code snippet shows:

```java
    ...
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    ...
```

### Connect to Google Play Services

To connect to the API, you need to create an instance of the Google Play services API client. For details about using the client, see the guide to [Accessing Google APIs](https://developer.android.com/google/auth/api-client.html#Starting).

In your activity's onCreate() method, create an instance of Google API Client, using the GoogleApiClient.Builder class to add the LocationServices API, as the following code snippet shows.

```java
// Create an instance of GoogleAPIClient.
if (mGoogleApiClient == null) {
    mGoogleApiClient = new GoogleApiClient.Builder(this)
        .addConnectionCallbacks(this)
        .addOnConnectionFailedListener(this)
        .addApi(LocationServices.API)
        .build();
}
```

To connect, call _connect()_ from the activity's _onStart()_ method. To disconnect, call _disconnect()_ from the activity's _onStop()_ method. The following snippet shows an example of how to use both of these methods.

```java
protected void onStart() {
    mGoogleApiClient.connect();
    super.onStart();
}

protected void onStop() {
    mGoogleApiClient.disconnect();
    super.onStop();
}
```

### Get the Last Known Location

Once you have connected to Google Play services and the location services API, you can get the last known location of a user's device. When your app is connected to these you can use the fused location provider's _getLastLocation()_ method to retrieve the device location. The precision of the location returned by this call is determined by the permission setting you put in your app manifest, as described in the [Specify App Permissions](https://developer.android.com/training/location/retrieve-current.html#permissions) section of this document.

To request the last known location, call the _getLastLocation()_ method, passing it your instance of the _GoogleApiClient_ object. Do this in the _onConnected()_ callback provided by Google API Client, which is called when the client is ready. The following code snippet illustrates the request and a simple handling of the response:

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    @Override
    public void onConnected(Bundle connectionHint) {
        mLastLocation = LocationServices.FusedLocationApi.getLastLocation(
                mGoogleApiClient);
        if (mLastLocation != null) {
            mLatitudeText.setText(String.valueOf(mLastLocation.getLatitude()));
            mLongitudeText.setText(String.valueOf(mLastLocation.getLongitude()));
        }
    }
}
```

The _getLastLocation()_ method returns a _Location_ object from which you can retrieve the latitude and longitude coordinates of a geographic location. The location object returned may be null in rare cases when the location is not available.

The next lesson, [Changing Location Settings], shows you how to detect the current location settings, and prompt the user to change settings as appropriate for your app's requirements.
