---
title: Software Engineering at Google--Fergus Henderson
date: 2017-2-16 09:39:02
tags:
categories: "Developer Blogs"
---

![](/images/categories/info/gbp.gif)

### Abstract 摘要

We catalog and describe Google’s key software engineering practices.

我们梳理并总结了Google的关键软件工程实践。

### Biography 个人简介

Fergus Henderson has been a software engineer at Google for over 10 years. He started
programming as a kid in 1979, and went on to academic research in programming language
design and implementation. With his PhD supervisor, he co-founded a research group at the
University of Melbourne that developed the programming language Mercury. He has been a
program committee member for eight international conferences, and has released over 500,000
lines of open-source code. He was a former moderator of the Usenet newsgroup comp.std.c++
and was an officially accredited “Technical Expert” to the ISO C and C++ committees. He has over
15 years of commercial software industry experience. At Google, he was one of the original
developers of Blaze, a build tool now used across Google, and worked on the server-side
software behind speech recognition and voice actions (before Siri!) and speech synthesis. He
currently manages Google's text-to-speech engineering team, but still writes and reviews plenty
of code. Software that he has written is installed on over a billion devices, and gets used over a
billion times per day.

Fergus Henderson 作为软件工程师已在Google工作已经10年了。1979年，当他还是一个孩子时就开始编程，后来继续在编程语言设计和实现上进行学术研究。他与他的博 士导师在墨尔本大学共同创立了一个研究小组，开发了编程语言Mercury（水星）。他是8个国际会议的计划委员会成员，并发布了超过50万行的开源代 码。他是Usenet新闻组comp.std.c ++的前主席，并且是ISO C和C ++委员会的官方认可的“技术专家”。他拥有超过15年的商业软件行业经验。在Google，他是Blaze的原创开发人员之一，Blaze是一个目前在 Google上使用的构建工具，并致力于语音识别和语音操作（在Siri！之前）的服务器端软件和语音合成开发工作。他目前管理Google的文字转语音 工程小组，但仍然撰写和审查了大量的代码。他写的软件安装在十亿多个设备上，每天使用量超过十亿次。

### 1. Introduction

**Google has been a phenomenally successful company​**. As well as the success of Google
Search and AdWords, Google has delivered many other stand-out products, including Google
Maps, Google News, Google Translate, Google speech recognition, Chrome, and Android.
Google has also greatly enhanced and scaled many products that were acquired by purchasing
smaller companies, such as YouTube, and has made significant contributions to a wide variety
of open-source projects. And Google has demonstrated some amazing products that are yet to
launch, such as self-driving cars.

**Google已经是一个非常成功的公司了**。 除了Google搜索和AdWords的成功之外，Google还提供了许多其它杰出的产品，包括Google地图、Google新闻、Google翻 译、Google语音识别、Chrome和Android 等。Google通过收购小型公司（如YouTube）获得的产品大幅增强和扩展了大量产品，并对各种开源项目做出了重大贡献。 Google已经展示了一些尚未推出的激动人心的产品，例如自驾车。

There are many reasons for Google’s success, including enlightened leadership, great people, a
high hiring bar, and the financial strength that comes from successfully taking advantage of an
early lead in a very rapidly growing market. But one of these reasons is that **Google has
developed excellent software engineering practices​**, which have helped it to succeed. These
practices have evolved over time based on the accumulated and distilled wisdom of many of the
most talented software engineers on the planet. We would like to share knowledge of our
practices with the world, and to share some of the lessons that we have learned from our
mistakes along the way.

Google的成功有着很多原因，包括开明的领导力、伟大的人才、超高的招聘标准，以及在迅速成长的市场中成功利用早期领先优势的财务实力。但其中一个原因是 **Google开发了优秀的软件工程实践**，帮助它取得成功。这些做法随着时间的流逝，根据地球上众多最有才华的软件工程师积累和凝练的智慧而演变。我们愿与世界分享我们的实践的经验，包括分享我们从错误中学到的一些教训。

**The aim of this paper is to catalogue and briefly describe Google’s key software
engineering practices**. Other organizations and individuals can then compare and contrast
these with their own software engineering practices, and consider whether to apply some of
these practices themselves.

**本文的目的是梳理并简要介绍Google的关键软件工程实践**。然后，其他组织和个人可以将它们与自己的软件工程实践进行比较和对比，并考虑是否自己应用一些实践经验。

Many authors (e.g. [9], [10], [11]) have written books or articles analyzing Google’s success and
history. But most of those have dealt mainly with business, management, and culture; only a
fraction of those (e.g. [1, 2, 3, 4, 5, 6, 7, 13, 14, 16, 21]) have explored the software engineering
side of things, and most explore only a single aspect; and none of them provide a brief written
overview of software engineering practices at Google as a whole, as this paper aims to do.

许多作者（例如[9]，[10]，[11]）都有分析Google的成功和历史的书籍或文章。但大多数主要涉及商业、管理和文化；只有一小部分作者（例如 [1,2,3,4,5,6,7,13,14,16,21]）探索了软件工程方面的事情，并且大多数只探索一个方面；而且没有一篇文章提供了Google整 体软件工程实践的简要概述，正如本文旨在做的那样。

<!--more-->

### 2. Software development 软件开发

####2.1. The Source Repository 源代码仓库

**Most of Google’s code is stored in a single unified source-code repository, and is
accessible to all software engineers at Google​**. There are some notable exceptions,
particularly the two large open-source projects Chrome and Android, which use separate
open-source repositories, and some high-value or security-critical pieces of code for which read
access is locked down more tightly. But most Google projects share the same repository. As of
January 2015, this 86 terabyte repository contained a billion files, including over 9 million source code files containing a total of 2 billion lines of source code​, with a history of 35 million
commits and a change rate of 40 thousand commits per work day [18]. Write access to the
repository is controlled: only the listed owners of each subtree of the repository can approve
changes to that subtree. But generally any engineer can access any piece of code, can check it
out and build it, can make local modifications, can test them, and can send changes for review
by the code owners, and if an owner approves, can check in (commit) those changes.
Culturally, engineers are encouraged to fix anything that they see is broken and know how to fix,
regardless of project boundaries. This empowers engineers and leads to higher-quality
infrastructure that better meets the needs of those using it.

**Google的大多数代码存储在一个统一的源代码存储库中，并且可供 Google 所有软件工程师访问**。请注意有些例外，特别是两个大型开源项目 Chrome 和Android，它们使用单独的开源仓库，另外一些高价值或安全关键的代码，其读取访问被锁定得更紧，管得更严。但大多数 Google 项目共享相同的存储库。截至2015年1月，这个86 TB的存储库包含了10亿个文件，包括900多万个源代码文件，包含总共20亿行源代码，历史记录3500万次提交，每个工作日更改4万次提交[18]。 控制对存储库的写访问：只有存储库的每个子树的列出的所有者都可以批准对该子树的更改。但一般来说，任何工程师都可以访问任何代码片段，可以检出并构建它，可以进行本地修改，可以测试它们，并可以发送更改供代码所有者审核，如果所有者批准，可以签入（提交）那些变化。在企业文化上，Google鼓励工程 师修复他们看到的任何东西，并且知道如何修复它们，不管项目的边界如何。这增强了工程师能力，带来了更高质量的基础设施，更好地满足那些使用它的人们的需 求。

**Almost all development occurs at the “head” of the repository​, not on branches**. This helps
identify integration problems early and minimizes the amount of merging work needed. It also
makes it much easier and faster to push out security fixes.

**几乎所有的开发都发生在仓库的“头”，而不是在分支上**。这有助于及早识别集成问题，并最大限度地减少所需的合并工作量。它还使得更容易和更快地推出安全修复程序。

**Automated systems run tests frequently**, often after every change to any file in the transitive
dependencies of the test, although this is not always feasible. These systems automatically
notify the author and reviewers of any change for which the tests failed, typically within a few
minutes. Most teams make the current status of their build very conspicuous by installing
prominent displays or even sculptures with color-coded lights (green for building successfully
and all tests passing, red for some tests failing, black for broken build). This helps to focus
engineers’ attention on keeping the build green. Most larger teams also have a “build cop” who
is responsible for ensuring that the tests continue to pass at head, by working with the authors
of the offending changes to quickly fix any problems or to roll back the offending change. (The
build cop role is typically rotated among the team or among its more experienced members.)
This focus on keeping the build green makes development at head practical, even for very large
teams.

尽 管这并不总是可行，在测试的传递依赖性中的任何文件每次更改之后，**自动化系统总是频繁运行测试**。通常在几分钟内，这些系统自动通知作者和审阅者任何导致测 试失败的更改。大多数团队通过安装突出的显示屏，甚至安装带有颜色编码的灯（绿色为成功构建和所有测试通过，红色为一些测试失败，黑色为构建失败）使他们 构建的当前状态非常显眼。这有助于将工程师的注意力集中在保持构建为绿色状态。大多数更大的团队还有一个“构建警察”，通过与错误的更改的作者合作，以快 速解决任何问题或回滚令人不安的更改，负责确保持续通过测试。（构建警察角色通常在团队成员之间或在其经验丰富的成员之间轮流充当）。这种关注保持构建绿 色状态，使开发变得实用，即使对于非常大的团队仍然如此。

