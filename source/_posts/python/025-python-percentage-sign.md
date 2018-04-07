---
title: Python中字符串格式化之%
date: 2018-04-07 14:55:08
tags:
categories: "Python学习笔记"
---

Python中字符的格式化主要有%和.format两种方式，这两种方式在Python2和Python3中都适用，这篇文章主要介绍%的字符串格式化。

### 模板

格式化字符串时，Python使用一个字符串作为模板。模板中有格式符，这些格式符为真实值预留位置，并说明真实数值应该呈现的格式。Python用一个tuple将多个值传递给模板，每个值对应一个格式符。

比如下面的例子：

```python
>>> print("I'm %s. I'm %d year old" % ('Viking Den', 29))
I'm Viking Den. I'm 29 year old
>>>
```

上面的例子中，

“I’m %s. I’m %d year old” 为我们的模板。%s为第一个格式符，表示一个字符串。%d为第二个格式符，表示一个整数。('Viking Den', 29)的两个元素’Viking Den’和99为替换%s和%d的真实值。在模板和tuple之间，有一个%号分隔，它代表了格式化操作。

整个”I’m %s. I’m %d year old” % ('Viking Den', 29) 实际上构成一个字符串表达式。我们可以像一个正常的字符串那样，将它赋值给某个变量。比如:

```python
>>> text = "I'm %s. I'm %d year old" % ('Viking Den', 29)
>>> print(text)
I'm Viking Den. I'm 29 year old
>>> 
```

我们还可以用词典来传递真实值。如下：

```python
>>> print("I'm %(name)s. I'm %(age)d year old" % {'name':'Viking Den','age':29})
I'm Viking Den. I'm 29 year old
>>>
```

可以看到，我们对两个格式符进行了命名。命名使用()括起来。每个命名对应词典的一个key。

### 格式符

格式符为真实值预留位置，并控制显示的格式。

#### 参数格式

>**%[(name)][flags][width].[precision]typecode**

* [(name)]

可选，用于选择指定的key

* [flags]

可选，可供选择的值有:

>+	    右对齐；正数前加正好，负数前加负号
>-	    左对齐；正数前无符号，负数前加负号
>space	右对齐；正数前加空格，负数前加负号
>0	    右对齐；正数前无符号，负数前加负号；用0填充空白处

* [width]

可选，占有宽度

* .[precision]

可选，小数点后保留的位数

* typecode

必选，参数如下：

>* %s    字符串 (采用str()的显示)

>* %r    字符串 (采用repr()的显示)

>* %c    单个字符

>* %b    二进制整数

>* %d    十进制整数

>* %i    十进制整数

>* %o    八进制整数

>* %x    十六进制整数

>* %e    指数 (基底写为e)

>* %E    指数 (基底写为E)

>* %f    浮点数

>* %F    浮点数，与上相同

>* %g    指数(e)或浮点数 (根据显示长度)

>* %G    指数(E)或浮点数 (根据显示长度)

>* %%    字符”%”

例子：

* 去浮点数后面的位数

```python
# %.2f小数后面只取两位
>>> string = "percent %.2f" % 99.97623
>>> string
'percent 99.98'

# 给浮点数起一个名字(key)
>>> string = "percent %(pp).2f" % {"pp": 99.97623}
>>> string
'percent 99.98'

# 两个百分号代表一个百分号
>>> string = "percent %(pp).2f%%" % {"pp": 99.97623}
>>> string
'percent 99.98%'
```

### 参考引用

[Python 快速教程（补充篇05）：字符串格式化 (%操作符)](http://python.jobbole.com/82673/)

[Python全栈之路系列之字符串格式化](https://segmentfault.com/a/1190000008285514)