---
title: Understand the Favicon
date: 2017-07-19 22:29:43
tags:
categories: "Web学习笔记"
---

When Alec Rust asked the HTML5 Boilerplate project to switch to a HiDPI favicon, I realized how little I knew about _favorite icons_, _touch icons_, and _tile icons_. When I decided to dive in a little deeper, things got interesting.

Since they were first introduced by Internet Explorer in 1999, almost nothing about favicons has changed. They have almost-always been ICO files, either nested in the root of the domain as _/favicon.ico_, or organized by a CMS into a theme or images directory and displayed with:

```HTML5
<link rel="shortcut icon" href="/path/to/favicon.ico">
```

The classic favicon.ico is a 16×16 ICO file, often served in either 16-color or 24bit alpha-transparency format. More recently, favicons have been served as 32×32, which is appropriately scaled down in all major and popular-legacy browsers. In IE10 Metro, the 32×32 icon is used in the address bar.

![](/images/categories/web/014/ie10-address-bar.png)

<!--more-->

The _rel_ attribute of a favicon is a product of evolution. Internet Explorer 5 intended shortcut icon to represent the relationship between the page and the icon, but when the specification separated relationships by space, this theoretically created two relationships, _shortcut_ and _icon_. It wasn’t until 2010 when the HTML5 specification declared _icon_ alone to be the standard identifier. In non-IE browsers, favicons can be served without the _shortcut_ property.

```HTML5
<!-- IE6-10 -->
<link rel="shortcut icon" href="path/to/favicon.ico">

<!-- Everybody else -->
<link rel="icon" href="path/to/favicon.ico">
```

The _type_ attribute of a favicon is about as useful as the type attribute of a <script>. As of Janaury 16, 2013, Wikipedia hints that the favicon’s type attribute may effect whether or not Internet Explorer will correctly display it. In reality, Internet Explorer only cares about the server mime for the ICO file, and otherwise ignores the type attribute. The _type_ attribute can be anything, and it can be nothing.

```HTML5
<!-- Still works in IE6+ -->
<link rel="shortcut icon" href="path/to/favicon.ico" type="image/vnd.microsoft.icon">

<!-- Still works in IE6+ -->
<link rel="shortcut icon" href="path/to/favicon.ico" type="image/x-icon">

<!-- Still works in IE6+ -->
<link rel="shortcut icon" href="path/to/favicon.ico">
```

![](/images/categories/web/014/02.png)

This really depresses me, because _Chrome_, _Firefox_, _Opera 7+_, and _Safari 4+_ all accept the PNG favicon, but Chrome and Safari will opt to use the ICO favicon when both are presented, regardless of the order in which they are declared. On the other hand, Internet Explorer does not support PNG favicons, but it will ignore the PNG favicon and use the ICO favicon, regardless of the order in which they are declared.

```HTML5
<!-- Chrome, Safari, IE -->
<link rel="shortcut icon" href="path/to/favicon.ico">

<!-- Firefox, Opera (Chrome and Safari say thanks but no thanks) -->
<link rel="icon" href="path/to/favicon.png">
```

Since PNG favicon files do not include multiple resolutions like ICO favicons, we can write out several favicon declarations and use the sizes attribute to target each resolution.

```HTML5
<link rel="icon" href="favicon-16.png" sizes="16x16">
<link rel="icon" href="favicon-32.png" sizes="32x32">
<link rel="icon" href="favicon-48.png" sizes="48x48">
<link rel="icon" href="favicon-64.png" sizes="64x64">
<link rel="icon" href="favicon-128.png" sizes="128x128">
```

How do these PNG-favicon-compatible browsers determine which favicon should be used? Firefox and Safari will use the favicon that comes last. Chrome for Mac will use whichever favicon is ICO formatted, otherwise the 32×32 favicon. Chrome for Windows will use the favicon that comes first if it is 16×16, otherwise the ICO. If none of the aforementioned options are available, both Chromes will use whichever favicon comes first, exactly the opposite of Firefox and Safari. Indeed, Chrome for Mac will ignore the 16×16 favicon and use the 32×32 version if only to scale it back down to 16×16 on non-retina devices. Opera, not wanting to take sides, will choose from any of the available icons at complete random. I love that Opera does this.
And that’s just the beginning. Now it’s time to learn about the Internet Explorer caveats.

While IE8-10 will display the favicon on first load of the page, IE7 will skip the first load and display the favicon during repeat visits. Worse yet, IE6 will only display the favicon once the site has been bookmarked and reopened in the browser. IE6 will also drop the favicon whenever the browser cache is cleared, and it will not display the favicon again until the site is either re-bookmarked, or the favicon is somehow reloaded. If IE6 and favicons mean a lot to you, you can force this reload with a little JavaScript snippet, preferably wrapped in a conditional comment.

