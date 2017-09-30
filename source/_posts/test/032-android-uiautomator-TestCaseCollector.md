---
title: Android UiAutomator TestCaseCollector
date: 2017-09-30 7:31:48
tags:
categories: "测试开发"
---


TestCaseCollector顾名思义就是测试用例的收集器，就是把所有的测试类都进行转换并存储在类变量中,以提供调用。

先来看看该类的开头,也就是定义的全局变量以及构造方法.

```java
private ClassLoader mClassLoader;
private List<TestCase> mTestCases;
private TestCaseFilter mFilter;

public TestCaseCollector(ClassLoader classLoader, TestCaseFilter filter) {
    mClassLoader = classLoader;
    mTestCases = new ArrayList<TestCase>();
    mFilter = filter;
}
```

第一个参数指定类加载器，第二个参数为测试用例的过滤类。

<!--more-->

```java
/**
 * Adds classes to test by providing a list of class names in string
 *
 * The class name may be in "<class name>#<method name>" format
 *
 * @param classNames class must be subclass of {@link UiAutomatorTestCase}
 * @throws ClassNotFoundException
 */
 //添加测试类集合的方法，这个是添加一个集合，会逐一调用下面的addTestClass方法
public void addTestClasses(List<String> classNames) throws ClassNotFoundException {
    for (String className : classNames) {
        addTestClass(className);
    }
}

/**
 * Adds class to test by providing class name in string.
 *
 * The class name may be in "<class name>#<method name>" format
 *
 * @param className classes must be subclass of {@link UiAutomatorTestCase}
 * @throws ClassNotFoundException
 */
 //添加单个测试类的方法，注意，文档里面提到的测试类指定的格式必须是<class name>#<method name>
public void addTestClass(String className) throws ClassNotFoundException {
    int hashPos = className.indexOf('#');
    //根据#进行class和method的拆分
    String methodName = null;
    if (hashPos != -1) {
        methodName = className.substring(hashPos + 1);
        className = className.substring(0, hashPos);
    }
    addTestClass(className, methodName);
}

/**
 * Adds class to test by providing class name and method name in separate strings
 *
 * @param className class must be subclass of {@link UiAutomatorTestCase}
 * @param methodName may be null, in which case all "public void testNNN(void)" functions
 *                   will be added
 * @throws ClassNotFoundException
 */
public void addTestClass(String className, String methodName) throws ClassNotFoundException {
    Class<?> clazz = mClassLoader.loadClass(className);
    //如果有指定要测试的方法 则只添加这个方法
    if (methodName != null) {
        addSingleTestMethod(clazz, methodName);
    } else {
        //指定的是测试用例类，则会去该类下过滤出来所有的测试用例方法
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            if (mFilter.accept(method)) {
                addSingleTestMethod(clazz, method.getName());
            }
        }
    }
}
```

接下往下看，我们来到addSingleTestMehod()方法了

```java

protected void addSingleTestMethod(Class<?> clazz, String method) {
    //先判断是否继承自UiAutomatorTestCase类
    if (!(mFilter.accept(clazz))) {
        throw new RuntimeException("Test class must be derived from UiAutomatorTestCase");
    }
    try {
        //通过反射的方法，创建出所有测试用例。如果类不能实例化，则抛出异常当然，里面包含了一个error类型的实例方法，这个方法在后面会讲到，也就是构建一个相同类型的方法用来打印显示该方法发生了错误。
        TestCase testCase = (TestCase) clazz.newInstance();
        testCase.setName(method);
        mTestCases.add(testCase);
    } catch (InstantiationException e) {
        mTestCases.add(error(clazz, "InstantiationException: could not instantiate " +
                "test class. Class: " + clazz.getName()));
    } catch (IllegalAccessException e) {
        mTestCases.add(error(clazz, "IllegalAccessException: could not instantiate " +
                "test class. Class: " + clazz.getName()));
    }
}
```

接下来看下error方法的实现：

```java
private UiAutomatorTestCase error(Class<?> clazz, final String message) {
    UiAutomatorTestCase warning = new UiAutomatorTestCase() {
        protected void runTest() {
            fail(message);
        }
    };

    warning.setName(clazz.getName());
    return warning;
}
```

这里，我们可以看到，这是一个私有的方法，目的很明确，就是构造一个UiAutomatorTestCase类型的错误类，并重新设置该类的名字，用来传递不同的错误信息，这里直接调用了Junit的fail()方法给我们传递信息，由于方法名的区别，我们可以完美识别错误到底发生在了哪个类的身上；

最后，则是返回这个失败的UiAutomatorTestCase类型的对象（UiAutomatorTestCase继承TestCase），所以，在addSingleTestCase()方法中的异常操作是调用私有方法所返回的相同类型的对象。

同时，该类还提供了一个内部接口:

```java
/**
  * Determine if a class and its method should be accepted into test suite
  *
  */
 public interface TestCaseFilter {

     /**
      * Determine that based on the method signature, if it can be accepted
      * @param method
      */
     public boolean accept(Method method);

     /**
      * Determine that based on the class type, if it can be accepted
      * @param clazz
      * @return
      */
     public boolean accept(Class<?> clazz);
 }
```

这个接口主要决定了该类或者该类中的方法是否是测试用例或者测试用例包含类，定义了两个抽象方法，第一个accept方法的参数是Method，主要是对类中的方法是否是测试用例的判断；第二个accept方法的参数是class，主要是决定判断该类是否是测试用例的包含类，比如上面有提到uiautomator的测试类必须继承自UiAutomatorTestCase类。
