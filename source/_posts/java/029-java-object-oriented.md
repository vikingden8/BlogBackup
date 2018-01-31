---
title: 面向对象
date: 2018-01-31 10:30:02
tags:
categories: "Java学习笔记"
---

本文转载自：[面向对象](https://zhuanlan.zhihu.com/p/27681007)

面试的时候，面试官问，你是怎么理解面向对象的编程？我想很多人会照着面试题来背吧，面向对象呀，不就是继承，封装，多态吗？注意，面试官问到这里，是想听听你对面向对象的理解，而不是让你来背概念。

面向对象，是对现实世界的模拟，下图我们简单模拟了一个动物世界。

![](/images/catgories/java/029/01.jpg)

<!--more-->

面向对象的三个基本特征之一继承，这里Primat继承了Animal，Person继承了Primat，继承很简单，就不多说了，看以下代码实现，代码中的注释比较重要，请重点看。

![](/images/catgories/java/029/02.jpg)

![](/images/catgories/java/029/03.jpg)

![](/images/catgories/java/029/04.jpg)

![](/images/catgories/java/029/05.jpg)

![](/images/catgories/java/029/06.jpg)

![](/images/catgories/java/029/07.jpg)

在代码中，不管是动物，鸟类，人类，猴子，我们都可以抽象成类，类是对象的模板，通过new关键字，可以创建一个个对象。仔细看蓝色框里的内容，**animal1和animal2，虽然都是同一个形态（Animal），由于指向的是子类对象，当调用同一个eat()方法，运行时会智能匹配到子类的实现，最后得到的结果也不一样，这种形为，我们称之为多态**。

多态要满足三个条件。

一、**要有继承**；（Person继承了Animal，Monkey也继承了Animal）
二、**要有重写**；（都重写了父类的eat方法）
三、**父类引用指向子类对象**。（红框内容）

再看下图，利用IDE的代码联想功能看一下，person4这个对象能访问到name、sex属性，eat()、printAge()方法，但无法访问age属性，isLady()方法。是因为我们在该属性和方法前面加了private关键字。隐藏了不想对客户端暴露的age属性和isLady()方法（这里的客户端是main方法），但是我们对客户端提供了一个printAge方法来打印年龄，但在打印年龄前，我们对年龄做了一系列处理（不打印女士年龄）。

![](/images/catgories/java/029/08.jpg)

**对于这种隐藏对象属性和实现细节，仅对外公开指定方法来控制程序中属性的访问和修改，我们称之为封装**。（这儿我们没有对age提供set方法，压根没法修改，提供了一个printAge()方法供外部访问）。

在工作中，我们可能会有这么一个需求，当Person对象的名字一致，年龄一致，性别一致，就当成同一人处理。

![](/images/catgories/java/029/09.jpg)

![](/images/catgories/java/029/10.jpg)

当用==和equals失败后，开发者可能会写下以下代码去做判断

![](/images/catgories/java/029/11.jpg)

写到年龄逻辑的时候发现，由于age属性是私有的，又没有提供getAge( )方法，获取不到age属性，可能会在Person这个类里提供一个getAge( )方法或把age的private关键字去掉，这样虽然也能完成逻辑，但会导致后续使用该类的人再也不调printAge去打印age了，而是直接访问age属性或getAge( )方法去打印，女士的age也就暴露了出去。很显然，**这种做法打破了我们之前对age的封装，不建议这么做**。

age是私有的，那我们把这段代码挪到Person对象里不就行了？好主意，**比较两个对象是否相等，不应该由客户端来决定，而是由对象本身来决定**。这也是面向对象的技巧之一。

有了以上思路，我们在Person里扩展了一个方法，

![](/images/catgories/java/029/12.jpg)

![](/images/catgories/java/029/13.jpg)

![](/images/catgories/java/029/14.jpg)

一切似乎很完美，结果也和我们期望的一样，问题似乎解决了。

我们知道Set里的元素无放入顺序,元素不可重复，请思考以下代码的执行结果：

![](/images/catgories/java/029/15.jpg)

希望看完本章后，读者对面向对象有了自己的一些看法和思考，而不是面试题中一味出现的六个字，多态，继承，封装。甚至你可以对面试官莞尔一笑，啊，我还没对象，有大把的时间投入到工作中来。

评论区里，有人说Animal，Primat应该声明为抽象类而不是接口，其实两种方式都是可以的。**如果Animal暂时不确定有属性或有具体的形为，建议先声明成接口，因为一个类可以实现多个接口，但只能继承一个类，声明为接口更灵活一些，也使实现类具有更好的扩展性**。