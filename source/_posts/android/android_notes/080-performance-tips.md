---
title: Performance Tips
date: 2018-01-24 16:55:50
tags:
categories: "Android学习笔记"
---

This document primarily covers micro-optimizations that can improve overall app performance when combined, but it's unlikely that these changes will result in dramatic performance effects. Choosing the right algorithms and data structures should always be your priority, but is outside the scope of this document. You should use the tips in this document as general coding practices that you can incorporate into your habits for general code efficiency.

这篇文章主要是介绍了一些小细节的优化技巧，当这些小技巧综合使用起来的时候，对于整个App的性能提升还是有作用的，只是不能较大幅度的提升性能而已。选择合适的算法与数据结构才应该是你首要考虑的因素，在这篇文章中不会涉及这方面。你应该使用这篇文章中的小技巧作为平时写代码的习惯，这样能够提升代码的效率。

There are two basic rules for writing efficient code:

通常来说，高效的代码需要满足下面两个规则：

* Don't do work that you don't need to do.不要做冗余工作

* Don't allocate memory if you can avoid it.如果能避免，尽量不要分配内存

<!--more-->

One of the trickiest problems you'll face when micro-optimizing an Android app is that your app is certain to be running on multiple types of hardware. Different versions of the VM running on different processors running at different speeds. It's not even generally the case that you can simply say "device X is a factor F faster/slower than device Y", and scale your results from one device to others. In particular, measurement on the emulator tells you very little about performance on any device. There are also huge differences between devices with and without a JIT: the best code for a device with a JIT is not always the best code for a device without.

你会面临最棘手的一个问题是当你优化一个肯定会在多种类型的硬件上运行的应用程序。不同版本的VM在不同的处理器上运行速度不同。它甚至不是你可以简单地说“设备X因为F原因比设备Y快/慢”那么简单,而且也不能简单地从一个设备拓展到另一个设备。特别提醒的是模拟器在性能方面和其他的设备没有可比性。通常有JIT优化和没有JIT优化的设备之间存在巨大差异:经过JIT代码优化的设备并不一定比没有经过JIT代码优化的设备好。

To ensure your app performs well across a wide variety of devices, ensure your code is efficient at all levels and aggressively optimize your performance.

代码的执行效果会受到设备CPU,设备内存,系统版本等诸多因素的影响。为了确保代码能够在不同设备上都运行良好，需要最大化代码的效率。

### Avoid Creating Unnecessary Objects 避免创建不必要的对象

Object creation is never free. A generational garbage collector with per-thread allocation pools for temporary objects can make allocation cheaper, but allocating memory is always more expensive than not allocating memory.

对象的创建是需要耗费资源的．虽然GC可以回收不用的对象，可是为这些对象分配内存，并回收它们同样是需要耗费资源的。

As you allocate more objects in your app, you will force a periodic garbage collection, creating little "hiccups" in the user experience. The concurrent garbage collector introduced in Android 2.3 helps, but unnecessary work should always be avoided.

在你的程序中分配更多的对象，将会不间断地的有垃圾回收操作，对用户有一定的影响．虽然Android2.3上的并发垃圾回收机制会有一定的帮助，但是，应避免非必要的工作．

Thus, you should avoid creating object instances you don't need to. Some examples of things that can help:

因此请尽量避免创建不必要的对象，有下面一些例子来说明这个问题：

* If you have a method returning a string, and you know that its result will always be appended to a StringBuffer anyway, change your signature and implementation so that the function does the append directly, instead of creating a short-lived temporary object.如果你需要返回一个String对象，并且你知道它最终会需要连接到一个StringBuffer，请修改你的实现方式，避免直接进行连接操作，应该采用创建一个临时对象来做这个操作.

* When extracting strings from a set of input data, try to return a substring of the original data, instead of creating a copy. You will create a new String object, but it will share the char[] with the data. (The trade-off being that if you're only using a small part of the original input, you'll be keeping it all around in memory anyway if you go this route.) 当从输入的数据集中抽取出Strings的时候，尝试返回原数据的substring对象，而不是创建一个重复的对象

A somewhat more radical idea is to slice up multidimensional arrays into parallel single one-dimension arrays:一个稍微激进点的做法是把所有多维的数据分解成一维的数组:

* An array of ints is a much better than an array of Integer objects, but this also generalizes to the fact that two parallel arrays of ints are also a lot more efficient than an array of (int,int) objects. The same goes for any combination of primitive types.　一组int数据要比一组Integer对象要好很多。可以得知，两组一维数组要比一个二维数组更加的有效率。同样的，这个道理可以推广至其他原始数据类型。

