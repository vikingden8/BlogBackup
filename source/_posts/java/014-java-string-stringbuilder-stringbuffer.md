---
title: Java中的String、StringBuilder和StringBuffer
date: 2017-04-07 21:02:07
tags:
categories: "Java学习笔记"
---

### 你了解String类吗？

想要了解一个类，最好的办法就是看这个类的实现源代码，String类的实现在

_\jdk1.6.0_14\src\java\lang\String.java_ 文件中。

打开这个类文件就会发现String类是被final修饰的：

![](/images/categories/java/014/01.png)

从上面可以看出几点：

  * String类是final类，也即意味着String类不能被继承，并且它的成员方法都默认为final方法。在Java中，被final修饰的类是不允许被继承的，并且该类中的成员方法都默认为final方法。在早期的JVM实现版本中，被final修饰的方法会被转为内嵌调用以提升执行效率。而从Java SE5/6开始，就渐渐摈弃这种方式了。因此在现在的Java SE版本中，不需要考虑用final去提升方法调用效率。只有在确定不想让该方法被覆盖时，才将方法设置为final。

  * 上面列举出了String类中所有的成员属性，从上面可以看出String类其实是通过char数组来保存字符串的。

<!--more-->

下面再继续看String类的一些方法实现：

![](/images/categories/java/014/02.png)

从上面的三个方法可以看出，无论是sub操、concat还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。

在这里要永远记住一点：

>“对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象”。

在了解了于String类基础的知识后，下面来看一些在平常使用中容易忽略和混淆的地方。

### 深入理解String、StringBuffer、StringBuilder

#### String str="hello world"和String str=new String("hello world")的区别

想必大家对上面2个语句都不陌生，在平时写代码的过程中也经常遇到，那么它们到底有什么区别和联系呢？下面先看几个例子：

![](/images/categories/java/014/03.png)

这段代码的输出结果为：

![](/images/categories/java/014/04.png)

为什么会出现这样的结果？下面解释一下原因：

在前面一篇讲解关于JVM内存机制的一篇博文中提到 ，在class文件中有一部分 来存储编译期间生成的 字面常量以及符号引用，这部分叫做class文件常量池，在运行期间对应着方法区的运行时常量池。

因此在上述代码中，String str1 = "hello world";和String str3 = "hello world"; 都在编译期间生成了 字面常量和符号引用，运行期间字面常量"hello world"被存储在运行时常量池（当然只保存了一份）。通过这种方式来将String对象跟引用绑定的话，JVM执行引擎会先在运行时常量池查找是否存在相同的字面常量，如果存在，则直接将引用指向已经存在的字面常量；否则在运行时常量池开辟一个空间来存储该字面常量，并将引用指向该字面常量。

总所周知，通过new关键字来生成对象是在堆区进行的，而在堆区进行对象生成的过程是不会去检测该对象是否已经存在的。因此通过new来创建对象，创建出的一定是不同的对象，即使字符串的内容是相同的。

#### String、StringBuffer以及StringBuilder的区别

既然在Java中已经存在了String类，那为什么还需要StringBuilder和StringBuffer类呢？

那么看下面这段代码：

![](/images/categories/java/014/05.png)

这句 string += "hello";的过程相当于将原有的string变量指向的对象内容取出与"hello"作字符串相加操作再存进另一个新的String对象当中，再让string变量指向新生成的对象。如果大家还有疑问可以反编译其字节码文件便清楚了：

![](/images/categories/java/014/06.jpg)

从这段反编译出的字节码文件可以很清楚地看出：从第8行开始到第35行是整个循环的执行过程，并且每次循环会new出一个StringBuilder对象，然后进行append操作，最后通过toString方法返回String对象。也就是说这个循环执行完毕new出了10000个对象，试想一下，如果这些对象没有被回收，会造成多大的内存资源浪费。从上面还可以看出：string+="hello"的操作事实上会自动被JVM优化成：

```java
StringBuilder str = new StringBuilder(string);

str.append("hello");

str.toString();
```

再看下面这段代码：

![](/images/categories/java/014/07.png)

反编译字节码文件得到：

![](/images/categories/java/014/08.jpg)

从这里可以明显看出，这段代码的for循环式从13行开始到27行结束，并且new操作只进行了一次，也就是说只生成了一个对象，append操作是在原有对象的基础上进行的。因此在循环了10000次之后，这段代码所占的资源要比上面小得多。
那么有人会问既然有了StringBuilder类，为什么还需要StringBuffer类？查看源代码便一目了然，事实上，StringBuilder和StringBuffer类拥有的成员属性以及成员方法基本相同，区别是StringBuffer类的成员方法前面多了一个关键字：synchronized，不用多说，这个关键字是在多线程访问时起到安全保护作用的,也就是说StringBuffer是线程安全的。

下面摘了2段代码分别来自StringBuffer和StringBuilder，insert方法的具体实现：

![](/images/categories/java/014/09.png)

### 不同场景下三个类的性能测试

从第二节我们已经看出了三个类的区别，这一小节我们来做个小测试，来测试一下三个类的性能区别：

![](/images/categories/java/014/10.png)

上面提到string+="hello"的操作事实上会自动被JVM优化，看下面这段代码：

![](/images/categories/java/014/11.png)

得到验证。

下面对上面的执行结果进行一般性的解释：

  * 对于直接相加字符串，效率很高，因为在编译器便确定了它的值，也就是说形如"I"+"love"+"java"; 的字符串相加，在编译期间便被优化成了"Ilovejava"。这个可以用javap -c命令反编译生成的class文件进行验证。

  对于间接相加（即包含字符串引用），形如s1+s2+s3; 效率要比直接相加低，因为在编译器不会对引用变量进行优化。

  * String、StringBuilder、StringBuffer三者的执行效率：

  StringBuilder > StringBuffer > String

  当然这个是相对的，不一定在所有情况下都是这样。

  比如String str = "hello"+ "world"的效率就比 StringBuilder st  = new StringBuilder().append("hello").append("world")要高。

因此，这三个类是各有利弊，应当根据不同的情况来进行选择使用：

当字符串相加操作或者改动较少的情况下，建议使用 String str="hello"这种形式；

当字符串相加操作较多的情况下，建议使用StringBuilder，如果采用了多线程，则使用StringBuffer。
