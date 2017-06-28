---
title: Getting Started With Gradle-Our First Java Project
date: 2017-06-28 20:38:41
tags:
categories: "持续构建"
---

This blog post describes how we can compile and package a simple Java project by using Gradle.

Our Java project has only one requirement:

Our build script must create an executable jar file. In other words, we must be able to run our program by using the command:

```sh
java -jar jarfile.jar
```

Let’s find out how we can fulfil this requirement.

### Creating a Java Project

We can create a Java project by applying the Java plugin. We can do this by adding the following line to our build.gradle file:

```xml
apply plugin: 'java'
```

That is it. We have now created a Java project.

The Java plugin adds new conventions (e.g. the default project layout), new tasks, and new properties to our build.

Let’s move on and take a quick look at the default project layout.

<!--more-->
#### The Project Layout of a Java Project

The default project layout of a Java project is following:

* The _src/main/java_ directory contains the source code of our project.

* The _src/main/resources_ directory contains the resources (such as properties files) of our project.

* The _src/test/java_ directory contains the test classes.

* The _src/test/resources_ directory contains the test resources.

All output files of our build are created under the _build_ directory. This directory contains the following subdirectories which are relevant to this blog post (there are other subdirectories too, but we will talk about them in the future):

* The _classes_ directory contains the compiled .class files.

* The _libs_ directory contains the jar or war files created by the build.

Let’s move on and add a simple main class to our project.

#### Adding a Main Class to Our Build

Let’s create a simple main class which prints the words: “Hello World” to _System.out_. The source code of the _HelloWorld_ class looks as follows:

```java
package com.viking.gradle;

public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

>The _HelloWorld_ class was added to the _src/main/java/com/viking/gradle_ directory.

That is nice. However, we still have to compile and package our project. Let’s move on and take a look at the tasks of a Java project.

#### The Tasks of a Java Project

The Java plugin adds many tasks to our build but the tasks which are relevant for this blog post are:

* The _assemble_ task compiles the source code of our application and packages it to a jar file. This task doesn’t run the unit tests.

* The _build_ task performs a full build of the project.

* The _clean_ task deletes the build directory.

* The _compileJava_ task compiles the source code of our application.

We can also get the full list of runnable tasks and their description by running the following command at the command prompt:

```sh
gradle tasks
```

This is a good way to get a brief overview of our project without reading the build script. If we run this command in the root directory of our example project, we see the following output:

```sh
> gradle tasks
:tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles classes 'main'.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
testClasses - Assembles classes 'test'.

Build Setup tasks
-----------------
init - Initializes a new Gradle build. [incubating]
wrapper - Generates Gradle wrapper files. [incubating]

Documentation tasks
-------------------
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
dependencies - Displays all dependencies declared in root project 'first-java-project'.
dependencyInsight - Displays the insight into a specific dependency in root project 'first-java-project'.
help - Displays a help message
projects - Displays the sub-projects of root project 'first-java-project'.
properties - Displays the properties of root project 'first-java-project'.
tasks - Displays the tasks runnable from root project 'first-java-project'.

Verification tasks
------------------
check - Runs all checks.
test - Runs the unit tests.

Rules
-----
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.
Pattern: clean<TaskName>: Cleans the output files of a task.

To see all tasks and more detail, run with --all.

BUILD SUCCESSFUL

Total time: 2.792 secs
```

Let’s move on and find out how we can package our Java project.

### Packaging Our Java Project

We can package our application by using two different tasks:

If the run the command _gradle assemble_ at command prompt, we see the following output:

```sh
> gradle assemble
:compileJava
:processResources
:classes
:jar
:assemble

BUILD SUCCESSFUL

Total time: 3.163 secs
```

If we run the command _gradle build_ at command prompt, we see the following output:

```sh
> gradle build
:compileJava
:processResources
:classes
:jar
:assemble
:compileTestJava
:processTestResources
:testClasses
:test
:check
:build

BUILD SUCCESSFUL

Total time: 3.01 secs
```

The outputs of these commands demonstrate that the difference of these tasks is that:

* The _assemble_ task runs only the tasks which are required to package our application.

* The _build_ task runs the tasks which are required to package our application AND runs automated tests.

Both of these commands create the _first-java-project.jar_ file to the _build/libs_ directory.

>The default name of the created jar file is created by using the following template: [project name].jar, and the default name of the project is the same than the name of the directory in which it is created. Because the name of our project directory is first-java-project, the name of created jar is first-java-project.jar.

We can now try to run our application by using the following command:

```sh
java -jar first-java-project.jar
```

When we do this, we see the following output:


```sh
> java -jar first-java.project.jar
No main manifest attribute, in first-java-project.jar
```

The problem is that we haven’t configured the main class of the jar file in the manifest file. Let’s find out how we can fix this problem.

### Configuring the Main Class of a Jar File

The Java plugin adds a jar task to our project, and every jar object has a manifest property which is an instance of _Manifest_.

We can configure the main class of the created jar file by using the _attributes()_ method of the _Manifest_ interface. In other words, we can specify the attributes added to the manifest file by using a map which contains key-value pairs.

We can set the entry point of our application by setting the value of the _Main-Class_ attribute. After we have made the required changes to the build.gradle file, its source code looks as follows (the relevant part is highlighted):

```XML
apply plugin: 'java'

jar {
    manifest {
        attributes 'Main-Class': 'com.viking.gradle.HelloWorld'
    }
}
```

After we have created a new jar file by running either the _gradle assemble_ or _gradle build_ command, we can run the jar file by using the following command:

```sh
java -jar first-java-project.jar
```

When we run our application, the following text is printed to the _System.out_:

```sh
> java -jar first-java-project.jar
Hello World!
```

That is all for today. Let’s find out what we learned from this blog post.

### Summary

We have now created a simple Java project by using Gradle. This blog post has taught us four things:

* We know that we can create a Java project by applying the Gradle Java plugin.

* We learned that the default directory layout of a Java project is the same than the default directory layout of a Maven project.

* We learned that all output files produced by our build can be found from the build directory.

* We learned how we can customize the attributes added to the manifest file.

From : [Getting Started With Gradle: Our First Java Project](https://www.petrikainulainen.net/programming/gradle/getting-started-with-gradle-our-first-java-project/)
