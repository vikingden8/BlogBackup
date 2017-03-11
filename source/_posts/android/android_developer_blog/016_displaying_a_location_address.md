---
title: Displaying a Location Address
date: 2017-03-11 21:54:19
tags:
categories: "Android官方博客"
---

The lessons _Getting the Last Known Location_ and _Receiving Location Updates_ describe how to get the user's location in the form of a Location object that contains latitude and longitude coordinates. Although latitude and longitude are useful for calculating distance or displaying a map position, in many cases the address of the location is more useful. For example, if you want to let your users know where they are or what is close by, a street address is more meaningful than the geographic coordinates (latitude/longitude) of the location.

Using the _Geocoder_ class in the Android framework location APIs, you can convert an address to the corresponding geographic coordinates. This process is called geocoding. Alternatively, you can convert a geographic location to an address. The address lookup feature is also known as reverse geocoding.

This lesson shows you how to use the _getFromLocation()_ method to convert a geographic location to an address. The method returns an estimated street address corresponding to a given latitude and longitude.

### Get a geographic location

The last known location of the device is a useful starting point for the address lookup feature. The lesson on Getting the Last Known Location shows you how to use the _getLastLocation()_ method provided by the fused location provider to find the latest location of the device.

To access the fused location provider, create an instance of the Google Play services API client. To learn how to connect your client, see Connect to Google Play Services.

To enable the fused location provider to retrieve a precise street address, set the location permission in your app manifest to _ACCESS_FINE_LOCATION_, as shown in the following example:

```java
  ...
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
  ...
```
<!--more-->017

### Define an intent service to fetch the address

The _getFromLocation()_ method provided by the Geocoder class accepts a latitude and longitude and returns a list of addresses. The method is synchronous and may take a long time to do its work, so you should not call it from the main, user interface (UI) thread of your app.

The _IntentService_ class provides a structure for running a task on a background thread. Using this class, you can handle a long-running operation without affecting your UI's responsiveness. Note that the _AsyncTask_ class also allows you to perform background operations, but it's designed for short operations. An _AsyncTask_ shouldn't keep a reference to the UI if the activity is re-created, such as when the device is rotated. In contrast, an _IntentService_ doesn't need to be cancelled when the activity is rebuilt.

Define a _FetchAddressIntentService_ class that extends _IntentService_. This class is your address lookup service. The intent service handles an intent asynchronously on a worker thread and stops itself when it runs out of work. The intent extras provide the data needed by the service, including a Location object for conversion to an address and a _ResultReceiver_ object to handle the results of the address lookup. The service uses a Geocoder to fetch the address for the location and sends the results to the _ResultReceiver_.

### Define the intent service in your app manifest

Add an entry to your app manifest that defines the intent service, as shown here:

```java
    ...
    <application
        ...
        <service
            android:name=".FetchAddressIntentService"
            android:exported="false"/>
    </application>
    ...

```

>Note: The <service> element in the manifest doesn't need to include an intent filter because your main activity creates an explicit intent by specifying the name of the class to use for the intent.

### Create a Geocoder

The process of converting a geographic location to an address is called reverse geocoding. To perform the main work of the intent service (your reverse geocoding request), implement _onHandleIntent()_ within the _FetchAddressIntentService_ class. Create a Geocoder object to handle the reverse geocoding.

A locale represents a specific geographical or linguistic region. Locale objects adjust the presentation of information, such as numbers or dates, to suit the conventions in the region that is represented by the locale. Pass a Locale object to the Geocoder object to ensure that the resulting address is localized to the user's geographic region. Here is an example:

```java
@Override
protected void onHandleIntent(Intent intent) {
    Geocoder geocoder = new Geocoder(this, Locale.getDefault());
    ...
}
```

### Retrieve the street address data

You can now retrieve the street address from the geocoder, handle any errors that may occur, and send the results back to the activity that requested the address. To report the results of the geocoding process, you need two numeric constants that indicate success or failure. Define a Constants class to contain the values, as shown in this code snippet:

```java
public final class Constants {
    public static final int SUCCESS_RESULT = 0;
    public static final int FAILURE_RESULT = 1;
    public static final String PACKAGE_NAME =
        "com.google.android.gms.location.sample.locationaddress";
    public static final String RECEIVER = PACKAGE_NAME + ".RECEIVER";
    public static final String RESULT_DATA_KEY = PACKAGE_NAME +
        ".RESULT_DATA_KEY";
    public static final String LOCATION_DATA_EXTRA = PACKAGE_NAME +
        ".LOCATION_DATA_EXTRA";
}
```

