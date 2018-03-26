---
title: Python中bytes和hex之间的转换
date: 2018-03-26 19:38:08
tags:
categories: "Python学习笔记"
---

### 在Python2.7.x上，hex字符串和bytes之间的转换是这样的：

```python
viking@ubuntu:~$ python2
Python 2.7.14 (default, Sep 23 2017, 22:06:14) 
[GCC 7.2.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> t = 'aabbccddeeff'
>>> t_bytes = t.decode('hex')
>>> print(t_bytes)
������
>>> t_a = t_bytes.encode('hex')
>>> print(t_a)
aabbccddeeff
>>> 
>>> 

```

这里不知为啥t_bytes显示的是乱码，应该是python 2的编码问题。

<!--more-->

### 在python 3中，因为string和bytes的实现发生了重大的变化，这个转换也不能再用encode/decode完成了

#### 在python3.5之前，这个转换的其中一种方式是这样的：

```python
>>> a = 'aabbccddeeff'
>>> a_bytes = bytes.fromhex(a)
>>> print(a_bytes)
b'\xaa\xbb\xcc\xdd\xee\xff'
>>> aa = ''.join(['%02x' % b for b in a_bytes])
>>> print(aa)
aabbccddeeff
>>>
```

**注：由于本地电脑的版本为最新的版本，上面没实践**

#### python 3.5之后，就可以像下面这么干了：

```python
viking@ubuntu:~$ python3
Python 3.6.3 (default, Oct  3 2017, 21:45:48) 
[GCC 7.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> t = 'aabbccddeeff'
>>> t_bytes = bytes.fromhex(t)
>>> print(t_bytes)
b'\xaa\xbb\xcc\xdd\xee\xff'
>>> t_a = t_bytes.hex()
>>> print(t_a)
aabbccddeeff
>>> 
>>>
```


