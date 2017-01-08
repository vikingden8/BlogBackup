---
title: Python的文件类型
date: 2017-01-08 09:36:11
tags:
categories: "Python学习笔记"
---

Python只有三种文件类型.py，.pyc，.pyo.

* .py

  这就是我们常见的py源文件，没什么好说的

* .pyc

 py源文件编译成的二进制字节码文件，依然由python加载执行，不过速度会提高，也会隐藏源码。

 如何生成.pyc文件呢？

 假设我们有一个1.py文件需要编译成pyc文件，则创建一个新的py文件，在新的文件中使用py_compile，如下：

 ![](/images/categories/python/python-file-type/type_01.png)

 <!--more-->

 执行之后，可以看到生成了1.pyc的文件，我们可以使用python 1.pyc执行：

 ![](/images/categories/python/python-file-type/type_02.png)

 * .pyo

 优化编译后的程序，也是二进制文件，适用于嵌入式系统。

 如何生成.pyo文件呢？我们可以使用下面的命令：

 ![](/images/categories/python/python-file-type/type_03.png)
