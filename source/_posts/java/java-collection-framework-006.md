---
title: Java集合框架学习-AbstractQueue
date: 2017-1-13 09:46:09
tags:
categories: "Java集合框架学习"
---

跟List，Set，Map一样，Queue接口有一个抽象接口：AbstractQueue。看下JDK源码：

```java
public abstract class AbstractQueue<E> extends AbstractCollection<E>
    implements Queue<E> {

    /**
     * Constructor for use by subclasses.
     */
    protected AbstractQueue() {
    }

     /**
     * 往队列中插入元素,主要是调用offer方法插入元素，如果插入成功，返回true，否则，抛出异常
     *
     */
    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }

     /**
     * 往队列中移除元素,主要是调用poll方法弹出元素，如果poll方法返回的不是null，移除成功，并把结果返回；否则，抛出异常
     *
     **/
    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

     /**
     * 往队列中获取队尾的元素，主要是调用peek方法，如果peek方法返回不是null，把结果返回；否则，抛出异常
     *
     **/
    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

     /**
     * 移除队列中的所有元素，主要调用poll方法，一个一个弹出，直到为null
     *
     **/
    public void clear() {
        while (poll() != null)
            ;
    }

     /**
     * 往队列中添加一个集合，如果集合为null，或者为本对象自己，则抛出异常，然后遍历添加集合中的对象到
     * 队列中，如果集合中不能所有的元素加入队列，则会抛出异常，因为在添加的时候，使用的是add方法
     *
     **/
    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

}
```

在JDK文档中提到，如果队列继承自这个抽象类时，至少要提供一个方法的实现：offer，来确保插入的元素不为null。为什么呢？

因为如果插入的元素为null，则在调用队列中的poll和peek方法时会出现异常。
