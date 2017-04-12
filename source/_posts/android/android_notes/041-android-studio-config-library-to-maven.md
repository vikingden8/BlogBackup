---
title: Android Studio配置Library项目到内部maven服务器
date: 2017-4-12 11:20:56
tags:
categories: "Android学习笔记"
---

### 配置maven上传插件设置

在项目根目录的gradle.properties文件中配置如下的信息：

![](/images/categories/android/android_notes/041/1.png)

然后在公共库module中的build.gradle中配置如下task：

```gradle
apply plugin: 'maven'

def isReleaseBuild() {
    return VERSION_NAME.contains("SNAPSHOT") == false
}
def getRepositoryUsername() {
    return hasProperty('NEXUS_USERNAME') ? NEXUS_USERNAME : ""
}
def getRepositoryPassword() {
    return hasProperty('NEXUS_PASSWORD') ? NEXUS_PASSWORD : ""
}
afterEvaluate { project ->
    uploadArchives {
        repositories {
            mavenDeployer {
                pom.groupId = GROUP
                pom.artifactId = ARTIFACT_ID
                pom.version = VERSION_NAME
                repository(url: RELEASE_REPOSITORY_URL) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }
                snapshotRepository(url: SNAPSHOT_REPOSITORY_URL) {
                    authentication(userName: getRepositoryUsername(), password: getRepositoryPassword())
                }
            }
        }
    }
    task androidJavadocs(type: Javadoc) {
        source = android.sourceSets.main.java.srcDirs
        classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    }
/*    task androidJavadocsJar(type: Jar, dependsOn: androidJavadocs) {
        classifier = 'javadoc'
        from androidJavadocs.destinationDir
    }*/
    task androidSourcesJar(type: Jar) {
        classifier = 'sources'
        from android.sourceSets.main.java.sourceFiles
    }
    artifacts {
        archives androidSourcesJar
//        archives androidJavadocsJar
    }
}
```

<!--more-->

配置好了之后，执行如下的命令：

![](/images/categories/android/android_notes/041/2.png)

上传成功后，可以看到如下的信息：

![](/images/categories/android/android_notes/041/3.png)

### 公共库的使用

在新建的项目中，如果要使用上面上传的library，可以通过使用maven中央仓库的方式，如果是内部的maven nexus服务器，但是需添加如下的配置：

  * 在项目根目录下的build.gradle中配置内部的nexus服务器地址：

  ![](/images/categories/android/android_notes/041/4.png)

  * 在需使用这个公共库module下的build.gradle添加groupId，artifactId，version即可：

  ![](/images/categories/android/android_notes/041/5.png)