**Code ownership**. Each subtree of the repository can have a file listing the user ids of the
“owners” of that subtree. Subdirectories also inherit owners from their parent directories,
although that can be optionally suppressed. The owners of each subtree control write access to
that subtree, as described in the code review section below. Each subtree is required to have at
least two owners, although typically there are more, especially in geographically distributed
teams. It is common for the whole team to be listed in the owners file. Changes to a subtree
can be made by anyone at Google, not just the owners, but must be approved by an owner.
This ensures that every change is reviewed by an engineer who understands the software being
modified.

**代码所有权**。 存储库的每个子树可以具有列出该子树的“所有者”的用户ID的文件。子目录还从父目录继承所有者，但可以选择禁止。每个子树的所有者控制对该子树具有写访 问权，如下面的代码审查部分所述。每个子树需要至少有两个所有者，虽然通常有更多的所有者，特别是在地理上分布的团队。通常将整个团队成员列在所有者文件 中。 Google的任何人都可以对子树进行更改，而不仅仅是所有者，但必须获得所有者的批准。这确保每个更改由理解所修改软件情况的工程师审核。

For more on the source code repository at Google, see [17, 18, 21]; and for how another large
company deals with the same challenge, see [19].

有关Google源代码存储库的更多信息，请参阅[17,18,21]；以及另一家大公司如何应对同样的挑战，见[19]。

#### 2.2 The Build System 构建系统

Google uses a distributed build system known as Blaze, which is responsible for compiling and
linking software and for running tests. It provides standard commands for building and testing
software that work across the whole repository. These standard commands and the highly
optimized implementation mean that **it is typically very simple and quick for any Google
engineer to build and test any software in the repository**​. This consistency is a key enabler
which helps to make it practical for engineers to make changes across project boundaries.

Google使用一个称为Blaze的分布式构建系统，该平台负责编译和链接软件以及运行测试。它提供了用于构建和测试在整个存储库中工作的软件的标准命令。这些标准命令和高度优化的实现意味着，**对于任何Google工程师来说，构建和测试存储库中的任何软件通常非常简单和快捷**。这种一致性是一个关键的推动因素，有助于使工程师能够跨项目边界进行更改。

Programmers write “BUILD” files that Blaze uses to determine how to build their software.
Build entities such as libraries, programs, and tests are declared using fairly high-level
**declarative build specifications** that specify, for each entity, its name, its source files, and the
libraries or other build entities that it depends on. These build specifications are comprised of
declarations called “build rules” that each specify high-level concepts like “here is a C++ library
with these source files which depends on these other libraries”, and it is up to the build system
to map each build rule to a set of build steps, e.g. steps for compiling each source file and steps
for linking, and for determining which compiler and compilation flags to use.

程序员编写“BUILD”文件，Blaze用它来确定如何构建他们的软件。使用相当高级别的 **声明式构建规范**，程序和测试等构建实体，为每个实体指定其名称，源文件以及它所依赖的库或其他构建实体。这些构建规范包括称为“构建规则”的声明，每个都指定高级概 念，如“这里是一个C ++库，这些源文件取决于这些其他库”，并且由构建系统将每个构建规则映射到一组构建步骤，例如编译每个源文件的步骤和用于链接的步骤，以及用于确定使用 哪个编译器和编译标志。

In some cases, notably Go programs, build files can be generated (and updated) automatically,
since the dependency information in the BUILD files is (often) an abstraction of the dependency
information in the source files. But they are nevertheless checked in to the repository. This
ensures that the build system can quickly determine dependencies by analyzing only the build
files rather than the source files, and it avoids excessive coupling between the build system and
compilers or analysis tools for the many different programming languages supported.

在某些情况 下，特别是Go程序，可以自动生成（和更新）构建文件，因为BUILD文件中的依赖性信息是（通常）源文件中的依赖性信息的抽象。但是，它们仍然检入到存 储库里。这确保构建系统可以通过仅分析构建文件而不是源文件来快速确定依赖性，并且避免构建系统和用于所支持的许多不同编程语言的编译器或分析工具之间的 过度耦合。

The build system’s implementation uses Google’s distributed computing infrastructure. The work
of each build is typically **distributed across hundreds or even thousands of machines**​. This
makes it possible to build extremely large programs quickly or to run thousands of tests in
parallel.

构建系统的实现使用Google的分布式计算基础设施。每个构建的工作通常 **分布在数百甚至数千个机器上**。这使得可以快速构建极大的程序或并行运行数千个测试。

**Individual build steps must be “hermetic”: they depend only on their declared inputs**.
Enforcing that all dependencies be correctly declared is a consequence of distributing the build:
only the declared inputs are sent to the machine on which the build step is run. As a result the
build system can be relied on to know the true dependencies. Even the compilers that the build
system invokes are treated as inputs.

**各个构建步骤必须是“气密的”：它们仅取决于他们所声明的输入**。强制所有依赖关系正确声明是分发构建的结果：只有声明的输入被发送到运行构建步骤的机器。因此，构建系统可以依赖于知道真正的依赖性。即使编译系统调用的编译器也被视为输入。

**Individual build steps are deterministic**. As a consequence, the build system can cache build
results. Software engineers can sync their workspace back to an old change number and can
rebuild and will get exactly the same binary. Furthermore, this cache can be safely shared
between different users. (To make this work properly, we had to eliminate non-determinism in
the tools invoked by the build, for example by scrubbing out timestamps in the generated output
files.)

**单独构建步骤是确定性的**。 因此，构建系统可以缓存构建结果。软件工程师可以将它们的工作空间同步到旧的更改号码，并且可以重新构建，最终将获得完全相同的二进制文件。此外，该高速 缓存可以在不同用户之间安全地共享。（为了正常工作，我们必须消除由构建调用的工具中的不确定性，例如通过清除生成的输出文件中的时间戳）。

**The build system is reliable**. ​The build system tracks dependencies on changes to the build
rules themselves, and knows to rebuild targets if the action to produce them changed, even if
the inputs to that action didn’t, for example when only the compiler options changed. It also
deals properly with interrupting the build part way, or modifying source files during the build: in
such cases, you need only rerun the build command. There is never any need to run the
equivalent of “make clean”.

**构建系统是可靠的**。 构建系统跟踪对构建规则本身的更改的依赖关系，并且如果生成目标的操作发生变化，即使该操作的输入没有改变，也知道重新构建目标，例如当只更改编译器选项 时也是如此。它还正确处理中断构建部分的方式，或在构建期间修改源文件：在这种情况下，您只需要重新运行build命令。没有任何需要运行相当于 “make clean”。

**Build results are cached “in the cloud”**​. This includes intermediate results. If another build
request needs the same results, the build system will automatically reuse them rather than
rebuilding, even if the request comes from a different user.

**构建结果被缓存在“云中”**。这包括中间结果。如果另一个构建请求需要相同的结果，即使请求来自不同的用户，构建系统将自动重用它们而不是重新构建。

**Incremental rebuilds are fast**. ​The build system stays resident in memory so that for rebuilds
it can incrementally analyze just the files that have changed since the last build.

**增量重新构建速度快**。构建系统保持驻留在内存中，因此对于重新构建，它可以增量地分析自上次构建以来更改的文件。

**Presubmit checks**. ​Google has tools for automatically running a suite of tests when initiating a
code review and/or preparing to commit a change to the repository. Each subtree of the
repository can contain a configuration file which determines which tests to run, and whether to
run them at code review time, or immediately before submitting, or both. The tests can be either
synchronous, i.e. run before sending the change for review and/or before committing the
change to the repository (good for fast-running tests); or asynchronous, with the results emailed
to the review discussion thread. [The review thread is the email thread on which the code
review takes place; all the information in that thread is also displayed in the web-based code
review tool.]

**预提交检查**。 Google提供了在启动代码审查和/或准备向存储库提交更改时自动运行一系列测试的工具。存储库的每个子树可以包含配置文件，该配置文件确定要运行哪些 测试，以及是在代码审查时还是在提交之前立即运行它们，还是两种情况下都运行测试。测试可以是同步的，即在发送更改以进行审查之前和/或在将更改提交到存 储库之前运行（有利于快速运行测试）;或者是异步的，结果通过电子邮件发送到审查讨论线程。 [审查线程是进行代码审查的电子邮件线程；该线程中的所有信息也显示在基于Web的代码审查工具中。]

#### 2.3. Code Review 代码审查

**Google has built excellent web-based code review tools**​, integrated with email, that allow
authors to request a review, and allows reviewers to view side-by-side diffs (with nice color
coding) and comment on them. When the author of a change initiates a code review, the
reviewers are notified by e-mail, with a link to the web review tool’s page for that change. Email
notifications are sent when reviewers submit their review comments. In addition, automated
tools can send notifications, containing for example the results of automated tests or the
findings of static analysis tools.

**Google已经建立了完善的基于Web的代码审查工具**， 并与电子邮件集成，允许作者提出审查，并允许审查者查看并排显示差异（使用漂亮的颜色编码）并对其进行评论。当更改作者发起代码审查时，系统会通过电子邮 件通知审查者，并提供指向该网页更改工具网页的链接。当审查人提交审查评论时，系统会发送电子邮件通知。此外，自动化工具可以发送通知，包含例如自动化测 试的结果或静态分析工具的结果。

