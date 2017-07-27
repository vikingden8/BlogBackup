---
title: An Overview of CSS Inheritance
date: 2017-07-27 21:51:04
tags:
categories: "Web学习笔记"
---

An important part of styling a website with CSS is understanding the concept of inheritance.

CSS inheritance is automatically defined by the style of the property being used. When you look up the style property **background-color**, you'll see a section titled "Inheritance". If you're like most web designers, you've ignored that section, but it does serve a purpose.

<!--more-->

### WHAT IS CSS INHERITANCE?

Each element in an HTML document is part of a tree and every element except the initial **<html>** element has a parent element that encloses it.

Whatever styles are applied to that parent element can be applied to the elements enclosed in it if the properties are ones that can be inherited.

For example, this HTML code below has an **H1** tag enclosing an **EM** tag:

```HTML
<h1>This is a <em>Big</em> Headline</h1>
```

The **EM** element is a child of the **H1** element, and any styles on the **H1** that are inherited will be passed on to the **EM** text as well. For example:

```css
h1 { font-size: 2em; }
```

Since the **font-size** property is inherited, the text that say "Big" (which is what is enclosed inside the **EM** tags) will be the same size as the rest of the **H1**. This is because it inherits the value set in the CSS property.

### HOW TO USE CSS INHERITANCE

The easiest way to use it is to become familiar with the CSS properties that are and are not inherited. If the property is inherited, then you know that the value will remain the same for every child element in the document.

The best way to use this is to set your basic styles on a very high level element, like the **BODY**.

If you set your **font-family** on the body property, then, thanks to inheritance, the entire document will keep that same **font-family**. This will actually make for smaller stylesheets that are easier to manage because there are fewer overall styles. For example:

```css
body {font-family: Arial, sans-serif; }
```

### USE THE INHERIT STYLE VALUE

Every CSS property includes the value **"inherit"** as a possible option.

This tells the Web browser, that even if the property would not normally be inherited, it should have the same value as the parent. If you set a style such as a margin that is not inherited, you can use the inherit value on subsequent properties to give them the same margin as the parent. For example:

```css
body { margin: 1em; }
p { margin: inherit; }
```

### INHERITANCE USES COMPUTED VALUES

This is important for inherited values like font sizes that use lengths. A computed value is a value that is relative to some other value on the Web page.

If you set a **font-size** of 1em on your **BODY** element, your entire page will not be all only 1em in size. This is because elements like headings (**H1**-**H6**) and other elements (some browsers compute table properties differently) have a relative size in the Web browser. In the absence of other font size information, the Web browser will always make an **H1** headline the largest text on the page, followed by **H2** and so on. When you set your **BODY** element to a specific font size, then that is used as the "average" font size, and the headline elements are computed from that.

### A NOTE ABOUT INHERITANCE AND BACKGROUND PROPERTIES

There are a number of styles that are listed has not inherited in CSS 2 on the W3C, but the Web browsers still inherit the values.

For example, if you wrote the following HTML and CSS:

```HTML
<style type="text/css">
 h1 {background-color: yellow; }
 </style>

 <h1>This is a <em>Big</em> headline</h1>
```

The word "Big" would still have a yellow background, even though the **background-color** property is not supposed to be inherited. This is because the initial value of a background property is **"transparent"**. So you're not seeing the background color on the **<em>** but rather that color is shining through from the **<h1>** parent.

From:[An Overview of CSS Inheritance](https://www.thoughtco.com/css-inheritance-overview-3466210)
