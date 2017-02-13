---
title: Consider static factory methods instead of constructors
date: 2017-02-12 21:59:14
categories: "Effective Java"
---

The normal way for a class to allow a client to obtain an instance of itself is to provide a public constructor. There is another technique that should be a part of every programmer’s toolkit. A class can provide a public static factory method, which is simply a static method that returns an instance of the class. Here’s a simple example from Boolean (the boxed primitive class for the primitive type boolean). This method translates a boolean primitive value into a Boolean object reference:

对于类而言，为了让客户端获取它自身的一个实例，最常用的方法就是提供一个公有的构造器。还有一种方法，也应该在每个程序员的工具箱中占有一席之地。类可以提供一个公有的静态工厂方法，它只是一个返回类的实例的静态方法。下面是一个来自Boolean（基本数据类型boolean的包装类）的简单示例。这个方法将boolean基本类型值转换成了一个Boolean对象引用：

```java

    public static Boolean valueOf(boolean b) {
      return b ? Boolean.TRUE : Boolean.FALSE;
    }

```

Note that a static factory method is not the same as the Factory Method pattern
from Design Patterns [Gamma95, p. 107]. The static factory method described in
this item has no direct equivalent in Design Patterns.

这里需要注意：静态工厂方法和设计模式[Gamma95, p. 107]中的工厂方法模式不同。本条目所指的静态工厂方法并不直接对应于设计模式中的工厂方法。

A class can provide its clients with static factory methods instead of, or in
addition to, constructors. Providing a static factory method instead of a public
constructor has both advantages and disadvantages.

类可以通过静态工厂方法来提供给它的客户端，而不是通过构造器。提供静态工厂方法而不是公有的构造器，这样做有优点也有缺点。

<!--more-->

**One advantage of static factory methods is that, unlike constructors, they
have names**. If the parameters to a constructor do not, in and of themselves,
describe the object being returned, a static factory with a well-chosen name is easier
to use and the resulting client code easier to read. For example, the constructor
BigInteger(int, int, Random), which returns a BigInteger that is probably
prime, would have been better expressed as a static factory method named BigInteger.probablePrime.
(This method was eventually added in the 1.4 release.)

**静态工厂方法与构造器不同的第一大优势在于，它们有名称**。如果构造器的参数本身没有确切地描述正被返回的对象，那么具有适当名称的静态工厂会更容易使用，产生的客户端代码也更容易阅读。例如，构造器BigInteger(int, int, Random)返回的BigInteger可能为素数，如果用名为BigInteger.probablePrime的静态工厂方法来表示，显然更为清楚（Java 1.4的发行版本最终增加了这个方法）。

A class can have only a single constructor with a given signature. Programmers
have been known to get around this restriction by providing two constructors
whose parameter lists differ only in the order of their parameter types. This is a
really bad idea. The user of such an API will never be able to remember which
constructor is which and will end up calling the wrong one by mistake. People
reading code that uses these constructors will not know what the code does without
referring to the class documentation.

一个类可只能有一个带有指定签名的构造器。程序员通常知道如何避开这一限制：通过提供两个构造器，它们的参数列表只在参数类型的顺序上有所不同。实际上这并不是个好主意，面对这样的API，用户永远也记不住该用哪个构造器，结果常常会调用错误的构造器。当读到使用了这些构造器的代码时，如果没有参考类的说明文档，往往不知所云。

Because they have names, static factory methods don’t share the restriction
discussed in the previous paragraph. In cases where a class seems to require multiple
constructors with the same signature, replace the constructors with static factory
methods and carefully chosen names to highlight their differences.

由于静态工厂方法有名称，所以它们不受上述的限制。当一个类需要多个带有相同签名的构造器时，就用静态工厂方法替代构造器，并且慎重地选择名称以便突出它们之间的区别。

