---
title: Java Internationalization
date: 2017-12-10 10:07:02
tags:
categories: "Java学习笔记"
---

翻译自：[i18n](https://docs.oracle.com/javase/tutorial/i18n/intro/index.html)

本篇内容主要介绍Java应用程序的国际化。

### Introduction

Internationalization is the process of designing an application so that it can be adapted to various languages and regions 
without engineering changes. Sometimes the term internationalization is abbreviated as i18n, because there are 18 letters 
between the first "i" and the last "n."

国际化是在设计程序时能适应多种语言和地区。有时候单词internationalization简称为i18n，是因为在第一个字母i和最后一个字母n中间还有18个字母。

An internationalized program has the following characteristics:

一个国际化的程序有以下的特性：

* With the addition of localized data, the same executable can run worldwide.

* 随着新的本地化数据增加，能同样地在世界运行。

* Textual elements, such as status messages and the GUI component labels, are not hardcoded in the program. Instead they 
are stored outside the source code and retrieved dynamically.

* 字符元素，必须状态信息和GUI组件的标签，没有硬编码进程序中。相反地，他们存放在程序源代码外部，能被动态的访问。

* Support for new languages does not require recompilation.

* 支持新的语言时不需要重新编译。

* Culturally-dependent data, such as dates and currencies, appear in formats that conform to the end user's region and language.

* 文化相关的数据，比如时间和汇率，显示的格式是根据终端用户的地区和语言决定的。

* It can be localized quickly.

* 能快速本地化

<!--more-->

Localization is the process of adapting software for a specific region or language by adding locale-specific components 
and translating text. The term localization is often abbreviated as l10n, because there are 10 letters between the "l" and the "n."

本地化是程序软件针对特定的地区或语言添加地区相关的组件和文本翻译。这个词localization简称为l10n，是因为在l和n中间还有10个字母。

The primary task of localization is translating the user interface elements and documentation. Localization involves not 
only changing the language interaction, but also other relevant changes such as display of numbers, dates, currency, and 
so on. Other types of data, such as sounds and images, may require localization if they are culturally sensitive. The 
better internationalized an application is, the easier it is to localize it for a particular language and character encoding scheme.

本地化的首要任务是用户界面和文档的翻译。本地化解决的不仅是语言交互的改变，像数字，时间，汇率等其他相关的内容显示也同样地变化。还有其他的数据，比如
声音和图像，如果它们是跟文化敏感的可能也需要本地化。最好的国际化应用程序就是对一个特殊的语言和字符编码很容易本地化。

Internationalization may seem a bit daunting at first. Reading the following sections will help ease you into the subject.

国际化在开始时可能看起来比较令人畏惧。当你读完下面的章节可能会让你觉得其实很容易。

### A Quick Example 简单示例

If you're new to internationalizing software, this lesson is for you. This lesson uses a simple example to demonstrate 
how to internationalize a program so that it displays text messages in the appropriate language. You'll learn how Locale 
and ResourceBundle objects work together and how to use properties files.

如果你对国际化软件是新手的话，那么这篇文章适合你。这篇主要是介绍在一个简单的国际化程序中根据语言显示文本信息。你将学习Local和ResourceBundle结合
使用。

#### Before Internationalization

Suppose that you've written a program that displays three messages, as follows:

试想你已经写了一个程序来显示三个信息，如下：

```java 
public class NotI18N {

    static public void main(String[] args) {

        System.out.println("Hello.");
        System.out.println("How are you?");
        System.out.println("Goodbye.");
    }
}
```

You've decided that this program needs to display these same messages for people living in France and Germany. Unfortunately 
your programming staff is not multilingual, so you'll need help translating the messages into French and German. Since the 
translators aren't programmers, you'll have to move the messages out of the source code and into text files that the 
translators can edit. Also, the program must be flexible enough so that it can display the messages in other languages, 
but right now no one knows what those languages will be.

现在你决定让你的程序在法国和德国生活的人们看到显示同样内容的信息。不幸地是，你得程序员不是使用多语言的，所以你将需要帮助他们把文本内容翻译成法语和德语。
因为翻译员不是程序员，所以你必须把那些文本内容在源代码中移出来放到文本文件中。同样地，程序也必须对显示其它的语言信息灵活适应，只是现在没有知道那些
语言到底是什么。

It looks like the program needs to be internationalized.

这使得你的程序看起来需要国际化支持。

#### After Internationalization

The source code for the internationalized program follows. Notice that the text of the messages is not hardcoded.

已经支持国际化的程序源代码如下，注意：文本内容没有硬编码进代码中：

```java 
import java.util.*;

public class I18NSample {

    static public void main(String[] args) {

        String language;
        String country;

        if (args.length != 2) {
            language = new String("en");
            country = new String("US");
        } else {
            language = new String(args[0]);
            country = new String(args[1]);
        }

        Locale currentLocale;
        ResourceBundle messages;

        currentLocale = new Locale(language, country);

        messages = ResourceBundle.getBundle("MessagesBundle", currentLocale);
        System.out.println(messages.getString("greetings"));
        System.out.println(messages.getString("inquiry"));
        System.out.println(messages.getString("farewell"));
    }
}
```

To compile and run this program, you need these source files:

为了运行这个程序，你还需要这些源文件：

[I18NSample.java](https://docs.oracle.com/javase/tutorial/i18n/intro/examples/I18NSample.java)
[MessagesBundle.properties](https://docs.oracle.com/javase/tutorial/i18n/intro/examples/MessagesBundle.properties)
[MessagesBundle_de_DE.properties](https://docs.oracle.com/javase/tutorial/i18n/intro/examples/MessagesBundle_de_DE.properties)
[MessagesBundle_en_US.properties](https://docs.oracle.com/javase/tutorial/i18n/intro/examples/MessagesBundle_en_US.properties)
[MessagesBundle_fr_FR.properties](https://docs.oracle.com/javase/tutorial/i18n/intro/examples/MessagesBundle_fr_FR.properties)

#### Running the Sample Program

The internationalized program is flexible; it allows the end user to specify a language and a country on the command line. 
In the following example the language code is fr (French) and the country code is FR (France), so the program displays the 
messages in French:

已国际化的程序十分灵活：它允许终端用户在命令行指定语言和国家。下面的实例使用fr作为语言码，FR作为国家码，于是程序运行时显示的文本内容是法语：

``` 
% java I18NSample fr FR
Bonjour.
Comment allez-vous?
Au revoir.
```

In the next example the language code is en (English) and the country code is US (United States) so the program displays 
the messages in English:

在下面的示例中使用en作为语言码，US作为国家码，于是程序显示的文本内容是英文：

```
% java I18NSample en US
Hello.
How are you?
Goodbye.
```

#### Internationalizing the Sample Program

If you look at the internationalized source code, you'll notice that the hardcoded English messages have been removed. 
Because the messages are no longer hardcoded and because the language code is specified at run time, the same executable 
can be distributed worldwide. No recompilation is required for localization. The program has been internationalized.

如果你看过已国际化的源代码，可以注意到硬编码的英文信息已经移除掉了。因为这些文本信息不再硬编码以及在运行时指定国家码，使得程序可以在全世界范围内分布执行。
本地化也不需要重新编译代码。这个程序已经支持国际化。

You may be wondering what happened to the text of the messages or what the language and country codes mean. Don't worry. 
You'll learn about these concepts as you step through the process of internationalizing the sample program.

你可能会有疑问，对文本内容到底做了什么，或者语言码和国家码意味着什么。不要担心，你将会很快学习那些概念。

**Create the Properties Files**

A properties file stores information about the characteristics of a program or environment. A properties file is in 
plain-text format. You can create the file with just about any text editor.

属性文件主要保存哪些程序的特性或环境变量。属性文件是纯文本文件，你可以使用任何的文本编辑器来创建。

In the example the properties files store the translatable text of the messages to be displayed. Before the program was 
internationalized, the English version of this text was hardcoded in the System.out.println statements. The default 
properties file, which is called MessagesBundle.properties, contains the following lines:

在上面的示例中属性文件用来保存那些翻译过的需要显示的文本内容。在国际化程序之前，英文版本的文本内容直接通过System.out.println来打印输出。默认的
属性文件MessagesBundle.properties包含下面几行内容：

```sh 
greetings = Hello
farewell = Goodbye
inquiry = How are you?
```

Now that the messages are in a properties file, they can be translated into various languages. No changes to the source 
code are required. The French translator has created a properties file called MessagesBundle_fr_FR.properties, which 
contains these lines:

现在所有的文本内容都保存在属性文件中，它们被翻译成多种语言。源代码不需要任何的改变。法语的属性文件为MessagesBundle_fr_FR.properties，包含的
内容：

```
greetings = Bonjour.
farewell = Au revoir.
inquiry = Comment allez-vous?
```

Notice that the values to the right side of the equal sign have been translated but that the keys on the left side have 
not been changed. These keys must not change, because they will be referenced when your program fetches the translated text.

注意，上面每一行等号右边的值就是翻译过的文本内容，而左边的键值是保持不变的。那些键值必须保证不变，因为在你的程序中将会根据这个键值来查找翻译的文本
内容。

The name of the properties file is important. For example, the name of the MessagesBundle_fr_FR.properties file contains 
the fr language code and the FR country code. These codes are also used when creating a Locale object.

属性文件的名称十分重要。比如，MessagesBundle_fr_FR.properties包含语言码fr和国家码FR。这两个码会在使用Local对象时用到。

**Define the Locale**

The Locale object identifies a particular language and country. The following statement defines a Locale for which the 
language is English and the country is the United States:

Locale对象意味着一个特殊的语言和国家。下面的片段定义了一个语言为英语，国家为美国的Locale：

```java 
aLocale = new Locale("en","US");
```

The next example creates Locale objects for the French language in Canada and in France:

下面是创建语言为法语，国家为加拿大和法国的Locale对象：

```java 
caLocale = new Locale("fr","CA");
frLocale = new Locale("fr","FR");
```

The program is flexible. Instead of using hardcoded language and country codes, the program gets them from the command line at run time:

程序使用在命令行读取语言码和国家码，替代在程序中硬编码指定他们：

```java 
String language = new String(args[0]);
String country = new String(args[1]);
currentLocale = new Locale(language, country);
```

Locale objects are only identifiers. After defining a Locale, you pass it to other objects that perform useful tasks, 
such as formatting dates and numbers. These objects are locale-sensitive because their behavior varies according to Locale. 
A ResourceBundle is an example of a locale-sensitive object.

在定义好Locale后，你可以使用些有用的特性，比如格式化时间和数字。那些对象针对不同的Locale是敏感:

#### Create a ResourceBundle

ResourceBundle objects contain locale-specific objects. You use ResourceBundle objects to isolate locale-sensitive data, 
such as translatable text. In the sample program the ResourceBundle is backed by the properties files that contain the 
message text we want to display.

ResourceBundle对象包含本地化的数据内容，你可以使用ResourceBundle对象来存放地区敏感的数据，比如字符的翻译。

The ResourceBundle is created as follows:

```
messages = ResourceBundle.getBundle("MessagesBundle", currentLocale);
```

The arguments passed to the getBundle method identify which properties file will be accessed. The first argument, 
MessagesBundle, refers to this family of properties files:

传递给getBundle方法的参数决定到底使用哪个属性文件。第一个参数MessagesBundle指的属性的主文件名：

```sh 
MessagesBundle_en_US.properties
MessagesBundle_fr_FR.properties
MessagesBundle_de_DE.properties
```

The Locale, which is the second argument of getBundle, specifies which of the MessagesBundle files is chosen. When the 
Locale was created, the language code and the country code were passed to its constructor. Note that the language and 
country codes follow MessagesBundle in the names of the properties files.

getBundle对象的第二个参数是Locale对象，指定哪个具体的MessagesBundle文件将会被选中。

**Fetch the Text from the ResourceBundle**

The properties files contain key-value pairs. The values consist of the translated text that the program will display. 
You specify the keys when fetching the translated messages from the ResourceBundle with the getString method. For example, 
to retrieve the message identified by the greetings key, you invoke getString as follows:

属性文件内容是键值对的形式。值是程序中要显示的翻译文本，键主要是ResourceBundle的getString方法来查找特定的翻译文本。下面的示例，通过getString来查找键
为greetings的文本内容：

```java 
String msg1 = messages.getString("greetings");
```

The sample program uses the key greetings because it reflects the content of the message, but it could have used another 
String, such as s1 or msg1. Just remember that the key is hardcoded in the program and it must be present in the properties 
files. If your translators accidentally modify the keys in the properties files, getString won't be able to find the messages.

你也可以使用s1或msg1来作为键值。只需要记住键值是在程序中硬编码的，而且它必须在属性文件中定义好。如果翻译员突然修改键值名称，getString将找不到要显示
的文本信息。

Now all you have to do is get the translated messages from the ResourceBundle.

现在你只需要做的就是通过ResourceBundle来获取已翻译过的文本信息。






