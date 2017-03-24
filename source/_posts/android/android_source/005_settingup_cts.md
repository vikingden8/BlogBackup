---
title: Setting up CTS
date: 2017-03-24 23:15:46
tags:
categories: "Android Sources Website"
---

### Physical environment

#### Wi-Fi and IPv6

CTS tests require a Wi-Fi network that supports IPv6, can treat the Device Under Test (DUT) as an isolated client, and has an internet connection. An isolated client refers to a configuration where the DUT does not have visibility to the broadcast/multinetwork messages on that subnetwork, either by a Wi-Fi AP configuration or by running the DUT on an isolated sub-network without other devices being connected.

If you don't have access to a native IPv6 network, an IPv6 carrier network, or a VPN to pass some tests depending on IPv6, you may instead use a Wi-Fi access point and an IPv6 tunnel. See Wikipedia list of [IPv6 tunnel brokers](https://en.wikipedia.org/wiki/List_of_IPv6_tunnel_brokers).

<!--more-->

#### Bluetooth LE beacons

If the DUT supports the Bluetooth LE feature, then at least three Bluetooth LE beacons should be placed within five meters of the DUT for Bluetooth LE scan testing. Those beacons can be any kind, do not need to be configured or emit anything specific, and can include iBeacon, Eddystone, or even devices simulating BLE beacons.

### Desktop machine setup

CTS currently supports 64-bit Linux and Mac OS host machines.

#### ADB and AAPT

