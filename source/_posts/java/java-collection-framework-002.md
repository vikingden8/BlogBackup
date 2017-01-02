---
title: Java集合框架学习-List
date: 2017-01-02 20:22:23
tags:
categories: "Java集合框架学习"
---

本篇文章来学习JDK集合框架中的List接口源码。  

Collection有两个直接的子接口List和Set，List允许其集合中有重复的元素，并且存放的顺序与插入时的顺序一致。

```java
public interface List<E> extends Collection<E> {
    ...
}
```

从上面的源码可以看出List是继承自Collection接口的，所以List也继承了Collection里面的所有抽象接口方法。但是我们在看源码的时候，发现大部分Collection接口中有的方法，在List接口中又重复定义了，这是为啥？可能最大的原因是在方法的doc上了吧，如果有人知道是其他什么原因，可以告知。

<!--more-->

但是在List接口中也增添了好几类新的抽象接口方法：

  * Positional Access Operations（位置访问操作）：如get()，set()，add()，remove()；  

  * Search Operations（搜索操作）：indexOf()，lastIndexOf()；

  * List Iterators（列表迭代器）：主要是listIterator方法，有两个重载的方法；

  * View（）：方法subList(int fromIndex, int toIndex)，返回子列表。

```java
public interface List<E> extends Collection<E> {
    //common methods in Collection
    ...
    // Positional Access Operations

    E get(int index);

    E set(int index, E element);

    void add(int index, E element);

    E remove(int index);

    // Search Operations

    int indexOf(Object o);

    int lastIndexOf(Object o);

    // List Iterators

    ListIterator<E> listIterator();


    ListIterator<E> listIterator(int index);

    // View

    List<E> subList(int fromIndex, int toIndex);
}
```

### ListIterator  

在List接口中，我们看到有其中的listIterator方法有返回ListIterator对象，进入源码查看：

```java
public interface ListIterator<E> extends Iterator<E> {
    // Query Operations

    boolean hasNext();

    E next();

    boolean hasPrevious();

    E previous();

    int nextIndex();

    int previousIndex();


    // Modification Operations

    void remove();

    void set(E e);

    void add(E e);
}
```

ListIterator同样继承自Iterator接口，除了Iterator原本的三个抽象接口方法外，ListIterator还提供了返回当前列表上一状态信息，比如hasPrevious，previous。还有就是索引值方法nextIndex，previousIndex。set和add方法分别用来修改特定位置的元素值和添加给定的元素值到指定的位置。

>有一点需要明确的时候，迭代器指向的位置是元素之前的位置

这里假设集合List由四个元素List1、List2、List3和List4组成，当使用语句Iterator it = List.Iterator()时，迭代器it指向的位置是上图中Iterator1指向的位置，当执行语句it.next()之后，迭代器指向的位置后移到上图Iterator2所指向的位置，如下图：

![](/images/categories/data_structure/java_collection_framework/iterator_position.png)  

### Iterator和ListIterator有什么不同

  * 使用范围不同，Iterator可以应用于所有的集合，Set、List和Map和这些集合的子类型。而ListIterator只能用于List及其子类型。

  * ListIterator有set，add方法，可以向List中修改，添加对象，而Iterator不能；

  * ListIterator和Iterator都有hasNext()和next()方法，可以实现顺序向后遍历，但是ListIterator有hasPrevious()和previous()方法，可以实现逆向（顺序向前）遍历。Iterator不可以。

  * ListIterator可以定位当前索引的位置，nextIndex()和previousIndex()可以实现。Iterator没有此功能。

  * 都可实现删除操作，但是ListIterator可以实现对象的修改，set()方法可以实现。Iterator仅能遍历，不能修改。

### 总结

List接口的子类很多，在实际的开发过程中我们用得比较多的比如ArrayList，LinkedList，Stack，Vector等，下面的章节我们将依次看下他们的具体实现，以及他们之间的差别。List的继承可以看下图：

![](/images/categories/data_structure/java_collection_framework/list_class_overview.png)
