---
title: Python中的尾递归
date: 2018-03-31 19:25:08
tags:
categories: "Python学习笔记"
---

转载自：[什么是尾递归？](https://www.zhihu.com/question/20761771)

### 尾递归是什么？

尾递归和一般的递归不同在对内存的占用，普通递归创建stack累积而后计算收缩，尾递归只会占用恒量的内存（和迭代一样）。SICP中描述了一个内存占用曲线，用以上答案中的Python代码为例（普通递归）：

```Python
def recsum(x):
  if x == 1:
    return x
  else:
    return x + recsum(x - 1)
```

<!--more-->

当调用recsum(5)，Python调试器中发生如下状况：

```
recsum(5)
5 + recsum(4)
5 + (4 + recsum(3))
5 + (4 + (3 + recsum(2)))
5 + (4 + (3 + (2 + recsum(1))))
5 + (4 + (3 + (2 + 1)))
5 + (4 + (3 + 3))
5 + (4 + 6)
5 + 10
15
```

这个曲线就代表内存占用大小的峰值，从左到右，达到顶峰，再从右到左收缩。而我们通常不希望这样的事情发生，所以使用迭代，只占据常量stack space(更新这个栈！而非扩展他)。

```Python
for i in range(6):
  sum += i
```

Python中可以写（尾递归）：

```Python
def tailrecsum(x, running_total=0):
  if x == 0:
    return running_total
  else:
    return tailrecsum(x - 1, running_total + x)
```

理论上类似上面：

```Python
tailrecsum(5, 0)
tailrecsum(4, 5)
tailrecsum(3, 9)
tailrecsum(2, 12)
tailrecsum(1, 14)
tailrecsum(0, 15)
15
```

观察到，tailrecsum(x, y)中形式变量y的实际变量值是不断更新的，对比普通递归就很清楚，后者每个recsum()调用中y值不变，仅在层级上加深。所以，尾递归是把变化的参数传递给递归函数的变量了。怎么写尾递归？形式上只要最后一个return语句是单纯函数就可以。如：

```python
return tailrec(x+1);
```

而

```python
return tailrec(x+1) + x;
```

则不可以。因为无法更新tailrec()函数内的实际变量，只是新建一个栈。

更多参考：

* [Python开启尾递归优化!](http://python.jobbole.com/86937/)

* [尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)
