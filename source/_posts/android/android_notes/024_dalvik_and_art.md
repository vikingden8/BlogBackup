---
title: ART and Dalvik
date: 2017-3-3 10:20:37
tags:
categories: "Android学习笔记"
---

## [ART and Dalvik](https://source.android.com/devices/tech/dalvik/index.html)

Android runtime (ART) is the managed runtime used by applications and some system services on Android. ART and its predecessor Dalvik were originally created specifically for the Android project. ART as the runtime executes the Dalvik Executable format and Dex bytecode specification.

ART and Dalvik are compatible runtimes running Dex bytecode, so apps developed for Dalvik should work when running with ART. However, some techniques that work on Dalvik do not work on ART. For information about the most important issues, see Verifying App Behavior on the Android Runtime (ART).

### ART Features

Here are some of the major features implemented by ART.

#### Ahead-of-time (AOT) compilation

ART introduces ahead-of-time (AOT) compilation, which can improve app performance. ART also has tighter install-time verification than Dalvik.

At install time, ART compiles apps using the on-device dex2oat tool. This utility accepts DEX files as input and generates a compiled app executable for the target device. The utility should be able to compile all valid DEX files without difficulty. However, some post-processing tools produce invalid files that may be tolerated by Dalvik but cannot be compiled by ART. For more information, see Addressing Garbage Collection Issues.

#### Improved garbage collection

Garbage collection (GC) can impair an app's performance, resulting in choppy display, poor UI responsiveness, and other problems. ART improves garbage collection in several ways:

  * One GC pause instead of two

  * Parallelized processing during the remaining GC pause

  * Collector with lower total GC time for the special case of cleaning up recently-allocated, short-lived objects

  * Improved garbage collection ergonomics, making concurrent garbage collections more timely, which makes GC_FOR_ALLOC events extremely rare in typical use cases

  * Compacting GC to reduce background memory usage and fragmentation

<!--more-->

#### Development and debugging improvements

ART offers a number of features to improve app development and debugging.

##### Support for sampling profiler

Historically, developers have used the Traceview tool (designed for tracing application execution) as a profiler. While Traceview gives useful information, its results on Dalvik have been skewed by the per-method-call overhead, and use of the tool noticeably affects run time performance.

ART adds support for a dedicated sampling profiler that does not have these limitations. This gives a more accurate view of app execution without significant slowdown. Sampling support was added to Traceview for Dalvik in the KitKat release.

##### Support for more debugging features

ART supports a number of new debugging options, particularly in monitor- and garbage collection-related functionality. For example, you can:

  * See what locks are held in stack traces, then jump to the thread that holds a lock.

  * Ask how many live instances there are of a given class, ask to see the instances, and see what references are keeping an object live.

  * Filter events (like breakpoint) for a specific instance.

  * See the value returned by a method when it exits (using “method-exit” events).

  * Set field watchpoint to suspend the execution of a program when a specific field is accessed and/or modified.

##### Improved diagnostic detail in exceptions and crash reports

ART gives you as much context and detail as possible when runtime exceptions occur. ART provides expanded exception detail for java.lang.ClassCastException, java.lang.ClassNotFoundException, and java.lang.NullPointerException. (Later versions of Dalvik provided expanded exception detail for java.lang.ArrayIndexOutOfBoundsException and java.lang.ArrayStoreException, which now include the size of the array and the out-of-bounds offset, and ART does this as well.)

For example, java.lang.NullPointerException now shows information about what the app was trying to do with the null pointer, such as the field the app was trying to write to, or the method it was trying to call. Here are some typical examples:

```java
java.lang.NullPointerException: Attempt to write to field 'int
android.accessibilityservice.AccessibilityServiceInfo.flags' on a null object
reference
java.lang.NullPointerException: Attempt to invoke virtual method
'java.lang.String java.lang.Object.toString()' on a null object reference
```

ART also provides improved context information in app native crash reports, by including both Java and native stack information.

