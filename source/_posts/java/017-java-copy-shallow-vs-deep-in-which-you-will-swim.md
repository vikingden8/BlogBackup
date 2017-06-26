---
title: Shallow vs Deep Copy in Java
date: 2017-06-26 22:00:51
tags:
categories: "Java学习笔记"
---

### What Is a Copy? 什么是拷贝？

To begin, I’d like to highlight what a copy in Java is. First, let’s differentiate between a _reference copy_ and an object copy. A reference copy, as the name implies, creates a copy of a reference variable pointing to an object. If we have a _Car_ object, with a _myCar_ variable pointing to it and we make a reference copy, we will now have two _myCar_ variables, but still one object.

在开始之前，我先强调下Java中的拷贝是什么。首先，让我们对引用拷贝和对象拷贝进行一下区分。 **引用拷贝**, 正如它的名称所表述的意思, 就是创建一个指向对象的引用变量的拷贝。如果我们有一个 _Car_ 对象，而且让 _myCar_ 变量指向这个变量，这时候当我们做引用拷贝，那么现在就会有两个 _myCar_ 变量，但是对象仍然只存在一个。

![](/images/categories/java/017/01.jpg)

An _object copy_ creates a copy of the object itself. So if we again copied our _car_ object, we would create a copy of the object itself, as well as a second reference variable referencing that copied object.

**对象拷贝** 会创建对象本身的一个副本。因此如果我们再一次服务我们 _car_ 对象，就会创建这个对象本身的一个副本, 同时还会有第二个引用变量指向这个被复制出来的对象。

![](/images/categories/java/017/02.jpg)

<!--more-->

### What Is an Object? 什么是对象?

Both a Deep Copy and a Shallow Copy are types of object copies, but what really is an object? Often, when we talk about an object, we speak of it as a single unit that can’t be broken down further, like a humble coffee bean. However, that’s oversimplified.

深拷贝和浅拷贝都是对象拷贝, 但一个对象实际是什么呢? 当我们谈论到对象时，我们经常会说它就像一粒浑圆的咖啡豆，已经是一个不能够被进一步分解的单位了，但这种说法太过于简化了。

![](/images/categories/java/017/03.jpg)

Say we have a _Person_ object. Our _Person_ object is in fact composed of other objects, as you can see in Example . Our _Person_ contains a _Name_ object and an _Address_ object. The _Name_ in turn, contains a _FirstName_ and a _LastName_ object; the _Address_ object is composed of a _Street_ object and a _City_ object. So when I talk about _Person_ in this article, I’m actually talking about this entire network of objects.

比方说我们有一个 _Person_ 对象。这个 _Person_ 对象实际上是由其它的对象组合而成的。如示例所示， _Person_ 对象包含了一个 _Name_ 对象和一个 _Address_ 对象。_Nam_e 对象又包含了一个 _FirstNam_e 对象和一个 _LastName_ 对象；_Address_ 对象又是由一个 _Street_ 对象以及一个 _City_ 对象组合而成的。那么当我们讨论本文中的这个 _Person_ 时，实际上我是在讨论这些个对象所组成的整个的对象联系网络。

![](/images/categories/java/017/04.jpg)

So why would we want to copy this _Person_ object? An object copy, usually called a clone, is created if we want to modify or move an object, while still preserving the original object. There are many different ways to copy an object that you can learn about in another article. In this article we’ll specifically be using a copy constructor to create our copies.

那么为什么我会要对这个 _Person_ 对象进行拷贝呢? 对象复制，经常也会被称作克隆，它是在我们想要修改或者移除某个对象，但仍然想要保留原来的那个对象时所要进行的操作。在另外一篇文章中你可以了解到许多拷贝一个对象的不同方法。在本文中我们将特别讲到如何利用拷贝构造器来创建拷贝。

### Shallow Copy

First let’s talk about the shallow copy. A shallow copy of an object copies the ‘main’ object, but doesn’t copy the inner objects. The ‘inner objects’ are shared between the original object and its copy. For example, in our _Person_ object, we would create a second _Person_, but both objects would share the same _Name_ and _Address_ objects.

