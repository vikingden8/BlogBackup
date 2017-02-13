---
title: Consider a builder when faced with many constructor parameters
date: 2017-02-13 20:04:56
categories: "Effective Java"
---

Static factories and constructors share a limitation: they do not scale well to large
numbers of optional parameters. Consider the case of a class representing the
Nutrition Facts label that appears on packaged foods. These labels have a few
required fields—serving size, servings per container, and calories per serving—
and over twenty optional fields—total fat, saturated fat, trans fat, cholesterol,
sodium, and so on. Most products have nonzero values for only a few of these
optional fields.

静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。考虑用一个类表示包装食品外面显示的营养成分标签。这些标签中有几个是必需的：每份的含量、每罐的含量以及每份的卡路里，还有超过20个可选成分：总脂肪量、饱和脂肪量、转化脂肪、胆固醇、钠等等。大多数产品在某几个可选成分中都会有非零的值。

What sort of constructors or static factories should you write for such a class?
Traditionally, programmers have used the telescoping constructor pattern, in
which you provide a constructor with only the required parameters, another with a
single optional parameter, a third with two optional parameters, and so on, culminating
in a constructor with all the optional parameters. Here’s how it looks in
practice. For brevity’s sake, only four optional fields are shown:

对于这样的类，应该用哪种构造器或者静态方法来编写呢？程序员一向习惯采用重叠构造器模式，在这种模式下，你提供第一个只有必要参数的构造器，第二个构造器有一个可选参数，第三个有两个可选参数，依此类推，最后一个构造器包含所有可选参数。下面有个示例，为了简单起见，它只显示四个可选域：

<!--more-->

```java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
    private final int servingSize; // (mL) required
    private final int servings; // (per container) required
    private final int calories; // optional
    private final int fat; // (g) optional
    private final int sodium; // (mg) optional
    private final int carbohydrate; // (g) optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                                        int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                                        int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                      int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

When you want to create an instance, you use the constructor with the shortest
parameter list containing all the parameters you want to set:

当你想要创建实例的时候，就利用参数列表最短的构造器，但该列表中包含了要设置的所有参数：

```java
  NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

Typically this constructor invocation will require many parameters that you don’t
want to set, but you’re forced to pass a value for them anyway. In this case, we
passed a value of 0 for fat. With “only” six parameters this may not seem so bad,
but it quickly gets out of hand as the number of parameters increases.

这个构造器调用通常需要许多你本不想设置的参数，但还是不得不为它们传递值。在这个例子中，我们给fat传递了一个值为0。如果仅仅是这6个参数，看起来还不算太糟，问题是随着参数数目的增加，它很快就失去了控制。

In short,**the telescoping constructor pattern works, but it is hard to write
client code when there are many parameters, and harder still to read it**. The
reader is left wondering what all those values mean and must carefully count
parameters to find out. Long sequences of identically typed parameters can cause
subtle bugs. If the client accidentally reverses two such parameters, the compiler
won’t complain, but the program will misbehave at runtime (Item 40).

一句话：**重叠构造器模式可行，但是许多参数的时候，客户端代码会很难编写，并且仍然较难以阅读**。如果读者想知道那些值是什么意思，必须仔细地数着这些参数来探个究竟。一长串类型相同的参数会导致一些微妙的错误。如果客户端不小心颠倒了其中两个参数的顺序，编译器也不会出错，但是程序在运行时会出现错误的行为。

A second alternative when you are faced with many constructor parameters is
the JavaBeans pattern, in which you call a parameterless constructor to create the
object and then call setter methods to set each required parameter and each
optional parameter of interest:

遇到许多构造器参数的时候，还有第二种代替方法，即JavaBean模式，在这种模式下，调用一个无参构造器来创建对象，然后调用setter方法来设置每个必要的参数，以及每个相关的可选参数：

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // " " " "
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    public NutritionFacts() { }

    // Setters
    public void setServingSize(int val) { servingSize = val; }

    public void setServings(int val) { servings = val; }

    public void setCalories(int val) { calories = val; }

    public void setFat(int val) { fat = val; }

    public void setSodium(int val) { sodium = val; }

    public void setCarbohydrate(int val) { carbohydrate = val; }

}
```

This pattern has none of the disadvantages of the telescoping constructor pattern.
It is easy, if a bit wordy, to create instances, and easy to read the resulting code:

这种模式弥补了重叠构造器模式的不足。说的明白一点，就是创建实例很容易，这样产生的代码读起来也很容易：

```java
    NutritionFacts cocaCola = new NutritionFacts();
    cocaCola.setServingSize(240);
    cocaCola.setServings(8);
    cocaCola.setCalories(100);
    cocaCola.setSodium(35);
    cocaCola.setCarbohydrate(27);
