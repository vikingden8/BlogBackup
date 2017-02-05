---
title: Jack和Jill应该在你的下一个Android应用中使用吗？
date: 2017-2-5 09:30:10
tags:
categories: "Android学习笔记"
---

  * With the release of Android N preview back in May 2016 Google also officially released a new compilation chain called Jack and Jill. Wait!!! This is not nursery rhymes. This is way more complex than those rhymes. :-)

  Google在2016年5月的时候release了Android N的预览版本，同时也release了一个新的编译链：Jack和Jill。等等，这可不是儿歌，这将比其更加复杂。

  * Jack which stands for Java Android Compiler Kit and Jill which is an acronym for Jack Intermediate Library Linker are intended to replace existing javac + dx toolchains.

  Jack是Java Android Compiler Kit的缩写，Jill是Jack Intermediate Library Linker的缩写，Jack和Jill主要是替代现有的javac+dx工具编译链。

  * Before diving into how jack and Jill works, let’s take a close look at how android compilation work till today’s date. After that, we will see how we can use Jack in your android application project.

  在深研Jack和Jill是怎样工作之前，我们先看下现有的Android编译工作是怎样的。然后，来看看在我们的Android应用程序中怎样使用Jack。

<!--more-->

### Android Compilation Process(Android编译过程):

  * Let’s look into how android compilation process works. Here, We are going to take bird eye’s view of the compilation process. I am not going to describe each and every component of the toolchain in detail.

  让我们先看下Android是怎样编译的。这里，我们将笼统的看下编译过程，我不会去描述每一个组件链的细节。

  * Since, the launch of Android OS, the compilation process works as below:

  自从Android系统诞生，其编译工作是下面这样的：

  ![](/images/categories/android/android_notes/013/old_compiler_process.png)

  * We have our java code into the .java file. When we start compiling the application source code, it compiles in two steps. First, the source code gets compiled using Java compiler (javac) and the .class file with compiled bytecode is generated. This is the common process to run java application on JVM (Java Virtual Machine). But, Android system does not use JVM to run the applications. It uses Dalvik (or ART in Android L+) which is highly optimized to for the mobile environment. Hence, the compiler has changed, the bytecode will be generated from the source code also need to modify. This is where the second step comes into the picture. In the second step, -dx tool will take .class file generated from Java Compiler as input and it modifies the bytecode into a .dex file. Now, our Dalvik (or ART) can understand and run those .dex files.

  我们将Java代码卸载.java文件中。当我们开始编译程序的源代码时，将会有两个步骤。首先，Java编译器（javac）编译.java文件，生成了对用的字节码文件.class.这个步骤和在Java虚拟机（JVM）上运行java程序是相同的处理。但是，Android系统并不使用JVM来运行程序。而是使用深度优化的移动环境Dalvik（Android L+之后是ART）来运行Android程序。这里，编译器已经改变，同样，生成的字节码文件也需修改。这也就是上图中的第二步，在这一步骤中，-dx工具把之前Java源代码生成的.class文件当作输入，然后编译生成了一个.dex文件。现在，我们的Dalvik（ART）能够解析和运行那些.dex文件了。

  * You might be thinking what about the third party libraries, which we include in our project? Those third-party libraries come into .jar or .aar format. The .jar file is nothing but the zip of .class files. So, whenever you compile those libraries in your project, -dx tool will convert them into .dex file.

  你或许在想那在我们程序的第三方libs是怎样的呢？那些第三方libs一般都是.jar或者.aar格式的文件。.jar文件不过都是.class的压缩文件而已，所以，当你在编译你的项目时，-dx工具也会将它们编译进.dex文件。

  ![](/images/categories/android/android_notes/013/2.png)

  * When we use tools that do bytecode manipulation, (like Proguard to remove unused code and obfuscate the source file or Retrolambda to use Java 8 features like lambda expression and method reference in Java 7) it takes .class file as input, modify their bytecode and generate new .class files. Later those modified class files will be converted to .dex file using -dx tool and that will produce the final result.

  当我们使用工具作字节码的组装时（比如，Proguard来移除无用的代码和混淆源代码，像Retrolambda来使用Java 8的lambda表达式，Java 7的方法引用），其会把.class文件当作输入，然后修改其字节码文件来生成新的.class文件。然后，再通过-dx工具把修改后的.class文件转换成.dex文件，如下图：

  ![](/images/categories/android/android_notes/013/2.png)

