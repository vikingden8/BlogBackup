---
title: Python装饰器入门
date: 2018-09-15 16:40:08
tags:
categories: "Python学习笔记"
---

本文翻译自[有删改]：[Primer on Python Decorators](https://realpython.com/primer-on-python-decorators/)

在这篇文章中，我们将介绍什么是装饰器和如何来创建以及使用它。装饰器通过使用高阶函数提供了一种简单的语法。

先来看一下定义，装饰器就是使用另一个方法并且继承这个函数的所有的功能而不要显式的修改这个函数的函数。

听起来比较疑惑不解，但是当你看了几个示例了解了装饰器是如何工作后就不会了。

<!--more-->

### 函数

如果你想要理解装饰器的话，首先你得先理解方法是如何工作的。一个函数其实就是根据给定的参数返回一个特定的值。看下面的示例：

```Python
>>> def add_one(number):
...     return number + 1

>>> add_one(2)
3
```

正常来说，Python中的函数可能不仅仅是根据输入而返回特定的输出。比如函数 print() ：它返回的是 None。然而，为了理解装饰器，可以理想地认为函数的作用就是根据输入的参数返回特定值。

### 第一类对象

在Python中，函数是第一类对象。这意味着函数可以被传递当作参数，就跟其他对象一样（string，int，float，list等等）。

```Python
def say_hello(name):
    return f"Hello {name}"

def be_awesome(name):
    return f"Yo {name}, together we are the awesomest!"

def greet_bob(greeter_func):
    return greeter_func("Bob")
```

上面的例子中，say_hello() 和 be_awesome() 函数是一般的函数，参数都是 name 。 然而，greet_bob() 函数的参数是一个函数。我们可以把 say_hello() 和 be_awesome() 当作参数传递给 greet_bob() 函数.

```Python
>>> greet_bob(say_hello)
'Hello Bob'

>>> greet_bob(be_awesome)
'Yo Bob, together we are the awesomest!'
```

需要注意的是， greet_bob(say_hello) 引用了两个函数，但是是不同的方式：greet_bob() 和 say_hello。say_hello 函数没有使用带括号。这也就是只是传递了函数的引用。这个函数并没有被执行。而 greet_bob() 函数，使用了括号，它就跟正常函数调用一样。

### 内部函数

在Python中可以在一个函数的内部再定义函数。这样的函数被称作为内部函数。下面是一个函数中包含两个内部函数：

```Python
def parent():
    print("Printing from the parent() function")

    def first_child():
        print("Printing from the first_child() function")

    def second_child():
        print("Printing from the second_child() function")

    second_child()
    first_child()
```

当你调用 parent() 函数时会发生什么呢？下面是输出：

```Python
>>> parent()
Printing from the parent() function
Printing from the second_child() function
Printing from the first_child() function
```

需要注意的是，内部函数定义的位置无关紧要。像其他函数一样，函数的打印只有在函数调用执行后
才有输出。

更重要的是，内部函数直到其父函数被调用后才被定义。它们的作用域只在 parent() 方法中：也就是它们只是 parent() 函数的局部变量。若在父函数之后调用 first_child() ，将得到一个错误：

```Python
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'first_child' is not defined
```

不管你什么时候调用 parent()，它的内部函数 first_child() 和 second_child() 也被同时调用了。但是因为它们的作用域，在 parent() 函数之外它们无法被调用。

### 函数返回函数

Python也允许你使用函数作为返回值。下面的例子返回一个定义好的内部函数：

```Python
def parent(num):
    def first_child():
        return "Hi, I am Emma"

    def second_child():
        return "Call me Liam"

    if num == 1:
        return first_child
    else:
        return second_child
```

注意，返回的 first_child 没有带括号。也就是说这里返回的是 first_child 的函数引用。反过来，如果返回的是 first_child() 带了括号则是代表这个函数执行后的结果值。

```Python
>>> first = parent(1)
>>> second = parent(2)

>>> first
<function parent.<locals>.first_child at 0x7f599f1e2e18>

>>> second
<function parent.<locals>.second_child at 0x7f599dad5268>
```

从输出可以简单的看出，first 变量指向的是 parent() 函数定义的局部变量 first_child() 的函数引用。而 second 则是 second_child() 的函数引用。

你现在可以像正常的函数那样使用 first 和 second 。即使它们所对应的内部函数不能被直接调用：

```Python
>>> first()
'Hi, I am Emma'

>>> second()
'Call me Liam'
```

### 简单的装饰器

你已经看到Python的函数就像其他的对象一样，现在更进一步来看下Python中的装饰器到底是怎样的。

```Python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

def say_whee():
    print("Whee!")

say_whee = my_decorator(say_whee)
```

你能猜出当调用 say_whee() 函数会输出什么吗？试试看：

```Python
>>> say_whee()
Something is happening before the function is called.
Whee!
Something is happening after the function is called.
```

这个装饰的功能其实是在下面这行代码：

```Python
say_whee = my_decorator(say_whee)
```

实际上，say_whee 现在传递给了内部函数 wrapper()。记住，当你调用 my_decorator(say_whee) 的时候，这里返回是函数 wrapper

```Python
>>> say_whee
<function my_decorator.<locals>.wrapper at 0x7f3c5dfd42f0>
```

然而， wrapper() 调用的过程中引用来原来的 say_whee()，在其执行的前后都调用了函数 print() 。

简单来说：**装饰器包含了一个函数并修改了它的行为**。

下面来看另外一个示例，为了不打扰你的邻居，下面的例子只会在白天运行被装饰过的代码：

```Python
from datetime import datetime

def not_during_the_night(func):
    def wrapper():
        if 7 <= datetime.now().hour < 22:
            func()
        else:
            pass  # Hush, the neighbors are asleep
    return wrapper

def say_whee():
    print("Whee!")

say_whee = not_during_the_night(say_whee)
```

如果你试图在晚间调用 say_whee(), 则不会有任何的输出：

```Python
>>> say_whee()
>>>
```

### 语法糖

使用上面的方式来修饰 say_whee() 看起来有点笨重。首先，你最终的代码写了三次的 say_whee。

可幸的是，Python允许你使用 @ 符号来使装饰器简化，下面的代码跟之前的装饰器例子是同样的作用：

```Python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_whee():
    print("Whee!")
```

可以得出，@my_decorator 只是 say_whee = my_decorator(say_whee) 的简写形式。

### 重复使用装饰器

创建一个 decorators.py 的文件，内容如下：

```Python
def do_twice(func):
    def wrapper_do_twice():
        func()
        func()
    return wrapper_do_twice
```

你现在可以通过使用 import 语句在其他的文件中使用这个新的装饰器。

```Python
from decorators import do_twice

@do_twice
def say_whee():
    print("Whee!")
```

当你运行这个示例时，你应当看到之前的 say_whee() 函数被执行了两次：

```Python
>>> say_whee()
Whee!
Whee!
```

### 装饰带参数的函数

现在你有一个函数带有几个参数。现在你仍可以装饰它吗？试试看：

```Python
from decorators import do_twice

@do_twice
def greet(name):
    print(f"Hello {name}")
```

不幸地是，当你运行下面的代码时会抛出异常：

```Python
>>> greet("World")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: wrapper_do_twice() takes 0 positional arguments but 1 was given
```

问题其实就出在内部函数 wrapper_do_twice() 没有带任何的参数，但是却传给了它 name="World"。你可以通过给  wrapper_do_twice() 添加一个参数，但是它却对之前不带参数的 say_whee() 不起作用了。

解决的方法就是在内部函数中使用 \*args 和 \*\*kwargs 。它会接收所有的位置参数和关键字参数。现在重写 decorators.py :

```Python
def do_twice(func):
    def wrapper_do_twice(*args, **kwargs):
        func(*args, **kwargs)
        func(*args, **kwargs)
    return wrapper_do_twice
```

内部函数 wrapper_do_twice() 现在接收所有类型的参数，通过它传递给被装饰的函数。现在你使用 say_whee() 和 greet() 都不会有问题出现：

```Python
>>> say_whee()
Whee!
Whee!

>>> greet("World")
Hello World
Hello World
```

### 被装饰的函数返回值

如果被装饰的函数有返回值会发生什么呢？这个取决于装饰器。看下下面的示例：

```Python
from decorators import do_twice

@do_twice
def return_greeting(name):
    print("Creating greeting")
    return f"Hi {name}"
```

尝试获取返回值：

```Python
>>> hi_adam = return_greeting("Adam")
Creating greeting
Creating greeting
>>> print(hi_adam)
None
```

函数返回了 None 。

主要是因为 do_twice_wrapper() 没有显式的返回任何值，当调用 return_greeting("Adam") 后隐式地返回了 None 。

为了修复这个问题，你需要在你的包装函数中返回被装饰函数的返回值。

```Python
def do_twice(func):
    def wrapper_do_twice(*args, **kwargs):
        func(*args, **kwargs)
        return func(*args, **kwargs)
    return wrapper_do_twice
```

最终调用函数的返回值当作返回值：

```Python
>>> return_greeting("Adam")
Creating greeting
Creating greeting
'Hi Adam'
```

### 到底是谁

在Python中我们可以在运行时查看到一个对象它的属性。比如，一个函数知道自己的名字和文档说明。

```Python
>>> print
<built-in function print>

>>> print.__name__
'print'

>>> help(print)
Help on built-in function print in module builtins:

print(...)
    <full help message>
```

它也同样适应于你自定义的函数：

```Python
>>> say_whee
<function do_twice.<locals>.wrapper_do_twice at 0x7f43700e52f0>

>>> say_whee.__name__
'wrapper_do_twice'

>>> help(say_whee)
Help on function wrapper_do_twice in module decorators:

wrapper_do_twice()
```

然而，当你的函数被装饰过后，say_whee() 看起来有点怀疑。从输出可以看到其成为了 do_twice() 装饰器内部的内部函数 wrapper_do_twice()(这段待优化)。

为了修复这个问题，装饰器可以使用  \@functools.wraps，这会保留原来函数的信息。

```Python
import functools

def do_twice(func):
    @functools.wraps(func)
    def wrapper_do_twice(*args, **kwargs):
        func(*args, **kwargs)
        return func(*args, **kwargs)
    return wrapper_do_twice
```

你现在无需修改被装饰的函数 say_whee() :

```Python
>>> say_whee
<function say_whee at 0x7ff79a60f2f0>

>>> say_whee.__name__
'say_whee'

>>> help(say_whee)
Help on function say_whee in module whee:

say_whee()
```

非常好！现在的 say_whee() 被装饰后原来信息还在。


### 真实示例

下面再来多看几个有用的示例。你会注意到其实它们和之前例子的模式如出一辙：

```Python
import functools

def decorator(func):
    @functools.wraps(func)
    def wrapper_decorator(*args, **kwargs):
        # Do something before
        value = func(*args, **kwargs)
        # Do something after
        return value
    return wrapper_decorator
```

#### 时间函数

创建一个 \@time 的装饰器。它将会计算一个函数的执行时长，并在控制台输出，下面是代码：

```Python
import functools
import time

def timer(func):
    """Print the runtime of the decorated function"""
    @functools.wraps(func)
    def wrapper_timer(*args, **kwargs):
        start_time = time.perf_counter()    # 1
        value = func(*args, **kwargs)
        end_time = time.perf_counter()      # 2
        run_time = end_time - start_time    # 3
        print(f"Finished {func.__name__!r} in {run_time:.4f} secs")
        return value
    return wrapper_timer

@timer
def waste_some_time(num_times):
    for _ in range(num_times):
        sum([i**2 for i in range(10000)])
```

这个函数的工作原理是在函数执行之前记录执行的时间点(\#1)以及在函数执行之后再记录其时间点(\#2)。其实这个函数执行所花的时间就是两者之差(\#3)。

```Python
>>> waste_some_time(1)
Finished 'waste_some_time' in 0.0010 secs

>>> waste_some_time(999)
Finished 'waste_some_time' in 0.3260 secs
```

#### 调试代码

下面的 \@debug 装饰器在每次调用其装饰过的函数时会打印出所有的参数和返回值：

```Python
import functools

def debug(func):
    """Print the function signature and return value"""
    @functools.wraps(func)
    def wrapper_debug(*args, **kwargs):
        args_repr = [repr(a) for a in args]                      # 1
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]  # 2
        signature = ", ".join(args_repr + kwargs_repr)           # 3
        print(f"Calling {func.__name__}({signature})")
        value = func(*args, **kwargs)
        print(f"{func.__name__!r} returned {value!r}")           # 4
        return value
    return wrapper_debug
```

当使用一个位置参数和一个关键字参数时，来看下这个装饰器是如何工作的：

```Python
@debug
def make_greeting(name, age=None):
    if age is None:
        return f"Howdy {name}!"
    else:
        return f"Whoa {name}! {age} already, you are growing up!"
```

注意 \@debug 装饰器的签名打印输出以及函数 make_greeting() 的返回值：

```Python
>>> make_greeting("Benjamin")
Calling make_greeting('Benjamin')
'make_greeting' returned 'Howdy Benjamin!'
'Howdy Benjamin!'

>>> make_greeting("Richard", age=112)
Calling make_greeting('Richard', age=112)
'make_greeting' returned 'Whoa Richard! 112 already, you are growing up!'
'Whoa Richard! 112 already, you are growing up!'

>>> make_greeting(name="Dorrisile", age=116)
Calling make_greeting(name='Dorrisile', age=116)
'make_greeting' returned 'Whoa Dorrisile! 116 already, you are growing up!'
'Whoa Dorrisile! 116 already, you are growing up!'
```

#### 降低代码运行频率

下面的代码可能看起来没有什么用途。为什么要使你的Python代码运行频率慢下来呢？最常见的场景就是你限制一个函数的调用，比如持续的检查一个资源是否已经被修改了。装饰器 \@slow_down 将会在调用被装饰的函数之前休眠一秒钟：

```Python
import functools
import time

def slow_down(func):
    """Sleep 1 second before calling the function"""
    @functools.wraps(func)
    def wrapper_slow_down(*args, **kwargs):
        time.sleep(1)
        return func(*args, **kwargs)
    return wrapper_slow_down

@slow_down
def countdown(from_number):
    if from_number < 1:
        print("Liftoff!")
    else:
        print(from_number)
        countdown(from_number - 1)
```

通过运行下面的代码来看看 \@slow_down 装饰器的作用：

```Python
>>> countdown(3)
3
2
1
Liftoff!
```

#### 注册插件

装饰器不必一定要包装其装饰的函数。

```Python
import random
PLUGINS = dict()

def register(func):
    """Register a function as a plug-in"""
    PLUGINS[func.__name__] = func
    return func

@register
def say_hello(name):
    return f"Hello {name}"

@register
def be_awesome(name):
    return f"Yo {name}, together we are the awesomest!"

def randomly_greet(name):
    greeter, greeter_func = random.choice(list(PLUGINS.items()))
    print(f"Using {greeter!r}")
    return greeter_func(name)
```

装饰器 \@register 在全局变量字典 PLUGINS 中存储了每个被装饰过函数的引用。注意到的是，这里并没有使用内部函数，或者使用 \@functools.wraps 。

函数 randomly_greet() 随机地选择一个已注册的函数来调用。

```Python
>>> PLUGINS
{'say_hello': <function say_hello at 0x7f768eae6730>,
 'be_awesome': <function be_awesome at 0x7f768eae67b8>}

>>> randomly_greet("Alice")
Using 'say_hello'
'Hello Alice'
```

### 高级装饰器

现在，你已经知道如何创建一个简单的装饰器。你已经对装饰器有了很好地理解以及它是如何工作的。

下面我们来看下装饰器的一些高级特性。

#### 装饰类

有两种不同的方式来装饰类。第一种方式很接近于你已经学会的如何装饰函数。

下面我们在一个类的某些方法上使用之前定义过的装饰器 \@debug 和 \@timer：

```Python
from decorators import debug, timer

class TimeWaster:
    @debug
    def __init__(self, max_num):
        self.max_num = max_num

    @timer
    def waste_time(self, num_times):
        for _ in range(num_times):
            sum([i**2 for i in range(self.max_num)])
```

当你使用这个类的时候，你会看到装饰器的作用：

```Python
>>> tw = TimeWaster(1000)
Calling __init__(<time_waster.TimeWaster object at 0x7efccce03908>, 1000)
'__init__' returned None

>>> tw.waste_time(999)
Finished 'waste_time' in 0.3376 secs
```

另外一种方法就是在整个类上应用装饰器。这类似下面的方式：

```Python
from dataclasses import dataclass

@dataclass
class PlayingCard:
    rank: str
    suit: str
```

基本和函数的装饰器语法类似。上面的例子其实也可以使用这种方式：PlayingCard = dataclass(PlayingCard) 。

写一个类的装饰器和写一个函数的装饰器十分类似。唯一的不同之处就是装饰器接收的参数是类而不是函数。实际上，之前定义过的装饰器应用在类上都没问题。但是当你在类上使用它们时，它的作用可能并不是你所要的结果。下面看看装饰器 \@timer 应用在一个类上：

```Python
from decorators import timer

@timer
class TimeWaster:
    def __init__(self, max_num):
        self.max_num = max_num

    def waste_time(self, num_times):
        for _ in range(num_times):
            sum([i**2 for i in range(self.max_num)])
```

这里，\@timer 仅仅只是衡量了初始化类的时间：

```Python
>>> tw = TimeWaster(1000)
Finished 'TimeWaster' in 0.0000 secs

>>> tw.waste_time(999)
>>>
```

#### 多个装饰器

你可以在一个函数上应用多个装饰器：

```Python
from decorators import debug, do_twice

@debug
@do_twice
def greet(name):
    print(f"Hello {name}")
```

试想，所有的装饰器都会根据给定的顺序执行。也就是说，\@debug 调用 \@do_twice，\@do_twice 调用 greet()，或者 debug(do_twice(greet())) ：

```Python
>>> greet("Eva")
Calling greet('Eva')
Hello Eva
Hello Eva
'greet' returned None
```

注意观察，如果我们改变 \@debug 和 \@do_twice 的顺序：

```Python
from decorators import debug, do_twice

@do_twice
@debug
def greet(name):
    print(f"Hello {name}")
```

在这种情况下，输出结果将会是下面这样：

```Python
>>> greet("Eva")
Calling greet('Eva')
Hello Eva
'greet' returned None
Calling greet('Eva')
Hello Eva
'greet' returned None
```

#### 装饰器带参数

有时候，给你的装饰器传递必要的参数很有用途。比如 \@do_twice 可以是 \@repeat(num_times) 。被装饰的函数执行次数可以通过参数来指定。

这将让你做类似下面的事情：

```Python
@repeat(num_times=4)
def greet(name):
    print(f"Hello {name}")
```

```Python
>>> greet("World")
Hello World
Hello World
Hello World
Hello World
```

你的代码可能是下面这样子：

```Python
def repeat(num_times):
    def decorator_repeat(func):
        ...  # Create and return a wrapper function
    return decorator_repeat
```

通常来说，装饰器创建和返回来一个内部的包装函数。

```Python
def repeat(num_times):
    def decorator_repeat(func):
        @functools.wraps(func)
        def wrapper_repeat(*args, **kwargs):
            for _ in range(num_times):
                value = func(*args, **kwargs)
            return value
        return wrapper_repeat
    return decorator_repeat
```

下面来看下结果的输出是不是预期：

```Python
@repeat(num_times=4)
def greet(name):
    print(f"Hello {name}")
```

输出：

```Python
>>> greet("World")
Hello World
Hello World
Hello World
Hello World
```

结果跟预期一致。
