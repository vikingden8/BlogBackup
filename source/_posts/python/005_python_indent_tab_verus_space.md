---
title: Python indentation:Tab or Space?
date: 2017-03-18 22:19:54
tags:
categories: "Python学习笔记"
---

[PEP8](https://www.python.org/dev/peps/pep-0008/) 是这么说的，建议使用空格缩进。

>Spaces are the preferred indentation method.

>Tabs should be used solely to remain consistent with code that is already indented with tabs.

>Python 3 disallows mixing the use of tabs and spaces for indentation.

>Python 2 code indented with a mixture of tabs and spaces should be converted to using spaces exclusively.

>When invoking the Python 2 command line interpreter with the -t option, it issues warnings about code that illegally mixes tabs and spaces. When using -tt these warnings become errors. These options are highly recommended!

<!--more-->


不同编辑器对空格的显示逻辑总是一样的，但是对于tab却五花八门。不仅仅是python，对于编程来说用space替代tab不管对于哪种语言都是一个好的选择，因为代码文件可能会在不同的环境、用不同的编辑器打开，而对于space的处理几乎所有的编辑器、所有的OS环境都是一样的，而对于tab的处理却不尽相同，有的会直接展开成空格，有的不会，而且展开为空格的话，有的是4个有的是8个，这样会造成代码的格式看起来不一致。对于python来说这个问题更加重要，因为缩进对于python来说是语法的一部分，所以将tab弄成space会减少很多麻烦。

### Why does Python use indentation for grouping of statements?

Guido van Rossum believes that using indentation for grouping is extremely elegant and contributes a lot to the clarity of the average Python program. Most people learn to love this feature after a while.

Since there are no begin/end brackets there cannot be a disagreement between grouping perceived by the parser and the human reader. Occasionally C programmers will encounter a fragment of code like this:

```C
if (x <= y)
        x++;
        y--;
z++;
```

Only the x++ statement is executed if the condition is true, but the indentation leads you to believe otherwise. Even experienced C programmers will sometimes stare at it a long time wondering why y is being decremented even for x > y.

Because there are no begin/end brackets, Python is much less prone to coding-style conflicts. In C there are many different ways to place the braces. If you’re used to reading and writing code that uses one style, you will feel at least slightly uneasy when reading (or being required to write) another style.

Many coding styles place begin/end brackets on a line by themselves. This makes programs considerably longer and wastes valuable screen space, making it harder to get a good overview of a program. Ideally, a function should fit on one screen (say, 20–30 lines). 20 lines of Python can do a lot more work than 20 lines of C. This is not solely due to the lack of begin/end brackets – the lack of declarations and the high-level data types are also responsible – but the indentation-based syntax certainly helps.

[Why does Python use indentation for grouping of statements?](https://docs.python.org/2/faq/design.html#why-does-python-use-indentation-for-grouping-of-statements)