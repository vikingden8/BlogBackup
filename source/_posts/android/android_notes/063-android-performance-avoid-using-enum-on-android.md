---
title: Android Performance-Avoid using ENUM on Android
date: 2017-08-09 21:30:52
tags:
categories: "Android学习笔记"
---

### What and why we use ENUM?

Enum in java is a data type that contains fixed set of constants. When we required predefined set of values which represents some kind of data, we use ENUM. We always use Enums when a variable can only take one out of a small set of possible values. For example:

```java
public enum Season
{
  WINTER, SPRING, SUMMER, FALL
}
```

Instead of using Integer or String constants, we generally use ENUM to increase compile time checking and avoid errors from passing in invalid constants.

<!--more-->

### Drawback of using ENUM

Each value in an ENUM is an object and each declaration will take some runtime memory simply to reference the object. So ENUM values will take more memory then Integer or String constant. Even in old android devices (<=2.2), there were some performance issue related to ENUM which were solved in JIT compiler.

Adding a single ENUM will increase the size (13x times than the Integer constant) of final DEX file. It also generates the problem of runtime overhead and your app will required more space.

So overuse of ENUM in Android would increase DEX size and increase runtime memory allocation size.

If your application is using more ENUM then better to use Integer or String constants instead of ENUM. But still problem remains…

### Solution

Android provide annotation library which has TypeDef annotations. These annotations ensure that a particular parameter, return value, or field references a specific set of constants. They also enable code completion to automatically offer the allowed constants.

IntDef and StringDef are two Magic Constant Annotation which can be used instead of Enum. These annotation will help us to check variable assignment like Enum in compile time.

### How to use?

Let’s take a simple example to understand the use. Below are code snippet of a ConstantSeason class.

```java
public class ConstantSeason {
    public static final int WINTER = 0;
    public static final int SPRING = 1;
    public static final int SUMMER = 2;
    public static final int FALL = 3;

    public ConstantSeason(int season) {
        System.out.println("Season :" + season);
    }

    public static void main(String[] args) {
        // Here chance to paas invalid value
        ConstantSeason constantSeason = new ConstantSeason(5);
    }
}
```

Unfortunately, there is no guarantee that the user will use the constructor with proper values — there is no type safety here.

Also the similar implementation using ENUM would be:

```java
public class EnumSeason {

    public EnumSeason(Season season) {
        System.out.println("Season :" + season);
    }

    public enum Season {
        WINTER, SPRING, SUMMER, FALL
    }

    public static void main(String[] args) {
        EnumSeason enumSeason = new EnumSeason(Season.SPRING);
    }
}
```

Now let’s look how to use magic constants from support-annotations.

Add the support-annotations dependency to your project by putting the following line in the dependencies block of your build.gradle file:

```gradle
dependencies { compile ‘com.android.support:support-annotations:24.2.0’ }
```

Declare the constants and @IntDef for these constants:

```java
// Constants
public static final int WINTER = 0;
public static final int SPRING = 1;
public static final int SUMMER = 2;
public static final int FALL = 3;

// Declare the @IntDef for these constants
@IntDef({WINTER, SPRING, SUMMER, FALL})
@Retention(RetentionPolicy.SOURCE)
public @interface Season {}
```

Here Typedef annotations use @interface to declare the new enumerated annotation type. The @IntDef and @StringDef annotations, along with @Retention, annotate the new annotation and are necessary in order to define the enumerated type. The @Retention(RetentionPolicy.SOURCE) annotation tells the compiler not to store the enumerated annotation data in the .class file.

So implementation of above example would be:

```java
public class AnnotationSeason {

    public static final int WINTER = 0;
    public static final int SPRING = 1;
    public static final int SUMMER = 2;
    public static final int FALL = 3;

    public AnnotationSeason(@Season int season) {
        System.out.println("Season :" + season);
    }

    @IntDef({WINTER, SPRING, SUMMER, FALL})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Season {
    }

    public static void main(String[] args) {
        AnnotationSeason annotationSeason = new AnnotationSeason(SPRING);
    }
}
```

Now if we pass different value in the constructor while create the instance of AnnotationSeason other than Season, compiler will give error.

@StringDef can be used in same manner.

```java
// Constants
public static final String WINTER = "Winter";
public static final String SPRING = "Spring";
public static final String SUMMER = "Summer";
public static final String FALL = "Fall";

// Declare the @ StringDef for these constants:
@ StringDef ({WINTER, SPRING, SUMMER, FALL})
@Retention(RetentionPolicy.SOURCE)
public @interface Season {}
```

### Conclusion

Enums add at least two times more bytes to the total APK size than plain constants and can use 5 to 10 times more RAM memory than equivalent constants. This is a best practice advice and performance matters of application.

From: [Android Performance: Avoid using ENUM on Android](https://android.jlelse.eu/android-performance-avoid-using-enum-on-android-326be0794dc3)
