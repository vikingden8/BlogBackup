---
title: VCS（版本控制系统）
date: 2017-01-07 17:53:18
tags:
categories: "版本控制系统"
---

### 版本控制系统（VCS）的概念

VCS的英文翻译是：Version Control System

版本控制系统是一种记录一个或若干文件的内容的变化，以便将来查阅特定版本修订情况的系统。

版本控制系统主要有以下用途：

  * 记录文件的所有文件变化；

  * 随时可恢复到任何一个历史状态；

  * 多人协作开发或修改；

  * 错误恢复；

  * 多功能并行开发。


### 版本控制系统的分类

现在的版本控制系统主要以下三类：

  * 本地版本控制系统（Local VCS）；

  * 集中化版本控制系统（Centralized VCS）；

  * 分布式版本控制系统（Distributed VCS）

<!--more-->

#### 本地版本控制系统（Local VCS）

![](/images/categories/vcs/lvcs.png)

比如RCS（Revision Control System）-修订版本控制系统

优点：

  * 简单，很多系统中内置 ；

  * 适合管理文本文件

缺点：

  * 只适合管理少量文件，不支持基于项目的管理；

  * 支持的文件类型单一；

  * 不支持网络，无法实现多人协作


#### 集中化版本控制系统（Centralized VCS）

![](/images/categories/vcs/cvcs.png)

比较常见的有CVS（Concurrent Version System），SVN

优点：

  * 适合多人团队协作开发；

  * 代码集中化管理

缺点：

  * 单点故障；

  * 必须联网工作，无法单击本地工作

#### 分布式版本控制系统（Distributed VCS）

![](/images/categories/vcs/dvcs.png)

常见的有Git，Mercurial

优点：

  * 适合多人团队协作开发；

  * 代码集中化管理；

  * 可以离线工作；

  * 每个计算机都是完整的仓库

#### 参考资料

[Version control](https://en.wikipedia.org/wiki/Version_control)

[Revision Control System](https://en.wikipedia.org/wiki/Revision_Control_System)

[Concurrent Versions System](https://en.wikipedia.org/wiki/Concurrent_Versions_System)

[Distributed version control](https://en.wikipedia.org/wiki/Distributed_version_control)

[What is the Difference Between Mercurial and Git?](http://stackoverflow.com/questions/35837/what-is-the-difference-between-mercurial-and-git)

[git和hg相互比较，各自的优缺点都是什么？](https://www.zhihu.com/question/21905835)

[Git和Mercurial(Hg)的分析](http://article.yeeyan.org/view/243154/211055)