To get a street address corresponding to a geographical location, call _getFromLocation()_, passing it the latitude and longitude from the location object and the maximum number of addresses that you want returned. In this case, you want just one address. The geocoder returns an array of addresses. If no addresses are found to match the given location, it returns an empty list. If there is no backend geocoding service available, the geocoder returns null.

Check for the following errors, as shown in the code sample below:

  * **No location data provided** – The intent extras do not include the Location object that is required for reverse geocoding.

  * **Invalid latitude or longitude used** – The latitude and/or longitude values that are provided in the Location object are invalid.

  * **No geocoder available** – The background geocoding service is not available due to a network error or IO exception.

  * **Sorry, no address found** – The geocoder can't find an address for the given latitude/longitude.
If an error occurs, place the corresponding error message in the _errorMessage_ variable so that you can send it back to the requesting activity.

To get the individual lines of an address object, use the _getAddressLine()_ method that is provided by the Address class. Join the lines into a list of address fragments ready to return to the activity that requested the address.

To send the results back to the requesting activity, call the _deliverResultToReceiver()_ method (defined in Return the address to the requestor). The results consist of the previously-mentioned numeric success/failure code and a string. In the case of a successful reverse geocoding, the string contains the address. In the case of a failure, the string contains the error message, as shown in this code sample:

```java
@Override
protected void onHandleIntent(Intent intent) {
    String errorMessage = "";

    // Get the location passed to this service through an extra.
    Location location = intent.getParcelableExtra(
            Constants.LOCATION_DATA_EXTRA);

    ...

    List<Address> addresses = null;

    try {
        addresses = geocoder.getFromLocation(
                location.getLatitude(),
                location.getLongitude(),
                // In this sample, get just a single address.
                1);
    } catch (IOException ioException) {
        // Catch network or other I/O problems.
        errorMessage = getString(R.string.service_not_available);
        Log.e(TAG, errorMessage, ioException);
    } catch (IllegalArgumentException illegalArgumentException) {
        // Catch invalid latitude or longitude values.
        errorMessage = getString(R.string.invalid_lat_long_used);
        Log.e(TAG, errorMessage + ". " +
                "Latitude = " + location.getLatitude() +
                ", Longitude = " +
                location.getLongitude(), illegalArgumentException);
    }

    // Handle case where no address was found.
    if (addresses == null || addresses.size()  == 0) {
        if (errorMessage.isEmpty()) {
            errorMessage = getString(R.string.no_address_found);
            Log.e(TAG, errorMessage);
        }
        deliverResultToReceiver(Constants.FAILURE_RESULT, errorMessage);
    } else {
        Address address = addresses.get(0);
        ArrayList<String> addressFragments = new ArrayList<String>();

        // Fetch the address lines using getAddressLine,
        // join them, and send them to the thread.
        for(int i = 0; i <= address.getMaxAddressLineIndex(); i++) {
            addressFragments.add(address.getAddressLine(i));
        }
        Log.i(TAG, getString(R.string.address_found));
        deliverResultToReceiver(Constants.SUCCESS_RESULT,
                TextUtils.join(System.getProperty("line.separator"),
                        addressFragments));
    }
}
```

### Return the address to the requestor

The final action that the intent service must complete is sending the address back to a _ResultReceiver_ in the activity that started the service. The _ResultReceiver_ class allows you to send a numeric result code as well as a message containing the result data. The numeric code is useful for reporting the success or failure of the geocoding request. In the case of a successful reverse geocoding, the message contains the address. In the case of a failure, the message contains some text describing the reason for the failure.

You have already retrieved the address from the geocoder, trapped any errors that may occur, and called the _deliverResultToReceiver()_ method, so now you must define the _deliverResultToReceiver()_ method that sends a result code and message bundle to the result receiver.

For the result code, use the value that you've passed to the _deliverResultToReceiver()_ method in the _resultCode_ parameter. To construct the message bundle, concatenate the RESULT_DATA_KEY constant from your Constants class (defined in Retrieve the street address data) and the value in the message parameter that is passed to the _deliverResultToReceiver()_ method, as shown in the following sample:

