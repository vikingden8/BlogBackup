---
title: URL详解
date: 2017-01-14 16:02:15
tags:
categories: "Web学习笔记"
---

### 端口

何为端口？端口(Port)，相当于一种数据的传输通道。用于接受某些数据，然后传输给相应的服务，而电脑将这些数据处理后，再将相应的回复通过开启的端口传给对方。

端口的作用：因为 IP 地址与网络服务的关系是一对多的关系。所以实际上因特网上是通过 IP 地址加上端口号来区分不同的服务的。

端口是通过端口号来标记的，端口号只有整数，范围是从0 到65535。

### URL 标准格式

通常而言，我们所熟悉的 URL 的常见定义格式为：

**&lt;scheme&gt;://&lt;user&gt;:&lt;password&gt;@&lt;host&gt;:&lt;port&gt;/&lt;path&gt;;&lt;url-params&gt;?&lt;query-string&gt;#&lt;frag&gt;**


  * **scheme** 访问服务器以获取资源时要使用哪种协议，比如http、https、ftp、rtsp以及著名的ed2k，迅雷的thunder等；

  * **user** 某些方案访问资源时需要的用户名，默认值是匿名；

  * **password** 用户名后面可能要包含的密码，中间由冒号（：）分隔

  * **host**   资源宿主服务器的主机名或者IP地址；

  * **port**  资源宿主服务器正在监听的端口号，很有方案都有默认的端口号（HTTP的默认端口号为80）；

  * **path**   服务器上资源的本地名；

  * **url-params**  某些方案用这个来指定输入参数；

  * **query-string**    发送给服务器的数据；

  * **frag** 一小片或者一部分资源的名字，比如html文档中的锚点。

<!--more-->

###  URL 编码

为什么要进行URL编码？通常如果一样东西需要编码，说明这样东西并不适合直接进行传输。

* **会引起歧义**：例如 URL 参数字符串中使用 key=value 这样的键值对形式来传参，键值对之间以 & 符号分隔，如 ?postid=5038412&t=1450591802326，服务器会根据参数串的 & 和 = 对参数进行解析，如果 value 字符串中包含了 = 或者 & ，如宝洁公司的简称为P&G，假设需要当做参数去传递，那么可能URL所带参数可能会是这样 ?name=P&G&t=1450591802326，因为参数中多了一个&势必会造成接收 URL 的服务器解析错误，因此必须将引起歧义的 & 和 = 符号进行转义， 也就是对其进行编码。

* **非法字符**：又如，URL 的编码格式采用的是 ASCII 码，而不是 Unicode，这也就是说你不能在 URL 中包含任何非 ASCII 字符，例如中文。否则如果客户端浏览器和服务端浏览器支持的字符集不同的情况下，中文可能会造成问题。