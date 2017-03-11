---
title: Receiving Location Updates
date: 2017-03-11 21:33:25
tags:
categories: "Android官方博客"
---

If your app can continuously track location, it can deliver more relevant information to the user. For example, if your app helps the user find their way while walking or driving, or if your app tracks the location of assets, it needs to get the location of the device at regular intervals. As well as the geographical location (latitude and longitude), you may want to give the user further information such as the bearing (horizontal direction of travel), altitude, or velocity of the device. This information, and more, is available in the _Location_ object that your app can retrieve from the _fused location provider_.

While you can get a device's location with _getLastLocation()_, as illustrated in the lesson on [Getting the Last Known Location](https://developer.android.com/training/location/retrieve-current.html), a more direct approach is to request periodic updates from the fused location provider. In response, the API updates your app periodically with the best available location, based on the currently-available location providers such as WiFi and GPS (Global Positioning System). The accuracy of the location is determined by the providers, the location permissions you've requested, and the options you set in the location request.

This lesson shows you how to request regular updates about a device's location using the _requestLocationUpdates()_ method in the fused location provider.

<!--more-->

### Get the Last Known Location

The last known location of the device provides a handy base from which to start, ensuring that the app has a known location before starting the periodic location updates. The lesson on [Getting the Last Known Location](https://developer.android.com/training/location/retrieve-current.html) shows you how to get the last known location by calling _getLastLocation()_. The snippets in the following sections assume that your app has already retrieved the last known location and stored it as a Location object in the global variable _mCurrentLocation_.

Apps that use location services must request location permissions. In this lesson you require fine location detection, so that your app can get as precise a location as possible from the available location providers. Request this permission with the uses-permission element in your app manifest, as shown in the following example:

```java
  ...
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
  ...
```

### Request Location Updates

Before requesting location updates, your app must connect to location services and make a location request. The lesson on [Changing Location Settings](https://developer.android.com/training/location/change-location-settings.html) shows you how to do this. Once a location request is in place you can start the regular updates by calling _requestLocationUpdates()_. Do this in the _onConnected()_ callback provided by Google API Client, which is called when the client is ready.

Depending on the form of the request, the fused location provider either invokes the _LocationListener.onLocationChanged()_ callback method and passes it a Location object, or issues a _PendingIntent_ that contains the location in its extended data. The accuracy and frequency of the updates are affected by the location permissions you've requested and the options you set in the location request object.

This lesson shows you how to get the update using the _LocationListener_ callback approach. Call _requestLocationUpdates()_, passing it your instance of the _GoogleApiClient_, the _LocationRequest_ object, and a _LocationListener_. Define a _startLocationUpdates()_ method, called from the _onConnected()_ callback, as shown in the following code sample:

```java
@Override
public void onConnected(Bundle connectionHint) {
    ...
    if (mRequestingLocationUpdates) {
        startLocationUpdates();
    }
}

protected void startLocationUpdates() {
    LocationServices.FusedLocationApi.requestLocationUpdates(
            mGoogleApiClient, mLocationRequest, this);
}
```

Notice that the above code snippet refers to a boolean flag, _mRequestingLocationUpdates_, used to track whether the user has turned location updates on or off. For more about retaining the value of this flag across instances of the activity, see [Save the State of the Activity](https://developer.android.com/training/location/receive-location-updates.html#save-state).

### Define the Location Update Callback

The fused location provider invokes the _LocationListener.onLocationChanged()_ callback method. The incoming argument is a Location object containing the location's latitude and longitude. The following snippet shows how to implement the _LocationListener_ interface and define the method, then get the timestamp of the location update and display the latitude, longitude and timestamp on your app's user interface:

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener, LocationListener {
    ...
    @Override
    public void onLocationChanged(Location location) {
        mCurrentLocation = location;
        mLastUpdateTime = DateFormat.getTimeInstance().format(new Date());
        updateUI();
    }

    private void updateUI() {
        mLatitudeTextView.setText(String.valueOf(mCurrentLocation.getLatitude()));
        mLongitudeTextView.setText(String.valueOf(mCurrentLocation.getLongitude()));
        mLastUpdateTimeTextView.setText(mLastUpdateTime);
    }
}
```

### Stop Location Updates

Consider whether you want to stop the location updates when the activity is no longer in focus, such as when the user switches to another app or to a different activity in the same app. This can be handy to reduce power consumption, provided the app doesn't need to collect information even when it's running in the background. This section shows how you can stop the updates in the activity's _onPause()_ method.

To stop location updates, call _removeLocationUpdates()_, passing it your instance of the _GoogleApiClient_ object and a _LocationListener_, as shown in the following code sample:

```java
@Override
protected void onPause() {
    super.onPause();
    stopLocationUpdates();
}

protected void stopLocationUpdates() {
    LocationServices.FusedLocationApi.removeLocationUpdates(
            mGoogleApiClient, this);
}

```

Use a boolean, _mRequestingLocationUpdates_, to track whether location updates are currently turned on. In the activity's _onResume()_ method, check whether location updates are currently active, and activate them if not:

```java
@Override
public void onResume() {
    super.onResume();
    if (mGoogleApiClient.isConnected() && !mRequestingLocationUpdates) {
        startLocationUpdates();
    }
}

```

### Save the State of the Activity

A change to the device's configuration, such as a change in screen orientation or language, can cause the current activity to be destroyed. Your app must therefore store any information it needs to recreate the activity. One way to do this is via an instance state stored in a _Bundle_ object.

The following code sample shows how to use the activity's _onSaveInstanceState()_ callback to save the instance state:

```java
public void onSaveInstanceState(Bundle savedInstanceState) {
    savedInstanceState.putBoolean(REQUESTING_LOCATION_UPDATES_KEY,
            mRequestingLocationUpdates);
    savedInstanceState.putParcelable(LOCATION_KEY, mCurrentLocation);
    savedInstanceState.putString(LAST_UPDATED_TIME_STRING_KEY, mLastUpdateTime);
    super.onSaveInstanceState(savedInstanceState);
}
```

Define an _updateValuesFromBundle()_ method to restore the saved values from the previous instance of the activity, if they're available. Call the method from the activity's _onCreate()_ method, as shown in the following code sample:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    ...
    updateValuesFromBundle(savedInstanceState);
}

private void updateValuesFromBundle(Bundle savedInstanceState) {
    if (savedInstanceState != null) {
        // Update the value of mRequestingLocationUpdates from the Bundle, and
        // make sure that the Start Updates and Stop Updates buttons are
        // correctly enabled or disabled.
        if (savedInstanceState.keySet().contains(REQUESTING_LOCATION_UPDATES_KEY)) {
            mRequestingLocationUpdates = savedInstanceState.getBoolean(
                    REQUESTING_LOCATION_UPDATES_KEY);
            setButtonsEnabledState();
        }

        // Update the value of mCurrentLocation from the Bundle and update the
        // UI to show the correct latitude and longitude.
        if (savedInstanceState.keySet().contains(LOCATION_KEY)) {
            // Since LOCATION_KEY was found in the Bundle, we can be sure that
            // mCurrentLocationis not null.
            mCurrentLocation = savedInstanceState.getParcelable(LOCATION_KEY);
        }

        // Update the value of mLastUpdateTime from the Bundle and update the UI.
        if (savedInstanceState.keySet().contains(LAST_UPDATED_TIME_STRING_KEY)) {
            mLastUpdateTime = savedInstanceState.getString(
                    LAST_UPDATED_TIME_STRING_KEY);
        }
        updateUI();
    }
}
```

For more about saving instance state, see the Android Activity class reference.

>Note: For a more persistent storage, you can store the user's preferences in your app's _SharedPreferences_. Set the shared preference in your activity's _onPause()_ method, and retrieve the preference in _onResume()_. For more information about saving preferences, read Saving [Key-Value Sets](https://developer.android.com/training/basics/data-storage/shared-preferences.html).

The next lesson, [Displaying a Location Address](https://developer.android.com/training/location/display-address.html), shows you how to display the street address for a given location.
