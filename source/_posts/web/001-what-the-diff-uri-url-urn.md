---
title: 你知道URL、URI和URN三者之间的区别吗？
date: 2017-01-14 14:45:35
tags:
categories: "Web学习笔记"
---

这是一个经典的技术争论，许多人都会自问：URL、URI，很可能还有URN，它们之间的区别是什么。虽然，现在我们简单地把 URN 和 URL 都看做 URI，但严格来说URI可以进一步划分为URL、URN或者这两者的组合，所以了解这三者之间的区别将会非常有趣并让人受益匪浅。如果你恰好在某个地方碰到了这些东西，那么至少应该知道它们的含义。

我认为，尽管对一般人来说，不了解这三个缩略词之间的技术差异以及它们各自的含义并不是什么问题。但是，如果你作为一个计算机科学家、一个Web开发者、一个系统管理员，或者更笼统地说，你工作在IT领域，那么了解这些知识就非常有必要了。

这篇文章旨在于清楚地讲解URL、URI和URN之间的区别，帮助你快速理解这些必备知识。

预备知识，三者的英文全名：

  * URI：Uniform Resource Identifier，统一资源标识符；

  * URL：Uniform Resource Locator，统一资源定位符；

  * URN：Uniform Resource Name，统一资源名称。

### 起源

这三个缩略词是Tim Berners-Lee在一篇名为[RFC 3986: Uniform Resource Identifier (URI): Generic Syntax](https://tools.ietf.org/html/rfc3986)的文档中定义的互联网标准追踪协议。

<!--more-->

>统一资源标识符（URI）提供了一个简单、可扩展的资源标识方式。URI规范中的语义和语法来源于万维网全球信息主动引入的概念，万维网从1990年起使用这种标识符数据，并被描述为“万维网中的统一资源描述符”。

![](/images/categories/web/tim-berners-lee.jpg)

Tim Berners-Lee ,万维网的发明者，同时也是万维网联盟（W3C）的负责人

### 区别

首先我们要弄清楚一件事：**URL和URN都是URI的子集**。

换言之，URL和URN都是URI，但是URI不一定是URL或者URN。为了更好的理解这个概念，看下面这张图片:

![](/images/categories/web/uri_url_urn.jpg)

通过下面的例子（源自 [Wikipedia](https://en.wikipedia.org/wiki/Uniform_resource_identifier)），我们可以很好地理解URN 和 URL之间的区别。如果是一个人，我们会想到他的姓名和住址。

URL类似于住址，它告诉你一种寻找目标的方式（在这个例子中，是通过街道地址找到一个人）。要知道，上述定义同时也是一个URI。

相对地，我们可以把一个人的名字看作是URN；因此可以用URN来唯一标识一个实体。由于可能存在同名（姓氏也相同）的情况，所以更准确地说，人名这个例子并不是十分恰当。更为恰当的是书籍的ISBN码和产品在系统内的序列号，尽管没有告诉你用什么方式或者到什么地方去找到目标，但是你有足够的信息来检索到它。

所有的URN都遵循如下语法（引号内的短语是必须的）：

> < URN > ::= "urn:" < NID > ":" < NSS >

其中NID是命名空间标识符，NSS是标识命名空间的特定字符串。

### 一个用于理解这三者的例子

我们来看一下上述概念如何应用于与我们息息相关的互联网。

再次引用[Wikipedia](https://en.wikipedia.org/wiki/Uniform_resource_identifier#URLs) ，这些引文给出的解释，比上面人员地址的例子更为专业：

关于URL：

>URL是URI的一种，不仅标识了Web 资源，还指定了操作或者获取方式，同时指出了主要访问机制和网络位置。

关于URN：

>URN是URI的一种，用特定命名空间的名字标识资源。使用URN可以在不知道其网络位置及访问方式的情况下讨论资源。

现在，如果到Web上去看一下，你会找出很多例子，这比其他东西更容易让人困惑。我只展示一个例子，非常简单清楚地告诉你在互联网中URI 、URL和URN之间的不同。

我们一起来看下面这个虚构的例子。这是一个URI：

>http://bitpoetry.io/posts/hello.html#intro

我们开始分析"http://",是定义如何访问资源的方式。另外"bitpoetry.io/posts/hello.html",是资源存放的位置，那么，在这个例子中，"#intro"是资源。

URL是URI的一个子集，告诉我们访问网络位置的方式。在我们的例子中，URL应该如下所示：

>	http://bitpoetry.io/posts/hello.html

URN是URI的子集，包括名字（给定的命名空间内），但是不包括访问方式，如下所示：

> bitpoetry.io/posts/hello.html#intro

就是这样。现在你应该能够辨别出URL和URN之间的不同。

如果你忘记了这篇文章的内容，至少要记住一件事：URI可以被分为URL、URN或两者的组合。如果你一直使用URI这个术语，就不会有错。

原文地址：[你知道URL、URI和URN三者之间的区别吗？](http://web.jobbole.com/83452/)

### 参考资料

[Uniform Resource Name](https://en.wikipedia.org/wiki/Uniform_Resource_Name)

[Difference between URL, URI and URN](http://bitpoetry.io/difference-between-url-uri-and-urn/)

[What is the difference between URI, URL and URN?](http://stackoverflow.com/questions/4913343/what-is-the-difference-between-uri-url-and-urn)

[What is the difference between a URI, a URL and a URN?](http://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn/1984225#1984225)

[URL vs URI vs URN](https://prateekvjoshi.com/2014/02/22/url-vs-uri-vs-urn/)