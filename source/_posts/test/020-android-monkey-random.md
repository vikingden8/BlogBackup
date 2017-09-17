---
title: Android Monkey伪随机事件的理解
date: 2017-09-17 18:21:12
tags:
categories: "测试开发"
---

通常在网上的文章中介绍到Monkey时都会说到Monkey是一个可以运行在模拟器或设备上的程序，它可以生成用户事件的伪随机序列，比如点击，滑动等，但是都会忘记了到底什么是伪随机呢？下面是Android官网对Moneky的介绍

>The Monkey is a program that runs on your emulator or device and generates pseudo-random streams of user events such as clicks, touches, or gestures, as well as a number of system-level events. You can use the Monkey to stress-test applications that you are developing, in a random yet repeatable manner.

<!--more-->

伪随机，其实通过字面上的理解就知道，就是假的随机，也就是这个随机是有规律的。在执行Monkey的时候，如果我们指定的Seed值如果两次都一样的话，那这两次所生成的事件序列就是一致的。

下面我们在Monkey源码中跟踪下代码。首先我们看下Monkey中Seed值的指定：

```java
/**
 * Print how to use this command.
 */
private void showUsage() {
    StringBuffer usage = new StringBuffer();
    usage.append("usage: monkey [-p ALLOWED_PACKAGE [-p ALLOWED_PACKAGE] ...]\n");
    usage.append("              [-c MAIN_CATEGORY [-c MAIN_CATEGORY] ...]\n");
    usage.append("              [--ignore-crashes] [--ignore-timeouts]\n");
    usage.append("              [--ignore-security-exceptions]\n");
    usage.append("              [--monitor-native-crashes] [--ignore-native-crashes]\n");
    usage.append("              [--kill-process-after-error] [--hprof]\n");
    usage.append("              [--match-description TEXT]\n");
    usage.append("              [--pct-touch PERCENT] [--pct-motion PERCENT]\n");
    usage.append("              [--pct-trackball PERCENT] [--pct-syskeys PERCENT]\n");
    usage.append("              [--pct-nav PERCENT] [--pct-majornav PERCENT]\n");
    usage.append("              [--pct-appswitch PERCENT] [--pct-flip PERCENT]\n");
    usage.append("              [--pct-anyevent PERCENT] [--pct-pinchzoom PERCENT]\n");
    usage.append("              [--pct-permission PERCENT]\n");
    usage.append("              [--pkg-blacklist-file PACKAGE_BLACKLIST_FILE]\n");
    usage.append("              [--pkg-whitelist-file PACKAGE_WHITELIST_FILE]\n");
    usage.append("              [--wait-dbg] [--dbg-no-events]\n");
    usage.append("              [--setup scriptfile] [-f scriptfile [-f scriptfile] ...]\n");
    usage.append("              [--port port]\n");
    usage.append("              [-s SEED] [-v [-v] ...]\n");
    usage.append("              [--throttle MILLISEC] [--randomize-throttle]\n");
    usage.append("              [--profile-wait MILLISEC]\n");
    usage.append("              [--device-sleep-time MILLISEC]\n");
    usage.append("              [--randomize-script]\n");
    usage.append("              [--script-log]\n");
    usage.append("              [--bugreport]\n");
    usage.append("              [--periodic-bugreport]\n");
    usage.append("              [--permission-target-system]\n");
    usage.append("              COUNT\n");
    Logger.err.println(usage.toString());
}
```

我们可以在执行Monkey通过-s指定每次的SEED值，看下参数解析：

```java
/**
 * Process the command-line options
 *
 * @return Returns true if options were parsed with no apparent errors.
 */
private boolean processOptions() {
    // quick (throwaway) check for unadorned command
    if (mArgs.length < 1) {
        showUsage();
        return false;
    }
    try {
        String opt;
        Set<String> validPackages = new HashSet<>();
        while ((opt = nextOption()) != null) {
            if (opt.equals("-s")) {
                mSeed = nextOptionLong("Seed");
            }
            ...
          }
          ...
    }
    ...
  }

/**
 * Returns a long converted from the next data argument, with error handling
 * if not available.
 *
 * @param opt The name of the option.
 * @return Returns a long converted from the argument.
 */
private long nextOptionLong(final String opt) {
    long result;
    try {
        result = Long.parseLong(nextOptionData());
    } catch (NumberFormatException e) {
        Logger.err.println("** Error: " + opt + " is not a number");
        throw e;
    }
    return result;
}


/**
 * Return the next data associated with the current option.
 *
 * @return Returns the data string, or null of there are no more arguments.
 */
private String nextOptionData() {
    if (mCurArgData != null) {
        return mCurArgData;
    }
    if (mNextArg >= mArgs.length) {
        return null;
    }
    String data = mArgs[mNextArg];
    Logger.err.println("data=\"" + data + "\"");
    mNextArg++;
    return data;
}
```

解析出来的SEED值保存在变量mSeed中，看下这个变量被引用的地方：

```java
/**
 * Application that injects random key events and other actions into the system.
 */
public class Monkey {
  ...
  /** The random number seed **/
  long mSeed = 0;


  /**
   * Run the command!
   *
   * @param args The command-line arguments
   * @return Returns a posix-style result code. 0 for no error.
   */
  private int run(String[] args) {
      ...
      mSeed = 0;
      ...
      if (mSeed == 0) {
              mSeed = System.currentTimeMillis() + System.identityHashCode(this);

      ...
      mRandom = new Random(mSeed);
      ...
  }

  ...
```

上面截取出来了在[Monkey.java](https://android.googlesource.com/platform/development/+/master/cmds/monkey/src/com/android/commands/monkey/Monkey.java)中mSeed的引用地方，可以看出，如果在命令行参数中没有给定的-s参数时，Monkey会生成一默认的SEED值。后面的mSeed变量传给了创建的Random对象，其实这个mRandom对象在最后生成各种事件的随机时都是共用这个mRandom变量的，所以这里的问题就是验证下给定一个mSeed值为什么mRandom都会是同一个值的问题：

在本地机器上写上如下代码，你会发现运行多次每次的随机值都会是一样的：

```java
/**
 * Author : Viking Den <vikingden7@gmail.com>
 * Date : 2017/9/17
 */
public class TestRandom {

    public static void main(String[] args) {
        long mSeed = 5 ;
        Random mRandom = new Random(mSeed);
        System.out.println("next int : " + mRandom.nextInt());
        System.out.println("next float : " + mRandom.nextFloat());
        System.out.println("next double : " + mRandom.nextDouble());

        //next int : -1157408321
        //next float : 0.17660207
        //next double : 0.08825840967622589
    }
}
```

你可以在自己的本地机器上尝试一下，这也就证明了为什么给定了同一个SEED，执行两次的Monkey的事件序列会是一致的疑问。

有兴趣的其实可以看看Random中到底怎样生成具体的随机值的。
