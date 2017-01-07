---
title: Java集合框架学习-AbstractCollection
date: 2017-01-08 00:14:14
tags:
categories: "Java集合框架学习"
---

在前面看过了Collection接口和List接口的定义，我们在看List的一个具体实现ArrayList时，看下它的类定义：

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
  ...
}
```

可以看到ArrayList实现了List接口，但是同时是继承自AbstractList抽象类的，但是问题来了，AbstractList是干什么的呢？我们跟踪其源码：

```java
public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
  ...
}
```

现在看到AbstractList是继承自AbstractCollection，我们继续追踪：

```java
public abstract class AbstractCollection<E> implements Collection<E> {
  ...
}
```

<!--more-->

通过阅读AbstractCollection的DOC，其主要提供了对集合类操作的一些基本实现。AbstractCollection是一个实现Collection接口的抽象类、本身没有提供任何额外的方法、所有想要实现Collection接口的实现类可以从AbstractCollection继承、它存在的意义就是大大减少编码的工作量，AbstractCollection中的方法都是从Collection接口中继承，除两个关键方法iterator()和size()方法外都提供了简单的实现（这里要注意add()只是抛出异常、要求子类必须去实现、因为不同数据结构添加元素的方式不一致），其他的方法都是通过Iterator的方法来实现的，所以在子类中要实现Iterator的方法。

我们看下其源码：

```java
/**
 * 此类提供 Collection 接口的骨干实现，以最大限度地减少了实现此接口所需的工作。
 *
 * 对于一个不可更改的集合，只要继承这个类并且实现迭代器和size()方法就行。
 *
 * 对于一个可更改的集合，需要实现add和返回Iterator的方法，当然可选的实现remove方法
 *
 * 子类要继承此抽象类、
 * 1、必须提供两个构造方法、一个无参一个有参。
 * 2、必须重写抽象方法 Iterator<e> iterator();方法体中必须有hasNext()
 *  next()两个方法、同时必须实现remove方法 用于操作集合中元素
 * 3、一般要重写add方法、否则子类调用add方法会抛UnsupportedOperationException异常。
 *
 * 子类通常要重写此类中的方法已得到更有效的实现、
 */

public abstract class AbstractCollection<E> implements Collection<E> {
    /**
     * Sole constructor.  (For invocation by subclass constructors, typically
     * implicit.)
     */
    protected AbstractCollection() {
    }

    // Query Operations

    public abstract Iterator<E> iterator();

    public abstract int size();

    public boolean isEmpty() {
        return size() == 0;
    }

