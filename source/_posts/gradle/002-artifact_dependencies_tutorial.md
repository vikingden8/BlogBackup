---
title: Dependency Management Basics
date: 2017-8-29 18:45:10
tags:
categories: "Gradle"
---

### What is dependency management?
    
Very roughly, dependency management is made up of two pieces. Firstly, Gradle needs to know about the things that your project needs to build or run, 
in order to find them. We call these incoming files the dependencies of the project. Secondly, Gradle needs to build and upload the things that your 
project produces. We call these outgoing files the publications of the project. Let’s look at these two pieces in more detail:

大概来说，依赖管理由两方面构成。一方面，Gradle需要知道在构建您的项目时所需要构建或者运行的东西以便Gradle找到它们，我们将这些被导入的文件称作项目的依赖。另一方面，Gradle
需要构建或者上传您的项目产出的东西，我们将这些由您的项目产出的文件称作项目的输出。接下来让我们更详细的看看这两方面内容：
    
Most projects are not completely self-contained. They need files built by other projects in order to be compiled or tested and so on. For example, in 
order to use Hibernate in my project, I need to include some Hibernate jars in the classpath when I compile my source. To run my tests, I might also 
need to include some additional jars in the test classpath, such as a particular JDBC driver or the Ehcache jars.

大部分项目并不是依靠项目本身独立完成的。它们需要加入其他项目构建的文件以便进行编译或测试等等。举个例子，为了在我的项目中使用Hibernate，在编译我的源码时我需要在classpath
中引入一些Hibernate的jar文件。为了运行我的测试项目，我可能也需要在测试项目的classpath中引入一些额外的jar文件，例如一个特别的JDBC驱动包或者Ehcache框架包。
    
These incoming files form the dependencies of the project. Gradle allows you to tell it what the dependencies of your project are, so that it can take 
care of finding these dependencies, and making them available in your build. The dependencies might need to be downloaded from a remote Maven or Ivy 
repository, or located in a local directory, or may need to be built by another project in the same multi-project build. We call this process dependency 
resolution.

对于这些由项目依赖导入的文件，Gradle允许您告诉它您的项目都依赖了那些文件以便Gradle能够负责找到这些依赖文件，然后让这些文件在您的构建项目中可用。这些依赖文件可能需要从远程
的Maven或者Ivy仓库下载，或者存在于本地目录中，或者在一些多项目构建中由其他项目构建而来。我们将这个过程称作<依赖解析>。

<!--more-->

Note that this feature provides a major advantage over Ant. With Ant, you only have the ability to specify absolute or relative paths to specific jars 
to load. With Gradle, you simply declare the “names” of your dependencies, and other layers determine where to get those dependencies from. You can get 
similar behavior from Ant by adding Apache Ivy, but Gradle does it better.

值得注意的是该特性（依赖解析）是Gradle相对于Ant最主要的优势。通过Ant，您只能使用绝对或相对路径来定义要加载的jar文件。而通过Gradle，您只需要简单的声明依赖文件的名称以及其
他层次结构就可以确定从哪里获取这些依赖文件。当然您也可以通过添加Apache Ivy来获取类似Ant的行为操作，但是Gradle能做得更好。
    
Often, the dependencies of a project will themselves have dependencies. For example, Hibernate core requires several other libraries to be present on 
the classpath with it runs. So, when Gradle runs the tests for your project, it also needs to find these dependencies and make them available. We call 
these transitive dependencies.

很多情况下，项目的依赖文件也会有它们自己的依赖文件。举个例子，Hibernate核心的运行需要在classpath中声明几个其他库文件。因此，当Gradle构建您的项目时，也需要找到这些被依赖
文件所依赖的文件以确保它们可用。我们将这种情况称作<可传递性的依赖>。
    
The main purpose of most projects is to build some files that are to be used outside the project. For example, if your project produces a Java library, 
you need to build a jar, and maybe a source jar and some documentation, and publish them somewhere.

大多数项目的主要目的都是构建出一个可以在项目之外运行的可执行文件。举例来说，如果你的项目能够产出一个Java库文件，那么你需要构建出一个jar文件，或者有可能是一个jar文件外加一
些文档，并且在某个地方将他们发布。
    
