---
title: Variables and properties
date: 2017-06-19 20:08:44
tags:
categories: "Kotlin学习笔记"
---

In Kotlin, **everything is an object**. We don’t find primitive types as the ones we can use in Java. That’s really helpful, because we have an homogeneous way to deal with all the available types.

### Basic types

Of course, basic types such as integers, floats, characters or booleans still exist, but they all act as an object. The name of the basic types and the way they work are very similar to Java, but there are some differences you might take into account:

There are no automatic conversions among numeric types. For instance, you cannot assign an Int to a Double variable. An explicit conversion must be done, using one of the many functions available:

```Kotlin
val i: Int = 7
val d: Double = i.toDouble()
```

<!--more-->

Characters (Char) cannot directly be used as numbers. We can, however, convert them to a number when we need it:

```Kotlin
val c: Char = 'c'
val i: Int = c.toInt()
```

Bitwise arithmetical operations are a bit different. In Android, we use bitwise or quite often for flags, so I’ll stick to “and” and “or “ as an example:

```Kotlin
// Java
int bitwiseOr = FLAG1 | FLAG2;
int bitwiseAnd = FLAG1 & FLAG2;
// Kotlin
val bitwiseOr = FLAG1 or FLAG2
val bitwiseAnd = FLAG1 and FLAG2
```

There are many other bitwise operations, such as shl, shs, ushr, xor or inv. You can take a look at the [official Kotlin reference](http://kotlinlang.org/docs/reference/basic-types.html#operations) for more information.

Literals can give information about its type. It’s not a requirement, but a common practice in Kotlin is to omit variable types (we’ll see it soon), so we can give some clues to the compiler to let it infer the type from the literal:

```Kotlin
val i = 12 // An Int
val iHex = 0x0f // An Int from hexadecimal literal
val l = 3L // A Long
val d = 3.5 // A Double
val f = 3.5F // A Float
```

A String can be accessed as an array and can be iterated:

```Kotlin
val s = "Example"
val c = s[2] // This is the Char 'a'
// Iterate over String
val s = "Example"
for (c in s) {
  print(c)
}
```

### Variables

Variables in Kotlin can be easily defined as mutable (var) or immutable (val). The idea is very similar to using final in Java variables. But immutability is a very important concept in Kotlin (and many other modern languages).

An immutable object is an object whose state cannot change after instantiation. If you need a modified version of the object, a new object needs to be created. This makes programming much more robust and predictable. In Java, most objects are mutable, which means that any part of the code which has access to the object can modify it, affecting the rest of the application.

Immutable objects are also thread-safe by definition. As they can’t change, no special access control must be defined, because all threads will always get the same object.

So the way we think about coding changes a bit in Kotlin if we want to make use of immutability. The key concept: just use val as much as possible. There will be situations (specially in Android, where we don’t have access to the constructor of many classes) where it won’t be possible, but it will most of the time.

Another thing mentioned before is that we usually don’t need to specify object types, they will be inferred from the value, which makes the code cleaner and faster to modify. We already have some examples from the section above.

```Kotlin
val s = "Example" // A String
val i = 23 // An Int
val actionBar = supportActionBar // An ActionBar in an Activity context
```

However, a type needs to be specified if we want to use a more generic type:

```Kotlin
val a: Any = 23
val c: Context = activity
```

### Properties

Properties are the equivalent to fields in Java, but much more powerful. Properties will do the work of a field plus a getter plus a setter. Let’s see an example to compare the difference. This is the code required in Java to safely access and modify a field:

```Java
public class Person {

  private String name;

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}

...

Person person = new Person();
person.setName("name");
String name = person.getName();

```

In Kotlin, only a property is required:

```Kotlin
public class Person {

  var name: String = ""

}

...

val person = Person()
person.name = "name"
val name = person.name
```

If nothing is specified, the property uses the default getter and setter. It can, of course, be modified to run whatever custom code you need, without having to change the existing code:

```Kotlin
public class Person {

  var name: String = ""
  get() = field.toUpperCase()
  set(value) {
    field = "Name: $value"
  }

}
```

If the property needs access to its own value in custom getter and setter (as in this case), it requires the creation of a backing field. It can be accessed by using field, a reserved word, and will be automatically created by the compiler when it finds that it’s being used. Take into account that if we used the property directly, we would be using the setter and getter, and not doing a direct assignment. The backing field can only be used inside property accessors.

As mentioned in some previous chapters, when operating with Java code Kotlin will allow to use the property syntax where a getter and a setter are defined in Java. The compiler will just link to the original getters and setters, so there are no performance penalties when using these mapped properties.
