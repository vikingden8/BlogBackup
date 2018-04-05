---
title: Python中set
date: 2018-04-05 10:24:08
tags:
categories: "Python学习笔记"
---

集合中包含一系列的元素，在Python中这些元素不需要是相同的类型，且这些元素在集合中是没有存储顺序的。

### 集合的赋值

集合的表示方法是花括号，这与字典是一样的，可以通过括号或构造函数来初始化一个集合，如果传入的参数有重复，会自动忽略：

```python
>>> {1,2,"hi",2.23}
{2.23, 2, 'hi', 1}
>>> set("hello")
{'l', 'h', 'e', 'o'}
```

>注：由于集合和字典都用{}表示，所以初始化空的集合只能通过set()操作，{}只是表示一个空的字典

<!--more-->

### 集合的增加

集合元素的增加支持两种类型，单个元素的增加用add方法，对序列的增加用update方法。add的作用类似列表中的append，而update类似extend方法。update方法可以支持同时传入多个参数：

```python
>>> a={1,2}
>>> a.update([3,4],[1,2,7])
>>> a
{1, 2, 3, 4, 7}
>>> a.update("hello")
>>> a
{1, 2, 3, 4, 7, 'h', 'e', 'l', 'o'}
>>> a.add("hello")
>>> a
{1, 2, 3, 4, 'hello', 7, 'h', 'e', 'l', 'o'}
```

>注：增加已有的元素不会对集合产生影响，也不会抛出异常

### 集合的删除

集合删除单个元素有两种方法，两者的区别是在元素不在原集合中时是否会抛出异常，set.discard(x)不会，set.remove(x)会抛出KeyError错误：

```python
>>> a={1,2,3,4}
>>> a.discard(1)
>>> a
{2, 3, 4}
>>> a.discard(1)
>>> a
{2, 3, 4}
>>> a.remove(1)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
KeyError: 1
```

集合也支持pop()方法，不过由于集合是无序的，pop返回的结果不能确定，且当集合为空时调用pop会抛出KeyError错误，可以调用clear方法来清空集合：

```python
>>> a={3,"a",2.1,1}
>>> a.pop()
1
>>> a.pop()
3
>>> a.clear()
>>> a
set()
>>> a.pop()
Traceback (most recent call last):
  File "<input>", line 1, in <module>
KeyError: 'pop from an empty set'
```

### 集合操作

* 并集：set.union(s),也可以用a|b计算

* 交集：set.intersection(s)，也可以用a&b计算

* 差集：set.difference(s)，也可以用a-b计算

需要注意的是Python提供了一个求对称差集的方法set.symmetric_difference(s)，相当于两个集合互求差集后再求并集，其实就是返回两个集合中只出现一次的元素，也可以用a^b计算。

```python
>>> a={1,2,3,4}
>>> b={3,4,5,6}
>>> a.symmetric_difference(b)
{1, 2, 5, 6}
```

set.update(s)操作相当于将两个集合求并集并赋值给原集合，其他几种集合操作也提供各自的update版本来改变原集合的值，形式如intersection_update(),也可以支持多参数形式。

### 包含关系

两个集合之间一般有三种关系，相交、包含、不相交。在Python中分别用下面的方法判断：

* set.isdisjoint(s)：判断两个集合是不是不相交

* set.issuperset(s)：判断集合是不是包含其他集合，等同于a>=b

* set.issubset(s)：判断集合是不是被其他集合包含，等同于a<=b

如果要真包含关系，就用符号操作>和<。

### 不变集合

Python提供了不能改变元素的集合的实现版本，即不能增加或删除元素，类型名叫frozenset，使用方法如下：

```python
>>> a = frozenset("hello")
>>> a
frozenset({'l', 'h', 'e', 'o'})
```

需要注意的是frozenset仍然可以进行集合操作，只是不能修改。

```python
>>> a = frozenset('hello')
>>> a
frozenset({'o', 'l', 'h', 'e'})
>>> 
>>> 
>>> type(a)
<class 'frozenset'>
>>> 
>>> 
>>> a.add('a')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'add'
>>> 
>>> 
>>> 
>>> a.update('adc')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'update'
>>> 
>>> 
>>> a.remove('h')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'remove'
>>>
```

可以看出，当调用add(), update(), remove()方法时都会出现AttributeError错误。


如果要一个有frozenset中的所有元素的普通集合，只需把它当作参数传入集合的构造函数中即可：

```python
>>> a = frozenset("hello")
>>> a = set(a)
>>> a.add(12)
>>> a
{'l', 12, 'h', 'e', 'o'}
```