**A second advantage of static factory methods is that, unlike constructors,
they are not required to create a new object each time they’re invoked**. This
allows immutable classes (Item 15) to use preconstructed instances, or to cache
instances as they’re constructed, and dispense them repeatedly to avoid creating
unnecessary duplicate objects. The Boolean.valueOf(boolean) method illustrates
this technique: it never creates an object. This technique is similar to the
Flyweight pattern [Gamma95, p. 195]. It can greatly improve performance if
equivalent objects are requested often, especially if they are expensive to create.

**静态工厂方法与构造器的第二大优势在于，不必在每次调用它们的时候都创建一个新对象**。这使得不可变类[见第15条]可以使用预先构建好的实例，或者将构建好的实例缓存起来，进行重复利用，从而避免创建不必要的重复对象。Boolean.valueOf(boolean)方法说明了这项技术：它从不创建对象。这样方法类似于Flyweigh模式[Gamma95, p. 195]。如果程序经常请求创建相同的对象，并且创建对象的代价很高，则这项技术可以极大地提升性能。

The ability of static factory methods to return the same object from repeated
invocations allows classes to maintain strict control over what instances exist at
any time. Classes that do this are said to be instance-controlled. There are several
reasons to write instance-controlled classes. Instance control allows a class to
guarantee that it is a singleton (Item 3) or noninstantiable (Item 4). Also, it allows
an immutable class (Item 15) to make the guarantee that no two equal instances
exist: a.equals(b) if and only if a==b. If a class makes this guarantee, then its clients
can use the == operator instead of the equals(Object) method, which may
result in improved performance. Enum types (Item 30) provide this guarantee.

静态工厂方法能够为重复的调用返回相同对象，这样有助于类总能严格控制在某个时刻哪些实例应该存在。这种类被称作实例受控的类。编写实例受控的类有几个原因：实例受控使得类可以确保它是一个Singleton[见第三条]或者是不可实例化的[见第4条]。它还使得不可变的类[见第15条]可以确保不会存在两个相等的实例，即当且仅当a==b的时候才有a.equals(b).如果类保证了这一点，它的客户端就可以使用==操作符来替代equals(Object)方法，这样可以提升性能。枚举（Enum）类型[见第30条]保证了这一点。

**A third advantage of static factory methods is that, unlike constructors,
they can return an object of any subtype of their return type**. This gives you
great flexibility in choosing the class of the returned object.

**静态工厂方法与构造器不同的第三大优势在于，它们可以返回原返回类型的任何子类型的对象**。这样我们在选择返回对象的类时就有了更大的灵活性。


One application of this flexibility is that an API can return objects without
making their classes public. Hiding implementation classes in this fashion leads to
a very compact API. This technique lends itself to interface-based frameworks
(Item 18), where interfaces provide natural return types for static factory methods.
Interfaces can’t have static methods, so by convention, static factory methods for
an interface named Type are put in a noninstantiable class (Item 4) named Types.

这种灵活性的一种应用是，API可以返回对象，同时又不会使对象的类变成公有的。以这种方式隐藏实现类会使API变得十分简洁。这项技术适用于基于接口的框架[见第18条]，因为在这种框架中，接口为静态工厂方法提供了自然返回类型。接口不能有静态方法，因此，按照惯例，接口Type的静态工厂方法被放在一个名为Types的不可实例化的类[见第4条]中。

For example, the Java Collections Framework has thirty-two convenience
implementations of its collection interfaces, providing unmodifiable collections,
synchronized collections, and the like. Nearly all of these implementations are
exported via static factory methods in one noninstantiable class (java.util.Collections).
The classes of the returned objects are all nonpublic.

例如，Java Collections Framework的集合接口有32个便利实现，分别提供了不可修改的集合，同步集合等等。几乎所有这些实现都通过一个不可实例化的类的静态工厂方法来提供。所有返回对象的类都是非公有的。

The Collections Framework API is much smaller than it would have been had
it exported thirty-two separate public classes, one for each convenience implementation.
It is not just the bulk of the API that is reduced, but the conceptual
weight. The user knows that the returned object has precisely the API specified by
its interface, so there is no need to read additional class documentation for the
implementation classes. Furthermore, using such a static factory method requires
the client to refer to the returned object by its interface rather than its implementation
class, which is generally good practice (Item 52).

