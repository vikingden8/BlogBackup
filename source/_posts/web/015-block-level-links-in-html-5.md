---
title: “Block-level” links in HTML5
date: 2017-07-20 22:32:46
tags:
categories: "Web学习笔记"
---

One new and exciting thing you can do in HTML 5 is wrap links round “block-level” elements.

Imagine you have a front page with lots of teasers for news articles, each of which leads to a page devoted to the full text of that article. Obviously, each story needs a heading and you’ll want an image too, and you’ll want them all clickable. In current mark-up you’ll probably have something like this:

<!--more-->

```HTML
<div class="story">
<h3><a href="story1.html">Bruce Lawson voted sexiest man on Earth</a></h3>
<p><a href="story1.html"><img src="bruce.jpg" alt="full story. " />A congress representing all the planet's women unanimously voted Bruce Lawson as sexiest man alive.</a></p>
<p><a href="story1.html">Read more</a></p>
</div>
```

Note that you have three identical links (or two if, like the BBC News site, you assume your readers don’t need a “read more” link).

In HTML 5, you code it like this:

```html
<article>
<a href="story1.html">
<h3>Bruce Lawson voted sexiest man on Earth</h3>
<p><img src="bruce.jpg" alt="gorgeous lovebundle. ">A congress representing all the planet's women unanimously voted Bruce Lawson as sexiest man alive.</p>
<p>Read more</p>
</a>
</article>
```

Now the single link surrounds the whole teaser, removing duplication and creating a much wider hit area to click.

Two points to make about accessibility in passing: firstly, you don’t need to worry that each "read more" goes to a different destination because the whole story is the link, so the link text is unique. Secondly, note that I’ve changed the alternate text on the image, as in the first instance the image is a link so I described its destination, whereas the second is just a picture so I describe the image itself—”gorgeous lovebundle”—and thereby give more information to a screenreader user.

### Shiny, but not new

What’s also very interesting about this technique is that actually this isn’t new: you can do it now. XHTML 2 had a similar mechanism, which allowed href on any element Eric Meyer called for this to be adopted in HTML 5 but, of course, this isn’t backwardly compatible. One of his other solutions to the same problem was changing the rules for a, and my test page shows that it works now in all the browsers.

That’s one of the interesting things about HTML 5—it documents existing browser behaviour. As the browsers already handle wrapping links around block-level elements, and there is an obvious use-case, there was no reason to artificially keep the structure as invalid.



Origin Post Url :[“Block-level” links in HTML5](http://html5doctor.com/block-level-links-in-html-5/)
