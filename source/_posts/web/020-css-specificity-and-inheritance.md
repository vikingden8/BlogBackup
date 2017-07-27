---
title: CSS Specificity And Inheritance
date: 2017-07-27 22:27:05
tags:
categories: "Web学习笔记"
---

### Importance

Style sheets can have a few different sources:

#### **User agent**

  For example, the browser’s default style sheet.

#### **User**

  Such as the user’s browser options.

#### **Author**

  This is the CSS provided by the page (whether inline, embedded or external)

<!--more-->

By default, this is the order in which the different sources are processed, so the author’s rules will override those of the user and user agent, and so on.

There is also the **!important** declaration to consider in the cascade. This declaration is used to balance the relative priority of user and author style sheets. While author style sheets take precedence over user ones, if a user rule has **!important** applied to it, it will override even an author rule that also has **!important** applied to it.

Knowing this, let’s look at the final order, in ascending order of importance:

* User agent declarations,

* User declarations,

* Author declarations,

* Author **!important** declarations,

* User **!important** declarations.

This flexibility in priority is key because it allows users to override styles that could hamper the accessibility of a website. (A user might want a larger font or a different color, for example.)

### Specificity

Every CSS rule has a particular weight, meaning it could be more or less important than the others or equally important. This weight defines which properties will be applied to an element when there are conflicting rules.

Upon assessing a rule’s importance, the cascade attributes a specificity to it; if one rule is more specific than another, it overrides it.

If two rules share the same weight, source and specificity, the later one is applied.

#### HOW TO CALCULATE SPECIFICITY?

There are several ways to calculate a selector’s specificity.

The quickest way is to do the following. Add **1** for each element and pseudo-element (for example, :before and :after); add **10** for each attribute (for example, [type=”text”]), class and pseudo-class (for example, :link or :hover); add **100** for each ID; and add **1000** for an inline style.

Let’s calculate the specificity of the following selectors using this method:

* **p.note**

  1 class + 1 element = 11

* **#sidebar p[lang="en"]**

  1 ID + 1 attribute + 1 element = 111

* **body #main .post ul li:last-child**

  1 ID + 1 class + 1 pseudo-class + 3 elements = 123

A similar method, described in the W3C’s specifications, is to start with a=0, b=0, c=0 and d=0 and replace the numbers accordingly:

* a = 1 if the style is inline,

* b = the number of IDs,

* c = the number of attribute selectors, classes and pseudo-classes,

* d = the number of element names and pseudo-elements

Let’s calculate the specificity of another set of selectors:

* **<p style="color:#000000;">**

  a=1, b=0, c=0, d=0 → 1000

* **footer nav li:last-child**

  a=0, b=0, c=1, d=3 → 0013

* **#sidebar input:not([type="submit"])**

  a=0, b=1, c=1, d=1 → 0111

(Note that the negation pseudo-class doesn’t count, but the selector inside it does.)

Remember that non-CSS presentational markup is attributed with a specificity of 0, which would apply, for example, to the font tag.

Getting back to the !important declaration, keep in mind that using it on a shorthand property is the same as declaring all of its sub-properties as !important (even if that would revert them to the default values).

If you are using imported style sheets (@import) in your CSS, you have to declare them before all other rules. Thus, they would be considered as coming before all the other rules in the CSS file.

Finally, if two selectors turn out to have the same specificity, the last one will override the previous one(s).

#### MAKING SPECIFICITY WORK FOR YOU

If not carefully considered, specificity can come back to haunt you and lead you to unwittingly transform your style sheets into a complex hierarchy of unnecessarily complicated rules.

You can follow a few guidelines to avoid major issues:

* When starting work on the CSS, use generic selectors, and add specificity as you go along;

* Using advanced selectors doesn’t mean using unnecessarily complicated ones;

* Rely more on specificity than on the order of selectors, so that your style sheets are easier to edit and maintain (especially by others).

A good rule of thumb can be found in Jim Jeffers’ article, “The Art and Zen of Writing CSS”:

>Refactoring CSS selectors to be less specific is exponentially more difficult than simply adding specific rules as situations arise.

From:[CSS Specificity And Inheritance](https://www.smashingmagazine.com/2010/04/css-specificity-and-inheritance/)