```HTML5
<!-- I "support" IE6 -->
<!--[if IE 6]><script>(new Image).src="path/to/favicon.ico"</script><![endif]-->
```

Back to HiDPI; have you asked yourself this question yet?

>If all good browsers support PNG favicons, and IE browsers need ICO favicons, but ICO favicons throw off Chrome and Safari, why not wrap the ICO favicon in IE conditional commments?

That is a great question, and it leads to a great idea. PNG files are a fraction the size of ICO files. We could serve a classic 32×32 ICO favicon to IE, and a super sleek 96×96 PNG favicon to everybody else.

```HTML5
<!-- Just IE? -->
<!--[if IE]><link rel="shortcut icon" href="path/to/favicon.ico"><![endif]-->

<!-- Everybody else? -->
<link rel="icon" href="path/to/favicon.png">
```

One. Big. Problem. **IE10 does not support conditional comments, and it does not support PNG favicons**. Yes, with the code above, legacy IE would get a better experience than Microsoft’s brightest flagship.

>Hey, hey — what if we stick the ICO favicon in the root directory and use <link rel="icon"> to assign the PNG favicon?

You. Win. The Internet! Given the limitations of Chrome, Safari, and IE, this method will give every browser the best favicon experience. IE will ignore the _<link rel="icon">_ and use the ICO favicon found in the root of the domain as _/favicon.ico_. All other browsers will use the PNG favicon displayed with:

```HTML5
<link rel="icon" href="path/to/favicon.png">
```

>But what if I want multiple favicons or my CMS doesn’t like to do things this way? Is there … another way?

Yea, but you’re not gonna like it.

```HTML5
<!-- I "support" IE -->
<script>
navigator.appName == "Microsoft Internet Explorer" && (function (i, d, s, l) {
   i.src = "favicon.ico";
   s = d.getElementsByTagName("script")[0];
   l = s.parentNode.insertBefore(d.createElement("link"), s);
   l.rel = "shortcut icon";
   l.href = i.src;
})(new Image, document);
</script>

<!-- Everybody else -->
<link rel="icon" href="path/to/favicon.png">
```

Unsatisfied with either solution? All is not lost. IE10 users are mostly Windows 8 users for now, and Windows 8 introduces a new kind of display icon for websites — _tile icons_.

With IE10 Metro we can display a unique tile icon when the visitor pins our site to their Start screen. These tile icons are 144×144 PNG files, and for best results they use a transparent background. A background tile color can be specified using a hex RGB color (using the six-character #RRGGBB notation), a CSS color name, or the CSS rgb() function. The markup is pretty simple.

```HTML5
<meta name="msapplication-TileColor" content="#D83434">
<meta name="msapplication-TileImage" content="path/to/tileicon.png">
```

![](/images/categories/web/014/tile-icon-pinnning.png)

Okay, so let’s put it all together, accepting the potential limitation of IE10, and keeping the sane parts of everything else.

```HTML5
<link rel="apple-touch-icon" href="path/to/touchicon.png">
<link rel="icon" href="path/to/favicon.png">
<!--[if IE]><link rel="shortcut icon" href="path/to/favicon.ico"><![endif]-->
<!-- or, set /favicon.ico for IE10 win -->
<meta name="msapplication-TileColor" content="#D83434">
<meta name="msapplication-TileImage" content="path/to/tileicon.png">
```

It’s a start, at least.
If you want to learn more about creating favicons, I recommend [Create the perfect favicon](http://www.creativebloq.com/net-magazine) from Jon Hicks’ [The Icon Handbook](https://www.fivesimplesteps.com/products/the-icon-handbook), and [Making a good favicon](https://snook.ca/archives/design/making_a_good_favicon) by Jonathan Snook. Also, I want to thank @alrra for telling me about tile icons.

Wait, I came here for touch icons.

If you want to learn more about embedding touch icons, read Mathias Bynens’ [Everything You Always Wanted To Know About Touch Icons](). Or, follow this summary of his article:


>It’s perfectly possible to just create one high-resolution icon. Lower display resolutions automatically resize the icon. The downside is, this affects performance negatively whenever the site is added to the home screen.

>As of March 2013, if you’re lazy and don’t really care about performance when the site is added to the home screen, just use a single 152×152 icon

And here’s the markup for that icon:

```HTML5
<link rel="apple-touch-icon-precomposed" href="apple-touch-icon-152x152-precomposed.png">
```

May all your BlackBerry – Android – iOS(7) dreams come true.



Origin Post :[http://www.jonathantneal.com/blog/understand-the-favicon/](http://www.jonathantneal.com/blog/understand-the-favicon/)