**All changes to the main source code repository MUST be reviewed by at least one other
engineer**. In addition, if the author of a change is not one of the owners of the files being
modified, then at least one of the owners must review and approve the change.

**对主源代码存储库的所有更改必须至少由另一位工程师审查**。 此外，如果更改的作者不是正在修改的文件的所有者，则至少有一个所有者必须审查并批准该更改请求。

In exceptional cases, an owner of a subtree can check in (commit) an urgent change to that
subtree before it is reviewed, but a reviewer must still be named, and the change author and
reviewer will get automatically nagged about it until the change has been reviewed and
approved. In such cases, any modifications needed to address review comments must be done
in a separate change, since the original change will have already been committed.

在特殊情况下，子树的所有者可以在审查之前签入（提交） 该子树的紧急更改，但审查者仍然必须命名，并且更改作者和审查者将自动对其进行标记，直到更改生效经过审查和批准。在这种情况下，为解决审查意见所需的任 何修改必须单独进行，因为原始更改已经提交。

Google has tools for automatically suggesting reviewer(s) for a given change, by looking at the
ownership and authorship of the code being modified, the history of recent reviewers, and the number of pending code reviews for each potential reviewer. At least one of the owners of each
subtree which a change affects must review and approve that change. But apart from that, the
author is free to choose reviewer(s) as they see fit.

Google 有一些工具可以通过查看正在修改的代码的所有权和作者身份，最近审查人的历史记录以及每个潜在审查者的待审代码数量，自动为特定更改提示审查人。变更所影 响的每个子树至少一个所有者必须审查并批准该更改请求。但除此之外，作者可以自由选择审查者，因为他们认为合适做审查。

One potential issue with code review is that if the reviewers are too slow to respond or are
overly reluctant to approve changes, this could potentially slow down development. The fact
that the code author chooses their reviewers helps avoid such problems, allowing engineers to
avoid reviewers that might be overly possessive about their code, or to send reviews for simple
changes to less thorough reviewers and to send reviews for more complex changes to more
experienced reviewers or to several reviewers.

代 码审查的一个潜在问题是，如果审查者的响应太慢，或者过于不愿意批准更改，这可能会减慢开发速度。代码作者选择他们的审查者的事实有助于避免这样的问题， 允许工程师避免可能对他们的代码过于占有的审查者，或者发送评论以对较不透彻的评论者进行简单的改变，并且将更复杂的改变发送给更有经验的审查者或几个审 查者。

**Code review discussions for each project are automatically copied to a mailing list designated by
the project maintainers**. Anyone is free to comment on any change, regardless of whether they
were named as a reviewer of that change, both before and after the change is committed. If a
bug is discovered, it’s common to track down the change that introduced it and to comment on
the original code review thread to point out the mistake so that the original author and reviewers
are aware of it.

**每个项目的代码审查讨论自动复制到项目维护者指定的邮件列表中**。任何人都可以对任何更改发表评论，无论他们是否被指名为该更改的审查人，无论是在更改之前还是之后。如果发现错误，通常追踪引入它的更改，并对原始代码审查线程发表评论，指出错误，以便原始作者和审查者知道情况。

It is also possible to send code reviews to several reviewers and then to commit the change as
soon as one of them has approved (provided either the author or the first responding reviewer is
an owner, of course), before the other reviewers have commented, with any subsequent review
comments being dealt with in follow-up changes. This can reduce the turnaround time for
reviews.

还可以向几个审查者发送代码审查，然后一旦其中一个审查者批准（当然作者或第一个作出回应的审查者是所有者，则在其他审查者评论之前）提交改变，随后的审查意见将在后续更改中处理。这样可以减少评论的周转时间。

In addition to the main section of the repository, **there is an “experimental” section of the
repository where the normal code review requirements are not enforced​**. However, code
running in production must be in the main section of the repository, and engineers are very
strongly encouraged to develop code in the main section of the repository, rather than
developing in experimental and then moving it to the main section, since code review is most
effective when done as the code is developed rather than afterwards. In practice engineers
often request code reviews even for code in experimental.

除了存储库的主要部分，**存储库中也存在一个“实验性”部分，它不强制执行正常的代码审查要求**。 但是，在生产环境中运行的代码必须位于存储库的主要部分，并且非常强烈地鼓励工程师在存储库的主要部分开发代码，而不是开发实验性代码，然后将其移动到主 要部分，当代码是开发时而不是之后，因为代码审查是最有效的。实际上，工程师经常要求代码审查，即使是实验代码也是如此。

**Engineers are encouraged to keep each individual change small**​, with larger changes
preferably broken into a series of smaller changes that a reviewer can easily review in one go.
This also makes it easier for the author to respond to major changes suggested during the
review of each piece; very large changes are often too rigid and resist reviewer-suggested
changes. One way in which keeping changes small is encouraged is that the code review tools
label each code review with a description of the size of the change, with changes of 30-99 lines
added/deleted/removed being labelled “medium-size” and with changes of above 300 lines
being labelled with increasingly disparaging labels, e.g. “large” (300-999), “freakin huge” (1000-1999), etc. (However, in a typically Googly way, this is kept fun by replacing these
familiar descriptions with amusing alternatives on a few days each year, such as
talk-like-a-pirate day. :)

**Google鼓励工程师坚持每个单独的更改较小的原则**， 较大的更改最好分成一系列较小的更改，审查者可以轻松地一次查看完毕。这也使作者对每篇作品的审查期间提出的主要变化作出反应更容易；非常大的更改通常太 僵硬，并阻碍审查者对更改提出建议。鼓励保持更改较小原则的一种方式是代码审查工具标记每个代码审查与更改的大小描述，添加/删除30-99行的更改标记 为“中型”和300行以上的更改标记具有越来越不相关的标签，例如““large”（300-999），“freakinhuge” （1000-1999）等（但是，以一个典型的Google式的自娱自乐的方式，每年花上几天用有趣的替代品替换这些熟悉的描述，正如像海盗行侠仗义似谈 资）。

#### 2.4. Testing 测试

**Unit Testing is strongly encouraged and widely practiced at Google​**. All code used in
production is expected to have unit tests, and the code review tool will highlight if source files
are added without corresponding tests. Code reviewers usually require that any change which
adds new functionality should also add new tests to cover the new functionality. Mocking
frameworks (which allow construction of lightweight unit tests even for code with dependencies
on heavyweight libraries) are quite popular.

**Google强烈鼓励并广泛使用单元测试**。 生产环境中使用的所有代码都需要进行单元测试，代码审查工具将突出显示是否添加了源文件而没有添加相应的测试。代码审查人员通常要求添加新功能的任何更改 都应添加新测试以涵盖新功能。模拟框架（允许构建轻量级单元测试，甚至对于具有对重量级库的依赖的代码）的使用时相当普遍的。

Integration testing and regression testing are also widely practiced.

集成测试和回归测试也广泛应用。

As discussed in "**Presubmit Checks**" above, testing can be automatically enforced as part of the
code review and commit process.

如上面的“**预提交检查**”中所述，测试可以作为的一部分自动执行代码审查和提交过程。

Google also has automated tools for measuring test coverage. The results are also integrated
as an optional layer in the source code browser.

Google还提供用于测量测试覆盖率的自动化工具。结果也作为源代码浏览器中的可选层被整合进来。

Load testing prior to deployment is also de rigueur at Google. Teams are expected to
produce a table or graph showing how key metrics, particularly latency and error rate, vary with
the rate of incoming requests.

在部署之前进行负载测试也是Google的重点。团队需要生成一个表格或图形，显示关键指标（特别是延迟和错误率）如何随传入请求的速率而变化。

#### 2.5. Bug tracking Bug 跟踪

Google uses a bug tracking system called Buganizer for tracking issues: bugs, feature requests,
customer issues, and processes (such as releases or clean-up efforts). Bugs are categorized
into hierarchical components and each component can have a default assignee and default
email list to CC. When sending a source change for review, engineers are prompted to
associate the change with a particular issue number.

Google 使用一个名为Buganizer的错误跟踪系统来跟踪问题：bug，功能请求，客户问题和流程（如发布或清理工作）。错误被分为层次化组件，每个组件可以 有一个默认的受托人和默认电子邮件列表到抄送列表（CC）。当发送源代码更改以供审查时，系统会提示工程师将更改与特定的问题编号相关联。

It is common (though not universal) for teams at Google to regularly scan through open issues
in their component(s), prioritizing them and where appropriate assigning them to particular
engineers. Some teams have a particular individual responsible for bug triage, others do bug
triage in their regular team meetings. Many teams at Google make use of labels on bugs to
indicate whether bugs have been triaged, and which release(s) each bug is targeted to be fixed
in.

Google 的团队通常（但不是通用）定期扫描其组件中的开放问题，确定优先级，并在适当时将其分配给特定工程师。一些团队有一个特定的个人负责Bug分类，其他人在 常规团队会议中进行Bug分类。 Google的许多团队都使用Bug标签来指明是否已对Bug进行了分类，以及每个Bug所针对的发布版本。

#### 2.6. Programming languages 编程语言

Software engineers at Google are strongly encouraged to program in one of four
officially-approved programming languages at Google: **C++, Java, Python, or Go​**. Minimizing
the number of different programming languages used reduces obstacles to code reuse and
programmer collaboration.

