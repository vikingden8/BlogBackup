---
title: What is the differences between shallow and retained heap
date: 2018-01-11 17:21:36
tags:
categories: "Android学习笔记"
---


下面是一篇关于Shallow Heap和Retained Heap区别的文章，转载自：[ANDROID: WHAT IS THE DIFFERENCES BETWEEN SHALLOW AND RETAINED HEAP](https://www.yourkit.com/docs/java/help/sizes.jsp)

**Shallow size** of an object is the amount of memory allocated to store the object itself, not taking into account the referenced objects. Shallow size of a regular (non-array) object depends on the number and types of its fields. Shallow size of an array depends on the array length and the type of its elements (objects, primitive types). Shallow size of a set of objects represents the sum of shallow sizes of all objects in the set.

**Retained size** of an object is its shallow size plus the shallow sizes of the objects that are accessible, directly or indirectly, only from this object. In other words, the retained size represents the amount of memory that will be freed by the garbage collector when this object is collected.

<!--more-->

To better understand the notion of the **retained size**, let us look at the following examples:

In order to measure the retained sizes, all objects in memory are treated as nodes of a graph where its edges represent references from objects to objects. There are also special nodes - GC root objects, which will not be collected by Garbage Collector at the time of measuring.

The pictures below show the same set of objects, but with varying internal references.

![](/images/categories/android/android_notes/071/shallow_and_retained_heap.png)

Let us consider _obj1_.

As you can see, in both pictures we have highlighted all of the objects that are directly or indirectly accessed only by _obj1_. If you look at Figure 1, you will see that _obj3_ is not highlighted, because it is also referenced by a _GC root_ object. On Figure 2, however, it is already included into the retained set, unlike _obj5_, which is still referenced by _GC root_.

Thus, the retained size of _obj1_ will represent the following respective values:

* For Figure 1: the sum of shallow sizes of _obj1_, _obj2_ and _obj4_

* For Figure 2: the sum of shallow sizes of _obj1_, _obj2_, _obj3_ and _obj4_

Looking at _obj2_, however, we see that its retained size in the above cases will be:

* For Figure 1: the sum of shallow sizes of _obj2_ and _obj4_

* For Figure 2: the sum of shallow sizes of _obj2_, _obj3_ and _obj4_

In general, retained size is an integral measure, which helps to understand the structure (clustering) of memory and the dependencies between object subgraphs, as well as find potential roots of those subgraphs.
