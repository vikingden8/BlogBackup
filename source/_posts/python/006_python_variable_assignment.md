---
title: Python变量赋值
date: 2017-03-19 22:15:20
tags:
categories: "Python学习笔记"
---

### 赋值操作符

Python语言中，等号（=）是主要的赋值操作符。

![](/images/categories/python/006/1.png)

>注意：在Python中，赋值并不是直接将一个值赋给一个变量。在赋值时，不管这个对象是新创建的，还是一个已经存在的，都是将该对象的引用赋值给变量

### 增量赋值

Python2.0开始，等号可以和一个算术操作符组合在一起，将计算结果重新赋值给左边的变量，即为增量赋值，类似如下的语句：

```Python
x = x + 1
```

可以写成：

```Python
x += 1
```

![](/images/categories/python/006/2.png)

<!--more-->

>注意：Python不支持类似x++或--x这样的前置/后置自增/自减运算。

### 多重赋值

![](/images/categories/python/006/3.png)

如上，一个值为1的整型对象被创建，该对象的同一个引用被赋值给x，y和z。也就是一个对象被赋值给了多个变量。

### “多元”赋值

![](/images/categories/python/006/4.png)

上面的例子中，两个整型对象及一个字符串对象，被分别赋值给x，y和z。通常元组需要圆括号括起来，尽管它们是可选的，建议总是加上圆括号以使你的代码有更高的可读性。

![](/images/categories/python/006/5.png)

在类似Java的语言中，如果你要交换两个值，你会想到使用一个临时变量如temp来临时保存其中一个值，如下：

```Java
temp = x ;
x = y ;
y = temp ;
```

但是，在Python中通过多元赋值方式可以实现无需中间变量交换两个变量的值：

![](/images/categories/python/006/6.png)
