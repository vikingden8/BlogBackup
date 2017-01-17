---
title: Android自动化-Monkey源码分析之运行流程
date: 2017-1-17 16:32:32
tags:
categories: "Android自动化"
---

在使用monkey时，我们在命令行窗口一般是输入adb shell monkey xxx，可知这执行的流程是怎样？

其实有很多像monkey这样的命令，如pm，am等，其实，我们在调用调用这个命令时，android会去/system/bin/目录下找对应的shell脚本，其实跟linux差不多。

看下monkey的shell脚本，文件目录：/system/bin/monkey

```sh
base=/system  
export CLASSPATH=$base/framework/monkey.jar  
trap "" HUP  
exec app_process $base/bin com.android.commands.monkey.Monkey $*  
```

如上，其实很简单，这是一个android上面java程序启动的标准流程。

android中可以通过多种方式启动java应用，通过app_process命令启动就是其中一种，它可以帮忙注册android JNI,而绕过dalvik以使用Native API。

上面的脚本所做的主要事情如下：

  * 设置monkey的CLASSPATH环境变量指向monkey.jar

  * 通过app_process指定monkey的入口，启动monkey.jar，并把shell脚本所带的参数传递给jar包。

<!--more-->

#### 解析参数执行

通过以上的app_process指定的monkey入口，我们可以知道我们的入口函数main是在com.android.commands.Monkey这个类里面的：

```java
/**
 * Command-line entry point.
 *
 * @param args The command-line arguments
 */
public static void main(String[] args) {
    // Set the process name showing in "ps" or "top"

    Process.setArgV0("com.android.commands.monkey");

    int resultCode = (new Monkey()).run(args);
    System.exit(resultCode);
}
```

Process.setArgV0主要是设置monkey进程名称，然后就执行run方法：

run方法主要的工作就是：

  * 处理命令行参数；

  * 根据命令行参数启动不同的事件源，也就是我们的测试事件究竟是从网络如monkeyrunner过来的还是monkey内部的random测试数据集过来的还是脚本过来的如此之类；

  * 跳入runMonkeyCyncle方法针对不同的事件源开始获取并执行不同的事件；

这个方法会比较长，只看关键片段，可以看到里面调用了命令行处理函数processOptions：

```java
private int run(String[] args) {  
    ...  
    if (!processOptions()) {  
        return -1;  
    }  

    if (!loadPackageLists()) {
        return -1;
    }

    if (!getMainApps()) {
        return -4;
    }
    ...  
}  
```

其就是很普通的读取命令行的参数然后一个个进行解析保存了.

loadPackageLists主要是对指定的-p或者白名单，以及黑名单的处理，这里需要主要的就是，-p或者白名单不能黑名单一起指定：

```java
/**
 * Load package blacklist or whitelist (if specified).
 *
 * @return Returns false if any error occurs.
 */
private boolean loadPackageLists() {
    if (((mPkgWhitelistFile != null) || (MonkeyUtils.getPackageFilter().hasValidPackages()))
            && (mPkgBlacklistFile != null)) {
        System.err.println("** Error: you can not specify a package blacklist "
                + "together with a whitelist or individual packages (via -p).");
        return false;
    }

    Set<String> validPackages = new HashSet<String>();
    if ((mPkgWhitelistFile != null)
            && (!loadPackageListFromFile(mPkgWhitelistFile, validPackages))) {
        return false;
    }
    MonkeyUtils.getPackageFilter().addValidPackages(validPackages);

    Set<String> invalidPackages = new HashSet<String>();
    if ((mPkgBlacklistFile != null)
            && (!loadPackageListFromFile(mPkgBlacklistFile, invalidPackages))) {
        return false;
    }
    MonkeyUtils.getPackageFilter().addInvalidPackages(invalidPackages);

    return true;
}
```

下面的getMainApp主要是通过在参数行指定的-p或者白名单，黑名单的包名，与系统中的所有应用进行对比筛选出要测试的应用。

然后就是对测试的类型进行判断，比如是否是指定的monkey脚本，还是网络测试，还有就是默认的伪随机monkey测试，其实大部分都是最后一个。

