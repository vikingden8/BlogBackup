---
title: Kotlin Basic Types
date: 2017-07-01 22:25:14
tags:
categories: "Kotlin学习笔记"
---

In Kotlin, everything is an object in the sense that we can call member functions and properties on any variable. Some types are built-in, because their implementation is optimized, but to the user they look like ordinary classes. In this section we describe most of these types: _numbers_, _characters_, _booleans_ and _arrays_.

在 Kotlin 中，所有变量的成员方法和属性都是一个对象。一些类型是内建的，它们的实现是优化过的，但对用户来说它们就像普通的类一样。在这节中，我们将会讲到大多数的类型：数值，字符，布尔，以及数组。+


### Numbers 数值

Kotlin handles numbers in a way close to Java, but not exactly the same. For example, there are no implicit widening conversions for numbers, and literals are slightly different in some cases.

Kotlin 处理数值的方法和 java 很相似，但不是完全一样。比如，不存在隐式转换数值的精度，并且在字面上有一些小小的不同。

Kotlin provides the following built-in types representing numbers (this is close to Java):

Kotlin 提供了如下内建数值类型(和 java 很相似)：

![](/images/categories/kotlin/006/01.png)

Note that characters are not numbers in Kotlin.

注意：字符在 Kotlin 中不是数值类型

<!--more-->

#### Literal Constants 字面值常量

There are the following kinds of literal constants for integral values:

主要是以下几种字面值常量：

* Decimals 十进制数值 : 123

  1. Longs are tagged by a capital L 长整型要加大写 L: 123L

* Hexadecimals 十六进制 : 0x0F

* Binaries 二进制: 0b00001011

NOTE: Octal literals are not supported.

注意：不支持八进制

Kotlin also supports a conventional notation for floating-point numbers:

Kotlin 也支持传统的浮点数表示：

* Doubles by default 默认双精度浮点数 : 123.5, 123.5e10

* Floats are tagged by f or F 单精度浮点数要添加 f 或 F: 123.5f

#### Underscores in numeric literals (since 1.1) 数值常量中可以添加下划线分割(1.1版本新特性)

You can use underscores to make number constants more readable:

您可以使用下划线增加数值常量的可读性:

```Kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

#### Representation 表示

On the Java platform, numbers are physically stored as JVM primitive types, unless we need a nullable number reference (e.g. Int?) or generics are involved. In the latter cases numbers are boxed.

在 java 平台上，数值被 JVM 虚拟机以字节码的方式物理存储的，除非我们需要做可空标识(比如说 Int?) 或者涉及泛型。在后者中数值是被装箱的。

Note that boxing of numbers does not necessarily preserve identity:

注意装箱过的数值是不保留特征的：

```Kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```

On the other hand, it preserves equality:

另一方面，它们是值相等的：

```Kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

#### Explicit Conversions 显式转换

Due to different representations, smaller types are not subtypes of bigger ones. If they were, we would have troubles of the following sort:

由于不同的表示，短类型不是长类型的子类型。如果是的话，我们就会碰到下面这样的麻烦了

```Kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(a == b) // Surprise! This prints "false" as Long's equals() check for other part to be Long as well
```

So not only identity, but even equality would have been lost silently all over the place.

因此不止是恒等，有时候连等于都会悄悄丢失。

As a consequence, smaller types are NOT implicitly converted to bigger types. This means that we cannot assign a value of type Byte to an Int variable without an explicit conversion.

所以，短类型是不会隐式转换为长类型的。这意味着我们必须显式转换才能把 Byte 赋值给 Int。

```Kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```

We can use explicit conversions to widen numbers

我们可以通过显式转换把数值类型提升

```Kotlin
val i: Int = b.toInt() // OK: explicitly widened
```

Every number type supports the following conversions:

每个数值类型都支持下面的转换：

```Kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```

Absence of implicit conversions is rarely noticeable because the type is inferred from the context, and arithmetical operations are overloaded for appropriate conversions, for example：

隐式转换一般情况下是不容易被发觉的，因为我们使用了上下文推断出类型，并且算术运算会为合适的转换进行重载，比如：

```kotlin
val l = 1L + 3 // Long + Int => Long
```

#### Operations 运算符

Kotlin supports the standard set of arithmetical operations over numbers, which are declared as members of appropriate classes (but the compiler optimizes the calls down to the corresponding instructions). See Operator overloading.

Kotlin支持标准的算术运算表达式，这些运算符被声明为相应类的成员(但是编译器将调用优化到相应的指令)。参看运算符重载。

As of bitwise operations, there're no special characters for them, but just named functions that can be called in infix form, for example:

至于位运算，Kotlin 并没有提供特殊的操作符，只是提供了可以叫中缀形式的方法，比如：

```kotlin
val x = (1 shl 2) and 0x000FF000
```

Here is the complete list of bitwise operations (available for Int and Long only):

下面是全部的位运算操作符(只可以用在 Int 和 Long 类型)：

```kotlin
shl(bits) – signed shift left (Java's <<)
shr(bits) – signed shift right (Java's >>)
ushr(bits) – unsigned shift right (Java's >>>)
and(bits) – bitwise and
or(bits) – bitwise or
xor(bits) – bitwise xor
inv() – bitwise inversion
```

### Characters 字符

Characters are represented by the type Char. They can not be treated directly as numbers

字符类型用 Char 表示。不能直接当做数值来使用

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

