---
title: Java中的for-each解析
date: 2017-1-6 15:49:06
tags:
categories: "Java学习笔记"
---

语法糖是一种几乎每种语言或多或少都提供过的一些方便程序员开发代码的语法，它只是编译器实现的一些小把戏罢了，编译期间以特定的字节码或者特定的方式对这些语法做一些处理，开发者就可以直接方便地使用了。这些语法糖虽然不会提供实质性的功能改进，但是它们或能提高性能、或能提升语法的严谨性、或能减少编码出错的机会。Java提供给了用户大量的语法糖，比如泛型、自动装箱、自动拆箱、foreach循环、变长参数、内部类、枚举类、断言（assert）等。

本篇主要是讲解foreach,foreach的语法经过编译之后解析成什么呢？

首先来看一个例子：

```java
import java.util.ArrayList;
import java.util.List;

public class ForeachTest
{
    public static void main(String[] args)
    {
        List<String> list = new ArrayList<>();
        list.add("s1");
        list.add("s2");

        for(String s : list)
        {
            System.out.println(s);
        }
    }
}
```

对这个类进行反编译：

```java
javac ForeachTest.java
javap -verbose ForeachTest
```
<!--more-->

结果如下所示：

```java
autotest@autotest-PowerEdge-R420:~/viking/java$ javap -verbose ForeachTest.class
Classfile /home/autotest/viking/java/ForeachTest.class
  Last modified Jan 6, 2017; size 790 bytes
  MD5 checksum fc4d49e0ad84e5d8677762a139f9f1a0
  Compiled from "ForeachTest.java"
public class ForeachTest
  SourceFile: "ForeachTest.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #14.#26        //  java/lang/Object."<init>":()V
   #2 = Class              #27            //  java/util/ArrayList
   #3 = Methodref          #2.#26         //  java/util/ArrayList."<init>":()V
   #4 = String             #28            //  s1
   #5 = InterfaceMethodref #29.#30        //  java/util/List.add:(Ljava/lang/Object;)Z
   #6 = String             #31            //  s2
   #7 = InterfaceMethodref #29.#32        //  java/util/List.iterator:()Ljava/util/Iterator;
   #8 = InterfaceMethodref #33.#34        //  java/util/Iterator.hasNext:()Z
   #9 = InterfaceMethodref #33.#35        //  java/util/Iterator.next:()Ljava/lang/Object;
  #10 = Class              #36            //  java/lang/String
  #11 = Fieldref           #37.#38        //  java/lang/System.out:Ljava/io/PrintStream;
  #12 = Methodref          #39.#40        //  java/io/PrintStream.println:(Ljava/lang/String;)V
  #13 = Class              #41            //  ForeachTest
  #14 = Class              #42            //  java/lang/Object
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LineNumberTable
  #19 = Utf8               main
  #20 = Utf8               ([Ljava/lang/String;)V
  #21 = Utf8               StackMapTable
  #22 = Class              #43            //  java/util/List
  #23 = Class              #44            //  java/util/Iterator
  #24 = Utf8               SourceFile
  #25 = Utf8               ForeachTest.java
  #26 = NameAndType        #15:#16        //  "<init>":()V
  #27 = Utf8               java/util/ArrayList
  #28 = Utf8               s1
  #29 = Class              #43            //  java/util/List
  #30 = NameAndType        #45:#46        //  add:(Ljava/lang/Object;)Z
  #31 = Utf8               s2
  #32 = NameAndType        #47:#48        //  iterator:()Ljava/util/Iterator;
  #33 = Class              #44            //  java/util/Iterator
  #34 = NameAndType        #49:#50        //  hasNext:()Z
  #35 = NameAndType        #51:#52        //  next:()Ljava/lang/Object;
  #36 = Utf8               java/lang/String
  #37 = Class              #53            //  java/lang/System
  #38 = NameAndType        #54:#55        //  out:Ljava/io/PrintStream;
  #39 = Class              #56            //  java/io/PrintStream
  #40 = NameAndType        #57:#58        //  println:(Ljava/lang/String;)V
  #41 = Utf8               ForeachTest
  #42 = Utf8               java/lang/Object
  #43 = Utf8               java/util/List
  #44 = Utf8               java/util/Iterator
  #45 = Utf8               add
  #46 = Utf8               (Ljava/lang/Object;)Z
  #47 = Utf8               iterator
  #48 = Utf8               ()Ljava/util/Iterator;
  #49 = Utf8               hasNext
  #50 = Utf8               ()Z
  #51 = Utf8               next
  #52 = Utf8               ()Ljava/lang/Object;
  #53 = Utf8               java/lang/System
  #54 = Utf8               out
  #55 = Utf8               Ljava/io/PrintStream;
  #56 = Utf8               java/io/PrintStream
  #57 = Utf8               println
  #58 = Utf8               (Ljava/lang/String;)V
{
  public ForeachTest();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 4: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: new           #2                  // class java/util/ArrayList
         3: dup           
         4: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
         7: astore_1      
         8: aload_1       
         9: ldc           #4                  // String s1
        11: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
        16: pop           
        17: aload_1       
        18: ldc           #6                  // String s2
        20: invokeinterface #5,  2            // InterfaceMethod java/util/List.add:(Ljava/lang/Object;)Z
        25: pop           
        26: aload_1       
        27: invokeinterface #7,  1            // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
        32: astore_2      
        33: aload_2       
        34: invokeinterface #8,  1            // InterfaceMethod java/util/Iterator.hasNext:()Z
        39: ifeq          62
        42: aload_2       
        43: invokeinterface #9,  1            // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
        48: checkcast     #10                 // class java/lang/String
        51: astore_3      
        52: getstatic     #11                 // Field java/lang/System.out:Ljava/io/PrintStream;
        55: aload_3       
        56: invokevirtual #12                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        59: goto          33
        62: return        
      LineNumberTable:
        line 8: 0
        line 9: 8
        line 10: 17
        line 12: 26
        line 14: 52
        line 15: 59
        line 16: 62
      StackMapTable: number_of_entries = 2
           frame_type = 253 /* append */
             offset_delta = 33
        locals = [ class java/util/List, class java/util/Iterator ]
           frame_type = 250 /* chop */
          offset_delta = 28

}
```

