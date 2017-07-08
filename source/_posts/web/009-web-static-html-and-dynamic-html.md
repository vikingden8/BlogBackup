---
title: Web资源介绍、静态资源和动态资源的区别、Web服务器种类汇总
date: 2017-07-08 09:24:40
tags:
categories: "Web学习笔记"
---

### Web资源介绍

* html：静态资源 ,浏览器可以看得懂，它可以有变量；

* JSP/Servlet：动态资源,需要先转换成html,再给浏览器看。

当然，除了JavaWeb程序，还有其他Web程序，例如：ASP、PHP等。

<!--more-->

### 静态资源和动态资源区别

![](/images/categories/web/009/01.jpg)

### Web服务器

Web服务器的作用是接收客户端的请求，给客户端作出响应。

对于JavaWeb程序而已，还需要有JSP/Servlet容器，JSP/Servlet容器的基本功能是把动态资源转换成静态资源.

我们需要使用的是Web服务器和JSP/Servlet容器，通常这两者会集于一身。下面是对JavaWeb服务器：

* Tomcat（Apache）：当前应用最广的JavaWeb服务器(注意只支持一部分JavaEE的规范)；

* JBoss（Redhat红帽）：支持JavaEE，应用比较广；EJB容器

* GlassFish（Orcale）：Oracle开发JavaWeb服务器，应用不是很广；

* Resin（Caucho）：支持JavaEE，应用越来越广；

* Weblogic（Orcale）：要钱的！支持JavaEE，适合大型项目；

* Websphere（IBM）：要钱的！支持JavaEE，适合大型项目；
