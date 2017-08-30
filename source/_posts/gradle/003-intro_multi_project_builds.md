---
title: Introduction to multi-project builds
date: 2017-8-30 20:13:56
tags:
categories: "Gradle"
---

Only the smallest of projects has a single build file and source tree, unless it happens to be a massive, monolithic application. It’s often much 
easier to digest and understand a project that has been split into smaller, inter-dependent modules. The word “inter-dependent” is important, though, 
and is why you typically want to link the modules together through a single build.

Gradle supports this scenario through multi-project builds.

<!--more-->

### Structure of a multi-project build
    
Such builds come in all shapes and sizes, but they do have some common characteristics:

* A **settings.gradle** file in the root or master directory of the project

* A **build.gradle** file in the root or master directory

* Child directories that have their own **\*.gradle** build files (some multi-project builds may omit child project build scripts)

The **settings.gradle** file tells Gradle how the project and subprojects are structured. Fortunately, you don’t have to read this file simply to 
learn what the project structure is as you can run the command **gradle projects**. Here's the output from using that command on the Java multiproject
 build in the Gradle samples:

Example Listing the projects in a build

Output of **gradle -q projects**

```sh
> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'multiproject'
+--- Project ':api'
+--- Project ':services'
|    +--- Project ':services:shared'
|    \--- Project ':services:webservice'
\--- Project ':shared'

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks
```

This tells you that multiproject has three immediate child projects: _api_, _services_ and _shared_. The _services_ project then has its own 
children, _shared_ and _webservice_. These map to the directory structure, so it’s easy to find them. For example, you can find _webservice_ in 
_<root>/services/webservice_.

By default, Gradle uses the name of the directory it finds the _settings.gradle_ as the name of the root project. This usually doesn't cause 
problems since all developers check out the same directory name when working on a project. On Continuous Integration servers, like Jenkins, the 
directory name may be auto-generated and not match the name in your VCS. For that reason, it's recommended that you always set the root project name 
to something predictable, even in single project builds. You can configure the root project name by setting rootProject.name.

Each project will usually have its own build file, but that's not necessarily the case. In the above example, the services project is just a container 
or grouping of other subprojects. There is no build file in the corresponding directory. However, multiproject does have one for the root project.

The root _build.gradle_ is often used to share common configuration between the child projects, for example by applying the same sets of plugins and
 dependencies to all the child projects. It can also be used to configure individual subprojects when it is preferable to have all the configuration 
 in one place. This means you should always check the root build file when discovering how a particular subproject is being configured.

Another thing to bear in mind is that the build files might not be called _build.gradle_. Many projects will name the build files after the 
subproject names, such as _api.gradle_ and _services.gradle_ from the previous example. Such an approach helps a lot in IDEs because it’s tough to 
work out which _build.gradle_ file out of twenty possibilities is the one you want to open. This little piece of magic is handled by the _settings
.gradle_ file, but as a build user you don’t need to know the details of how it’s done. Just have a look through the child project directories to 
find the files with the .gradle suffix.

Once you know what subprojects are available, the key question for a build user is how to execute the tasks within the project.

### Executing a multi-project build
    
From a user's perspective, multi-project builds are still collections of tasks you can run. The difference is that you may want to control which 
project's tasks get executed. You have two options here:

* Change to the directory corresponding to the subproject you’re interested in and just execute **gradle <task>** as normal.

* Use a qualified task name from any directory, although this is usually done from the root. For example: **gradle :services:webservice:build** will 
build the webservice subproject and any subprojects it depends on.

The first approach is similar to the single-project use case, but Gradle works slightly differently in the case of a multi-project build. The command 
gradle test will execute the test task in any subprojects, relative to the current working directory, that have that task. So if you run the command 
from the root project directory, you’ll run test in api, shared, services:shared and services:webservice. If you run the command from the services 
project directory, you’ll only execute the task in services:shared and services:webservice.

For more control over what gets executed, use qualified names (the second approach mentioned). These are paths just like directory paths, but use ‘:’ 
instead of ‘/’ or ‘\’. If the path begins with a ‘:’, then the path is resolved relative to the root project. In other words, the leading ‘:’ represents 
the root project itself. All other colons are path separators.

This approach works for any task, so if you want to know what tasks are in a particular subproject, just use the tasks task, e.g. gradle :services:webservice:tasks .

Regardless of which technique you use to execute tasks, Gradle will take care of building any subprojects that the target depends on. You don’t have to 
worry about the inter-project dependencies yourself. If you’re interested in how this is configured, you can read about writing multi-project builds 
later in the user guide.

There’s one last thing to note. When you’re using the Gradle wrapper, the first approach doesn’t work well because you have to specify the path to the 
wrapper script if you’re not in the project root. For example, if you’re in the webservice subproject directory, you would have to run ../../gradlew build.

That’s all you really need to know about multi-project builds as a build user. You can now identify whether a build is a multi-project one and you can 
discover its structure. And finally, you can execute tasks within specific subprojects.