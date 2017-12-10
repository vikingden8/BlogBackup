---
title: Android MonkeyRunner源码解读（四）
date: 2017-12-10 20:50:48
tags:
categories: "测试开发"
---

这章节主要介绍monkeyrunner交互模式。

从上一章节的分析可以看出，如果没有指定monkeyrunner的脚本文件，那么monkeyrunner会进入交互模式。

```java
private int run() {
    ...
    if (options.getScriptFile() == null) {
        ScriptRunner.console(monkeyRunnerPath);
        chimp.shutdown();
        return 0;
    }
    ...
}
```

我们看下ScriptRunner的console方法实现：

```Java
/**
 * Start an interactive python interpreter.
 */
public static void console(String executablePath) {
    initPython(executablePath);
    InteractiveConsole python = new JLineConsole();
    python.interact();
}
```

<!--more-->

console方法主要分为两步，一是初始化Python的执行环境，而是创建一个InteractiveConsole对象，并且调用其的interact方法。

```java
private static void initPython(String executablePath) {
    List<String> arg = Collections.emptyList();
    initPython(executablePath, arg, new String[] {""});
}

private static void initPython(String executablePath,
        Collection<String> pythonPath, String[] argv) {
    Properties props = new Properties();

    // Build up the python.path
    StringBuilder sb = new StringBuilder();
    sb.append(System.getProperty("java.class.path"));
    for (String p : pythonPath) {
        sb.append(":").append(p);
    }
    props.setProperty("python.path", sb.toString());

    /** Initialize the python interpreter. */
    // Default is 'message' which displays sys-package-mgr bloat
    // Choose one of error,warning,message,comment,debug
    props.setProperty("python.verbose", "error");

    // This needs to be set for sys.executable to function properly
    props.setProperty("python.executable", executablePath);

    PythonInterpreter.initialize(System.getProperties(), props, argv);

    String frameworkDir = System.getProperty("java.ext.dirs");
    File monkeyRunnerJar = new File(frameworkDir, "monkeyrunner.jar");
    if (monkeyRunnerJar.canRead()) {
        PySystemState.packageManager.addJar(monkeyRunnerJar.getAbsolutePath(), false);
    }
}
```

其实这里需要读者要对Jython有部分的了解，其实Jython就是对Python的JVM实现。initPython首先是对一系列的系统属性进行设置，比如python.path，就是加载非内建的模块，python.verbose设置Log的输出级别等，主要是通过PythonInterpreter.initialize()方法实现，然后重要的一步就是加载monkeyrunner.jar。

初始化完成后直接调用Jython类库提供的JLineConsole创建InteractiveConsole对象，并调用interact()方法即可进入交互模式。

有时间可以研究下Jython的实现。
