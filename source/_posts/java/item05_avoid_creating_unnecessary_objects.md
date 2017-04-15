---
title:  Avoid creating unnecessary objects
date: 2017-04-14 21:33:26
categories: "Effective Java"
---

It is often appropriate to reuse a single object instead of creating a new functionally equivalent object each time it is needed. Reuse can be both faster and more stylish. An object can always be reused if it is immutable (Item 15).

As an extreme example of what not to do, consider this statement:

```Java
String s = new String("stringette"); // DON'T DO THIS!
```

The statement creates a new String instance each time it is executed, and none of those object creations is necessary. The argument to the String constructor ("stringette") is itself a String instance, functionally identical to all of the objects created by the constructor. If this usage occurs in a loop or in a frequently
invoked method, millions of String instances can be created needlessly.

The improved version is simply the following:

```java
 String s = "stringette";
```

This version uses a single String instance, rather than creating a new one each time it is executed. Furthermore, it is guaranteed that the object will be reused by any other code running in the same virtual machine that happens to contain the same string literal [JLS, 3.10.5].

You can often avoid creating unnecessary objects by using static factory methods (Item 1) in preference to constructors on immutable classes that provide both. For example, the static factory method Boolean.valueOf(String) is almost always preferable to the constructor Boolean(String). The constructor creates a new object each time it’s called, while the static factory method is never required
to do so and won’t in practice.

In addition to reusing immutable objects, you can also reuse mutable objects if you know they won’t be modified. Here is a slightly more subtle, and much more common, example of what not to do. It involves mutable Date objects that are never modified once their values have been computed. This class models a
person and has an isBabyBoomer method that tells whether the person is a “baby boomer,” in other words, whether the person was born between 1946 and 1964:

<!--more-->

```java
public class Person {
    private final Date birthDate;
    // Other fields, methods, and constructor omitted
    // DON'T DO THIS!
    public boolean isBabyBoomer() {
        // Unnecessary allocation of expensive object
        Calendar gmtCal =
                  Calendar.getInstance(TimeZone.getTimeZone("GMT"));

        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomStart = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomEnd = gmtCal.getTime();

        return birthDate.compareTo(boomStart) >= 0 &&
                                  birthDate.compareTo(boomEnd) < 0;
    }
}
```

The _isBabyBoomer_ method unnecessarily creates a new _Calendar_, _TimeZone_,
and two _Date_ instances each time it is invoked. The version that follows avoids
this inefficiency with a static initializer:

```java
class Person {
private final Date birthDate;
    // Other fields, methods, and constructor omitted
    /**
    * The starting and ending dates of the baby boom.
    */
    private static final Date BOOM_START;
    private static final Date BOOM_END;

    static {
        Calendar gmtCal =
                      Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }

    public boolean isBabyBoomer() {
        return birthDate.compareTo(BOOM_START) >= 0 &&
                birthDate.compareTo(BOOM_END) < 0;
    }
}
```

The improved version of the _Person_ class creates _Calendar_, _TimeZone_, and _Date_ instances only once, when it is initialized, instead of creating them every time _isBabyBoomer_ is invoked. This results in significant performance gains if the method is invoked frequently. On my machine, the original version takes 32,000 ms for 10 million invocations, while the improved version takes 130 ms, which is about 250 times faster. Not only is performance improved, but so is clarity. Changing _boomStart_ and _boomEnd_ from local variables to static final fields makes it clear that these dates are treated as constants, making the code more understandable. In the interest of full disclosure, the savings from this sort of optimization will not always be this dramatic, as _Calendar_ instances are particularly expensive to create.

If the improved version of the _Person_ class is initialized but its _isBabyBoomer_ method is never invoked, the _BOOM_START_ and _BOOM_END_ fields will be initialized unnecessarily. It would be possible to eliminate the unnecessary initializations by lazily initializing these fields (Item 71) the first time the _isBabyBoomer_ method is invoked, but it is not recommended. As is often the case with lazy initialization, it would complicate the implementation and would be unlikely to result in a noticeable performance improvement beyond what we’ve already achieved (Item 55).

In the previous examples in this item, it was obvious that the objects in question could be reused because they were not modified after initialization. There are other situations where it is less obvious. Consider the case of adapters [Gamma95, p. 139], also known as views. An adapter is an object that delegates to a backing object, providing an alternative interface to the backing object. Because an adapter has no state beyond that of its backing object, there’s no need to create more than one instance of a given adapter to a given object.

For example, the _keySet_ method of the _Map_ interface returns a _Set_ view of the _Map_ object, consisting of all the keys in the map. Naively, it would seem that every call to _keySet_ would have to create a new _Set_ instance, but every call to _keySet_ on a given _Map_ object may return the same _Set_ instance. Although the returned _Set_ instance is typically mutable, all of the returned objects are functionally identical: when one of the returned objects changes, so do all the others because they’re all backed by the same _Map_ instance. While it is harmless to create multiple instances of the _keySet_ view object, it is also unnecessary.

There’s a new way to create unnecessary objects in release 1.5. It is called _autoboxing_, and it allows the programmer to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. _Autoboxing_ blurs but does not erase the distinction between primitive and boxed primitive types. There are subtle semantic distinctions, and not-so-subtle performance differences (Item 49). Consider the following program, which calculates the sum of all the positive _int_ values. To do this, the program has to use long arithmetic, because an _int_ is not big enough to hold the sum of all the positive _int_ values:

```java
// Hideously slow program! Can you spot the object creation?
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```

This program gets the **right** answer, but it is much slower than it should be, due to a one-character typographical error. The variable sum is declared as a Long instead of a long, which means that the program constructs about 2**31 unnecessary Long instances (roughly one for each time the long i is added to the Long sum). Changing the declaration of sum from Long to long reduces the runtime from 43 seconds to 6.8 seconds on my machine. The lesson is clear: **prefer primitives to boxed primitives, and watch out for unintentional autoboxing**.

This item should not be misconstrued to imply that object creation is expensive and should be avoided. On the contrary, the creation and reclamation of small objects whose constructors do little explicit work is cheap, especially on modern JVM implementations. Creating additional objects to enhance the clarity, simplicity, or power of a program is generally a good thing.

Conversely, avoiding object creation by maintaining your own object pool is a bad idea unless the objects in the pool are extremely heavyweight. The classic example of an object that does justify an object pool is a database connection. The cost of establishing the connection is sufficiently high that it makes sense to reuse these objects. Also, your database license may limit you to a fixed number of connections. Generally speaking, however, maintaining your own object pools clutters your code, increases memory footprint, and harms performance. Modern JVM implementations have highly optimized garbage collectors that easily outperform such object pools on lightweight objects.

The counterpoint to this item is Item 39 on defensive copying. Item 5 says, “Don’t create a new object when you should reuse an existing one,” while Item 39 says, “Don’t reuse an existing object when you should create a new one.” Note that the penalty for reusing an object when defensive copying is called for is far greater than the penalty for needlessly creating a duplicate object. Failing to make defensive copies where required can lead to insidious bugs and security holes; creating objects unnecessarily merely affects style and performance.
