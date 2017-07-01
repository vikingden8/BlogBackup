---
title: Kotlin Retruns And Jumps
date: 2017-07-01 23:39:51
tags:
categories: "Kotlin学习笔记"
---

Kotlin has three structural jump expressions:

Kotlin 有三种结构跳转表达式：

* _return_. By default returns from the nearest enclosing function or anonymous function.

* _break_. Terminates the nearest enclosing loop.

* _continue_. Proceeds to the next step of the nearest enclosing loop.

All of these expressions can be used as part of larger expressions:

上述表达式都可以作为更大的表达式的一部分：

```Kotlin
val s = person.name ?: return
```

The type of these expressions is the Nothing type.

这些表达式的类型是 Nothing type

<!--more-->

### Break and Continue Labels

Any expression in Kotlin may be marked with a label. Labels have the form of an identifier followed by the @ sign, for example: abc@, fooBar@ are valid labels (see the grammar). To label an expression, we just put a label in front of it.

在 Kotlin 中表达式可以添加标签。标签通过 @ 结尾来表示，比如：abc@，fooBar@ 都是有效的(参看语法)。使用标签语法只需像这样：

```Kotlin
loop@ for (i in 1..100) {
    // ...
}
```

Now, we can qualify a break or a continue with a label:

现在我们可以用标签实现 break 或者 continue 的快速跳转：

```Kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

A break qualified with a label jumps to the execution point right after the loop marked with that label. A continue proceeds to the next iteration of that loop.

break 是跳转标签后面的表达式，continue 是跳转到循环的下一次迭代。

### Return at Labels

With function literals, local functions and object expression, functions can be nested in Kotlin. Qualified returns allow us to return from an outer function. The most important use case is returning from a lambda expression. Recall that when we write this:

在字面函数，局部函数，以及对象表达式中，函数可以在 Kotlin 中被包裹。return 允许我们返回到外层函数。最重要的例子就是从字面函数中返回，还记得我们之前的写法吗：

```Kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return  // nonlocal return from inside lambda directly to the caller of foo()
        print(it)
    }
}
```

The return-expression returns from the nearest enclosing function, i.e. foo. (Note that such non-local returns are supported only for lambda expressions passed to inline functions.) If we need to return from a lambda expression, we have to label it and qualify the return:

return 表达式返回到最近的闭合函数，比如 foo (注意这样非局部返回仅仅可以在内联函数中使用)。如果我们需要从一个字面函数返回可以使用标签修饰 return :

```Kotlin
fun foo() {
    ints.forEach lit@ {
        if (it == 0) return@lit
        print(it)
    }
}
```

Now, it returns only from the lambda expression. Oftentimes it is more convenient to use implicits labels: such a label has the same name as the function to which the lambda is passed.

现在它仅仅从字面函数中返回。经常用一种更方便的含蓄的标签：比如用和传入的 lambda 表达式名字相同的标签。

```Kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

Alternatively, we can replace the lambda expression with an anonymous function. A return statement in an anonymous function will return from the anonymous function itself.

另外，我们可以用函数表达式替代匿名函数。在函数表达式中使用 return 语句可以从函数表达式中返回。+

```Kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
}
```

When returning a value, the parser gives preference to the qualified return, i.e.

当返回一个值时，解析器给了一个参考：

```Kotlin
return@a 1
```

means "return 1 at label @a" and not "return a labeled expression (@a 1)".

表示 “在标签 @a 返回 1 ” 而不是返回一个标签表达式 (@a 1).
