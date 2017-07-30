---
title: Difference Between Inline, External and Internal CSS Styles
date: 2017-07-30 22:15:57
tags:
categories: "Web学习笔记"
---

### Introduction

There are 3 ways to add CSS styles to your website: you can use internal CSS and include CSS rules in **<head>** section of HTML document, link to an external .css file which contains all CSS rules or use inline CSS to apply rules for specific elements. This tutorial reviews all three ways, their advantages and disadvantages.

### Option 1 – Internal CSS

Internal CSS code is put in the **<head>** section of a particular page. The classes and IDs can be used to refer to the CSS code, but they are only active on that particular page. CSS styles embedded this way are downloaded each time the page loads so it may increase loading speed. However, there are some cases when using internal stylesheet is useful. One example would be sending someone a page template – as everything is in one page, it is a lot easier to see a preview. Internal CSS is put in between **<style></style>** tags. An example of internal stylesheet:

```HTML
<head>
  <style type="text/css">
    p {color:white; font-size: 10px;}
    .center {display: block; margin: 0 auto;}
    #button-go, #button-back {border: solid 1px black;}
  </style>
</head>
```

<!--more-->

* **Advantages of Internal CSS**:

* Only one page is affected by stylesheet.
* Classes and IDs can be used by internal stylesheet.
* There is no need to upload multiple files. HTML and CSS can be in the same file.

* **Disadvantages of Internal CSS**:

* Increased page loading time.
* It affects only one page – not useful if you want to use the same CSS on multiple documents.

### How to add Internal CSS to HTML page

* Open your HTML page with any text editor. If the page is already uploaded to your hosting account, you can use a text editor provided by your hosting. If you have an HTML document on your computer, you can use any text editor to edit it and then re-upload the file to your hosting account using FTP client.

* Locate **<head>** opening tag and add the following code just after it:
  ```xml
  <style type="text/css">
  ```

* Now jump to a new line and add CSS rules, for example:
 ```html
  body {
      background-color: blue;
  }
  h1 {
      color: red;
      padding: 60px;
  }
  ```

* Once you are done adding CSS rules, add the closing style tag:

At the end, HTML document with internal stylesheet should look like this:

```HTML
<!DOCTYPE html>
<html>
<head>
  <style>
    body {
        background-color: blue;
    }
    h1 {
        color: red;
        padding: 60px;
    }
  </style>
</head>
<body>

  <h1>Hostinger Tutorials</h1>
  <p>This is our paragraph.</p>

</body>
</html>
```

### Option 2 – External CSS

Probably the most convenient way to add CSS to your website, is to link it to an external .css file. That way any changes you made to an external CSS file will be reflected on your website globally. A reference to an external CSS file is put in the **<head>** section of the page:

```html
<head>
  <link rel="stylesheet" type="text/css" href="style.css" />
</head>
```

while the style.css contains all the style rules. For example:

```css
.xleftcol {
   float: left;
   width: 33%;
   background:#809900;
}
.xmiddlecol {
   float: left;
   width: 34%;
   background:#eff2df;
}
```

* **Advantages of External CSS**:

* Smaller size of HTML pages and cleaner structure.
* Faster loading speed.
* Same .css file can be used on multiple pages.

* **Disadvantages of External CSS**:

* Until external CSS is loaded, the page may not be rendered correctly.

### Option 3 – Inline CSS

Inline CSS is used for a specific HTML tag. **<style>** attribute is used to style a particular HTML tag. Using CSS this way is not recommended, as each HTML tag needs to be styled individually. Managing your website may become too hard if you only use inline CSS. However, it can be useful in some situations. For example, in cases when you don’t have an access to CSS files or need to apply style for a single element only. An example of HTML page with inline CSS would look like this:

```HTML
<!DOCTYPE html>
<html>
<body style="background-color:black;">

<h1 style="color:white;padding:30px;">Hostinger Tutorials</h1>
<p style="color:white;">Something usefull here.</p>

</body>
</html>
```

* **Advantages of Inline CSS**:

* Useful if you want to test and preview changes.
* Useful for quick-fixes.
* Lower HTTP requests.

* **Disadvantages of Inline CSS**:

* Inline CSS must be applied to every element.

From:[Difference Between Inline, External and Internal CSS Styles](https://www.hostinger.com/tutorials/difference-between-inline-external-and-internal-css)