Google强烈鼓励软件工程师使用以下四种官方认可的编程语言之一进行编程：**C++，Java，Python或Go**。最小化所使用的不同编程语言的数量扫平了代码重用和程序员协作的障碍。

There are also **Google style guides** for each language, to ensure that code all across the
company is written with similar style, layout, naming conventions, etc. In addition there is a
company-wide readability training process, whereby experienced engineers who care about
code readability train other engineers in how to write readable, idiomatic code in a particular
language, by reviewing a substantial change or series of changes until the reviewer is satisfied
that the author knows how to write readable code in that language. Each change that adds
non-trivial new code in a particular language must be approved by someone who has passed
this “readability” training process in that language.

另外，每种语言都有 **Google代码规范**，以确保整个公司的代码都具有类似的风格、布局、命名约定等。此外，还有一个公司范围的可读性培 训流程，由经验丰富的工程师关心代码可读性通过检查一个实质性的变化或一系列变化，直到审查人确信作者知道如何用该语言编写可读代码为止，来训练其他工程 师如何以特定语言编写可读的惯用代码。每个以特定语言添加不重要的新代码的更改都必须由已通过该语言的“可读性”培训流程的人批准。

In addition to these four languages, many **specialized domain-specific languages** are used
for particular purposes (e.g. the build language used for specifying build targets and their
dependencies).

除了这四种语言之外，许多 **专用的领域特定语言** 用于特定目的（例如用于指定构建目标及其依赖性的构建语言）。

Interoperation between these different programming languages is done mainly using **Protocol
Buffers**​. Protocol Buffers is a way of encoding structured data in an efficient yet extensible
way. It includes a domain-specific language for specifying structured data, together with a
compiler that takes in such descriptions and generates code in C++, Java, Python, for
constructing, accessing, serializing, and deserializing these objects. Google’s version of
Protocol Buffers is integrated with Google’s RPC libraries, enabling simple cross-language
RPCs, with serialization and deserialization of requests and responses handled automatically by
the RPC framework.

这些不同的编程语言之间的互操作主要使用 **Protocol Buffers（协议缓冲区）** 完成。ProtocolBuffers 是一种以高效且可扩展的方式编码结构化数据的方法。它包括一门用于指定结构化数据的领域特定语言以及一个编译器，它接受领域特定语言的描述，并生成C ++，Java，Python中的代码，用于构建、访问、序列化和反序列化这些对象。 Google的Protocol Buffers版本与Google的RPC库集成，支持简单的跨语言RPC，对RPC框架自动处理的请求和响应进行序列化和反序列化。

**Commonality of process** is a key to making development easy even with an enormous code
base and a diversity of languages: there is a single set of commands to perform all the usual
software engineering tasks (such as check out, edit, build, test, review, commit, file bug report,
etc.) and the same commands can be used no matter what project or language. Developers
don’t need to learn a new development process just because the code that they are editing
happens to be part of a different project or written in a different language.

**过程的通用性** 是使开发变得容易的关键，即使具有巨大的代码库和多样化的语言也是如此：无论什么项目或语言，有一组命令来执行所有常见的软件工程任务（例如签出、编辑、构建、测试、审查、提交、文件错误报告等）和相同的命令可以使用。开发人员不需要学习新的开发过程，因为他们正在编辑的代码恰好是不同项目的一部分或用不同的语言编写的。

#### 2.7. Debugging and Profiling tools 调试和剖析工具

Google servers are linked with libraries that provide a number of tools for debugging running
servers. In case of a server crash, a signal handler will automatically dump a stack trace to a
log file, as well as saving the core file. If the crash was due to running out of heap memory, the
server will dump stack traces of the allocation sites of a sampled subset of the live heap objects.
There are also web interfaces for debugging that allow examining incoming and outgoing RPCs
(including timing, error rates, rate limiting, etc.), changing command-line flag values (e.g. to
increase logging verbosity for a particular module), resource consumption, profiling, and more.
These tools greatly increase the overall ease of debugging to the point where it is rare to fire up
a traditional debugger such as gdb.

Google 服务器与库相链接，这些库提供了许多用于调试运行中服务器的工具。在服务器崩溃的情况下，信号处理程序自动将堆栈跟踪转储到日志文件，并保存核心文件。如 果崩溃是由于堆内存不足，服务器将转储活动堆对象的采样子集的分配站点的堆栈跟踪。还有用于调试的网络接口，其允许检查传入和传出RPC（包括定时、错误 率、速率限制等），改变命令行标志值（例如以增加特定模块的日志冗长度）、资源消耗、剖析等等。这些工具大大增加了调试的总体便利性，以至于很少启动传统 的调试器（如gdb）来完成调试工作。

#### 2.8. Release engineering 发布工程

A few teams have dedicated release engineers, but for most teams at Google, the release
engineering work is done by regular software engineers.

在Google，只有几个团队有专门的发布工程师，但对于的大多数团队，发布工程工作都是由一般软件工程师完成的。

**Releases are done frequently** for most software; weekly or fortnightly releases are a common
goal, and some teams even release daily. This is made possible by **automating most of the
normal release engineering tasks​**. Releasing frequently helps to keep engineers motivated
(it’s harder to get excited about something if it won’t be released until many months or even
years into the future) and increases overall velocity by allowing more iterations, and thus more
opportunities for feedback and more chances to respond to feedback, in a given time.

大多数软件 **频繁发布**； 每周或每两周发布是比较常见的目标，一些团队甚至每天发布。这是通过 **自动化大多数正常发布工程任务** 实现的。频繁发布有助于保持工程师的积极性（如果在几个 月甚至几年后才发布，那么就很难激发工程师的积极性），并通过允许更多的迭代增加总体速度，从而获得更多的反馈机会和更多在给定时间内响应反馈的机会。

A release typically starts in a fresh workspace, by syncing to the change number of the latest
“green” build (i.e. the last change for which all the automatic tests passed), and making a
release branch. The release engineer can select additional changes to be “cherry-picked”, i.e.
merged from the main branch onto the release branch. Then the software will be rebuilt from
scratch and the tests are run. If any tests fail, additional changes are made to fix the failures
and those additional changes are cherry-picked onto the release branch, after which the
software will be rebuilt and the tests rerun. When the tests all pass, the built executable(s) and
data file(s) are packaged up. All of these steps are automated so that the release engineer
need only run some simple commands, or even just select some entries on a menu-driven UI,
and choose which changes (if any) to cherry pick.

通常通过同步到最新的“绿色”构建（即，所有自动测试通过的最后一个改变）的更改号并进行发布分支，在新的工作区中开始发布。发布工程师可以选择额外的更改以“cherry-picked”，即从主分支合并到发布分支上。然后软件将从头重新构建并运行测试。如果存在任何测试失败，则进行其他更改以修复故障，并将这些附加更改挑选到发布分支上，之后将重新构建软件并重新运行测试。当测试全部通过时，将构建的可执行文件和数据文件打包。所有这些步骤都是自动化的，所以发布工程师只需要运行一些简单的命令，甚至只需在菜单驱动的UI上选择一些菜单项，选中哪些更改（如果有的话）即可。

Once a candidate build has been packaged up, it is typically loaded onto a “**staging**​” server for
further **integration testing by small set of users​** (sometimes just the development team).

一旦候选构建已经被打包，它通常被加载到“**分段（Staging）**”服务器上，以便由小组用户（有时仅仅是开发团队）进行进一步的集成测试。

A useful technique involves sending a copy of (a subset of) the requests from production traffic
to the staging server, but also sending those same requests to the current production servers
for actual processing. The responses from the staging server are discarded, and the responses
from the live production servers are sent back to the users. This helps ensure that any issues
that might cause serious problems (e.g. server crashes) can be detected before putting the
server into production.

一 种有用的技术涉及将来自生产流量的请求的副本（子集）发送到分段（staging）服务器，但是也将这些相同请求发送到当前生产服务器用于实际处理。来自 分段服务器的响应被丢弃，并且来自实况生产服务器的响应被发送回给用户。这有助于确保在将服务器投入生产之前，可以检测到任何可能导致严重问题（例如服务 器崩溃）的问题。

The next step is to usually roll out to one or more “**canary**​” servers that are **processing a
subset of the live production traffic**. Unlike the “staging” servers, these are processing and
responding to real users.

下一步是通常推出到正在处理实况生产流量子集的一个或多个“**金丝雀（canary）**”服务器。与“分段”服务器不同，这些是处理和响应真实用户。

Finally the release can be rolled out to all servers in all data centers. For very high-traffic,
high-reliability services, this is done with a **gradual roll-out** over a period of a couple of days, to
help reduce the impact of any outages due to newly introduced bugs not caught by any of the
previous steps.

最后，该版本可以推广到所有数据中心中的所有服务器。对于非常高流量，高可靠性的服务，这在几天的时间内 **逐步推出** ，以帮助减少任何中断的影响，因为新引入的bug没有被任何以前的步骤捕获到。

For more information on release engineering at Google, see chapter 8 of the SRE book [7]. See
also [15].

有关Google发布工程的更多信息，请参见SRE手册[7]的第8章。参见[15]。

