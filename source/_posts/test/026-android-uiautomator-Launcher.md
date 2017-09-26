---
title: Android UiAutomator Launcher
date: 2017-09-26 22:31:48
tags:
categories: "测试开发"
---

由于UiAutomator在启动的时候,需要调用uiautomator的命令,由于android的特性,这个命令肯定是在/system/bin当中的,于是,我们从bin当中讲这个命令导出,准确的来讲,这是一个shell脚本,定义了环境变量和参数后,然后在启动uiautomator架包当中的执行类Launcher.

<!--more-->

### uiautomator脚本

```
#
# Copyright (C) 2012 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# Script to start "uiautomator" on the device
#
# The script does a couple of things:
# * Use an alternative dalvik cache when running as non-root. Jar file needs
#   to be dexopt'd to run in Dalvik. For plain jar files, this is done at first
#   use. shell user does not have write permission to default system Dalvik
#   cache so we redirect to an alternative cache
# * special processing for subcommand 'runtest':
#    * '--nohup' allows process continue to run even if parent process that
#      started it has already terminated. We parse for this parameter and set
#      signal trap. This is useful for testing with USB disconnected
#    * all jar files that the test classes resides in, or dependent on are
#      provided on command line and exported to CLASSPATH environment variable
#      before starting the Java code. This offloads the task of class loading
#      and resolving of cross jar class dependency to Dalvik
#    * all other subcommand or options are directly passed into Java code for
#      further parsing

export run_base=/data/local/tmp
export base=/system

# if not running as root, trick dalvik into using an alternative dex cache
if [ ${USER_ID} -ne 0 ]; then
  tmp_cache=${run_base}/dalvik-cache

  if [ ! -d ${tmp_cache} ]; then
    mkdir -p ${tmp_cache}
  fi

  export ANDROID_DATA=${run_base}
fi

# take first parameter as the command
cmd=${1}

if [ -z "${1}" ]; then
  cmd="help"
fi

# strip the command parameter
if [ -n "${1}" ]; then
  shift
fi

CLASSPATH=/system/framework/android.test.runner.jar:${base}/framework/uiautomator.jar

# eventually args will be what get passed down to Java code
args=
# we also pass the list of jar files, so we can extract class names for tests
# if they are not explicitly specified
jars=

# special case pre-processing for 'runtest' command
if [ "${cmd}" == "runtest" ]; then
  # Print deprecation warning
  echo "Warning: This version of UI Automator is deprecated. New tests should be written using"
  echo "UI Automator 2.0 which is available as part of the Android Testing Support Library."
  echo "See https://developer.android.com/training/testing/ui-testing/uiautomator-testing.html"
  echo "for more details."
  # first parse the jar paths
  while [ true ]; do
    if [ -z "${1}" ] && [ -z "${jars}" ]; then
      echo "Error: more parameters expected for runtest; please see usage for details"
      cmd="help"
      break
    fi
    if [ -z "${1}" ]; then
      break
    fi
    jar=${1}
    if [ "${1:0:1}" = "-" ]; then
      # we are done with jars, starting with parameters now
      break
    fi
    # if relative path, append the default path prefix
    if [ "${1:0:1}" != "/" ]; then
      jar=${run_base}/${1}
    fi
    # about to add the file to class path, check if it's valid
    if [ ! -f ${jar} ]; then
      echo "Error: ${jar} does not exist"
      # force to print help message
      cmd="help"
      break
    fi
    jars=${jars}:${jar}
    # done processing current arg, moving on
    shift
  done
  # look for --nohup: if found, consume it and trap SIG_HUP, otherwise just
  # append the arg to args
  while [ -n "${1}" ]; do
    if [ "${1}" = "--nohup" ]; then
      trap "" HUP
      shift
    else
      args="${args} ${1}"
      shift
    fi
  done
else
  # if cmd is not 'runtest', just take the rest of the args
  args=${@}
fi

args="${cmd} ${args}"
if [ -n "${jars}" ]; then
   args="${args} -e jars ${jars}"
fi

CLASSPATH=${CLASSPATH}:${jars}
export CLASSPATH
exec app_process ${base}/bin com.android.commands.uiautomator.Launcher ${args}
```

