---
title: Application security
date: 2017-04-05 21:56:22
tags:
categories: "Android Sources Website"
---

### Elements of Applications

Android provides an open source platform and application environment for mobile devices. The core operating system is based on the Linux kernel. Android applications are most often written in the Java programming language and run in the Dalvik virtual machine. However, applications can also be written in native code. Applications are installed from a single file with the .apk file extension.

The main Android application building blocks are:

  * **AndroidManifest.xml**: The AndroidManifest.xml file is the control file that tells the system what to do with all the top-level components (specifically activities, services, broadcast receivers, and content providers described below) in an application. This also specifies which permissions are required.

  * **Activities**: An Activity is, generally, the code for a single, user-focused task. It usually includes displaying a UI to the user, but it does not have to -- some Activities never display UIs. Typically, one of the application's Activities is the entry point to an application.

  * **Services**: A Service is a body of code that runs in the background. It can run in its own process, or in the context of another application's process. Other components "bind" to a Service and invoke methods on it via remote procedure calls. An example of a Service is a media player: even when the user quits the media-selection UI, the user probably still intends for music to keep playing. A Service keeps the music going even when the UI has completed.

  * **Broadcast Receiver**: A BroadcastReceiver is an object that is instantiated when an IPC mechanism known as an Intent is issued by the operating system or another application. An application may register a receiver for the low battery message, for example, and change its behavior based on that information.

<!--more-->
### The Android Permission Model: Accessing Protected APIs

All applications on Android run in an Application Sandbox, described earlier in this document. By default, an Android application can only access a limited range of system resources. The system manages Android application access to resources that, if used incorrectly or maliciously, could adversely impact the user experience, the network, or data on the device.

These restrictions are implemented in a variety of different forms. Some capabilities are restricted by an intentional lack of APIs to the sensitive functionality (e.g. there is no Android API for directly manipulating the SIM card). In some instances, separation of roles provides a security measure, as with the per-application isolation of storage. In other instances, the sensitive APIs are intended for use by trusted applications and protected through a security mechanism known as Permissions.

These protected APIs include:

  * Camera functions

  * Location data (GPS)

  * Bluetooth functions

  * Telephony functions

  * SMS/MMS functions

  * Network/data connections

These resources are only accessible through the operating system. To make use of the protected APIs on the device, an application must define the capabilities it needs in its manifest. When preparing to install an application, the system displays a dialog to the user that indicates the permissions requested and asks whether to continue the installation. If the user continues with the installation, the system accepts that the user has granted all of the requested permissions. The user can not grant or deny individual permissions -- the user must grant or deny all of the requested permissions as a block.

Once granted, the permissions are applied to the application as long as it is installed. To avoid user confusion, the system does not notify the user again of the permissions granted to the application, and applications that are included in the core operating system or bundled by an OEM do not request permissions from the user. Permissions are removed if an application is uninstalled, so a subsequent re-installation will again result in display of permissions.

Within the device settings, users are able to view permissions for applications they have previously installed. Users can also turn off some functionality globally when they choose, such as disabling GPS, radio, or wi-fi.

In the event that an application attempts to use a protected feature which has not been declared in the application's manifest, the permission failure will typically result in a security exception being thrown back to the application. Protected API permission checks are enforced at the lowest possible level to prevent circumvention. An example of the user messaging when an application is installed while requesting access to protected APIs is shown in Figure 2.

