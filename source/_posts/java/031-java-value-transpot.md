---
title: Java中参数的值传递
date: 2018-01-31 13:45:02
tags:
categories: "Java学习笔记"
---

Java的方法参数传递只有一种，就是 “pass-by-value”，也就是值传递。

* 如果是基本类型，就是将原有的数据拷贝一份，方法内的操作对原有的数据不会有影响。

* 如果是对象类型，这里是容易误解的地方，因为正好规定对象的地址也叫做"reference", 我们将对象作为参数传递的时候实际上是将对象的地址传递进去。

<!--more-->

第一种很好理解，我们主要来分析第二种。

有这么一个方法：

```java
public void tricky(Point arg1, Point arg2) {
    arg1.x = 100;
    arg1.y = 100;
    Point temp = arg1;
    arg1 = arg2;
    arg2 = temp;

    System.out.println("Inside func arg1 x: " + arg1.x + ", y: " + arg1.y);
    System.out.println("Inside func arg2 x: " + arg2.x + ", y: " + arg2.y);
}
```

主函数：

```java
public static void main(String[] args) {
    Point p1 = new Point(2, 3);
    Point p2 = new Point(2, 3);
    System.out.println("p1 x: " + p1.x + ", y: " + p1.y);
    System.out.println("p2 x: " + p2.x + ", y: " + p2.y);

    tricky(p1, p2);

    System.out.println("p1 x: " + p1.x + ", y: " + p1.y);
    System.out.println("p2 x: " + p2.x + ", y: " + p2.y);
}
```

输出：

```sh
viking@Viking ~/work/java-projects/java-demo $ java TestJava
p1 x: 2, y: 3
p2 x: 2, y: 3
Inside func arg1 x: 2, y: 3
Inside func arg2 x: 100, y: 100
p1 x: 100, y: 100
p2 x: 2, y: 3
viking@Viking ~/work/java-projects/java-demo $
```

我们来逐一分析：

第1，2条容易，新建两个Point对象，赋值（2，3）

![](/images/catgories/java/031/01.jpg)

此时输出：

```sh
p1 x: 2, y: 3
p2 x: 2, y: 3
```

进入到方法中，p1和p2的对象地址传给arg1和arg2。所以arg1和arg2也指向了对应的对象：

![](/images/catgories/java/031/02.jpg)

在tricky()方法中：

```java
arg1.x = 100;    
arg1.y = 100;
```

因此其所指的对象内的值发生了变化（注意这里地址没有改变）。

![](/images/catgories/java/031/03.jpg)

```java
Point temp = arg1;
arg1 = arg2;
arg2 = temp;
```

![](/images/catgories/java/031/04.jpg)

观察上图，这时方法内的输入如下：

```sh
Inside func arg1 x: 2, y: 3
Inside func arg2 x: 100, y: 100
```

跳出方法后：

![](/images/catgories/java/031/05.jpg)

此时p1和p2的输出如下：

```sh
p1 x: 100, y: 100
p2 x: 2, y: 3
```

最后请记住，

**值传递**

**值传递**

**值传递**！

即使传递的对象，也是传递对象的地址(英文就叫reference了)的值！

Happy Coding!

From : [Java 到底是值传递还是引用传递？](https://www.zhihu.com/question/31203609)
