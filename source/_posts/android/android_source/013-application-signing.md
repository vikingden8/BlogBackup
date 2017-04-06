---
title: Application Signing
date: 2017-04-06 21:40:56
tags:
categories: "Android Sources Website"
---

Application signing allows developers to identify the author of the application and to update their application without creating complicated interfaces and permissions. Every application that is run on the Android platform must be signed by the developer. Applications that attempt to install without being signed will be rejected by either Google Play or the package installer on the Android device.

On Google Play, application signing bridges the trust Google has with the developer and the trust the developer has with their application. Developers know their application is provided, unmodified, to the Android device; and developers can be held accountable for behavior of their application.

On Android, application signing is the first step to placing an application in its Application Sandbox. The signed application certificate defines which user ID is associated with which application; different applications run under different user IDs. Application signing ensures that one application cannot access any other application except through well-defined IPC.

<!--more-->

When an application (APK file) is installed onto an Android device, the Package Manager verifies that the APK has been properly signed with the certificate included in that APK. If the certificate (or, more accurately, the public key in the certificate) matches the key used to sign any other APK on the device, the new APK has the option to specify in the manifest that it will share a UID with the other similarly-signed APKs.

Applications can be signed by a third-party (OEM, operator, alternative market) or self-signed. Android provides code signing using self-signed certificates that developers can generate without external assistance or permission. Applications do not have to be signed by a central authority. Android currently does not perform CA verification for application certificates.

Applications are also able to declare security permissions at the Signature protection level, restricting access only to applications signed with the same key while maintaining distinct UIDs and Application Sandboxes. A closer relationship with a shared Application Sandbox is allowed via the shared UID feature where two or more applications signed with same developer key can declare a shared UID in their manifest.

### APK signing schemes

Android supports two application signing schemes, one based on JAR signing (v1 scheme) and APK Signature Scheme v2 (v2 scheme), which was introduced in Android Nougat (Android 7.0).

For maximum compatibility, applications should be signed both with v1 and v2 schemes. Android Nougat and newer devices install apps signed with v2 scheme more quickly than those signed only with v1 scheme. Older Android platforms ignore v2 signatures and thus need apps to contain v1 signatures.

#### JAR signing (v1 scheme)

APK signing has been a part of Android from the beginning. It is based on signed JAR. For details on using this scheme, see the Android Studio documentation on Signing your app.

v1 signatures do not protect some parts of the APK, such as ZIP metadata. The APK verifier needs to process lots of untrusted (not yet verified) data structures and then discard data not covered by the signatures. This offers a sizeable attack surface. Moreover, the APK verifier must uncompress all compressed entries, consuming more time and memory. To address these issues, Android 7.0 introduced APK Signature Scheme v2.

#### APK Signature Scheme v2 (v2 scheme)

Android 7.0 introduces APK signature scheme v2 (v2 scheme). The contents of the APK are hashed and signed, then the resulting APK Signing Block is inserted into the APK. For details on applying the v2 scheme to an application, refer to APK Signature Scheme v2 in the Android N Developer Preview.

During validation, v2 scheme treats the APK file as a blob and performs signature checking across the entire file. Any modification to the APK, including ZIP metadata modifications, invalidates the APK signature. This form of APK verification is substantially faster and enables detection of more classes of unauthorized modifications.

The new format is backwards compatible, so APKs signed with the new signature format can be installed on older Android devices (which simply ignore the extra data added to the APK), as long as these APKs are also v1-signed.

![](/images/categories/android/android-sources/013/apk-validation-process.png)

Whole-file hash of the APK is verified against the v2 signature stored in the APK Signing Block. The hash covers everything except the APK Signing Block, which contains the v2 signature. Any modification to the APK outside of the APK Signing Block invalidates the APK's v2 signature. APKs with stripped v2 signature are rejected as well, because their v1 signature specifies that the APK was v2-signed, which makes Android Nougat and newer refuse to verify APKs using their v1 signatures.

For details on the APK signature verification process, see the Verification section of APK Signature Scheme v2.
