---
title: 10 Example of Lambda Expressions in Java 8
date: 2017-6-12 10:57:38
tags:
categories: "Java学习笔记"
---

Java 8 release is just a couple of weeks away, scheduled at 18th March 2014, and there is lot of buzz and excitement about this path breaking release in Java community. One of feature, which is synonymous to this release is lambda expressions, which will provide ability to pass behaviours to methods. Prior to Java 8, if you want to pass behaviour to a method, then your only option was Anonymous class, which will take 6 lines of code and most important line, which defines the behaviour is lost in between. Lambda expression replaces anonymous classes and removes all boiler plate, enabling you to write code in functional style, which is some time more readable and expression.

This mix of bit of functional and full of object oriented capability is very exciting development in Java eco-system, which will further enable development and growth of parallel third party libraries to take advantage of multi-processor CPUs.

Though industry will take its time to adopt Java 8, I don't think any serious Java developer can overlook key features of Java 8 release e.g. lambda expressions, functional interface, stream API, default methods and new Date and Time API.

As a developer, I have found that best way to learn and master lambda expression is to try it out, do as many examples of lambda expressions as possible. Since biggest impact of Java 8 release will be on Java Collections framework its best to try examples of Stream API and lambda expression to extract, filter and sort data from Lists and Collections.

I have been writing about Java 8 and have shared [some useful resources to master Java 8](http://javarevisited.blogspot.com/2013/11/java-8-tutorials-resources-and-examples-lambda-expression-stream-api-functional-interfaces.html#axzz4jklwzvKC) in past. In this post, I am going to share you 10 most useful ways to use lambda expressions in your code, these examples are simple, short and clear, which will help you to pick lambda expressions quickly.

I am personally very excited about Java 8, particularly lambda expression and stream API. More and more I look them, it makes me enable to write more clean code in Java. Though it was not like this always; when I first saw a Java code written using lambda expression, I was very disappointed with cryptic syntax and thinking they are making Java unreadable now, but I was wrong.

After spending just a day and doing couple of examples of lambda expression and stream API, I was happy to see more cleaner Java code then before. It's like the Generics, when I first saw I hated it. I even continued using old Java 1.4 way of dealing with Collection for few month, until one of my friend explained me benefits of using Generics.

Bottom line is, don't afraid with initial cryptic impression of lambda expressions and method reference, you will love it once you do couple of examples of extracting and filtering data from Collection classes. So let's start this wonderful journey of learning lambda expressions in Java 8 by simple examples.

<!--more-->

### Example 1 - implementing Runnable using Lambda expression

One of the first thing, I did with Java 8 was trying to replace anonymous class with lambda expressions, and what could have been best example of anonymous class then implementing _Runnable_ interface. Look at the code of implementing runnable prior to Java 8, it's taking four lines, but with lambda expressions, it's just taking one line. What we did here? the whole anonymous class is replaced by _() -> {} code block_.

```java
  // Before Java 8：
  new Thread(new Runnable() {
      @Override
      public void run() {
      System.out.println("Before Java8, too much code for too little to do");
      }
  }).start();

  //Java 8 way：
  new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
```

output:

```sh
Before Java8, too much code, for too little to do
In Java8, Lambda expression rocks !!
```

This example brings us syntax of lambda expression in Java 8. You can write following kind of code using lambdas :

* _(params) -> expression_
* _(params) -> statement_
* _(params) -> { statements }_

for example, if your method don't change/write a parameter and just print something on console, you can write it like this :

```java
() -> System.out.println("Hello Lambda Expressions");
```

If your method accept two parameters then you can write them like below :

```java
(int even, int odd) -> even + odd
```

By the way, it's general practice to keep variable name short inside lambda expressions. This makes your code shorter, allowing it to fit in one line. So in above code, choice of variable names as a,b or x, y is better than even and odd.


### Example 2 - Event handling using Java 8 Lambda expressions

If you have ever done coding in Swing API, you will never forget writing event listener code. This is another classic use case of plain old Anonymous class, but no more. You can write better event listener code using lambda expressions as shown below.

```java
// Before Java 8:
JButton show = new JButton("Show");
show.addActionListener(new ActionListener() {
  @Override
  public void actionPerformed(ActionEvent e)
  {
    System.out.println("Event handling without lambda expression is boring");
  }
});

// Java 8 way:

show.addActionListener((e) -> {
   System.out.println("Light, Camera, Action !! Lambda expressions Rocks");
 });

```

Another place where Java developers frequently use anonymous class is for providing custom Comparator to Collections.sort() method. In Java 8, you can replace your ugly anonymous class with more readable lambda expression. I leave that to you for exercise, should be easy if you follow the pattern, I have shown during implementing Runnable and ActionListener using lambda expression.

### Example 3 - Iterating over List using Lambda expressions

If you are doing Java for few years, you know that most common operation with Collection classes are iterating over them and applying business logic on each elements, for example processing a list of orders, trades and events. Since Java is an imperative language, all code looping code written prior to Java 8 was sequential i.e. their is on simple way to do parallel processing of list items. If you want to do parallel filtering, you need to write your own code, which is not as easy as it looks. Introduction of lambda expression and default methods has separated what to do from how to do, which means now Java Collection knows how to iterate, and they can now provide parallel processing of Collection elements at API level. In below example, I have shown you how to iterate over List using with and without lambda expressions, you can see that now List has a forEach() method, which can iterate through all objects and can apply whatever you ask using lambda code.

```java
//Prior Java 8 :
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
for (String feature : features) {
  System.out.println(feature);
}

//In Java 8:
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
features.forEach(
  n -> System.out.println(n)
);

// Even better use Method reference feature of Java 8
// method reference is denoted by :: (double colon) operator
// looks similar to score resolution operator of C++

features.forEach(System.out::println);

```

The last example of  looping over List shows how to use method reference in Java 8. You see the double colon, scope resolution operator form C++, it is now used for method reference in Java 8.

### Example 4 - Using Lambda expression and Functional interface Predicate

Apart from providing support for functional programming idioms at language level, Java 8 has also added a new package called java.util.function, which contains lot of classes to enable functional programming in Java. One of them is Predicate, By using java.util.function.Predicate functional interface and lambda expressions, you can provide logic to API methods to add lot of dynamic behaviour in less code. Following examples of Predicate in Java 8 shows lot of common ways to filter Collection data in Java code. Predicate interface is great for filtering.

```java
public static void main(args[]){
  List languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
  System.out.println("Languages which starts with J :");
  filter(languages, (str)->str.startsWith("J"));
  System.out.println("Languages which ends with a ");
  filter(languages, (str)->str.endsWith("a"));
  System.out.println("Print all languages :");
  filter(languages, (str)->true);
  System.out.println("Print no language : ");
  filter(languages, (str)->false);
  System.out.println("Print language whose length greater than 4:");
  filter(languages, (str)->str.length() > 4);
}
public static void filter(List names, Predicate condition) {
  for(String name: names) {
    if(condition.test(name)) {
       System.out.println(name + " ");
     }
   }
 }
}
```

outputs:

```sh

Languages which starts with J :
Java
Languages which ends with a
Java
Scala

Print all languages :
Java
Scala
C++
Haskell
Lisp

Print no language :

Print language whose length greater than 4:
Scala
Haskell
```

```java
//Even better
public static void filter(List names, Predicate condition) {                    names.stream().filter((name) -> (condition.test(name))).forEach((name) -> { System.out.println(name + " ");
});
}
```

You can see that filter method from Stream API also accept a Predicate, which means you can actually replace our custom filter() method with the in-line code written inside it, that's the power of lambda expression. By the way, Predicate interface also allows you test for multiple condition, which we will see in our next example.

### Example 5 : How to combine Predicate in Lambda Expressions

As I said in previous example, java.util.function.Predicate allows you to combine two or more Predicate into one. It provides methods similar to logical operator AND and OR named as and(), or() and xor(), which can be used to combine the condition you are passing to filter() method. For example, In order to get all languages, which starts with J and are four character long, you can define two separate Predicate instance covering each condition and then combine them using Predicate.and() method, as shown in below example :

```java
// We can even combine Predicate using and(), or() And xor() logical functions
// for example to find names, which starts with J and four letters long, you
// can pass combination of two Predicate
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
Predicate<String> fourLetterLong = (n) -> n.length() == 4;
names.stream()
  .filter(startsWithJ.and(fourLetterLong))
  .forEach((n) -> System.out.print("\nName, which starts with 'J' and four letter long is : " + n));
```

Similarly you can also use or() and xor() method. This example also highlight important fact about using Predicate as individual condition and then combining them as per your need. In short, you can use Predicate interface as traditional Java imperative way, or  you can take advantage of lambda expressions to write less and do more.

### Example 6 : Map and Reduce example in Java 8 using lambda expressions

This example is about one of the popular functional programming concept called map. It allows you to transform your object. Like in this example we are transforming each element of costBeforeTeax list to including Value added Test. We passed a lambda expression x -> x*x to map() method which applies this to all elements of the stream. After that we use forEach() to print the all elements of list. You can actually get a List of all cost with tax by using Stream API's Collectors class. It has methods like toList() which will combine result of map or any other operation. Since Collector perform terminal operator on Stream, you can't re-use Stream after that. You can even use reduce() method from Stream API to reduce all numbers into one, which we will see in next example

```Java

// applying 12% VAT on each purchase
// Without lambda expressions:
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
for (Integer cost : costBeforeTax) {
  double price = cost + .12*cost;
  System.out.println(price);
}

// With Lambda expression:
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);
```

outputs:

```sh
112.0
224.0
336.0
448.0
560.0
112.0
224.0
336.0
448.0
560.0
```

### Example 6.2 - Map Reduce example using Lambda Expressions in Java 8

In previous example, we have seen how map can transform each element of a Collection class e.g. List. There is another function called reduce() which can combine all values into one. Map and Reduce operations are core of functional programming, reduce is also known as fold operation because of its folding nature. By the way reduce is not a new operation,  you might have been already using it. If you can recall SQL aggregate functions like sum(), avg() or count(), they are actually reduce operation because they accept multiple values and return a single value. Stream API defines reduce() function which can accept a lambda expression, and combine all values. Stream classes like IntStream has built-in methods like average(), count(), sum() to perform reduce operations and mapToLong(), mapToDouble() methods for transformations. It doesn't limit you, you can either use built-in reduce function or can define yours. In this Java 8 Map Reduce example, we are first applying 12% VAT on all prices and then calculating total of that by using reduce() method.

```java
// Applying 12% VAT on each purchase
// Old way:
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double total = 0;
for (Integer cost : costBeforeTax) {
  double price = cost + .12*cost;
  total = total + price;
}
System.out.println("Total : " + total);

// New way:
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
double bill = costBeforeTax.stream()
                           .map((cost) -> cost + .12*cost)
                           .reduce((sum, cost) -> sum + cost)
                           .get();
System.out.println("Total : " + bill);
```

outputs:

```sh
Total : 1680.0
Total : 1680.0
```

### Example 7: Creating a List of String by filtering

Filtering is one of the common operation Java developers perform with large collections, and you will be surprise how much easy it is now to filter bulk data/large collection using lambda expression and stream API.  Stream provides a filter() method, which accepts a Predicate object, means you can pass lambda expression to this method as filtering logic. Following examples of filtering collection in Java with lambda expression will make it easy to understand.

```java
// Create a List with String more than 2 characters
List<String> filtered = strList.stream()
                               .filter(x -> x.length()> 2)
                               .collect(Collectors.toList());
System.out.printf("Original List : %s, filtered list : %s %n", strList, filtered);
```

outputs:

```sh
Original List : [abc, , bcd, , defg, jk], filtered list : [abc, bcd, defg]
```

By the way, their is a common confusion regarding filter() method. In real world, when we filter, we left with something which is not filtered, but in case of using filter() method, we get a new list which is actually filtered by satisfying filtering criterion.


### Example 8: Applying function on Each element of List

We often need to apply certain function to each element of List e.g. multiplying each element by certain number or dividing it, or doing anything with that. Those operations are perfectly suited for map() method, you can supply your transformation logic to map() method as lambda expression and it will transform each element of that collection, as shown in below example.

```java
// Convert String to Uppercase and join them using coma
List<String> G7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
String G7Countries = G7.stream()
                       .map(x -> x.toUpperCase())
                       .collect(Collectors.joining(", "));
System.out.println(G7Countries);
```

outputs:

```sh
USA, JAPAN, FRANCE, GERMANY, ITALY, U.K., CANADA
```

### Example 9: Creating a Sub List by Copying distinct values

This example shows how you can take advantage of distinct() method of Stream class to filter duplicates in Collection.

```java
// Create List of square of all distinct numbers
List<Integer> numbers = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
List<Integer> distinct = numbers.stream()
                                .map( i -> i*i)
                                .distinct()
                                .collect(Collectors.toList());
System.out.printf("Original List : %s, Square Without duplicates : %s %n", numbers, distinct);
```

outputs:

```sh
Original List : [9, 10, 3, 4, 7, 3, 4], Square Without duplicates : [81, 100, 9, 16, 49]
```

### Example 10 : Calculating Maximum, Minimum, Sum and Average of List elements

There is a very useful method called summaryStattics() in stream classes like IntStream, LongStream and DoubleStream. Which returns returns an IntSummaryStatistics, LongSummaryStatistics or DoubleSummaryStatistics describing various summary data about the elements of this stream. In following example, we have used this method to calculate maximum and minimum number in a List. It also has getSum() and getAverage() which can give sum and average of all numbers from List.

```java
//Get count, min, max, sum, and average for numbers
List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
IntSummaryStatistics stats = primes.stream()
                                   .mapToInt((x) -> x)
                                   .summaryStatistics();
System.out.println("Highest prime number in List : " + stats.getMax());
System.out.println("Lowest prime number in List : " + stats.getMin());
System.out.println("Sum of all prime numbers : " + stats.getSum());
System.out.println("Average of all prime numbers : " + stats.getAverage());
```
outputs:

```sh
Highest prime number in List : 29
Lowest prime number in List : 2
Sum of all prime numbers : 129
Average of all prime numbers : 12.9
```


From：[10 Example of Lambda Expressions in Java 8](http://javarevisited.blogspot.sg/2014/02/10-example-of-lambda-expressions-in-java8.html)
