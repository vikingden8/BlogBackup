---
title: Algorithm-Heap Sort
date: 2017-12-9 22:51:57
categories: "数据结构与算法"
---

文章翻译自：[Heap Sort Algorithm](https://www.programiz.com/dsa/heap-sort)

Heap Sort is a popular and efficient sorting algorithm in computer programming. Learning how to write the heap sort 
algorithm requires knowledge of two types of data structures - arrays and trees.

在计算机程序中堆排序是一种非常流行和效率的排序算法。学习怎样编写堆排序算法需要两种类型的数据结构知识：数组和树。

The initial set of numbers that we want to sort is stored in an array e.g. \[10, 3, 76, 34, 23, 32] and after sorting, 
we get a sorted array \[3,10,23,32,34,76]

排序之前我们想要排序的数组为：\[10, 3, 76, 34, 23, 32]，经过排序之后数组为：\[3,10,23,32,34,76]。

Heap sort works by visualizing the elements of the array as a special kind of complete binary tree called heap.

堆排序的工作原理就是把数组中元素当作一种特殊的完全二叉树，也被称作为堆。

<!--more-->

### What is a complete Binary Tree? 什么是完全二叉树？

**Binary Tree 二叉树**

Tree data structure in which each parent node can have at most two children

二叉树的定义就是每个父节点最多拥有两个子节点的树数据结构。

![](/images/categories/data_structure/010/BINARY-TREE.jpg)

In the above image, each element has at most two children.

在上图中，每一个节点最多包含两个子节点。

**Full Binary Tree 满二叉树**

A full Binary tree is a special type of binary tree in which every parent node has either two or no children.

满二叉树的定义就是每个父节点包含两个子节点或无子节点的一种特殊类型二叉树。

![](/images/categories/data_structure/010/FULL-TREE.jpg)

**Complete binary tree 完全二叉树**

A complete binary tree is just like a full binary tree, but with two major differences:

完全二叉树和满二叉树类似，但是有两个大的不同之处：

* Every level must be completely filled

* 每一层都必须完全被填充

* All the leaf elements must lean towards the left.

* 所有的叶子节点元素必须倾向于左节点，也就是只有一个子节点时，必须先填充左节点

* The last leaf element might not have a right sibling i.e. a complete binary tree doesn’t have to be a full binary tree.

* 最后一个叶子元素可能没有兄弟节点，也就是右节点。也就是完全二叉树不一定是满二叉树。

![](/images/categories/data_structure/010/BINARY-FULL-COMPLETE-COMPARISON_0.jpg)

### How to create a complete binary tree from an unsorted list (array)? 如何从无序的集合（数组）创建一个完全二叉树？

* Select first element of the list to be the root node. (First level - 1 element)

* 选择集合中的第一个元素为根节点（第一层级，只有一个元素）

* Put the second element as a left child of the root node and the third element as a right child. (Second level - 2 elements)

* 把集合中的第二元素作为根节点的左节点，第三个元素作为根节点的右节点（第二层级，二个元素）

* Put next two elements as children of left node of second level. Again, put the next two elements as children of right 
node of second level (3rd level - 4 elements).

* 把集合中下两个元素分别当作第二层级左节点的左右节点。同样地，把下两个元素分别当作第二层级右节点的左右节点（第三层级，四个元素）

* Keep repeating till you reach the last element.

* 如此重复，直至最后一个元素。

![](/images/categories/data_structure/010/Building-complete-tree.jpg)

### Relationship between array indexes and tree elements 集合索引值和树元素之间的关系

Complete binary tree has an interesting property that we can use to find the children and parents of any node.

完全二叉树有一个非常有趣的特性，我们可以用来在任何节点查找其的父节点和子节点。

If the index of any element in the array is _i_, the element in the index _2i+1_ will become the left child and element in 
_2i+2_ index will become the right child. Also, the parent of any element at index _i_ is given by the lower bound of _(i-1)/2_.

如果任意的一个元素，在集合中的索引值为 _i_，那么索引值为 _2i+1_ 的这个元素就是其的左节点，索引值为 _2i+2_ 的这个元素就是其的右节点。同时，索引值为 _i_ 的任何元素的
父节点索引值是大于或等于 _(i-1)/2_ 的整型值。

![](/images/categories/data_structure/010/array-vs-heap-indices.jpg)

来测试以下：

Left child of 1 (index 0)
= element in (2*0+1) index 
= element in 1 index 
= 12


Right child of 1
= element in (2*0+2) index
= element in 2 index 
= 9

Similarly,
Left child of 12 (index 1)
= element in (2*1+1) index
= element in 3 index
= 5

Right child of 12
= element in (2*1+2) index
= element in 4 index
= 6

Let us also confirm that the rules holds for finding parent of any node

同时确认下规则来查找任何节点的父节点

Parent of 9 (position 2) 
= (2-1)/2 
= ½ 
= 0.5
~ 0 index 
= 1

Parent of 12 (position 1) 
= (1-1)/2 
= 0 index 
= 1

Understanding this mapping of array indexes to tree positions is critical to understanding how the Heap Data Structure 
works and how it is used to implement Heap Sort.

理解了数组索引值在树中的位置对应关系对理解堆排序数据结构是什么原理至关重要，以及它是怎样应用在堆排序中。

### What is Heap Data Structure ? 什么是堆数据结构？

Heap is a special tree-based data structure. A binary tree is said to follow a heap data structure if

堆是一种基于树的特殊数据结构。如果一个二叉树满足下面的条件，则被称作为堆数据结构：

* it is a complete binary tree

* 它是一完全二叉树

* All nodes in the tree follow the property that they are greater than their children i.e. the largest element is at the 
root and both its children and smaller than the root and so on. Such a heap is called a max-heap. If instead all nodes 
are smaller than their children, it is called a min-heap

* 树中的所有子节点都大于他们的孩子，比如，最大的元素在根节点以及它的孩子都小于根节点，如此类推。这样的堆叫最大堆。如果相反地，所有的节点都比子节点要
小，那么，这样的堆就叫最小堆。

![](/images/categories/data_structure/010/max-heap-min-heap.jpg)

Following example diagram shows Max-Heap and Min-Heap.

### How to “heapify” a tree 如何让树变成堆？

Starting from a complete binary tree, we can modify it to become a Max-Heap by running a function called heapify on all 
the non-leaf elements of the heap.

从完全二叉树的根节点开始，我们可以通过在所有的非叶子节点上运行一个heapify函数使之称为最大堆。

Since heapfiy uses recursion, it can be difficult to grasp. So let’s first think about how you would heapify a tree with 
just three elements.

因为heapfiy函数使用了递归，它可能难以控制。所以让我们先来看下只有三个元素是怎样应用heapify函数的。

```
heapify(array)
    Root = array[0]
    Largest = largest( array[0] , array [2*0 + 1]. array[2*0+2])
    if(Root != Largest)
          Swap(Root, Largest)
```

![](/images/categories/data_structure/010/heapify-base-case.jpg)

The example above shows two scenarios - one in which the root is the largest element and we don’t need to do anything. 
And another in which root had larger element as a child and we needed to swap to maintain max-heap property.

上面的示例展示了两种情形-一种是根节点就是最大的元素，我们不需要作任何调整。另一种是子节点是最大的元素，我们需要交换元素来保证最大堆特性。

If you’re worked with recursive algorithms before, you’ve probably identified that this must be the base case.

如果你之前接触过递归排序算法，你可能已经识别出上面两种是最基本的情形。

Now let’s think of another scenario in which there are more than one levels.

现在，让我们试想下另一种不止一个层级的情形。

![](/images/categories/data_structure/010/heapify-when-children-are-heaps.jpg)

The top element isn’t a max-heap but all the sub-trees are max-heaps.

顶层的元素不是最大堆，但是所有的子树都符合最大堆。

To maintain the max-heap property for the entire tree, we will have to keep pushing 2 downwards until it reaches its correct 
position.

为了保证整树的最大堆特性，我们可能需要往下比较两层知道它在合适的位置。

![](/images/categories/data_structure/010/heapfy-root-element-when-subtrees-are-max-heaps.jpg)

Thus, to maintain the max-heap property in a tree where both sub-trees are max-heaps, we need to run heapify on the root 
element repeatedly until it is larger than its children or it becomes a leaf node.

这样，为了保证树的最大堆特性，我们需要从根节点开始重复的执行heapify函数直到它大于它的子节点，或者它成了叶子节点为止。

We can combine both these conditions in one heapify function as

我们可以在一个heapify函数中结合上面的那些条件：

```
void heapify(int arr[], int n, int i)
{
   int largest = i;
   int l = 2*i + 1;
   int r = 2*i + 2;

   if (l < n && arr[l] > arr[largest])
     largest = l;

   if (r < n && arr[r] > arr[largest])
     largest = r;

   if (largest != i)
   {
     swap(arr[i], arr[largest]);

     // Recursively heapify the affected sub-tree
     heapify(arr, n, largest);
   }
}
```

This function works for both the base case and for a tree of any size. We can thus move the root element to the correct 
position to maintain the max-heap status for any tree size as long as the sub-trees are max-heaps.

这个函数适合上面的基本情形和任何大小的树。为了保证最大堆状态我们可以为任意大小的树移动根元素到合适的位置直到所有的节点都是最大堆。

### Build max-heap 构建最大堆

To build a max-heap from any tree, we can thus start heapifying each sub-tree from the bottom up and end up with a max-heap 
after the function is applied on all the elements including the root element.

In the case of complete tree, the first index of non-leaf node is given by n/2 - 1. All other nodes after that are leaf-nodes 
and thus don’t need to be heapified.

So, we can build a maximum heap as

```java 
// Build heap (rearrange array)
for (int i = n / 2 - 1; i >= 0; i--)
 heapify(arr, n, i);
```

![](/images/categories/data_structure/010/build-max-heap.jpg)

As show in the above diagram, we start by heapifying the lowest smallest trees and gradually move up until we reach the 
root element.

If you’ve understood everything till here, congratulations, you are on your way to mastering the Heap sort.

### Procedures to follow for Heapsort

* Since the tree satisfies Max-Heap property, then the largest item is stored at the root node.

* Remove the root element and put at the end of the array (nth position) Put the last item of the tree (heap) at the vacant place.

* Reduce the size of the heap by 1 and heapify the root element again so that we have highest element at root.

* The process is repeated until all the items of the list is sorted.

![](/images/categories/data_structure/010/heap_sort.jpg)

The code below shows the operation.

```java 
for (int i=n-1; i>=0; i--){
 // Move current root to end
 swap(arr[0], arr[i]);

 // call max heapify on the reduced heap
 heapify(arr, i, 0);
}
```

### Performance

Heap Sort has _O(nlogn)_ time complexities for all the cases ( best case, average case and worst case).

Let us understand the reason why. The height of a complete binary tree containing _n_ elements is _log(n)_

As we have seen earlier, to fully heapify an element whose subtrees are already max-heaps, we need to keep comparing the 
element with its left and right children and pushing it downwards until it reaches a point where both its children are smaller than it.

In the worst case scenario, we will need to move an element from the root to the leaf node making a multiple of _log(n)_
comparisons and swaps.

During the build_max_heap stage, we do that for _n/2_ elements so the worst case complexity of the build_heap step is 
_n/2*log(n)_ ~ _nlogn_.

During the sorting step, we exchange the root element with the last element and heapify the root element. For each element, 
this again takes _logn_ worst time because we might have to bring the element all the way from the root to the leaf. Since 
we repeat this _n_ times, the heap_sort step is also _nlogn_.

Also since the build_max_heap and heap_sort steps are executed one after another, the algorithmic complexity is not 
multiplied and it remains in the order of nlogn.

Also it performs sorting in _O(1)_ space complexity. Comparing with Quick Sort, it has better worst case ( _O(nlogn)_ ). 
Quick Sort has complexity _O(n^2)_ for worst case. But in other cases, Quick Sort is fast. Introsort is an alternative to 
heapsort that combines quicksort and heapsort to retain advantages of both: worst case speed of heapsort and average speed 
of quicksort.

### Application of Heap Sort

Systems concerned with security and embedded system such as Linux Kernel uses Heap Sort because of the _O(n log n)_ upper 
bound on Heapsort's running time and constant _O(1)_ upper bound on its auxiliary storage.

Although Heap Sort has _O(n log n)_ time complexity even for worst case, it doesn’t have more applications ( compared to 
other sorting algorithms like Quick Sort, Merge Sort ). However, its underlying data structure, heap, can be efficiently 
used if we want to extract smallest (or largest) from the list of items without the overhead of keeping the remaining 
items in the sorted order. For e.g Priority Queues.

### Heap Sort Implementation in different Programming Languages

**C++ Implementation**

```c++
// C++ program for implementation of Heap Sort
#include <iostream>
using namespace std;

void heapify(int arr[], int n, int i)
{
   // Find largest among root, left child and right child
   int largest = i;
   int l = 2*i + 1;
   int r = 2*i + 2;

   if (l < n && arr[l] > arr[largest])
     largest = l;

   if (right < n && arr[r] > arr[largest])
     largest = r;

   // Swap and continue heapifying if root is not largest
   if (largest != i)
   {
     swap(arr[i], arr[largest]);
     heapify(arr, n, largest);
   }
}

// main function to do heap sort
void heapSort(int arr[], int n)
{
   // Build max heap
   for (int i = n / 2 - 1; i >= 0; i--)
     heapify(arr, n, i);

   // Heap sort
   for (int i=n-1; i>=0; i--)
   {
     swap(arr[0], arr[i]);
     
     // Heapify root element to get highest element at root again
     heapify(arr, i, 0);
   }
}

void printArray(int arr[], int n)
{
   for (int i=0; i<n; ++i)
     cout << arr[i] << " ";
   cout << "\n";
}

int main()
{
   int arr[] = {1,12,9,5,6,10};
   int n = sizeof(arr)/sizeof(arr[0]);
   heapSort(arr, n);

   cout << "Sorted array is \n";
   printArray(arr, n);
}
```

**Java program for implementation of Heap Sort**

```java 
// Java program for implementation of Heap Sort
public class HeapSort
{	
  
    public void sort(int arr[])
    {
        int n = arr.length;
 
      	// Build max heap
        for (int i = n / 2 - 1; i >= 0; i--) {
          heapify(arr, n, i);
        }
            
 
		// Heap sort
        for (int i=n-1; i>=0; i--)
        {
            int temp = arr[0];
            arr[0] = arr[i];
            arr[i] = temp;
 
          	// Heapify root element
            heapify(arr, i, 0);
        }
    }
 
    void heapify(int arr[], int n, int i)
    {
      	// Find largest among root, left child and right child
        int largest = i; 
        int l = 2*i + 1; 
        int r = 2*i + 2;  
 
        if (l < n && arr[l] > arr[largest])
            largest = l;
 
        if (r < n && arr[r] > arr[largest])
            largest = r;
 
      	// Swap and continue heapifying if root is not largest
        if (largest != i)
        {
            int swap = arr[i];
            arr[i] = arr[largest];
            arr[largest] = swap;
 
            heapify(arr, n, largest);
        }
    }
 
    static void printArray(int arr[])
    {
        int n = arr.length;
        for (int i=0; i < n; ++i)
            System.out.print(arr[i]+" ");
        System.out.println();
    }
 
    public static void main(String args[])
    {
        int arr[] = {1,12,9,5,6,10};

        HeapSort hs = new HeapSort();
        hs.sort(arr);
 
        System.out.println("Sorted array is");
        printArray(arr);
    }
}
```

**Python program for implementation of heap sort (Python 3)**

```python
def heapify(arr, n, i):
    # Find largest among root and children
    largest = i
    l = 2 * i + 1
    r = 2 * i + 2 
 
    if l < n and arr[i] < arr[l]:
        largest = l
 
    if r < n and arr[largest] < arr[r]:
        largest = r
 
    # If root is not largest, swap with largest and continue heapifying
    if largest != i:
        arr[i],arr[largest] = arr[largest],arr[i]
        heapify(arr, n, largest)
 
def heapSort(arr):
    n = len(arr)
 
    # Build max heap
    for i in range(n, 0, -1):
        heapify(arr, n, i)
 
    
    for i in range(n-1, 0, -1):
        # swap
        arr[i], arr[0] = arr[0], arr[i]  
        
        #heapify root element
        heapify(arr, i, 0)
 

arr = [ 12, 11, 13, 5, 6, 7]
heapSort(arr)
n = len(arr)
print ("Sorted array is")
for i in range(n):
    print ("%d" %arr[i])
```