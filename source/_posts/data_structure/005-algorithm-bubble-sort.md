---
title: Algorithm-Bubble Sort
date: 2017-12-2 23:26:57
categories: "数据结构与算法"
---

Bubble Sort - 冒泡排序，下面是Wikipedia对其的定义：

>**Bubble sort** sometimes referred to as _sinking sort_, is a simple sorting algorithm that repeatedly steps through the list to be 
sorted, compares each pair of adjacent items and swaps them if they are in the wrong order. The pass through the list 
is repeated until no swaps are needed, which indicates that the list is sorted. 

简单翻译以下: 冒泡排序，有时也称作 _sinking sort_（实在没有很优雅的词汇来描述这个sinking，sinking的意思就是沉下来）,是一种简单的排序算法，
通过比较相邻的两个元素，如果他们的顺序不正确的情况下，他们的元素值互换。直到没有需要互换的元素为止，也就意味着列表集合已经是有序的。

<!--more-->

### 实现

使用java语言实现如下：

```java 
public void originSort(Comparable[] a){
    while(true) {
        int N = a.length ;
        boolean  hasSwaped = false ;
        for (int i = 0 ; i < N - 1 ; i ++){
            if (less(a[i + 1], a[i])){
                exchange(a, i, i + 1);
                hasSwaped = true ;
            }
        }
        if (!hasSwaped){
            break ;
        }
    }
     ;
}
```

下面分别是less，exchange方法的实现：

```java
/**
 * 判断对象a是否小于对象b
 * @param a 对象a
 * @param b 对象b
 * @return 如果对象a小于对象b，返回true；否则，返回false
 */
public static boolean less(Comparable a , Comparable b){
    return a.compareTo(b) < 0 ;
}

/**
 * 交换数组对象中的两个指定位置的对象
 * @param a 交换的数组对象
 * @param i 要交换的对象位置i
 * @param j 要交换的对象位置j
 */
public static void exchange(Comparable[] a  , int i , int j){
    Comparable temp = a[i] ;
    a[i] = a[j] ;
    a[j] = temp ;
}

/**
 * 显示对象数组的元素
 * @param a 要显示的数组对象
 */
public static void show(Comparable[] a){
    for (int i = 0 ; i < a.length ; i++){
        System.out.print(a[i] + " ");
    }
    System.out.println();
}

```

在main方法中测试：

```java 
public static void main(String[] args){
    String[] nums = new String[]{"R" , "G" , "A" , "F" , "V" , "M"} ;
    BubbleSort sSort = new BubbleSort() ;
    System.out.println("Before Sort");
    show(nums);
    sSort.originSort(nums);
    System.out.println("After Sort");
    show(nums);
}
```

输出如下：

``` 
Before Sort
R G A F V M 
After Sort
A F G M R V 
```

### 优化1

其实在实现每一次迭代比较时，可以看出两两比较最终位置的数值是最大值（如果是降序排序的话，最后的位置是最小值）。所以，可以在优化如下：

```java
public void optimizedSort_!(Comparable[] a) {
    int N = a.length ;
    for (int i = 0 ; i < N ; i++){
        for (int j = 0 ; j < N - i - 1 ; j++){
            if (less(a[j + 1] , a[j])) {
                exchange(a, j, j+1);
            }
        }
    }
}
```

测试可以得出，同样的结果。

### 优化2

如果上面优化1代码中，里面一层循环在某次扫描中没有执行交换，则说明此时数组已经全部有序列，无需再扫描了。因此，增加一个标记，每次发生交换，就标记，
如果某次循环完没有标记，则说明已经完成排序。

```java 
public void optimizedSort_2(Comparable[] a) {
    int N = a.length ;
    for (int i = 0 ; i < N ; i++){
        boolean hasSwaped = false ;
        for (int j = 0 ; j < N - i - 1 ; j++){
            if (less(a[j + 1] , a[j])) {
                exchange(a, j, j+1);
                hasSwaped = true ;
            }
        }
        if (!hasSwaped) break ;
    }
}
```

### 优化3

在第一步优化的基础上发进一步思考：如果a[i..N]已是有序区间，上次的扫描区间是R[0..n], 其中N为整个集合的长度， n > i。记上次扫描时最后一次执行
交换的位置为lastSwapPos，即为i，不难发现R[i..n]区间也是有序的，否则这个区间也会发生交换；所以下次扫描区间就可以由R[0..n-1]缩减到[0..lastSwapPos]，
实现如下：

```java 
public void bestSort(Comparable[] a){
    int N = a.length ;
    int lastSwapPos = N ;
    int k ;
    while (lastSwapPos > 0){
        k = lastSwapPos ;
        lastSwapPos = 0 ;
        for (int j = 0 ; j < k -1  ; j++){
            if (less(a[j + 1] , a[j])) {
                exchange(a, j, j + 1);
                lastSwapPos = (j + 1)  ;
            }
        }
    }
}
```

上面就是关于冒泡排序的Java实现以及优化。