* If you need to implement a container that stores tuples of (Foo,Bar) objects, try to remember that two parallel Foo[] and Bar[] arrays are generally much better than a single array of custom (Foo,Bar) objects. (The exception to this, of course, is when you're designing an API for other code to access. In those cases, it's usually better to make a small compromise to the speed in order to achieve a good API design. But in your own internal code, you should try and be as efficient as possible.)　如果你需要实现一个数组用来存放(Foo,Bar)的对象，尝试分解为Foo[]与Bar[]要比(Foo,Bar)好很多。(当然，为了某些好的API的设计，可以适当做一些妥协。但是在自己的代码内部，你应该多多使用分解后的容易

Generally speaking, avoid creating short-term temporary objects if you can. Fewer objects created mean less-frequent garbage collection, which has a direct impact on user experience.　通常来说，需要避免创建更多的短生命周期的临时对象。更少的对象意味者更少的GC动作，GC会对用户体验有比较直接的影响。

### Prefer Static Over Virtual 选择Static而不是Virtual

If you don't need to access an object's fields, make your method static. Invocations will be about 15%-20% faster. It's also good practice, because you can tell from the method signature that calling the method can't alter the object's state.

如果你不需要访问一个对象的值域,请保证这个方法是static类型的,这样方法调用将快15%-20%。这是一个好的习惯，因为你可以从方法声明中得知调用无法改变这个对象的状态。

### Use Static Final For Constants 常量声明为Static Final

Consider the following declaration at the top of a class:

先看下面这种声明的方式

```java
static int intVal = 42;
static String strVal = "Hello, world!";
```

The compiler generates a class initializer method, called <clinit>, that is executed when the class is first used. The method stores the value 42 into intVal, and extracts a reference from the classfile string constant table for strVal. When these values are referenced later on, they are accessed with field lookups.

编译器会在类首次被使用到的时候，使用初始化<clinit>方法来初始化上面的值，之后访问的时候会需要先到它那里查找，然后才返回数据。

We can improve matters with the "final" keyword:

我们可以通过使用关键字"final"来提升性能：

```java
static final int intVal = 42;
static final String strVal = "Hello, world!";
```

The class no longer requires a <clinit> method, because the constants go into static field initializers in the dex file. Code that refers to intVal will use the integer value 42 directly, and accesses to strVal will use a relatively inexpensive "string constant" instruction instead of a field lookup.

这个类不再需要一个<clinit>方法，因为常量在静态变量初始化中．intVal将直接使用整数值42，而strVal将会使用相对的字符常量来替代字段搜索．

>Note: This optimization applies only to primitive types and String constants, not arbitrary reference types. Still, it's good practice to declare constants static final whenever possible.注意：这个优化只适应于原始类型和String常量，而非引用类型．

### Use Enhanced For Loop Syntax 使用增强的循环写法

The enhanced for loop (also sometimes known as "for-each" loop) can be used for collections that implement the Iterable interface and for arrays. With collections, an iterator is allocated to make interface calls to hasNext() and next(). With an ArrayList, a hand-written counted loop is about 3x faster (with or without JIT), but for other collections the enhanced for loop syntax will be exactly equivalent to explicit iterator usage.

There are several alternatives for iterating through an array:

```java
static class Foo {
    int mSplat;
}

Foo[] mArray = ... ;

public void zero() {
    int sum = 0;
    for (int i = 0; i < mArray.length; ++i) {
        sum += mArray[i].mSplat;
    }
}

public void one() {
    int sum = 0;
    Foo[] localArray = mArray;
    int len = localArray.length;

    for (int i = 0; i < len; ++i) {
        sum += localArray[i].mSplat;
    }
}

public void two() {
    int sum = 0;
    for (Foo a : mArray) {
        sum += a.mSplat;
    }
}

```

* zero() is slowest, because the JIT can't yet optimize away the cost of getting the array length once for every iteration through the loop.

* one() is faster. It pulls everything out into local variables, avoiding the lookups. Only the array length offers a performance benefit.

* two() is fastest for devices without a JIT, and indistinguishable from one() for devices with a JIT. It uses the enhanced for loop syntax introduced in version 1.5 of the Java programming language.

So, you should use the enhanced for loop by default, but consider a hand-written counted loop for performance-critical ArrayList iteration.

>Tip: Also see Josh Bloch's Effective Java, item 46.

#### Consider Package Instead of Private Access with Private Inner Classes

Consider the following class definition:

```Java
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }

    private int mValue;

    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }

    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
```

What's important here is that we define a private inner class (Foo$Inner) that directly accesses a private method and a private instance field in the outer class. This is legal, and the code prints "Value is 27" as expected.

这里重要的是，我们定义了一个私有的内部类（Foo$Inner），它直接访问了外部类中的私有方法以及私有成员对象。这是合法的，这段代码也会如同预期一样打印出”Value is 27”。

The problem is that the VM considers direct access to Foo's private members from Foo$Inner to be illegal because Foo and Foo$Inner are different classes, even though the Java language allows an inner class to access an outer class' private members. To bridge the gap, the compiler generates a couple of synthetic methods:

问题是，VM因为Foo和Foo$Inner是不同的类，会认为在Foo$Inner中直接访问Foo类的私有成员是不合法的。即使Java语言允许内部类访问外部类的私有成员。为了去除这种差异，编译器会产生一些仿造函数：

```Java
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
```

The inner class code calls these static methods whenever it needs to access the mValue field or invoke the doStuff() method in the outer class. What this means is that the code above really boils down to a case where you're accessing member fields through accessor methods. Earlier we talked about how accessors are slower than direct field accesses, so this is an example of a certain language idiom resulting in an "invisible" performance hit.

每当内部类需要访问外部类中的mValue成员或需要调用doStuff()函数时，它都会调用这些静态方法。这意味着，上面的代码可以归结为，通过accessor函数来访问成员变量。早些时候我们说过，通过accessor会比直接访问域要慢。所以，这是一个特定语言用法造成性能降低的例子。

If you're using code like this in a performance hotspot, you can avoid the overhead by declaring fields and methods accessed by inner classes to have package access, rather than private access. Unfortunately this means the fields can be accessed directly by other classes in the same package, so you shouldn't use this in public API.

如果你正在性能热区（hotspot:高频率、重复执行的代码段）使用像这样的代码，你可以把内部类需要访问的域和方法声明为包级访问，而不是私有访问权限。不幸的是，这意味着在相同包中的其他类也可以直接访问这些域，所以在公开的API中你不能这样做。

### Avoid Using Floating-Point

As a rule of thumb, floating-point is about 2x slower than integer on Android-powered devices.

In speed terms, there's no difference between float and double on the more modern hardware. Space-wise, double is 2x larger. As with desktop machines, assuming space isn't an issue, you should prefer double to float.

Also, even for integers, some processors have hardware multiply but lack hardware divide. In such cases, integer division and modulus operations are performed in software—something to think about if you're designing a hash table or doing lots of math.

### Know and Use the Libraries

In addition to all the usual reasons to prefer library code over rolling your own, bear in mind that the system is at liberty to replace calls to library methods with hand-coded assembler, which may be better than the best code the JIT can produce for the equivalent Java. The typical example here is String.indexOf() and related APIs, which Dalvik replaces with an inlined intrinsic. Similarly, the System.arraycopy() method is about 9x faster than a hand-coded loop on a Nexus One with the JIT.

>Tip: Also see Josh Bloch's Effective Java, item 47.

### Use Native Methods Carefully

Developing your app with native code using the Android NDK isn't necessarily more efficient than programming with the Java language. For one thing, there's a cost associated with the Java-native transition, and the JIT can't optimize across these boundaries. If you're allocating native resources (memory on the native heap, file descriptors, or whatever), it can be significantly more difficult to arrange timely collection of these resources. You also need to compile your code for each architecture you wish to run on (rather than rely on it having a JIT). You may even have to compile multiple versions for what you consider the same architecture: native code compiled for the ARM processor in the G1 can't take full advantage of the ARM in the Nexus One, and code compiled for the ARM in the Nexus One won't run on the ARM in the G1.

Native code is primarily useful when you have an existing native codebase that you want to port to Android, not for "speeding up" parts of your Android app written with the Java language.

If you do need to use native code, you should read our JNI Tips.

>Tip: Also see Josh Bloch's Effective Java, item 54.

### Performance Myths


On devices without a JIT, it is true that invoking methods via a variable with an exact type rather than an interface is slightly more efficient. (So, for example, it was cheaper to invoke methods on a HashMap map than a Map map, even though in both cases the map was a HashMap.) It was not the case that this was 2x slower; the actual difference was more like 6% slower. Furthermore, the JIT makes the two effectively indistinguishable.

在没有做JIT之前，使用一种确切的数据类型确实要比抽象的数据类型速度要更有效率。

On devices without a JIT, caching field accesses is about 20% faster than repeatedly accessing the field. With a JIT, field access costs about the same as local access, so this isn't a worthwhile optimization unless you feel it makes your code easier to read. (This is true of final, static, and static final fields too.)

### Always Measure

Before you start optimizing, make sure you have a problem that you need to solve. Make sure you can accurately measure your existing performance, or you won't be able to measure the benefit of the alternatives you try.

You may also find Traceview useful for profiling, but it's important to realize that it currently disables the JIT, which may cause it to misattribute time to code that the JIT may be able to win back. It's especially important after making changes suggested by Traceview data to ensure that the resulting code actually runs faster when run without Traceview.

Originally from : [Performance Tips](https://developer.android.com/training/articles/perf-tips.html)
