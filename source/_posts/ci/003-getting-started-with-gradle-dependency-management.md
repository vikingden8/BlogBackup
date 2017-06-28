---
title: Getting Started With Gradle-Dependency Management
date: 2017-06-28 20:55:49
tags:
categories: "持续构建"
---

It is challenging, if not impossible, to create real life applications which don’t have any external dependencies. That is why dependency management is a vital part of every software project.

This blog post describes how we can manage the dependencies of our projects with Gradle. We will learn to configure the used repositories and the required dependencies. We will also apply this theory to practice by implementing a simple example application.

Let’s get started.

### Introduction to Repository Management

Repositories are essentially dependency containers, and each project can use zero or more repositories.

Gradle supports the following repository formats:

* Ivy repositories

* Maven repositories

* Flat directory repositories

Let’s find out how we can configure each repository type in our build.

#### Adding Ivy Repositories to Our Build

We can add an Ivy repository to our build by using its url address or its location in the local file system.

If we want to add an Ivy repository by using its url address, we have to add the following code snippet to the _build.gradle_ file:

```xml
repositories {
    ivy {
        url 'http://ivy.petrikainulainen.net/repo'
    }
}
```

<!--more-->

If we want to add an Ivy repository by using its location in the file system, we have to add the following code snippet to the build.gradle file:

```xml
repositories {
    ivy {       
        url '../ivy-repo'
    }
}
```

Let’s move on and find out how we can add Maven repositories to our build.

#### Adding Maven Repositories to Our Build

We can add a Maven repository to our build by using its url address or its location in the local file system.

If we want to add a Maven repository by using its url, we have to add the following code snippet to the _build.gradle_ file:

```xml
repositories {
    maven {
        url 'http://maven.petrikainulainen.net/repo'
    }
}
```

If we want to add a Maven repository by using its location in the file system, we have to add the following code snippet to the _build.gradle_ file:

```xml
repositories {
    maven {       
        url '../maven-repo'
    }
}
```

Gradle has three “aliases” which we can use when we are adding Maven repositories to our build. These aliases are:

* The _mavenCentral()_ alias means that dependencies are fetched from the central Maven repository.

* The _jcenter()_ alias means that dependencies are fetched from the Bintray’s JCenter Maven repository.

* The _mavenLocal()_ alias means that dependencies are fetched from the local Maven repository.

If we want to add the central Maven repository in our build, we must add the following snippet to our build.gradle file:

```xml
repositories {
    mavenCentral()
}
```

Let’s move on and find out how we can add flat directory repositories to our build.

#### Adding Flat Directory Repositories to Our Build

If we want to use flat directory repositories, we have to add the following code snippet to our _build.gradle_ file:

```xml
repositories {
    flatDir {
        dirs 'lib'
    }
}
```

This means that dependencies are searched from the lib directory. Also, if we want to, we can use multiple directories by adding the following snippet to the _build.gradle_ file:

```xml
repositories {
    flatDir {
        dirs 'libA', 'libB'
    }
}
```

Let’s move on and find out how we can manage the dependencies of our project with Gradle.

### Introduction to Dependency Management

After we have configured the repositories of our project, we can declare its dependencies. If we want to declare a new dependency, we have to follow these steps:

* Specify the configuration of the dependency.

* Declare the required dependency.

Let’s take a closer look at these steps.

#### Grouping Dependencies into Configurations

In Gradle dependencies are grouped into a named set of dependencies. These groups are called configurations, and we use them to declare the external dependencies of our project.

The Java plugin specifies several dependency configurations which are described in the following:

* The dependencies added to the **compile** configuration are required when our the source code of our project is compiled.

* The **runtime** configuration contains the dependencies which are required at runtime. This configuration contains the dependencies added to the compile configuration.

* The **testCompile** configuration contains the dependencies which are required to compile the tests of our project. This configuration contains the compiled classes of our project and the dependencies added to the compile configuration.

* The **testRuntime** configuration contains the dependencies which are required when our tests are run. This configurations contains the dependencies added to compile, runtime, and testCompile configurations.

* The **archives** configuration contains the artifacts (e.g. Jar files) produced by our project.

* The **default** configuration group contains the dependencies which are required at runtime.

Let’s move on and find out how we can declare the dependencies of our Gradle project.

#### Declaring the Dependencies of a Project

The most common dependencies are called external dependencies which are found from an external repository. An external dependency is identified by using the following attributes:

* The **group** attribute identifies the group of the dependency (Maven users know this attribute as **groupId**).

* The **name** attribute identifies the name of the dependency (Maven users know this attribute as **artifactId**).

* The **version** attribute specifies the version of the external dependency (Maven users know this attribute as **version**).

