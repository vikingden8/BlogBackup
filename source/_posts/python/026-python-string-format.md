---
title: Python使用{}和format的字符串格式化
date: 2018-04-07 20:47:08
tags:
categories: "Python学习笔记"
---

### 语法：

> **[[fill]align][sign][#][0][width][,][.precision][type]**

#### [fill]

可选，空白处填充的字符

#### [align]

可选，对齐方式（需配合width使用）

参数说明

* <	强制内容左对齐
* \>	强制内容右对齐(默认)
* ＝	强制内容右对齐，将符号放置在填充字符的左侧，且只对数字类型有效。 即使：符号+填充物+数字
* ^	强制内容居中

<!--more-->

#### [sign]

可选，有无符号数字

参数说明

* +	正号加正，负号加负
* -	正号不变，负号加负
* space	正号空格，负号加负

#### [#]

可选，对于二进制、八进制、十六进制，如果加上#，会显示 0b/0o/0x，否则不显示

#### [,]

可选，为数字添加分隔符，如：1,000,000

#### [width]

可选，格式化位所占宽度

#### [.precision]

可选，小数位保留精度

#### [type]

可选，格式化类型

##### 传入” 字符串类型 “的参数

参数说明

* s	格式化字符串类型数据
* 空白	未指定类型，则默认是None，同s

##### 传入“ 整数类型 ”的参数

参数说明

* b	将10进制整数自动转换成2进制表示然后格式化
* c	将10进制整数自动转换为其对应的unicode字符
* d	十进制整数
* o	将10进制整数自动转换成8进制表示然后格式化；
* x	将10进制整数自动转换成16进制表示然后格式化（小写x）
* X	将10进制整数自动转换成16进制表示然后格式化（大写X）

##### 传入“ 浮点型或小数类型 ”的参数

参数说明

* e	转换为科学计数法（小写e）表示，然后格式化；
* E	转换为科学计数法（大写E）表示，然后格式化;
* f	转换为浮点型（默认小数点后保留6位）表示，然后格式化；
* F	转换为浮点型（默认小数点后保留6位）表示，然后格式化；
* g	自动在e和f中切换
* G	自动在E和F中切换
* %	显示百分比（默认显示小数点后6位）

### 实例

* 第一种基本format格式化方式

```python
>>> text = 'My name is {}, I am {} years old, {} engineer'.format('Viking Den', 29, 'Python')
>>> print(text)
My name is Viking Den, I am 29 years old, Python engineer
>>>
```

* 第二种基本format格式化方式

```python
>>> text = 'My name is {}, I am {} years old, {} engineer'.format(*['Viking Den', 29, 'Python'])
>>> print(text)
My name is Viking Den, I am 29 years old, Python engineer
>>>
```

* 给传入的参数加一个索引

```python
>>> text = 'My name is {0}, I am {1} years old, {2} engineer'.format(*['Viking Den', 29, 'Python'])
>>> print(text)
My name is Viking Den, I am 29 years old, Python engineer
>>> 
```

* 给参数起一个名字(key)

```python
>>> text = 'My name is {name}, I am {age} years old, {language} engineer'.format(name='Viking Den', age=29, language='Python')
>>> print(text)
My name is Viking Den, I am 29 years old, Python engineer
>>> 
```

* 字典的方式

```python
>>> text = 'My name is {name}, I am {age} years old, {language} engineer'.format(**{'name':'Viking Den', 'age':29, 'language':'Python'})
>>> print(text)
My name is Viking Den, I am 29 years old, Python engineer
>>>
```

* 制定参数类型

```python
>>> text = 'My name is {:s}, I am {:d} years old.'.format('Viking Den', 29, )
>>> print(text)
My name is Viking Den, I am 29 years old.
>>>
```

* 制定名称(key)的值类型

```python
>>> text = 'My name is {name:s}, I am {age:d} years old.'.format(name='Viking Den', age=29)
>>> print(text)
My name is Viking Den, I am 29 years old.
>>> # 或者字典的方式
>>> text = 'My name is {name:s}, I am {age:d} years old.'.format(**{'name':'Viking Den', 'age':29})
>>> print(text)
My name is Viking Den, I am 29 years old.
>>>
```

* 进制实例

```python
>>> nums = "numbers: {:b},{:o},{:d},{:x},{:X}, {:%}, {}".format(15, 15, 15, 15, 15, 0.87623, 2)
>>> print(nums)
numbers: 1111,17,15,f,F, 87.623000%, 2
>>>
```

### 参考资料

[Python全栈之路系列之字符串格式化](https://segmentfault.com/a/1190000008285514)