Character literals go in single quotes: '1'. Special characters can be escaped using a backslash. The following escape sequences are supported: \\t, \\b, \\n, \\r, \\', \\", \\\\ and \\$. To encode any other character, use the Unicode escape sequence syntax: '\\uFF00'.

字符是由单引号包裹的'1'，特殊的字符通过反斜杠\\转义，下面的字符序列支持转义：\\t,\\b,\\n,\\r,\\',\\",\\\\和\\$。编码任何其他字符，使用 Unicode 转义语法：\\uFF00。+

We can explicitly convert a character to an Int number:

我们可以将字符显示的转义为Int数字：

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

Like numbers, characters are boxed when a nullable reference is needed. Identity is not preserved by the boxing operation.

和数值类型一样，需要一个可空引用时，字符会被装箱。特性不会被装箱保留。

### Booleans 布尔值

The type Boolean represents booleans, and has two values: _true_ and _false_.

布尔值只有 true 或者 false

Booleans are boxed if a nullable reference is needed.

如果需要一个可空引用，则可以将布尔值装箱

Built-in operations on booleans include

布尔值的内建操作包括

* || – lazy disjunction

* && – lazy conjunction

* ! - negation

### Arrays 数组

Arrays in Kotlin are represented by the Array class, that has get and set functions (that turn into [] by operator overloading conventions), and size property, along with a few other useful member functions:

数组在 Kotlin 中由 Array 类表示，有 get 和 set 方法(通过运算符重载可以由[]调用)，以及 size 方法，以及一些常用的函数：

```Kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

To create an array, we can use a library function _arrayOf()_ and pass the item values to it, so that _arrayOf(1, 2, 3)_ creates an array [1, 2, 3]. Alternatively, the _arrayOfNulls()_ library function can be used to create an array of a given size filled with null elements.

我们可以给库函数 arrayOf() 传递每一项的值来创建Array，arrayOf(1, 2, 3) 创建了一个[1, 2, 3] 这样的数组。也可以使用库函数 arrayOfNulls() 创建一个指定大小的空Array。

Another option is to use a factory function that takes the array size and the function that can return the initial value of each array element given its index:

或者通过指定Array大小并提供一个通过索引产生数组元素值的工厂函数：

```Kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

As we said above, the [] operation stands for calls to member functions get() and set().

像我们上面提到的，[] 操作符表示调用　get() set() 函数

Note: unlike Java, arrays in Kotlin are invariant. This means that Kotlin does not let us assign an Array<String> to an Array<Any>, which prevents a possible runtime failure (but you can use Array<out Any>, see Type Projections).

注意：和 java 不一样，arrays 在 kotlin 中是不可变的。这意味这 kotlin 不允许我们把 Array<String> 转为 Array<Any> ,这样就阻止了可能的运行时错误(但你可以使用 Array<outAny> , 参看 Type Projections)。

Kotlin also has specialized classes to represent arrays of primitive types without boxing overhead: ByteArray, ShortArray, IntArray and so on. These classes have no inheritance relation to the Array class, but they have the same set of methods and properties. Each of them also has a corresponding factory function:

Kotlin 有专门的类来表示原始类型从而避免过度装箱： ByteArray, ShortArray, IntArray 等等。这些类与 Array 没有继承关系，但它们有一样的方法与属性。每个都有对应的库函数：

```Kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

### Strings 字符串

Strings are represented by the type String. Strings are immutable. Elements of a string are characters that can be accessed by the indexing operation: s[i]. A string can be iterated over with a for-loop:

字符串是由 String 表示的。字符串是不变的。字符串的元素可以通过索引操作读取: s[i] 。字符串可以用 for 循环迭代：

```Kotlin
for (c in str) {
    println(c)
}
```

#### String Literals 字符串字面量

Kotlin has two types of string literals: escaped strings that may have escaped characters in them and raw strings that can contain newlines and arbitrary text. An escaped string is very much like a Java string:

Kotlin 有两种类型的字符串字面量：一种是可以带分割符的，一种是可以包含新行以及任意文本的。带分割符的 string 很像 java 的 string:

```Kotlin
val s = "Hello, world!\n"
```

Escaping is done in the conventional way, with a backslash. See Characters above for the list of supported escape sequences.

转义是使用传统的反斜线的方式。参见Characters，以获得支持的转义序列。

A raw string is delimited by a triple quote ("""), contains no escaping and can contain newlines and any other characters:

整行String 是由三个引号包裹的("""),不可以包含分割符但可以包含其它字符：

```Kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

You can remove leading whitespace with trimMargin() function:

你可以通过trim-margin()函数移除空格：

```Kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

By default _|_ is used as margin prefix, but you can choose another character and pass it as a parameter, like _trimMargin(">")_.

#### String Templates 字符串模板

Strings may contain template expressions, i.e. pieces of code that are evaluated and whose results are concatenated into the string. A template expression starts with a dollar sign ($) and consists of either a simple name:

字符串可以包含模板表达式，即可求值的代码片段，并将其结果连接到字符串中。一个模板表达式由一个 $ 开始并包含另一个简单的名称：

```Kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

or an arbitrary expression in curly braces:

或者是一个带大括号的表达式：

```Kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

Templates are supported both inside raw strings and inside escaped strings. If you need to represent a literal $ character in a raw string (which doesn't support backslash escaping), you can use the following syntax:

模板既可以原始字符串中使用，也可以在转义字符串中使用。如果需要在原始字符串(不支持反斜杠转义)中表示一个文字$字符，那么可以使用以下语法：

```Kotlin
val price = """
${'$'}9.99
"""
```
