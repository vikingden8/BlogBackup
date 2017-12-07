---
title: Android MonkeyRunner
date: 2017-12-7 23:00:48
tags:
categories: "测试开发"
---

The monkeyrunner tool provides an API for writing programs that control an Android device or emulator from outside of 
Android code. With monkeyrunner, you can write a Python program that installs an Android application or test package, 
runs it, sends keystrokes to it, takes screenshots of its user interface, and stores screenshots on the workstation. 
The monkeyrunner tool is primarily designed to test applications and devices at the functional/framework level and for 
running unit test suites, but you are free to use it for other purposes.

monkeyrunner工具提供了一套API在Android框架代码以外通过编写程序来控制Android真机和模拟器。通过monkeyrunner，你能通过编写一个Python程序
来安装Android应用或者测试应用，启动，发送按键事件，截取当前界面，以及保存截图到本地工作机器。monkeyrunner工具最初设计是用来测试功能和框架层
的应用和设备，并用来进行单元测试，但是，你也可以用它作其他用途。

<!--more-->

The monkeyrunner tool is not related to the UI/Application Exerciser Monkey, also known as the monkey tool. The monkey 
tool runs in an adb shell directly on the device or emulator and generates pseudo-random streams of user and system 
events. In comparison, the monkeyrunner tool controls devices and emulators from a workstation by sending specific 
commands and events from an API.

monkeyrunner工具和另外一个monkey工具Monkey没有联系。Monkey工具是通过adb的方式直接运行在设备或模拟器上，生成伪随机的用户和系统事件流。相反的，
monkeyrunner工具在工作站（本地电脑）使用API发送特殊的命令和事件控制设备和模拟器。

The monkeyrunner tool provides these unique features for Android testing:

monkeyrunner工具提供以下功能：

* **Multiple device control**: The monkeyrunner API can apply one or more test suites across multiple devices or emulators. 
You can physically attach all the devices or start up all the emulators (or both) at once, connect to each one in turn 
programmatically, and then run one or more tests. You can also start up an emulator configuration programmatically, run 
one or more tests, and then shut down the emulator.

* **多设备控制**：通过monkeyrunner API能让一个或多个测试集运行在多台真机或模拟器上。你可以同时获得所有的真机或启动所有的模拟器，在代码中连接
指定的设备，然后运行一个或多个测试。你也可以通过代码配置模拟器的参数，运行一个或多个测试，测试完成后关闭模拟器。

* **Functional testing**: monkeyrunner can run an automated start-to-finish test of an Android application. You provide input
 values with keystrokes or touch events, and view the results as screenshots.
 
* **功能测试**: monkeyrunner可以对一个应用运行完整的测试.你只需要提供按键和触摸事件，通过截图的图片来浏览结果。
 
* **Regression testing**： monkeyrunner can test application stability by running an application and comparing its output 
screenshots to a set of screenshots that are known to be correct.

* **回归测试**：monkeyrunner能被用来测试应用的稳定性，只需提供事前准备好的一系列正确截图，然后在运行应用时通过比对图片来验证。

* **Extensible automation**： Since monkeyrunner is an API toolkit, you can develop an entire system of Python-based modules
 and programs for controlling Android devices. Besides using the monkeyrunner API itself, you can use the standard Python
  os and subprocess modules to call Android tools such as Android Debug Bridge.
  
* **扩展自动化**：因为monkeyrunner是一API套件，你可以开发一个基于Python的完整系统模块来控制Android设备。除了使用monkeyrunner API，你还可以使用
标准的Python语言以及子模块来调用Android工具，比如说Android调试桥。
  
You can also add your own classes to the monkeyrunner API. This is described in more detail in the section Extending 
monkeyrunner with plugins.

你也可以添加你自己的类到monkeyrunner API中。这将在下面的通过插件的方式继承monkeyrunner介绍到。

The monkeyrunner tool uses Jython, a implementation of Python that uses the Java programming language. Jython allows the
 monkeyrunner API to interact easily with the Android framework. With Jython you can use Python syntax to access the 
 constants, classes, and methods of the API.
 
monkeyrunner工具使用Jython（使用Java语言来实现的Python）。Jython允许monkeyrunner API轻松地和Android框架进行交互。通过Jython你可以使用
Python语法来使用monkeyrunner API的常量，类和方法。

### A Simple monkeyrunner Program 简单的monkeyrunner程序

Here is a simple monkeyrunner program that connects to a device, creating a MonkeyDevice object. Using the MonkeyDevice 
object, the program installs an Android application package, runs one of its activities, and sends key events to the 
activity. The program then takes a screenshot of the result, creating a MonkeyImage object. From this object, the program 
writes out a .png file containing the screenshot.

下面的程序使用monkeyrunner来连接一个设备，创建MonkeyDevice对象。通过MonkeyDevice实例对象来安装一个Android应用，启动它的界面，并且发送按键事件到
该界面。然后程序截取当前屏幕，其返回值创建了一MonkeyImage对象。通过这个MonkeyImage对象把当前的截图保存到一个.png的文件中。

