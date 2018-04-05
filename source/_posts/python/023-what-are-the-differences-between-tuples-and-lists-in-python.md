---
title: What are the differences between tuples and lists in Python?
date: 2018-04-05 08:45:08
tags:
categories: "Python学习笔记"
---

Origin from : [What are the differences between tuples and lists in Python?](https://www.quora.com/What-are-the-differences-between-tuples-and-lists-in-Python)

First let me clear what is a list and tuple in Python and then I will differentiate it on the basis of mutability, functions methods, tuple in a list, list in a tuple, nested tuples, nested lists

首先，我会阐述在Python中什么是list和tuple，然后我会列出两者的不同，比如可变性，函数方法，list中的tuple，tuple中的list，嵌套的tuple，嵌套的list。


and When Can You Use Which?

还有，就是在什么情况下该选择哪种。

* **Tuple**: A tuple is a collection of values and it is declared using parentheses. However, we can also use a tuple packing to do the same, and unpacking to assign its values to a sequence.

* **元组**: 一个元组就是一系列值的组合，并且使用圆括号声明。然而，我们也可以使用tuple关键字来组装一个元组，也可以把元组拆分成序列。

* **Lists**: Unlike in C++, we don’t have arrays to work with in Python. Here, we have a list instead.

* **列表**: 与C++不同，在Python中没有数组。这里，我们使用list来替代。

<!--more-->

### Mutability 可变性

* A List is Mutable list是可变的

Let’s first see lists. Let’s take a new list for exemplar purposes.

首先我们先来看下list。创建一个list来用于测试。

```python
>>> list1=[0,1,2,3,4,5,6,7]
```

Now first, we’ll try reassigning an element of a list. Let’s reassign the second element to hold the value 3.

我们尝试给list中的一个元素重新赋值。我们把第二个元素重新赋值为3

```python
>>> list1[1]=3
>>> list1
[0, 3, 2, 3, 4, 5, 6, 7]
```

Again, let’s see how we can reassign the entire list.

同样地，我们来看下给list整个重新赋值。

```python
>>> list1=[7,6,5,4,3,2,1,0]
>>> list1
[7, 6, 5, 4, 3, 2, 1, 0]
```

It worked, great.

Now, we will delete just one element from the list.

现在，我们来删除list中的一个元素。

```python
>>> del list1[1]
>>> list1
[7, 5, 4, 3, 2, 1, 0]
```

This was easy, but could we delete a slice of the list? Let’s try it.

这很容易，我们是否能以切片的方式删除list中的元素？试试看。

```python
>>> del list1[3:]
>>> list1
[7, 5, 4]
```

We can access a slice the same way. Can we reassign a slice?

从上可以看出，我们可以通过切片的方式处理list。我们能否以切片的方式赋值呢？

```python
>>> nums=[1,2,3,4,5]
>>> nums[1:3]=[6,7,8]
>>> nums
[1, 6, 7, 8, 4, 5]
```

Indeed, we can. Finally, let’s try deleting the entire list.

的确，可以。最后，我们试着删除整个list。

```python
>>> del list1
>>> list1
Traceback (most recent call last):
File "<pyshell#67>", line 1, in <module>
list1
NameError: name ‘list1’ is not defined
```

The list doesn’t exist anymore.

可以看出，list不再存在。

### A Tuple is Immutable 元组是不可变的

Now, let’s try doing the same things to a tuple. We know that a tuple is immutable, so some of these operations shouldn’t work. We’ll take a new tuple for this purpose.

现在，我们尝试在tuple上进行相同的操作。我们知道tuple是不可变的，所以有些操作可能不能工作。首先，我们创建一个tuple来测试。

```python
>>> mytuple=0,1,2,3,4,5,6,7
First, let’s try reassigning the second element.

>>> mytuple[1]=3
Traceback (most recent call last):
File "<pyshell#70>", line 1, in <module>
mytuple[1]=3
TypeError: ‘tuple’ object does not support item assignment
```

As you can see, a tuple doesn’t support item assignment.

可以看到，tuple不支持元素的重新赋值。

However, we can reassign an entire tuple.

但是，我们可以给整个tuple重新赋值。
 
```python
>>> mytuple=2,3,4,5,6
>>> mytuple
(2, 3, 4, 5, 6)
```

Next, let’s try slicing a tuple to access or delete it.

下面，我们来尝试切片访问和删除。

```python
>>> mytuple[3:]
(5, 6)

>>> del mytuple[3:]
Traceback (most recent call last):
File "<pyshell#74>", line 1, in <module>
del mytuple[3:]
TypeError: ‘tuple’ object does not support item deletion
```

As is visible, we can slice it to access it, but we can’t delete a slice. This is because it is immutable.

我们可以以切片的方式访问tuple，但是我们不能以这种方式删除。这是因为tuple是不可变的。

Can we delete a single element?

我们可以删除单个元素吗？

```python
>>> del mytuple[3]
Traceback (most recent call last):
File "<pyshell#75>", line 1, in <module>
del mytuple[3]
TypeError: ‘tuple’ object doesn’t support item deletion
```

Apparently, the answer is no.

很显然，答案是不可以。

Finally, let’s try deleting the entire tuple.

最后，我们尝试删除整个tuple。

```python
>>> del mytuple
>>> mytuple
Traceback (most recent call last):
File "<pyshell#77>", line 1, in <module>
mytuple
NameError: name ‘mytuple’ is not defined
```

So, here, we conclude that you can slice a tuple, reassign it whole, or delete it whole. But you cannot delete or reassign just a few elements or a slice.

这里，我们可以总结出，tuple可以以切片的方式访问，重新赋值或删除整个tuple。但是，不能删除或重新赋值tuple中一个或多个元素。

Let us proceed with more differences between python tuples vs lists.

我们再来多看下python中lists和tuples的不同之处。

### Functions 函数

Some python functions apply on both, these are- len(), max(), min(), sum(), any(), all(), sorted(). We’ll take just one example here for both containers.

python中的某些函数在两者都能使用，len(), max(), min(), sum(), any(), all(), sorted().我们这里只拿一个来举例。

```python
>>> max((1,3,-1))
3

>>> max([1,3,-1])
3
```

### Methods 方法

Lists and tuples share the index() and count() methods. But other than those, there are a few methods that apply to lists. These are- append(), insert(), remove(), pop(), clear(), sort(), and reverse(). Let’s take an example of one of these.

lists和tuples共同的方法有index()和count()。lists有几个特有的方法，append(), insert(), remove(), pop(), clear(), sort(), and reverse()。我们拿一个来示例：

```python
>>> [1,3,2].index(3)
1

>>> (1,3,2).index(3)
1
```

### Tuples in a List 

We can store tuples in a list when we want to.

我们可以在list中存储tuples

```python
>>> mylist=[(1,2,3),(4,5,6)]
>>> type(mylist)
<class 'list'>
>>> type(mylist[1])
<class ‘tuple’>
```

But when would we need to do this? Take an example.

但是，在什么时候我们需要这样做呢？看一个示例。

```python
>>> list(students)
[(1, ‘ABC’), (2, ‘DEF’), (3, ‘GHI’)]
```

### Lists in a Tuple

Likewise, we can also use a tuple to store lists. Let’s see how.

像前面一样，我们也可以在tuple中存储lists，示例：

```python
>>> mytuple=([1,2],[3,4],[5,6])
```

### Nested Tuples

A tuple may hold more tuples, and this can go on in more than two dimensions.

一个tuple中可以嵌套多个tuples，而且这个不止是二维：

```python
>>> mytuple=((1,2),(3,(4,5),(6,(7,(8,9)))))
```

To access the element with the value 8, we write the following code.

我们使用下面的代码来访问元素8

```python
>>> mytuple[1][2][1][1][0]
8
```

### Nested Lists

Similarly, a list may hold more lists, in as many dimensions as you want.

同样地，list可以嵌套多个lists：

```python
>>> mylist=[[1,2],[3,4]]
>>> myotherlist=[[1,2],[3,[4,5]]]
```

To access the element with the value 5, we write the following code.

可以使用下面的代码来访问元素5

```python
>>> myotherlist[1][1][1]
5
```

### When to Use Which? 怎么选择？

Use a tuple when you know what information goes in the container that it is. For example, when you want to store a person’s credentials for your website.

当你确切知道集合中要存储的内容时，使用tuple。例如，当你存储个人网站的信息时：

```python
>>> person=('ABC','admin','12345')
```

But when you want to store similar elements, like in an array in C++, you should use a list.

但是，你想要在C++中一样使用数组来存储差不多的信息时，你应该使用list。也就是说，list不能确保是否需要添加其他元素时

```python
>>> groceries=['bread','butter','cheese']
```



