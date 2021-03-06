---
title: Python引用计数
date: 2017-04-13 21:45:16
tags:
categories: "Python学习笔记"
---

要保持追踪内存中的对象，Python使用了引用计数这一简单技术。也就是说Python内部记录着所有使用中的对象各有多少引用。

### 增加引用计数

当对象被创建并（将其引用）赋值给变量时，该对象的引用计数就被设置为1.

当同一个对象（的引用）又被赋值给其他变量时，或作为参数传递给函数、方法或类实例，或被赋值为一个集合对象的成员时，该对象的一个新的引用，或者称作别名，就被创建（则该对象的引用计数自动加1）。

```sh
x = 3.14
y = x
```

<!--more-->

语句x=3.14创建了一个浮点型对象并将其引用赋值给x。x是第一个引用，因此，该对象的引用计数被设置为1.语句y=x创建了一个指向同一对象的别名y。事实上，并没有为y创建一个新对象，而是该对象的引用计数增加了一次。

对象的引用计数增加主要是以下的几种方式：

  * 对象被创建
  ```sh
  x = 3.14
  ```

  * 或另外的别名被创建
  ```sh
  y = x
  ```

  * 或被作为参数传递给函数（新的本地引用）
  ```sh
  foo(x)
  ```

  * 或称为容器对象的一个元素
  ```sh
  myList = [123, x , "xyz"]
  ```

### 减少引用计数

当对象的引用被销毁时，引用计数会减少。最明显的例子就是当引用离开其作用范围时，这种情况最经常出现在函数运行结束时，所有局部变量都被自动销毁，对象的引用计数也就随之减少。

当变量被赋值给另外一个对象时，原对象的引用计数也会自动减1：

```sh
x = 3.14
y = x
x = 6.543
```

当浮点型对象3.14被创建并被赋值给x时，它的引用计数是1.当增加了一个别名y时，它的引用计数变成了2.不过当x被重新赋值为6.543时，3.14对象的引用计数自动减1，又重新变成了1。

一个对象的引用在以下的情况中会减少：

  * 有个本地引用离开了其作用范围。

  * 对象的别名被显式销毁

  ```sh
  del x
  ```

  * 对象的一个别名被赋值给其他对象
  ```sh
  x = 123
  ```

  * 对象从一个容器中被移除
  ```sh
  myList.remove(x)
  ```

  * 容器对象本身被销毁
  ```sh
  del myList
  ```

当对象的引用计数减为0，这会导致该对象从此“无法访问”或“无法抵达”。从此刻起，该对象就成为垃圾回收机制的回收对象。注意任何追踪或者调试程序会给一个对象增加一个额外的引用，这会推迟该对象被回收的时间。
