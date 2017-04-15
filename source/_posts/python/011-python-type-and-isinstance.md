---
title: python判断变量类型时，为什么不推荐使用type()方法
date: 2017-04-15 22:37:08
tags:
categories: "Python学习笔记"
---

我们可以使用type()内建方法来判断当前的变量类型，如：

![](/images/categories/python/011/1.png)

当然，我们还有另外一种方法来判断，使用isinstance():

![](/images/categories/python/011/2.png)

isinstance() 和 type()的区别在于：

![](/images/categories/python/011/3.png)

也就是说type()不能用于判断子类和继承类