```java
public class FetchAddressIntentService extends IntentService {
    protected ResultReceiver mReceiver;
    ...
    private void deliverResultToReceiver(int resultCode, String message) {
        Bundle bundle = new Bundle();
        bundle.putString(Constants.RESULT_DATA_KEY, message);
        mReceiver.send(resultCode, bundle);
    }
}
```

### Start the intent service

The intent service, as defined in the previous section, runs in the background and fetches the address corresponding to a given geographic location. When you start the service, the Android framework instantiates and starts the service if it isn't already running, and it creates a process if needed. If the service is already running, it remains running. Because the service extends _IntentService_, it shuts down automatically after all intents are processed.

Start the service from your app's main activity and create an Intent to pass data to the service. You need an explicit intent because you want only your service to respond to the intent. For more information, see Intent Types.

To create an explicit intent, specify the name of the class to use for the service:   

_FetchAddressIntentService.class_. Pass this information in the intent extras:

  * A _ResultReceiver_ to handle the results of the address lookup.

  * A _Location_ object containing the latitude and longitude that you want to convert to an address.

The following code sample shows you how to start the intent service:

>Caution: To ensure that your app is secure, always use an explicit intent when starting a Service and do not declare intent filters for your services. Using an implicit intent to start a service is a security hazard because you cannot be certain of the service that will respond to the intent, and the user cannot see which service starts.

Call the above _startIntentService()_ method when the user takes an action that requires a geocoding address lookup. For example, the user may press a Fetch address button on your app's UI. Before starting the intent service, you need to check that the connection to Google Play services is present. The following code snippet shows the call to the _startIntentService()_ method in the button handler:

```java
public void fetchAddressButtonHandler(View view) {
    // Only start the service to fetch the address if GoogleApiClient is
    // connected.
    if (mGoogleApiClient.isConnected() && mLastLocation != null) {
        startIntentService();
    }
    // If GoogleApiClient isn't connected, process the user's request by
    // setting mAddressRequested to true. Later, when GoogleApiClient connects,
    // launch the service to fetch the address. As far as the user is
    // concerned, pressing the Fetch Address button
    // immediately kicks off the process of getting the address.
    mAddressRequested = true;
    updateUIWidgets();
}
```

You must also start the intent service when the connection to Google Play services is established, if the user has already clicked the button on your app's UI. The following code snippet shows the call to the startIntentService() method in the onConnected() callback that is provided by the Google API Client:

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    @Override
    public void onConnected(Bundle connectionHint) {
        // Gets the best and most recent location currently available,
        // which may be null in rare cases when a location is not available.
        mLastLocation = LocationServices.FusedLocationApi.getLastLocation(
                mGoogleApiClient);

        if (mLastLocation != null) {
            // Determine whether a Geocoder is available.
            if (!Geocoder.isPresent()) {
                Toast.makeText(this, R.string.no_geocoder_available,
                        Toast.LENGTH_LONG).show();
                return;
            }

            if (mAddressRequested) {
                startIntentService();
            }
        }
    }
}
```

### Receive the geocoding results

After the intent service handles the geocoding request, it uses a _ResultReceiver_ to return the results to the activity that made the request. In the activity that makes the request, define an AddressResultReceiver that extends _ResultReceiver_ to handle the response from FetchAddressIntentService.

The result includes a numeric result code (resultCode) as well as a message containing the result data (resultData). If the reverse geocoding process is successful, the resultData contains the address. In the case of a failure, the resultData contains text describing the reason for the failure. For details of the possible errors, see Return the address to the requestor.

Override the _onReceiveResult()_ method to handle the results that are delivered to the result receiver, as shown in the following code sample:

```java
public class MainActivity extends ActionBarActivity implements
        ConnectionCallbacks, OnConnectionFailedListener {
    ...
    class AddressResultReceiver extends ResultReceiver {
        public AddressResultReceiver(Handler handler) {
            super(handler);
        }

        @Override
        protected void onReceiveResult(int resultCode, Bundle resultData) {

            // Display the address string
            // or an error message sent from the intent service.
            mAddressOutput = resultData.getString(Constants.RESULT_DATA_KEY);
            displayAddressOutput();

            // Show a toast message if an address was found.
            if (resultCode == Constants.SUCCESS_RESULT) {
                showToast(getString(R.string.address_found));
            }

        }
    }
}
```
