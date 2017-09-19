---
title: Java Reflection-Difference between getMethods() and getDeclaredMethods()
date: 2017-9-19 19:00:02
tags:
categories: "Java学习笔记"
---

### getMethods()

Returns an array containing Method objects reflecting all the public member methods of the class or interface represented by this Class object, including those declared by the class or interface and those inherited from superclasses and superinterfaces. Array classes return all the (public) member methods inherited from the Object class. The elements in the array returned are not sorted and are not in any particular order. This method returns an array of length 0 if this Class object represents a class or interface that has no public member methods, or if this Class object represents a primitive type or void.

The class initialization method <clinit> is not included in the returned array. If the class declares multiple public member methods with the same parameter types, they are all included in the returned array.

<!--more-->

此方法返回这个Class所代表的类或者接口的所有公共方法，也就是带public方法的。这些公共方法集合包括这个类或者接口中声明的，同时也包括继承类或者接口中的公共方法。

### getDeclaredMethods()

Returns an array of Method objects reflecting all the methods declared by the class or interface represented by this Class object. This includes public, protected, default (package) access, and private methods, but excludes inherited methods. The elements in the array returned are not sorted and are not in any particular order. This method returns an array of length 0 if the class or interface declares no methods, or if this Class object represents a primitive type, an array class, or void. The class initialization method <clinit> is not included in the returned array. If the class declares multiple public member methods with the same parameter types, they are all included in the returned array.

此方法返回这个Class所代表的类或者接口的所有方法，包括public,protected,default或者private的。但是会剔除掉继承的方法。

上述的两个方法返回的数组都是没有顺序的。