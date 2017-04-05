---
title: Implementing security
date: 2017-04-05 22:18:54
tags:
categories: "Android Sources Website"
---

The Android Security Team regularly receives requests for information about preventing potential security issues on Android devices. We also occasionally spot check devices and let device manufacturers and affected partners know of potential issues.

This page provides security best practices based on our experiences, extending the Designing for Security documentation we've provided for developers and includes details unique to building or installing system-level software on devices.

To facilitate adoption of these best practices, where possible the Android Security Team incorporates tests into the Android Compatibility Test Suite (CTS) and Android Lint. We encourage partners to contribute tests that can help other Android users (view security-related tests at _root/cts/tests/tests/security/src/android/security/cts_).

<!--more-->

### Development process

Use the following best practices in your development processes and environment.

#### Reviewing source code

Source code review can detect a broad range of security issues, including those identified in this document. Android strongly encourages both manual and automated source code review. Best practices:

  * Run Android Lint on all application code using the Android SDK and correct any identified issues.

  * Native code should be analyzed using an automated tool that can detect memory management issues such as buffer overflows and off-by-one errors.

#### Using automated testing

Automated testing can detect a broad range of security issues, including several issues discussed below. Best practices:

  * CTS is regularly updated with security tests; run the most recent version of CTS to verify compatibility.

  * Run CTS regularly throughout the development process to detect problems early and reduce time to correction. Android uses CTS as part of continuous integration in our automated build process, which builds multiple times per day.

  * Device manufacturers should automate security testing of interfaces, including testing with malformed inputs (fuzz testing).

#### Signing system images

The signature of the system image is critical for determining the integrity of the device. Best practices:

  * Devices must not be signed with a key that is publicly known.

  * Keys used to sign devices should be managed in a manner consistent with industry standard practices for handling sensitive keys, including a hardware security module (HSM) that provides limited, auditable access.

#### Signing applications (APKs)

Application signatures play an important role in device security and are used for permissions checks as well as software updates. When selecting a key to use for signing applications, it is important to consider whether an application will be available only on a single device or common across multiple devices. Best practices:

  * Applications must not be signed with a key that is publicly known.

  * Keys used to sign applications should be managed in a manner consistent with industry standard practices for handling sensitive keys, including an HSM that provides limited, auditable access.

  * Applications should not be signed with the platform key.

  * Applications with the same package name should not be signed with different keys. This often occurs when creating an application for different devices, especially when using the platform key. If the application is device-independent, use the same key across devices. If the application is device-specific, create unique package names per device and key.

#### Publishing applications

Google Play provides device manufacturers the ability to update applications without performing a complete system update. This can expedite response to security issues and delivery of new features, as well as provide a way to ensure your application has a unique package name. Best practices:

  * Upload your applications to Google Play to allow automated updates without requiring a full over-the-air (OTA) update. Applications that are uploaded but unpublished are not directly downloadable by users but the applications are still updated. Users who have previously installed the app can re-install it and/or install on other devices.

  * Create an application package name that is clearly associated with your company, such as by using a company trademark.

  * Applications published by device manufacturers should be uploaded on the Google Play store to avoid package name impersonation by third-party users. If a device manufacturer installs an app on a device without publishing the app on the Play store, another developer could upload the same app, use the same package name, and change the metadata for the app. When the app is presented to the user, this unrelated metadata could create confusion.

#### Responding to incidents

External parties must have the ability to contact device manufacturers about device-specific security issues. We recommend creating a publicly accessible email address for managing security incidents. Best practices:

  * Create a security@your-company.com or similar address and publicize it.

  * If you become aware of a security issue affecting Android OS or Android devices from multiple device manufacturers, you should contact the Android Security Team by filing a Security bug report.

### Product implementation

Use the following best practices when implementing a product.

#### Isolating root processes

