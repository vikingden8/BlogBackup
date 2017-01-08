---
title: 不同的列表应该选择不同的遍历方法
date: 2017-01-08 09:43:29
tags:
categories: "Java学习笔记"
---

### 分析

在Java中，RandomAccess和Cloneable、Serializable一样都是标识接口，不需要任何实现，只是用来表明其实现类具有某种特质的，实现了Cloneable表明可以被拷贝，实现了Serializable接口表明被序列化了，实现了RandomAccess则表明这个类可以随机存取。

ArrayList数组实现了RandomAccess接口（随机存取接口），标识着ArrayList是一个可以随机存取的列表，即元素之间没有关联，即两个位置相邻的元素之间没有相互依赖关系，可以随机访问和存储。

LinkedList类也是一个列表，它是有序存取的，实现了双向链表、每个数据节点都有单个数据项，前面节点的引用（Previous Node）、本节点元素（Node Element）、后续节点的引用（Next Node）。也就是说LinkedList两个元素本来就是有联系的，我知道你存在，你知道我存在。

<!--more-->

### 场景

我们来看一个场景，统计一个科目考试的平均分：

```java
public static void main(String[] args){   
    //学生数量，80万   
    int stuNum = 80 * 10000;   
    //List集合，记录所有学生的分数   
    List<Integer> socres = new ArrayList<Integer>(stuNum);   
    //写入分数   
    for(int i = 0; i < stuNum; i++){   
        scores.add(new Random.nextIne(150));   
    }   

    //记录开始计算时间   
    long start = System.currentTimeMillis();   
    System.out.println("平均分是：" + average(scores));   
    System.out.println("执行时间：" + (System.currentTimeMillis() - start) + "ms");   
}   

//计算平均数   
public static int average(List<Integer> list){   
    int sum = 0;   
    //遍历求和   
    for(int i : list){   
        num += i;   
    }   

    //除以人数，计算平均值   
    return sum/list.size();   
}
```

输出结果：

```java
平均分是：74
执行时间：47ms
```

仅仅求一个平均值就花费了47毫秒，考虑其他诸如加权平均值、补充平均值等的话，花费时间肯定更长。我们仔细分析一下arverage方法，加号操作是最基本操作，没有可以优化，我们可以尝试对List遍历进行优化。

我们尝试一下，List的遍历还有另外一种形式，即通过下表方式来遍历，如下：

```java
public static int average(List<Integer> list){   
    int sum = 0;   
    //遍历求和   
    for(int i = 0, size = list.size(); i < size; i++){   
        sum += list.get(i);   
    }   
    //除以人数，计算平均值   
    return sum/list.size();   
}
```

采用上面方式遍历，输出结果：

```java
平均分：74
执行时间：16ms
```

执行时间大幅提升，性能提升65%。

为什么会有如此提升呢？我们知道foreacher与下面代码等价：

```java
for(Iterator<Integer> i = list.iterator(); i.hasNext;){   
    sum += i.next();   
}
```

迭代器是23中设计模式的一种，提供一种方法访问一个容器对象中的各个元素，同时又无须暴露该对象的内部细节。也就是说对于ArrayList，需要先创建一个迭代器容器，然后屏蔽内部遍历细节，对外提供hasNext、next等方法。

问题是ArrayList实现了RandomAccess接口，表明元素之间本没有关系，为了使用迭代器就需要强制建立一种互相“知晓”的关系，比如上一个元素可以判断是否有下一个元素，以及下一个元素是什么等关系，这也就是通过foreach遍历耗时的原因。

对于LinkedList由分析讲述，元素之间已经有关联了，使用foreach也就是迭代器方式是不是更高呢？代码如下：

```java
public static void main(String[] args){   
    //学生数量，80万   
    int stuNum = 80 * 10000;   
    //List集合，记录所有学生分数   
    List<Integer> scores = new LinkedList<Integer>();   

    //写入分数   
    for(int i = 0; i < stuNum; i++){   
        scores.add(new Random.nextIne(150));   
    }   

    //记录开始计算时间   
    long start = System.currentTimeMillis();   
    System.out.println("平均分是：" + average(scores));   
    System.out.println("执行时间：" + (System.currentTimeMillis() - start) + "ms");   
}  

public static int average(List<Integer> list){   
    int sum = 0;   
    //foreach遍历求和   
    for(int i : list){   
        sum += i;   
    }   
    //除以人数，计算平均值   
    return sum/list.size();   
}
```

运行结果：

```java
平均分数：74
执行时间：16ms
```

确实如此效率非常高。你也可以测试一下使用下标的方式遍历LinkedList元素，效率非常低。

我们查看源码：

```java
public E get(int index){   
    return entry(index).element;   
}
```

由entry方法查找指定下标的节点，然后返回其包含的元素，看entry方法：

```java
private Entry<E> entry(int index){   
    //检查下标是否越界   
    Entry<E> e = header;   
    if(index < (size >> 1)){   
        //如果下标小于中间值，则从头节点开始搜索   
        for(int i = 0; i <= index; I++){   
        e = e.next;   
    }   
    }else{   
        //如果下标大于等于中间值，则从尾节点反向遍历   
        for(int i = size; i > index; i++){   
            e = e.previous;   
        }   
    }   
    return e;   
}
```

想想，每次get方法，都遍历，性能从何说起！

### 建议

列表的遍历不是那么简单，其中有很多“学问”，适时的选择最优的遍历方式，不要固化一种。

明白了随机存取列表和有序列表的区别，我们average方法就必须重构了，以便实现不同列表采用不同的遍历方式，代码如下：

```java
public static int average(List<Integer> list){   
    int sum = 0;   
    if(list instanceof RandomAccess){   
        //可以随机存取，则使用下标遍历   
        for(int i = 0, size = list.size(); i < size; i++)
        {   
            sum += list.get(i);   
        }   
    }else{   
        //有序存储，使用foreach方式   
        for(int i : list)
        {   
            sum += I;   
        }   
    }   

    //除与人数，计算平均值   
    return sum/list.size();   
}
```

如此一来，列表遍历就可以”以不变应万变“了，无论是随机存储列表（for-i索引）还是有序列表（foreach），它都可以提供快速的遍历.
