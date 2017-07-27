---
title: !important CSS Declarations: How and When to Use Them
date: 2017-07-27 21:18:32
tags:
categories: "Web学习笔记"
---

### What is Important CSS?

An **!important** declaration provides a way for a stylesheet author to give a CSS value **more weight** than it naturally has.


### Syntax and Description

Here is a simple code example that clearly illustrates how **!important** affects the natural way that styles are applied:

```css
#example {
	font-size: 14px !important;
}

#container #example {
	font-size: 10px;
}
```

<!--more-->

In the above code sample, the element with the id of “example” will have text sized at 14px, due to the addition of **!important**.

Without the use of **!important**, there are two reasons why the second declaration block should naturally have more weight than the first: The second block is **later** in the stylesheet (i.e. it’s listed second). Also, the second block has more specificity (**#container** followed by **#example** instead of just **#example**). But with the inclusion of **!important**, the first font-size rule now has more weight.

Some things to note about **!important** declarations:

* When **!important** was first introduced in CSS1, an author rule with an **!important** declaration held more weight than a user rule with an **!important** declaration; to improve accessibility, this was reversed in CSS2

* If **!important** is used on a shorthand property, this adds “importance” to all the sub-properties that the shorthand property represents

* The **!important** keyword (or statement) **must be** placed at the end of the line, immediately before the semicolon, otherwise it will have no effect (although a space before the semicolon won’t break it)

* If for some particular reason you have to write the same property twice in the same declaration block, then add **!important** to the end of the first one, the first one will have more weight in every browser except IE6 (this works as an IE6-only hack, but doesn’t invalidate your CSS)

* In IE6 and IE7, if you use a different word in place of **!important** (like !hotdog), the CSS rule will still be given extra weight, while other browsers will ignore it

### When Should !important Be Used?

As with any technique, there are pros and cons depending on the circumstances. So when should it be used, if ever? Here’s my subjective overview of potential valid uses.

#### NEVER

**!important** declarations should not be used unless they are absolutely necessary after all other avenues have been exhausted. If you use **!important** out of laziness, to avoid proper debugging, or to rush a project to completion, then you’re abusing it, and you (or those that inherit your projects) will suffer the consequences.

If you include it even sparingly in your stylesheets, you will soon find that certain parts of your stylesheet will be harder to maintain. As discussed above, CSS property importance happens naturally through the cascade and specificity. When you use **!important**, you’re disrupting the natural flow of your rules, giving more weight to rules that are undeserving of such weight.

If you never use **!important**, then that’s a sign that you understand CSS and give proper forethought to your code before writing it.

That being said, the old adage “never say never” would certainly apply here. So below are some legitimate uses for **!important**.

#### TO AID OR TEST ACCESSIBILITY

As mentioned, user stylesheets can include **!important** declarations, allowing users with special needs to give weight to specific CSS rules that will aid their ability to read and access content.

A special needs user can add **!important** to typographic properties like font-size to make text larger, or to color-related rules in order to increase the contrast of web pages.

In the screen grab below, Smashing Magazine’s home page is shown with a user-defined stylesheet overriding the normal text size, which can be done using Firefox’s Developer Toolbar:

![](/images/categories/web/017/screen-grab-below1.jpg)

In this case, the text size was adjustable without using **!important**, because a user-defined stylesheet will override an author stylesheet regardless of specificity. If, however, the text size for body copy was set in the author stylesheet using an **!important** declaration, the user stylesheet could not override the text-size setting, even with a more specific selector. The inclusion of **!important** resolves this problem and keeps the adjustability of text size within the user’s power, even if the author has abused **!important**.

#### TO TEMPORARILY FIX AN URGENT PROBLEM

There will be times when something bugs out in your CSS on a live client site, and you need to apply a fix very quickly. In most cases, you should be able to use Firebug or another developer tool to track down the CSS code that needs to be fixed. But if the problem is occurring on IE6 or another browser that doesn’t have access to debugging tools, you may need to do a quick fix using **!important**.

After you move the temporary fix to production (thus making the client happy), you can work on fixing the issue locally using a more maintainable method that doesn’t muck up the cascade. When you’ve figured out a better solution, you can add it to the project and remove **!important** — and the client will be none the wiser.

#### TO OVERRIDE STYLES WITHIN FIREBUG OR ANOTHER DEVELOPER TOOL

Inspecting an element in Firebug or Chrome’s developer tools allows you to edit styles on the fly, to test things out, debug, and so on — without affecting the real stylesheet. Take a look at the screen grab below, showing some of Smashing Magazine’s styles in Chrome’s developer tools:

![](/images/categories/web/017/overriding-styles.jpg)

The highlighted background style rule has a line through it, indicating that this rule has been overridden by a later rule. In order to reapply this rule, you could find the later rule and disable it. You could alternatively edit the selector to make it more specific, but this would give the entire declaration block more specificity, which might not be desired.

**!important** could be added to a single line to give weight back to the overridden rule, thus allowing you to test or debug a CSS issue without making major changes to your actual stylesheet until you resolve the issue.

Here’s the same style rule with **!important** added. You’ll notice the line-through is now gone, because this rule now has more weight than the rule that was previously overriding it:

![](/images/categories/web/017/overriding-styles-2.jpg)

#### TO OVERRIDE INLINE STYLES IN USER-GENERATED CONTENT

One frustrating aspect of CSS development is when user-generated content includes inline styles, as would occur with some WYSIWYG editors in CMSs. In the CSS cascade, inline styles will override regular styles, so any undesirable element styling that occurs through generated content will be difficult, if not impossible, to change using customary CSS rules. You can circumvent this problem using an **important** declaration, because a CSS rule with **!important** in an author stylesheet will override inline CSS.

#### FOR PRINT STYLESHEETS

Although this wouldn’t be necessary in all cases, and might be discouraged in some cases for the same reasons mentioned earlier, you could add **!important** declarations to your print-only stylesheets to help override specific styles without having to repeat selector specificity.

#### FOR UNIQUELY-DESIGNED BLOG POSTS

If you’ve dabbled in uniquely-designed blog posts (many designers take issue with using “art direction” for this technique, and rightly so), as showcased on Heart Directed, you’ll know that such an undertaking requires each separately-designed article to have its own stylesheet, or else you need to use inline styles. You can give an individual page its own styles using the code presented in this post on the Digging Into WordPress blog.

The use of **!important** could come in handy in such an instance, allowing you to easily override the default styles in order to create a unique experience for a single blog post or page on your site, without having to worry about natural CSS specificity.

### Conclusion

**!important** declarations are best reserved for special needs and users who want to make web content more accessible by easily overriding default user agent or author stylesheets. So you should do your best to give your CSS proper forethought and avoid using **!important** wherever possible. Even in many of the uses described above, the inclusion of **!important** is not always necessary.

Nonetheless, **!important** is valid CSS. You might inherit a project wherein the previous developers used it, or you might have to patch something up quickly — so it could come in handy. It’s certainly beneficial to understand it better and be prepared to use it should the need arise.

From:[](https://www.smashingmagazine.com/2010/11/the-important-css-declaration-how-and-when-to-use-it/)