    /**
    * 判断是否包含指定的元素
    * （1）如果参数为null，查找值为null的元素，如果存在，返回true，
    *  否则返回false。
    * （2）如果参数不为null，则根据equals方法查找与参数相等的元素，
    *  如果存在，则返回true，否则返回false。
    * 注意：这里必须对null单独处理，否则null.equals会报空指针异常
    */
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }

    /**
    * 功能：将集合元素转换为数组
    * 实现：
    * （1）创建一个数组，大小为集合中元素的数量
    * （2）通过迭代器遍历集合，将当前集合中的元素复制到数组中（复制引用）
    * （3）如果集合中元素比预期的少，则调用Arrays.copyOf()
    *  方法将数组的元素复制到新数组中，并返回新数组
    * （4）如果集合中元素比预期的多，则调用finishToArray方法生成新数组，
    *  并返回新数组，否则返回（1）中创建的数组
    */
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) // fewer elements than expected
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    /**  
    * 功能：通过泛型约束返回指定类型的数组
    * 实现：
    * （1）如果传入数组的长度的长度大于等于集合的长度，
    *  则将当前集合的元素复制到传入的数组中
    * （2）如果传入数组的长度小于集合的大小，则将创建一个新的数组来进行集合元素的存储
    */
    public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
    * 功能：数组扩容
    * （1）当数组索引指向最后一个元素+1时，对数组进行扩容：
    *  即创建一个更长的数组，然后将原数组的内容复制到新数组中
    * （2）扩容大小：cap + cap/2 +1
    * （3）扩容前需要先判断是否数组长度是否溢出
    * 注意：这里的迭代器是从上层的方法（toArray）传过来的，
    * 并且这个迭代器已执行了一部分，而不是从头开始迭代的
    */
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int i = r.length;
        while (it.hasNext()) {
            int cap = r.length;
            if (i == cap) {
                int newCap = cap + (cap >> 1) + 1;
                // overflow-conscious code
                if (newCap - MAX_ARRAY_SIZE > 0)
                    newCap = hugeCapacity(cap + 1);
                r = Arrays.copyOf(r, newCap);
            }
            r[i++] = (T)it.next();
        }
        // trim if overallocated
        return (i == r.length) ? r : Arrays.copyOf(r, i);
    }

    /**
    *判断数组容量是否溢出，最大为整型数据的最大值
    */
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError
                ("Required array size too large");
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    // Modification Operations

    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }

    /**
    * 功能：移除指定元素
    * （1）如果参数为null，则找到第一个值为null的元素，
    * 并将其删除，返回true，如果不存在null的元素，返回false。
    * （2）如果参数不为null，则根据equals方法找到第一个与参数相等的元素，
    * 并将其删除，返回true，如果找不到，返回false。
    */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }


    // Bulk Operations

    /**
    * 遍历参数集合，依次判断参数集合中的元素是否在当前集合中，
    * 只要有一个不存在，则返回false
    * 如果参数集合中所有的元素都在当前集合中，则返回true
    */
    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    /**
     * 遍历参数集合，依次将参数集合中的元素添加当前集合中
     */
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    /**
    * 功能：移除参数集合的元素
    * （1）获取当前集合的迭代器进行遍历
    * （2）如果当前集合中的元素包含在参数集合中，则删除当前集合中的元素  
    * 注：只要参数集合中有任何一个元素在当前元素中，
    * 则返回true，表示当前集合有发送变化，否则返回false。
    */
    public boolean removeAll(Collection<?> c) {
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    /***  
    * 功能：求参数集合与当前集合的交集    
    * （1）获取当前集合的迭代器进行遍历
    * （2）如果当前集合中的元素不在参数集合中，则将其移除。
    * 注意：如果当前集合是参数集合中的子集，则返回false，
    * 表示当前集合未发送变化，否则返回true。
    */
    public boolean retainAll(Collection<?> c) {
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    /**
     * 迭代删除集合中的全部元素
     */
    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }


    //  String conversion

     /**
     * 以字符串的方式返回该集合列表。此字符串是以其在集合中的顺序返回，
     * 字符串会以"[]"外包围。元素之间通过逗号+空格分隔，如”, "
     */
    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

}
```

整体上来说，AbstractCollection的源码还是比较容易理解，尤其是集合增、删、查等操作都非常简单。

**问题点：那为什么最大长度要减8？根据官方的解释：**

>* The maximum size of array to allocate.

>* Some VMs reserve some header words in an array.

>* Attempts to allocate larger arrays may result in

>* OutOfMemoryError: Requested array size exceeds VM limit

也就是说有的虚拟机实现，数组对象的头部会占用这8个字节。

**AbstractCollection 的 add(E) 方法默认是抛出异常，这样会不会容易导致问题？为什么不定义为抽象方法**

答案译自 [stackoverflow](http://stackoverflow.com/questions/23889410/why-is-add-method-not-abstract-in-abstractcollection) :

* 如果你想修改一个不可变的集合时，抛出 UnsupportedOperationException 是标准的行为，比如 当你用 Collections.unmodifiableXXX() 方法对某个集合进行处理后，再调用这个集合的 修改方法（add,remove,set…），都会报这个错。因此 AbstractCollection.add(E) 抛出这个错误是遵从标准；

**参考资料：**

[源码分析-java-AbstractCollection](http://blog.csdn.net/u011518120/article/details/51924587)

[Java 集合深入理解（5）：AbstractCollection](http://blog.csdn.net/u011240877/article/details/52829912)

[Java集合类：AbstractCollection源码解析](http://www.cnblogs.com/paddix/p/5560933.html)
