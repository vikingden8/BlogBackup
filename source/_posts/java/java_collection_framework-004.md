---
title: Java集合框架学习-Queue
date: 2017-1-12 10:33:17
tags:
categories: "Java集合框架学习"
---

### 概念

Queue也叫队列，是一种很常见的数据结构， 其是一种先入先出的模型(FIFO)，和我们日常生活中的排队模型很类似。根据不同的实现，主要有数组和链表两种实现形式。如下图：

![](/images/categories/data_structure/java_collection_framework/queue_01.png)


### JDK中Queue

在集合框架中Collection的三个子接口分别是List，Set和Queue，看下JDK中的Queue是怎么定义的。

首先，看下类的定义：

```java
public interface Queue<E> extends Collection<E> {

  ...

}
```

<!--more-->

Queue是一个抽象接口，定义队列的抽象方法，从类的继承来看，其直接继承了Collection接口，拥有Collection的所有抽象方法。

我们再看下具体的抽象方法定义：

```java
public interface Queue<E> extends Collection<E> {

    boolean add(E e);

    boolean offer(E e);

    E remove();

    E poll();

    E element();

    E peek();
}
```

看起来很简单，之定义了六个抽象方法，其实按照分类的话可以分为三组：插入（insert），删除（remove），检查（Examine），如下图：

![](/images/categories/data_structure/java_collection_framework/queue-methods.png)

**插入**：offer方法是尽可能的插入数据，插入成功，返回true，如果不能插入，就返回false；offer方法和Collection的add方法唯一不同就是，add方法在插入失败时再抛出未检测的异常，而offer方法的设计是在插入失败时程序不会异常，保持正常。比如说固定容量的队列，在队列满了之后，继续使用offer方法插入数据，会插入失败，但是不应该出现异常。

**删除**：remove和poll方法都是移除队列的队头元素，并把值返回。remove和poll方法的不同是在处理队列为空的情况下，remove方法会出现异常，而poll方法不会，其会返回null。

**检查**：也叫查找，其两个方法element和peek与remove和pool的不同之处就是，只会返回队头的元素值，而不会删除元素。

Queue的实现一般并不容许插入null，只有LinkedList是一个意外，它容许插入null，但使用者必须注意，不要与poll和peek方法返回的null值混淆。

Queue接口并没有定义并行程序中常用的阻塞方法，也就是说，元素进入Queue之前不必检查Queue中是否有足够的空间，不过作为Queue扩展的java.util.concurrent.BlockingQueue接口定义了这些方法。
