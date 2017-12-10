---
title: Android MonkeyRunner源码解读（三）
date: 2017-12-9 11:26:48
tags:
categories: "测试开发"
---

本章节翻译自Android官方文档： [Extending monkeyrunner with Plugins](https://developer.android.com/studio/test/monkeyrunner/index.html#Plugins)

You can extend the monkeyrunner API with classes you write in the Java programming language and build into one or more .jar files. You can use this feature to extend the monkeyrunner API with your own classes or to extend the existing classes. You can also use this feature to initialize the monkeyrunner environment.

可以通过编写Java程序生成.jar文件来扩展monkeyrunner API。你可以使用这个特性来扩展你自己的类，或者扩展现有的monkeyrunner API。同时，你也可以使用这个特性来初始化monkeyrunner环境。

To provide a plugin to monkeyrunner, invoke the monkeyrunner command with the -plugin <plugin_jar> argument described in table 1.

提供一个插件给monkeyrunner，通过monkeyrunner的-plugin <plugin_jar>参数来指定，后面的plugin_jar表示的是插件的路径。

<!--more-->

In your plugin code, you can import and extend the the main monkeyrunner classes MonkeyDevice, MonkeyImage, and MonkeyRunner in com.android.monkeyrunner (see The monkeyrunner API).

在你的插件代码中，你可以导入和继承现有的MonkeyDevice，MonkeyImage和MonkeyRunner类。

Note that plugins do not give you access to the Android SDK. You can't import packages such as com.android.app. This is because monkeyrunner interacts with the device or emulator below the level of the framework APIs.
 
需要注意的是，所有的插件没有访问Android SDK的权限。也就是你不能导入像com.android.app这样的包。这是因为monkeyrunner与真机或模拟器交互是在框架层之下的。

### The plugin startup class 插件启动类

The .jar file for a plugin can specify a class that is instantiated before script processing starts. To specify this class, add the key MonkeyRunnerStartupRunner to the .jar file's manifest. The value should be the name of the class to run at startup. The following snippet shows how you would do this within an ant build script:

.jar的插件可以指定一个类在脚本处理之前加载。你只需要在这个.jar的manifest xml文件中定义一个MonkeyRunnerStartupRunner的属性。值就为你要启动的那个全类名。下面的片段示例是使用ant打包程序的配置：

```xml 
<jar jarfile="myplugin" basedir="${build.dir}">
<manifest>
<attribute name="MonkeyRunnerStartupRunner" value="com.myapp.myplugin"/>
</manifest>
</jar>
```

To get access to monkeyrunner's runtime environment, the startup class can implement com.google.common.base.Predicate<PythonInterpreter>. For example, this class sets up some variables in the default namespace:

为了能访问monkeyrunner的运行时环境，你的启动类可以实现com.google.common.base.Predicate<PythonInterpreter>这个类。如下，这个类设置了几个默认的变量值：

```java 
package com.android.example;

import com.google.common.base.Predicate;
import org.python.util.PythonInterpreter;

public class Main implements Predicate<PythonInterpreter> {
    @Override
    public boolean apply(PythonInterpreter anInterpreter) {

        /*
        * Examples of creating and initializing variables in the monkeyrunner environment's
        * namespace. During execution, the monkeyrunner program can refer to the variables "newtest"
        * and "use_emulator"
        *
        */
        anInterpreter.set("newtest", "enabled");
        anInterpreter.set("use_emulator", 1);

        return true;
    }
}
```

其实从上一章节也可以看出，你的类也可以直接实现Runnable类。

monkeyrunner的插件我自己也没编写过，有兴趣的可以自行试试，下一章节看下monkeyrunner交互模式的实现。