---
title: Android UiAutomator UiAutomatorTestCaseFilter
date: 2017-09-30 8:31:48
tags:
categories: "测试开发"
---

上节介绍了TestCaseCollector，其中过滤测试类和测试用例的类是在哪里定义的呢？

在UiAutomatorTestRunner中创建TestCaseCollector时，指定了具体的TestCaseFilter：

```java
protected TestCaseCollector getTestCaseCollector(ClassLoader classLoader) {
    return new TestCaseCollector(classLoader, getTestCaseFilter());
}

/**
 * Returns an object which determines if the class and its methods should be
 * accepted into the test suite.
 * @return
 */
public UiAutomatorTestCaseFilter getTestCaseFilter() {
    return new UiAutomatorTestCaseFilter();
}
```

<!--more-->

下面看下UiAutomatorTestCaseFilter的定义：

```java
/**
 * A {@link TestCaseFilter} that accepts testFoo methods and {@link UiAutomatorTestCase} classes
 *
 * @hide
 */
public class UiAutomatorTestCaseFilter implements TestCaseFilter {

    @Override
    public boolean accept(Method method) {
        return ((method.getParameterTypes().length == 0) &&
                (method.getName().startsWith("test")) &&
                (method.getReturnType().getSimpleName().equals("void")));
    }

    @Override
    public boolean accept(Class<?> clazz) {
        return UiAutomatorTestCase.class.isAssignableFrom(clazz);
    }

}
```

很简单的定义，这里要指出的，那时的uiautomator框架里面还是集成的junit3.x．

先看下测试用例方法的过滤规则：

* 首先方法参数的长度必须等于０，也就是不带参数；

* 方法名必须以test开头；

* 方法的返回类型必须是void．

再看下测试用例类的过滤规则：

其实就一个，也就是要继承自UiAutomatorTestCase
