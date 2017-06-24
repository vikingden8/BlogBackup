---
title: Data Classes
date: 2017-06-24 19:34:19
tags:
categories: "Kotlin学习笔记"
---

Data classes are a powerful kind of classes which avoid the boilerplate we need in Java to create POJO: classes which are used to keep state, but are very simple in the operations they do. They usually only provide plain getters and setters to access to their fields. Defining a new data class is very easy:

```Kotlin
data class Forecast(val date: Date, val temperature: Float, val details: String)
```

### Extra functions

Along with a data class, we get a handful of interesting functions for free, apart from the properties we already talked about (which prevents us from writing getters and setters):

  • _equals()_: it compares the properties from both objects to ensure they are identical.

  • _hashCode()_: we get a hash code for free, also calculated from the values of the properties.

  • _copy()_: you can copy an object, modifying the properties you need. We’ll see an example later.

  • A set of numbered functions that are useful to map an object into variables. It will also be
explained soon

<!--more-->

### Copying a data class

If we use immutability, as talked some chapters ago, we’ll find that if we want to change the state of an object, a new instance of the class is required, with one or more of its properties modified. This task can be rather repetitive and far from clean. However, data classes include the _copy()_ method, which will make the process really easy and intuitive.

For instance, if we need to modify the _temperature_ of a _Forecast_, we can just do

```Kotlin
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val f2 = f1.copy(temperature = 30f)
```

This way, we copy the first forecast and modify only the _temperature_ property without changing the state of the original object.

>**Be careful with immutability when using Java classes**

>If you decide to work with immutability, be aware that Java classes weren’t designed with this in mind, and there are still some situations where you will be able to modify the state. In the previous example, you could still access the Date object and change its value. The easy (and unsafe) option is to remember the rules of not modifying the state of any object, but copying it when necessary.

>Another option is to wrap these classes. You could create an ImmutableDate class which wraps a Date and doesn’t allow to modify its state. It’s up to you to decide which solution you take. In this book, I won’t be very strict with immutability (as it’s not its main goal), so I won’t create wrappers for every potentially dangerous classes.

###  Mapping an object into variables

This process is known as multi-declaration and consists of mapping each property inside an object into a variable. That’s the reason why the componentX functions are automatically created. An example with the previous _Forecast_ class:

```kotlin
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val (date, temperature, details) = f1
```

This multi-declaration is compiled down to the following code:

```kotlin
val date = f1.component1()
val temperature = f1.component2()
val details = f1.component3()
```

The logic behind this feature is very powerful, and can help simplify the code in many situations. For instance, Map class has some extension functions implemented that allow to recover its keys and values in an iteration:

```kotlin
for ((key, value) in map) {
  Log.d("map", "key:$key, value:$value")
}
```
