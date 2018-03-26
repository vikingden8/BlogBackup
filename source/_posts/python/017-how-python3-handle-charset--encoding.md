---
title: Python3是如何解决棘手的字符编码问题的？
date: 2018-03-26 20:10:08
tags:
categories: "Python学习笔记"
---

转载自：[Python3 是如何解决棘手的字符编码问题的？](https://foofish.net/how-python3-handle-charset-encoding.html)

Python3 最重要的一项改进之一就是解决了 Python2 中字符串与字符编码遗留下来的这个大坑。[Python 编码为什么那么蛋疼？](https://foofish.net/why-python-encoding-is-tricky.html)已经介绍过 Python2 字符串设计上的一些缺陷：

* 使用 ASCII 码作为默认编码方式，对中文处理很不友好。
* 把字符串的牵强地分为 unicode 和 str 两种类型，误导开发者

当然这并不算 Bug，只要处理的时候多留心也可以避免这些坑。但在 Python3 两个问题都很好的解决了。

首先，Python3 把系统默认编码设置为 UTF-8

```python
>>> import sys
>>> sys.getdefaultencoding()
'utf-8'
>>>
```

然后，文本字符和二进制数据区分得更清晰，分别用 str 和 bytes 表示。文本字符全部用 str 类型表示，str 能表示 Unicode 字符集中所有字符，而二进制字节数据用一种全新的数据类型，用 bytes 来表示。

<!--more-->

### str

```python
>>> a = "a"
>>> a
'a'
>>> type(a)
<class 'str'>

>>> b = "禅"
>>> b
'禅'
>>> type(b)
<class 'str'>
```

### bytes

Python3 中，在字符引号前加‘b’，明确表示这是一个 bytes 类型的对象，实际上它就是一组二进制字节序列组成的数据，bytes 类型可以是 ASCII范围内的字符和其它十六进制形式的字符数据，但不能用中文等非ASCII字符表示。

```python
>>> c = b'a'
>>> c
b'a'
>>> type(c)
<class 'bytes'>

>>> d = b'\xe7\xa6\x85'
>>> d
b'\xe7\xa6\x85'
>>> type(d)
<class 'bytes'>
>>>

>>> e = b'禅'
  File "<stdin>", line 1
SyntaxError: bytes can only contain ASCII literal characters.
```

bytes 类型提供的操作和 str 一样，支持分片、索引、基本数值运算等操作。但是 str 与 bytes 类型的数据不能执行 + 操作，尽管在py2中是可行的。

```python
>>> b"a"+b"c"
b'ac'
>>> b"a"*2
b'aa'
>>> b"abcdef\xd6"[1:]
b'bcdef\xd6'
>>> b"abcdef\xd6"[-1]
214

>>> b"a" + "b"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't concat bytes to str
```

### encode 与 decode

str 与 bytes 之间的转换可以用 encode 和从decode 方法。

![](/images/categories/python/017/python3-str2.jpg)

encode 负责字符到字节的编码转换。默认使用 UTF-8 编码准换。

```python
>>> s = "Python之禅"
>>> s.encode()
b'Python\xe4\xb9\x8b\xe7\xa6\x85'
>>> s.encode("gbk")
b'Python\xd6\xae\xec\xf8'
```

decode 负责字节到字符的解码转换，通用使用 UTF-8 编码格式进行转换。

```python
>>> b'Python\xe4\xb9\x8b\xe7\xa6\x85'.decode()
'Python之禅'
>>> b'Python\xd6\xae\xec\xf8'.decode("gbk")
'Python之禅'
```



