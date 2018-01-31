---
title: Java中设置方法执行的超时时间
date: 2018-01-31 17:35:02
tags:
categories: "Java学习笔记"
---

**[java.util.concurrent.Future](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html)**

Future代表一个异步计算的结果。它提供了方法来检查是否计算已经完成，还是正在计算而处于等待状态，并且也提供了获取计算结果方法。当计算完成后，只能通过get方法来获取执行结果，必要的话该方法会阻塞。通过cancel方法可以取消计算。一旦计算已经完成，便无法取消。

<!--more-->

比如，要设置一个方法执行的超时时间：

![](/images/categories/java/033/01.png)

输出:

![](/images/categories/java/033/02.png)
