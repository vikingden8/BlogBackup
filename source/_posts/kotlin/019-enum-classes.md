---
title: Kotlin Enum Classes
date: 2017-07-02 09:24:32
tags:
categories: "Kotlin学习笔记"
---

The most basic usage of _enum_ classes is implementing type-safe enums

```Kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

Each enum constant is an object. Enum constants are separated with commas.

<!--more-->

### Initialization

Since each enum is an instance of the enum class, they can be initialized

```Kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

### Anonymous Classes

Enum constants can also declare their own anonymous classes

```Kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

with their corresponding methods, as well as overriding base methods. Note that if the enum class defines any members, you need to separate the enum constant definitions from the member definitions with a semicolon, just like in Java.

### Working with Enum Constants

Just like in Java, enum classes in Kotlin have synthetic methods allowing to list the defined enum constants and to get an enum constant by its name. The signatures of these methods are as follows (assuming the name of the enum class is EnumClass):

```Kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

The _valueOf()_ method throws an _IllegalArgumentException_ if the specified name does not match any of the enum constants defined in the class.

Since Kotlin 1.1, it's possible to access the constants in an enum class in a generic way, using the _enumValues<T>()_ and _enumValueOf<T>()_ functions:

```Kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // prints RED, GREEN, BLUE
```

Every enum constant has properties to obtain its name and position in the enum class declaration:

```Kotlin
val name: String
val ordinal: Int
```

The enum constants also implement the Comparable interface, with the natural order being the order in which they are defined in the enum class.
