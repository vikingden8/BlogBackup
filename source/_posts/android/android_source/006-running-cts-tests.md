---
title: Running CTS tests
date: 2017-03-24 23:46:56
tags:
categories: "Android Sources Website"
---

### Getting started with CTS tradefed

See the [Trade Federation Overview](http://source.android.com/devices/tech/test_infra/tradefed/index.html) for an explanation of the Trade Federation (tradefed or TF for short) continuous test framework.

To run a test plan:

  * Connect at least one device.

  * Press the home button to set the device to the home screen at the start of CTS.

  * While a device is running tests, it must not be used for any other tasks and must be kept in a stationary position (to avoid triggering sensor activity) with the cameras pointing at an object that could be focused.

  * Do not press any keys on the device while the CTS is running. Pressing keys or touching the screen of a test device will interfere with the running tests and may lead to test failures.

  * Launch the CTS console by running the cts-tradefed script from the folder where the CTS package has been unzipped, e.g. $ _./android-cts/tools/cts-tradefed_

  * Start the default test plan (contains all test packages) by appending: _run cts --plan CTS_ . This kicks off all CTS tests required for compatibility.

  >For CTS v1 (Android 6.0 and earlier), enter _list plans_ to view a list of test plans in the repository or _list packages_ to view a list of test packages in the repository.

  >For CTS v2 (Android 7.0 and later), enter _list modules_ to see a list of test modules.

  * Alternately, run the CTS plan of your choosing from the command line using: _cts-tradefed run cts --plan_

  >Note: When running Android 6.0 (Marshmallow) CTS only, we recommend you use the _--skip-preconditions_ option to skip the experimental pre-conditions feature that may cause issues for when executing CTS tests.

  * View test progress and results reported on the console.

  * If your device is Android 5.0 or later and declares support for an ARM and a x86 ABI, you should run both the ARM and x86 CTS packages.

### Using the CTS v1 console

For Android 6.0 or earlier, you'll use CTS v1.

#### Selecting plans

The following test plans are available:

  * **CTS**—all tests required for compatibility.

  * **Signature**—the signature verification of all public APIs

  * **Android**—tests for the Android APIs

  * **Java**—tests for the Java core library

  * **VM**—tests for ART or Dalvik

  * **Performance**—performance tests for your implementation

These can be executed with the _run cts_ command.

#### CTS v1 console command reference

![](/images/categories/android/android-sources/006/v1.png)

### Using the CTS v2 console

For Android 7.0 or later, you'll use CTS v2.

#### Selecting plans

Available test plans include the following:

  * _cts_—Runs CTS from an pre-existing CTS installation.

  * _cts-camera_— Runs CTS-camera from a pre-existing CTS installation.

  * _cts-java_— Runs Core Java Tests from a pre-existing CTS installation.

  * _cts-pdk_— Runs Tests useful on validating a PDK fusion build.

  * _everything_— Common config for Compatibility suites.

Other available configurations include the following:

  * _basic-reporters_— Configuration with basic CTS reporters.
  * _collect-tests-only_—Runs CTS from a pre-existing CTS installation.
  * _common-compatibility_-config— Common config for Compatibility suites.
  * _cts-filtered-sample_— Common config for Compatibility suites.
  * _cts-known-failures_— Configuration with CTS known failures.
  * _cts-preconditions_— CTS precondition configs.
  * _host_— Runs a single host-based test on an existing device.
  * _instrument_— Runs a single Android instrumentation test on an existing device.
  * _native-benchmark_— Runs a native stress test on an existing device.
  * _native-stress_— Runs a native stress test on an existing device.
  * _recharge_— A fake test that waits for nearly-discharged devices and holds them for charging.
  * _testdef_— Runs tests contained in test_def.xml files on an existing device.
  * _util/wifi_— Utility config to configure Wi-Fi on device.
  * _util/wipe_— Wipes user data on device.

All of these plans and configs can be executed with the run cts command.

#### CTS v2 console command reference

![](/images/categories/android/android-sources/006/v2.png)
