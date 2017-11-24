---
title: 大Θ记号、大Ω记号、大Θ记号
date: 2017-11-23 22:26:57
categories: "数据结构"
---

**大Θ记号**

Big O nonation, 上界：最坏情况，以大O记号形式表示的时间复杂度，给出了一个算法的最坏情况，即对于规模为n的任意输入，算法的运行时间都不会超过O(f(n))，即存在正数c和N，对于所有的n>=N，有f(n) <= c * g(n)，则f(n)=O(g(n))，记为：T(n) = Ω(f(n))

**大Ω记号**

Big Omega，下界：最好情况, 如果存在正的常数c和函数g(n)，对任意n>>2，有T(n) > c * g(n)，即认为：在n足够 大后，g(n)给出了T(n)的一个下界，记为：T(n) = Ω(g(n))

**大Θ记号**

Big Theta,确界：大 Θ记号-->存在正的常数c1和c2，以及函数h(n)，对任意n>>2，有 c1*h(n) < T(n) < c2 * h(n)，即认为：在n足够大后，h(n)给出了T(n)的一个确界，记为：T(n) = Θ(g(n))

<!--more-->

下面是三者的图示：

![](/images/categories/data_structure/004/order_theta_omega.png)

**参考资料**

[大 Θ记号、大 Ω记号、空间复杂度、时间复杂度](https://www.cnblogs.com/joh-n-zhang/p/5759250.html)

[简明解释算法中的大O符号](http://blog.jobbole.com/55184/)

[为什么见周围人描述算法复杂度都用大 O 符号而不是大 Θ?](https://www.zhihu.com/question/20677334)