Root processes are the most frequent target of privilege escalation attacks, so reducing the number of root processes reduces risk of privilege escalation. CTS includes an informational test that lists root processes. Best practices:

  * Devices should run the minimum necessary code as root. Where possible, use a regular Android process rather than a root process. The ICS Galaxy Nexus has only six root processes: vold, inetd, zygote, tf_daemon, ueventd, and init. If a process must run as root on a device, document the process in an AOSP feature request so it can be publicly reviewed.

  * Where possible, root code should be isolated from untrusted data and accessed via IPC. For example, reduce root functionality to a small Service accessible via Binder and expose the Service with signature permission to an application with low or no privileges to handle network traffic.

  * Root processes must not listen on a network socket.

  * Root processes must not provide a general-purpose runtime for applications (e.g. a Java VM).

#### Isolating system apps

In general, pre-installed apps should not run with the shared system UID. If is it necessary, however, for an app to use the shared UID of system or another privileged service (i.e., phone), the app should not export any services, broadcast receivers, or content providers that can be accessed by third-party apps installed by users. Best practices:

  * Devices should run the minimum necessary code as system. Where possible, use an Android process with its own UID rather than reusing the system UID.

  * Where possible, system code should be isolated from untrusted data and expose IPC only to other trusted processes.

  * System processes must not listen on a network socket.

#### Isolating processes

The Android Application Sandbox provides applications with an expectation of isolation from other processes on the system, including root processes and debuggers. Unless debugging is specifically enabled by the application and the user, no application should violate that expectation. Best practices:

  * Root processes must not access data within individual application data folders, unless using a documented Android debugging method.

  * Root processes must not access memory of applications, unless using a documented Android debugging method.

  * Devices must not include any application that accesses data or memory of other applications or processes.

#### Securing SUID files

New setuid programs should not be accessible by untrusted programs. Setuid programs have frequently been the location of vulnerabilities that can be used to gain root access, so strive to minimize the availability of the setuid program to untrusted applications. Best practices:

  * SUID processes must not provide a shell or backdoor that can be used to circumvent the Android security model.

  * SUID programs must not be writable by any user.

  * SUID programs should not be world readable or executable. Create a group, limit access to the SUID binary to members of that group, and place any applications that should be able to execute the SUID program into that group.

  * SUID programs are a common source of user rooting of devices. To reduce this risk, SUID programs should not be executable by the shell user.

CTS verifier includes an informational test listing SUID files; some setuid files are not permitted per CTS tests.

#### Securing listening sockets

CTS tests fail when a device is listening on any port, on any interface. In the event of a failure, Android verifies the following best practices are in use:

  * There should be no listening ports on the device.

  * Listening ports must be able to be disabled without an OTA. This can be either a server or user-device configuration change.

  * Root processes must not listen on any port.

  * Processes owned by the system UID must not listen on any port.

  * For local IPC using sockets, applications must use a UNIX Domain Socket with access limited to a group. Create a file descriptor for the IPC and make it +RW for a specific UNIX group. Any client applications must be within that UNIX group.

  * Some devices with multiple processors (e.g. a radio/modem separate from the application processor) use network sockets to communicate between processors. In such instances, the network socket used for inter-processor communication must use an isolated network interface to prevent access by unauthorized applications on the device (i.e. use iptables to prevent access by other applications on the device).

  * Daemons that handle listening ports must be robust against malformed data. Google may conduct fuzz-testing against the port using an unauthorized client, and, where possible, authorized client. Any crashes will be filed as bugs with an appropriate severity.

#### Logging data

Logging data increases the risk of exposure of that data and reduces system performance. Multiple public security incidents have occurred as the result of logging sensitive user data by applications installed by default on Android devices. Best practices:

  * Applications or system services should not log data provided from third-party applications that might include sensitive information.

  * Applications must not log any Personally Identifiable Information (PII) as part of normal operation.

CTS includes tests that check for the presence of potentially sensitive information in the system logs.

#### Limiting directory access

