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


不同编辑器对空格的显示逻辑总是一样的，但是对于tab却五花八门。不仅仅是python，对于编程来说用space替代tab不管对于哪种语言都是一个好的选择，因为代码文件可能会在不同的环境、用不同的编辑器打开，而对于space的处理几乎所有的编辑器、所有的OS环境都是一样的，而对于tab的处理却不尽相同，有的会直接展开成空格，有的不会，而且展开为空格的话，有的是4个有的是8个，这样会造成代码的格式看起来不一致。对于python来说这个问题更加重要，因为缩进对于python来说是语法的一部分，所以将tab弄成space会减少很多麻烦。