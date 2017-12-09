---
title: Android MonkeyRunner源码解读（二）
date: 2017-12-9 11:03:48
tags:
categories: "测试开发"
---

本章节主要分析MonkeyRunnerStarter的handlePlugins()方法的实现。

```java 
private int run() {
    // This system property gets set by the included starter script
    String monkeyRunnerPath = System.getProperty("com.android.monkeyrunner.bindir") +
            File.separator + "monkeyrunner";
    //加载命令行指定的插件（.jar结尾的文件）
    Map<String, Predicate<PythonInterpreter>> plugins = handlePlugins();
    if (options.getScriptFile() == null) {
        ScriptRunner.console(monkeyRunnerPath);
        chimp.shutdown();
        return 0;
    } else {
        int error = ScriptRunner.run(monkeyRunnerPath, options.getScriptFile().getAbsolutePath(),
                options.getArguments(), plugins);
        chimp.shutdown();
        return error;
    }
}
```

可以看出，插件只用在非交互模式下，最终传递给了ScriptRunner的run()方法。

<!--more-->

看下handlePlugins方法的实现：

```java 
private Map<String, Predicate<PythonInterpreter>> handlePlugins() {
    ImmutableMap.Builder<String, Predicate<PythonInterpreter>> builder = ImmutableMap.builder();
    for (File f : options.getPlugins()) {
        builder.put(f.getAbsolutePath(), handlePlugin(f));
    }
    return builder.build();
}
```

handlePlugins()方法主要针对命令行给定的多个插件，分别加载，然后保存到一个Map集合中。

handlePlugin()的实现如下，具体的步骤见代码注释：

```java 
private Predicate<PythonInterpreter> handlePlugin(File f) {
    //首先判断文件是否是一个jar文件
    JarFile jarFile;
    try {
        jarFile = new JarFile(f);
    } catch (IOException e) {
        LOG.log(Level.SEVERE, "Unable to open plugin file.  Is it a jar file? " +
                f.getAbsolutePath(), e);
        return Predicates.alwaysFalse();
    }
    //然后判断是否是否包含Manifest文件
    Manifest manifest;
    try {
        manifest = jarFile.getManifest();
    } catch (IOException e) {
        LOG.log(Level.SEVERE, "Unable to get manifest file from jar: " +
                f.getAbsolutePath(), e);
        return Predicates.alwaysFalse();
    }
    //判断Manifest xml文件中是否有MONKEY_RUNNER_MAIN_MANIFEST_NAME属性
    //MONKEY_RUNNER_MAIN_MANIFEST_NAME的定义：    private static final String MONKEY_RUNNER_MAIN_MANIFEST_NAME = "MonkeyRunnerStartupRunner";
    Attributes mainAttributes = manifest.getMainAttributes();
    String pluginClass = mainAttributes.getValue(MONKEY_RUNNER_MAIN_MANIFEST_NAME);
    if (pluginClass == null) {
        // No main in this plugin, so it always succeeds.
        return Predicates.alwaysTrue();
    }
    //把文件对象转换为URL对象，主要是为了后面的ClassLoader来加载
    URL url;
    try {
        url =  f.toURI().toURL();
    } catch (MalformedURLException e) {
        LOG.log(Level.SEVERE, "Unable to convert file to url " + f.getAbsolutePath(),
                e);
        return Predicates.alwaysFalse();
    }
    //使用系统当前的ClassLoader来加载这个Java程序到JVM中
    URLClassLoader classLoader = new URLClassLoader(new URL[] { url },
            ClassLoader.getSystemClassLoader());
    Class<?> clz;
    //使用Class.forName加载class文件，pluginClass在Manifest xml文件中定义的
    try {
        clz = Class.forName(pluginClass, true, classLoader);
    } catch (ClassNotFoundException e) {
        LOG.log(Level.SEVERE, "Unable to load the specified plugin: " + pluginClass, e);
        return Predicates.alwaysFalse();
    }
    //调用构造对象
    Object loadedObject;
    try {
        loadedObject = clz.newInstance();
    } catch (InstantiationException e) {
        LOG.log(Level.SEVERE, "Unable to load the specified plugin: " + pluginClass, e);
        return Predicates.alwaysFalse();
    } catch (IllegalAccessException e) {
        LOG.log(Level.SEVERE, "Unable to load the specified plugin " +
                "(did you make it public?): " + pluginClass, e);
        return Predicates.alwaysFalse();
    }
    //转换为具体的类型，这里主要是两个，一个是Runnable类型，一个是Predicate类型；
    //如果两者都是则返回false
    // Cast it to the right type
    if (loadedObject instanceof Runnable) {
        final Runnable run = (Runnable) loadedObject;
        return new Predicate<PythonInterpreter>() {
            public boolean apply(PythonInterpreter i) {
                run.run();
                return true;
            }
        };
    } else if (loadedObject instanceof Predicate<?>) {
        return (Predicate<PythonInterpreter>) loadedObject;
    } else {
        LOG.severe("Unable to coerce object into correct type: " + pluginClass);
        return Predicates.alwaysFalse();
    }
}
```

其实这个加载插件不难，主要是判断文件是否合法，再通过ClassLoader来加载。

可能需要了解的就是monkeyrunner的插件怎么编写，有什么用途，下一章节将针对Android官方提供的文档进行翻译介绍。