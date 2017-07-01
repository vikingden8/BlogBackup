---
title: Kotlin Packages
date: 2017-07-01 23:11:33
tags:
categories: "Kotlin学习笔记"
---

A source file may start with a package declaration:

```Kotlin
package foo.bar

fun baz() {}

class Goo {}

// ...
```

All the contents (such as classes and functions) of the source file are contained by the package declared. So, in the example above, the full name of baz() is foo.bar.baz, and the full name of Goo is foo.bar.Goo.

If the package is not specified, the contents of such a file belong to "default" package that has no name.

### Default Imports

A number of packages are imported into every Kotlin file by default:

* kotlin.*
* kotlin.annotation.*
* kotlin.collections.*
* kotlin.comparisons.* (since 1.1)
* kotlin.io.*
* kotlin.ranges.*
* kotlin.sequences.*
* kotlin.text

Additional packages are imported depending on the target platform:

* JVM:

  1. java.lang.*
  2. kotlin.jvm.*

* JS:

  1. kotlin.js.*

<!--more-->

### Imports

Apart from the default imports, each file may contain its own import directives. Syntax for imports is described in the grammar.

We can import either a single name, e.g.

```kotlin
import foo.Bar // Bar is now accessible without qualification
```

or all the accessible contents of a scope (package, class, object etc):

```kotlin
import foo.* // everything in 'foo' becomes accessible
```

If there is a name clash, we can disambiguate by using as keyword to locally rename the clashing entity:

```kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```

The _import_ keyword is not restricted to importing classes; you can also use it to import other declarations:

* top-level functions and properties;

* functions and properties declared in object declarations;

* enum constants

Unlike Java, Kotlin does not have a separate "[import static](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html)" syntax; all of these declarations are imported using the regular import keyword.

### Visibility of Top-level Declarations

If a top-level declaration is marked private, it is private to the file it's declared in (see [Visibility Modifiers](http://kotlinlang.org/docs/reference/visibility-modifiers.html)).
