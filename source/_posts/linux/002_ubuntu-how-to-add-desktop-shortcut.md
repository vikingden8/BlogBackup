---
title: Ubuntu创建应用的桌面快捷方式
date: 2017-4-1 14:26:45
tags:
categories: "Linux学习笔记"
---

在Ubuntu上并没有像Windows那样类似的创建桌面快捷方式，比如我在安装Intellij IDEA的时候，只需要解压缩安装包，执行bin目录下的可执行文件就可以运行程序。但是，这样很不方便，在Ubuntu上怎样创建桌面快捷方式呢?

其实，在Ubuntu上，应用的桌面快捷方式全部保存在以下的目录中：

```sh
viking@Viking /usr/share/applications $
viking@Viking /usr/share/applications $ pwd
/usr/share/applications
viking@Viking /usr/share/applications $ ls -l
total 1552
-rw-r--r-- 1 root   root     125 Dec 16  2015 apturl.desktop
-rw-r--r-- 1 root   root     223 Nov  6 00:58 apturl_mime.desktop
-rw-r--r-- 1 root   root     282 Feb 20 08:51 atom.desktop
-rw-r--r-- 1 root   root    5175 May 24  2016 blueberry.desktop
-rw-r--r-- 1 root   root     396 Jan 29  2016 bluetooth-sendto.desktop
-rw-r--r-- 1 root   root    1200 May 27  2016 brasero.desktop
-rw-r--r-- 1 root   root     530 May 27  2016 brasero-nautilus.desktop
-rw-r--r-- 1 root   root     363 Dec 14 08:12 cinnamon2d.desktop
-rw-r--r-- 1 roojiket   root     448 Dec 13 01:28 cinnamon-color-panel.desktop
-rw-r--r-- 1 root   root     300 Dec 13 01:28 cinnamon-control-center.desktop
-rw-r--r-- 1 root   root     484 Dec 13 01:28 cinnamon-datetime-panel.desktop
...

```

<!--more-->

我们可以在这个目录下，创建一个jetbrains-idea.desktop的文件，并添加如下属性：

```sh
[Desktop Entry]
Version=1.0
Type=Application
Name=IntelliJ IDEA
Icon=/home/viking/tools/idea-IU-171.3780.107/bin/idea.png　#IntelliJ IDEA图标
Exec="/home/viking/tools/idea-IU-171.3780.107/bin/idea.sh" %f  #IntelliJ IDEA的启动文件路径
Comment=The Drive to Develop
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-idea
```

保存即可
