---
title: MDN box-sizing
date: 2017-08-08 22:14:12
tags:
categories: "Web学习笔记"
---

The CSS box-sizing property is used to alter the default CSS box model used to calculate width and height of the elements.

```sh
/* Keyword values */
box-sizing: content-box;
box-sizing: border-box;

/* Global values */
box-sizing: inherit;
box-sizing: initial;
box-sizing: unset;
```

<!--more-->

In CSS, by default, the width and height you assign to an element is applied only to the element's content box. If the element has any border or padding, this is then added to the width and height to arrive at the size of the box that's rendered on the screen. This means that when you set width and height you have to adjust the value you give to allow for any border or padding that may be added. This is especially tricky when implementing a responsive design.

The **box-sizing** property can be used to adjust this behavior:

* **content-box** is the **default**, and gives you the default CSS box-sizing behavior. If you set an element's width to 100 pixels, then the element's content box will be 100 pixels wide, and the width of any border or padding will be added to the final rendered width.

* **border-box** tells the browser to account for any border and padding in the value you specify for width and height. If you set an element's width to 100 pixels, that 100 pixels will include any border or padding you added, and the content box will shrink to absorb that extra width. This typically makes it much easier to size elements.

Some experts recommend that web developers should consider routinely applying _box-sizing: border-box_ to all elements.

![](/images/categories/web/024/01.png)

The example above shows three scenarios. In each scenario there is a parent DIV (with an orange border) that contains a child DIV. The child has width: 100% set, and a pale blue background.

* The first scenario uses the default _box-sizing: content-box_. The child DIV has no padding and no border, and fits neatly inside its parent.

* The second scenario uses the default _box-sizing: content-box_. The child DIV has added padding and a border. The child then spills outside the parent because its width is calculated using only the content: padding and border are then added to make the rendered width.

* The third scenario uses _box-sizing: border-box_. The child DIV now fits neatly inside its parent because its width: 100% accounts for the padding and border.

### Syntax

The _box-sizing_ property is specified as a single keyword chosen from the list of values below.

#### Values

**content-box**

This is the initial and default value as specified by the CSS standard. The width and height properties are measured including only the _content_, but not the _padding_, _border_ or _margin_. For example, if you set .box {width: 350px;}, then apply {border: 10px solid black;} , then the rendered result is a box of width: 370px.

Here the dimensions of the element are calculated as:, width = width of the content, and height = height of the content (excluding the values of border and padding).

**border-box**

The width and height properties include the _content_, the _padding_ and _border_, but not the _margin_. Note that _padding_ and _border_ will be inside of the box e.g.  .box {width: 350px; border: 10px solid black;} leads to a box rendered in the browser of width: 350px. The content box can't be negative and is floored to 0, making it impossible to use border-box to make the element disappear.

Here the dimensions of the element are calculated as: width = border + padding + width of the content, and height = border + padding + height of the content.
