---
title: Memory Leak In Java
date: 2017-4-25 09:18:37
tags:
categories: "Java学习笔记"
---

A memory leak occurs when memory acquired by a program for execution is never freed-up to be used by other programs and applications.

Many java programmers believe that they don’t have to worry about allocating and freeing up memory. You simply create objects, and Java takes care of removing them when they are no longer needed by the application through a mechanism known as garbage collection. By saying that, programmers assume that memory leaks are taken care by java programming language. But that’s the case entirely.

### What is garbage collector’s role?

The job of the garbage collector is to find objects that are no longer be accessed or referenced by an application and to remove them. The garbage collector starts at the root nodes, classes that persist throughout the life of a Java application, and sweeps through all of the nodes that are referenced. As it traverses the nodes, it keeps track of which objects are actively being referenced. Any classes that are no longer being referenced are then eligible to be garbage collected. The memory resources used by these objects can be returned to the Java virtual machine (JVM) when the objects are deleted.

So it is evident that unused objects are automatially garbage collected. However, the key point to remember is that an object is only counted as being unused when it is no longer referenced If an object reference is unintentionally retained, not only is that object excluded from garbage collection, but so too are any objects referenced by that object, and so on. Even if only a few object references are unintentionally retained, many, many objects may be prevented from being garbage collected, with potentially large effects on performance.

<!--more-->

### What causes memory leaks in Java?

#### Obsolete object references

An obsolete reference is simply a reference that will never be dereferenced again. As I mentioned above, if any intentional referernces retained for unused objects, garbage collectors fails to recognize those objects as unused.

Following is the example code that causes memory leaks because of obsolete object references.

```java
import java.util.Iterator;
import java.util.NoSuchElementException;


public class Stack < E > implements Iterable < E > {

    private int N;
    private E[] array;

    @
    SuppressWarnings("unchecked")
    public Stack(int capacity) {
        array = (E[]) new Object[capacity];
    }

    @
    Override
    public Iterator < E > iterator() {
        return new StackIterator();
    }

    private class StackIterator implements Iterator < E > {

        private int i = N - 1;@
        Override
        public boolean hasNext() {
            return i >= 0;
        }@
        Override
        public E next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            return array[i--];
        }@
        Override
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
    public void push(E item) {
        if (isFull()) {
            throw new RuntimeException("Stack overflow");
        }
        array[N++] = item;
    }
    public E pop() {
        if (isEmpty())
            throw new RuntimeException("Stack underflow");
        E item = array[--N];
        // array[N] = null; if not set this, it will memory leak
        return item;
    }
    public boolean isEmpty() {
        return N == 0;
    }
    public int size() {
        return N;
    }
    public boolean isFull() {
        return N == array.length;
    }
    public E peek() {
        if (isEmpty())
            throw new RuntimeException("Stack underflow");
        return array[N - 1];
    }
}
```

Test class that pushes 10000 integers and pop them. At the end of the these operations, we expect all Integer objects gets destroyed. But when i check heap profile with visual VM, all 10000 Integer objects still there in Java heap.

```java
public class StackTest {

    public static void main(String[] args) {
        Stack < Integer > s = new Stack < > (10000);
        for (int i = 0; i < 10000; i++) {
            s.push(i);
        }
        while (!s.isEmpty()) {
            s.pop();
        }
        while (true) {
            //do something
        }
    }
}
```

#### Heap dump from visualVM

![](MemoryLeaks1.jpg)

Easy way to fix for this sort of problem is to null out references once they become obsolete. In the case of our Stack class, the reference to an item becomes obsolete as soon as it’s popped off the stack. The corrected version of the pop method looks like this.

```java
public E pop() {
    if (isEmpty())
        throw new RuntimeException("Stack underflow");
    E item = array[--N];
    array[N] = null;
    return item;
}
```

![](MemoryLeaks2.jpg)

### Good practices:

Programmers should not be obsessed with nulling out object references. Nulling out object references should be the exception rather than the norm. The best way to eliminate an obsolete reference is to let the variable that contained the reference fall out of scope. This occurs naturally if you define each variable in the narrowest possible scope

### When should we null out a reference?

In case of statck, it is maintaining it’s own memory for storing elements. When array elements fell out of scope, we just need to let the garbage collector knows this fact by nulling out those references. whenever a class manages its own memory, the programmer should be alert for memory leaks. Whenever an element is freed, any object references contained in the element should be nulled out.