```

Unfortunately, the JavaBeans pattern has serious disadvantages of its own.
Because construction is split across multiple calls, **a JavaBean may be in an
inconsistent state partway through its construction**. The class does not have
the option of enforcing consistency merely by checking the validity of the constructor
parameters. Attempting to use an object when it’s in an inconsistent state
may cause failures that are far removed from the code containing the bug, hence
difficult to debug. A related disadvantage is that **the JavaBeans pattern precludes
the possibility of making a class immutable (Item 15)**, and requires
added effort on the part of the programmer to ensure thread safety.

遗憾的是，JavaBean模式自身有着很严重的缺点。因为构造过程被分到了几个调用中，在够做过程中JavaBean可能处于不一致的状态。类无法仅仅通过检验构造器参数的有效性来保证一致性。试图使用处于不一致状态的对象，将会导致失败，这种失败与包含错误的代码大相径庭，因为它调试起来十分困难。与此相关的另一点不足在于，JavaBean模式阻止了把类做成不可变的可能[见第15条]，这就需要程序员付出额外的努力来确保它的线程安全。

It is possible to reduce these disadvantages by manually “freezing” the object
when its construction is complete and not allowing it to be used until frozen, but
this variant is unwieldy and rarely used in practice. Moreover, it can cause errors
at runtime, as the compiler cannot ensure that the programmer calls the freeze
method on an object before using it.

当对象的构造完成，并且不允许在解冻之前使用时，通过手工“冻结”对象，可以弥补这些不足，但是这种方式十分不明智，在实践中很少使用。此外，它设置会在运行时导致错误，因为编译器无法确保程序员会在使用之前先在对象上调用freeze方法。

Luckily, there is a third alternative that combines the safety of the telescoping
constructor pattern with the readability of the JavaBeans pattern. It is a form of the
Builder pattern [Gamma95, p. 97]. Instead of making the desired object directly,
the client calls a constructor (or static factory) with all of the required parameters
and gets a builder object. Then the client calls setter-like methods on the builder
object to set each optional parameter of interest. Finally, the client calls a parameterless
build method to generate the object, which is immutable. The builder is a
static member class (Item 22) of the class it builds. Here’s how it looks in practice:

幸运的是，还有第三种替代方法，既能保证像重叠构造器模式那样的可读性，也能保证像JavaBeans模式那么好的安全性。这就是Builder模式[Gamma95, p. 97]的一种形式。不直接生成想要的对象，而是让客户端利用所有必要的参数调用构造器（或者静态工厂），得到一个builder对象。然后客户端在builder对象上调用类似与setter的方法，来设置每个相关的可选参数。最后，客户端调用无参的build方法来生成不可变的对象。这个builder是它构建的类的静态成员类[见第22条]。下面就是它的示例：

```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int carbohydrate = 0;
        private int sodium = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val){
            calories = val;
            return this;
        }

        public Builder fat(int val){
            fat = val;
            return this;
        }

        public Builder carbohydrate(int val){
            carbohydrate = val;
            return this;
        }

        public Builder sodium(int val){
            sodium = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

Note that *NutritionFacts* is immutable, and that all parameter default values
are in a single location. The builder’s setter methods return the builder itself so
that invocations can be chained. Here’s how the client code looks:

注意 *NutritionFacts* 是不可变的，所有的默认参数值都单独放在一个地方。builder的setter方法返回builder本身，以便可以把调用链接起来。下面就是客户端代码：

```java
  NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                  .calories(100).sodium(35).carbohydrate(27).build();
```

This client code is easy to write and, more importantly, to read. **The Builder pattern
simulates named optional parameters as found in Ada and Python**.

这样的客户端代码很容易编写，更为重要的是，易于阅读。**builder模式模拟了具名的可选参数，就像Ada和Python中的一样**.

Like a constructor, a builder can impose invariants on its parameters. The
build method can check these invariants. It is critical that they be checked after
copying the parameters from the builder to the object, and that they be checked on
the object fields rather than the builder fields (Item 39). If any invariants are violated,
the build method should throw an IllegalStateException (Item 60). The
exception’s detail method should indicate which invariant is violated (Item 63).

buidler像构造器一样，可以对其参数强加约束条件。build方法可以检验这些约束条件，将参数从builder拷贝到对象中之后，并在对象域而不是builder域[见第39条]中对它们进行检验，这一点很重要。如果违反了任何约束条件，build方法就应该抛出IllegalStateException异常[见第60条]。异常的详细信息应该显示出违反了哪个约束条件[见第63条]。

Another way to impose invariants involving multiple parameters is to have
setter methods take entire groups of parameters on which some invariant must
hold. If the invariant isn’t satisfied, the setter method throws an IllegalArgumentException.
This has the advantage of detecting the invariant failure as soon
as the invalid parameters are passed, instead of waiting for build to be invoked.

对于多个参数强加约束条件的另一种方法是，用多个setter方法对某个约束条件必须持有的所有参数进行检查。如果该约束条件没有得到满足，setter方法就会抛出IllegalArgumentException异常。这有个好处，就是一旦传递了无效的参数，立即就会发现约束条件失败，而不是等着调用build方法。

A minor advantage of builders over constructors is that builders can have multiple
varargs parameters. Constructors, like methods, can have only one varargs
parameter. Because builders use separate methods to set each parameter, they can
have as many varargs parameters as you like, up to one per setter method.

与构造器相比，builder的微略优势在于，builder可以有多个可变参数。构造器就像方法一样，只能有一个可变参数。因为builder利用单独的方法来设置每个参数，你想要多少个可变参数，它们就可以有多少个，知道每个setter方法都有一个可变参数。

The Builder pattern is flexible. A single builder can be used to build multiple
objects. The parameters of the builder can be tweaked between object creations to
vary the objects. The builder can fill in some fields automatically, such as a serial
number that automatically increases each time an object is created.

builder模式十分灵活，可以利用单个builder构建多个对象。builder的参数可以在创建对象期间进行调整，也可以随着不同的对象而改变。builder可以自动填充某些域，例如每次创建对象时自动增加序列号。

A builder whose parameters have been set makes a fine Abstract Factory
[Gamma95, p. 87]. In other words, a client can pass such a builder to a method to
enable the method to create one or more objects for the client. To enable this
usage, you need a type to represent the builder. If you are using release 1.5 or a
later release, a single generic type (Item 26) suffices for all builders, no matter
what type of object they’re building:

设置了参数的builder生成了一个很好的抽象工厂[Gamma95, p. 87]。换句话说，客户端可以将这样一个builder传给方法，使该方法能够为客户端创建一个或者多个对象。要使用这种用法，需要有个类型来表示builder。如果使用的使发行版本1.5或者更新的版本，只要一个泛型[见第26条]就能满足所有的builder，无论它们在构建哪种类型的对象：

```java
// A builder for objects of type T
public interface Builder<T> {
    public T build();
}
```

Note that our *NutritionFacts.Builder* class could be declared to implement
*Builder<NutritionFacts>*.

注意，可以声明 *NutritionFacts.Builder* 类来实现 *Builder<NutritionFacts>* 。

Methods that take a Builder instance would typically constrain the builder’s
type parameter using a bounded wildcard type (Item 28). For example, here is a
method that builds a tree using a client-provided Builder instance to build each
node:

带有builder实例的方法通常利用有限制的通配符类型[见第28条]来约束构建起的类型参数。例如，下面就是构建每个节点的方法，它利用一个客户端提供的Builder实例来构建树：

```java
Tree buildTree(Builder<? extends Node> nodeBuilder) { ... }
```

The traditional Abstract Factory implementation in Java has been the Class
object, with the newInstance method playing the part of the build method. This
usage is fraught with problems. The newInstance method always attempts to
invoke the class’s parameterless constructor, which may not even exist. You don’t
get a compile-time error if the class has no accessible parameterless constructor.
Instead, the client code must cope with InstantiationException or IllegalAccessException
at runtime, which is ugly and inconvenient. Also, the newInstance
method propagates any exceptions thrown by the parameterless
constructor, even though newInstance lacks the corresponding throws clauses. In
other words, **Class.newInstance breaks compile-time exception checking**. The
Builder interface, shown above, corrects these deficiencies.

Java中传统的抽象工厂实现使Class对象，用newInstance方法充当build方法的一部分。这种用法隐含着许多问题。newInstance方法总是企图调用类的无参构造器，这个构造器甚至根本不存在。如果类没有可以访问的无参构造器，你也不会收到编译时错误。相反，客户端代码必须在运行时处理InstantiationException或者IllegalAccessException，这样既不雅观也不方便。newInstance方法还会传播由无参构造器抛出的任何异常，即使newInstance缺乏相应的throws子句。换句话说，**Class.newInstance破坏了编译时的异常检查**。上面讲过的builder接口弥补了这些不足。

The Builder pattern does have disadvantages of its own. In order to create an
object, you must first create its builder. While the cost of creating the builder is
unlikely to be noticeable in practice, it could be a problem in some performancecritical
situations. Also, the Builder pattern is more verbose than the telescoping
constructor pattern, so it should be used only if there are enough parameters, say,
four or more. But keep in mind that you may want to add parameters in the future.
If you start out with constructors or static factories, and add a builder when the
class evolves to the point where the number of parameters starts to get out of hand,
the obsolete constructors or static factories will stick out like a sore thumb. Therefore,
it’s often better to start with a builder in the first place.

Builder模式的确也有它自身的不足。为了创建对象，必须先创建它的构建器。虽然创建构建器的开销在实践中可能不那么明显，但是在某些部分注重性能的情况下，可能就成问题了。builder模式还比重叠构造器模式更加冗长，因此它只在有很多参数的时候才使用，比如4个或者更多个参数。但是记住，将来你可能需要添加参数。如果一开始就使用构造器或者静态工厂，等到类需要多个参数时才添加构建器，就会无法控制，那些过时的构造器或者静态工厂显得十分不协调。因此，通常最好一开始就使用构建器。

In summary, **the Builder pattern is a good choice when designing classes
whose constructors or static factories would have more than a handful of
parameters, especially if most of those parameters are optional**. Client code is
much easier to read and write with builders than with the traditional telescoping
constructor pattern, and builders are much safer than JavaBeans.

总而言之，**如果类的构造器或者静态工厂中具有多个参数，设计这种类时，Builder模式就是中不错的选择**，特别是当大多数的参数都是可选的时候。与使用传统的重叠构造器模式相比，使用Builder模式的客户端将更易于阅读和编写，构建器也比JavaBeans更加安全。