反编译出来的内容很多，看不懂也没关系，关键看到Iterator这个标志，其实在对有实现Iterable接口的对象采用foreach语法糖的话，编译器会将这个for关键字转化为对目标的迭代器使用。

上面的代码会被转化为如下的代码：

```java
import java.util.ArrayList;
import java.util.List;

public class ForeachTest
{
    //编译器会对没有显示构造函数的类添加一个默认的构造函数，这个过程是在"语义分析"阶段完成的
    public ForeachTest()
    {
        super();
    }

    public static void main(String[] args)
    {
        List<String> list = new ArrayList<>();
        list.add("s1");
        list.add("s2");

        for(java.util.Iterator i$ = list.iterator(); i$.hasNext();)
        {
            String s = (String) i$.next();
            {
                System.out.println(s);
            }
        }
    }
}
```

>注：如果要想使自己自定义的类可以采用foreach语法糖就必须实现Iterable接口。

细心的朋友可能会注意到，java中的数组也可以采用foreach的语法糖，但是数组并没有实现Iterable接口呀。

下面再举一个例子：

```java
public class ForeachTest2
{
    public static void main(String[] args)
    {
        String arr[] = {"s1","s2"};

        for(String s:arr)
        {
            System.out.println(s);
        }
    }
}
```

 反编译结果：