现在的Collections Framework API比提供32个独立公有类的那种实现方式要小得多，每种便利实现都去对应一个类。这不仅仅是指API数量上的减少，也是概念意义上的减少。用户知道，被返回的对象是由相关的接口精确指定的，所以他们不需要阅读有关的类文档。使用这种静态工厂方法时，甚至要求客户端通过接口来引用被返回的对象，而不是通过它的实现类来引用被返回的对象，这是一种良好的习惯[见第52条]。

Not only can the class of an object returned by a public static factory method
be nonpublic, but the class can vary from invocation to invocation depending on
the values of the parameters to the static factory. Any class that is a subtype of the
declared return type is permissible. The class of the returned object can also vary
from release to release for enhanced software maintainability and performance.

公有的静态工厂方法所返回的对象的类不仅可以是非公有的，而且该类还可以随着每次调用而发生变化，这取决于静态工厂方法的参数值。只要是已声明的返回类型的子类型，都是允许的。为了提升软件的可维护性和性能，返回对象的类也可以随着发行版本的不同而不同。

The class java.util.EnumSet (Item 32), introduced in release 1.5, has no
public constructors, only static factories. They return one of two implementations,
depending on the size of the underlying enum type: if it has sixty-four or fewer
elements, as most enum types do, the static factories return a RegularEnumSet
instance, which is backed by a single long; if the enum type has sixty-five or more
elements, the factories return a JumboEnumSet instance, backed by a long array.

发行版本1.5中引入的类java.util.EnumSet[见第32条]没有公有构造器，只有静态工厂方法。它们返回两种实现之一，具体则取决于底层枚举类型的大小：如果它的元素有64个或者更少，就像大多数枚举类型一样，静态工厂方法就会返回一个RegalarEnumSet实例，用单个long进行支持；如果枚举类型有65个或者更多元素，工厂就返回JumboEnumSet实例，用long数组进行支持。

The existence of these two implementation classes is invisible to clients. If
RegularEnumSet ceased to offer performance advantages for small enum types, it
could be eliminated from a future release with no ill effects. Similarly, a future
release could add a third or fourth implementation of EnumSet if it proved beneficial
for performance. Clients neither know nor care about the class of the object
they get back from the factory; they care only that it is some subclass of EnumSet.

这两种实现类的存在对于客户端来说是不可见的。如果RegularEnumSet不能再给小的枚举类型提供性能优势，就可能从未来的发行版本将它删除，不会造成不良的影响。同样地，如果事实证明对性能有好处，也可能在未来的发行版本中添加第三甚至第四个EnumSet实现。客户端永远不知道也不关心他们从工厂方法中得到的对象的类，他们只关心它是EnumSet的某个子类即可。

The class of the object returned by a static factory method need not even exist
at the time the class containing the method is written. Such flexible static factory
methods form the basis of service provider frameworks, such as the Java Database
Connectivity API (JDBC). A service provider framework is a system in which
multiple service providers implement a service, and the system makes the implementations
available to its clients, decoupling them from the implementations.

静态工厂方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不必存在。这种灵活的静态工厂方法构成了服务提供者框架的基础，例如JDBC（Java数据库连接，Java Database Connectivity ）API。服务提供者框架是指这样一个系统：多个服务提供者实现一个服务，系统为服务提供者的客户端提供多个实现，并把他们从多个实现中解耦出来。

There are three essential components of a service provider framework: a service
interface, which providers implement; a provider registration API, which the
system uses to register implementations, giving clients access to them; and a service
access API, which clients use to obtain an instance of the service. The service
access API typically allows but does not require the client to specify some criteria
for choosing a provider. In the absence of such a specification, the API returns an
instance of a default implementation. The service access API is the “flexible static
factory” that forms the basis of the service provider framework.

服务提供者框架中有三个重要的组件:服务接口，这是提供者实现的；提供者注册API，这是系统用来注册实现，让客户端访问他们的；服务访问API，是客户端用来获取服务的实例。服务访问API一般允许但是不要求客户端指定某种选择提供者的条件。如果没有这样的规定，API就会返回默认实现的一个实例。服务访问API是“灵活的静态工厂”，它构成了服务提供者框架的基础。