```java
private int run(String[] args) {  
     ...  
     mRandom = new Random(mSeed);

     if (mScriptFileNames != null && mScriptFileNames.size() == 1) {
         // script mode, ignore other options
         mEventSource = new MonkeySourceScript(mRandom, mScriptFileNames.get(0), mThrottle,
                 mRandomizeThrottle, mProfileWaitTime, mDeviceSleepTime);
         mEventSource.setVerbose(mVerbose);

         mCountEvents = false;
     } else if (mScriptFileNames != null && mScriptFileNames.size() > 1) {
         if (mSetupFileName != null) {
             mEventSource = new MonkeySourceRandomScript(mSetupFileName,
                     mScriptFileNames, mThrottle, mRandomizeThrottle, mRandom,
                     mProfileWaitTime, mDeviceSleepTime, mRandomizeScript);
             mCount++;
         } else {
             mEventSource = new MonkeySourceRandomScript(mScriptFileNames,
                     mThrottle, mRandomizeThrottle, mRandom,
                     mProfileWaitTime, mDeviceSleepTime, mRandomizeScript);
         }
         mEventSource.setVerbose(mVerbose);
         mCountEvents = false;
     } else if (mServerPort != -1) {
         try {
             mEventSource = new MonkeySourceNetwork(mServerPort);
         } catch (IOException e) {
             System.out.println("Error binding to network socket.");
             return -5;
         }
         mCount = Integer.MAX_VALUE;
     } else {
         // random source by default
         if (mVerbose >= 2) { // check seeding performance
             System.out.println("// Seeded: " + mSeed);
         }
         mEventSource = new MonkeySourceRandom(mRandom, mMainApps,
                 mThrottle, mRandomizeThrottle, mPermissionTargetSystem);
         mEventSource.setVerbose(mVerbose);
         // set any of the factors that has been set
         for (int i = 0; i < MonkeySourceRandom.FACTORZ_COUNT; i++) {
             if (mFactors[i] <= 0.0f) {
                 ((MonkeySourceRandom) mEventSource).setFactors(i, mFactors[i]);
             }
         }

         // in random mode, we start with a random activity
         ((MonkeySourceRandom) mEventSource).generateActivity();
     }

     // validate source generator
     if (!mEventSource.validate()) {
         return -5;
     }
     ...
}
```

其实这一段的主要作用就是：

  * 从指定的源获取命令；

  * 把命令翻译成monkey事件然后放到命令队列EventQueue。

这里需要注意的是每一个EventSource类都是实现了MonkeyEventSource这个接口的，这个接口最重要的就是要求实现类必须实现getNextEvent这个方法来生成并获取事件。这样子做的好处就是屏蔽了每一个具体事件源的实现细节，其他地方的代码只需要调用这个接口的getNextEvent方法获得事件源就行了，而无需关心这些事件是从哪个源过来的。这些都是面向对象的面向接口编程的基础了，大家有不清楚的最好先去了解下java的一些基本知识，这样理解起来会快很多。这个就好比Java集合框架中在迭代对象时候的Iterator接口。

创建事件后，主要是进入了下面的逻辑：

```java
private int run(String[] args) {  
    ...
    mNetworkMonitor.start();
    int crashedAtCycle = 0;
    try {
        crashedAtCycle = runMonkeyCycles();
    } finally {
        // Release the rotation lock if it's still held and restore the
        // original orientation.
        new MonkeyRotationEvent(Surface.ROTATION_0, false).injectEvent(
            mWm, mAm, mVerbose);
    }
    mNetworkMonitor.stop();
    ...
}
```

进入runMonkeyCycles，如前所述，会根据不同的数据源开始一条条的获取事件并进行执行：

```java
/**
 * Run mCount cycles and see if we hit any crashers.
 * <p>
 * TODO: Meta state on keys
 *
 * @return Returns the last cycle which executed. If the value == mCount, no
 *         errors detected.
 */  
private int runMonkeyCycles() {  
    int eventCounter = 0;  
    int cycleCounter = 0;  

    boolean shouldReportAnrTraces = false;  
    boolean shouldReportDumpsysMemInfo = false;  
    boolean shouldAbort = false;  
    boolean systemCrashed = false;  

    // TO DO : The count should apply to each of the script file.  
    while (!systemCrashed && cycleCounter < mCount) {  
            ...  
        MonkeyEvent ev = mEventSource.getNextEvent();  
        if (ev != null) {  
            int injectCode = ev.injectEvent(mWm, mAm, mVerbose);  
            ...  
         }  
    ...  
    }  
   ....  
}  

```

注意这里的mEventSource就是我们上面提到的事件源的接口，它屏蔽了每个事件实现类的具体细节，我们只需要告诉这个接口我们现在需要取一条事件然后执行它，该结构根据面向对象的多态原理，就会自动取事件的实现类获得对应的事件进行返回。
