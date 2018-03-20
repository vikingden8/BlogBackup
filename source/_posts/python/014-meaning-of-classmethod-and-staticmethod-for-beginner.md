---
title: Meaning of classmethod and staticmethod for beginner
date: 2018-03-20 20:08:08
tags:
categories: "Python学习笔记"
---

From:[Meaning of @classmethod and @staticmethod for beginner?](https://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner)

Though **classmethod** and **staticmethod** are quite similar, there's a slight difference in usage for both entities: **classmethod** must have a reference to a class object as the first parameter, whereas **staticmethod** can have no parameters at all.

虽然 **classmethod** 和 **staticmethod** 十分相似，但是它们在使用上有很大的区别：**classmethod** 必须把类的引用当作第一个参数，而 **staticmethod** 可不带任何的参数。

<!--more-->

### Example 示例

```Python
class Date(object):

    def __init__(self, day=0, month=0, year=0):
        self.day = day
        self.month = month
        self.year = year

    @classmethod
    def from_string(cls, date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        date1 = cls(day, month, year)
        return date1

    @staticmethod
    def is_date_valid(date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        return day <= 31 and month <= 12 and year <= 3999

date2 = Date.from_string('11-09-2012')
is_date = Date.is_date_valid('11-09-2012')
```

### Explanation 解释

Let's assume an example of a class, dealing with date information (this is what will be our boilerplate to cook on):

假如有这样一个类，专门用来处理时间信息：

```Python
class Date(object):

    def __init__(self, day=0, month=0, year=0):
        self.day = day
        self.month = month
        self.year = year
```

This class obviously could be used to store information about certain dates (without timezone information; let's assume all dates are presented in UTC).

这个类很明显的用途就是可用来存取具体的时间信息（没有时区信息；假定所有的时间都使用UTC）。

Here we have **__init__**, a typical initializer of Python class instances, which receives arguments as a typical instancemethod, having the first non-optional argument (self) that holds reference to a newly created instance.

这个有一个 **__init__** 方法，典型的Python类对象初始化方法，接收参数，第一个必须的参数（self）用于指向已经创建好的对象。

#### Class Method

We have some tasks that can be nicely done using **classmethod** s.

利用 **classmethod** 可以做一些很棒的东西。

Let's assume that we want to create a lot of Date class instances having date information coming from outer source encoded as a string of next format ('dd-mm-yyyy'). We have to do that in different places of our source code in project.

比如我们可以支持从特定格式的日期字符串来创建对象，它的格式是 ('dd-mm-yyyy')。很明显，我们只能在其他地方而不是 init 方法里实现这个功能。

So what we must do here is:

所以我们必须这样做：

* Parse a string to receive day, month and year as three integer variables or a 3-item tuple consisting of that variable.

* 解析字符串，得到整数 day， month， year。

* Instantiate Date by passing those values to initialization call.

* 使用得到的信息初始化对象

This will look like:

代码如下：

```Python
day, month, year = map(int, string_date.split('-'))
date1 = Date(day, month, year)
```

For this purpose, C++ has such feature as overloading, but Python lacks that feature- so here's when **classmethod** applies. Lets create another "constructor".

C++ 可以方便的使用重载来解决这个问题，但是Python不具备类似的特性。 所以接下来我们要使用 **classmethod** 来帮我们实现。

```Python
@classmethod
def from_string(cls, date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    date1 = cls(day, month, year)
    return date1

date2 = Date.from_string('11-09-2012')
```

Let's look more carefully at the above implementation, and review what advantages we have here:

让我们在仔细的分析下上面的实现，看看它的好处。

* We've implemented date string parsing in one place and it's reusable now.

* 我们在一个方法中实现了功能，因此它是可重用的。

* Encapsulation works fine here (if you think that you could implement string parsing as a single function elsewhere, this solution fits OOP paradigm far better).

* 封装处理的不错（如果你发现还可以在代码的任意地方添加一个不属于 Date 的函数来实现类似的功能，那很显然上面的办法更符合 OOP 规范）。

* _cls_ is an object that holds class itself, not an instance of the class. It's pretty cool because if we inherit our Date class, all children will have from_string defined also.

* _cls_ 是一个保存了 class 的对象（所有的一切都是对象）。 更妙的是， Date 类的衍生类都会具有 from_string 这个有用的方法。

#### Static method

What about **staticmethod** ? It's pretty similar to **classmethod** but doesn't take any obligatory parameters (like a class method or instance method does).

那什么是 **staticmethod** ? 它跟 **classmethod** 非常相似但是不带任何的必选的参数（类方法和实例方法有必选参数）。

Let's look at the next use case.

让我们看下另一用户场景。

We have a date string that we want to validate somehow. This task is also logically bound to Date class we've used so far, but still doesn't require instantiation of it.

有时候我们需要验证一个时间字符串。这个任务同样地和之前的Date类绑定，但是仍然不需要去实例化它。

Here is where **staticmethod** can be useful. Let's look at the next piece of code:

这就是 **staticmethod** 能派上用途了。看下代码的实现：

```Python
@staticmethod
def is_date_valid(date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    return day <= 31 and month <= 12 and year <= 3999

# usage:
is_date = Date.is_date_valid('11-09-2012')
```

So, as we can see from usage of **staticmethod**, we don't have any access to what the class is- it's basically just a function, called syntactically like a method, but without access to the object and it's internals (fields and another methods), while **classmethod** does.

所以，从静态方法 **staticmethod** 的使用中可以看出，我们不会访问到 class 本身 - 它基本上只是一个函数，在语法上就像一个方法一样，但是没有访问对象和它的内部（字段和其他方法），相反 **classmethod** 会访问 cls， 。
