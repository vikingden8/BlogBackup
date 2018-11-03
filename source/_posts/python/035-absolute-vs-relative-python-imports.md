---
title: Python中的绝对和相对导入
date: 2018-11-03 13:48:08
tags:
categories: "Python学习笔记"
---

翻译自：[Absolute vs Relative Imports in Python](https://realpython.com/absolute-vs-relative-python-imports/)

如果你正在工作中的Python项目中有多个源文件存在时，你肯定会碰到要使用 import 导入的语法。

即使是拥有几个项目的高手，有时候也会对使用 import 语句感到困惑。或许你阅读此文的目的就是你想要深入了解下Python的 import 工作原理，特别是绝对导入和相对导入。

在这个系列中，你将会了解两者之间的区别，以及它们各自的有缺点。

<!--more-->

### Import 快速入门

若想要知道 import 是怎样工作的，你需要先对 Python 的模块和包有很好的理解和掌握。一个  Python 模块其实就是以 .py 为后缀的文件，一个 Python 包就是在任意的文件夹中包含有多个模块文件（在Python 2中此文件夹下必须有一个 \_\_init\_\_.py 的文件）。

当你写的模块文件中需要访问另一个模块或包下的代码时，该怎么做呢？你可以通过导入实现。

#### 导入的工作原理

但是，导入是如何具体工作的呢？假设你需要导入一个为 abc 的模块，你可以像如下导入：

```python
import abc
```

首先，Python会在 sys.modules 中寻找名为 abc 的模块。这里面保存了上一次导入的所有缓存模块。

如果在模块缓存中没有找到它，Python会在系统内建的模块中继续搜查。内建模块就是Python安装的时候已存在，可以在 [Python 标准库](https://docs.python.org/3/library/)中找到所有的内建模块。如果，此时在内建模块中仍未找到名为 abc 的库时，Python 会在定义的 sys.path 列出的目录下去继续查找。这个目录集合一般会包含当前的工作目录，同时也会优先从当前目录开始。

当Python找到模块后，它将会在局部作用域中绑定。这意味着已定义好 abc 以及可以在当前的模块被使用而不至抛出一个 NameError 错误。

如果在以上的搜索中都未找到，你将会得到一个 ModuleNotFoundError。你可以在[官方文档](https://docs.python.org/3/reference/import.html)中查看了解更多的导入文档说明。

#### 导入语句语法

现在你了解了 import 的工作原理，下面来看下其语法。你可以同时导入包和模块，你也可以导入一个包或者模块中的特定对象。

一般来说有两种类型的导入语法。当你使用下面的方式时，意味着直接导入资源，如下：

```Python
import abc
```

abc 可以为一个包或者一个模块。

当你使用如下的语法时，你导入的是另外一个包或模块下的资源：

```Python
from abc import xyz
```

xyz 可以是一个模块，子包或者对象，比如一个类或者函数。

你也可以选择性的对导入的资源进行重命名，像下面那样：

```Python
import abc as other_name
```

在当前的脚本中，把导入的资源 abc 重命名为了 other_name。需要注意的是，现在必须通过 other_name 来引用，而之前的 abc 将不会被识别。

#### 导入语句的样式

在[PEP 8](https://pep8.org/#imports)中对编写导入语句有下面的几条建议，总结如下：

1. 导入语句必须写在源代码文件的顶部且在模块的说明和文档之后。

2. 导入语句必须根据导入的类型进行分隔。主要有如下的三类：

* 标准库导入

* 第三方库导入

* 本地程序导入

3. 每一种类型的导入应当以一个空白行来分隔。

同时，一个很好的建议就是在每一类的导入语句中按导入的字母顺序进行排序导入。这将会在导入语句中寻找特定的导入更加的容易，尤其是当你的文件中包含很多的导入语句情况下。

下面是如何格式化导入语句的示例：

```Python
"""Illustration of good import statement styling.

Note that the imports come after the docstring.

"""

# Standard library imports
import datetime
import os

# Third party imports
from flask import Flask
from flask_restful import Api
from flask_sqlalchemy import SQLAlchemy

# Local application imports
from local_module import local_class
from local_package import local_function
```

上面的导入语句通过空白的行，被分成了很明显的三组。同时，每一组内都是以字母的顺序排列。


### 绝对导入

绝对导入指的是以项目中的根目录为全路径导入资源文件。

#### 语法

假如有如下的目录结构：

```sh
└── project
    ├── package1
    │   ├── module1.py
    │   └── module2.py
    └── package2
        ├── __init__.py
        ├── module3.py
        ├── module4.py
        └── subpackage1
            └── module5.py
```

目录 project 下有两个子目录 package1 和 package2。在 package1 中有两个文件 module1.py 和 module2.py。

在 package2 中有两个文件 module3.py，module4.py以及 \_\_init\_\_.py。同时还包含有一个 subpackage1 目录，在此目录下包含 module5.py 文件。

让我们假设下列情况：

1. package1/module2.py 中有一个 function1 函数。

2. package2/\_\_init\_\_.py 中有一个 class1 的类。

3. package2/subpackage1/module5.py 中有一个 function2 的函数。

下面是绝对导入的示例代码：

```Python
from package1 import module1
from package1.module2 import function1
from package2 import class1
from package2.subpackage1.module5 import function2
```

可以看出你必须给出每个包和文件的详细路径。就像文件路径一样，只是使用了.来代替 / 。

#### 绝对导入的优缺点

绝对路径的有点就是它们非常的清晰明了。通过查看导入语句，它很容易的区分导入的资源位置。

有时候，绝对导入看起来十分的冗长，依赖于具体的项目结构复杂程度。假如有如下的导入语句：

```Python
from package1.subpackage2.subpackage3.subpackage4.module5 import function6
```

这看起来很荒唐，不是吗？幸运地是，在这种情况下，相对导入就派上用场了。

### 相对导入

相对导入指的是针对当前的位置导入资源，当前的位置也就是导入语句所在的位置。有两种类型的相对导入：显式和隐式。在Python 3中已经废弃了隐式的导入，在这里将不会作介绍。

#### 语法

相对导入依赖当前的位置。下面是几个示例：

```Python
from .some_module import some_class
from ..some_package import some_function
from . import some_class
```

可以看到，在每一个导入中至少包含有一个 . 。相对导入通过使用 . 来标识特别的路径。

一个 . 意味着要导入的包或者模块就在当前的目录下。两个 . 意味着当前目录的父目录。三个 . 意味着当前目录的父目录的父目录，依此类推。如果使用过 Unix 的操作系统的话你可能会非常的熟悉。

假想有以下的目录结构：

```Python
└── project
    ├── package1
    │   ├── module1.py
    │   └── module2.py
    └── package2
        ├── __init__.py
        ├── module3.py
        ├── module4.py
        └── subpackage1
            └── module5.py
```

同样地：

1. package1/module2.py 中有一个 function1 函数。

2. package2/\_\_init\_\_.py 中有一个 class1 的类。

3. package2/subpackage1/module5.py 中有一个 function2 的函数。

当你在 package1/module1.py 中导入 function1 时，你可以以这种方式：

```Python
# package1/module1.py

from .module2 import function1
```

这里你只需要使用一个 . ，因为 module2 和 module1 在同一目录下。

你可以通过如下的方式在 package2/module3.py 中导入 class1 和 function2 ：

```Python
# package2/module3.py

from . import class1
from .subpackage1.module5 import function2
```

#### 相对导入的优缺点

相对导入一个很明显的优点就是很简洁。依赖于当前的位置，它会把之前的所见的冗长导入语句简化成如下：

```Python
from ..subpackage4.module5 import function6
```

不幸地是，相对导入看起来会显得很混乱，特别当那些共用的项目目录结构经常变动。相对导入的可读性相对于绝对导入来说会比较差，同时也不容易看出被导入的资源路径所在。
