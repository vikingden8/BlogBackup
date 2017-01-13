---
title: Java集合框架学习-Deque
date: 2017-1-13 09:12:28
tags:
categories: "Java集合框架学习"
---

### 概念

Deque是double ended queue的缩写，发音和英文的deck一样。

Deque是从Queue上扩展而来的，其支持在线性集合中可以对两端（队头和队尾）元素的插入和删除。

### JDK中的Deque

首先来看下类的定义：

```java
public interface Deque<E> extends Queue<E> {
  ...
}
```
Deque直接继承自Queue，看下Deque的所有方法：

```java
public interface Deque<E> extends Queue<E> {

    void addFirst(E e);

    void addLast(E e);

    boolean offerFirst(E e);

    boolean offerLast(E e);

    E removeFirst();

    E removeLast();

    E pollFirst();

    E pollLast();

    E getFirst();

    E getLast();

    E peekFirst();

    E peekLast();

    boolean removeFirstOccurrence(Object o);

    boolean removeLastOccurrence(Object o);

    // *** Queue methods ***
    ...


    // *** Stack methods ***

    void push(E e);

    E pop();

    // *** Collection methods ***
    ...

}

```

跟Queue队列一样，Deque中定义的方法也有插入、删除、获取操作，但是，由于是双向的队列，故每一个有两种抽象接口：xxxFirst，xxxLast。

![](/images/categories/data_structure/java_collection_framework/deque-methods.png)

其中，如果将Deque限制为只能从一端入队和出队，则可实现栈的数据结构。对于栈而言，有入栈(push)和出栈(pop)，遵循先进后出原则，所以Deque里面也提供了栈的两个方法push和pop。
