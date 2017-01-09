---
title: Identifying App Installations
date: 2017-01-09 20:33:42
tags:
categories: "Android官方博客"
---

![](/images/categories/android/android-developer-blog/android_developer_blog.png)

In the Android group, from time to time we hear complaints from developers about problems they’re having coming up with reliable, stable, unique device identifiers. This worries us, because we think that tracking such identifiers isn’t a good idea, and that there are better ways to achieve developers’ goals.

### Tracking Installations

It is very common, and perfectly reasonable, for a developer to want to track individual installations of their apps. It sounds plausible just to call TelephonyManager.getDeviceId() and use that value to identify the installation. There are problems with this: First, it doesn’t work reliably (see below). Second, when it does work, that value survives device wipes (“Factory resets”) and thus you could end up making a nasty mistake when one of your customers wipes their device and passes it on to another person.

To track installations, you could for example use a UUID as an identifier, and simply create a new one the first time an app runs after installation. Here is a sketch of a class named “Installation” with one static method Installation.id(Context context). You could imagine writing more installation-specific data into the INSTALLATION file.

<!--more-->

```java
public class Installation {
    private static String sID = null;
    private static final String INSTALLATION = "INSTALLATION";

    public synchronized static String id(Context context) {
        if (sID == null) {  
            File installation = new File(context.getFilesDir(), INSTALLATION);
            try {
                if (!installation.exists())
                    writeInstallationFile(installation);
                sID = readInstallationFile(installation);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
        return sID;
    }

    private static String readInstallationFile(File installation) throws IOException {
        RandomAccessFile f = new RandomAccessFile(installation, "r");
        byte[] bytes = new byte[(int) f.length()];
        f.readFully(bytes);
        f.close();
        return new String(bytes);
    }

    private static void writeInstallationFile(File installation) throws IOException {
        FileOutputStream out = new FileOutputStream(installation);
        String id = UUID.randomUUID().toString();
        out.write(id.getBytes());
        out.close();
    }
}
```

### Identifying Devices

Suppose you feel that for the needs of your application, you need an actual hardware device identifier. This turns out to be a tricky problem.

In the past, when every Android device was a phone, things were simpler: TelephonyManager.getDeviceId() is required to return (depending on the network technology) the IMEI, MEID, or ESN of the phone, which is unique to that piece of hardware.

However, there are problems with this approach:

  * <b>Non-phones</b>: Wifi-only devices or music players that don’t have telephony hardware just don’t have this kind of unique identifier.

  * <b>Persistence</b>: On devices which do have this, it persists across device data wipes and factory resets. It’s not clear at all if, in this situation, your app should regard this as the same device.

  * <b>Privilege</b>: It requires READ_PHONE_STATE permission, which is irritating if you don’t otherwise use or need telephony.

  * <b>Bugs</b>: We have seen a few instances of production phones for which the implementation is buggy and returns garbage, for example zeros or asterisks.

#### Mac Address

It may be possible to retrieve a Mac address from a device’s WiFi or Bluetooth hardware. We do not recommend using this as a unique identifier. To start with, not all devices have WiFi. Also, if the WiFi is not turned on, the hardware may not report the Mac address.

#### Serial Number

Since Android 2.3 (“Gingerbread”) this is available via android.os.Build.SERIAL. Devices without telephony are required to report a unique device ID here; some phones may do so also.

#### ANDROID_ID

More specifically, Settings.Secure.ANDROID_ID. This is a 64-bit quantity that is generated and stored when the device first boots. It is reset when the device is wiped.

ANDROID_ID seems a good choice for a unique device identifier. There are downsides: First, it is not 100% reliable on releases of Android prior to 2.2 (“Froyo”). Also, there has been at least one widely-observed bug in a popular handset from a major manufacturer, where every instance has the same ANDROID_ID.

### Conclusion

For the vast majority of applications, the requirement is to identify a particular installation, not a physical device. Fortunately, doing so is straightforward.

There are many good reasons for avoiding the attempt to identify a particular device. For those who want to try, the best approach is probably the use of ANDROID_ID on anything reasonably modern, with some fallback heuristics for legacy devices.

原文地址：

[Identifying App Installations](http://android-developers.blogspot.tw/2011/03/identifying-app-installations.html)

[如何获取Android唯一标识（唯一序列号）](http://blog.csdn.net/ljz2009y/article/details/22895297)
