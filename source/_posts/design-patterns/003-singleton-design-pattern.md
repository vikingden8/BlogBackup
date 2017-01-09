---
title: Singleton Design Pattern-单例设计模式
date: 2017-01-09 23:34:25
tags:
categories: "设计模式"
---

### 概念

单例模式，又称单件模式或者单子模式，指的是一个类只有一个实例，并且提供一个全局访问点。

> Ensure a class has only one instance, and provide a global point of access to it.

单例模式确保某个类只有一个实例，而且自行实例化并向整个系统提供这个实例。在计算机系统中，
线程池、缓存、日志对象、对话框、打印机、显卡的驱动程序对象常被设计成单例。这些应用都或
多或少具有资源管理器的功能。每台计算机可以有若干个打印机，但只能有一个Printer Spooler，
以避免两个打印作业同时输出到打印机中。每台计算机可以有若干通信端口，系统应当集中管理这
些通信端口，以避免一个通信端口同时被两个请求同时调用。

### 实现思路

* 在单例的类中设置一个private静态变量sInstance，sInstance类型为当前类，用来持有单例唯一的实例。  

* 将（无参数）构造器设置为private，避免外部使用new构造多个实例。  

* 提供一个public的静态方法，如getInstance，用来返回该类的唯一实例sInstance。  

其中上面的单例的实例可以有以下几种创建形式，每一种实现都需要保证实例的唯一性。

<!--more-->

### 实现方式

#### 饿汉式

饿汉式指的是单例的实例在类装载时进行创建。如果单例类的构造方法中没有包含过多的操作处理，饿汉式其实是可以接受的。

饿汉式的常见代码如下,当SingleInstance类加载时会执行private static SingleInstance sInstance = new SingleInstance();
初始化了唯一的实例，然后getInstance()直接返回sInstance即可。  

```java
public class SingleInstance {
  private static SingleInstance sInstance = new SingleInstance();

  private SingleInstance() {
  }

  public static SingleInstance getInstance() {
      return sInstance;
  }
}
```

饿汉式的问题

* 如果构造方法中存在过多的处理，会导致加载这个类时比较慢，可能引起性能问题。  

* 如果使用饿汉式的话，只进行了类的装载，并没有实质的调用，会造成资源的浪费。

#### 懒汉式

懒汉式指的是单例实例在第一次使用时进行创建。这种情况下避免了上面饿汉式可能遇到的问题。

但是考虑到多线程的并发操作，我们不能简简单单得像下面代码实现。

```java
public class SingleInstance {
  private static SingleInstance sInstance;
  private SingleInstance() {
  }

  public static SingleInstance getInstance() {
      if (null == sInstance) {
          sInstance = new SingleInstance();
      }
      return sInstance;
  }
}
```

上述的代码在多个线程密集调用getInstance时，存在创建多个实例的可能。比如线程A进入null == sInstance这段代码块，
而在A线程未创建完成实例时，如果线程B也进入了该代码块，必然会造成两个实例的产生。  

##### synchronized修饰方法

使用synchrnozed修饰getInstance方法可能是最简单的一个保证多线程保证单例唯一性的方法。  

synchronized修饰的方法后，当某个线程进入调用这个方法，该线程只有当其他线程离开当前方法后才会进入该方法。所以可以
保证getInstance在任何时候只有一个线程进入。  

```java
public class SingleInstance {
  private static SingleInstance sInstance;
  private SingleInstance() {
  }

  public static synchronized SingleInstance getInstance() {
      if (null == sInstance) {
          sInstance = new SingleInstance();
      }
      return sInstance;
  }
}
```

但是使用synchronized修饰getInstance方法后必然会导致性能下降，而且getInstance是一个被频繁调用的方法。
虽然这种方法能解决问题，但是不推荐。

##### 双重检查加锁

使用双重检查加锁，首先进入该方法时进行null == sInstance检查，如果第一次检查通过，即没有实例创建，则进
入synchronized控制的同步块,并再次检查实例是否创建，如果仍未创建，则创建该实例。

双重检查加锁保证了多线程下只创建一个实例，并且加锁代码块只在实例创建的之前进行同步。如果实例已经创建后，进
入该方法，则不会执行到同步块的代码。  

```java
public class SingleInstance {
  private static volatile SingleInstance sInstance;
  private SingleInstance() {
  }

  public static SingleInstance getInstance() {
      if (null == sInstance) {
          synchronized (SingleInstance.class) {
              if (null == sInstance) {
                  sInstance = new SingleInstance();
              }
          }
      }
      return sInstance;
  }
}
```

##### volatile是什么

Volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，
。使用volatile修饰sInstance变量之后，可以确保多个线程之间正确处理sInstance变量。  

关于volatile，可以访问[深入分析Volatile的实现原理](http://www.infoq.com/cn/articles/ftf-java-volatile)了解更多。

#### 利用static机制

在Java中，类的静态初始化会在类被加载时触发，我们利用这个原理，可以实现利用这一特性，结合内部类，可以实现如下的代码，
进行懒汉式创建实例。

```java
public class SingleInstance {
  private SingleInstance() {
  }

  public static SingleInstance getInstance() {
      return SingleInstanceHolder.sInstance;
  }

  private static class SingleInstanceHolder {
      private static SingleInstance sInstance = new SingleInstance();
  }
}
```

关于这种机制，可以具体了解[双重检查锁定与延迟初始化]
(http://www.infoq.com/cn/articles/double-checked-locking-with-delay-initialization)。  

#### 枚举

这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象，
可谓是很坚强的壁垒啊，不过，个人认为由于1.5中才加入enum特性，用这种方式写不免让人感觉生疏，在实际工作中，我也很少看见有人这么写过。

代码示例如下：  

```java

public enum Singleton {  
     INSTANCE;  
     public void whateverMethod() {  
     }  
}  
```

#### 真的只有一个对象么

其实，单例模式并不能保证实例的唯一性，只要我们想办法的话，还是可以打破这种唯一性的。以下几种方法都能实现。

* 使用反射，虽然构造器为非公开，但是在反射面前就不起作用了。

* 如果单例的类实现了cloneable，那么还是可以拷贝出多个实例的。

* Java中的对象序列化也有可能导致创建多个实例。避免使用readObject方法。

* 使用多个类加载器加载单例类，也会导致创建多个实例并存的问题。

#### 参考资料

原文地址(有删减，有添加)：[单例这种设计模式](http://droidyue.com/blog/2015/01/11/looking-into-singleton/)

[双重检查锁定与延迟初始化](http://www.infoq.com/cn/articles/double-checked-locking-with-delay-initialization)

[Java：单例模式的七种写法](http://www.blogjava.net/kenzhh/archive/2013/03/15/357824.html)

[Java Singleton Design Pattern Best Practices with Examples]
(http://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples)

[Java Inner Class](http://www.journaldev.com/996/java-inner-class)

[Thread Safety in Java Singleton Classes with Example Code](http://www.journaldev.com/171/thread-safety-in-java-singleton-classes-with-example-code)
