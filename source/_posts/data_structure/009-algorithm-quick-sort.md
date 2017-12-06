---
title: Algorithm-Quick Sort
date: 2017-12-6 21:05:57
categories: "数据结构与算法"
---

### 定义

快速排序由C. A. R. Hoare在1962年提出。它的基本思想是：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有
数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

### 步骤

快速排序一般基于递归实现。其思路是这样的：

* 选定一个合适的值（理想情况中值最好，但实现中一般使用数组第一个值）,称为“枢轴”(pivot)。

* 基于这个值，将数组分为两部分，较小的分在左边，较大的分在右边。

* 可以肯定，如此一轮下来，这个枢轴的位置一定在最终位置上。

* 对两个子数组分别重复上述过程，直到每个数组只有一个元素。

* 排序完成。

<!--more-->

### 实现

```java 
public void sort(Comparable[] a) {
    realSort(a, 0, a.length - 1);
}

public static void realSort(Comparable[] a, int low, int height) {
    int i, j ;
    Comparable index;
    if (low > height) {
        return;
    }
    i = low;
    j = height;
    index = a[i];
    while (i != j) {
        while (i < j && less(index, a[j]))
            j--;
        while (i < j && less(a[i], index))
            i++;
        if (i < j){
            exchange(a, i, j);
        }
    }
    a[low] = a[i] ;
    a[i] = index;
    show(a);
    realSort(a, low, i - 1); // 对低子表进行递归排序
    realSort(a, i + 1, height); // 对高子表进行递归排序
}

public static boolean less(Comparable a , Comparable b){
    return a.compareTo(b) <= 0 ;
}
```

### 图解

![](/images/categories/data_structure/009/quick-sort.jpg)

### 参考 

* [坐在马桶上看算法：快速排序](http://developer.51cto.com/art/201403/430986.htm)

* [快速排序原理及Java实现](http://blog.csdn.net/jianyuerensheng/article/details/51258374)

* [用Java写算法之五：快速排序](http://blog.51cto.com/flyingcat2013/1281614)

* [快速排序](https://baike.baidu.com/item/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95/369842?fr=aladdin&fromid=2084344&fromtitle=%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F)