## [What is difference between DVM and ART?](http://stackoverflow.com/questions/31957568/what-is-difference-between-dvm-and-art-why-dvm-has-been-officially-replaced-wi)

There are some **major performance improvements that ART brings** which were lacking in Dalvik. But every pros have some cons too. I will try to discuss both the advantages and disadvantages here.

### Compilation Approach

This is by far the biggest advantage of ART over Dalvik. **The old guy Dalvik used Just-In-Time (JIT) approach** in which the compilation was done on demand. All the dex files were converted into their respective native representations only when it was needed.

But **ART uses the Ahead-Of-Time (AOT) approach**, in which the dex files were compiled before they were demanded. This itself massively improves the performance and battery life of any Android device.

For example

In case of Dalvik, whenever you touch an app icon to open it, the necessary dex files gets converted into their equivalent native codes. The app will only start working when this compilation is done. So, the app is unresponsive until this finishes.

Moreover, **this process is repeated every single time you open an app wasting CPU cycles and valuable battery juice**.

But in case of ART, whenever you install an app, **all the dex files gets converted once and for all**. So the installation takes some time and the app takes more space than in Dalvik, **but the performance is massively improved and battery life is smartly conserved**.

### Boot Time

In case of Dalvik, the cache is built with time the device runs and apps are used as is indicated by the JIT approach. **So the boot time is very fast**.

But in case of ART, **the cache is built during the first boot, so the boot time is considerably more in case of ART**. You might see an "Optimizing apps" dialog box sometimes you boot.

### Space Usage

The space used by apps being run on ART is much more than that of Dalvik. Like a 20 MB app on Dalvik, takes more than 35 MB on ART.

So **if you are on a low storage device, then this can be a huge disadvantage for you**.

### ART is Damn Fast

As discussed above, **ART is extremely fast and smooth**. Apps are very snappy and responsive. Any comparison between Dalvik and ART, will surely make the ART device win by a significant margin.

ART is the answer to all those who argued that iOS is faster and smoother than Android and is also more battery efficient.

## [Why is ART better than Dalvik?](https://www.quora.com/Why-is-ART-better-than-Dalvik)

>KitKat (Android 4.4) has an option to enable the ART runtime instead of Dalvik. From what I've read online, it seems like Google is planning to default to ART in a future release. What makes ART better?


>Robert Love, I work at Google, previously on Android：

**ART (Android RunTime)** is the next version of Dalvik. Dalvik is the runtime, bytecode, and VM used by the Android system for running Android applications.

ART has two main features compared to Dalvik:

  * **Ahead-of-Time (AOT) compilation**, which improves speed (particularly startup time) and reduces memory footprint (no JIT)

  * **Improved Garbage Collection (GC)**

AOT means your apps are compiled to native code once. What is stored on your phone and run is effectively native, not bytecode. This differs from the traditional VM model, which interprets bytecode. Interpreters are slow, so VM developers added a technology called Just-in-Time (JIT) compilation, which compiles (and hopefully optimizes) your code to native code on-the-fly. Dalvik is a JIT'ing VM. The downside to JIT is that the JIT compiler runs while you are using your app, adding latency and memory pressure. The upside is that the JIT compiler can look at how you are using your code to perform profile-directed optimizations.

AOT is like JIT, but it runs once—say, at app installation time. While it lacks the ability to perform profile-directed optimizations, it is possible to perform more extensive optimizations since the compilation is less time sensitive. AOT is useful on systems, such as mobile devices, where JIT adds an unacceptable latency or memory cost to running apps. I think AOT is the right step for Android and ART looks quite impressive.

ART was first included in Android KitKat, but isn't yet enabled by default. You can enable it via Settings > Developer options > Select runtime > Use ART.

>Abhishek Jain, Android app developer and trainer

To understand that you need to understand the functioning of JVM and Dalvik VM first or probably we can start with some pretty basic stuff.

### Basics

