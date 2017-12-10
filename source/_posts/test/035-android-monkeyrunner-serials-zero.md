---
title: Android MonkeyRunner源码解读（一）
date: 2017-12-8 23:42:48
tags:
categories: "测试开发"
---

monkeyrunner的执行程序在sdk下的tools目录下，最新的sdk位置tools下的bin目录下。

monkeyrunner的源码路径：[monkeyrunner](https://android.googlesource.com/platform/tools/swt/+/master/monkeyrunner/)

sdk下的monkeyrunner.bat就是来自于: [monkeyrunner.bat](https://android.googlesource.com/platform/tools/swt/+/master/monkeyrunner/etc/monkeyrunner.bat). 可以看出，带.bat是Windows下的。Linux或Mac OS下的来自于SH脚本：[monkeyrunner](https://android.googlesource.com/platform/tools/swt/+/master/monkeyrunner/etc/monkeyrunner)

有兴趣的可以看下这个脚本的实现，这里关注的是最后一行的代码：

```sh 
# need to use "java.ext.dirs" because "-jar" causes classpath to be ignored
# might need more memory, e.g. -Xmx128M
exec java -Xmx128M $os_opts $java_debug -Djava.ext.dirs="$frameworkdir:$swtpath" -Djava.library.path="$libdir" -Dcom.android.monkeyrunner.bindir="$progdir" -jar "$jarpath" "$@"
```

也就是最终是通过java -jar的方式调用SDK下tools/libs下的monkeyrunner-\[version].jar。

<!--more-->

通过查看项目的[build.gradle](https://android.googlesource.com/platform/tools/swt/+/master/monkeyrunner/build.gradle)编译配置文件：

```
...
// configure the manifest of the buildDistributionJar task.
sdkJar.manifest.attributes("Main-Class": "com.android.monkeyrunner.MonkeyRunnerStarter")
...
```

可以看出程序的入口类是：[com.android.monkeyrunner.MonkeyRunnerStarter]()，由于是一个Java程序，那么这个类的main方法肯定就是程序的起始方法：

```
public static void main(String[] args) {
    MonkeyRunnerOptions options = MonkeyRunnerOptions.processOptions(args);

    if (options == null) {
        return;
    }

    // logging property files are difficult
    replaceAllLogFormatters(MonkeyFormatter.DEFAULT_INSTANCE, options.getLogLevel());

    MonkeyRunnerStarter runner = new MonkeyRunnerStarter(options);
    int error = runner.run();

    // This will kill any background threads as well.
    System.exit(error);
}
```

main方法首先根据输入的参数，构建出了MonkeyRunnerOptions对象，如果参数不合法，则直接退出；然后，设置Log相关的参数；最后，把参数options传给MonkeyRunnerStarter，并调用它的run方法来执行，最后根据运行返回的值关闭程序。

先来看下MonkeyRunnerOptions中参数的处理：

```java 
//当参数设置不正确时，显示错误的原因以及帮助信息
private static void printUsage(String message) {
    System.out.println(message);
    System.out.println("Usage: monkeyrunner [options] SCRIPT_FILE");
    System.out.println("");
    System.out.println("    -s      MonkeyServer IP Address.");
    System.out.println("    -p      MonkeyServer TCP Port.");
    System.out.println("    -v      MonkeyServer Logging level (ALL, FINEST, FINER, FINE, CONFIG, INFO, WARNING, SEVERE, OFF)");
    System.out.println("");
    System.out.println("");
}

/**
 * Process the command-line options
 * 处理命令行参数的方法
 *
 * @return the parsed options, or null if there was an error.
 */
public static MonkeyRunnerOptions processOptions(String[] args) {
    // parse command line parameters.
    int index = 0;

    String hostname = DEFAULT_MONKEY_SERVER_ADDRESS;
    File scriptFile = null;
    int port = DEFAULT_MONKEY_PORT;
    String backend = "adb";
    Level logLevel = Level.SEVERE;

    ImmutableList.Builder<File> pluginListBuilder = ImmutableList.builder();
    ImmutableList.Builder<String> argumentBuilder = ImmutableList.builder();
    while (index < args.length) {
        String argument = args[index++];
        //匹配-s参数 MonkeyRunner服务的IP地址
        if ("-s".equals(argument)) {
            if (index == args.length) {
                printUsage("Missing Server after -s");
                return null;
            }
            hostname = args[index++];
        //匹配-p参数 MonkeyRunner服务的TCP端口
        } else if ("-p".equals(argument)) {
            // quick check on the next argument.
            if (index == args.length) {
                printUsage("Missing Server port after -p");
                return null;
            }
            port = Integer.parseInt(args[index++]);
        //匹配-v参数 指Log的级别
        } else if ("-v".equals(argument)) {
            // quick check on the next argument.
            if (index == args.length) {
                printUsage("Missing Log Level after -v");
                return null;
            }

            logLevel = Level.parse(args[index++]);
        } else if ("-be".equals(argument)) {
            // quick check on the next argument.
            if (index == args.length) {
                printUsage("Missing backend name after -be");
                return null;
            }
            backend = args[index++];
         //MonkeyRunner的插件
        } else if ("-plugin".equals(argument)) {
            // quick check on the next argument.
            if (index == args.length) {
                printUsage("Missing plugin path after -plugin");
                return null;
            }
            //判断给定的参数是否存在
            File plugin = new File(args[index++]);
            if (!plugin.exists()) {
                printUsage("Plugin file doesn't exist");
                return null;
            }
            //判断给定的参数是否可读
            if (!plugin.canRead()) {
                printUsage("Can't read plugin file");
                return null;
            }
            //添加到插件的列表中
            pluginListBuilder.add(plugin);
        } else if ("-u".equals(argument)){
            // This is asking for unbuffered input. We can ignore this.
            //如果脚本文件不存在，但是还有带-的参数，提示不识别的参数，报错返回
        } else if (argument.startsWith("-") &&
            // Once we have the scriptfile, the rest of the arguments go to jython.
            scriptFile == null) {
            // we have an unrecognized argument.
            printUsage("Unrecognized argument: " + argument + ".");
            return null;
        } else {
            //如果脚本文件为null，则添加当前的参数为脚本文件的路径
            //如果指定了monkeyrunner运行的脚本文件，其余的参数都识别为传给这个脚本文件的参数
            if (scriptFile == null) {
                // get the filepath of the script to run.
                // This will be the last undashed argument.
                //判断脚本文件是否存在
                scriptFile = new File(argument);
                if (!scriptFile.exists()) {
                    printUsage("Can't open specified script file");
                    return null;
                }
                 //判断脚本文件是否可读
                if (!scriptFile.canRead()) {
                    printUsage("Can't open specified script file");
                    return null;
                }
            } else {
                //添加脚本文件的参数到参数集合中
                argumentBuilder.add(argument);
            }
        }
    };
    //根据解析处理的参数，构建出MonkeyRunnerOptions对象
    return new MonkeyRunnerOptions(hostname, port, scriptFile, backend, logLevel,
             pluginListBuilder.build(), argumentBuilder.build());
 }
```

下面是MonkeyRunnerOptions的构造函数和类成员变量的定义：

```java 
private static final Logger LOG = Logger.getLogger(MonkeyRunnerOptions.class.getName());
private static String DEFAULT_MONKEY_SERVER_ADDRESS = "127.0.0.1";
private static int DEFAULT_MONKEY_PORT = 12345;

private final int port;
private final String hostname;
private final File scriptFile;
private final String backend;
private final Collection<File> plugins;
private final Collection<String> arguments;
private final Level logLevel;

private MonkeyRunnerOptions(String hostname, int port, File scriptFile, String backend,
        Level logLevel, Collection<File> plugins, Collection<String> arguments) {
    this.hostname = hostname;
    this.port = port;
    this.scriptFile = scriptFile;
    this.backend = backend;
    this.logLevel = logLevel;
    this.plugins = plugins;
    this.arguments = arguments;
}

public int getPort() {
    return port;
}

public String getHostname() {
    return hostname;
}

public File getScriptFile() {
    return scriptFile;
}

public String getBackendName() {
    return backend;
}

public Collection<File> getPlugins() {
    return plugins;
}

public Collection<String> getArguments() {
    return arguments;
}

public Level getLogLevel() {
    return logLevel;
}
```

如果解析的参数无误，那么进入下一步的逻辑处理，也就是replaceAllLogFormatters方法。

```java 
/* 
 * Similar to above, when this fails, it no longer throws a
 * runtime exception, but merely will log the failure.
 * 
 * 主要设置Log的格式以及输出级别
 */
private static final void replaceAllLogFormatters(Formatter form, Level level) {
    LogManager mgr = LogManager.getLogManager();
    Enumeration<String> loggerNames = mgr.getLoggerNames();
    while (loggerNames.hasMoreElements()) {
        String loggerName = loggerNames.nextElement();
        Logger logger = mgr.getLogger(loggerName);
        for (Handler handler : logger.getHandlers()) {
            handler.setFormatter(form);
            handler.setLevel(level);
        }
    }
}
```

设置好了之后，创建MonkeyRunnerStarter对象：

```java 
public class MonkeyRunnerStarter {
    private static final Logger LOG = Logger.getLogger(MonkeyRunnerStarter.class.getName());
    private static final String MONKEY_RUNNER_MAIN_MANIFEST_NAME = "MonkeyRunnerStartupRunner";

    private final ChimpChat chimp;
    private final MonkeyRunnerOptions options;

    public MonkeyRunnerStarter(MonkeyRunnerOptions options) {
        Map<String, String> chimp_options = new TreeMap<String, String>();
        chimp_options.put("backend", options.getBackendName());
        this.options = options;
        this.chimp = ChimpChat.getInstance(chimp_options);
        MonkeyRunner.setChimpChat(chimp);
    }
    ...
}
```

可以看出，options传给了ChimpChat，ChimpChat可以堪称是monkeyruner与设备通信，如发送事件的一个通信组件，后面有章节会介绍到其的具体实现，然后调用MonkeyRunnerStarter的run方法：

```java 
private int run() {
    // This system property gets set by the included starter script
    String monkeyRunnerPath = System.getProperty("com.android.monkeyrunner.bindir") +
            File.separator + "monkeyrunner";

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

run()方法其实做的主要是两件事情，第一件：加载插件，hanlePlugins；第二件：根据给定的参数，如果包含用Python写的脚本文件，则会执行脚本文件里面的内容；如果没有指定脚本文件，mokeyrunner则会以交互的方式运行，就像python的交互模式一样。

下一章节主要介绍handlePlugins()的实现逻辑。