An optional fourth component of a service provider framework is a service
provider interface, which providers implement to create instances of their service
implementation. In the absence of a service provider interface, implementations
are registered by class name and instantiated reflectively (Item 53). In the case of
JDBC, Connection plays the part of the service interface, DriverManager.registerDriver
is the provider registration API, DriverManager.getConnection is
the service access API, and Driver is the service provider interface.

服务提供者框架的第四个组件是可选的：服务提供者接口，这些提供者负责创建其服务实现的实例。如果没有服务提供者接口，实现就按照类名注册，并通过反射方式进行实例化[见第53条]。对于JDBC来说，Connection就是它的服务接口，DriverManager.registerDriver是提供者注册API，DriverManager.getConnection是服务访问API，Driver就是服务提供者接口。

There are numerous variants of the service provider framework pattern. For
example, the service access API can return a richer service interface than the one
required of the provider, using the Adapter pattern [Gamma95, p. 139]. Here is a
simple implementation with a service provider interface and a default provider:

服务提供者框架模式有无数种变体。例如，服务访问API可以利用适配器模式[Gamma95, p. 139]，返回比提供者需要的更丰富的服务接口。下面是一个简单的实现，包含一个服务提供者接口和一个默认提供者：

```java
// Service provider framework sketch
// Service interface
public interface Service {
  ... // Service-specific methods go here
}
// Service provider interface
public interface Provider {
  Service newService();
}
// Noninstantiable class for service registration and access
public class Services {
  private Services() { } // Prevents instantiation (Item 4)
  // Maps service names to services
  private static final Map<String, Provider> providers =
                      new ConcurrentHashMap<String, Provider>();
  public static final String DEFAULT_PROVIDER_NAME = "<def>";
  // Provider registration API
  public static void registerDefaultProvider(Provider p) {
    registerProvider(DEFAULT_PROVIDER_NAME, p);
  }
  public static void registerProvider(String name, Provider p){
    providers.put(name, p);
  }
  // Service access API
  public static Service newInstance() {
    return newInstance(DEFAULT_PROVIDER_NAME);
  }
  public static Service newInstance(String name) {
    Provider p = providers.get(name);
    if (p == null){
      throw new IllegalArgumentException(
                "No provider registered with name: " + name);
    }
    return p.newService();
  }
}

```
**A fourth advantage of static factory methods is that they reduce the verbosity
of creating parameterized type instances**. Unfortunately, you must specify
the type parameters when you invoke the constructor of a parameterized class
even if they’re obvious from context. This typically requires you to provide the
type parameters twice in quick succession:

**静态工厂方法的第四大优势在于，在创建参数化类型实例的时候，它们使代码变得更加简单**。遗憾的是，在调用参数化类的构造器时，即使类型参数很明显，也必须指明。这通常要求你接连两次提供类型参数：

```java
Map<String, List<String>> m =
        new HashMap<String, List<String>>();
```
This redundant specification quickly becomes painful as the length and complexity
of the type parameters increase. With static factories, however, the compiler
can figure out the type parameters for you. This is known as type inference. For
example, suppose that HashMap provided this static factory:

随着类型参数变得越来越长，越来越复杂，这一冗长的说明也很快变得痛苦起来。但是有了静态工厂方法，编译器就可以替你找到类型参数。这被称作类型推导。例如，假设HashMap提供了这个静态工厂：

```java
public static <K, V> HashMap<K, V> newInstance() {
      return new HashMap<K, V>();
}
```

Then you could replace the wordy declaration above with this succinct alternative:

你就可以用下面这句简洁的代码代替上卖弄这段繁琐的声明：

```java
Map<String, List<String>> m = HashMap.newInstance();
```

Someday the language may perform this sort of type inference on constructor
invocations as well as method invocations, but as of release 1.6, it does not.

总有一天，Java将能够在构造器调用以及方法调用中执行这种类型推导，但到发行版本1.6为止暂时还无法这么做。

