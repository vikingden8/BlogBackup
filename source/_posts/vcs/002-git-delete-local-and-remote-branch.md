---
title: How to delete local and remote branch in Git
date: 2017-05-27 17:53:18
tags:
categories: "版本控制系统"
---

* First, change directory to your project

Command : $ cd <ProjectPath>

```sh
viking@Viking ~ $
viking@Viking ~ $ cd work/android-projects/Android-Architecture/
viking@Viking ~/work/android-projects/Android-Architecture $
viking@Viking ~/work/android-projects/Android-Architecture $
```

* list all branchs under your project

Command : $ git branch -a

```sh
viking@Viking ~/work/android-projects/Android-Architecture $ git branch -a
  02-mvc
  02-mvp1
  02-mvp2
* 02-mvp3
  empty
  master
  remotes/origin/02-mvc
  remotes/origin/02-mvp1
  remotes/origin/02-mvp2
  remotes/origin/02-mvp3
  remotes/origin/master
```

<!--more-->

* delete local branch

Command : $ git branch -d <BranchName>

```sh
viking@Viking ~/work/android-projects/Android-Architecture $ git branch -D 02-mvp1
Deleted branch 02-mvp1 (was 47bd042).
viking@Viking ~/work/android-projects/Android-Architecture $ git branch -D 02-mvp2
Deleted branch 02-mvp2 (was 804a53b).
viking@Viking ~/work/android-projects/Android-Architecture $ git branch -D 02-mvp3
Deleted branch 02-mvp3 (was 36f6666).
```

* delete remote branch

Command : $ git push origin --delete <BranchName>

```sh
viking@Viking ~/work/android-projects/Android-Architecture $ git push origin --delete 02-mvp1
Username for 'http://code.viking.com:3000': dengwj
Password for 'http://dengwj@code.viking.com:3000':
To http://code.viking.com:3000/dengwj/Android-Architecture.git
 - [deleted]         02-mvp1
viking@Viking ~/work/android-projects/Android-Architecture $ git push origin --delete 02-mvp2
Username for 'http://code.viking.com:3000': dengwj
Password for 'http://dengwj@code.viking.com:3000':
To http://code.viking.com:3000/dengwj/Android-Architecture.git
 - [deleted]         02-mvp2
 viking@Viking ~/work/android-projects/Android-Architecture $ git push origin --delete 02-mvp3
 Username for 'http://code.viking.com:3000': dengwj
 Password for 'http://dengwj@code.viking.com:3000':
 To http://code.viking.com:3000/dengwj/Android-Architecture.git
  - [deleted]         02-mvp3

```