```python 
# Imports the monkeyrunner modules used by this program
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice

# Connects to the current device, returning a MonkeyDevice object
device = MonkeyRunner.waitForConnection()

# Installs the Android package. Notice that this method returns a boolean, so you can test
# to see if the installation worked.
device.installPackage('myproject/bin/MyApplication.apk')

# sets a variable with the package's internal name
package = 'com.example.android.myapplication'

# sets a variable with the name of an Activity in the package
activity = 'com.example.android.myapplication.MainActivity'

# sets the name of the component to start
runComponent = package + '/' + activity

# Runs the component
device.startActivity(component=runComponent)

# Presses the Menu button
device.press('KEYCODE_MENU', MonkeyDevice.DOWN_AND_UP)

# Takes a screenshot
result = device.takeSnapshot()

# Writes the screenshot to a file
result.writeToFile('myproject/shot1.png','png')

```

### The monkeyrunner API 

The monkeyrunner API is contained in three modules in the package _com.android.monkeyrunner_:

monkeyrunner API位于 _com.android.monkeyrunner_ 包下，包含有下面的三个模块:

* **MonkeyRunner**: A class of utility methods for monkeyrunner programs. This class provides a method for connecting monkeyrunner 
to a device or emulator. It also provides methods for creating UIs for a monkeyrunner program and for displaying the built-in help.

* **MonkeyRunner**: monkeyrunner程序的公用方法类。这个类主要提供了连接真机或模拟器。同时，也提供了创建界面和显示帮助信息。

* **MonkeyDevice**: Represents a device or emulator. This class provides methods for installing and uninstalling packages, 
starting an Activity, and sending keyboard or touch events to an application. You also use this class to run test packages.

* **MonkeyDevice**: 代表一台真机或模拟器。这个类主要提供了安装或卸载应用，启动应用界面，以及发送按键或触摸事件到应用程序。你也可以通过这个类来
运行测试应用。

* **MonkeyImage**: Represents a screen capture image. This class provides methods for capturing screens, converting bitmap 
images to various formats, comparing two MonkeyImage objects, and writing an image to a file.

* **MonkeyImage**: 代表当前屏幕的截图。这个类提供了截图当前屏幕，多种位图格式转换，比较两个MonkeyImage对象，以及保存图像到文件。

In a Python program, you access each class as a Python module. The monkeyrunner tool does not import these modules automatically.
To import a module, use the Python from statement:

在Python程序中，你通过Python模块的方式访问以上的每一个类。monkeyrunner工具不支持自动导入这些模块。你可以通过Python的import申明导入：

```python 
from com.android.monkeyrunner import <module>
```

where <module> is the class name you want to import. You can import more than one module in the same from statement by 
separating the module names with commas.

这里的<module>就是你要导入的类名。你可以在一个import语句中导入多个module，只需通过逗号分隔即可。

### Running monkeyrunner 运行monkeyrunner

You can either run monkeyrunner programs from a file, or enter monkeyrunner statements in an interactive session. You do 
both by invoking the monkeyrunner command which is found in the tools/ subdirectory of your SDK directory. If you provide 
a filename as an argument, the monkeyrunner command runs the file's contents as a Python program; otherwise, it starts an 
interactive session.

你可以通过指定一个文件来运行monkeyrunner，也可以进入monkeyrunner的交互模式。两种方式都是通过SDK中的tools目录下的monkeyrunner命令。如果
你指定了运行的文件名，那么monkeyrunner命令将会对文件的内容以Python程序处理；否则，它将会启动一个交互模式。

The syntax of the monkeyrunner command is：

monkeyrunner的语法如下：

```python 
monkeyrunner -plugin <plugin_jar> <program_filename> <program_options>
```

* -plugin <plugin_jar> : (Optional) Specifies a .jar file containing a plugin for monkeyrunner. To learn more about 
monkeyrunner plugins, see Extending monkeyrunner with plugins. To specify more than one file, include the argument multiple times.

* -plugin <plugin_jar> : (可选)指定以.jar结尾的文件当作monkeyrunner的插件。关于monkeyrunner插件，可以后面的内容。如果要指定多个插件，
只需这个参数指定多次即可。

* <program_filename>：If you provide this argument, the monkeyrunner command runs the contents of the file as a Python 
program. If the argument is not provided, the command starts an interactive session.

* <program_filename>：如果提供了这个参数，monkeyrunner命令将会把文件的内容当作Python程序来处理。如果这个参数没有指定，那么monkeyrunner将
会以交互的方式运行。

* <program_options>：(Optional) Flags and arguments for the program in <program_file>.

* <program_options>：(可选) <program_file>的运行参数。

### monkeyrunner Built-in Help 内建的帮助信息

You can generate an API reference for monkeyrunner by running:

你可以通过下面的方式来生成monkeyrunner的API文档：

```python 
monkeyrunner help.py <format> <outfile>
```

The arguments are:

两个参数分别是：

* <format> is either text for plain text output or html for HTML output.

* <format>指定生成的格式是text还是html。

* <outfile> is a path-qualified name for the output file.

* <outfile>指定要生成的文件路径。