---
title: Anko and Extension Functions
date: 2017-06-20 22:01:14
tags:
categories: "Kotlin学习笔记"
---


### What is Anko?

Anko is a powerful library developed by JetBrains. Its main purpose is the generation of UI layouts by using code instead of XML. This is an interesting feature I recommend you to try, but I won’t be using it in this project. To me (probably due to years of experience writing UIs) using XML is much easier, but you could like this approach.

However, this is not the only feature we can get from this library. Anko includes a lot of extremel helpful functions and properties that will avoid lots of boilerplate. We will see many examples throughout this book, but you’ll quickly see which kind of problems this library solves.

Though Anko is really helpful, I recommend you to understand what it is doing behind the scenes. You can navigate at any moment to Anko source code using ctrl + click (Windows) or cmd + click (Mac). Anko implementation is really helpful to learn useful ways to get the most out of Kotlin language.

<!--more-->

### Start using Anko

Before going any further, let’s use Anko to simplify some code. As you will see, anytime you use something from Anko, it will include an import with the name of the property or function to the file. This is because Anko uses extension functions to add new features to Android framework. We’ll see right below what an extension function is and how to write it.

In MainActivity:onCreate, an Anko extension function can be used to simplify how to find the RecyclerView:

```Kotlin
val forecastList: RecyclerView = find(R.id.forecast_list)
```

We can’t use more from the library yet, but Anko can help us to simplify, among others, the instantiation of intents, the navigation between activities, creation of fragments, database access, alerts creation… We’ll find lots of interesting examples while we implement the App.

### Extension functions

An extension function is a function that adds a new behaviour to a class, even if we don’t have access to the source code of that class. It’s a way to extend classes which lack some useful functions. In Java, this is usually implemented in utility classes which include a set of static methods. The advantage of using extension functions in Kotlin is that we don’t need to pass the object as an argument. The extension function acts as if it belonged to the class, and we can implement it using _this_ and all its public methods.

For instance, we can create a toast function which doesn’t ask for the context, which could be used by any Context objects, and those whose type extends Context, such as Activity or Service:

```kotlin
fun Context.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, duration).show()
}
```

This function can be used inside an activity, for instance:

```kotlin
toast("Hello world!")
toast("Hello world!", Toast.LENGTH_LONG)
```

Of course, Anko already includes its own toast extension function, very similar to this one. Anko provides functions for both CharSequence and resources, and different functions for short and long toasts:

```kotlin
toast("Hello world!")
longToast(R.id.hello_world)
```

Extension functions can also be properties. So you can create extension properties too in a very similar way. The following example is showing a way to generate a property with its own getters and setters. Kotlin already provides this property for us as an interoperability feature, but it’s a good exercise to understand the idea behind extension properties:

```kotlin
public var TextView.text: CharSequence
get() = getText()
set(v) = setText(v)
```

Extension functions don’t really modify the original class, but the function is added as a static import where it is used. Extension functions can be declared in any file, so a common practice is to create files which include a set of related functions.

And this is the magic behind many Anko features. From now own, you can create your own magic too.