Before running the CTS, make sure you have recent versions of both [Android Debug Bridge (adb)](http://developer.android.com/tools/help/adb.html) and [Android Asset Packaging Tool (AAPT)](https://developer.android.com/guide/topics/manifest/uses-feature-element.html#testing) installed and those tools' location added to the system path of your machine.

To install ADB, download the Android SDK Tools package for your operating system, open it, and follow the instructions in the included README file. For troubleshooting information, see Installing the Stand-alone SDK Tools.

Ensure adb and aapt are in your system path. The following command assumes you've opened the package archive in your home directory:

```sh
$ export PATH=$PATH:$HOME/android-sdk-linux/build-tools/<version>
```

>Note: Please ensure your starting path and directory name are correct.

#### Java Development Kit (JDK)

Install the proper version of the Java Development Kit (JDK). For Android 7.0—

  * On Ubuntu, use OpenJDK 8.

  * On Mac OS, use jdk 8u45 or newer.

For details, see the JDK requirements.

#### CTS files

[Download](http://source.android.com/compatibility/cts/downloads.html) and open the CTS packages matching your devices' Android version and all the Application Binary Interfaces (ABIs) your devices support.

Download and open the latest version of the [CTS Media Files](http://source.android.com/compatibility/cts/downloads.html#cts-media-files).

#### Device detection

Follow the step to [set up your system to detect your device](http://developer.android.com/tools/device.html#setting-up), such as creating a udev rules file for Ubuntu Linux.

### Android device setup

#### User builds

A compatible device is defined as a device with a user/release-key signed build, so your device should be running a system image based on the known to be compatible user build (Android 4.0 and later) from Codenames, Tags, and Build Numbers.

>Caution: When used to confirm Android compatibility of your final system image, CTS must be executed on devices with a user build.

#### First API level build property

Certain CTS requirements depend on the build a device was originally shipped with. For example, devices that originally ship with earlier builds may be excluded from system requirements that apply to devices that ship with later builds.

To make this information available to CTS, device manufacturers may define the build-time property: _ro.product.first_api_level_. The value of this property is the first API level the device was commercially launched with.

OEMs can add _PRODUCT_PROPERTY_OVERRIDES_ into their device.mk file to set this property, as shown in the following example:

```sh
#ro.product.first_api_level indicates the first api level, device has been commercially launched on.
PRODUCT_PROPERTY_OVERRIDES +=\
ro.product.first_api_level=21
```

###CTS Shim apps

Android 7.0 includes the following pre-built apps (built from this [source](https://android.googlesource.com/platform/frameworks/base/+/master/packages/CtsShim/build/)) which do not contain any code except for the manifest:

  * [_frameworks/base/packages/CtsShim/CtsShim.apk_](https://android.googlesource.com/platform/frameworks/base/+/android-7.0.0_r1/packages/CtsShim/CtsShim.apk)
  This apk file is copied to **/system/app/CtsShimPrebuilt.apk** on the system image.
  * [_frameworks/base/packages/CtsShim/CtsShimPriv.apk_](https://android.googlesource.com/platform/frameworks/base/+/android-7.0.0_r1/packages/CtsShim/CtsShimPriv.apk)
  This apk file is copied to **/system/priv-app/CtsShimPrivPrebuilt.apk** on the system image.

CTS uses these apps to test privileges and permissions. To pass the tests, you must preload the apps into the appropriate directories on the system image without re-signing them.

#### Storage requirements

The CTS media stress tests require video clips to be on external storage (/sdcard). Most of the clips are from [Big Buck Bunny](https://peach.blender.org/) which is copyrighted by the Blender Foundation under the Creative Commons Attribution 3.0 license.

The required space depends on the maximum video playback resolution supported by the device (See section 5 in the compatibility definition document for the platform version of the required resolutions.) Note that the video playback capabilities of the device under test will be checked via the android.media.CamcorderProfile APIs for earlier versions of Android and the android.media.MediaCodecInfo.CodecCapabilities APIs from Android 5.0.

Here are the storage requirements by maximum video playback resolution:

  * 480x360: 98MB

  * 720x480: 193MB

  * 1280x720: 606MB

  * 1920x1080: 1863MB

#### Screen and storage

Any device that does not have an embedded screen needs to be connected to a screen.
If the device has a memory card slot, plug in an empty SD card. Use an SD card that supports Ultra High Speed (UHS) Bus with SDHC or SDXC capacity or one with at least speed class 10 or higher to ensure it can pass the CTS.

>Warning: CTS may modify/erase data on the SD card plugged into the device.
If the device has SIM card slots, plug in an activated SIM card to each slot. If the device supports SMS, each SIM card should have its own number field populated.

#### Developer UICC

In order to run CTS carrier API tests, the device needs to has a SIM card with carrier privilege rules on it. See [Preparing the UICC](http://source.android.com/devices/tech/config/uicc.html#prepare_uicc).

### Android device configuration

  1. Factory data reset the device: **Settings > Backup & reset > Factory data reset**

  >Warning: This will erase all user data from the device.

  2. Set your device's language to English (**United States**) from: **Settings > Language & input > Language**

  3. Turn on the location setting if there is a GPS or Wi-Fi / Cellular network feature on the device: **Settings > Location > On**

  4. Connect to a Wi-Fi network that supports IPv6, can treat the Device Under Test (DUT) as an isolated client (see the Physical Environment section above), and has an internet connection: **Settings > Wi-Fi**

  5. Make sure no lock pattern or password is set on the device: **Settings > Security > Screen lock > None**

  6. Enable USB debugging on your device: **Settings > Developer options > USB debugging**.

  >Note: On Android 4.2 and later, Developer options is hidden by default. To make them available, go to **Settings > About phone** and tap Build number seven times. Return to the previous screen to find Developer options. [See Enabling On-device Developer Options](https://developer.android.com/studio/run/device.html#developer-device-options) for additional details.

  7. Make sure the time is set to 12-hour format: **Settings > Date & time > Use 24-hour format > Off**

  8. Select: **Settings > Developer options > Stay Awake > On**

  9. Select: **Settings > Developer options > Allow mock locations > On**

  >Note: This mock locations setting is applicable only in Android 5.x and 4.4.x.

  10. Select: **Settings > Developer options > Verify apps over USB > Off**

  >Note: This verify apps step became required in Android 4.2.

  11. Launch the browser and dismiss any startup/setup screen.

  12. Connect the desktop machine that will be used to test the device with a USB cable

  >Note: When you connect a device running Android 4.2.2 or later to your computer, the system shows a dialog asking whether to accept an RSA key that allows debugging through this computer. Select Allow USB debugging.

  13. Install and configure helper apps on the device.

  >Note: For CTS versions 2.1 R2 through 4.2 R4, set up your device (or emulator) to run the accessibility tests with:

```sh
adb install -r android-cts/repository/testcases/CtsDelegatingAccessibilityService.apk
```

  On the device, enable: **Settings > Accessibility > Accessibility > Delegating Accessibility Service**

  >Note: For CTS versions prior to 7.0, on devices that declare android.software.device_admin, set up your device to run the device administration test using:

```sh
adb install -r android-cts/repository/testcases/CtsDeviceAdmin.apk
```

  In **Settings > Security > Select device administrators**, enable the two _android.deviceadmin.cts.CtsDeviceAdminReceiver*_ device administrators. Ensure the _android.deviceadmin.cts.CtsDeviceAdminDeactivatedReceiver_ and any other preloaded device administrators remain disabled.

  14. Copy the CTS media files to the device as follows:

  >Note: For CTS 2.3 R12 and later, if the device supports video codecs, the CTS media files must be copied to the device.

  * Navigate (cd) to the path the media files are downloaded and unzipped to.

  * Change the file permissions: _chmod u+x copy_media.sh_

  * Run _copy_media.sh_:

    * To copy clips up to a resolution of 720x480, run: _./copy_media.sh 720x480_

    * If you are not sure about the maximum resolution, try _./copy_media.sh all_ so that all files are copied.

    * If there are multiple devices under adb, add the -s (serial) option to the end. For example, to copy up to 720x480 to the device with serial 1234567, run: _./copy_media.sh 720x480 -s 1234567_
