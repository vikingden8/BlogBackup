---
title: Getting Started With Gradle Introduction
date: 2017-06-27 20:14:41
tags:
categories: "持续构建"
---

Gradle is a build tool which replaces XML based build scripts with an internal DSL which is based on Groovy programming language.

It has gained a lot of traction recently and that is I why I decided to take a closer look at it.

This blog post is the first part of my Gradle tutorial, and this blog post has two goals:

* help us to install Gradle

* describe some of its basic concepts which helps us to understand the future parts of this tutorial.

Let’s start by finding out how we can install Gradle.

### Installing Gradle


* [Download the binaries from the downloads page](http://www.gradle.org/downloads).

* Unpack the zip file and add the **GRADLE_HOME/bin** directory to the **PATH** environment variable.

* If we are using OS X, we can install Gradle by using Homebrew. We can do this by running the following command at command prompt:

```sh
brew install gradle
```

We can verify that Gradle is working properly by running the command **gradle -v** at command prompt. If Gradle is working properly, we should the following output (Windows and Linux users will naturally see a bit different output):

```sh
viking@Viking ~ $ gradle -v

------------------------------------------------------------
Gradle 3.4
------------------------------------------------------------

Build time:   2017-02-20 14:49:26 UTC
Revision:     73f32d68824582945f5ac1810600e8d87794c3d4

Groovy:       2.4.7
Ant:          Apache Ant(TM) version 1.9.6 compiled on June 29 2015
JVM:          1.8.0_65 (Oracle Corporation 25.65-b01)
OS:           Linux 4.4.0-53-generic amd64

viking@Viking ~ $

```

<!--more-->

Let’s take a quick look at the basic concepts of a Gradle build.

### A Short Introduction to Gradle Build

Gradle has two basic concepts: **projects** and **tasks**. These concepts are explained in the following:

* A project is either something we build (e.g. a jar file) or do (deploy our application to production environment). A project consists of one or more tasks.

* A task is an atomic unit work which is performed our build (e.g. compiling our project or running tests).

So, how are these concepts related to a Gradle build? Well, every Gradle build contains one or more projects.

The relationships between these concepts are illustrated in the following figure:

![](/images/categories/ci/001/gradlebuild.jpg)

We can configure our Gradle build by using the following configuration files:

* The Gradle build script (_build.gradle_) specifies a project and its tasks.

* The Gradle properties file (_gradle.properties_) is used to configure the properties of the build.

* The Gradle Settings file (_settings.gradle_) is optional in a build which has only one project. If our Gradle build has more than one projects, it is mandatory because it describes which projects participate to our build. Every multi-project build must have a settings file in the root project of the project hierarchy.

Let’s move on and find out how we can add functionality to a Gradle build by using Gradle plugins.

### Even Shorter Introduction to Gradle Plugins

The design philosophy of Gradle is that all useful features are provided by Gradle plugins. A Gradle plugin can:

* Add new tasks to the project.

* Provide a default configuration for the added tasks. The default configuration adds new conventions to the project (e.g. the location of source code files).

* Add new properties which are used to override the default configuration of the plugin.

* Add new dependencies to the project.

We can apply a Gradle plugin (this term is used when we add a plugin to a project) by using either its name or type.

We can apply a plugin by name (the name of the plugin is foo) by adding the following line to the build.gradle file:

```sh
apply plugin: 'foo'
```

On the other hand, if we want to apply a plugin by type (the type of the plugin is com.bar.foo), we must add the following line to the build.gradle file:

```sh
apply plugin: 'com.bar.foo'
```

That’s all for today. Let’s summarize what we learned from this blog post.

### Summary

This blog post has taught us three things:

* We learned how we can install Gradle.

* We understand the basic building blocks of a Gradle build.

* We know how we can add functionality to our build by using Gradle plugins.

From:[Getting Started With Gradle: Introduction](https://www.petrikainulainen.net/programming/gradle/getting-started-with-gradle-introduction/)