#### 2.9. Launch approval 启动批准

The launch of any user-visible change or significant design change requires approvals from a
number of people outside of the core engineering team that implements the change. In
particular approvals (often subject to detailed review) are required to ensure that code complies
with legal requirements, privacy requirements, security requirements, reliability requirements
(e.g. having appropriate automatic monitoring to detect server outages and automatically notify
the appropriate engineers), business requirements, and so forth.

启动任何用户可见的更改或重大设计更改需要来自实施更改的核心工程团队之外的许多人员的批准。需要特别的批准（通常需要详细审查），以确保代码符合法律需求、隐私需求、安全需求、可靠性需求（例如，具有适当的自动监控以检测服务器中断并自动通知相应的工程师），业务需求等等。

The launch process is also designed to ensure that appropriate people within the company are
notified whenever any significant new product or feature launches.

启动过程还旨在确保在任何重要的新产品或功能启动时通知公司内的相应人员。

**Google has an internal launch approval tool that is used to track the required reviews
and approvals and ensure compliance with the defined launch processes for each
product**. This tool is easily customizable, so that different products or product areas can have
different sets of required reviews and approvals.

**oogle有一个内部启动批准工具，用于跟踪所需的审查和批准，以确保符合每个产品定义的启动流程**。此工具可轻松自定义，以便不同的产品或产品区域可以具有不同的所需的审查和批准。

For more information about launch processes, see chapter 27 of the SRE book [7].

有关启动过程的更多信息，请参见SRE手册[7]的第27章。

#### 2.10. Post-mortems

Whenever there is a significant outage of any of our production systems, or similar mishap, the
people involved are required to write a post-mortem document. This document describes the
incident, including title, summary, impact, timeline, root cause(s), what worked/what didn’t, and
action items. **The focus is on the problems, and how to avoid them in future, not on the
people or apportioning blame**. ​The impact section tries to quantify the effect of the incident, in
terms of duration of outage, number of lost queries (or failed RPCs, etc.), and revenue. The
timeline section gives a timeline of the events leading up to the outage and the steps taken to
diagnose and rectify it. The what worked/what didn’t section describes the lessons learnt --
which practices helped to quickly detect and resolve the issue, what went wrong, and what
concrete actions (preferably filed as bugs assigned to specific people) can be take to reduce the
likelihood and/or severity of similar problems in future.

每当我们的任何生产系统发生严重停机或类似事故时，所涉及的人员都必须写一份复盘文档。本文档描述了事件，包括标题、摘要、影响、时间表、根本原因、什么正常工作/什么无法工作和行动措施。**重点是问题，以及如何避免它们在未来再次发生，而不是对人或分摊责任**。 影响部分尝试根据中断持续时间，丢失的查询（或失败的RPC等）数量和收入来量化事件的影响。时间线部分给出了导致中断的事件的时间表，以及诊断和纠正它 所采取的步骤。“什么正常工作/什么无法工作”部分描述教训 – 哪些实践帮助迅速发现和解决问题，什么出现错误，采取什么具体行动（最好把错误分配给特定的人）可以采取减少未来类似问题的可能性和/或严重程度。

For more information on post-mortem culture at Google, see chapter 15 of the SRE book [7].

有关Google复盘文化的更多信息，请参阅SRE手册[7]的第15章。

#### 2.11. Frequent rewrites 频繁重写

**Most software at Google gets rewritten every few years.**

**Google大多数软件每隔几年就会重写一次**。

This may seem incredibly costly. Indeed, it does consume a large fraction of Google’s
resources. However, it also has some crucial benefits that are key to Google’s agility and
long-term success. In a period of a few years, it is typical for the requirements for a product to
change significantly, as the software environment and other technology around it change, and
as changes in technology or in the marketplace affect user needs, desires, and expectations.
Software that is a few years old was designed around an older set of requirements and is
typically not designed in a way that is optimal for current requirements. Furthermore, it has
typically accumulated a lot of complexity. Rewriting code cuts away all the unnecessary
accumulated complexity that was addressing requirements which are no longer so important. In
addition, rewriting code is a way of transferring knowledge and a sense of ownership to newer
team members. This sense of ownership is crucial for productivity: engineers naturally put more
effort into developing features and fixing problems in code that they feel is “theirs”. Frequent
rewrites also encourage mobility of engineers between different projects which helps to
encourage cross-pollination of ideas. Frequent rewrites also help to ensure that code is written
using modern technology and methodology.

这看起来似乎非常昂贵。事实上，它消耗了很大一部分Google资源。然而，它也有一些关键的好处，这是Google保持敏捷性和长期成功的关键所在。在几 年的时间里，随着软件环境和其周围的其他技术发生变化，以及随着技术或市场的变化影响用户需求、愿望和期望，产品的需求通常会发生显著变化。几年前的软件 是围绕一组较旧的需求设计的，通常不是以对当前需求最佳的方式设计的。此外，它通常累积了很多复杂性。重写代码减少了所有不必要的累积复杂性，这些复杂性正在解决不再那么重要的要求。此外，重写代码是一种向新的团队成员传递知识和所有权感的方式。这种所有权感对生产力至关重要：工程师自然会更努力地开发特 性，并在他们认为是“他们的”的代码在解决问题。频繁的重写也鼓励工程师在不同项目之间的换岗，有助于鼓励想法的异质授粉。频繁的重写也有助于确保代码使 用现代技术和方法书写。

### 3. Project management 项目管理

#### 3.1. 20% time

**Engineers are permitted to spend up to 20% of their time working on any project of their
choice, without needing approval from their manager or anyone else**. This trust in
engineers is extremely valuable, for several reasons. Firstly, it allows anyone with a good idea,
even if it is an idea that others would not immediately recognize as being worthwhile, to have
sufficient time to develop a prototype, demo, or presentation to show the value of their idea.
Secondly, it provides management with visibility into activity that might otherwise be hidden. In
other companies that don’t have an official policy of allowing 20% time, engineers sometimes
work on “skunkwork” projects without informing management. It’s much better if engineers can
be open about such projects, describing their work on such projects in their regular status
updates, even in cases where their management may not agree on the value of the project.
Having a company-wide official policy and a culture that supports it makes this possible.
Thirdly, by allowing engineers to spend a small portion of their time working on more fun stuff, it
keeps engineers motivated and excited by what they do, and stops them getting burnt out,
which can easily happen if they feel compelled to spend 100% of their time working on more
tedious tasks. The difference in productivity between engaged, motivated engineers and burnt
out engineers is a lot more than 20%. Fourthly, it encourages a culture of innovation. Seeing
other engineers working on fun experimental 20% projects encourages everyone to do the
same.

**Google允许工程师高达20％的时间花在他们选择的任何项目上工作，而无需他们的经理或任何其他人的批准**。 这种对工程师的信任是非常有价值的，有几个原因。首先，它允许任何人有一个好主意（即使这是一个想法，其他人不会立即认出是值得的）有足够的时间来开发原 型，演示或展示示，以证明他们的想法具有价值。其次，它向管理层提供可能隐藏活动的可见性。在没有允许20％ 时间的官方政策其他公司中，工程师有时会在不通知管理层的情况下工作在“偷偷摸摸”的项目。如果工程师可以正大光明地从事这样的项目，描述他们在这些项目 的工作的正常状态更新，甚至在他们的管理可能不认可项目价值的情况下，这是更好的选择。拥有一个全公司范围的官方政策和支持它的企业文化使这成为可能。第 三，允许工程师花费一小部分时间在更有趣的东西上工作，保持工程师的动机和兴奋，他们做什么，并阻止他们被烧毁，这很容易发生，如果他们觉得被迫花费 100％他们的时间工作在更繁琐的任务上面。在工作积极性方面，积极性的工程师和烧坏的工程师之间的生产力差异是超过20％。第四，它鼓励创新企业文化。 看到其他工程师工作在有趣的实验性的20％项目鼓励大家做同样的事情。

#### 3.2. Objectives and Key Results (OKRs) 目标和关键结果（OKR）

**Individuals and teams at Google are required to explicitly document their goals and to
assess their progress towards these goals**. Teams set quarterly and annual objectives, with
measurable key results that show progress towards these objectives. This is done at every
level of the company, going all the way up to defining goals for the whole company. Goals for
individuals and small teams should align with the higher-level goals for the broader teams that
they are part of and with the overall company goals. At the end of each quarter, progress
towards the measurable key results is recorded and each objective is given a score from 0.0 (no
progress) to 1.0 (100% completion). OKRs and OKR scores are normally made visible across
Google (with occasional exceptions for especially sensitive information such as highly
confidential projects), but they not used directly as input to an individual’s performance
appraisal.

**Google的个人和团队需要明确记录他们的目标，并评估他们在实现这些目标方面取得的进展**。 团队制定季度和年度目标，可衡量的关键结果显示在实现这些目标方面取得进展。这是在公司的每一个层面上进行的，一直到整个公司的定义目标。个人和小团队的 目标应该与他们所参与的更广泛团队以及整个公司目标的更高级目标保持一致。在每个季度末，记录可衡量的关键结果的进展，每个目标的得分为0.0（无进展） 至1.0（完成100％）。 OKR和OKR分数通常在Google整个公司上可见（对于特别敏感的信息，例如高度保密的项目偶尔有例外），但它们不直接用作个人绩效评估的输入。