World-writable directories can introduce security weaknesses and enable an application to rename trusted files, substitute files, or conduct symlink-based attacks (attackers may use a symlink to a file to trick a trusted program into performing actions it shouldn't). Writable directories can also prevent the uninstall of an application from properly cleaning up all files associated with an application.

As a best practice, directories created by the system or root users should not be world writable. CTS tests help enforce this best practice by testing known directories.

#### Securing configuration files

Many drivers and services rely on configuration and data files stored in directories such as /system/etc and /data. If these files are processed by a privileged process and are world writable, it is possible for an app to exploit a vulnerability in the process by crafting malicious contents in the world-writable file. Best practices:

  * Configuration files used by privileged processes should not be world readable.

  * Configuration files used by privileged processes must not be world writable.

#### Storing native code libraries

Any code used by privileged device manufacturer processes must be in /vendor or /system; these filesystems are mounted read-only on boot. As a best practice, libraries used by system or other highly-privileged apps installed on the phone should also be in these filesystems. This can prevent a security vulnerability that could allow an attacker to control the code that a privileged process executes.

#### Limiting device driver access

Only trusted code should have direct access to drivers. Where possible, the preferred architecture is to provide a single-purpose daemon that proxies calls to the driver and restricts driver access to that daemon. As a best practice, driver device nodes should not be world readable or writable. CTS tests help enforce this best practice by checking for known instances of exposed drivers.

#### Disabling ADB

Android debug bridge (adb) is a valuable development and debugging tool, but is designed for use in controlled, secure environments and should not be enabled for general use. Best practices:

  * ADB must be disabled by default.

  * ADB must require the user to turn it on before accepting connections.

#### Unlocking bootloaders

Many Android devices support unlocking, enabling the device owner to modify the system partition and/or install a custom operating system. Common use cases include installing a third-party ROM and performing systems-level development on the device. For example, a Google Nexus device owner can run fastboot oem unlock to start the unlocking process, which presents the following message to the user:

>Unlock bootloader?

>If you unlock the bootloader, you will be able to install custom operating system software on this phone.

>A custom OS is not subject to the same testing as the original OS, and can cause your phone and installed applications to stop working properly.

>To prevent unauthorized access to your personal data, unlocking the bootloader will also delete all personal data from your phone (a "factory data reset").

>Press the Volume Up/Down buttons to select Yes or No. Then press the Power button to continue.

>Yes: Unlock bootloader (may void warranty)

>No: Do not unlock bootloader and restart phone.

As a best practice, unlockable Android devices must securely erase all user data prior to being unlocked. Failure to properly delete all data on unlocking may allow a physically proximate attacker to gain unauthorized access to confidential Android user data. To prevent the disclosure of user data, a device that supports unlocking must implement it properly (we've seen numerous instances where device manufacturers improperly implemented unlocking). A properly implemented unlocking process has the following properties:

  * When the unlocking command is confirmed by the user, the device must start an immediate data wipe. The unlocked flag must not be set until after the secure deletion is complete.

  * If a secure deletion cannot be completed, the device must stay in a locked state.

  * If supported by the underlying block device, ioctl(BLKSECDISCARD) or equivalent should be used. For eMMC devices, this means using a Secure Erase or Secure Trim command. For eMMC 4.5 and later, this means using a normal Erase or Trim followed by a Sanitize operation.

  * If BLKSECDISCARD is not supported by the underlying block device, ioctl(BLKDISCARD) must be used instead. On eMMC devices, this is a normal Trim operation.

  * If BLKDISCARD is not supported, overwriting the block devices with all zeros is acceptable.

  * An end user must have the option to require that user data be wiped prior to flashing a partition. For example, on Nexus devices, this is done via the fastboot oem lock command.

  * A device may record, via efuses or similar mechanism, whether a device was unlocked and/or relocked.

These requirements ensure that all data is destroyed upon the completion of an unlock operation. Failure to implement these protections is considered a moderate level security vulnerability.

A device that is unlocked may be subsequently relocked using the fastboot oem lock command. Locking the bootloader provides the same protection of user data with the new custom OS as was available with the original device manufacturer OS (e.g. user data will be wiped if the device is unlocked again).