>These attributes are required when you use Maven repositories. If you use other repositories, some attributes might be optional.
For example, if you use a flat directory repository, you might have to specify only **name** and **version**.

Let’s assume that we have to declare the following dependency:

* The group of the dependency is _‘foo’_.

* The name of the dependency is _‘foo’_.

* The version of the dependency is _0.1_.

* The dependency is required when our project is _compiled_.

We can declare this dependency by adding the following code snipped to the _build.gradle_ file:

```xml
dependencies {
    compile group: 'foo', name: 'foo', version: '0.1'
}
```

We can also declare the dependencies of our project by using a shortcut form which follows this syntax: [_group_]:[_name_]:[_version_]. If we want to use the shortcut form, we have to add the following code snippet to the _build.gradle_ file:

```xml
dependencies {
    compile 'foo:foo:0.1'
}
```

We can also add multiple dependencies to the same configuration. If we want to use the “normal” syntax when we declare our dependencies, we have to add the following code snippet to the _build.gradle_ file:

```xml
dependencies {
    compile (
        [group: 'foo', name: 'foo', version: '0.1'],
        [group: 'bar', name: 'bar', version: '0.1']
    )
}
```

On the other hand, if we want to use the shortcut form, the relevant part of the build.gradle file looks as follows:

```xml
dependencies {
    compile 'foo:foo:0.1', 'bar:bar:0.1'
}
```

It is naturally possible to declare dependencies which belong to different configurations. For example, if we want to declare dependencies which belong to the _compile_ and _testCompile_ configurations, we have to add the following code snippet to the _build.gradle_ file:

```xml
dependencies {
    compile group: 'foo', name: 'foo', version: '0.1'
    testCompile group: 'test', name: 'test', version: '0.1'
}
```

Again, it is possible to use the shortcut form. If we want to declare the same dependencies by using the shortcut form, the relevant part of the _build.gradle_ file looks as follows:

```xml
dependencies {
    compile 'foo:foo:0.1'
    testCompile 'test:test:0.1'
}
```

We have now learned the basics of dependency management. Let’s move on and implement our example application.

### Creating the Example Application

The requirements of our example application are described in thefollowing:

* The build script of the example application must use the Maven central repository.

* The example application must write the received message to log by using Log4j.

* The example application must contain unit tests which ensure that the correct message is returned. These unit tests must be written by using JUnit.

* Our build script must create an executable jar file.

Let’s find out how we can fulfil these requirements.

#### Configuring the Repositories of Our Build

One of the requirements of our example application was that its build script must use the Maven central repository. After we have configured our build script to use the Maven central repository, its source code looks as follows (The relevant part is highlighted):

```xml
apply plugin: 'java'

repositories {
    mavenCentral()
}

jar {
    manifest {
        attributes 'Main-Class': 'com.viking.gradle.HelloWorld'
    }
}
```

Let’s move on and declare the dependencies of our example application.

#### Declaring the Dependencies of Our Example Application

We have to declare two dependencies in the build.gradle file:

* **Log4j** (version 1.2.17) is used to write the received message to the log.

* **JUnit** (version 4.11) is used to write unit tests for our example application.

After we have declared these dependencies, the _build.gradle_ file looks as follows (the relevant part is highlighted):

```xml
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'log4j:log4j:1.2.17'
    testCompile 'junit:junit:4.11'
}

jar {
    manifest {
        attributes 'Main-Class': 'net.petrikainulainen.gradle.HelloWorld'
    }
}
```

Let’s move on and write some code.

#### Writing the Code

In order to fulfil the requirements of our example application, “we have to over-engineer it”. We can create the example application by following these steps:

* Create a _MessageService_ class which returns the string _‘Hello World!’_ when its _getMessage()_ method is called.

* Create a _MessageServiceTest_ class which ensures that the _getMessage()_ method of the _MessageService_ class returns the string _‘Hello World!’_.

* Create the main class of our application which obtains the message from a _MessageService_ object and writes the message to log by using _Log4j_.

* Configure _Log4j_.

Let’s go through these steps one by one.

**First**, we have to create a _MessageService_ class to the _src/main/java/com/viking/gradle_ directory and implement it. After we have do this, its source code looks as follows:

```java
package com.viking.gradle;

public class MessageService {

    public String getMessage() {
        return "Hello World!";
    }
}
```

**Second**, we have create a _MessageServiceTest_ to the _src/test/java/com/viking/gradle_ directory and write a unit test to the _getMessage()_ method of the _MessageService_ class. The source code of the _MessageServiceTest_ class looks as follows:

```java
package com.viking.gradle;

import org.junit.Before;
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class MessageServiceTest {

    private MessageService messageService;

    @Before
    public void setUp() {
        messageService = new MessageService();
    }

    @Test
    public void getMessage_ShouldReturnMessage() {
        assertEquals("Hello World!", messageService.getMessage());
    }
}
```

