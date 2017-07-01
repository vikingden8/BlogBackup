---
title: Kotlin Retruns And Jumps
date: 2017-07-01 23:39:51
tags:
categories: "Kotlin学习笔记"
---

Kotlin has three structural jump expressions:

* _return_. By default returns from the nearest enclosing function or anonymous function.

* _break_. Terminates the nearest enclosing loop.

* _continue_. Proceeds to the next step of the nearest enclosing loop.

All of these expressions can be used as part of larger expressions:

```Kotlin
val s = person.name ?: return
```

The type of these expressions is the Nothing type.

<!--more-->

### Break and Continue Labels

Any expression in Kotlin may be marked with a label. Labels have the form of an identifier followed by the @ sign, for example: abc@, fooBar@ are valid labels (see the grammar). To label an expression, we just put a label in front of it

```Kotlin
loop@ for (i in 1..100) {
    // ...
}
```

Now, we can qualify a break or a continue with a label:

```Kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

A break qualified with a label jumps to the execution point right after the loop marked with that label. A continue proceeds to the next iteration of that loop.

### Return at Labels

With function literals, local functions and object expression, functions can be nested in Kotlin. Qualified returns allow us to return from an outer function. The most important use case is returning from a lambda expression. Recall that when we write this:

```Kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return  // nonlocal return from inside lambda directly to the caller of foo()
        print(it)
    }
}
```

The return-expression returns from the nearest enclosing function, i.e. foo. (Note that such non-local returns are supported only for lambda expressions passed to inline functions.) If we need to return from a lambda expression, we have to label it and qualify the return:

```Kotlin
fun foo() {
    ints.forEach lit@ {
        if (it == 0) return@lit
        print(it)
    }
}
```

Now, it returns only from the lambda expression. Oftentimes it is more convenient to use implicits labels: such a label has the same name as the function to which the lambda is passed.

```Kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

Alternatively, we can replace the lambda expression with an anonymous function. A return statement in an anonymous function will return from the anonymous function itself.

```Kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
}
```

When returning a value, the parser gives preference to the qualified return, i.e.

```Kotlin
return@a 1
```

means "return 1 at label @a" and not "return a labeled expression (@a 1)".
