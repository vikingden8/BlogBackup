---
title: Java集合框架学习-Collection
date: 2017-01-02 00:32:34
tags:
categories: "Java集合框架学习"
---

现阶段在学习数据结构和算法，学到了线性表，打算做一个Java集合框架的学习笔记，主要是通过学习集合框架JDK的源码，了解其具体的实现，学习其设计模式。

简单的历史回顾，在Java 2之前，Java是没有完整的集合框架的。它只有一些简单的可以自扩展的容器类，比如Vector，Stack，Hashtable等。这些容器类在使用的过程中由于效率问题饱受诟病，因此在Java 2中，Java设计者们进行了大刀阔斧的整改，重新设计，于是就有了现在的集合框架。需要注意的是，之前的那些容器类库并没有被弃用而是进行了保留，主要是为了向下兼容的目的，但我们在平时使用中还是应该尽量少用。

下面是JDK中集合框架图：

![](/images/categories/data_structure/java_collection_framework/collection-framework-hierarchy.jpg)  

从上面的集合框架图可以看到，Java集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合；另一种是图（Map），存储键/值对映射。Collection接口又有3种子类型，List、Set和Queue，再下面是一些抽象类，最后是具体实现类，常用的有ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap、LinkedHashMap等等。

<!--more-->

本文主要是学习Java集合框架中的Collection接口类。

### Iterable(可迭代)

在查看java.util.Collection.java的源码时，第一行类的定义就是下面的代码：

```java
public interface Collection<E> extends Iterable<E> {
    ...
}
```

可以看到Collection接口继承自Iterable接口，我们先看看Iterable中定义了什么：

```java
public interface Iterable<T> {

    /**
     * Returns an iterator over a set of elements of type T.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
}
```

Iterable接口是一个泛型接口，里面就只有一个iterator的接口方法，返回一个Iterator的类对象，根据doc的描述，Iterable接口主要是为了实现类能使用“for-each”语句（Implementing this interface allows an object to be the target of the "foreach" statement.）

Iterator那又是什么呢？

进入Iterator的源码，根据类的注释，其实Iterator就是之前Enumeration的一个替代，相对于Enumeration，其主要有以下的两个改进：

  * Iterator允许调用者在迭代具体集合时删除对象；

  * 方法命名的改进（比如之前Enumeration中hasMoreElements，nextElement方法名，在Iterator分别改进成了hasNext和next，明显的缩短了方法名）。

Iterator类其实就只有三个抽象的方法，如下：  
```java
public interface Iterator<E> {
    /**
     * Returns {@code true} if the iteration has more elements.
     * (In other words, returns {@code true} if {@link #next} would
     * return an element rather than throwing an exception.)
     *
     * @return {@code true} if the iteration has more elements
     */
    boolean hasNext();

    /**
     * Returns the next element in the iteration.
     *
     * @return the next element in the iteration
     * @throws NoSuchElementException if the iteration has no more elements
     */
    E next();

    /**
     * Removes from the underlying collection the last element returned
     * by this iterator (optional operation).  This method can be called
     * only once per call to {@link #next}.  The behavior of an iterator
     * is unspecified if the underlying collection is modified while the
     * iteration is in progress in any way other than by calling this
     * method.
     *
     * @throws UnsupportedOperationException if the {@code remove}
     *         operation is not supported by this iterator
     *
     * @throws IllegalStateException if the {@code next} method has not
     *         yet been called, or the {@code remove} method has already
     *         been called after the last call to the {@code next}
     *         method
     */
    void remove();
}
```

  * hasNext(): 迭代器是否还有元素，也就是说下面的next方法返回一个元素对象而不是抛出一个异常；  

  * next(): 返回迭代器中的下一个元素，如果迭代器中无下一个元素，将会抛出一个NoSuchElementException异常；

  * remove()：删除当前迭代的对象，如果在调用此方法时，没有调用过依次next方法，将会抛出IllegalStateException异常。  

#### 总结下Iterator和Iterable:

Iterable接口实现后的功能是“生成”一个迭代器，而Iterator接口实现后的功能是“操作”一个迭代器。

#### 为什么容器中的类都实现了Iterable接口而非Iterator接口？

因为Iterator接口的核心方法next()或者hasNext() 是依赖于迭代器的当前迭代位置的。如果Collection直接实现Iterator接口，势必导致集合对象中包含当前迭代位置的数据(指针)。 当集合在不同方法间被传递时，由于当前迭代位置不可预置，那么next()方法的结果会变成不可预知。  

而Iterable则不然，每次调用都会返回一个从头开始计数的迭代器。  多个迭代器是互不干扰的。  

### Collection（集合）接口


Collection接口是处理对象集合的根接口，其中定义了很多对元素进行操作的方法，AbstractCollection是提供Collection部分实现的抽象类。下图展示了Collection接口中的全部方法。

![](/images/categories/data_structure/java_collection_framework/collection-methods.png)  

从上面的方法图可以看到，其主要包含集合的基本操作有：添加、删除、清空、遍历、是否为空、获取大小等。

通过查看源码，Collection类主要定义以下四个方面的顶级抽象功能点：

  * Query Operations（查询操作）：比如size()，isEmpty()，contains()等；

  * Modification Operations（修改操作）：如add()，remove()；

  * Bulk Operations（批量操作）：如containsAll()，addAll()，removeAll()，retainAll()，clear()等；

  * Comparison and hashing（比较和Hash）：equals()和hashCode()。

关于具体每一个方法具体代表啥抽象内容，大部分都能从方法名猜到，也可以看下器的doc描述[Collection Collection (Java Platform SE 8 )](http://docs.oracle.com/javase/8/docs/api/java/util/Collection.html)。


### 参考文献

[Java Collections Tutorial](http://tutorials.jenkov.com/java-collections/index.html)  

[Java Collections - Iterable](http://tutorials.jenkov.com/java-collections/iterable.html)  

[Java Generics - Implementing the Iterable Interface](http://tutorials.jenkov.com/java-generics/implementing-iterable.html)  

[Lesson: Introduction to Collections](http://docs.oracle.com/javase/tutorial/collections/intro/index.html)  

[What is the difference between iterator and iterable and how to use them?](http://stackoverflow.com/questions/6863182/what-is-the-difference-between-iterator-and-iterable-and-how-to-use-them)  
