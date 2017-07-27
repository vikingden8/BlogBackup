---
title: Inheritance, Cascading, and Specificity in CSS Explained
date: 2017-07-27 22:11:21
tags:
categories: "Web学习笔记"
---

### Introduction

If we boil down web development into its core components, it’ll probably look something like this: HTML represents the structural layer, CSS represents the presentation layer, and JavaScript represents the behavioral layer. And if you’re a visual person like me, you’ll be interested in learning CSS because … well because you’ll want to make your web apps look amazing! This blog post will focus specifically on an aspect of CSS that’s often challenging for beginners: **how exactly does a HTML element end up looking the way it does on a web page**?

### Let’s Talk About Inheritance

When you think of the word **“inheritance”** your first thought is probably related to how children inherit certain physical characteristics from their parents — if both your parents have blue eyes, you’ll probably inherit those same blue eyes. Well, understanding CSS inheritance isn’t too far off from this simple concept. Let’s take a look at the following HTML and CSS snippet:

<!--more-->

![](/images/categories/web/019/CSS-Inheritance.png)

The first thing to notice is that we’ve created a CSS class called **“container”** and associated styles with it (namely, font-family, font-size, and border). The second thing to notice is that we’ve applied the **“container”** class to a **div** tag, and within the **div** tag, we’ve include a **p** tag. So how does this relate to CSS inheritance? Well, notice how the text inside the **p** tag is getting styled with the **font-family** and **font-size** styles even though the **“container”** class is applied to the **div** tag. How’d that happen?? It’s all thanks for CSS inheritance — in other words, because the **p** tag is a child of the **div** tag, it inherited those styles as well!

Those with a keen eye will notice something else — what happened to the **“border”** style? Why is there a border on only the **div** tag and not on both the **div** and **p** tags? So just like real-life inheritance, you don’t inherit all characteristics from your parents and in much the same way, not all CSS styles are inheritable from parent to children. In this case, the **“border”** style is one of those styles that doesn’t get automatically inherited from parent to children.

### What does Cascading in Cascading Style Sheets (CSS) mean?

In addition to understanding inheritance, the other key to dissecting how exactly a HTML element ends up looking the way it does is understanding what “cascading” means.

When we talked about inheritance, it was always about styles that weren’t directly applied to a particular HTML tag but nonetheless ended up styling the tag. In our above example, this was the **p** tag — even though it didn’t have the **“font-family”** and **“font-size”** styles applied directly to it, those styles were inherited from its parent **div** tag. When we’re talking about cascading, what this refers to are those CSS styles that are directly applied to a particular tag. Let’s take a look at another HTML and CSS snippet:

![](/images/categories/web/019/CSS-cascade.png)

This snippet shows 2 CSS rules, one for all **p** tags and another for tags with the **“special-paragraph”** class applied to it. The snippet also has 2 **** tags in the HTML, and one of them has the **“special-paragraph”** class applied to it.

What’s interesting to note is the following:

* Notice how both **p** tags have the **“font-family”** and **“font-size”** styles applied to it.

* But also notice how the **p** tag with the **“special-paragraph”** class applied to it overrode the **“font-size: 14px”** style from the previous CSS rule with its own **“font-size: 24px”** style.

* And in addition to overriding a style, the **“special-paragraph”** class gave the second **p** tag an additional **“color: red”** style.

If all this is confusing, let me give a simple summary:

The way CSS works is that if there are any styles that are directly applied to a HTML tag, then that particular style will impact the tag. Where the “Cascading” part of CSS comes into place is when there are multiple styles that are directly applied to a HTML tag. And in the case of multiple styles, the actual final style applied to a HTML tag is the most “specific” one.

Using our above example, both **p** tags will have the **“font-family”** style applied to it (because of the p tag CSS rule). Also as a base, both **p** tags will have a **font-size** of 14px’s but notice that the second **p** tag has 2 conflicting CSS styles for **“font-size”** — and in this case, it resolved to applied the more specific **“font-size: 24px”** style. The concept of CSS specificity is what we’ll discuss next, but before that, I want to share something that trips up a lot of CSS beginners.

Many CSS beginners think of CSS rules as **“all-or-nothing”** meaning that if a CSS rule is applied to a tag, then only those styles in that particular rule will be applied. That’s not the case — in fact, the way CSS works is that for each and every HTML tag, it’ll find all the CSS styles that apply to that particular tag and it’ll apply the most specific one for each style. That’s exactly how **“font-size”** worked for the second **p** tag in our above example; we had 2 CSS rules with different values for the **“font-size”** style, and the more specific style (the one from the “special-paragraph” rule) ended up being applied to the second **p** tag. And that’s what “cascading” means in Cascading Style Sheets — more than one rule can apply to a particular HTML tag and there’s a hierarchy to determining which styles from the rules get applied to the particular HTML tag. Speaking of hierarchy, this leads up to the next topic, which is CSS specificity.

### What do you mean one CSS rule is more specific than another?

In our previous example, I glossed over why the **“font-size: 24px”** style was more specific than the **“font-size: 14px”** style. Now it’s time to address this point. In order to determine which of the **“font-size”** styles get applied to the second **p** tag, CSS followed a set of instructions to compute a specificity number associated with each CSS rule, and the CSS rule with the largest number will have its style applied to the tag.

While there’s some complexity in understanding the complete set of instructions to calculating the specificity number associated with each CSS rule, the following article explains a great general heuristic that works for almost all cases:

* **Element selectors** have a specificity value of **1**.

* **Class selectors** have a specificity value of **10**.

* **ID selectors** have a specificity value of **100**.

If you now return to our previous example, you’ll understand why the **“font-size: 24px”** style was applied to the second **p** tag:

The **p** tag selector has a specificity value of **1** while the **“special-paragraph”** class selector has a specificity value of **10**.

And that’s all there is to CSS specificity — doesn’t seem as daunting now, right?

### Conclusion

So that’s a quick guide to understanding how CSS works! And while it doesn’t cover many of the intricacies associated with CSS, I hope that it gives you a solid enough understanding to continue exploring CSS on your own!

I’ll finish off this post with a quick outline of some key points:

* CSS inheritance refers to the relationship between HTML tags (think parent and children tags) and how certain CSS styles can apply to a tag even though there aren’t any CSS rules directly applied to it.

* Cascading refers to the fact that cumulative styles across multiple CSS rules are applied to each and every HTML tag.
And in the event of conflicting styles between multiple CSS rules, the CSS rule with the highest specificity number will be the one which wins and gets applied to the particular HTML tag.

* As a bonus for reading this far, some extra tidbit not mentioned above: in the event that multiple CSS rules have the same specificity number, then the the CSS rule that appears last in the stylesheet order of the HTML document will win out.


From:[Inheritance, Cascading, and Specificity in CSS Explained](http://altitudelabs.com/blog/quick-guide-to-understanding-css/)