源码地址：[uiautomator](https://github.com/android/platform_frameworks_base/blob/master/cmds/uiautomator/cmds/uiautomator/uiautomator)

### Launcher分析

由上面的uiautomator脚本可以看出，最终解析出来的参数存在args中传给了com.android.commands.uiautomator.Launcher，这个就是UiAutomator函数的主入口．

```java
public static void main(String[] args) {
    // show a meaningful process name in `ps`
    Process.setArgV0("uiautomator");
    if (args.length >= 1) {
        Command command = findCommand(args[0]);
        if (command != null) {
            String[] args2 = {};
            if (args.length > 1) {
                // consume the first arg
                args2 = Arrays.copyOfRange(args, 1, args.length);
            }
            command.run(args2);
            return;
        }
    }
    HELP_COMMAND.run(args);
}
```

在/framework/base/cmds/uiautomator的源码中,我们找到这个Launcher类,主函数入口main方法，首先看到的就是process.argv0,这行代码,正如注释中所标注的那样,将启动的进程定义为uiautomator.再检查参数，如果参数不合法，则显示帮助信息：

```java
/**
 * A simple abstraction class for supporting generic sub commands
 */
public static abstract class Command {
    private String mName;

    public Command(String name) {
        mName = name;
    }

    /**
     * Returns the name of the sub command
     * @return
     */
    public String name() {
        return mName;
    }

    /**
     * Returns a one-liner of the function of this command
     * @return
     */
    public abstract String shortHelp();

    /**
     * Returns a detailed explanation of the command usage
     *
     * Usage may have multiple lines, indentation of 4 spaces recommended.
     * @return
     */
    public abstract String detailedOptions();

    /**
     * Starts the command with the provided arguments
     * @param args
     */
    public abstract void run(String args[]);
}

private static Command HELP_COMMAND = new Command("help") {
    @Override
    public void run(String[] args) {
        System.err.println("Usage: uiautomator <subcommand> [options]\n");
        System.err.println("Available subcommands:\n");
        for (Command command : COMMANDS) {
            String shortHelp = command.shortHelp();
            String detailedOptions = command.detailedOptions();
            if (shortHelp == null) {
                shortHelp = "";
            }
            if (detailedOptions == null) {
                detailedOptions = "";
            }
            System.err.println(String.format("%s: %s", command.name(), shortHelp));
            System.err.println(detailedOptions);
        }
    }

    @Override
    public String detailedOptions() {
        return null;
    }

    @Override
    public String shortHelp() {
        return "displays help message";
    }
};
```
打印出来的帮助信息类似如下：

```sh
viking@Viking ~/work/blog/BlogBackup $ adb shell uiautomator -h
Usage: uiautomator <subcommand> [options]

Available subcommands:

help: displays help message

runtest: executes UI automation tests
    runtest <class spec> [options]
    <class spec>: <JARS> < -c <CLASSES> | -e class <CLASSES> >
      <JARS>: a list of jar files containing test classes and dependencies. If
        the path is relative, it's assumed to be under /data/local/tmp. Use
        absolute path if the file is elsewhere. Multiple files can be
        specified, separated by space.
      <CLASSES>: a list of test class names to run, separated by comma. To
        a single method, use TestClass#testMethod format. The -e or -c option
        may be repeated. This option is not required and if not provided then
        all the tests in provided jars will be run automatically.
    options:
      --nohup: trap SIG_HUP, so test won't terminate even if parent process
               is terminated, e.g. USB is disconnected.
      -e debug [true|false]: wait for debugger to connect before starting.
      -e runner [CLASS]: use specified test runner class instead. If
        unspecified, framework default runner will be used.
      -e <NAME> <VALUE>: other name-value pairs to be passed to test classes.
        May be repeated.
      -e outputFormat simple | -s: enabled less verbose JUnit style output.

dump: creates an XML dump of current UI hierarchy
    dump [--verbose][file]
      [--compressed]: dumps compressed layout information.
      [file]: the location where the dumped XML should be stored, default is
      /sdcard/window_dump.xml

events: prints out accessibility events until terminated

viking@Viking ~/work/blog/BlogBackup $
```

在主类中,能看见一个内部静态抽象类,此类的作用就是不允许外部实例化,只能内部调用,当然,这保证了它的安全性,这也就是为什么不写一个接口,而使用了该内部静态抽象类的原因,如果使用了接口,是否会在uiautomator中留下一个后门呢?

这个内部抽象类也是用来给各种类型的命令类继承用的，那首先，我们可以看到找不到参数时，它会执行一个HELP_COMMAND.run的方法，，它在主类使用匿名的方式构造了一个Command的HELP_COMMAND对象，并重写了该类的方法，以便对这个对象直接引用。

如果参数的长度大于1的时候，那么，它则先根据第一个参数args[0]去寻找相对应的方法。接着，我们就来到了findCommand这个方法当中。

```java
private static Command findCommand(String name) {
    for (Command command : COMMANDS) {
        if (command.name().equals(name)) {
            return command;
        }
    }
    return null;
}

private static Command[] COMMANDS = new Command[] {
    HELP_COMMAND,
    new RunTestCommand(),
    new DumpCommand(),
    new EventsCommand(),
};

```

下面主要去看下RunTestCommand，DumpCommand，EventsCommand类的实现．