首先让我们来说说浅拷贝。对象的浅拷贝会对“主”对象进行拷贝，但不会复制主对象里面的对象。"里面的对象“会在原来的对象和它的副本之间共享。例如，我们会为一个 _Person_ 对象创建第二个 _Person_ 对象, 而两个 _Person_ 会共享相同的 _Name_ 和 _Address_ 对象。

Let’s look at a coding example. In Example, we have our class _Person_, which contains a _Name_ and _Address_ object. The copy constructor takes the _originalPerson_ object and copies its reference variables.

让我们来看看代码示例。在示例中，我们有一个类 _Person_，类里面包含了一个 _Name_ 和 _Address_ 对象。拷贝构造器会拿到 _originalPerson_ 对象，然后对其引用变量进行复制。

```java
public class Person {
    private Name name;
    private Address address;

    public Person(Person originalPerson) {
         this.name = originalPerson.name;
         this.address = originalPerson.address;
    }
}
```

The problem with the shallow copy is that the two objects are not independent. If you modify the _Name_ object of one _Person_, the change will be reflected in the other _Person_ object.

浅拷贝的问题就是两个对象并非独立的。如果你修改了其中一个 _Person_ 对象的 _Name_ 对象，那么这次修改也会影响另外一个 _Person_ 对象。

Let’s apply this to an example. Say we have a _Person_ object with a reference variable _mother_; then, we make a copy of mother, creating a second _Person_ object, _son_. If later on in the code, the _son_ tries to _moveOut()_ by modifying his _Address_ object, the _mother_ moves with him!

让我们在示例中看看这个问题。假如说我们有一个 _Person_ 对象，然后也会有一个引用变量 _monther_ 来指向它；然后当我们对 _mother_ 进行拷贝时，创建第二个 _Person_ 对象 _son_。如果在此后的代码中， _son_ 尝试用 _moveOut()_ 来修改他的 _Address_ 对象, 那么 _mother_ 也会跟着他一起搬走!

```java
Person mother = new Person(new Name(…), new Address(…));
Person son  = new Person(mother);
son.moveOut(new Street(…), new City(…));
```

This occurs because our _mother_ and _son_ objects share the same _Address_ object, as you can see illustrated in Example. When we change the _Address_ in one object, it changes in both!

这种现象之所以会发生，是因为 _mother_ 和 _son_ 对象共享了相同的 _Address_ 对象，如你在示例中所看到的描述。当我们在一个对象中修改了 _Address_ 对象，那么也就表示两个对象总的 _Address_ 都被修改了。

![](/images/categories/java/017/05.jpg)

### Deep Copy

Unlike the shallow copy, a deep copy is a **fully independent copy of an object**. If we copied our _Person_ object, we would copy the entire object structure.

不同于浅拷贝，深拷贝是一个 **整个独立的对象拷贝**。如果我们对整个 _Person_ 对象进行深拷贝，我们会对整个对象的结构都进行拷贝。

![](/images/categories/java/017/06.jpg)

A change in the _Address_ object of one _Person_ wouldn’t be reflected in the other object as you can see by the diagram in Example. If we take a look at the code in example below, you can see that we’re not only using a copy constructor on our _Person_ object, but we are also utilizing copy constructors on the inner objects as well.

如你在示例中所见，对一个 _Person_ 的 _Address_ 对象进行了修改并不会对另外一个对象造成影响。当我们观察下面示例中的代码，会发现我们不单单对 _Person_ 对象使用了拷贝构造器，同时也会对里面的对象使用拷贝构造器。

```java
public class Person {
    private Name name;
    private Address address;

    public Person(Person otherPerson) {
         this.name    =  new Name(otherPerson.name);
         this.address =  new Address(otherPerson.address);
    }
}
```

Using this deep copy, we can retry the _mother-son_ example . Now the _son_ is able to successfully move out!

使用这种深拷贝，我们可以重新尝试上面示例中的 _mother-son_ 这个用例。现在 _son_ 可以成功的搬走了！

However, that’s not the end of the story. To create a true deep copy, we need to keep copying all of the _Person_ object’s nested elements, until there are only primitive types and “Immutables” left. Let’s look at the _Street_ class to better illustrate this:

