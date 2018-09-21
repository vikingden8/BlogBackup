---
title: Where to include your <script> tags
date: 2018-09-22 02:08:45
categories: "JavaScript"
---

Source: [Where should I put <script> tags in HTML markup?](http://stackoverflow.com/a/24070373)

Here's what happens when a browser loads a website with a <script> tag on it:

下面是当一个网页中包含有 <script> 标签时在加载的时候到底做了什么工作。

* 1.Fetch the HTML page (e.g. index.html) 获取HTML页面内容

* 2.Begin parsing the HTML 开始解析HTML内容

* 3.The parser encounters a <script> tag referencing an external script file. 解析器碰到一个引用了外部资源文件的 <script> 标签

* 4.The browser requests the script file. Meanwhile, the parser blocks and stops parsing the other HTML on your page. 浏览器请求脚本文件内容，并在此时，解析器阻塞并停止解析HTML页面的其他内容

* 5.After some time the script is downloaded and subsequently executed. 当在一定的时间后，下载完成并一次被执行

* 6.The parser continues parsing the rest of the HTML document. 然后解析器再解析HTML余下为解析的内容

Step #4 causes a bad user experience. Your website basically stops loading until you've downloaded all scripts. If there's one thing that users hate it's waiting for a website to load.

步骤4会引起坏的用户体验。你的网页会停止加载，直到你所有的脚本下载完成。人们最讨厌的事情就是等待网页加载。
<!--more-->

### Why does this even happen?

Any script can insert its own HTML via document.write() or other DOM manipulations. This implies that the parser has to wait until the script has been downloaded & executed before it can safely parse the rest of the document. After all, the script could have inserted its own HTML in the document.

任何的脚本都可以通过 document.write() 和 其他DOM操作来插入HTML内容。这也意味着解释器必须要等待脚本文件下载以及执行完后才能正确安全地解析余下的文档内容。毕竟，脚本是可以在文档内容中插入的。

However, most JavaScript developers no longer manipulate the DOM while the document is loading. Instead, they wait until the document has been loaded before modifying it. For example:

然而，大部分的JavaScript开发者不会在文档加载时操作DOM。相对地，他们会等到所有的文档已经被加载后才会去修改文档的内容。看下下面的例子：

```JavaScript
<!-- index.html -->
<html>
    <head>
        <title>My Page</title>
        <script type="text/javascript" src="my-script.js"></script>
    </head>
    <body>
        <div id="user-greeting">Welcome back, user</div>
    </body>
</html>
```

JavaScript:

```JavaScript
// my-script.js
document.addEventListener("DOMContentLoaded", function() { 
    // this function runs when the DOM is ready, i.e. when the document has been parsed
    document.getElementById("user-greeting").textContent = "Welcome back, Bart";
});
```

Because your browser does not know my-script.js isn't going to modify the document until it has been downloaded & executed, the parser stops parsing.

因为你的浏览器并不知道 my-script.js 会去修改文档的内容，直到它被下载及被执行后，

### Antiquated recommendation

The old approach to solving this problem was to put <script> tags at the bottom of your <body>, because this ensures the parser isn't blocked until the very end.

以前解决这个问题的方式就是把 <script> 标签放到 <body> 标签的最底部，因为这保证了解释器只有在解析到最后才被阻塞。

This approach has its own problem: the browser cannot start downloading the scripts until the entire document is parsed. For larger websites with large scripts & stylesheets, being able to download the script as soon as possible is very important for performance. If your website doesn't load within 2 seconds, people will go to another website.

这种方式有个问题：浏览器必须在整个文档被解析之后采取下载执行所有的脚本文件。但是，对于大的网站有很大的脚本文件或CSS，尽可能早去下载那些文件对性能十分重要。如果你的网站不能在2秒内加载完毕，那么人们可能会放弃使用。

In an optimal solution, the browser would start downloading your scripts as soon as possible, while at the same time parsing the rest of your document.

有一个优化的方案，就是浏览器在解析余下的文档内容时，同时尽可能早的下载脚本文件。

#### The modern approach

Today, browsers support the **async** and **defer** attributes on scripts. These attributes tell the browser it's safe to continue parsing while the scripts are being downloaded.

现在，浏览器支持脚本中设置 **async** 和 **defer** 属性。那些属性会告诉浏览器脚本文件在下载的过程中去解析余下的文档内容是很安全的。

**async**

```JavaScript
<script type="text/javascript" src="path/to/script1.js" async></script>
<script type="text/javascript" src="path/to/script2.js" async></script>
```

Scripts with the **async** attribute are executed asynchronously. This means the script is executed as soon as it's downloaded, without blocking the browser in the meantime.

带有 **async** 属性的脚本文件会被浏览器异步执行。这也意味着脚本文件在下载完成后立即被执行，同时也不会阻塞浏览器执行其他。


This implies that it's possible to script 2 is downloaded & executed before script 1.

这也暗示着，脚本文件2可能比脚本文件1先下载完并被执行。

According to http://caniuse.com/#feat=script-async, 94.57% of all browsers support this.

**defer**

```JavaScript
<script type="text/javascript" src="path/to/script1.js" defer></script>
<script type="text/javascript" src="path/to/script2.js" defer></script>
```

Scripts with the **defer** attribute are executed in order (i.e. first script 1, then script 2). This also does not block the browser.

带有 **defer** 属性的脚本文件将会按照顺序依次执行。同时，这也不会阻塞浏览器。

Unlike **async** scripts, **defer** scripts are only executed after the entire document has been loaded.

与 **async** 属性的脚本文件不一样的是，带 **defer** 属性的脚本文件只会在整个文档加载后才会被执行。

According to http://caniuse.com/#feat=script-defer, 94.59% of all browsers support this. 94.92% support it at least partially.

An important note on browser compatibility: in some circumstances IE <= 9 may execute deferred scripts out of order. If you need to support those browsers, please read this first!

重要的浏览器兼容性注意点：在某些情况下 IE <= 9 ，带 **defer** 属性的脚本文件可能不会按照顺序执行。


### Conclusion

The current state-of-the-art is to put scripts in the <head> tag and use the **async** or **defer** attributes. This allows your scripts to be downloaded asap without blocking your browser.

现有的流行做法就是把脚本文件放到 <head> 中，并适当的加以 **async** 或 **defer** 属性。这将让你的脚本文件尽可能快的被加载执行，同时也不会阻塞浏览器。

The good thing is that your website should still load correctly on the 6% of browsers that do not support these attributes while speeding up the other 94%.

