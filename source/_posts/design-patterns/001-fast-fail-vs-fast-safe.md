---
title: Java中Fail-Fast和Fail-Safe
date: 2017-01-05 23:08:48
tags:
categories: "设计模式"
---

### 概念

在概念之前先说下Java中的<b>并行修改（Concurrent Modification）</b>：

>当一个或多个线程在遍历迭代集合对象时，其中的一个线程改变了集合中的数据结构（如添加，删除或者更改集合中的元素），被称之为并行修改。

两者的概念简单点：

"Fail-Fast" : 意味着可能会失败

"Fail-Safe" : 意味着不会失败

<!--more-->

### Fail-Fast和Fail-Safe两种迭代器的不同之处

  * Fail-Fast ：

    Fail-Fast迭代器挡在迭代集合对象时，如果集合对象的数据结构发生修改，当迭代器检测到修改后，将会立即抛出异常，终止操作，而不会冒着危险在未来的某个时候有未确定的行为。

    Fail-Fast在以下的两种情形下会抛出ConcurrentModificationException异常：

      * 单线程环境（Single Threaded Environment）

        在迭代器创建之后，除了迭代器自己的remove方法操作，其他任何时候修改集合结构的任何方法；

      * 多线程环境（Multiple Threaded Environment）

        当一个线程在迭代的时候，另外一个现在正在修改集合的结构时。

    问题点：Fail-Fast迭代器是怎样知道集合内部的结构已发生变化？

      迭代器直接读取内部数据结构，当在迭代集合对象时内部的数据结构应该不被修改。为了保障这个条件，它维护了一个内部标记”mods”，迭代器在每次得到下次元素值时（也就是调用迭代器的hasNext()方法和next()方法）检查“mods”标记，如果集合内部的数据结构被修改的话，”mods“值将会发生变化。然后导致迭代器抛出ConcurrentModificationException异常。

  * Fail-Safe ：

    Fail-Safe迭代器主要时通过拷贝一份原来的集合对象数据元素来遍历集合的。所以任何的修改都是拷贝的集合内部结构上，之前的原集合内部数据结构仍旧不变，故无ConcurrentModificationException异常抛出。

    Fail-Safe会带来两个问题：

      * 复制集合，产生大量的无效对象，开销大

      * 法保证读取的数据是目前原始数据结构中的数据

### 示例

Fail-Fast示例：
```java
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class FailFastExample
{


    public static void main(String[] args)
    {
        Map<String,String> premiumPhone = new HashMap<String,String>();
        premiumPhone.put("Apple", "iPhone");
        premiumPhone.put("HTC", "HTC one");
        premiumPhone.put("Samsung","S5");

        Iterator iterator = premiumPhone.keySet().iterator();

        while (iterator.hasNext())
        {
            System.out.println(premiumPhone.get(iterator.next()));
            premiumPhone.put("Sony", "Xperia Z");
        }

    }

}
```

输出：

```java
iPhone
Exception in thread "main" java.util.ConcurrentModificationException
        at java.util.HashMap$HashIterator.nextEntry(Unknown Source)
        at java.util.HashMap$KeyIterator.next(Unknown Source)
        at FailFastExample.main(FailFastExample.java:20)
```

Fail-Safe示例：

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.Iterator;


public class FailSafeExample
{


    public static void main(String[] args)
    {
        ConcurrentHashMap<String,String> premiumPhone =
                               new ConcurrentHashMap<String,String>();
        premiumPhone.put("Apple", "iPhone");
        premiumPhone.put("HTC", "HTC one");
        premiumPhone.put("Samsung","S5");

        Iterator iterator = premiumPhone.keySet().iterator();

        while (iterator.hasNext())
        {
            System.out.println(premiumPhone.get(iterator.next()));
            premiumPhone.put("Sony", "Xperia Z");
        }

    }

}
```

正常输出：

```java
S5
HTC one
iPhone
```

Fail-Fast和Fail-Safe的比较图，如下：

![](/images/categories/design-patterns/fail-fast-vs-fail-safe.png)

### 参考文献

[英文原文：Fail Fast Vs Fail Safe Iterator In Java : Java Developer Interview Questions](http://javahungry.blogspot.com/2014/04/fail-fast-iterator-vs-fail-safe-iterator-difference-with-example-in-java.html)

[What is fail fast and fail safe in java collection framework?](https://www.quora.com/What-is-fail-fast-and-fail-safe-in-java-collection-framework)

[http://stackoverflow.com/questions/17377407/what-is-fail-safe-fail-fast-iterators-in-java-how-they-are-implemented](http://stackoverflow.com/questions/17377407/what-is-fail-safe-fail-fast-iterators-in-java-how-they-are-implemented)

[Fail Fast vs Fail Safe](http://javapapers.com/core-java/fail-fast-vs-fail-safe/)
