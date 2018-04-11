---
title: Python 中的枚举类型
date: 2018-04-11 19:01:08
tags:
categories: "Python学习笔记"
---

转载自：[Python 中的枚举类型](http://python.jobbole.com/84112/)

枚举类型可以看作是一种标签或是一系列常量的集合，通常用于表示某些特定的有限集合，例如星期、月份、状态等。Python 的原生类型（Built-in types）里并没有专门的枚举类型，但是我们可以通过很多方法来实现它，例如字典、类等：

```python
WEEKDAY = {
    'MON': 1,
    'TUS': 2,
    'WEN': 3,
    'THU': 4,
    'FRI': 5
}
class Color:
    RED   = 0
    GREEN = 1
    BLUE  = 2
```

上面两种方法可以看做是简单的枚举类型的实现，如果只在局部范围内用到了这样的枚举变量是没有问题的，但问题在于它们都是可变的（mutable），也就是说可以在其它地方被修改从而影响其正常使用：

```python
WEEKDAY['MON'] = WEEKDAY['FRI']
print(WEEKDAY)
{'FRI': 5, 'TUS': 2, 'MON': 5, 'WEN': 3, 'THU': 4}
```

通过类定义的枚举甚至可以实例化，变得不伦不类：

```python
c = Color()
print(c.RED)
Color.RED = 2
print(c.RED)
```

```sh
0
2
```

<!--more-->

当然也可以使用不可变类型（immutable），例如元组，但是这样就失去了枚举类型的本意，将标签退化为无意义的变量：

```python
COLOR = ('R', 'G', 'B')
print(COLOR[0], COLOR[1], COLOR[2])
```

```sh
R G B
```

为了提供更好的解决方案，Python 通过 PEP 435 在 3.4 版本中添加了 enum 标准库，3.4 之前的版本也可以通过 pip install enum 下载兼容支持的库。enum 提供了 Enum/IntEnum/unique 三个工具，用法也非常简单，可以通过继承 Enum/IntEnum 定义枚举类型，其中 IntEnum 限定枚举成员必须为（或可以转化为）整数类型，而 unique 方法可以作为修饰器限定枚举成员的值不可重复：

```python
from enum import Enum, IntEnum, unique
 
try:
    @unique
    class WEEKDAY(Enum):
        MON = 1
        TUS = 2
        WEN = 3
        THU = 4
        FRI = 1
except ValueError as e:
    print(e)
```

输出:

```python
duplicate values found in : FRI -> MON
```

```python
try:
    class Color(IntEnum):
        RED   = 0
        GREEN = 1
        BLUE  = 'b'
except ValueError as e:
    print(e)
```

输出：

```sh
invalid literal for int() with base 10: 'b'
```

更有趣的是 Enum 的成员均为单例（Singleton），并且不可实例化，不可更改：

```python
class Color(Enum):
    R = 0
    G = 1
    B = 2
```

```python
try:
    Color.R = 2
except AttributeError as e:
    print(e)
```

```
Cannot reassign members.
```

虽然不可实例化，但可以将枚举成员赋值给变量：

```python
red = Color(0)
green = Color(1)
blue = Color(2)
print(red, green, blue)
```

```
Color.R Color.G Color.B
```

也可以进行比较判断：

```python
print(red is Color.R)
print(red == Color.R)
print(red is blue)
print(green != Color.B)
print(red == 0) # 不等于任何非本枚举类的值
```

```
True
True
False
True
False
```

最后一点，由于枚举成员本身也是枚举类型，因此也可以通过枚举成员找到其它成员：

```python
print(red.B)
print(red.B.G.R)
```

```
Color.B
Color.R
```

但是要谨慎使用这一特性，因为可能与成员原有的命名空间中的名称相冲突：

```python
print(red.name, ':', red.value)
 
class Attr(Enum):
    name  = 'NAME'
    value = 'VALUE'
print(Attr.name.value, Attr.value.name)
```

```
R : 0
NAME value
```