OKRs should be set high: the desired target overall average score is 65%, meaning that a team
is encouraged to set as goals about 50% more tasks than they are likely to actually accomplish.
If a team scores significantly higher than that, they are encouraged to set more ambitious OKRs
for the following quarter (and conversely if they score significantly lower than that, they are
encouraged to set their OKRs more conservatively the next quarter).

OKR 应该往高设置：期望的目标总平均分是65％，这意味着鼓励团队将大约50％的任务设置为比它们可能实际完成的任务要多。如果一个团队的得分明显高于这个数 字，鼓励他们在下一个季度设置更加雄心勃勃的OKR（反之，如果他们得分明显低于这个数字，鼓励他们在下个季度更加保守地设置OKR）。

OKRs provide a key mechanism for communicating what each part of the company is working
on, and for encouraging good performance from employees via social incentives… engineers
know that their team will have a meeting where the OKRs will be scored, and have a natural
drive to try to score well, even though OKRs have no direct impact on performance appraisals
or compensation. Defining key results that are objective and measurable helps ensure that this
human drive to perform well is channelled to doing things that have real concrete measurable
impact on progress towards shared objectives.

OKR 提供了一个关键的机制，用于沟通公司的每个部分工作，并鼓励员工通过社会奖励良好的绩效…工程师知道他们的团队将有一个会议，OKR将得分，并有自然 的驱动力尽管OKR对绩效考核或薪酬没有直接影响，但是尽力取得好成绩。定义客观和可衡量的关键结果有助于确保这种良好表现的人类驱动力被引导到对实现共 同目标的进展具有实际具体可衡量影响的事情。

#### 3.3 Project approval 项目审批

Although there is a well-defined process for launch approvals, Google does not have a
well-defined process for project approval or cancellation. Despite having been at Google for
nearly 10 years, and now having become a manager myself, I still don’t fully understand how
such decisions are made. In part this is because the approach to this is not uniform across the
company. Managers at every level are responsible and accountable for what projects their
teams work on, and exercise their discretion as they see fit. In some cases, this means that
such decisions are made in a quite bottom-up fashion, with engineers being given freedom to
choose which projects to work on, within their team’s scope. In other cases, such decisions are
made in a much more top-down fashion, with executives or managers making decisions about
which projects will go ahead, which will get additional resources, and which will get cancelled.

尽管有一个明确定义的启动批准流程，但Google没有明确定义的项目审批或取消流程。尽管已经在Google工作了近10年，现在已经成为一名经理，我仍 然不能完全理解如何做出这样的决定。在某种程度上，这是因为在整个公司，这种方法是不统一的。每个级别的经理都对他们的团队工作的项目负责，并且在他们认 为合适的时候行使其自由裁量权。在某些情况下，这意味着这种决策是以自下而上的方式进行的，工程师可以自由选择在其团队范围内工作的项目。在其他情况下， 这种决策以更自上而下的方式进行，公司高层或中层管理人员决定哪些项目将进行，哪些将获得额外的资源，哪些将被取消。

#### 3.4 Corporate reorganizations 机构重组

Occasionally an executive decision is made to cancel a large project, and then the many
engineers who had been working on that project may have to find new projects on new teams.
Similarly there have been occasional “defragmentation” efforts, where projects that are split
across multiple geographic locations are consolidated into a smaller number of locations, with
engineers in some locations being required to change team and/or project in order to achieve
this. In such cases, engineers are generally given freedom to choose their new team and role
from within the positions available in their geographic location, or in the case of
defragmentation, they may also be given the option of staying on the same team and project by
moving to a different location.

偶尔，公司高层决定取消一个大型项目，然后许多工程师在该项目上工作可能需要在新的团队上找到新的项目。类似地，偶尔进行“碎片整理”工作，其中分散在多个 地理位置的项目被合并到较少数量的地点，其中一些地点中的工程师被要求改变团队和/或项目以实现这一点。在这种情况下，工程师通常可以自由地从他们的地理 地点可用的位置中选择他们的新团队和角色，或者在碎片整理的情况下，他们也可以被选择留在同一个团队和项目通过移动到不同的地点。

In addition, other kinds of corporate reorganizations, such as merging or splitting teams and
changes in reporting chains, seem to be fairly frequent occurrences, although I don’t know how
Google compares with other large companies on that. In a large, technology-driven
organization, somewhat frequent reorganization may be necessary to avoid organizational
inefficiencies as the technology and requirements change.

此外，其他类型的机构重组，如合并或拆分团队和报告链中的变化，似乎是相当频繁的事件，虽然我不知道Google如何与其他大公司比较有啥区别。在一个大型，技术驱动的组织中，可能需要进行频繁的重组，以避免由于技术和需求的变化而导致组织效率低下。

### 4. People management 人员管理

#### 4.1. Roles 角色

As we’ll explain in more detail below, Google separates the engineering and management
career progression ladders, separates the tech lead role from management, embeds research
within engineering, and supports engineers with product managers, project managers, and site
reliability engineers (SREs). It seems likely that at least some of these practices are important
to sustaining the culture of innovation that has developed at Google.

我们将在下面更详细地解释，Google将工程和管理职业发展阶梯分离开，将技术主导角色与管理层分开，将研究嵌入工程，并支产品经理，项目经理和现场可靠性工程师（SRE）支持工程师 。看来，至少有一些做法对维持Google开发的创新文化很重要。

Google has a small number of different roles within engineering. Within each role, there is a
career progression possible, with a sequence of levels, and the possibility of promotion (with
associated improvement to compensation, e.g. salary) to recognize performance at the next
level.

Google在工程中有少量不同的角色。在每个角色中，有一个职业发展有不同的可能发展途径，一系列的水平，以及晋升（与相关的改善薪酬，如薪水）的可能性，以承认在下一个水平的绩效。

The main roles are these:

