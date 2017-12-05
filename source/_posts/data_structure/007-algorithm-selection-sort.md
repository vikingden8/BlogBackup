---
title: Algorithm-Selection Sort
date: 2017-12-5 19:49:57
categories: "数据结构与算法"
---

### 定义

选择排序(Selection sort)是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小元素，存放到排序序列的起始位置，然后，再从剩余
未排序元素中继续寻找最小元素，然后放到排序序列末尾。以此类推，直到所有元素均排序完毕。

<!--more-->

### 实现

```java 

public void sort(Comparable[] a) {
    int N = a.length ;
    for (int i = 0 ; i < N ; i++){
        int min = i ;
        for (int j = i+1 ; j < N ; j++){
            if (less(a[j] , a[min])) min = j ;
        }
        if (i == min) continue;
        exchange(a , i , min);
    }
}
```

### 优化

如果在每一次查找最小值的时候，也可以找到一个最大值，然后将两者分别放在它们应该出现的位置，这样遍历的次数就比较少了。

实现如下：

```java 
public void optimizedSort(Comparable[] a){
    int left = 0 ;
    int right = a.length -1 ;
    int min ;
    int max ;
    while (left < right){
        min = left ;
        max = right ;

        for (int i = left ; i <= right ; i++){
            if (less(a[i], a[min])){
                min = i ;
            }
            if (less(a[max], a[i])){
                max = i ;
            }
        }

        if (min != left){
            exchange(a, min, left);
        }
        //in case left has exchanged
        if (max == left ) max = min ;
        if (max != right){
            exchange(a, max, right);
        }

        left ++;
        right --;

    }
}
```

这里要注意这一句：

```java 
//in case left has exchanged
if (max == left ) max = min ;
```

也就是要注意，如果最大值等于当前遍历的左边值时，要把max的索引值设置为可能交换后的min值。

