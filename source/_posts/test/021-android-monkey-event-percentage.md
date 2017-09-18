---
title: Android Monkey各事件的概率大小设置
date: 2017-09-18 21:26:35
tags:
categories: "测试开发"
---

在Monkey的命令帮助中有如下的参数：

```java
...
usage.append("              [--pct-touch PERCENT] [--pct-motion PERCENT]\n");
usage.append("              [--pct-trackball PERCENT] [--pct-syskeys PERCENT]\n");
usage.append("              [--pct-nav PERCENT] [--pct-majornav PERCENT]\n");
usage.append("              [--pct-appswitch PERCENT] [--pct-flip PERCENT]\n");
usage.append("              [--pct-anyevent PERCENT] [--pct-pinchzoom PERCENT]\n");
usage.append("              [--pct-permission PERCENT]\n");
...
```

<!--more-->

如上就是设置各事件的概率参数，总共有是一种事件的设置。若在命令参数中指定了特定的百分比，Monkey将会按照给定的比例重新分配，下面看下代码的实现，解析参数在之前的random章节已有介绍，在[Monkey.java](https://android.googlesource.com/platform/development/+/master/cmds/monkey/src/com/android/commands/monkey/Monkey.java)的processOptions方法中：

```java
...
else if (opt.equals("--pct-touch")) {
   int i = MonkeySourceRandom.FACTOR_TOUCH;
   mFactors[i] = -nextOptionLong("touch events percentage");
} else if (opt.equals("--pct-motion")) {
   int i = MonkeySourceRandom.FACTOR_MOTION;
   mFactors[i] = -nextOptionLong("motion events percentage");
} else if (opt.equals("--pct-trackball")) {
   int i = MonkeySourceRandom.FACTOR_TRACKBALL;
   mFactors[i] = -nextOptionLong("trackball events percentage");
} else if (opt.equals("--pct-rotation")) {
   int i = MonkeySourceRandom.FACTOR_ROTATION;
   mFactors[i] = -nextOptionLong("screen rotation events percentage");
} else if (opt.equals("--pct-syskeys")) {
   int i = MonkeySourceRandom.FACTOR_SYSOPS;
   mFactors[i] = -nextOptionLong("system (key) operations percentage");
} else if (opt.equals("--pct-nav")) {
   int i = MonkeySourceRandom.FACTOR_NAV;
   mFactors[i] = -nextOptionLong("nav events percentage");
} else if (opt.equals("--pct-majornav")) {
   int i = MonkeySourceRandom.FACTOR_MAJORNAV;
   mFactors[i] = -nextOptionLong("major nav events percentage");
} else if (opt.equals("--pct-appswitch")) {
   int i = MonkeySourceRandom.FACTOR_APPSWITCH;
   mFactors[i] = -nextOptionLong("app switch events percentage");
} else if (opt.equals("--pct-flip")) {
   int i = MonkeySourceRandom.FACTOR_FLIP;
   mFactors[i] = -nextOptionLong("keyboard flip percentage");
} else if (opt.equals("--pct-anyevent")) {
   int i = MonkeySourceRandom.FACTOR_ANYTHING;
   mFactors[i] = -nextOptionLong("any events percentage");
} else if (opt.equals("--pct-pinchzoom")) {
   int i = MonkeySourceRandom.FACTOR_PINCHZOOM;
   mFactors[i] = -nextOptionLong("pinch zoom events percentage");
} else if (opt.equals("--pct-permission")) {
   int i = MonkeySourceRandom.FACTOR_PERMISSION;
   mFactors[i] = -nextOptionLong("runtime permission toggle events percentage");
}
...
```

这里需要注意的是，在程序开始的时候，会把各事件的概率分别设置为1.0f，在Monkey.java的run方法中：

```java
// set a positive value, indicating none of the factors is provided yet
for (int i = 0; i < MonkeySourceRandom.FACTORZ_COUNT; i++) {
    mFactors[i] = 1.0f;
}
```

主要是为了后面来区分哪些事件是用户设置了的，哪些不是。在完成了参数的解析之后，由于我们这里要分析的是MonkeySourceRandom的实现，在实例化MonkeySourceRandom之后，直接设置各事件的概率，如下：

```java
...
{
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
```

在实例化MonkeySourceRandom之后，通过其validate方法来验证参数的正确性，到底验证了些啥呢？查看下[MonkeySourceRandom.java](https://android.googlesource.com/platform/development/+/master/cmds/monkey/src/com/android/commands/monkey/MonkeySourceRandom.java)的源码：

```java
...
public MonkeySourceRandom(Random random, List<ComponentName> MainApps,
        long throttle, boolean randomizeThrottle, boolean permissionTargetSystem) {
    // default values for random distributions
    // note, these are straight percentages, to match user input (cmd line args)
    // but they will be converted to 0..1 values before the main loop runs.
    mFactors[FACTOR_TOUCH] = 15.0f;
    mFactors[FACTOR_MOTION] = 10.0f;
    mFactors[FACTOR_TRACKBALL] = 15.0f;
    // Adjust the values if we want to enable rotation by default.
    mFactors[FACTOR_ROTATION] = 0.0f;
    mFactors[FACTOR_NAV] = 25.0f;
    mFactors[FACTOR_MAJORNAV] = 15.0f;
    mFactors[FACTOR_SYSOPS] = 2.0f;
    mFactors[FACTOR_APPSWITCH] = 2.0f;
    mFactors[FACTOR_FLIP] = 1.0f;
    // disbale permission by default
    mFactors[FACTOR_PERMISSION] = 0.0f;
    mFactors[FACTOR_ANYTHING] = 13.0f;
    mFactors[FACTOR_PINCHZOOM] = 2.0f;
    mRandom = random;
    mMainApps = MainApps;
    mQ = new MonkeyEventQueue(random, throttle, randomizeThrottle);
    mPermissionUtil = new MonkeyPermissionUtil();
    mPermissionUtil.setTargetSystemPackages(permissionTargetSystem);
}
...

/**
* Adjust the percentages (after applying user values) and then normalize to a 0..1 scale.
*/
private boolean adjustEventFactors() {
   // go through all values and compute totals for user & default values
   float userSum = 0.0f;
   float defaultSum = 0.0f;
   int defaultCount = 0;
   for (int i = 0; i < FACTORZ_COUNT; ++i) {
       if (mFactors[i] <= 0.0f) {   // user values are zero or negative
           userSum -= mFactors[i];
       } else {
           defaultSum += mFactors[i];
           ++defaultCount;
       }
   }
   // if the user request was > 100%, reject it
   if (userSum > 100.0f) {
       Logger.err.println("** Event weights > 100%");
       return false;
   }
   // if the user specified all of the weights, then they need to be 100%
   if (defaultCount == 0 && (userSum < 99.9f || userSum > 100.1f)) {
       Logger.err.println("** Event weights != 100%");
       return false;
   }
   // compute the adjustment necessary
   float defaultsTarget = (100.0f - userSum);
   float defaultsAdjustment = defaultsTarget / defaultSum;
   // fix all values, by adjusting defaults, or flipping user values back to >0
   for (int i = 0; i < FACTORZ_COUNT; ++i) {
       if (mFactors[i] <= 0.0f) {   // user values are zero or negative
           mFactors[i] = -mFactors[i];
       } else {
           mFactors[i] *= defaultsAdjustment;
       }
   }
   // if verbose, show factors
   if (mVerbose > 0) {
       Logger.out.println("// Event percentages:");
       for (int i = 0; i < FACTORZ_COUNT; ++i) {
           Logger.out.println("//   " + i + ": " + mFactors[i] + "%");
       }
   }
   if (!validateKeys()) {
       return false;
   }
   // finally, normalize and convert to running sum
   float sum = 0.0f;
   for (int i = 0; i < FACTORZ_COUNT; ++i) {
       sum += mFactors[i] / 100.0f;
       mFactors[i] = sum;
   }
   return true;
}
...

public boolean validate() {
    boolean ret = true;
    // only populate & dump permissions if enabled
    if (mFactors[FACTOR_PERMISSION] != 0.0f) {
        ret &= mPermissionUtil.populatePermissionsMapping();
        if (ret && mVerbose >= 2) {
            mPermissionUtil.dump();
        }
    }
    return ret & adjustEventFactors();
}
...
```

首先，MonkeySourceRandom中的各事件是有默认的事件概率的，也就是当参数中没有指定事件的概率时，会使用其构造函数中的默认概率，validate方法先会对权限事件不为0的情况下坐下处理，然后调用adjustEventFactors来调整所有事件的概率，步骤主要是：

* 遍历所有事件的概率，找出userSum（用户设定）和defaultSum （默认）分别的总和；

* 如果用户设置的总和大于100，则表明错误，直接返回；也就是说用户设置的各事件总和可以为100，也可以小于100；

* 如果用户设置了所有事件的概率，则需满足所有的事件概率总和为100；

* 如果未设置所有事件的概率，计算出用户未设置事件的概率值，计算规则就是 默认值的概率 * （100f-用户设置事件的概率总和）/ 用户未设置事件的概率的个数；

* 最后，把各个事件的百分比值，转化为0~1.0f之间的区域值。

这样就设置了Monkey在测试过程中对各个事件的概率值的设定。
