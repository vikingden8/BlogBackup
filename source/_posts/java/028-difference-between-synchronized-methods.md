---
title: Difference between synchronized static and instance methods
date: 2018-01-30 17:30:02
tags:
categories: "Java学习笔记"
---

From : [Difference between synchronized static and instance methods](http://omtlab.com/difference-between-synchronized-methods/)

This article is about difference between synchronized static and instance methods with example. Here i am going to explain this with 5 different scenarios.

**Scenario : 1**

When one thread accessing synchronized instance method at that time other threads can NOT access ANY synchronized instance methods.

**Scenario : 2**

When one thread accessing synchronized instance method at that time other threads can access ANY NON synchronized instance methods.

**Scenario : 3**

When one thread accessing synchronized instance method at that time other thread can access synchronized static methods.

**Scenario : 4**

When one thread accessing synchronized static method at that time other threads can NOT access ANY synchronized static methods.

**Scenario : 5**

When one thread accessing synchronized static method at that time other threads can access ANY NON synchronized static methods.

<!--more-->

Example containing all above scenarios.

**MyClass**

```java
public class MyClass {
    /**
     * Method Id 1
     */
    public synchronized void syncInstanceMethod1() {
        System.out.println("sync instance method 1 call Start");
        makeThreadSleep();
        System.out.println("sync instance method 1 call Ended");
    }

    /**
     * Method Id 2
     */
    public synchronized void syncInstanceMethod2() {
        System.out.println("sync instance method 2 call Start");
        makeThreadSleep();
        System.out.println("sync instance method 2 call Ended");
    }

    /**
     * Method Id 3
     */
    public void nonSyncInstanceMethod() {
        System.out.println("non sync instance method call Start");
        System.out.println("non sync instance method call Ended");
    }

    /**
     * Method Id 4
     */
    public static synchronized void syncStaticMethod1() {
        System.out.println("sync static method 1 call Start");
        staticMakeThreadSleep();
        System.out.println("sync static method 1 call Ended");
    }

    /**
     * Method Id 5
     */
    public static synchronized void syncStaticMethod2() {
        System.out.println("sync static method 2 call Start");
        staticMakeThreadSleep();
        System.out.println("sync static method 2 call Ended");
    }

    /**
     * Method Id 6
     */
    public static void nonSyncStaticMethod() {
        System.out.println("non sync static method call Start");
        System.out.println("non sync static method call Ended");
    }

    private void makeThreadSleep() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static void staticMakeThreadSleep() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

**MyThread**

```java
class MyThread extends Thread {

    private int methodId = 0;
    private MyClass myClass = null;

    public MyThread(String threadName, int methodId, MyClass myClass) {
        super(threadName);
        this.methodId = methodId;
        this.myClass = myClass;
    }

    @Override
    public void run() {
        switch (methodId) {
        case 1: {
            myClass.syncInstanceMethod1();
            break;
        }
        case 2: {
            myClass.syncInstanceMethod2();
            break;
        }
        case 3: {
            myClass.nonSyncInstanceMethod();
            break;
        }
        case 4: {
            MyClass.syncStaticMethod1();
            break;
        }
        case 5: {
            MyClass.syncStaticMethod2();
            break;
        }
        case 6: {
            MyClass.nonSyncStaticMethod();
            break;
        }
        }
    }
}
```

**Main**

```java
public class Main {

    public static void main(String[] args) {

        MyClass myClass = new MyClass();

        /**
         * Scenario : 1 When one thread accessing synchronized instance method
         * at that time other threads can NOT access ANY synchronized instance
         * methods.
         */
        int methodIdForThreadOne = 1;
        int methodIdForThreadTwo = 2;

        /**
         * Scenario : 2 When one thread accessing synchronized instance method
         * at that time other threads can access ANY NON synchronized instance
         * methods.
         */
        // int methodIdForThreadOne = 1;
        // int methodIdForThreadTwo = 3;

        /**
         * Scenario : 3 When one thread accessing synchronized instance method
         * at that time other thread can access synchronized static methods.
         */
        // int methodIdForThreadOne = 1;
        // int methodIdForThreadTwo = 4;

        /**
         * Scenario : 4 When one thread accessing synchronized static method at
         * that time other threads can NOT access ANY synchronized static
         * methods.
         */
        // int methodIdForThreadOne = 4;
        // int methodIdForThreadTwo = 5;

        /**
         * Scenario : 5 When one thread accessing synchronized static method at
         * that time other threads can access ANY NON synchronized static
         * methods.
         */
        // int methodIdForThreadOne = 4;
        // int methodIdForThreadTwo = 6;

        MyThread myThreadOne = new MyThread("ONE", methodIdForThreadOne,
                myClass);
        MyThread myThreadTwo = new MyThread("TWO", methodIdForThreadTwo,
                myClass);

        myThreadOne.start();
        myThreadTwo.start();

    }

}
```

Output for different scenarios

**Scenario : 1**

```sh
viking@Viking ~/work/java-projects/java-demo $ java Main
sync instance method 1 call Start
sync instance method 1 call Ended
sync instance method 2 call Start
sync instance method 2 call Ended
viking@Viking ~/work/java-projects/java-demo $
```

**Scenario : 2**

```sh
viking@Viking ~/work/java-projects/java-demo $ java Main
sync instance method 1 call Start
non sync instance method call Start
non sync instance method call Ended
sync instance method 1 call Ended
viking@Viking ~/work/java-projects/java-demo $
```

**Scenario : 3**

```sh
viking@Viking ~/work/java-projects/java-demo $ java Main
sync instance method 1 call Start
sync static method 1 call Start
sync instance method 1 call Ended
sync static method 1 call Ended
viking@Viking ~/work/java-projects/java-demo $
```

**Scenario : 4**

```sh
viking@Viking ~/work/java-projects/java-demo $ java Main
sync static method 1 call Start
sync static method 1 call Ended
sync static method 2 call Start
sync static method 2 call Ended
viking@Viking ~/work/java-projects/java-demo $
```

**Scenario : 5**

```sh
viking@Viking ~/work/java-projects/java-demo $ java Main
sync static method 1 call Start
non sync static method call Start
non sync static method call Ended
sync static method 1 call Ended
viking@Viking ~/work/java-projects/java-demo $
```

In Java, the following:

```java
public class MyClass
{
    public synchronized void nonStaticMethod()
    {
        // code
    }

    public synchronized static void staticMethod()
    {
        // code
    }
}
```

is equivalent to the following:

```java
public class MyClass
{
    public void nonStaticMethod()
    {
      synchronized(this)
      {
          // code
      }
    }

    public void static staticMethod()
    {
      synchronized(MyClass.class)
      {
          // code
      }
    }
}
```

As you see, static methods use **this** as monitor object, and non-static methods use class object as monitor.

As **this** and **MyClass.class** are different objects, static and non-static methods may run concurrently.

To "fix" this, create a dedicated static monitor object and use it in both static and non-static methods:

```java
public class MyClass
{
    private static Object monitor = new Object();

    public void nonStaticMethod()
    {
      synchronized(monitor)
      {
          // code
      }
    }

    public static void staticMethod()
    {
      synchronized(monitor)
      {
          // code
      }
    }
}
```

See More : [thread safety with two synchronized methods, one static, one non static
](https://stackoverflow.com/questions/27039943/thread-safety-with-two-synchronized-methods-one-static-one-non-static)
