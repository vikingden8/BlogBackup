---
title: Python range()方法使用介绍
date: 2018-10-20 08:13:08
tags:
categories: "Python学习笔记"
---

翻译自：[Python's range() Function (Guide)](https://realpython.com/python-range/)

Python中的内建方法 range 在需要执行特定的多次行为时非常方便。

### range()的历史

虽然，在Python 3 和Python 2中使用了同一个 range() 名称，但是它们是完全不一样的。事实上，Python 3中的 range() 方法是Python 2中的 xrange() 的重命名而已。

起先，range() 和 xrange() 都是生成数字供 for 循环迭代使用，但是前者是即时地生成所有的数字集合，而后者则是以懒惰的方式生成数字，也就是说在需要的时候只返回生成一个数字。

当即时地生成大的数字集合时会占用大的内存，所以，在Python 3的版本中使用 xrange() 来替代 range() 一点也不惊奇。你可以通过阅读这个[PEP 3100](https://www.python.org/dev/peps/pep-3100)来了解更多的 xrang() 和 range() 背景。

<!--more-->

文中下述提到的 range() 都是在Python 3的环境。

### 使用循环

在我们深入了解 range() 工作原理之前，需要了解下循环是如何工作的。循环是计算机科学的重要概念。如果你想要成为好的程序员，那么掌握循环将是你首先要做的事情。

下面是Python中的for循环示例：

```Python
captains = ['Janeway', 'Picard', 'Sisko']

for captain in captains:
    print(captain)
```

输出：

```Python
Janeway
Picard
Sisko
```

如你所见，一个循环能让你执行特定的代码多次。在这个示例中，我们循环一首都的集合，并依次打印出它们的名字。

有时候，你可能只需要执行特定的代码段给定的特定次数，尝试下面的代码：

```Python
numbers_divisible_by_three = [3, 6, 9, 12, 15]

for num in numbers_divisible_by_three:
    quotient = num / 3
    print("{} divided by 3 is {}.".format(num, int(quotient)))
```

输出：

```Python
3 divided by 3 is 1.
6 divided by 3 is 2.
9 divided by 3 is 3.
12 divided by 3 is 4.
15 divided by 3 is 5.
```

得到了我们想要地输出，所以循环能完成需求，但是，还有一种方法，也就是使用 range() 也能得到同样地结果。

现在你已经熟悉了循环的使用，下面我们来看下使用 range() 方法来简便我们的日常。

#### 开始使用 range()

究竟Python中的 range 方法是如何工作的呢？简单来说，range() 允许你在给定的范围内生成一系列的数字。具体取决于你给这个方法传入的参数个数，你可以决定这个数字集合的开始和结束，以及数字和数字之间的间隔大小。

看下下面的示例：

```Python
for i in range(3, 16, 3):
    quotient = i / 3
    print("{} divided by 3 is {}.".format(i, int(quotient)))
```

在这个循环中，你能简单的创建一系列能被3整除的数字，所以你没必要自己提供。

有以下的三种方式来调用 range():

* 使用一个参数的 range(stop)

* 使用两个参数的 range(start, stop)

* 使用三个参数的 range(start, stop, step)

#### range(stop)

当你调用 range(stop) 的时候只传了一个参数，那么你将会得到一系列以0开始，一直到 stop ，但不包括 stop 的数字集合。

在实际中如下：

```Python
for i in range(3):
    print(i)
```

输出：

```Python
0
1
2
```

得到的结果：返回了0到2的数字集合，但是不包括3。

#### range(start, stop)

当你在调用 range(start, stop) 时给定了两个参数，那么你返回的数字集合决定了从 start 开始到 stop 结束，也就是某种情况下你并不是所有的数字集合都是从0开始。下面我们来实践下如何生成从1开始的数字集合。

尝试在调用 range() 时使用两个参数：

```Python
for i in range(1, 8):
    print(i)
```

输出：

```Python
1
2
3
4
5
6
7
```

从上可看出，返回了从1开始到7的数字，但是不包括给定的结束点8。

#### range(start, stop, step)

当你调用 range() 时传了三个参数，那么决定了返回的数字集合的开始和结束，同时也指定了两个相近数字间的间隔大小。如果你没有指定 step ，那么默认的 range() 方法会设置 step 为1。

step 可正可负，但是不可以为0：

```Python
>>> range(1, 4, 0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: range() arg 3 must not be zero
```

如果你尝试给 step 指定为0，那么你将得到上述的错误。

### 递增的 range()

如果你想要递增的数字集合，那么 range() 中的 step 必须为正整数。

```Python
for i in range(3, 100, 25):
    print(i)
```

输出：

```Python
3
28
53
78
```

#### 递减的 range()

如果 range() 方法中 step 指定为负整数，那么返回的数字集合将是递减的。

```Python
for i in range(10, -6, -2):
    print(i)
```

输出：

```Python
10
8
6
4
2
0
-2
-4
```

在Python中创建递减的数字集合就是使用range(start, stop, step)。但是Python有一个内建函数 reversed。如果你的 range() 被 reversed() 包裹使用，则得到的整数是反序的。

```Python
for i in reversed(range(5)):
    print(i)
```

输出：

```Python
4
3
2
1
0
```

### 深入了解 range()

现在你了解了怎样使用 range()， 现在是时候再深入了解了。

range() 主要有两种使用目的：

* 执行循环中的代码块多次

* 创建比集合或元组更高效地的可迭代数字集合

在Python中，range() 是一种类型：

```Python
>>> type(range(3))
<class 'range'>
```

你可以通过索引的方式访问，就像访问集合一样：

```Python
>>> range(3)[1]
1
>>> range(3)[2]
2
```

甚至你可以使用切片，但是得到就是一个 REPL ：

```Python
>>> range(6)[2:5]
range(2, 5)
```

虽然返回看起来有点奇怪，但是 range() 的切片操作仅仅是返回另一个 range() 对象。

### Floats 和 range()

你或许已注意到我们上述的示例中都是使用的整数，包括参数以及返回的数字集合。这是因为 range() 方法的参数必须是整数。

在Python中，如果一个数字不是整数，那么就是浮点型。

下面尝试下在 range() 方法中使用 float，看会发生什么：

```Python
for i in range(3.3):
    print(i)
```

应当会得到如下的错误信息：

```Python
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'float' object cannot be interpreted as an integer
```

如果你确实需要使用 float 的集合，那么你可以使用 NumPy。

```Python
import numpy as np

np.arange(0.3, 1.6, 0.3)
```

返回：

```Python
array([0.3, 0.6, 0.9, 1.2, 1.5])
```
