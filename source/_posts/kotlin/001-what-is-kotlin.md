---
title: Kotlin Introduction
date: 2017-6-16 11:22:21
tags:
categories: "Kotlin学习笔记"
---

### What is Kotlin?

Kotlin, as described before, is a JVM based language developed by [JetBrains](https://www.jetbrains.com/), a company knownfor the creation of IntelliJ IDEA, a powerful IDE for Java development. Android Studio, the official Android IDE, is based on IntelliJ.

Kotlin was created with Java developers in mind, and with IntelliJ as its main development IDE. And these are two very interesting features for Android developers:

* **Kotlin is very intuitive and easy to learn for Java developers**. Most parts of the language are very similar to what we already know, and the differences in basic concepts can be learnt in no time.

* **We have total integration with our daily IDE for free**. Android Studio can understand, compile and run Kotlin code. And the support for this language comes from the company who develops the IDE, so we Android developers are first-class citizens. But this is only related to how the language integrates with our tools.

What are the advantages of the language when compared to Java 7?

* **It’s more expressive**: this is one of its most important qualities. You can write more with much less code.

* **It’s safer**: Kotlin is null safe, which means that we deal with possible null situations in compile time, to prevent execution time exceptions. We need to explicitly specify if an object can be null, and then check its nullity before using it. You will save a lot of time debugging null pointer exception and fixing nullity bugs.

* **It’s functional**: Kotlin is basically an object oriented language, not a pure functional language. However, as many other modern languages, it uses many concepts from functional programming, such as lambda expressions, to resolve some problems in a much easier way. Another nice feature is the way it deals with collections.

* **It makes use of extension functions**: This means we can extend any class with new features even if we don’t have access to the its source code.

* **It’s highly interoperable**: You can continue using most libraries and code written in Java, because the interoperability between both languages is excellent. It’s even possible to create mixed project, with both Kotlin and Java files coexisting.

<!--more-->

### What do we get with Kotlin?

#### Expresiveness

With Kotlin, it’s much easier to avoid boilerplate because most common situations are covered by default in the language. For instance, in Java, if we want to create a data class, we’ll need to write (or at least generate) this code:

```java
public class Artist {
    private long id;
    private String name;
    private String url;
    private String mbid;

    public long getId() {
        return id;
    }


    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getMbid() {
        return mbid;
    }

    public void setMbid(String mbid) {
        this.mbid = mbid;
    }

    @Override public String toString() {
        return "Artist{" +
          "id=" + id +
          ", name='" + name + '\'' +
          ", url='" + url + '\'' +
          ", mbid='" + mbid + '\'' +
          '}';
    }
}
```

With Kotlin, you just need to make use of a data class:

```Kotlin
data class Artist(
    var id: Long,
    var name: String,
    var url: String,
    var mbid: String)
```

This data class auto-generates all the fields and property accessors, as well as some useful methods such as toString().

#### Null Safety

When we develop using Java, a big part of our code is defensive. We need to check continuously whether something is null before using it if we don’t want to find unexpected _NullPointerException_. Kotlin, as many other modern languages, is null safe because we need to explicitly specify if an object can be null by using the safe call operator (written ?).

We can do things like this:

```Kotlin
// This won't compile. Artist can't be null
var notNullArtist: Artist = null

// Artist can be null
var artist: Artist? = null

// Won't compile, artist could be null and we need to deal with that
artist.print()

// Will print only if artist != null
artist?.print()

// Smart cast. We don't need to use safe call operator if we previously
// checked nullity
if (artist != null) {
artist.print()
}

// Only use it when we are sure it's not null. Will throw an exception otherwise.
artist!!.print()

// Use Elvis operator to give an alternative in case the object is null.
val name = artist?.name ?: "empty"

```

#### Extension functions

We can add new functions to any class. It’s a much more readable substitute to the usual utility classes we all have in our projects. We could, for instance, add a new method to fragments to show a toast:

```Kotlin
fun Fragment.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
  Toast.makeText(getActivity(), message, duration).show()
}

```

We can now do:

```Kotlin
fragment.toast("Hello world!")
```


#### Functional support (Lambdas)

What if, instead of having to implement an anonymous class every time we need to implement a click listener, we could just define what we want to do? We can indeed. This (and many more
interesting things) is what we get thanks to lambdas:

```Kotlin
view.setOnClickListener { toast("Hello world!") }
```

This is only a small selection of what Kotlin can do to simplify your code. Now that you know some of the many interesting features of the language, you may decide this is not for you. If you continue, we’ll start with the practice right away in the next chapter.
