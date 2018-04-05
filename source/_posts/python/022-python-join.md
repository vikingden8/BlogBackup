---
title: Python join函数
date: 2018-04-04 23:45:08
tags:
categories: "Python学习笔记"
---

join()连接字符串数组。将字符串、元组、列表中的元素以指定的字符(分隔符)连接生成一个新的字符串。

语法：  'sep'.join(seq)

参数说明

* sep：分隔符。可以为空

* seq：要连接的元素序列、字符串、元组、字典

上面的语法即：以sep作为分隔符，将seq所有的元素合并成一个新的字符串

返回值：返回一个以分隔符sep连接各个元素后生成的字符串

<!--more-->

### 示例

1. 对序列进行操作（分别使用' '与':'作为分隔符）

```python
>>> seq = ['hello','python','c','java']
>>> print(' '.join(seq))
hello python c java
>>> print(':'.join(seq))
hello:python:c:java
```

2. 对字符串进行操作

```python
>>> str = 'hello python c java'
>>> 
>>> print(' '.join(str))
h e l l o   p y t h o n   c   j a v a
>>> 
>>> print(':'.join(str))
h:e:l:l:o: :p:y:t:h:o:n: :c: :j:a:v:a
>>> 
```

3. 对元组进行操作

```python
>>> tuple = ('hello', 'python', 'java')
>>> 
>>> print(' '.join(tuple))
hello python java
>>> 
>>> print(':'.join(tuple))
hello:python:java
>>> 
```

4. 对字典进行操作

```python
>>> dict = {'hello':1, 'python':2, 'java':3}
>>> 
>>> print(' '.join(dict))
hello java python
>>> 
>>> print(':'.join(dict))
hello:java:python
>>>
```
可以看出，字典的叠加只是针对于键，而且顺序是变化的。