```java
autotest@autotest-PowerEdge-R420:~/viking/java$ javap -verbose ForeachTest2.class
Classfile /home/autotest/viking/java/ForeachTest2.class
  Last modified Jan 6, 2017; size 573 bytes
  MD5 checksum 3ec1bd01b8d93b101f0fbb658a30c6cf
  Compiled from "ForeachTest2.java"
public class ForeachTest2
  SourceFile: "ForeachTest2.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #8.#19         //  java/lang/Object."<init>":()V
   #2 = Class              #20            //  java/lang/String
   #3 = String             #21            //  s1
   #4 = String             #22            //  s2
   #5 = Fieldref           #23.#24        //  java/lang/System.out:Ljava/io/PrintStream;
   #6 = Methodref          #25.#26        //  java/io/PrintStream.println:(Ljava/lang/String;)V
   #7 = Class              #27            //  ForeachTest2
   #8 = Class              #28            //  java/lang/Object
   #9 = Utf8               <init>
  #10 = Utf8               ()V
  #11 = Utf8               Code
  #12 = Utf8               LineNumberTable
  #13 = Utf8               main
  #14 = Utf8               ([Ljava/lang/String;)V
  #15 = Utf8               StackMapTable
  #16 = Class              #29            //  "[Ljava/lang/String;"
  #17 = Utf8               SourceFile
  #18 = Utf8               ForeachTest2.java
  #19 = NameAndType        #9:#10         //  "<init>":()V
  #20 = Utf8               java/lang/String
  #21 = Utf8               s1
  #22 = Utf8               s2
  #23 = Class              #30            //  java/lang/System
  #24 = NameAndType        #31:#32        //  out:Ljava/io/PrintStream;
  #25 = Class              #33            //  java/io/PrintStream
  #26 = NameAndType        #34:#35        //  println:(Ljava/lang/String;)V
  #27 = Utf8               ForeachTest2
  #28 = Utf8               java/lang/Object
  #29 = Utf8               [Ljava/lang/String;
  #30 = Utf8               java/lang/System
  #31 = Utf8               out
  #32 = Utf8               Ljava/io/PrintStream;
  #33 = Utf8               java/io/PrintStream
  #34 = Utf8               println
  #35 = Utf8               (Ljava/lang/String;)V
{
  public ForeachTest2();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0       
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return        
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=4, locals=6, args_size=1
         0: iconst_2      
         1: anewarray     #2                  // class java/lang/String
         4: dup           
         5: iconst_0      
         6: ldc           #3                  // String s1
         8: aastore       
         9: dup           
        10: iconst_1      
        11: ldc           #4                  // String s2
        13: aastore       
        14: astore_1      
        15: aload_1       
        16: astore_2      
        17: aload_2       
        18: arraylength   
        19: istore_3      
        20: iconst_0      
        21: istore        4
        23: iload         4
        25: iload_3       
        26: if_icmpge     49
        29: aload_2       
        30: iload         4
        32: aaload        
        33: astore        5
        35: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
        38: aload         5
        40: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        43: iinc          4, 1
        46: goto          23
        49: return        
      LineNumberTable:
        line 5: 0
        line 7: 15
        line 9: 35
        line 7: 43
        line 11: 49
      StackMapTable: number_of_entries = 2
           frame_type = 255 /* full_frame */
          offset_delta = 23
          locals = [ class "[Ljava/lang/String;", class "[Ljava/lang/String;", class "[Ljava/lang/String;", int, int ]
          stack = []
           frame_type = 248 /* chop */
          offset_delta = 25

}
```

看不懂？同样没关系~这里可以注意到数组的foreach语法糖并没有采用Iterable实现转换。如上面的信息只是涉及一些压栈出站的内容。

真正的解析结果如下所示：

```java
public class ForeachTest2
{
    public ForeachTest2()
    {
        super();
    }

    public static void main(String[] args)
    {
        int arr[] = {1,2};
        int [] arr$ = arr;
        for(int len$ = arr$.length, i$ = 0; i$<len$; ++i$ )
        {
            int i = arr$[i$];
            {
                System.out.println(i);
            }
        }
    }
}
```

可以看到对于数组而言，其实就是转换为普通的遍历而已；

关于foreach语法糖的信息就这样结束了嚒？ 显然没有！

对于实现RandomAccess接口的集合比如ArrayList，应当使用最普通的for循环而不是foreach循环来遍历，所以第一个例子中有欠妥之处。

首先看一下JDK1.7中对RandomAccess接口的定义：  

![](/images/categories/java/java_for_each.png)

看不懂英文也没关系，我来解释一下最后一句加粗的话：实际经验表明，实现RandomAccess接口的类实例，假如是随机访问的，使用普通for循环效率将高于使用foreach循环；反过来，如果是顺序访问的，则使用Iterator会效率更高。

可以使用类似如下的代码作判断：

```java
if (list instanceof RandomAccess){
    for (int i = 0; i < list.size(); i++){
      ...
    }
}else{
    Iterator<?> iterator = list.iterable();
    while (iterator.hasNext()){
      Object obj = iterator.next();
      ...
    }
}
```

>其实如果看过ArrayList源码的同学也可以注意到：ArrayList底层是采用数组实现的，如果采用Iterator遍历，那么还要创建许多指针去执行这些值（比如next();hasNext()）等，这样必然会增加内存开销以及执行效率。
