---
title: Python's Instance, Class and Static Methods
date: 2018-07-07 11:01:08
tags:
categories: "Python学习笔记"
---

这篇文章主要介绍Python的实例方法（Instance Methods），类方法（Class Methods）以及静态方法（Static Methods）。掌握和理解三者的用法以及区别，对编写面向对象的程序很有帮助。

### 三种方法的简介

首先在 MyClass 中分别创建这三个方法：

```python
class MyClass:
    def method(self):
        return 'instance method called', self

    @classmethod
    def classmethod(cls):
        return 'class method called', cls

    @staticmethod
    def staticmethod():
        return 'static method called'
```

<!--more-->

#### 实例方法（Instance Methods）

MyClass中的第一方法 method() 是一个普通的实例方法。这是最基本的方法，很多的时候都会使用它。这个方法的第一个参数是 self，也就是当调用这个方法时的那个实例对象。

因为有参数 self ，实例方法能直接访问这个实例的所有属性和方法，所以能修改这个对象实例的状态。另外，通过 self.__class__ 属性能访问类自己，所以实例方法也可以修改类的状态，

#### 类方法（Class Methods）

来看以下 MyClass.classmethod() ，这里使用了装饰器 @classmethod 来标记此方法是一个类方法。

与实例方法不同的是，方法的第一个参数使用的是 cls ，而不是 self。也意味着，在调用这个方法时，是使用的类而不是实例对象。

因为类方法只能访问类 cls 参数，它不能修改实例的状态。

#### 静态方法（Static Methods）

第三个方法 MyClass.staticmethod() 使用装饰器 staticmethod 来标记此方法是一个静态方法。

这个方法的参数既不带 self 也不带 cls，但是，可以带其他自定义的参数。所以静态方法既不能修改对象实例和类状态。

### 三者区别

下面来看下这三个方法分别怎么调用以及调用后的行为。

调用实例方法：

```python
# instance method

myclass = MyClass()
print(myclass.method())
```

输出：

```sh
viking@ubuntu:~/python/methods$ python methods.py 
('instance method called', <__main__.MyClass object at 0x7f5fd28d1198>)
viking@ubuntu:~/python/methods$ 
```

通过上面的输出可以看出，通过实例对象访问了 method() 方法，Python在调用 method() 方法时，会把 self 替换为 调用此方法的实例对象。obj.method() 其实只是Python提供的语法糖，也就是方便使用，其实等效于下面的方法：

```python
# no sugar sytax

print(MyClass.method(myclass))

```

输出：

```sh
viking@ubuntu:~/python/methods$ python methods.py 
('instance method called', <__main__.MyClass object at 0x7f50a54d1198>)
('instance method called', <__main__.MyClass object at 0x7f50a54d1198>)
viking@ubuntu:~/python/methods$
```

可以看出，输出了同样地结果。

同时，对象实例还可以通过 self.__class__ 属性访问类本身，这让对象实例突破了约束，其既可以修改对象的状态，也可以修改类本身的状态。

试着使用类方法：

```python
# instance call class method
print(myclass.classmethod())
```

输出：

```sh
viking@ubuntu:~/python/methods$ python methods.py 
('instance method called', <__main__.MyClass object at 0x7fbb4943b198>)
('instance method called', <__main__.MyClass object at 0x7fbb4943b320>)
('class method called', <class '__main__.MyClass'>)
viking@ubuntu:~/python/methods$
```

可以看出，调用 classmethod() 并没有通过对象实例，而是通过的类对象。

这里需要注意的是 self 和 cls 可以任意替换，如 the_object 或者 the_class，会是同样地的效果，但是都必须是第一个参数。

现在来调用下静态方法：

```python
# install call static method
print(myclass.staticmethod())
```

输出：

```sh
viking@ubuntu:~/python/methods$ python methods.py 
('instance method called', <__main__.MyClass object at 0x7fe608f892e8>)
('instance method called', <__main__.MyClass object at 0x7fe608f89278>)
('class method called', <class '__main__.MyClass'>)
static method called
viking@ubuntu:~/python/methods$
```

可以看出，同样通过对象实例可以调用类中定义的静态方法。

下面来看看，使用类来调用上述的方法，会有什么输出。

```python
# class call class method
print(MyClass.classmethod())

# class call static method
print(MyClass.staticmethod())

# class call instance method
print(MyClass.method())
```

输出：

```sh
viking@ubuntu:~/python/methods$ 
viking@ubuntu:~/python/methods$ python methods.py 
('class method called', <class '__main__.MyClass'>)
static method called
Traceback (most recent call last):
  File "methods.py", line 42, in <module>
    print(MyClass.method())
TypeError: method() missing 1 required positional argument: 'self'
viking@ubuntu:~/python/methods$
```

可以看出，可以正常的调用 classmethod() 和 staticmethod()，但是在调用 method() 的时候却出现了 TypeError 错误。

这正是所期望的，因为如果要使用实例方法的话，必须要创建一个对象实例。

那到底什么时候用 classmethod ？什么时候用 staticmethod 呢？下篇继续。
