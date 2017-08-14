---
title: CSS padding tutorial
date: 2017-08-14 20:07:54
tags:
categories: "Web学习笔记"
---

The padding CSS property sets the padding area on all four sides of an element. It is a shorthand that sets all individual paddings at once: padding-top, padding-right, padding-bottom, and padding-left.

```shell
/* Apply to all four sides */
padding: 1em;

/* vertical | horizontal */
padding: 5% 10%;

/* top | horizontal | bottom */
padding: 1em 2em 2em;

/* top | right | bottom | left */
padding: 2px 1em 0 1em;

/* Global values */
padding: inherit;
padding: initial;
padding: unset;
```

<!--more--->

more examples:

```CSS
padding: 5%;                /* all sides: 5% padding */

padding: 10px;              /* all sides: 10px padding */

padding: 10px 20px;         /* top and bottom: 10px padding */
                            /* left and right: 20px padding */

padding: 10px 3% 20px;      /* top:            10px padding */
                            /* left and right: 3% padding   */
                            /* bottom:         20px padding */

padding: 1em 3px 30px 5px;  /* top:    1em padding  */
                            /* right:  3px padding  */
                            /* bottom: 30px padding */
                            /* left:   5px padding  */
```
