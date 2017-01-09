---
title: Iterator Design Pattern-迭代器设计模式
date: 2017-01-09 23:33:28
tags:
categories: "设计模式"
---

### 定义

提供一种方法顺序访问访问一个聚合（也就是Java中常见的List，Set，Map）对象中的各个元素，而又不暴露该对象内部表示（The Iterator
Pattern lets you sequentially move through a collection of objects using a standard interface, and without having
to know the internal representation of that collection.）。

### 结构和说明

![迭代器结构图](/images/categories/design-patterns/iterator_diagram.png)

* Iterator : 迭代器接口，定义访问和遍历元素的接口；

* ConcreteIterator ： 具体的迭代器实现对象，实现对聚合对象的遍历，并跟踪遍历时的当前位置；

* Aggregate ： 聚合对象，定义创建响应迭代器对象的接口；

* ConcreteAggregate ： 具体的聚合对象，实现创建相应的迭代器对象。

<!--more-->

### GoF迭代器模式

原始的Gof迭代器模式提供了一个接口，像以下的代码：

```java
public interface Iterator
{
  public Object first();
  public Object next();
  public Object currentItem();
  public boolean isDone();
}
```

正如这些方法所示，first表示移动到第一个元素，next表示移动到下一个元素，currentItem表示获取当前指向的对象元素，isDone表示判断集合中是否还有元素。

### Java中的Enumeration接口

在之前旧的JDK中已有实现迭代器模式，在Enumeration接口中：

```java
public interface Enumeration
{
  public boolean hasMoreElements();
  public Object nextElement();
}
```

正如你看到的，Enumeration接口没有完全遵循GoF的迭代器模式。我们无需去执行first方法，而且用harMoreElements的正面判断比isDone的负面判断要好很多：

```java
while (hasMoreElements()) {
  // work done here
}
```

相比于下面的判断逻辑要好很多：

```java
while (!isDone()) {
  // work done here
}
``

### Java 迭代器接口

在Java 1.2中提供了迭代器接口，如下：

```java
public interface Iterator
{
  public boolean hasNext();
  public Object next();
  public void remove();
}
```

正如JDK中的doc描述，新的Iterator接口相对于Enumeration接口有以下两点的不同：

    * 新添加了一个remove方法，可以在迭代的时候删除当前需处理的元素，这个解决了在遍历集合的时候常出现的ConcurrentModificationException异常的出现；

    * 优化方法的命名

### Java中使用迭代器模式

定义一个集合对象：

```java
List<Person> people = new ArrayList()<Person>;
```

然后我们遍历集合对象，可以使用迭代器：

```java
while (people.hasNext())
{
  // no need to cast the Object to Person
  Person person = people.next();
  // do something with that person here ...
}
```

注意：如果这里是Java 1.5之前，则需要在next方法调用后把对象类型转换，Java 1.5以上由于有泛型，则完全不需要了，更加的简单。

其实在JDK提供了更简单的for-each方法来遍历集合对象：

```java
for (Person person : people)
{
  // do something with 'person' here ...
}
```

### 总结

优点：

    * 封装容器的内部实现细节，简化客户端的访问和获取容器内数据；

    * 客户端可以通过相同的方式遍历不同的容器对象；

    * 我们可以根据客户端需求让容器实现不同的迭代算法，从而可以用不同的遍历方式来访问容器数据。
