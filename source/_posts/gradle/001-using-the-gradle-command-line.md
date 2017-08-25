---
title: Using The Gradle Command-Line
date: 2017-8-25 10:06:20
tags:
categories: "Gradle"
---

### Executing multiple tasks

You can execute multiple tasks in a single build by listing each of the tasks on the command-line. For example, the command
**gradle compile test** will execute the **compile** and **test** tasks. Gradle will execute the tasks in the order that they are listed
on the command-line, and will also execute the dependencies for each task. Each task is executed once only, regardless of how
it came to be included in the build: whether it was specified on the command-line, or as a dependency of another task, or both.
Let's look at an example.

Below four tasks are defined. Both _dist_ and _test_ depend on the _compile_ task. Running **gradle dist test** for this build script results
in the compile task being executed only once.

![](/images/categories/gradle/001/commandLineTutorialTasks.png)

<!--more-->

build.gradle

```gradle
task compile {
    doLast {
        println 'compiling source'
    }
}

task compileTest(dependsOn: compile) {
    doLast {
        println 'compiling unit tests'
    }
}

task test(dependsOn: [compile, compileTest]) {
    doLast {
        println 'running unit tests'
    }
}

task dist(dependsOn: [compile, test]) {
    doLast {
        println 'building the distribution'
    }
}
```

Output of **gradle dist test**

```sh
> gradle dist test
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL in 0s
4 actionable tasks: 4 executed
```

Each task is executed only once, so **gradle test test** is exactly the same as **gradle test**.

### Excluding tasks

You can exclude a task from being executed using the **-x** command-line option and providing the name of the task to exclude. Let's try this with 
the sample build file above.

Output of **gradle dist -x test**

```sh
> gradle dist -x test
:compile
compiling source
:dist
building the distribution

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

You can see from the output of this example, that the test task is not executed, even though it is a dependency of the _dist_ task. 
You will also notice that the _test_ task's dependencies, such as _compileTest_ are not executed either. Those dependencies of _test_ 
that are required by another task, such as _compile_, are still executed.

### Continuing the build when a failure occurs
    
By default, Gradle will abort execution and fail the build as soon as any task fails. This allows the build to complete sooner, but hides other 
failures that would have occurred. In order to discover as many failures as possible in a single build execution, you can use the **--continue** 
option.
    
When executed with **--continue**, Gradle will execute every task to be executed where all of the dependencies for that task completed without failure, 
instead of stopping as soon as the first failure is encountered. Each of the encountered failures will be reported at the end of the build.
    
If a task fails, any subsequent tasks that were depending on it will not be executed, as it is not safe to do so. For example, tests will not run if 
there is a compilation failure in the code under _test_; because the _test_ task will depend on the compilation task (either directly or indirectly).

### Task name abbreviation
    
When you specify tasks on the command-line, you don't have to provide the full name of the task. You only need to provide enough of the task name to 
uniquely identify the task. For example, in the sample build above, you can execute task dist by running **gradle d**:

Output of **gradle di**

```sh
> gradle di
:compile
compiling source
:compileTest
compiling unit tests
:test
running unit tests
:dist
building the distribution

BUILD SUCCESSFUL in 0s
4 actionable tasks: 4 executed
```

You can also abbreviate each word in a camel case task name. For example, you can execute task _compileTest_ by running **gradle compTest** or even 
**gradle cT**

Output of **gradle cT**

```sh
> gradle cT
:compile
compiling source
:compileTest
compiling unit tests

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

You can also use these abbreviations with the -x command-line option.

### Selecting which build to execute
    
When you run the gradle command, it looks for a build file in the current directory. You can use the **-b** option to select another build file. If 
you use **-b** option then **settings.gradle** file is not used. Example:

subdir/myproject.gradle

```gradle
task hello {
    doLast {
        println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
    }
}
```

Output of **gradle -q -b subdir/myproject.gradle hello**

```sh
> gradle -q -b subdir/myproject.gradle hello
using build file 'myproject.gradle' in 'subdir'.
```

