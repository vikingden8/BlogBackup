---
title: Getting Started With Gradle: Creating a Binary Distribution
date: 2017-06-30 22:05:49
tags:
categories: "持续构建"
---

After we have created a useful application, the odds are that we want to share it with other people. One way to do this is to create a binary distribution that can be downloaded from our website.

This blog post describes how we can build a binary distribution that fulfils the following requirements:

* Our binary distribution must not use so called “fat jar” approach. In other words, the dependencies of our application must not be packaged into the same jar file than our application.

* Our binary distribution must contain startup scripts for \*nix and Windows operating systems.

* root directory of our binary distribution must contain the license of our application.

Let’s get started.

<!--more-->

### Creating a Binary Distribution

The application plugin is a Gradle plugin that allows us to run our application, install it, and create a binary distribution that doesn’t use the “fat jar” approach.

We can create a binary distribution by making the following changes to the _build.gradle_ file of the example application that we created during the previous part of my Getting Started with Gradle tutorial:

* Remove the configuration of the jar task.

* Apply the application plugin to our project.

* Configure the main class of our application by setting the value of the mainClassName property.

After we have made these changes to our _build.gradle_ file, it looks as follows (the relevant parts are highlighted):

```xml
apply plugin: 'application'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'log4j:log4j:1.2.17'
    testCompile 'junit:junit:4.11'
}

mainClassName = 'com.viking.gradle.HelloWorld'
```

The application plugin adds five tasks to our project:

* The _run_ task starts the application.

* The _startScripts_ task creates startup scripts to the build/scripts directory. This tasks creates startup scripts for Windows and \*nix operating systems.

* The _installApp_ task installs the application into the build/install/[project name] directory.

* The _distZip_ task creates the binary distribution and packages it into a zip file that is found from the build/distributions directory.

* The _distTar_ task creates the binary distribution and packages it into a tar file that is found from the build/distributions directory.

We can create a binary distribution by running one of the following commands in the root directory of our project: _gradle distZip_ or _gradle distTar_. If we create a binary distribution that is packaged to a zip file, see the following output:

```sh
> gradle distZip
:compileJava
:processResources
:classes
:jar
:startScripts
:distZip

BUILD SUCCESSFUL

Total time: 4.679 secs
```

If we unpackage the created binary distribution created by the application plugin, we get the following directory structure:

* The _bin_ directory contains the startup scripts.

* The _lib_ directory contains the jar file of our application and its dependencies.

We can now create a binary distribution that fulfils almost all of our requirements. However, we still need to add the license of our application to the root directory of our binary distribution. Let’s move on and find out how we can do it.

### Adding the License File of Our Application to the Binary Distribution

We can add the license of our application to our binary distribution by following these steps:

* Create a task that copies the license file from the root directory of our project to the build directory.

* Add the license file to the root directory of the created binary distribution.

Let’s move on and take a closer look at these steps.

#### Copying the License File to the Build Directory

The name of the file that contains the license of our application is LICENSE, and it is found from the root directory of our project.

We can copy the license file to the build directory by following these steps:

* Create a new Copy task called the _copyLicense_.

* Configure the source file by using the _from()_ method of the CopySpec interface. Pass the string ‘LICENSE’ as a method parameter.

* Configure the target directory by using the _into()_ method of the CopySpec interface. Pass the value of the _$buildDir_ property as a method parameter.

After we have followed these steps, our _build.gradle_ file looks as follows (the relevant part is highlighted):

```xml
apply plugin: 'application'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'log4j:log4j:1.2.17'
    testCompile 'junit:junit:4.11'
}

mainClassName = 'net.petrikainulainen.gradle.HelloWorld'

task copyLicense(type: Copy) {
    from "LICENSE"
    into "$buildDir"
}
```

We have now created a task that copies the LICENSE file from the root directory of our project to the build directory. However, when we run the command _gradle distZip_ in the root directory of our project, we see the following output:

```sh
> gradle distZip
:compileJava
:processResources
:classes
:jar
:startScripts
:distZip

BUILD SUCCESSFUL

Total time: 4.679 secs
```

In other words, our new task is not invoked and this naturally means that the license file is not included in our binary distribution. Let’s fix this problem.

#### Adding the License File to the Binary Distribution

We can add the license file to the created binary distribution by following these steps:

* Transform the _copyLicense_ task from a Copy task to a “regular” Gradle task by removing the string _‘(type: Copy)’_ from its declaration.

* Modify the implementation of the _copyLicense_ task by following these steps:

  1. Configure the output of the _copyLicense_ task. Create a new File object that points to the license file found from the build directory and set it as the value of the _outputs.file_ property.

  2. Copy the license file from the root directory of our project to the build directory.

* The application plugin sets a CopySpec property called the _applicationDistribution_ to our project. We can use it to include the license file to the created binary distribution. We can do this by following these steps:

  1. Configure the location of the license file by using the _from()_ method of the CopySpec interface and pass the output of the _copyLicense_ task as method parameter.

  2. Configure the target directory by using the _into()_ method of the CopySpec interface and pass an empty String as a method parameter.

After we have followed these steps, our _build.gradle_ file looks as follows (the relevant part is highlighted):

```xml
apply plugin: 'application'
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'log4j:log4j:1.2.17'
    testCompile 'junit:junit:4.11'
}

mainClassName = 'net.petrikainulainen.gradle.HelloWorld'

task copyLicense {
    outputs.file new File("$buildDir/LICENSE")
    doLast {
        copy {
            from "LICENSE"
            into "$buildDir"
        }
    }
}

applicationDistribution.from(copyLicense) {
    into ""
}
```

When we run the command _gradle distZip_ in the root directory of our project, we see the following output:

```sh
> gradle distZip
:copyLicense
:compileJava
:processResources
:classes
:jar
:startScripts
:distZip

BUILD SUCCESSFUL

Total time: 5.594 secs
```

As we can see, the _copyLicense task_ is now invoked and if we unpackage our binary distribution, we notice that the LICENSE file is found from its root directory.

Let’s move on summarize what we have learned from this blog post.

### Summary

This blog post taught us three things:

* We learned that we can create a binary distribution by using the application plugin.

* We learned how we can copy a file from the source directory to the target directory by using the Copy task.

* We learned how we can add files to the binary distribution that is created by the application plugin.
