---
title: Algorithm-Counting Sort
date: 2017-12-3 21:13:57
categories: "数据结构与算法"
---

Counting sort-计数排序，下面是Wikipedia对其描述：

>In computer science, counting sort is an algorithm for sorting a collection of objects according to keys that are small
 integers; that is, it is an integer sorting algorithm.
 
简单翻译，在计算机科学里，计数排序是针对小的正数集合对象进行排序，也就是说，是一种整数排序算法。

<!--more-->

由于用来计数的数组C的长度取决于待排序数组中数据的最大值，这使得计数排序对于数据范围很大的数组，需要大量内存。计数排序是用来排序0到100之间的数字
的最好的算法，但是它不适合按字母顺序排序人名。但是，计数排序可以用在基数排序中的算法来排序数据范围很大的数组。

算法的步骤如下：

* 初始化一个计数数组，大小是输入数组中的最大的数加1。
  
* 遍历输入数组，遇到一个数就在计数数组对应的位置上加一。例如：遇到5，就将计数数组第五个位置的数加一。
  
* 把计数数组直接覆盖到输出数组（节约空间）。

### 举例

输入{3, 4, 3, 2, 1}，最大是4，数组长度是5。

建立计数数组{0, 0, 0, 0, 0}。

遍历输入数组：

```java 
{**3**, 4, 3, 2, 1} -> {0, 0, 0, **1**, 0}
{3, **4**, 3, 2, 1} -> {0, 0, 0, 1, **1**}
{3, 4, **3**, 2, 1} -> {0, 0, 0, **2**, 1}
{3, 4, 3, **2**, 1} -> {0, 0, **1**, 2, 1}
{3, 4, 3, 2, **1**} -> {0, **1**, 1, 2, 1}
```

计数数组现在是{0, 1, 1, 2, 1}，我们现在把它写回到输入数组里：

```java 
{0, **0**, 1, 2, 1} -> {**1**, 4, 3, 2, 1}
{0, 0, **0**, 2, 1} -> {1, **2**, 3, 2, 1}
{0, 0, 0, **1**, 1} -> {1, 2, **3**, 2, 1}
{0, 0, 0, **0**, 1} -> {1, 2, 3, **3**, 1}
{0, 0, 0, 0, **0**} -> {1, 2, 3, 3, **4**}
```

这样就排好序了。

Java实现如下：

```java 
public void sortInteger(int[] array){
    int max = maximum(array);
    System.out.println("max : " + max);
    int[] counting_array = new int[max + 1]; // Zeros out the array

    for(int curr = 0; curr < array.length; curr ++){
        counting_array[array[curr]]++;
    }

    printArray(counting_array);

    int num = 0;
    int curr = 0;

    while(curr < array.length){
        while(counting_array[num] > 0){
            array[curr] = num;
            counting_array[num]--;
            curr++;
        }
        num++;
    }
}

public void printArray(int[] array){
    for (int i = 0; i < array.length ; i++){
        System.out.print(array[i] + " ");
    }
    System.out.println("\n");
}

private int maximum(int[] array){
    int max = array[0];
    if (array.length > 1){
        for(int curr = 1; curr < array.length; curr++){
            if(array[curr] > max){ max = array[curr]; }
        }
    }
    return max;
}

public static void main(String[] args) {
    int test1[] = {2, 6, 4, 3, 2, 3, 4, 6, 3, 4, 3, 5, 2, 6, 0};
    CountingSort countingSort = new CountingSort() ;
    countingSort.printArray(test1);
    countingSort.sortInteger(test1);
    countingSort.printArray(test1);
}
```

输出如下：

```java 
2 6 4 3 2 3 4 6 3 4 3 5 2 6 0 

max : 6
1 0 3 4 3 1 3 

0 2 2 2 3 3 3 3 4 4 4 5 6 6 6 
```

### 总结

计数排序其实用途不大，因为这个是针对于整数进行的排序；而且，如果要排序的集合分布太散的话，也就是不均匀，那么将会很浪费内存。