Alternatively, you can use the **-p** option to specify the project directory to use. For multi-project builds you should use **-p** option instead 
of **-b** option.

Output of **gradle -q -p subdir hello**

```sh
> gradle -q -p subdir hello
using build file 'build.gradle' in 'subdir'.
```

### Forcing tasks to execute
    
Many tasks, particularly those provided by Gradle itself, support incremental builds. Such tasks can determine whether they need to run or not based 
on whether their inputs or outputs have changed since the last time they ran. You can easily identify tasks that take part in incremental build when 
Gradle displays the text _UP-TO-DATE_ next to their name during a build run.
    
You may on occasion want to force Gradle to run all the tasks, ignoring any _up-to-date_ checks. If that's the case, simply use the **--rerun-tasks** 
option. Here's the output when running a task both without and with **--rerun-tasks**:

Output of **gradle doIt**

```sh
> gradle doIt
:doIt UP-TO-DATE
```

Output of **gradle --rerun-tasks doIt**

```sh
> gradle --rerun-tasks doIt
:doIt
```

Note that this will force all required tasks to execute, not just the ones you specify on the command line. It's a little like running a clean, but 
without the build's generated output being deleted.

### Obtaining information about your build
    
Gradle provides several built-in tasks which show particular details of your build. This can be useful for understanding the structure and dependencies 
of your build, and for debugging problems.
    
In addition to the built-in tasks shown below, you can also use the project report plugin to add tasks to your project which will generate these reports.

#### Listing projects

Running gradle projects gives you a list of the sub-projects of the selected project, displayed in a hierarchy. Here is an example:

Output of **gradle -q projects**

```sh
> gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'projectReports'
+--- Project ':api' - The shared API for the application
\--- Project ':webapp' - The Web application implementation

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :api:tasks
```

The report shows the description of each project, if specified. You can provide a description for a project by setting the _description_ property:

#### Listing tasks
     
Running gradle tasks gives you a list of the main tasks of the selected project. This report shows the default tasks for the project, if any, and a 
description for each task. Below is an example of this report:

Output of **gradle -q tasks**

```sh
> gradle -q tasks

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
dists - Builds the distribution
libs - Builds the JAR

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
components - Displays the components produced by root project 'projectReports'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
properties - Displays the properties of root project 'projectReports'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>
```

By default, this report shows only those tasks which have been assigned to a task _group_, so-called visible tasks. You can do this by setting the 
group property for the task. You can also set the _description_ property, to provide a description to be included in the report.

```gradle
dists {
    description = 'Builds the distribution'
    group = 'build'
}
```

You can obtain more information in the task listing using the **--all** option. With this option, the task report lists all tasks in the project, 
including tasks which have not been assigned to a task group, so-called hidden tasks. Here is an example:

Output of **gradle -q tasks --all**

```sh
> gradle -q tasks --all

------------------------------------------------------------
All tasks runnable from root project
------------------------------------------------------------

Default tasks: dists

Build tasks
-----------
clean - Deletes the build directory (build)
api:clean - Deletes the build directory (build)
webapp:clean - Deletes the build directory (build)
dists - Builds the distribution
api:libs - Builds the JAR
webapp:libs - Builds the JAR

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'projectReports'.
api:buildEnvironment - Displays all buildscript dependencies declared in project ':api'.
webapp:buildEnvironment - Displays all buildscript dependencies declared in project ':webapp'.
components - Displays the components produced by root project 'projectReports'. [incubating]
api:components - Displays the components produced by project ':api'. [incubating]
webapp:components - Displays the components produced by project ':webapp'. [incubating]
dependencies - Displays all dependencies declared in root project 'projectReports'.
api:dependencies - Displays all dependencies declared in project ':api'.
webapp:dependencies - Displays all dependencies declared in project ':webapp'.
dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
api:dependencyInsight - Displays the insight into a specific dependency in project ':api'.
webapp:dependencyInsight - Displays the insight into a specific dependency in project ':webapp'.
dependentComponents - Displays the dependent components of components in root project 'projectReports'. [incubating]
api:dependentComponents - Displays the dependent components of components in project ':api'. [incubating]
webapp:dependentComponents - Displays the dependent components of components in project ':webapp'. [incubating]
help - Displays a help message.
api:help - Displays a help message.
webapp:help - Displays a help message.
model - Displays the configuration model of root project 'projectReports'. [incubating]
api:model - Displays the configuration model of project ':api'. [incubating]
webapp:model - Displays the configuration model of project ':webapp'. [incubating]
projects - Displays the sub-projects of root project 'projectReports'.
api:projects - Displays the sub-projects of project ':api'.
webapp:projects - Displays the sub-projects of project ':webapp'.
properties - Displays the properties of root project 'projectReports'.
api:properties - Displays the properties of project ':api'.
webapp:properties - Displays the properties of project ':webapp'.
tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
api:tasks - Displays the tasks runnable from project ':api'.
webapp:tasks - Displays the tasks runnable from project ':webapp'.

Other tasks
-----------
api:compile - Compiles the source files
webapp:compile - Compiles the source files
docs - Builds the documentation
```

