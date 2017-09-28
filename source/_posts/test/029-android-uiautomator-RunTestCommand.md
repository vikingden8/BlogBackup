---
title: Android UiAutomator RunTestCommand
date: 2017-09-28 8:31:48
tags:
categories: "测试开发"
---

RunTestCommand是UiAutomator这个自动化测试工具框架的核心内容,因为所有的测试用例,都是基于这个类去执行的,没有这个类,就没有以后的操作,这是重中之重.

### run方法

```java
@Override
public void run(String[] args) {
    int ret = parseArgs(args);
    switch (ret) {
        case ARG_FAIL_INCOMPLETE_C:
            System.err.println("Incomplete '-c' parameter.");
            System.exit(ARG_FAIL_INCOMPLETE_C);
            break;
        case ARG_FAIL_INCOMPLETE_E:
            System.err.println("Incomplete '-e' parameter.");
            System.exit(ARG_FAIL_INCOMPLETE_E);
            break;
        case ARG_FAIL_UNSUPPORTED:
            System.err.println("Unsupported standalone parameter.");
            System.exit(ARG_FAIL_UNSUPPORTED);
            break;
        default:
            break;
    }
    if (mTestClasses.isEmpty()) {
        addTestClassesFromJars();
        if (mTestClasses.isEmpty()) {
            System.err.println("No test classes found.");
            System.exit(ARG_FAIL_NO_CLASS);
        }
    }
    getRunner().run(mTestClasses, mParams, mDebug, mMonkey);
}
```

run方法主要分为三步，第一步：解析参数；第二步，对测试类的添加；第三步：执行测试类的用例．

这里将命令参数传入到该parseArgs()方法中进行检查并返回ret对象，根据ret对象的值，来决定测试执行时候往下进行。

<!--more-->

```java
private int parseArgs(String[] args) {
    // we are parsing for these parameters:
    // -e <key> <value>
    // key-value pairs
    // special ones are:
    // key is "class", parameter is passed onto JUnit as class name to run
    // key is "debug", parameter will determine whether to wait for debugger
    // to attach
    // -c <class name>
    // -s turns on the simple output format
    // equivalent to -e class <class name>, i.e. passed onto JUnit
    for (int i = 0; i < args.length; i++) {
        //如果当前参数是-e，则后面必须要还有两个参数才能符合，如果不符合，返回ARG_FAIL_INCOMPLETE_E
        if (args[i].equals("-e")) {
            if (i + 2 < args.length) {
                //分别把-e后的key value读出来
                String key = args[++i];
                String value = args[++i];
                //是否是指定执行类
                if (CLASS_PARAM.equals(key)) {
                    addTestClasses(value);
                }
                //是否是Debug模式
                else if (DEBUG_PARAM.equals(key)) {
                    mDebug = "true".equals(value) || "1".equals(value);
                }
                //是否指定了自定义Runner
                else if (RUNNER_PARAM.equals(key)) {
                    mRunnerClassName = value;
                }
                //如果是其他的参数键值对，则放到mParams中
                else {
                    mParams.putString(key, value);
                }
            } else {
                return ARG_FAIL_INCOMPLETE_E;
            }
        }
        //如果指定了-c参数，则判断是否长度+1小于参数的总长度，如果不符合，则返回ARG_FAIL_INCOMPLETE_C
        else if (args[i].equals("-c")) {
            //添加到测试类集合中
            if (i + 1 < args.length) {
                addTestClasses(args[++i]);
            } else {
                return ARG_FAIL_INCOMPLETE_C;
            }
        }
        //是否有--monkey参数
        else if (args[i].equals("--monkey")) {
            mMonkey = true;
        }
        //是否有-s参数
        else if (args[i].equals("-s")) {
            mParams.putString(OUTPUT_FORMAT_KEY, OUTPUT_SIMPLE);
        }
        //如果还有其他未定义的参数传入，则返回ARG_FAIL_UNSUPPORTED
        else {
            return ARG_FAIL_UNSUPPORTED;
        }
    }
    //参数无误后，返回ARG_OK
    return ARG_OK;
}
```

下面是上面方法解析中用到的常量定义：

```java
private static final String OUTPUT_SIMPLE = "simple";
private static final String OUTPUT_FORMAT_KEY = "outputFormat";
private static final String CLASS_PARAM = "class";
private static final String JARS_PARAM = "jars";
private static final String DEBUG_PARAM = "debug";
private static final String RUNNER_PARAM = "runner";
private static final String CLASS_SEPARATOR = ",";
private static final String JARS_SEPARATOR = ":";
private static final int ARG_OK = 0;
private static final int ARG_FAIL_INCOMPLETE_E = -1;
private static final int ARG_FAIL_INCOMPLETE_C = -2;
private static final int ARG_FAIL_NO_CLASS = -3;
private static final int ARG_FAIL_RUNNER = -4;
private static final int ARG_FAIL_UNSUPPORTED = -99;
```

