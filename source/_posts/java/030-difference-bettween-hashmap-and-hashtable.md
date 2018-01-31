---
title: HashMap和Hashtable有啥区别
date: 2018-01-31 11:10:02
tags:
categories: "Java学习笔记"
---


在Java 2以前，一般使用Hashtable来映射键值和元素。为了使用Java集合框架，Java对Hashtable进行了重新设计，但是，为了向后兼容保留了所有的方法。Hashtable实现了Map接口，除了Hashtable具有同步功能之外，它与HashMap的用法是一样的。·在使用时一般是用ArrayList代替Vector，LinkedList代替Stack，HashMap代替HashTable，即使在多线程中需要同步，也是用同步包装类。

另外在使用上还有一些小的差异，

<!--more-->

* HashTable的key和value都不允许为null值，而HashMap的key和value则都是允许null值的。这个其实没有好坏之分，只是Sun为了统一Collection的操作特性而改进的。

* HashTable有一个contains(Object value)方法，功能上与containsValue(Object value)一样，但是在实现上花销更大，现在已不推荐使用。而HashMap只有containsValue(Object value)方法。

* HashTable使用Enumeration，HashMap使用Iterator。Iterator其实与Enmeration功能上很相似，只是多了删除的功能。用Iterator不过是在名字上变得更为贴切一些。模式的另外一个很重要的功用，就是能够形成一种交流的语言（或者说文化）。有时候，你说Enumeration大家都不明白，说Iterator就都明白了。

从内部机制实现上的区别如下：

* 1.哈希值的使用不同，Hashtable直接使用对象的hashCode

```java
int hash = key.hashCode();  
int index = (hash & 0x7FFFFFFF) % tab.length;
```

而HashMap重新计算hash值，而且用与代替求模：

```java
int hash = hash(k);  
int i = indexFor(hash, table.length);  

static int hash(Object x) {  
　　int h = x.hashCode();  

　　h += ~(h << 9);  
　　h ^= (h >>> 14);  
　　h += (h << 4);  
　　h ^= (h >>> 10);  
　　return h;  
}  

static int indexFor(int h， int length) {  
　　return h & (length-1);
```

* 2.Hashtable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。

另外，其实java是不推荐使用Hashtable的，见doc:

>If a thread-safe implementation is not needed, it is recommended to use HashMap in place of Hashtable. If a thread-safe highly-concurrent implementation is desired, then it is recommended to use java.util.concurrent.ConcurrentHashMap in place of Hashtable.

>非线程安全情况下，推荐使用HashMap替代Hashtable；高并发线程安全情况下，推荐使用ConcurrentHashMap替代Hashtable。