**Third**, we have create a _HelloWorld_ class to the _src/main/java/com/viking/gradle_ directory. This class is the main class of our application. It obtains the message from a _MessageService_ object and writes it to a log by using _Log4j_. The source code of the _HelloWorld_ class looks as follows:

```java
package com.viking.gradle;

import org.apache.log4j.Logger;

public class HelloWorld {

    private static final Logger LOGGER = Logger.getLogger(HelloWorld.class);

    public static void main(String[] args) {
        MessageService messageService = new MessageService();

        String message = messageService.getMessage();
        LOGGER.info("Received message: "+ message);
    }
}
```

**Fourth**, we have to configure _Log4j_ by using the _log4j.properties_ which is found from the _src/main/resources_ directory. The _log4j.properties_ file looks as follows:

```xml
log4j.appender.Stdout=org.apache.log4j.ConsoleAppender
log4j.appender.Stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.Stdout.layout.conversionPattern=%-5p - %-26.26c{1} - %m\n

log4j.rootLogger=DEBUG,Stdout
```

That is it. Let’s find out how we can run the tests of our example application.

###  Running the Unit Tests
We can run our unit test by using the following command:

```sh
gradle test
```

When our test passes, we see the following output:

```sh
>gradle test
:compileJava
:processResources
:classes
:compileTestJava
:processTestResources
:testClasses
:test

BUILD SUCCESSFUL

Total time: 4.678 secs
```

However, if our unit test would fail, we would see the following output (the interesting section is highlighted):

```sh
gradle test
:compileJava
:processResources
:classes
:compileTestJava
:processTestResources
:testClasses
:test

com.viking.gradle.MessageServiceTest &gt; getMessage_ShouldReturnMessageFAILED
    org.junit.ComparisonFailure at MessageServiceTest.java:22

1 test completed, 1 failed
:test FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':test'.
&gt; There were failing tests. See the report at: file:///Users/loke/Projects/Java/Blog/gradle-examples/dependency-management/build/reports/tests/index.html

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 4.461 secs
```

As we can see, if our unit tests fails, the describes:

* which tests failed.

* how many tests were run and how many tests failed.

* the location of the test report which provides additional information about the failed (and passed) tests.

When we run our unit tests, Gradle creates test reports to the following directories:

* The _build/test-results_ directory contains the raw data of each test run.

* The _build/reports/tests_ directory contains a HTML report which describes the results of our tests.

The HTML test report is very useful tool because it describes the reason why our test failed. For example, if our unit test would expect that the _getMessage()_ method of the _MessageService_ class returns the string ‘Hello Worl1d!’, the HTML test report of that test case would look as follows:

![](/images/categories/ci/003/testfailure-1024x493.png)

Let’s move on and find out how we can package and run our example application.

### Packaging and Running Our Example Application

We can package our application by using one of these commands: _gradle assembly_ or _gradle build_. Both of these commands create the _dependency-management.jar_ file to the _build/libs_ directory.

When run our example application by using the command _java -jar dependency-management.jar_, we see the following output:

```sh
>java -jar dependency-management.jar

Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/log4j/Logger
    at com.viking.gradle.HelloWorld.<clinit>(HelloWorld.java:10)
Caused by: java.lang.ClassNotFoundException: org.apache.log4j.Logger
    at java.net.URLClassLoader$1.run(URLClassLoader.java:372)
    at java.net.URLClassLoader$1.run(URLClassLoader.java:361)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:360)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
    ... 1 more
```

The reason for this exception is that the Log4j dependency isn’t found from the classpath when we run our application.

The easiest way to solve this problem is to create a so called “fat” jar file. This means that we will package the required dependencies to the created jar file.

After we have followed the instructions given in this blog post, our build script looks as follows (the relevant part is highlighted):

```xml
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile 'log4j:log4j:1.2.17'
    testCompile 'junit:junit:4.11'
}

jar {
    from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
    manifest {
        attributes 'Main-Class': 'com.viking.gradle.HelloWorld'
    }
}
```

We can now run the example application (after we have packaged it) and as we can see, everything is working properly:

```sh
>java -jar dependency-management.jar
INFO  - HelloWorld                 - Received message: Hello World!
```

That is all for today. Let’s summarize what we learned from this blog post.

### Summary

This blog post has taught us four things:

* We learned how we can configure the repositories used by our build.

* We learned how we can declare the required dependencies and group these dependencies into configurations.

* We learned that Gradle creates a HTML test report when our tests are run.

* We learned how we can create a so called “fat” jar file.

From : [Getting Started With Gradle: Dependency Management](https://www.petrikainulainen.net/programming/gradle/getting-started-with-gradle-dependency-management/)