下面看下参数解析中对addTestClasses方法的定义：

```java
/**
 * Add test classes from a potentially comma separated list
 * @param classes
 */
private void addTestClasses(String classes) {
    String[] classArray = classes.split(CLASS_SEPARATOR);
    for (String clazz : classArray) {
        mTestClasses.add(clazz);
    }
}
```

这个其实很简单，就是通过字符串的切分，把测试类加入到mTestClasses中，因为可以指定多个测试类，多个类之间用CLASS_SEPARATOR，也就是逗号分隔．

第二步，从上面定义的类变量mTestClasses中，首先检查它是否为空，如果为空，则调用addTestClassesFromJars()方法来进行测试方法的添加。当添加完后再次检查，如果还为空，则退出执行，返回一个错误值。这是因为在指定要测试的JAR包时，可以不指定要具体测试的类，而可以测试给定测试JAR包中的所有测试用例．但是如果要制定的JAR包中没有任何的测试用例要执行，会抛出异常．

下面看下addTestClassesFromJars的实现：

```java
/**
 * Add test classes from jars passed on the command line. Use this if nothing was explicitly
 * specified on the command line.
 */
private void addTestClassesFromJars() {
    //获取所有的jar包定义地址
    String jars = mParams.getString(JARS_PARAM);
    if (jars == null) return;
    //如果指定了多个，则通过分隔符分隔
    String[] jarFileNames = jars.split(JARS_SEPARATOR);
    for (String fileName : jarFileNames) {
        fileName = fileName.trim();
        if (fileName.isEmpty()) continue;
        try {
            //通过jar的路径，创建一个DexFile，然后迭代里面的类文件判断是否是测试类文件
            DexFile dexFile = new DexFile(fileName);
            for(Enumeration<String> e = dexFile.entries(); e.hasMoreElements();) {
                String className = e.nextElement();
                if (isTestClass(className)) {
                    mTestClasses.add(className);
                }
            }
            dexFile.close();
        } catch (IOException e) {
            Log.w(LOGTAG, String.format("Could not read %s: %s", fileName, e.getMessage()));
        }
    }
}

/**
 * Tries to determine if a given class is a test class. A test class has to inherit from
 * UiAutomator test case and it must be a top-level class.
 * @param className
 * @return
 */
private boolean isTestClass(String className) {
    try {
        Class<?> clazz = this.getClass().getClassLoader().loadClass(className);
        if (clazz.getEnclosingClass() != null) return false;
        return getRunner().getTestCaseFilter().accept(clazz);
    } catch (ClassNotFoundException e) {
        return false;
    }
}
```

其实这里面要注意的是，我们通过adb shell uiautomator runtest指定的jar会通过[uiautomator](https://github.com/android/platform_frameworks_base/blob/master/cmds/uiautomator/cmds/uiautomator/uiautomator)的执行shell脚本进行转换，转换后的参数格式就是通过-e jars [jar path]格式指定．

测试参数无误，测试用例类都准备好了之后，差不多可以启动进行测试了，这就到了第三步．

```java
 getRunner().run(mTestClasses, mParams, mDebug, mMonkey);
```

但是这里有一个getRunner()方法,因为有可能有自定义的测试执行类：

```java
protected UiAutomatorTestRunner getRunner() {
    if (mRunner != null) {
        return mRunner;
    }
    //如果指定的自定义Runner为空，则使用UiAutomatorTestRunner，创建后立即返回
    if (mRunnerClassName == null) {
        mRunner = new UiAutomatorTestRunner();
        return mRunner;
    }
    // use reflection to get the runner
    //使用反射的方法得到自定义的Runner类，并返回
    Object o = null;
    try {
        Class<?> clazz = Class.forName(mRunnerClassName);
        o = clazz.newInstance();
    } catch (ClassNotFoundException cnfe) {
        System.err.println("Cannot find runner: " + mRunnerClassName);
        System.exit(ARG_FAIL_RUNNER);
    } catch (InstantiationException ie) {
        System.err.println("Cannot instantiate runner: " + mRunnerClassName);
        System.exit(ARG_FAIL_RUNNER);
    } catch (IllegalAccessException iae) {
        System.err.println("Constructor of runner " + mRunnerClassName + " is not accessibile");
        System.exit(ARG_FAIL_RUNNER);
    }
    try {
        UiAutomatorTestRunner runner = (UiAutomatorTestRunner)o;
        mRunner = runner;
        return runner;
    } catch (ClassCastException cce) {
        System.err.println("Specified runner is not subclass of "
                + UiAutomatorTestRunner.class.getSimpleName());
        System.exit(ARG_FAIL_RUNNER);
    }
    // won't reach here
    return null;
}
```
UiAutomatorTestRunner或者自定义的UiAutomatorTestRunner有一个run方法，其实可以猜到是测试用例类的执行，这个在后续的章节中解析.
