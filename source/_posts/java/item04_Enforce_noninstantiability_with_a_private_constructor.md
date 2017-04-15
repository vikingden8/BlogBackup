---
title:  Enforce noninstantiability with a private constructor
date: 2017-04-14 22:05:41
categories: "Effective Java"
---

Occasionally you’ll want to write a class that is just a grouping of static methods and static fields. Such classes have acquired a bad reputation because some people abuse them to avoid thinking in terms of objects, but they do have valid uses. They can be used to group related methods on primitive values or arrays, in the manner of _java.lang.Math_ or _java.util.Arrays_. They can also be used to group static methods, including factory methods (Item 1), for objects that implement a particular interface, in the manner of _java.util.Collections_. Lastly, they can be used to group methods on a final class, instead of extending the class.

Such utility classes were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

<!--more->

**Attempting to enforce noninstantiability by making a class abstract does not work**. The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 17). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so **a class can be
made noninstantiable by including a private constructor**:

```java
// Noninstantiable utility class
public class UtilityClass {
      // Suppress default constructor for noninstantiability
      private UtilityClass() {
      throw new AssertionError();
      }
      ... // Remainder omitted
}

```

Because the explicit constructor is private, it is inaccessible outside of the class. The AssertionError isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees that the class will never be instantiated under any circumstances. This idiom is mildly counterintuitive, as the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment, as shown above.

As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.
