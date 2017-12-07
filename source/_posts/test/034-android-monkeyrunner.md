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

###