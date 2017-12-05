---
title: Algorithm-Insert Sort
date: 2017-12-5 20:32:57
categories: "数据结构与算法"
---

### 定义

插入排序（Insertion Sort）的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。插入排序在实现上，通
常采用in-place排序（即只需用到O(1)的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。

<!--more-->

### 步骤

* 从第一个元素开始，该元素可以认为已经被排序

* 取出下一个元素，在已经排序的元素序列中从后向前扫描

* 如果该元素（已排序）大于新元素，将该元素移到下一位置

* 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置

* 将新元素插入到该位置中

* 重复步骤2

### 实现

```java 
public void optimizedSort(Comparable[] a){
    int N = a.length ;
    for (int i = 1; i < N ; i++){
        Comparable insertNode = a[i] ;
        int j = i - 1 ;
        while (j >= 0 && less(insertNode, a[j])){
            a[j + 1] = a[j] ;
            j -- ;
        }
        a[j + 1] = insertNode ;
    }
}
```

### 参考

* [视觉直观感受 7 种常用的排序算法](http://blog.jobbole.com/11745/)