主要角色是：

  * **Engineering Manager 工程经理**

  This is the only people management role in this list. Individuals in other roles such as
  Software Engineer may manage people, but Engineering Managers always manage
  people. Engineering Managers are often former Software Engineers, and invariably
  have considerable technical expertise, as well as people skills.

  这是此列表中唯一的人员管理角色。具有其他角色的人员（例如软件工程师）可以管理人员，但是工程经理总是管理人员。工程经理通常是前软件工程师，并且总是有相当多的技术专长，以及管理人的技能。

  **There is a distinction between technical leadership and people management**.
  Engineering Managers do not necessarily lead projects; projects are led by a Tech Lead,
  who can be an Engineering Manager, but who is more often a Software Engineer. A
  project’s Tech Lead has the final say for technical decisions in that project.

  **技术领导和人员管理之间有着区别**。工程经理不一定领导项目；项目由技术主管领导；技术主管可以是工程经理，但更多时候是软件工程师。项目的技术主管对该项目的技术决策有最终决定权。

  Managers are responsible for selecting Tech Leads, and for the performance of their
  teams. They perform coaching and assisting with career development, do performance
  evaluation (using input from peer feedback, see below), and are responsible for some
  aspects of compensation. They are also responsible for some parts of the hiring
  process.

  经理负责选择技术主管，评估他们团队的表现。他们执行指导和协助职业发展，做绩效评估（使用同行反馈的意见，见下文），并且负责一些薪酬一些方面的事情。他们还负责招聘过程的某些环节。

  Engineering Managers normally directly manage anywhere between 3 and 30 people,
  although 8 to 12 is most common.

  工程经理通常直接管理3到30人之间，虽然8到12是最常见的。

  * **Software Engineer (SWE) 软件工程师**

  Most people doing software development work have this role. The hiring bar for
  software engineers at Google is very high; by hiring only exceptionally good software
  engineers, a lot of the software problems that plague other organizations are avoided or
  minimized.

  大多数做软件开发工作的人都是这个角色。 Google的软件工程师的招聘标准非常高；通过聘用特别好的软件工程师，困扰其他组织的许多软件问题被避免或最小化。

  **Google has separate career progression sequences for engineering and
  management**​. Although it is possible for a Software Engineer to manage people, or to
  transfer to the Engineering Manager role, managing people is not a requirement for
  promotion, even at the highest levels. At the higher levels, showing leadership is
  required, but that can come in many forms. For example creating great software that
  has a huge impact or is used by very many other engineers is sufficient. This is
  important, because it means that people who have great technical skills but lack the
  desire or skills to manage people still have a good career progression path that does not
  require them to take a management track. This avoids the problem that some
  organizations suffer where people end up in management positions for reasons of career
  advancement but neglect the people management of the people in their team.

  **Google有着独立的工程和管理职业发展序列**。 虽然软件工程师可以管理人员或转移到工程经理角色，但管理人员不是升级的要求，即使是在最高级别。在更高层次，显示领导才能是必需的，但可以有许多形式。 例如，创建具有巨大影响或被许多其他工程师使用的伟大软件就足够了。这是非常重要的，因为这意味着那些具有很强的技术技能但缺乏管理人的愿望或技能的人仍 然有良好的职业发展道路，不需要他们选择管理道路。这避免了一些组织遭受的问题，其中人们由于职业晋升的原因而最终爬上管理职位，但忽略了他们团队中的人的管理。

  * **Research Scientist 研究科学家**

  The hiring criteria for this role are very strict, and the bar is extremely high, requiring
  demonstrated exceptional research ability evidenced by a great publication record * and *
  ability to write code. Many very talented people in academia who would be able to
  qualify for a Software Engineer role would not qualify for a Research Scientist role at
  Google; most of the people with PhDs at Google are Software Engineers rather than
  Research Scientists. Research scientists are evaluated on their research contributions,
  including their publications, but apart from that and the different title, there is not really
  that much difference between the Software Engineer and Research Scientist role at
  Google. Both can do original research and publish papers, both can develop new
  product ideas and new technologies, and both can and do write code and develop
  products. Research Scientists at Google usually work alongside Software Engineers, in the same teams and working on the same products or the same research. This practice of embedding research within engineering contributes greatly to the ease with which new research can be incorporated into shipping products.

  这个角色的招聘标准非常严格，并且条件非常高，需要表现出杰出的研究能力，这是出色的出版记录*和*编写代码的能力证明。许多非常有才能的学术界人士，如果 能够获得软件工程师的资格，就不能在Google获得研究科学家的资格；Google大多数获得博士学位的人是软件工程师，而不是研究科学家。研究科学家 被评估他们的研究贡献，包括他们的出版物，但除此之外，不同的标题，软件工程师和研究科学家在谷歌的作用没有太大的区别。两者都可以做原创研究和发表论 文，都可以开发新的产品思想和新技术，并可以写代码和开发产品。 Google的研究科学家通常与软件工程师一起，在相同的团队中在相同的产品或相同的研究上工作。这种在工程中嵌入研究的做法极大地促进了新的研究能够被 容纳到运输产品中。

  * **Site Reliability Engineer (SRE) 站点可靠性工程师**

  The maintenance of operational systems is done by software engineering teams, rather
  than traditional sysadmin types, but the hiring requirements for software engineering
  skills for the SRE are slightly lower than the requirements for the Software Engineering
  position. The nature and purpose of the SRE role is explained very well and in detail in
  the SRE book [7], so we won’t discuss it further here.

  操作系统的维护由软件工程团队完成，而不是传统的系统管理员类型，但SRE的软件工程技能的招聘要求略低于软件工程职位的要求。 SRE角色的性质和目的在SRE书[7]中有详细的解释，因此我们不在这里进一步讨论。

  * **Product Manager 产品经理**

  Product Managers are responsible for the management of a product; as advocates for
  the product users, they coordinate the work of software engineers, evangelizing features
  of importance to those users, coordinating with other teams, tracking bugs and
  schedules, and ensuring that everything needed is in place to produce a high quality
  product. Product Managers usually do NOT write code themselves, but work with
  software engineers to ensure that the right code gets written.

  产品经理负责管理产品；作为产品用户的倡导者，他们协调软件工程师的工作，对这些用户传播重要的功能，与其他团队协调，跟踪Bug和计划，以及确保所需的一切都到位，以产生高质量的产品。产品经理通常不会自己编写代码，而是与软件工程师合作，以确保编写正确的代码。

  * **Program Manager / Technical Program Manager 项目经理/技术项目经理**

  Program Managers have a role that is broadly similar to Product Manager, but rather
  than managing a product, they manage projects, processes, or operations (e.g. data
  collection). Technical Program Managers are similar, but also require specific technical
  expertise relating to their work, e.g. linguistics for dealing with speech data.

  项目经理的角色与产品经理大致相似，但是他们不是管理产品，而是管理项目、流程或运维（例如数据收集）。技术计划经理相似，但也需要与他们的工作有关的专门 的技术专门知识。处理语音数据的语言学。

  The ratio of Software Engineers to Product Managers and Program Managers varies
  across the organization, but is generally high, e.g. in the range 4:1 to 30:1.

  在整个组织中，软件工程师与产品经理和计划管理者的比例不尽相同，但一般较高，例如，在4：1至30：1的范围 内。


#### 4.2. Facilities 设施

Google is famous for it’s fun facilities, with features like slides, ball pits, and games rooms.
That helps attract and retain good talent. Google’s excellent cafes, which are free to
employees, provide that function too, and also subtly encourage Googlers to stay in the office;
hunger is never a reason to leave. The frequent placement of “microkitchens” where employees
can grab snacks and drinks serves the same function too, but also acts as an important source
of informal idea exchange, as many conversations start up there. Gyms, sports, and on-site
massage help keep employees fit, healthy, and happy, which improves productivity and
retention.

Google 是以有趣的设施而闻名天下，它具有幻灯片、球技和游戏室等功能。这有助于吸引和留住优秀的人才。 Google优秀咖啡馆对员工免费，提供了那些功能，也巧妙鼓励Google员工留在办公室；饥饿绝不是离开的理由。 “微型厨房”的频繁安置，员工可以享用小吃和饮料，也提供同样的功能，但也作为非正式的想法交流的重要来源，因为许多对话开始在那里展开。健身房、运动和 现场按摩帮助保持员工健身、健康和快乐，这提高了生产力和保持。

The seating at Google is open-plan, and often fairly dense. While controversial [20], this
encourages communication, sometimes at the expense of individual concentration, and is
economical.

Google的座位是开放式的，通常相当密集。虽然有争议[20]，这鼓励沟通，虽然有时牺牲个人太过集中的利益，但是很经济的。

Employees are assigned an individual seat, but seats are re-assigned fairly frequently (e.g.
every 6-12 months, often as a consequence of the organization expanding), with seating chosen
by managers to facilitate and encourage communication, which is always easier between
adjacent or nearly adjacent individuals.

员工被分配一个单独的座位，但座位重新分配相当频繁（通常由于组织扩展的结果，例如每6-12个月重新分配一次），由管理者安排座位，以促进和鼓励沟通，这是相邻之间或几乎相邻的个体沟通总是很容易。

Google’s facilities all have meeting rooms fitted with state-of-the-art video conference facilities,
where connecting to the other party for a prescheduled calendar invite is just a single tap on the
screen.

Google的设施都配备最先进的视频会议设施的会议室，连接到对方预订的日历邀请只是一个点击在屏幕上就可完成。

#### 4.3. Training 培训

Google encourages employee education in many ways:

Google鼓励员工在许多方面受到教育：

  * New Googlers (“Nooglers”) have a mandatory initial training course. 新的Google员工（“Noogler”）有一个强制性的初始培训课程。

  * Technical staff (SWEs and research scientists) start by doing “Codelabs”: short online
training courses in individual technologies, with coding exercises.技术人员（SWE和研究科学家）从“Codelabs”开始：在线短期个人技术培训课程，包括编码练习。

  * Google offers employees a variety of online and in-person training courses.Google为员工提供各种在线和面对面培训课程。

  * Google also offers support for studying at external institutions.Google还支持在外部机构学习。

In addition, each Noogler is usually appointed an official “Mentor” and a separate “Buddy” to
help get them up to speed. Unofficial mentoring also occurs via regular meetings with their
manager, team meetings, code reviews, design reviews, and informal processes.

此外，每个Noogler通常被任命为官方“导师”和一个单独的“伙伴”，以帮助他们入职的速度。非正式指导还通过与其经理、小组会议、代码审查、设计审查和非正式流程的定期会议进行。

#### 4.4 Transfers 换岗

**Transfers between different parts of the company are encouraged​**, to help spread
knowledge and technology across the organization and improve cross-organization
communication. Transfers between projects and/or offices are allowed for employees in good
standing after 12 months in a position. Software engineers are also encouraged to do
temporary assignments in other parts of the organization, e.g. a six-month “rotation” (temporary
assignment) in SRE (Site Reliability Engineering).

**Google 鼓励公司不同部门之间的换岗**，以帮助在整个组织传播知识和技术，并改善跨组织沟通。允许在12个月后处于良好状态的员工在项目和/或办公室之间进行转移。 还鼓励软件工程师在组织的其他部门进行临时指派任务。例如， SRE（站点可靠性工程）中的六个月“轮换”（临时任务）。


#### 4.5. Performance appraisal and rewards  绩效考核和奖励

Feedback is strongly encouraged at Google. Engineers can give each other explicit positive
feedback via “peer bonuses” and “kudos”. Any employee can nominate any other employee for
a “peer bonus” -- a cash bonus of $100 -- up to twice per year, for going beyond the normal call
of duty, just by filling in a web form to describe the reason. Team-mates are also typically
notified when a peer bonus is awarded. Employees can also give “kudos”, formalized
statements of praise which provide explicit social recognition for good work, but with no financia
reward; for “kudos” there is no requirement that the work be beyond the normal call of duty, and
no limit on the number of times that they can be bestowed.

Google 极力鼓励反馈。工程师可以通过“对等奖金”和“kudos”给彼此明确的积极反馈。任何员工都可以任何其他员工提名“同伴奖金” – 现金奖金100美元，每年最多两次。超越正常的工作职责，只需填写一个网络表单来描述原因。当同伴奖励被授予时，队友也通常得到通知。员工也可以给予 “kudos”，形式化的赞美声明，为良好的工作提供明确的社会认可，但没有财政奖励;对于“kudos”，没有要求工作超出正常的职责，并且对它们可以 被授予的次数的限制没有限制。