### How Jack is different from legacy javac+dx（Jack和传统的javac+dx有啥不同）?

  *　Jack compiler combines above two step (javac +dx) from the compilation toolchain into one single step. It takes .java file as an input and provides .dex files in the output. Pretty cool, huh!!! We don’t need to produce intermediate .class files. So, no more need for the .dx tool.

  Jack编译器直接组合了上面的两个步骤（javac+dx）为一个步骤，也就是它把.java源文件当作输入，输出就是.dex文件。非常酷，我们不再产生临时的.class文件，同时，我们也就不再需要-dx工具了。

  ![](/images/categories/android/android_notes/013/4.png)

  * Wait, what about the third party library? They come into .class file format. As we discussed, Jack Compiler do not take .class files as input. Here, Jill comes to the picture. Jill will process those .class files and convert them into special .jack format. Jack compiler can take .jack files as input and generate the .dex file from that.

  等等，那第三方libs怎么处理呢？它们一般都是.class格式的文件。上面我们说过，Jack编译器不是把.class当作输入的。这里，Jill就派上用场了。Jill将处理那些.class文件然后转换成特殊的.jack格式文件。Jack编译器能够将.jack文件当作输入，并且生成.dex的文件，如下图：

  ![](/images/categories/android/android_notes/013/5.png)

  * What about those tools we get addicted using like Proguard and Retrolambda? Those plugins use .class file and process them. Now with Jack toolchain don’t generate .class file during compilation.

  那怎样处理那些我们常用的工具，比如Proguard和Retrolambda？那些插件是基于.class文件处理的。现在的Jack编译器在编译过程中不再生成.class文件。

  * Proguard is baked inside the Jack toolchain. So, we can use those in our application without any worries.

  Proguard在Jack编译链的内部，所以我们无需担忧。

  * For Retrolambda, we don’t need that anymore. Jack can handle all lambdas perfectly. (And I am very excited about that.)

  对于Retrolambda，我们将不再需要了。Jack能完美处理所有的lambda表达式。

  * Of Course, you won’t be able to use other useful tools like Jacoco for now. But honestly, I don’t care about them! :-) Hope, Google will support them in the future.

  当然，你现在不能使用其他有用的工具，比如Jacoco。但是，我并不关心，希望Google将在未来支持。

### How to enable Jack toolchain in your applications（你的项目中怎样使用Jack）?

  * Enable support for the Jack toolchain in your module level build.gradle file.

  在你的module中的build.gradle文件中开启支持Jack

  ```java
  android {
      buildToolsVersion ‘21.1.2’//Use the latest version at the time.
      defaultConfig {
        // Enable the experimental Jack build tools.
         jackOptions {
           enabled true
         }
      }
      ...
  }
  ```

  * To enable support for Java 8 features add below lines in your module level build.gradle.

  为了支持Java 8的新特性，在module的build.gradle文件中配置如下几行。

  ```java
  android {
    defaultConfig {
      jackOptions {
        enabled true
      }
    }

    //Add support for java 8 features.
    compileOptions {
      sourceCompatibility JavaVersion.VERSION_1_8
      targetCompatibility JavaVersion.VERSION_1_8
    }
  }
  ```

  >If you want to enable more features like annotation processing visit [this](http://tools.android.com/tech-docs/jackandjill).
  >如果你想要更多的特性，比如注解处理，请看[这里](http://tools.android.com/tech-docs/jackandjill)

### What are we gaining（Jack优点）?

  * The biggest gain of using Jack compiler over legacy javac+dx is the ability to process Java 8 expressions. Jack supports lambda expressions, method references and typed annotations out of the box. You may argue that tools like Retrolambda also providing the support for those features. But, the overall method counts generated by Jack while compiling lambda expressions and method references in the final bytecode is far less than Retrolambda. This a huge gain as we want to keep method counts less than 64K to prevent multidexing in android.

  相对于javac+dx，使用Jack编译器最大的好处就是处理Java 8表达式。Jack支持lambda表达式，方法引用，注解类型。你可能会争论说，Retrolambda也提供支持了那些属性。但是，使用Jack来编译lambda表达式和方法引用产生的方法数要比Retrolambda要少。对于我们需要保持方法数不能超过64K是一个很大的好处。

  ![](/images/categories/android/android_notes/013/6.png)

  * Previously, toolchain uses javac. That means android compilation toolchain implementations are controlled by other groups like Oracle, Eclipse, or the OpenJDK project. Now, due to new Jack toolchain, Google has total control over the compilation toolchain, they can add some features specific to the Android compilation in the toolchain itself. (e.g. Proguard is now available in Jack toolchain by default.)

  以前，使用javac来编译源代码文件，意味着Android编译工具链的实现被其他的组织如Oracle, Eclipse,以及OpenJDK控制着。现在，由于有了Jack编译器，Google在编译上有自己的完全控制权，他们随时可以添加新的Android编译插件（比如Proguard默认已在Jack编译器内部）。

### What are the downsides（缺点）?

  * Transform APIs are not supported by the Jack compiler. So, third party plugins like Jacoco are currently not supported by Jack toolchain.

  Jack编译器暂时不支持Transform APIs。所以，第三方的插件如Jacoco当前不被Jack编译器支持。

  * Jack is currently slower than javac + dx.

  Jack当前要比javac+dx慢。

  * Lint detectors which operate on a Java bytecode level will not work since there isn’t intermediate bytecode (.class files) when you use Jack.

  Lint在Java字节码上的检测不能正常工作，因为Jack编译过程中不再生成.class文件。

  * Instant Run in android studio 2.0 and higher does not currently work with Jack and will be disabled while using the new toolchain.

  如果在你的项目中开启了Jack编译器，Instant Run将会被屏蔽。

### Conclusion（总结）:

Jack and Jill toolchain is a very cool move by Google to make android specific compilation toolchain. It will surely make a strong impact on Android development in future. But, right now it is in very early stage and it will take the time to gain popularity.

Jack和Jill工具链是Google提供的安卓系统特定的编译工具迈出的重大一步,未来在Android开发会有很大的影响.但,现在是非常早期的阶段,它将用时间来取得更高的声望.

### 资料

  * 原文地址：[The Jack and Jill: Should you use in your next Android Application?](https://blog.mindorks.com/the-jack-and-jill-should-you-use-in-your-next-android-application-ce7d0b0309b7#.qg0i6ept4)

  * [Experimental New Android Tool Chain - Jack and Jill](http://tools.android.com/tech-docs/jackandjill)