不过，故事到这儿并没有结束。要创建一个真正的深拷贝，就需要我们一直这样拷贝下去，一直覆盖到 _Person_ 对象所有的内部元素, 最后只剩下原始的类型以及“不可变对象（Immutables）”。让我们观察下如下这个 _Street_ 类以获得更好的理解:

```java
public class Street {
    private String name;
    private int number;

    public Street(Street otherStreet){
         this.name = otherStreet.name;
         this.number = otherStreet.number;
    }
}
```

The _Street_ object is composed of two instance variables – _String name_ and _int number_. _int number_ is a primitive value and not an object. It’s just a simple value that can’t be shared, so by creating a second instance variable, we are automatically creating an independent copy. _String_ is an _Immutable._ In short, **an Immutable is an Object, that, once created, can never be changed again**. Therefore, you can share it without having to create a deep copy of it.

Street 对象有两个实体变量组成 – String 类型的 name 以及 int 类型的 number。int  类型的 number 是一个原始类型，并非对象。它只是一个简单的值，不能共享, 因此在创建第二个实体变量时，我们可以自动创建一个独立的拷贝。String 是一个不可变对象（Immutable）。简言之，不可变对象也是对象，可一旦创建好了以后就再也不能被修改了。因此，你可以不用为其创建深拷贝就能对其进行共享。

### Conclusion 总结

To conclude, I’d like to talk about some coding techniques we used in our mother-son example. Just because a deep copy will let you change the internal details of an object, such as the _Address_ object, it doesn’t mean that you should. Doing so would decrease code quality, as it would make the _Person_ class more fragile to changes – whenever the _Address_ class is changed, you will have to (potentially) apply changes to the _Person_ class also. For example, if the _Address_ class no longer contains a _Street_ object, we’d have to change the _moveOut()_ method in the _Person_ class on top of the changes we already made to the _Address_ class.

作为总结，我要说说上面在 mother-son 示例中所用到的一些编码技术。只是因为深拷贝可以让你修改一个对象里面的详细信息，比如 Address 对象，这并不意味着你就该这样做。这样做会提高代码的质量, 因为它可以使得 Person 更容易修改 – 不管 Address 类什么时候被修改了，你也都会要修改应用到 Person 类。例如，如果 Address 类型不再包含 Street 对象了，我们就得根据已经对 Address 类做出的修改来对Person 类中的 moveOut()  方法进行修改。

In Example of this article I only chose to use a new _Street_ and _City_ object to better illustrate the difference between a shallow and a deep copy. Instead, I would recommend that you assign a new _Address_ object instead, effectively converting to a hybrid of a shallow and a deep copy, as you can see in Example below:

在本文的示例中，我只选择使用了一个新的 Street 和 City 对象，这样可以更好的对浅拷贝和深拷贝的不同之处进行描述。不过，我会建议你给方法分配一个新的 Address 对象，这样能有效的将其转换成一个浅拷贝和深拷贝的混合体, 见示例 10：

```java
Person mother = new Person(new Name(…), new Address(…));
Person son  = new Person(mother);
son.moveOut(new Address(...));
```

In object-oriented terms, this violates encapsulation, and therefore should be avoided. Encapsulation is one of the most important aspects of Object Oriented programming. In this case, I had violated encapsulation by accessing the internal details of the _Address_ object in our _Person_ class. This harms our code because we have now entangled the _ class in the _Address_ class and if we make changes to the _Address_ class down the line, it could harm the _Person_ class as I explained above. While you obviously need to interconnect your various classes to have a coding project, whenever you connect two classes, you need to analyze the costs and benefits.

在面向对象领域，这样做违背了封装的原则，因此应该被避免。封装是面向对象编程中一个最重要的方面。在这里，我已经违背封装的原则，对 Person 类中 Address 对象的内部细节进行了访问。这样做对我们的代码造成了伤害，因为我们现在跟 Person 类中的 Address 类纠缠在一起，如果对 Address 类进行了修改，就会如我上面所解释的对代码造成伤害。不过是你显然是会需要将你定义的各种类互相联系在一起以构成代码工程的，但在你要将两个类联系在一起时，需要好好分析一下成本和收益。

From:[Shallow vs. Deep Copy in Java](https://dzone.com/articles/java-copy-shallow-vs-deep-in-which-you-will-swim)
