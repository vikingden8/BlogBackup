---
title: [HTML/CSS]盒子模型
date: 2017-08-08 21:31:48
tags:
categories: "Web学习笔记"
---

css盒子模型分为两种，一种是遵循w3c标准的标准盒子模型，另外一种就是IE盒子模型。

### 标准盒子模型

![](/images/categories/web/022/01.png)

### IE盒子模型

![](/images/categories/web/022/02.png)

<!--more-->

**通过上面两张图可以看出，两种盒子模型都包括padding，margin，border，content，但是ie盒子模型的content包括border和padding**。

#### 一个例子

一个盒子的 margin 为 20px，border 为 1px，padding 为 10px，content 的宽为 200px、高为 50px。

#### 标准盒子模型

盒子需要占据的位置为：宽 20*2+1*2+10*2+200=262px、高 20*2+1*2*10*2+50=112px，

盒子的实际大小为：宽 1*2+10*2+200=222px、高 1*2+10*2+50=72px；

#### IE盒子模型

盒子需要占据的位置为：宽 20*2+200=240px、高 20*2+50=70px，

盒子的实际大小为：宽 200px、高 50px。

#### 选择多了，就要比个哪个好？

当然是“标准 W3C 盒子模型”了。怎么样才算是选择了“标准 W3C 盒子模型”呢？很简单，就是在网页的顶部加上 DOCTYPE 声明。

```xml
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

如果不加 DOCTYPE 声明，那么各个浏览器会根据自己的行为去理解网页，即 IE 浏览器会采用 IE 盒子模型去解释你的盒子，而 FF 会采用标准 W3C 盒子模型解释你的盒子，所以网页在不同的浏览器中就显示的不一样了。反之，如果加上了 DOCTYPE 声明，那么所有浏览器都会采用标准 W3C 盒子模型去解释你的盒子，网页就能在各个浏览器中显示一致了。
