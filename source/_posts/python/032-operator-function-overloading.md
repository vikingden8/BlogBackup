---
title: Python中的操作符重载
date: 2018-09-01 10:40:08
tags:
categories: "Python学习笔记"
---

本文翻译自[有删改]：[Operator and Function Overloading in Custom Python Classes](https://realpython.com/operator-function-overloading/)

如果你已经在Python使用过 str 的 + 和 * 操作符，你肯定注意到它相对于 int 或 float 对象的 + 和 * 操作有不同之处。

```Python
>>> # Adds the two numbers
>>> 1 + 2
3

>>> # Concatenates the two strings
>>> 'Real' + 'Python'
'RealPython'


>>> # Gives the product
>>> 3 * 2
6

>>> # Repeats the string
>>> 'Python' * 3
'PythonPythonPython'
```

<!--more-->

你可能会好奇，同样的内建操作符或函数为啥会对不同的对象表现出不同的行为。其实，这在Python中被称作为操作符重载或函数重载。这篇文章将帮助你理解这种机制，你可以在自己的Python类中实现同样的重载功能，让你写的对象更具Pythonic。

### Python对象模型

假想你现在有一个类代表网上的订单，里面有购物清单（list），以及一个消费者对象（可以是一 str 或另一个代表消费者的对象类）。

在这种情况下，很自然地想要获取当前购物清单的长度。有一些Python新学者可能决定实现一个叫  get_cart_len() 的方法。但是，你可以查到，其实在Python中提供了内建函数 len() 来返回购物清单的长度。

另外一种情况，我们可能想要往购物清单中添加新的。同样，有些Python新学者可能会去实现一个方法叫 append_to_cart() 来添加新的清单。但是，你可以发现，其实内建的 + 操作符同样可以达到添加新的清单到购物清单中。

Python使用特别的方法都为你实现了上述的功能。那些特别的方法都是以双下划线开始，然后就是一个名称，可以以双下划线结束，比如 \_\_name\_\_.

本质上，每一个内建函数或操作符都对应有一个特别的方法。例如， __len__() 对应内建函数 len() ，以及 __add__() 对应操作符 + .

默认情况下，大部分的内建函数或者操作符对你的自定义类并不起作用。你必须在你的类中实现对应的特殊方法来使你的对应兼容内建函数或操作符。

当你这样实现之后，你类中的函数或操作符行为将会根据你定义的那些方法随着改变。

### 操作符 len() 和 [] 的内部实现原理

Python中的每个类都有定义内建函数或方法对应的行为。比如，当你在某个类的实例上使用内建函数或者操作符时，其实也就是相当于在调用它定义的特殊方法。

如果有一个内建函数 func() ，其对应的特殊方法就是 __func__()，Python会把 func() 理解为 obj.__func__() 。对于操作符，如果你有一个操作符 opr， 那么其对应的特殊方法就是 __opr__()，Python会把 obj1 opr obj2 解释为 obj1.__opr__(obj2) 。

于是，当你在对象上使用 len() 时，Python会处理成 obj.__len__() 。当你在一个可迭代对象上使用 [] 操作符来获取特定位置上的值时，Python其实会使用 itr.__getitem__(index) 。

当你在自己的实现类中定义了那些特殊的方法后，你就重写了那些函数或操作符的行为。

```Python
>>> a = 'Real Python'
>>> b = ['Real', 'Python']
>>> len(a)
11
>>> a.__len__()
11
>>> b[0]
'Real'
>>> b.__getitem__(0)
'Real'
```

从上面的例子可以看出，不管你使用函数或者对应的特殊方法，得到了同样地的输出结果。其实， 但你使用内建的 dir() 函数查看 str 对象的所有属性和方法时，你将会看到那些特殊方法包含在其中。

```Python
>>> dir(a)
['__add__',
 '__class__',
 '__contains__',
 '__delattr__',
 '__dir__',
 ...,
 '__iter__',
 '__le__',
 '__len__',
 '__lt__',
 ...,
 'swapcase',
 'title',
 'translate',
 'upper',
 'zfill']
 ```

 如果你调用的内建函数或操作符在类中没有对应的特殊方法实现，而你调用后将会得到一个 TypeError 错误。

 那如果在你自定义的类中使用特殊方法呢？

 ### 内建函数的重载

 在对象模型中定义的许多特殊方法能应用在改变函数行为上，如 len, ads, hash, div, mod 等等。为了实现，你可以在自定义的类中实现相对应的特殊方法。

 #### 使用 len() 返回对象长度

 为了改变 len() 的默认行为，你需要在你的类中实现 __len__() 特殊方法。不管什么时候调用 len() 方法，都会使用你自定义的 __len__() 方法。现在来实现下开始说到的购物清单：

 ```Python
 >>> class Order:
 ...     def __init__(self, cart, customer):
 ...         self.cart = list(cart)
 ...         self.customer = customer
 ...
 ...     def __len__(self):
 ...         return len(self.cart)
 ...
 >>> order = Order(['banana', 'apple', 'mango'], 'Real Python')
 >>> len(order)
 3
 ```

 可以看出，现在可以通过内建的 len() 方法来获取清单的大小。当你没有自定义 __len__() 方法时，去使用 len() 时，会出现 TypeError :

 ```Python
 >>> class Order:
 ...     def __init__(self, cart, customer):
 ...         self.cart = list(cart)
 ...         self.customer = customer
 ...
 >>> order = Order(['banana', 'apple', 'mango'], 'Real Python')
 >>> len(order)  # Calling len when no __len__
 Traceback (most recent call last):
   File "<stdin>", line 1, in <module>
 TypeError: object of type 'Order' has no len()
 ```

 但是，需要注意的是，在重载 len() 方法时，必须要返回一个整型值。如果你的方法返回的不是整型值，那么就会得到一个 TypeError 。

 ```Python
 >>> class Order:
...     def __init__(self, cart, customer):
...         self.cart = list(cart)
...         self.customer = customer
...
...     def __len__(self):
...         return float(len(self.cart))  # Return type changed to float
...
>>> order = Order(['banana', 'apple', 'mango'], 'Real Python')
>>> len(order)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'float' object cannot be interpreted as an integer
```

#### 使用 str() 打印对象

内建函数 str() 的作用是把一个对象实例转换成 str 对象，或者更形象地说，变成一个对非计算机用户来说更友好的字符表示形式。你可以通过 __str__() 方法定义字符的格式。而且， __str__() 方法也被Python在你的对象被使用 print() 函数时调用。

### 重载操作符

改变操作符的默认行为和内建函数类似。在你的自定义类中定义对应的特殊方法，操作符的行为将是你定义的方法行为。

#### 自定义对象使用 +

+ 操作符对应的特殊方法为 __add__() 。在类中添加自定义的 __add__() 方法将改变操作符的默认行为。在这里建议，自定义 __add__() 方法时返回一个新的对象而不是在原有的对象上修改返回。

```Python
>>> a = 'Real'
>>> a + 'Python'  # Gives new str instance
'RealPython'
>>> a  # Values unchanged
'Real'
>>> a = a + 'Python'  # Creates new instance and assigns a to it
>>> a
'RealPython'
```

可以看出，当在 str 上使用 + 操作符时是返回一个新的对象实例，从而使得 a 的值未曾改变。如果要改变它，我们需要显式的给 a 赋值一个新的值。

让我们来实现之前的例子，使用操作符 + 在原有的清单上添加新的清单。

```Python
>>> class Order:
...     def __init__(self, cart, customer):
...         self.cart = list(cart)
...         self.customer = customer
...
...     def __add__(self, other):
...         new_cart = self.cart.copy()
...         new_cart.append(other)
...         return Order(new_cart, self.customer)
...
>>> order = Order(['banana', 'apple'], 'Real Python')

>>> (order + 'orange').cart  # New Order instance
['banana', 'apple', 'mango']
>>> order.cart  # Original instance unchanged
['banana', 'apple']

>>> order = order + 'mango'  # Changing the original instance
>>> order.cart
['banana', 'apple', 'mango']
```

#### 操作符 +=

操作符 += 代表是的是表达式 obj1 = obj1 + obj2 的简写形式。其对应的特殊方法是 __iadd__() 。需要注意的是， __iadd__() 方法应该修改对象自身并返回结果。

```Python
>>> class Order:
...     def __init__(self, cart, customer):
...         self.cart = list(cart)
...         self.customer = customer
...
...     def __iadd__(self, other):
...         self.cart.append(other)
...         return self
...
>>> order = Order(['banana', 'apple'], 'Real Python')
>>> order += 'mango'
>>> order.cart
['banana', 'apple', 'mango']
```

从上可以看出，任何的改变都直接修改了对象本身并返回。那如果在返回值是随机的值，比如字符或整数，会发生什么呢？

```Python
>>> class Order:
...     def __init__(self, cart, customer):
...         self.cart = list(cart)
...         self.customer = customer
...
...     def __iadd__(self, other):
...         self.cart.append(other)
...         return 'Hey, I am string!'
...
>>> order = Order(['banana', 'apple'], 'Real Python')
>>> order += 'mango'
>>> order
'Hey, I am string!'
```

虽然相对应的条目被添加进清单中，但是最终的返回值被 __iadd__() 方法修改了。Python隐式地为你作了一次赋值操作。如果在你实现时忘记返回值是，这将会出现一些意想不到的行为：

```Python
>>> class Order:
...     def __init__(self, cart, customer):
...         self.cart = list(cart)
...         self.customer = customer
...
...     def __iadd__(self, other):
...         self.cart.append(other)
...
>>> order = Order(['banana', 'apple'], 'Real Python')
>>> order += 'mango'
>>> order  # No output
>>> type(order)
NoneType
```

因为所有的Python函数或方法如果没有显式返回值时，默认会返回 None。当你的交互模式下打印 order 时不会有任何的输出，但是当你查看其对象类型时，你会发现返回了 NoneType。所以，在确保你自定义实现 __iadd__() 方法时记得指定返回值。

和 __iadd__() 类似地，还有 __isub__(), __imul__(), __idiv__()，分别对应操作符 -=, \*=, /=。

#### 在你的对象上使用切片

操作符 [] 被称作为索引操作，常被用在序列中得到某个索引的值，或得到字典中某个键的值，或通过切片的方式得到序列的部分值。你可以通过 __getitem__() 特殊方法来改变其行为。

在 Order 类中我们可以直接通过对象访问到具体的清单

```Python
>>> class Order:
...     def __init__(self, cart, customer):
...         self.cart = list(cart)
...         self.customer = customer
...
...     def __getitem__(self, key):
...         return self.cart[key]
...
>>> order = Order(['banana', 'apple'], 'Real Python')
>>> order[0]
'banana'
>>> order[-1]
'apple'
```

可以注意到，__getitem__()方法的参数名不是 index 而是 key 。这是因为此参数有三种可能的类型，第一种是：整型值，这种情况下它可能是索引或者字典的键；一种情况是：字符，这种情况下是字典的键；还有一种情况是：切片对象，这种情况下，它将对对象进行切片操作。

因为我们内部的数据结构是列表，我们可以使用操作符 [] 进行列表的切片操作， 参数 key 是一切片对象。

#### 反向操作符

当你定义好了 __add__(), __sub__(), __mul__() 之后，你的特殊方法只会对左边的值操作起作用，这些操作符将不会对右边的操作方式起作用，看下面的例子：

```Python
>>> class Mock:
...     def __init__(self, num):
...         self.num = num
...     def __add__(self, other):
...         return Mock(self.num + other)
...
>>> mock = Mock(5)
>>> mock = mock + 6
>>> mock.num
11

>>> mock = 6 + Mock(5)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'Mock'
```

可以看出，自定义对象虽然定义了 __add__() 方法，但是对自定义对象在右边的操作符 + 处理会出现 TypeError 错误。其实，如果上述的方式同样起作用的话，Python提供了反向特殊方法比如
 __radd__(), __rsub__(), __rmul__() 等等。

 它们处理比如 x + obj, x - obj, 和 x * obj，这里的 x 不是我们自定义的类实例。和 __add__() 一样，这些反向特殊函数应该返回新的实例对象而不是去修改对象实例本身。

 下面看下在 Order 中实现 __radd__() 方法：

 ```Python
 >>> class Order:
 ...     def __init__(self, cart, customer):
 ...         self.cart = list(cart)
 ...         self.customer = customer
 ...
 ...     def __add__(self, other):
 ...         new_cart = self.cart.copy()
 ...         new_cart.append(other)
 ...         return Order(new_cart, self.customer)
 ...
 ...     def __radd__(self, other):
 ...         new_cart = self.cart.copy()
 ...         new_cart.insert(0, other)
 ...         return Order(new_cart, self.customer)
 ...
 >>> order = Order(['banana', 'apple'], 'Real Python')

 >>> order = order + 'orange'
 >>> order.cart
 ['banana', 'apple', 'orange']

 >>> order = 'mango' + order
 >>> order.cart
 ['mango', 'banana', 'apple', 'orange']
 ```