Unfortunately, the standard collection implementations such as HashMap do
not have factory methods as of release 1.6, but you can put these methods in your
own utility class. More importantly, you can provide such static factories in your
own parameterized classes.

遗憾的是，到发行版本1.6为止，标准的集合实现如HashMap并没有工厂方法，但是可以把这些方法放在你自己的工具类中。更重要的是，可以把这样的静态工厂放在你自己的参数化的类中。

**The main disadvantage of providing only static factory methods is that
classes without public or protected constructors cannot be subclassed**. The
same is true for nonpublic classes returned by public static factories. For example,
it is impossible to subclass any of the convenience implementation classes in the
Collections Framework. Arguably this can be a blessing in disguise, as it encourages
programmers to use composition instead of inheritance (Item 16).

**静态工厂方法的主要缺点在于，类如果不含公有的或者受保护的构造器，就不能被子类化**。对于公有的静态工厂所返回的非公有类，也同样如此。例如，要想将Collections Framework中的任何方便的实现类子类化，这是不可能的。但是这样也许会因祸得福，因为它鼓励程序员使用复合，而不是继承[见第16条]。

**A second disadvantage of static factory methods is that they are not
readily distinguishable from other static methods**. They do not stand out in API
documentation in the way that constructors do, so it can be difficult to figure out
how to instantiate a class that provides static factory methods instead of constructors.
The Javadoc tool may someday draw attention to static factory methods. In
the meantime, you can reduce this disadvantage by drawing attention to static factories
in class or interface comments, and by adhering to common naming conventions.
Here are some common names for static factory methods:

**静态工厂方法的第二个缺点在于，它们与其他的静态方法实际上没有任何区别**。在API文档中，它们没有像构造器那样在API文档中明确标识出来，因此，对于提供了静态工厂方法而不是构造器的类来说，要想查明如何实例化一个类，这是非常困难的.javadoc工具总有一天会注意到静态工厂方法。同时，你通过在类或者接口注释中关注静态工厂，并遵守标准的命名习惯，也可以弥补这一劣势。下面是静态工厂方法的一些惯用名称：

  * valueOf—Returns an instance that has, loosely speaking, the same value as its
parameters. Such static factories are effectively type-conversion methods.

  * valueOf-不太严格地讲，该方法返回的实例与它的参数具有相同的值。这样的静态工厂方法实际上是类型转换方法。

  * of—A concise alternative to valueOf, popularized by EnumSet (Item 32).

  * of-valueOf的一种更为简洁的替代，在EnumSet[见第32条]中使用并流行起来。

  * getInstance—Returns an instance that is described by the parameters but
cannot be said to have the same value. In the case of a singleton, getInstance
takes no parameters and returns the sole instance.

  * getInstance-返回的实例是通过方法的参数来描述的，但是不能够说与参数具有同样的值。对于Singleton来说，该方法没有参数，并返回唯一的实例。

  * newInstance—Like getInstance, except that newInstance guarantees that
each instance returned is distinct from all others.

  * newInstance—像getInstance一样，但newInstance能够确保返回的每个实例都与所有其他实例不同。

  * getType—Like getInstance, but used when the factory method is in a different
class. Type indicates the type of object returned by the factory method.

  * getType—像getInstance一样，但是在工厂方法处于不同的类中的时候使用。Type表示工厂方法所返回的对象类型。

  * newType—Like newInstance, but used when the factory method is in a different
class. Type indicates the type of object returned by the factory method.

  * newType—像newInstance一样，但是在工厂方法处于不同的类中的时候使用。Type表示工厂方法所返回的对象类型。

In summary, static factory methods and public constructors both have their
uses, and it pays to understand their relative merits. Often static factories are preferable,
so avoid the reflex to provide public constructors without first considering
static factories.

简而言之，静态工厂方法和公有构造器都各有好处，我们需要理解它们各自的长处。静态工厂通常更加合适，因此切忌第一反应就是提供公有的构造器，而不先考虑静态工厂。
