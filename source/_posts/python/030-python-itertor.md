---
title: Python中的for工作原理
date: 2018-07-12 19:12:08
tags:
categories: "Python学习笔记"
---

这篇文章主要介绍Python中的for循环工作原理。同时，也会介绍可迭代对象，迭代器和迭代协议。

### 基于索引迭代：失败的例子

在Python中使用像C或者Java的基于索引的循环写法：

```python
colors = ["red", "green", "blue", "purple"]
i = 0
while i < len(colors):
    print(colors[i])
    i += 1
```

上面的方式仅对列表适用，对集合则会出现错误：

```python
>>> colors = {"red", "green", "blue", "purple"}
>>> i = 0
>>> while i < len(colors):
...     print(colors[i])
...     i += 1
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
TypeError: 'set' object does not support indexing
```

<!--more-->

基于索引的循环写法只适应于序列，也就是数据类型的索引值从0到其长度。像列表，元组和字符串是序列，而字典，集合以及其他的可迭代对象就不是序列。

### 什么是可迭代对象

在Python世界中，一个可迭代的对象是可以使用for循环来遍历的任何对象。

可迭代对象并不总是有索引的，而且也不总是有长度的，甚至也不总是无穷的。

看看下面的例子：

```python
from itertools import count
multiples_of_five = count(step=5)
```

当我们使用for循环来获取迭代值：

```python
for n in multiples_of_five:
    if n > 100:
        break
    print(n)
```

如果在上面的代码中移除 break 语句，则上述的代码将会一直运行。

### 可迭代对象和迭代器

上面定义了可迭代对象，但是在Python中可迭代对象是如何工作的呢？

所有的可迭代对象可以使用内建函数 iter 返回一个迭代器：

```python
>>> iter(['some', 'list'])
<list_iterator object at 0x7f227ad51128>
>>> iter({'some', 'set'})
<set_iterator object at 0x7f227ad32b40>
>>> iter('some string')
<str_iterator object at 0x7f227ad51240>
```

什么是迭代器呢？

迭代器只有一个目的：返回可迭代对象的下一项。

你可以任何可迭代对象中获取迭代器：

```python
>>> iterator = iter('hi')
```

使用 next 函数返回可迭代对象中的下一项：

```python
>>> next(iterator)
'h'
>>> next(iterator)
'i'
>>> next(iterator)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

从上可以看出，我们可以使用内建函数 next 返回可迭代对象的下一个元素，当迭代到最后时，会出现 StopIteration 异常，证明已迭代完成。

### 迭代器也是可迭代对象

可以使用 iter 内建函数应用于迭代器，会返回一个迭代器。这也说明，迭代器也是可迭代的对象：

```python
>>> iterator = iter('hi')
>>> iterator2 = iter(iterator)
>>> iterator is iterator2
True
```

### 迭代协议

**可迭代对象**

可以使用内建函数 iter 返回一个迭代器。

**迭代器**

1. 使用内建函数 next 返回可迭代对象的下一个元素或者抛出 StopIteration 异常；
2. 使用内建函数 iter 返回自身

### 迭代器的遍历

有了可迭代对象和迭代器的概念了解，我们可以不使用Python中的for循环而自己实现迭代器的遍历。

```python
def print_each(iterable):
    iterator = iter(iterable)
    while True:
        try:
            item = next(iterator)
        except StopIteration:
            break  # Iterator exhausted: stop the loop
        else:
            print(item)
```

现在我们可以使用 print_each 函数来对任意的可迭代对象进行遍历：

```python
>>> print_each({1, 2, 3})
1
2
3
```

上面的代码其实完全等效于：

```python
def print_each(iterable):
    for item in iterable:
        print(item)
```

for 循环其实在底层就是类似上面的工作原理：使用内建函数 iter 函数可迭代对象的迭代器，使用内建函数 next 遍历出每一项，直至出现 StopIteration 异常。

### 参考资料

[The Iterator Protocol: How “For Loops” Work in Python](http://treyhunner.com/2016/12/python-iterator-protocol-how-for-loops-work/)