The code we write can not be understood by computer(or CPU to be accurate). It can understand only machine language(binary codes). So to make it run on CPU, the code must be converted to machine code, which is done by a translator. This part I think every CS student knows.
The first gen translators were Assemblers, who directly translate assembly codes to machine codes. Since the translation was direct without any intermediate steps, it was fast. The next gen was compilers, which translates the code into assembly codes and then use assemblers to tranlate the code into binary. The compilation was slower than assembly translation for obvious reasons, but the execution of the program were almost as fast as assembly code. C and C++ compilers are from this generation. The problem with this approach was the code was not cross platform, means the code which runs or one machine, may or may not run on other machine.

Then the next gen was interpreters, which translates the code while executing it. Which means it reads a line converts into a binary command and executes it and then go to the next line. Since the translation happens at runtime, the execution was slow.

### Java

Java is different in this case. To maintain cross platformness, it makes a virtual machine (JVM) which is specific to every platform. The Java compiler converts the .Java files into .class files(or archives classes into .jar), which is called byte code. This byte code is given to the JVM which converts it into machine code. Since the byte code for a java file is same on any machine, the cross platformness was achieved(Java's slogan was "code once, run anywhere" at that time, although it's not exactly true. For ex. consider 32 bit and 64 bit byte codes).
Although it was faster than interpretation, but slower than true c++ compilation. That's why C++ is always faster than Java in most of the cases.

### Android

Android uses similar approach, it converts the java files into byte code and put it into apk file. Dalvik VM works similarly and uses JIT compilation. JIT compilation is available on most of the recent devices. Rest all devices which doesn't have any JIT compilers, were even slower. If you say that you can always use C++ code on Android using NDK, then I'd say it uses JNI, which is even slower, so the performance is almost similar, even slower in some cases (Google forbids NDK usage, unless required. It says that using NDK will not gain any performance for you).

### Other platforms

Now if you compare it to other platforms, iOS apps are always faster than android apps, because they use compiled native codes in ipa files(.a and .o files). They can because all of their devices follow same architecture. Android can't because every android device have different architecture.


### ART

ART aka Android RunTime, uses different approach. At the time of installation of apk file, ART compiles the apk into machine code which can be run directly by the device(with the help of ART). Although the compilation takes some time, thus the installation duration will increase, which is negligible, unless you're installing a pretty big apk. Now since the compilation is already done, the apk will work as fast as Native C++ code.
Dalvik VM is ditched in this case.
Now since the compilation will be done on device, it'll be acc. to the architecture of the device(ART would be device specific, thus it'll be acc. to the architecture of the device).

Google was working on it for almost two years and they did include it in Nexus 5. It seems pretty mature and feels fast. It's definitely going to get included in all future devices. I'd say that it'd be the biggest ever change in Android OS history, if they ditched DVM completely and put ART in charge. Probably in next version (4.5 or may be 5.0, since the change will be too radical)

### Disclaimer:

The facts mentioned in this answer are from my memory and understanding. Some facts may be wrong. Please correct them, if you find any mistake. Thanks

###  Update:

Well, as predicted ART is the default runtime in the latest released Lollipop version and since the change was too radical, the version name was chosen to be 5.0 instead of 4.5.

>Aniket Thakur, Software Developer

Better? Well lets keep that decision for end users to take. So far I have heard mixed reviews.  

From technical perspective following are the pros and cons -

### Benefits of ART

  * Reduces startup time of applications as native code is directly executed.

  * Improves battery performance as power utilized to interprete byte codes line by line is saved.

  * As JIT is not in the picture no on the fly compilation and native code storage. This significantly reduces memory footprint (less RAM is required for application to run).

### Drawbacks of ART

  * As dex bytecodes are converted to native machine code on installation itself, installation takes more time.

  * As the native machine code generated on installation is stored in internal storage, more internal storage is required.

More details -> [Difference between Dalvik and ART runtimes in Android](http://opensourceforgeeks.blogspot.in/2015/02/difference-between-dalvik-and-art.html)