Managers can also award bonuses, including spot bonuses, e.g. for project completion. And as
with many companies, Google employees get annual performance bonuses and equity awards
based on their performance.

经理也可以发放奖金，包括现金奖金，例如项目完成。和许多公司一样，Google员工根据其绩效获得年度绩效奖金和股权奖励。

Google has a very careful and detailed promotion process, which involves nomination by self or
manager, self-review, peer reviews, manager appraisals; the actual decisions are then made by
promotion committees based on that input, and the results can be subject to further review by
promotion appeals committees. Ensuring that the right people get promoted is critical to
maintaining the right incentives for employees.

Google有一个非常仔细和详细的晋升过程，包括由毛遂自荐或经理提名、自我审查、同行评审、经理评价;然后由促进委员会根据这些意见作出实际决定，结果可以由促进上诉委员会进一步审查。确保正确的人得到晋升对于为员工维持适当的激励是至关重要。

Poor performance, on the other hand, is handled with manager feedback, and if necessary with
performance improvement plans, which involve setting very explicit concrete performance
targets and assessing progress towards those targets. If that fails, termination for poor
performance is possible, but in practice this is extremely rare at Google.

另一方面，绩效不佳则由经理反馈处理，如果有必要，还需要制定绩效改进计划，其中包括制定非常明确的具体绩效目标，并评估实现这些目标的进展情况。如果失败，可能会导致效果不佳，但实际上这在Google上极为罕见。

Manager performance is assessed with feedback surveys; every employee is asked to fill in an
survey about the performance of their manager twice a year, and the results are anonymized
and aggregated and then made available to managers. This kind of upward feedback is very
important for maintaining and improving the quality of management throughout the organization.

经理绩效通过反馈调查进行评估； 要求每个员工每年两次填写关于他们的经理业绩的调查，结果被匿名化和聚合，然后提供给经理。这种向上反馈对于维护和提高整个组织的管理质量非常重要。

### 5. Conclusions 总结

We have briefly described most of the key software engineering practices used at Google. Of
course Google is now a large and diverse organization, and some parts of the organization have
different practices. But the practices described here are generally followed by most teams at
Google.

我们简要介绍了Google使用的大多数关键软件工程实践。当然，Google现在是一个庞大而多元化的组织，组织的一些部门有不同的做法。但是这里描述的做法一般由Google的大多数团队遵循。

With so many different software engineering practices involved, and with so many other reasons
for Google’s success that are not related to our software engineering practices, it is extremely
difficult to give any quantitative or objective evidence connecting individual practices with
improved outcomes. However, these practices are the ones that have stood the test of time at
Google, where they have been subject to the collective subjective judgement of many
thousands of excellent software engineers.

由于涉及这么多不同的软件工程实践，以及Google成功的许多其他原因与我们的软件工程实践无关，因此很难给出任何定量或客观的证据，将个体实践与改进的结果联系起来。然而，Google的这些做法是经受了时间考验的，在那里他们受到了成千上万的优秀软件工程师的集体主观判断。

For those in other organizations who are advocating for the use of a particular practice that
happens to be described in this paper, perhaps it will help to say “it’s good enough for Google”.

对于那些主张使用本文中描述的特定实践的其他组织中的人来说，也许有助于说“对Google来说足够好了”。

### Acknowledgements 致谢

Special thanks to Alan Donovan for his extremely detailed and constructive feedback, and
thanks also to Yaroslav Volovich, Urs Hölzle, Brian Strope, Alexander Gutkin, Alex Gruenstein
and Hameed Husaini for their very helpful comments on earlier drafts of this paper.


特别感谢Alan Donovan的非常详细和建设性的反馈，并感谢Y aroslav Volovich，UrsHölzle，Brian Strope，Alexander Gutkin，Alex Gruenstein和Hameed Husaini对本文的早期草稿非常有帮助的意见。

### References 参考文献

[1] Build in the Cloud: Accessing Source Code, Nathan York,
[http://google-engtools.blogspot.com/2011/06/build-in-cloud-accessing-source-code.html](http://google-engtools.blogspot.com/2011/06/build-in-cloud-accessing-source-code.html)
[2] Build in the Cloud: How the Build System works, Christian Kemper,
[http://google-engtools.blogspot.com/2011/08/build-in-cloud-how-build-system-works.htm](http://google-engtools.blogspot.com/2011/08/build-in-cloud-how-build-system-works.htm)
[3] Build in the Cloud: Distributing Build Steps, Nathan York
[http://google-engtools.blogspot.com/2011/09/build-in-cloud-distributing-build-steps.html](http://google-engtools.blogspot.com/2011/09/build-in-cloud-distributing-build-steps.html)
[4] Build in the Cloud: Distributing Build Outputs, Milos Besta, Yevgeniy Miretskiy and Jeff Cox
[http://google-engtools.blogspot.com/2011/10/build-in-cloud-distributing-build.html](http://google-engtools.blogspot.com/2011/10/build-in-cloud-distributing-build.html)
[5] Testing at the speed and scale of Google, Pooja Gupta, Mark Ivey, and John Penix, Google
engineering tools blog, June 2011.
[http://google-engtools.blogspot.com/2011/06/testing-at-speed-and-scale-of-google.html]()
[6] Building Software at Google Scale Tech Talk, Michael Barnathan, Greg Estren, Pepper
Lebeck-Jone, Google tech talk, [http://www.youtube.com/watch?v=2qv3fcXW1mg](http://www.youtube.com/watch?v=2qv3fcXW1mg)
[7] Site Reliability Engineering, Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy,
O'Reilly Media, April 2016, ISBN 978-1-4919-2909-4.
[https://landing.google.com/sre/book.html](https://landing.google.com/sre/book.html)
[8] How Google Works, Eric Schmidt, Jonathan Rosenberg. [http://www.howgoogleworks.net](http://www.howgoogleworks.net)
[9] What would Google Do?: Reverse-Engineering the Fastest Growing Company in the History
of the World, Jeff Jarvis, Harper Business, 2011.
[https://books.google.co.uk/books/about/What_Would_Google_Do.html?id=GvkEcAAACAAJ&redir_esc=y](https://books.google.co.uk/books/about/What_Would_Google_Do.html?id=GvkEcAAACAAJ&redir_esc=y)
[10] The Search: How Google and Its Rivals Rewrote the Rules of Business and Transformed
Our Culture, John Battelle, 8 September 2005.
[https://books.google.co.uk/books/about/The_Search.html?id=4MY8PgAACAAJ&redir_esc=y](https://books.google.co.uk/books/about/The_Search.html?id=4MY8PgAACAAJ&redir_esc=y)
[11] The Google Story, David A. Vise, Pan Books, 2008. [http://www.thegooglestory.com/](http://www.thegooglestory.com/)
[12] Searching for Build Debt: Experiences Managing Technical Debt at Google, J. David
Morgenthaler, Misha Gridnev, Raluca Sauciuc, and Sanjay Bhansali.
[http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/37755.pdf](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/37755.pdf)
[13] Development at the speed and scale of Google, A. Kumar, December 2010, presentation,
QCon. [http: //www.infoq.com/presentations/Development-at-Google]
[14] How Google Tests Software, J. A. Whittaker, J. Arbon, and J. Carollo, Addison-Wesley,
2012.
[15] Release Engineering Practices and Pitfalls, H. K. Wright and D. E. Perry, in Proceedings of
the 34th International Conference on Software Engineering (ICSE ’12), IEEE, 2012, pp.
1281–1284. [http://www.hyrumwright.org/papers/icse2012.pdf](http://www.hyrumwright.org/papers/icse2012.pdf)
[16] Large-Scale Automated Refactoring Using ClangMR, H. K. Wright, D. Jasper, M. Klimek, C.
Carruth, Z. Wan, in Proceedings of the 29th International Conference on Software Maintenance
(ICSM ’13), IEEE, 2013, pp. 548–551.
[17] Why Google Stores Billions of Lines of Code in a Single Repository, Rachel Potvin,
presentation. [https://www.youtube.com/watch?v=W71BTkUbdqE](https://www.youtube.com/watch?v=W71BTkUbdqE)
[18] The Motivation for a Monolithic Codebase, Rachel Potvin, Josh Levenberg, to be published
in Communications of the ACM, July 2016.
[19] Scaling Mercurial at Facebook, Durham Goode, Siddharth P. Agarwa, Facebook blog post,
January 7th, 2014.
[https://code.facebook.com/posts/218678814984400/scaling-mercurial-at-facebook/](https://code.facebook.com/posts/218678814984400/scaling-mercurial-at-facebook/)
[20] Why We (Still) Believe In Private Offices, David Fullerton, Stack Overflow blog post,
January 16th, 2015.
[https://blog.stackoverflow.com/2015/01/why-we-still-believe-in-private-offices/](https://blog.stackoverflow.com/2015/01/why-we-still-believe-in-private-offices/)
[21] Continuous Integration at Google Scale, John Micco, presentation, EclipseCon, 2013.
[http://eclipsecon.org/2013/sites/eclipsecon.org.2013/files/2013-03-24%20Continuous%20Integration%20at%20Google%20Scale.pdf](http://eclipsecon.org/2013/sites/eclipsecon.org.2013/files/2013-03-24%20Continuous%20Integration%20at%20Google%20Scale.pdf)
