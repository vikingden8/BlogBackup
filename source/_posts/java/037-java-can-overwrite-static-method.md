---
title: Java中能否复写父类中的静态方法
date: 2018-03-03 21:01:02
tags:
categories: "Java学习笔记"
---

Java中能否复写父类中的静态方法，其实可以通过一个简单的示例来说明：

先是父类的代码:

```java
public class Animal {  

    public static void show() {  
        System.out.println("父类的静态方法");  
    }  

    public void method() {  
        System.out.println("父类的一般方法");  
    }  
}  
```

下面是子类的方法：

```java
public class Dog extends Animal {  

    public static void main(String[] args) {  
        Animal animal = new Dog();  
        animal.show();  
        animal.method();  
    }  

    public static void show() {  
        System.out.println("子类的静态方法");  
    }  

    public void method() {  
        System.out.println("子类的一般方法");  
    }  
      
}  
```

输出结果是：

```sh
父类的静态方法
子类的一般方法
```

其实，静态方法的调用最好是通过Animal.show()的方式来调用，回到正题，父类的静态方法不能被子类继承，更谈不上重写，就算是子类中有一个和父类一模一样的静态方法，那也是子类本身的，和父类的那个静态方法不是一回事，方法加静态后就属于类不属于对象了。
