---
title: Use adb by TCP 通过网络连接ADB
date: 2017-03-26 10:00:42
tags:
categories: "Android学习笔记"
---

在ADB的文档 [platform/system/core/adb/OVERVIEW.TXT](https://android.googlesource.com/platform/system/core/+/android-4.4_r1/adb/OVERVIEW.TXT) 介绍中，有提到这样一段：

> An ADB transport models a connection between the ADB server and one device
    or emulator. There are currently two kinds of transports:

>   - USB transports, for physical devices through USB

>   - Local transports, for emulators running on the host, connected to the server through TCP

也就是说当前adb支持两种传输机制，第一种是USB传输，即通过USB连接的物理设备；另一种使用本地传输，比如模拟器使用的是TCP传输。

实际上，在物理设备上，也可以让adb通过TCP协议来连接设备（当然前提条件是你的设备要有网口）。

<!--more-->

首先看下这段源代码，出自 [platform/system/core/adb/adb.c](https://android.googlesource.com/platform/system/core/+/android-4.4_r1/adb/adb.c) :

```c
int usb = 0;
if (access(USB_ADB_PATH, F_OK) == 0 || access(USB_FFS_ADB_EP0, F_OK) == 0) {
    // listen on USB
    usb_init();
    usb = 1;
}
// If one of these properties is set, also listen on that port
// If one of the properties isn't set and we couldn't listen on usb,
// listen on the default port.
property_get("service.adb.tcp.port", value, "");
if (!value[0]) {
    property_get("persist.adb.tcp.port", value, "");
}
if (sscanf(value, "%d", &port) == 1 && port > 0) {
    printf("using port=%d\n", port);
    // listen on TCP port specified by service.adb.tcp.port property
    local_init(port);
} else if (!usb) {
    // listen on default port
    local_init(DEFAULT_ADB_LOCAL_TRANSPORT_PORT);
}
```

从上面的源代码可以看出，如果有USB设备存在，则使用USB作为连接方式；然后再判断是否有设置 _service.adb.tcp.port_ ,如果设置了，则使用TCP作为连接方式。如果没有设置，并且没有USB设备存在，则选择默认的TCP连接方式。

其中USB_ADB_PATH，USB_FFS_ADB_EP0， DEFAULT_ADB_LOCAL_TRANSPORT_PORT 在 [platform/system/core/adb/adb.h](https://android.googlesource.com/platform/system/core/+/android-4.4_r1/adb/adb.h) 有定义：

```c
...
#define DEFAULT_ADB_LOCAL_TRANSPORT_PORT 5555
...
#if !ADB_HOST
#define USB_ADB_PATH     "/dev/android_adb"
#define USB_FFS_ADB_PATH  "/dev/usb-ffs/adb/"
#define USB_FFS_ADB_EP(x) USB_FFS_ADB_PATH#x
#define USB_FFS_ADB_EP0   USB_FFS_ADB_EP(ep0)
#define USB_FFS_ADB_OUT   USB_FFS_ADB_EP(ep1)
#define USB_FFS_ADB_IN    USB_FFS_ADB_EP(ep2)
#endif
...
```

因此只需要在启动adbd之前设置service.adb.tcp.port，就可以让adbd选则TCP模式，也就可以通过网络来连接adb了。这需要修改init.rc文件。如果不想修改，也可以在系统启动之后，在控制台上执行下列命令：

```sh
stop adbd
set service.adb.tcp.port 6666
start adbd
```

>注：上述命令的执行需要root权限

然后，就可以在HOST主机上通过下列命令来连接设备了：

```sh
adb connect <ip-of-device>:6666
```

>注：需保证HOST主机和设备在同一个网段，执行上述命令之后就可以通过无USB的方式操作设备了，跟通过USB的连接方式无异。