These outgoing files form the publications of the project. Gradle also takes care of this important work for you. You declare the publications of your 
project, and Gradle take care of building them and publishing them somewhere. Exactly what “publishing” means depends on what you want to do. You might 
want to copy the files to a local directory, or upload them to a remote Maven or Ivy repository. Or you might use the files in another project in the 
same multi-project build. We call this process publication.

对于这些包含在您的项目最终产物里的文件。Gradle也负责帮您处理这项重要工作。您只需要声明项目的产物，然后Gradle将会负责构建它们并且在某些地方发布他们。至于具体发布哪些内容
完全取决于您个人。您可能想要把这些文件拷贝到本地目录下，或者将它们上传到远程的Maven或者Ivy仓库里。或者在一些多项目构建活动中您可能希望某一个项目使用其他项目的产物。我们
将这个过程称作<发布>。

### Declaring your dependencies
    
Let’s look at some dependency declarations. Here’s a basic build script:

让我们来看看一些依赖关系的声明。这里有一个基础的构建脚本：

Example Declaring dependencies

```sh
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

What’s going on here? This build script says a few things about the project. Firstly, it states that Hibernate core 3.6.7.Final is required to 
compile the project’s production source. By implication, Hibernate core and its dependencies are also required at runtime. The build script also states 
that any junit >= 4.0 is required to compile the project’s tests. It also tells Gradle to look in the Maven central repository for any dependencies 
that are required. The following sections go into the details.

在上面的脚本中都发生了些什么？这个构建脚本描述了关于项目的少许内容。首先，它规定了编译项目时需要的Hibernate核心版本为 3.6.7.Final。同时这也暗示着在运行项目时，Hibernate
核心所依赖的其他文件也会被包含进来。构建脚本也规定了在编译测试项目时所需要的junit版本必须大于等于4.0。它还告诉Gradle从Maven central仓库中搜索任何需要的依赖文件。在接下
来的部分里将会有更详细的描述。

### Dependency configurations
    
A Configuration is a named set of dependencies and artifacts. There are three main purposes for a Configuration:

* Declaring Dependencies

The plugin uses configurations to make it easy for build authors to declare what other subprojects or external artifacts are needed for various purposes 
during the execution of tasks defined by the plugin.

* Resolving Dependencies

The plugin uses configurations to find (and possibly download) inputs to to the tasks it defines.

* Exposing Artifacts for Consumption

The plugin uses configurations to define what artifacts it generates for other projects to consume.

With those three purposes in mind, let’s take a look at a few of the standard configurations defined by the Java Library Plugin. You can find more 
details in Section 48.4, “The Java Library plugin configurations”.

* implementation

The dependencies required to compile the production source of the project, but which are not part of the api exposed by the project. This configuration 
is an example of a configuration used for Declaring Dependencies.

* runtimeClasspath

The dependencies required by the production classes at runtime. By default, this includes the dependencies declared in the api, implementation, and 
runtimeOnly configurations. This configuration is an example of a configuration used for Resolving Dependencies, and as such, users should never 
declare dependencies directly in the runtimeClasspath configuration.

* apiElements

The dependencies which are part of this project’s externally consumable API as well as the classes which are defined in this project which should be 
consumable by other projects. This configuration is an example of Exposing Artifacts for Consumption.

Various plugins add further standard configurations. You can also define your own custom configurations to use in your build. 

### External dependencies
    
There are various types of dependencies that you can declare. One such type is an external dependency. This is a dependency on some files built 
outside the current build, and stored in a repository of some kind, such as Maven central, or a corporate Maven or Ivy repository, or a directory in 
the local file system.

Gradle中有多种不同类型的依赖可以供你声明。其中一种就是外部依赖。这是一种依赖于不在当前构建项目中的文件的依赖，并且这些文件被存储在某种类型的仓库中，比如Maven central，
或者一个Maven或Ivy仓库的组合，或者一个本地文件系统的目录中。

To define an external dependency, you add it to a dependency configuration:

要定义一个外部依赖，你需要将它添加到一个依赖配置项中。

Example Definition of an external dependency

```sh
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```

An external dependency is identified using _group_, _name_ and _version_ attributes. Depending on which kind of repository you are using, group and 
version may be optional.

一个外部依赖由group，name和version三个属性构成。是否使用group，name和version这三个属性完全取决于你使用的是何种类型的仓库。

The shortcut form for declaring external dependencies looks like _“group:name:version”_.

你可以使用快捷方式声明外部依赖，就像这样“group:name:version”。

Example Shortcut definition of an external dependency

```sh
dependencies {
    compile 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

To find out more about defining and working with dependencies, have a look at [Section 25.4, “How to declare your dependencies”](https://docs.gradle.org/current/userguide/dependency_management.html#sec:how_to_declare_your_dependencies).

想要了解更多有关定义和使用依赖的内容，你可以查阅[Section 25.4, “How to declare your dependencies”](https://docs.gradle.org/current/userguide/dependency_management.html#sec:how_to_declare_your_dependencies).

### Repositories
    
How does Gradle find the files for external dependencies? Gradle looks for them in a repository. A repository is really just a collection of files, 
organized by group, name and version. Gradle understands several different repository formats, such as Maven and Ivy, and several different ways of 
accessing the repository, such as using the local file system or HTTP.

Gradle使如何找到外部依赖的文件呢？答案是Gradle在仓库里找到这些文件。一个仓库就是一些文件的集合，这些文件通过group，name和version三个属性组织在一起。Gradle知道好几种
不同的仓库形式，比如Maven和Ivy，并且它知道好几种使用这些仓库的方式，比如使用本地文件系统或者HTTP协议。

By default, Gradle does not define any repositories. You need to define at least one before you can use external dependencies. One option is use the 
Maven central repository:

默认情况下，Gradle并没有定义任何的仓库。在使用外部依赖之前你需要至少定义一个仓库。Maven central仓库是一个选择：

Example Usage of Maven central repository

```sh
repositories {
    mavenCentral()
}
```

Or Bintray’s JCenter:

或者Bintray的JCenter仓库：

Example Usage of JCenter repository

```sh
repositories {
    jcenter()
}
```

Or a any other remote Maven repository:

或者一个远程Maven仓库：

Example Usage of a remote Maven repository

```sh
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

Or a remote Ivy repository:

或者一个远程Ivy仓库：

Example Usage of a remote Ivy directory

```sh
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

You can also have repositories on the local file system. This works for both Maven and Ivy repositories.

你也可以使用存在于本地文件系统中的仓库。这种方式对Maven和Ivy都是有效的。

Example Usage of a local Ivy directory

```sh
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

A project can have multiple repositories. Gradle will look for a dependency in each repository in the order they are specified, stopping at the first 
repository that contains the requested module.

一个项目可以有多个仓库。Gradle将会根据仓库的声明顺序逐个检索每一个仓库，当发现需要的模块之后就停止检索。

### Publishing artifacts
    
Dependency configurations are also used to publish files.We call these files publication artifacts, or usually just artifacts.

依赖配置项也用来发布文件。我们称之为publication artifacts，或者常常简单称为artifacts。

The plugins do a pretty good job of defining the artifacts of a project, so you usually don’t need to do anything special to tell Gradle what needs 
to be published. However, you do need to tell Gradle where to publish the artifacts. You do this by attaching repositories to the uploadArchives task. 
Here’s an example of publishing to a remote Ivy repository:

插件在定义项目的产出物这项工作上已经做得很好了，所以你通常不需要做任何特别的事情来告诉Gradle哪些东西需要被发布。然而，你需要告诉Gradle项目的产出物要发布到哪里。你通过将仓
库依附到uploadArchives任务上来实现发布工作。接下来有一个关于发布产出物到一个远程Ivy仓库的示例：

Example Publishing to an Ivy repository

```sh
uploadArchives {
    repositories {
        ivy {
            credentials {
                username "username"
                password "pw"
            }
            url "http://repo.mycompany.com"
        }
    }
}
```

Now, when you run **gradle uploadArchives**, Gradle will build and upload your Jar. Gradle will also generate and upload an ivy.xml as well.

当你运行 **gradle uploadArchives** 命令时，Gradle将会构建和上传你的jar文件。同时Gradle也会生成并上传一个ivy.xml配置文件。

You can also publish to Maven repositories. The syntax is slightly different. Note that you also need to apply the Maven plugin in order to publish
 to a Maven repository. when this is in place, Gradle will generate and upload a pom.xml.
 
你也可以发布artifacts到Maven仓库。在语法上面将会稍有不同。需要注意的是你同样需要定义Maven插件以便发布artifacts到一个Maven仓库。当一切准备就绪后，Gradle会生成并上传
pom.xml配置文件。

Example Publishing to a Maven repository

```sh
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

