---
title: CSS中的quotes属性
date: 2019-7-14 10:31:08
tags:
categories: "CSS学习笔记"
---

在CSS中，quotes属性常常被用来指定当作content属性的插入值。

q和blockquote标签在渲染的时候是没有引用符号的，所以当有需要的时候我们必须自己去指定。这里需要注意的是，我们可以把引用符号添加到任何的元素上，而不仅仅去q或者blockquote标签。

<!--more-->

我们必须通过伪元素::before和::after的content属性来分别指定引用的开始和结束符号。content的属性值可以为open-quote和close-quote，open-quote用来指定引用开始的符号，close-quote用来指定引用结束的符号。

这里有几个问题就是，世界上不同的国家对引用有不同的符号来标记，那浏览器怎样知道在不同的语言环境下使用不同的引用标记呢？还有一种情况就是引用嵌套的问题，有可能在不同层次下需要使用不同的引用符号，同样地，那浏览器怎么知道在嵌套的情况下该使用哪种引用符号呢？

这时候，quotes属性就能派上用场了。

首先，通过使用quotes属性，你可以提供给浏览器在不同引用层次下的多对引用符号。多少对是没有限制的。具体完全取决于引用的嵌套层级。简单来说，如果嵌套的层级有三层，且定义了三对引用标记，那么这三对将会顺序的应用在嵌套层级上。假如，定义的引用标记不够，也就是说，小于引用的嵌套层级，那么，最后的一对引用符号将会重复地应用在其后的所有引用上。

且看下面的例子：

```css
q, blockquote {

    quotes: "“" "”"; /* one pair */

    /* OR */

    quotes: "“" "”" "‘" "’"; /* two pairs */
}
```

例如，quotes中左边的第一对，将会在最外层；然后，第二对，将在最外层的临近一层；以此类推。

 ![](/images/categories\css\quotes_in_css\quotes-pairs.png)

如上图所示，open-quote 的值其实就是引用符号对中的第一个引用符号，close-quote 的值就是相同引用符号对中的第二个引用符号。

下面是使用 open-quote 和 close-quote的示例：

```CSS
q {
    quotes: "“" "”" "‘" "’";
}

/* insert the quotes using pseudo-elements and the `content` property */

q::before {
    content: open-quote;
}

q::after {
    content: close-quote;
}
```

### 资料

* [quotes](https://tympanus.net/codrops/css_reference/quotes/)

* [quotes](https://css-tricks.com/almanac/properties/q/quotes/)

* [straight and curly quotes](https://typographyforlawyers.com/straight-and-curly-quotes.html)

* [Quotes & Accents](http://quotesandaccents.com/)
