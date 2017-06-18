---
title: Classes and Functions
date: 2017-06-18 21:55:36
tags:
categories: "Kotlin学习笔记"
---

Classes in Kotlin follow a really simple structure. However, there are some slight differences from Java that you will want to know before we continue. You can use [try.kotlinlang.org](https://try.kotlinlang.org) to test this and some other simple examples without the need of a real project.

### How to declare a class

If you want to declare a class, you just need to use the keyword _class_:

```Kotlin
class MainActivity {

}
```

Classes have a unique default constructor. We’ll see that we can create extra constructors for some exceptional cases, but keep in mind that most situations only require a single constructor. Parameters are written just after the name. Brackets are not required if the class doesn’t have any content:

```Kotlin
class Person(name: String, surname: String)
```

Where’s the body of the constructor then? You can declare an _init_ block:

```Kotlin
class Person(name: String, surname: String) {
  init {
    ...
  }
}

```

<!--more-->

### Class inheritance

By default, a class always extends from _Any_ (similar to Java Object), but we can extend any other classes. Classes are closed by default (final), so we can only extend a class if it’s explicitly declared as _open_ or _abstract_:

```Kotlin
open class Animal(name: String)

class Person(name: String, surname: String) : Animal(name)

```

Note that when using the single constructor nomenclature, we need to specify the parameters we’re using for the parent constructor. That’s the equivalent to super() call in Java.

###  Functions

Functions (our methods in Java) are declared just using the _fun_ keyword:

```Kotlin
fun onCreate(savedInstanceState: Bundle?) {

}
```

It you don’t specify a return value, it will return _Unit_, similar to _void_ in Java, though this is really an object. You can, of course, specify any type as a return value:

```Kotlin
fun add(x: Int, y: Int) : Int {
  return x + y
}
```

>Tip: Semi-colons are not necessary

>As you can see in the example above, I’m not using semi-colons at the end of the sentences. While you can use them, semi-colons are not necessary and it’s a good practice not to use them. When you get used, you’ll find that it saves you a lot of time.

However, if the result can be calculated using a single expression, you can get rid of brackets and use equal:

```Kotlin
fun add(x: Int, y: Int) : Int = x + y
```

### Constructor and functions parameters

Parameters in Kotlin are a bit different from Java. As you can see, we first write the name of the parameter and then its type.

```Kotlin
fun add(x: Int, y: Int) : Int {
  return x + y
}
```

An extremely useful thing about parameters is that we can make them optional by specifying a default value. Here it is an example of a function you could create in an activity, which uses a toast to show a message:

```Kotlin
fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
  Toast.makeText(this, message, length).show()
}
```

As you can see, the second parameter (length) specifies a default value. This means you can write
the second value or not, which avoids the need of function overloading:

```Kotlin
toast("Hello")
toast("Hello", Toast.LENGTH_LONG)
```

This would be equivalent to the next code in Java:

```Java
void toast(String message){
  toast(message, Toast.LENGTH_SHORT);
}

void toast(String message, int length){
  Toast.makeText(this, message, length).show();
}
```

And this can be as complex as you want. Check this other example:

```Kotlin
fun niceToast(message: String,
tag: String = MainActivity::class.java.simpleName,
length: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, "[$tag] $message", length).show()
}
```

I’ve added a third parameter that includes a tag which defaults to the class name. The amount of overloads we’d need in Java grows exponentially. You can now write these calls:

```Kotlin
toast("Hello")
toast("Hello", "MyTag")
toast("Hello", "MyTag", Toast.LENGTH_SHORT)
```

And there is even another option, because named arguments can be used, which means you can write the name of the argument preceding the value to specify which one you want:

```Kotlin
toast(message = "Hello", length = Toast.LENGTH_SHORT)
```

>Tip: String templates

>You can use template expressions directly in your strings. This will make it easy to
write complex strings based on static and variable parts. In the previous example, I used
"[$className] $message".

>As you can see, anytime you want to add an expression, you need to use the $ symbol. If
the expression is a bit more complex, you’ll need to add a couple of brackets: "Your name
is ${user.name}".