The system default permissions are described at [https://developer.android.com/reference/android/Manifest.permission.html](https://developer.android.com/reference/android/Manifest.permission.html). Applications may declare their own permissions for other applications to use. Such permissions are not listed in the above location.

When defining a permission a protectionLevel attribute tells the system how the user is to be informed of applications requiring the permission, or who is allowed to hold a permission. Details on creating and using application specific permissions are described at [https://developer.android.com/guide/topics/security/security.html](https://developer.android.com/guide/topics/security/security.html).

There are some device capabilities, such as the ability to send SMS broadcast intents, that are not available to third-party applications, but that may be used by applications pre-installed by the OEM. These permissions use the signatureOrSystem permission.

### How Users Understand Third-Party Applications

Android strives to make it clear to users when they are interacting with third-party applications and inform the user of the capabilities those applications have. Prior to installation of any application, the user is shown a clear message about the different permissions the application is requesting. After install, the user is not prompted again to confirm any permissions.

There are many reasons to show permissions immediately prior to installation time. This is when user is actively reviewing information about the application, developer, and functionality to determine whether it matches their needs and expectations. It is also important that they have not yet established a mental or financial commitment to the app, and can easily compare the application to other alternative applications.

Some other platforms use a different approach to user notification, requesting permission at the start of each session or while applications are in use. The vision of Android is to have users switching seamlessly between applications at will. Providing confirmations each time would slow down the user and prevent Android from delivering a great user experience. Having the user review permissions at install time gives the user the option to not install the application if they feel uncomfortable.

Also, many user interface studies have shown that over-prompting the user causes the user to start saying "OK" to any dialog that is shown. One of Android's security goals is to effectively convey important security information to the user, which cannot be done using dialogs that the user will be trained to ignore. By presenting the important information once, and only when it is important, the user is more likely to think about what they are agreeing to.

Some platforms choose not to show any information at all about application functionality. That approach prevents users from easily understanding and discussing application capabilities. While it is not possible for all users to always make fully informed decisions, the Android permissions model makes information about applications easily accessible to a wide range of users. For example, unexpected permissions requests can prompt more sophisticated users to ask critical questions about application functionality and share their concerns in places such as Google Play where they are visible to all users.

### Interprocess Communication
Processes can communicate using any of the traditional UNIX-type mechanisms. Examples include the filesystem, local sockets, or signals. However, the Linux permissions still apply.

Android also provides new IPC mechanisms:

  * **Binder**: A lightweight capability-based remote procedure call mechanism designed for high performance when performing in-process and cross-process calls. Binder is implemented using a custom Linux driver. See https://developer.android.com/reference/android/os/Binder.html.

  * **Services**: Services (discussed above) can provide interfaces directly accessible using binder.

  * **Intents**: An Intent is a simple message object that represents an "intention" to do something. For example, if your application wants to display a web page, it expresses its "Intent" to view the URL by creating an Intent instance and handing it off to the system. The system locates some other piece of code (in this case, the Browser) that knows how to handle that Intent, and runs it. Intents can also be used to broadcast interesting events (such as a notification) system-wide. See https://developer.android.com/reference/android/content/Intent.html.

  * **ContentProviders**: A ContentProvider is a data storehouse that provides access to data on the device; the classic example is the ContentProvider that is used to access the user's list of contacts. An application can access data that other applications have exposed via a ContentProvider, and an application can also define its own ContentProviders to expose data of its own. See https://developer.android.com/reference/android/content/ContentProvider.html.

While it is possible to implement IPC using other mechanisms such as network sockets or world-writable files, these are the recommended Android IPC frameworks. Android developers will be encouraged to use best practices around securing users' data and avoiding the introduction of security vulnerabilities.

### Cost-Sensitive APIs

A cost sensitive API is any function that might generate a cost for the user or the network. The Android platform has placed cost sensitive APIs in the list of protected APIs controlled by the OS. The user will have to grant explicit permission to third-party applications requesting use of cost sensitive APIs. These APIs include:

  * Telephony

  * SMS/MMS

  * Network/Data

  * In-App Billing

  * NFC Access

Android 4.2 adds further control on the use of SMS. Android will provide a notification if an application attempts to send SMS to a short code that uses premium services which might cause additional charges. The user can choose whether to allow the application to send the message or block it.

### SIM Card Access

Low level access to the SIM card is not available to third-party apps. The OS handles all communications with the SIM card including access to personal information (contacts) on the SIM card memory. Applications also cannot access AT commands, as these are managed exclusively by the Radio Interface Layer (RIL). The RIL provides no high level APIs for these commands.

### Personal Information

Android has placed APIs that provide access to user data into the set of protected APIs. With normal usage, Android devices will also accumulate user data within third-party applications installed by users. Applications that choose to share this information can use Android OS permission checks to protect the data from third-party applications.

![](/images/categories/android/android-sources/011/permissions_check.png)

System content providers that are likely to contain personal or personally identifiable information such as contacts and calendar have been created with clearly identified permissions. This granularity provides the user with clear indication of the types of information that may be provided to the application. During installation, a third-party application may request permission to access these resources. If permission is granted, the application can be installed and will have access to the data requested at any time when it is installed.

Any applications which collect personal information will, by default, have that data restricted only to the specific application. If an application chooses to make the data available to other applications though IPC, the application granting access can apply permissions to the IPC mechanism that are enforced by the operating system.

### Sensitive Data Input Devices

Android devices frequently provide sensitive data input devices that allow applications to interact with the surrounding environment, such as camera, microphone or GPS. For a third-party application to access these devices, it must first be explicitly provided access by the user through the use of Android OS Permissions. Upon installation, the installer will prompt the user requesting permission to the sensor by name.

If an application wants to know the user's location, the application requires a permission to access the user's location. Upon installation, the installer will prompt the user asking if the application can access the user's location. At any time, if the user does not want any application to access their location, then the user can run the "Settings" application, go to "Location & Security", and uncheck the "Use wireless networks" and "Enable GPS satellites". This will disable location based services for all applications on the user's device.

### Device Metadata

Android also strives to restrict access to data that is not intrinsically sensitive, but may indirectly reveal characteristics about the user, user preferences, and the manner in which they use a device.

By default applications do not have access to operating system logs, browser history, phone number, or hardware / network identification information. If an application requests access to this information at install time, the installer will prompt the user asking if the application can access the information. If the user does not grant access, the application will not be installed.

### Certificate authorities

Android includes a set of installed system Certificate Authorities, which are trusted system-wide. Prior to Android 7.0, device manufacturers could modify the set of CAs shipped on their devices. However, devices running 7.0 and above will have a uniform set of system CAs as modification by device manufacturers is no longer permitted.

To be added as a new public CA to the Android stock set, the CA must complete the Mozilla CA Inclusion Process and then file a feature request against Android (https://code.google.com/p/android/issues/entry) to have the CA added to the stock Android CA set in the Android Open Source Project (AOSP).

There are still CAs that are device-specific and should not be included in the core set of AOSP CAs, like carriers’ private CAs that may be needed to securely access components of the carrier’s infrastructure, such as SMS/MMS gateways. Device manufacturers are encouraged to include the private CAs only in the components/apps that need to trust these CAs. See Network Security Configuration for more details.

### Application Signing

Code signing allows developers to identify the author of the application and to update their application without creating complicated interfaces and permissions. Every application that is run on the Android platform must be signed by the developer. Applications that attempt to install without being signed will rejected by either Google Play or the package installer on the Android device.

On Google Play, application signing bridges the trust Google has with the developer and the trust the developer has with their application. Developers know their application is provided, unmodified to the Android device; and developers can be held accountable for behavior of their application.

On Android, application signing is the first step to placing an application in its Application Sandbox. The signed application certificate defines which user id is associated with which application; different applications run under different user IDs. Application signing ensures that one application cannot access any other application except through well-defined IPC.

When an application (APK file) is installed onto an Android device, the Package Manager verifies that the APK has been properly signed with the certificate included in that APK. If the certificate (or, more accurately, the public key in the certificate) matches the key used to sign any other APK on the device, the new APK has the option to specify in the manifest that it will share a UID with the other similarly-signed APKs.

Applications can be signed by a third-party (OEM, operator, alternative market) or self-signed. Android provides code signing using self-signed certificates that developers can generate without external assistance or permission. Applications do not have to be signed by a central authority. Android currently does not perform CA verification for application certificates.

Applications are also able to declare security permissions at the Signature protection level, restricting access only to applications signed with the same key while maintaining distinct UIDs and Application Sandboxes. A closer relationship with a shared Application Sandbox is allowed via the shared UID feature where two or more applications signed with same developer key can declare a shared UID in their manifest.

### Application Verification

Android 4.2 and later support application verification. Users can choose to enable “Verify Apps" and have applications evaluated by an application verifier prior to installation. App verification can alert the user if they try to install an app that might be harmful; if an application is especially bad, it can block installation.

### Digital Rights Management
The Android platform provides an extensible DRM framework that lets applications manage rights-protected content according to the license constraints that are associated with the content. The DRM framework supports many DRM schemes; which DRM schemes a device supports is left to the device manufacturer.

The Android DRM framework is implemented in two architectural layers (see figure below):

  * A DRM framework API, which is exposed to applications through the Android application framework and runs through the Dalvik VM for standard applications.

  * A native code DRM manager, which implements the DRM framework and exposes an interface for DRM plug-ins (agents) to handle rights management and decryption for various DRM schemes

![](/images/categories/android/android-sources/011/ape_fwk_drm_2.png)
