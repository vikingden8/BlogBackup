---
title: Kotlin Control Flow
date: 2017-07-01 23:23:59
tags:
categories: "Kotlin学习笔记"
---

### If Expression

In Kotlin, if is an expression, i.e. it returns a value. Therefore there is no ternary operator (condition ? then : else), because ordinary if works fine in this role.

```Kotlin
// Traditional usage
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// As expression
val max = if (a > b) a else b
```

if branches can be blocks, and the last expression is the value of a block:

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

If you're using if as an expression rather than a statement (for example, returning its value or assigning it to a variable), the expression is required to have an else branch.

<!--more-->

### When Expression

when replaces the switch operator of C-like languages. In the simplest form it looks like this

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

when matches its argument against all branches sequentially until some branch condition is satisfied. when can be used either as an expression or as a statement. If it is used as an expression, the value of the satisfied branch becomes the value of the overall expression. If it is used as a statement, the values of individual branches are ignored. (Just like with if, each branch can be a block, and its value is the value of the last expression in the block.)

The else branch is evaluated if none of the other branch conditions are satisfied. If when is used as an expression, the else branch is mandatory, unless the compiler can prove that all possible cases are covered with branch conditions.

If many cases should be handled in the same way, the branch conditions may be combined with a comma:

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

We can use arbitrary expressions (not only constants) as branch conditions

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

We can also check a value for being in or !in a range or a collection:

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

Another possibility is to check that a value is or !is of a particular type. Note that, due to smart casts, you can access the methods and properties of the type without any extra checks.

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

when can also be used as a replacement for an if-else if chain. If no argument is supplied, the branch conditions are simply boolean expressions, and a branch is executed when its condition is true:

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

### For Loops

for loop iterates through anything that provides an iterator. The syntax is as follows:

```kotlin
for (item in collection) print(item)
```

The body can be a block.

```kotlin
for (item: Int in ints) {
    // ...
}
```

As mentioned before, for iterates through anything that provides an iterator, i.e.

* has a member- or extension-function _iterator()_, whose return type

* has a member- or extension-function _next()_, and

* has a member- or extension-function _hasNext()_ that returns _Boolean_.

All of these three functions need to be marked as operator.

A for loop over an array is compiled to an index-based loop that does not create an iterator object.

If you want to iterate through an array or a list with an index, you can do it this way:

```kotlin
for (i in array.indices) {
    print(array[i])
}
```

Note that this "iteration through a range" is compiled down to optimal implementation with no extra objects created.

Alternatively, you can use the withIndex library function:

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

### While Loops

while and do..while work as usual

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

### Break and continue in loops

Kotlin supports traditional break and continue operators in loops. See [Returns and jumps](http://kotlinlang.org/docs/reference/returns.html).