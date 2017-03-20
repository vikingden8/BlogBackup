---
title: Android UiAutomator2.0测试用例执行时间
date: 2017-3-20 14:27:51
tags:
categories: "移动性能测试"
---

### 问题点

最近的使用UiAutomator2.0执行的自动化case时，错误率较高，尤其是单个执行时长比较长的Case，出现问题的Case log信息：

![](/images/categories/android/android-performance/1.png)

![](/images/categories/android/android-performance/2.png)

通过上述的两个相邻Case执行的log发现，UiAutomator2.0的执行case的超时时间为5分钟。

<!--more-->

### 如何解决

#### 在执行时指定

通过查阅官方文档，有下面的说明:

>Set timeout (in milliseconds) that will be applied to each test: -e timeout_msec 5000

>Supported for both JUnit3 and JUnit4 style tests. For JUnit3 tests, this flag is the only way to specify timeouts. For JUnit4 tests, this flag overrides timeouts specified via org.junit.rules.Timeout. Please note that in JUnit4 org.junit.Test#timeout() annotation take precedence over both, this flag and org.junit.Test#timeout() annotation.

可以通过设置timeout_msec参数来指定单个Case的执行超时时长，如：

```shell
adb shell am instrument -w -e class com.android.foo.FooTest#testFoo \
com.android.foo/android.support.test.runner.AndroidJUnitRunner
```

#### Test注解

示例：

```Java
package com.viking.uiautomator.test;

import org.junit.Test;

public class TimeoutTest {

    @Test(timeout = 1000)
    public void method01(){
        Thread.sleep(2000);
    }
}
```

>注意：当在测试Case上通过注解设置timeout时，只对当前的测试方法有效。

#### \@Rule

因为AndroidJUnitRunner也是基于JUnit的，所以也可以通过注解的方式设置。其实如何设置也在上面的Android官方文档有说明，因为现在一般是基于JUnit4，这里只对JUnit4的情况说明。

具体的示例：

```Java
package com.viking.uiautomator.test;

import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.Timeout;

public class TimeoutRuleTest {

    @Rule
    public Timeout globalTimeout = Timeout.seconds(1);

    @Test
    public void method01(){
        Thread.sleep(2000) ;
    }

    ...

    @Test
    public void method02(){
        Thread.sleep(200) ;
    }
}
```

>注意，通过上述设置可以为该类的所有测试方法设置执行超时时长。


### 引用

[Getting Started with Testing](https://developer.android.com/training/testing/start/index.html)

[AndroidJUnitRunner](https://developer.android.com/reference/android/support/test/runner/AndroidJUnitRunner.html)

[What's the best way to set up a per-test or per-class timeout when using <junit> in perBatch forkmode?](http://stackoverflow.com/questions/8743594/whats-the-best-way-to-set-up-a-per-test-or-per-class-timeout-when-using-junit)

[JUnit – Timeout Test](https://www.mkyong.com/unittest/junit-4-tutorial-4-time-test/)

[Annotation Type Test](http://junit.sourceforge.net/javadoc/org/junit/Test.html)