#### Show task usage details

Running **gradle help --task someTask** gives you detailed information about a specific task or multiple tasks matching the given task name in your 
multi-project build. Below is an example of this detailed information:

Output of **gradle -q help --task libs**

```sh
> gradle -q help --task libs
Detailed task information for libs

Paths
     :api:libs
     :webapp:libs

Type
     Task (org.gradle.api.Task)

Description
     Builds the JAR

Group
     build
```

This information includes the full task path, the task type, possible commandline options and the description of the given task.

#### Listing project dependencies

Running gradle dependencies gives you a list of the dependencies of the selected project, broken down by configuration. For each configuration, the 
direct and transitive dependencies of that configuration are shown in a tree. Below is an example of this report:

Output of **gradle -q dependencies api:dependencies webapp:dependencies**

```sh
> gradle -q dependencies api:dependencies webapp:dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

No configurations

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

compile
\--- org.codehaus.groovy:groovy-all:2.4.10

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3

------------------------------------------------------------
Project :webapp - The Web application implementation
------------------------------------------------------------

compile
+--- project :api
|    \--- org.codehaus.groovy:groovy-all:2.4.10
\--- commons-io:commons-io:1.2

testCompile
No dependencies
```

Since a dependency report can get large, it can be useful to restrict the report to a particular configuration. This is achieved with the optional 
**--configuration** parameter:

Output of **gradle -q api:dependencies --configuration testCompile**

```sh
> gradle -q api:dependencies --configuration testCompile

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

testCompile
\--- junit:junit:4.12
     \--- org.hamcrest:hamcrest-core:1.3
```

#### Listing project properties

Running **gradle properties** gives you a list of the properties of the selected project. This is a snippet from the output:

Output of **gradle -q api:properties**

```sh
> gradle -q api:properties

------------------------------------------------------------
Project :api - The shared API for the application
------------------------------------------------------------

allprojects: [project ':api']
ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@12345
asDynamicObject: DynamicObject for project ':api'
baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle
```

#### Profiling a build

The **--profile** command line option will record some useful timing information while your build is running and write a report to the 
**build/reports/profile** directory. The report will be named using the time when the build was run.
     
This report lists summary times and details for both the configuration phase and task execution. The times for configuration and task execution are 
sorted with the most expensive operations first. The task execution results also indicate if any tasks were skipped (and the reason) or if tasks that 
were not skipped did no work.
     
Builds which utilize a buildSrc directory will generate a second profile report for buildSrc in the **buildSrc/build** directory.

![](/images/categories/gradle/001/profile.png)

### Dry Run
    
Sometimes you are interested in which tasks are executed in which order for a given set of tasks specified on the command line, but you don't want the
tasks to be executed. You can use the **-m** option for this. For example, if you run **“gradle -m clean compile”**, you'll see all the tasks that 
would be executed as part of the **clean** and **compile** tasks. This is complementary to the **tasks** task, which shows you the tasks which are 
available for execution.
     
     