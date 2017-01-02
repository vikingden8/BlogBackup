---
title: Java中的字符串常量池
date: 2017-01-02 13:56:46
tags:
categories: "Java学习笔记"
---

Java中字符串对象创建有两种形式，一种为字面量形式，如String str = "vikingden";，另一种就是使用new这种标准的构造对象的方法，如String str = new String("vikingden");，这两种方式我们在代码编写时都经常使用，尤其是字面量的方式。然而这两种实现其实存在着一些性能和内存占用的差别。这一切都是源于JVM为了减少字符串对象的重复创建，其维护了一个特殊的内存，这段内存被成为字符串常量池或者字符串字面量池。

### 工作原理

当代码中出现字面量形式创建字符串对象时，JVM首先会对这个字面量进行检查，如果字符串常量池中存在相同内容的字符串对象的引用，则将这个引用返回，否则新的字符串对象被创建，然后将这个引用放入字符串常量池，并返回该引用。

<!--more-->

### 举例说明

#### 字面量创建形式

```java
String str1 = "vikingden";
```

JVM检测这个字面量，这里我们认为没有内容为vikingden的对象存在。JVM通过字符串常量池查找不到内容为vikingden的字符串对象存在，那么会创建这个字符串对象，然后将刚创建的对象的引用放入到字符串常量池中,并且将引用返回给变量str1。

如果接下来有这样一段代码:

```java
String str2 = "vikingden";
```

同样JVM还是要检测这个字面量，JVM通过查找字符串常量池，发现内容为”droid”字符串对象存在，于是将已经存在的字符串对象的引用返回给变量str2。注意这里不会重新创建新的字符串对象。

验证是否为str1和str2是否指向同一对象，我们可以通过这段代码：

```java
System.out.println(str1 == str2);
```

结果为true。

#### 使用new创建

```java
String str3 = new String("vikingden");
```

当我们使用了new来构造字符串对象的时候，不管字符串常量池中有没有相同内容的对象的引用，新的字符串对象都会创建。因此我们使用下面代码测试一下:

```java
String str3 = new String("vikingden");
System.out.println(str1 == str3);
```

结果如我们所想，为false，表明这两个变量指向的为不同的对象。

#### intern

对于上面使用new创建的字符串对象，如果想将这个对象的引用加入到字符串常量池，可以使用intern方法。

调用intern后，首先检查字符串常量池中是否有该对象的引用，如果存在，则将这个引用返回给变量，否则将引用加入并返回给变量。

```java
String str4 = str3.intern();
System.out.println(str4 == str1);
```

输出的结果为true。

转载自：[Java中的字符串常量池](http://droidyue.com/blog/2014/12/21/string-literal-pool-in-java/)
