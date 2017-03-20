---
title: What is 'if __name__ == "__main__"' for?
date: 2017-03-19 23:41:00
tags:
categories: "Python学习笔记"
---

The if \__name__ == \"\__main__": ... trick exists in Python so that our Python files can act as either **reusable modules**, or as **standalone programs**. As a toy example, let’s say that we have two files:

![](/images/categories/python/008/1.png)

<!--more-->

In this example, we’ve written mymath.py to be both used as a utility module, as well as a standalone program. We can run mymath standalone by doing this:

![](/images/categories/python/008/2.png)

But we can also use mymath.py as a module; let’s see what happens when we run mygame.py:

![](/images/categories/python/008/3.png)

Notice that here we don’t see the ‘test’ line that mymath.py had near the bottom of its code. That’s because, in this context, mymath is not the main program. That’s what the if \__name__ == \"\__main__": ... trick is used for.
