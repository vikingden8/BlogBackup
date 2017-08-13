---
title: Inline vs Inline-Block Display in CSS
date: 2017-08-13 23:12:33
tags:
categories: "Web学习笔记"
---

>**display: inline-block** brought a new way to create side by side boxes that collapse and wrap properly depending on the available space in the containing element. It makes layouts that were previously accomplished with floats easier to create. No need to clear floats anymore.

Compared to **display: inline**, the major difference is that inline-block allows to set a width and height on the element. Also, with **display: inline**, top and bottom margins & paddings are not respected, and with **display: inline-block** they are.

Now, the difference between **display: inline-block** and **display: block** is that, with **display: block**, a line break happens after the element, so a block element doesn’t sit next to other elements. Here are some visual examples:

<!--more-->

### display: inline

Notice here how the width and height are not respected, and how the padding top and bottom are present, but overlap over the lines above and under.

![](/images/categories/web/026/01.png)

### display: inline-block

Here the width, height and padding are respected, but the two copies of the element can still sit side by side.

![](/images/categories/web/026/02.png)

### display: block

Here again everything is respected, but the elements don’t sit side by side.

![](/images/categories/web/026/03.png)

From:[Inline vs Inline-Block Display in CSS](https://alligator.io/css/display-inline-vs-inline-block/)
