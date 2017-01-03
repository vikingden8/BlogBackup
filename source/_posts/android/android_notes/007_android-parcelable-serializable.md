---
title: Android系统中Parcelable和Serializable的区别
date: 2017-01-03 22:18:27
tags:
categories: "Android学习笔记"
---

进行Android开发的时候，我们都知道不能将对象的引用传给Activities或者Fragments，我们需要将这些对象放到一个Intent或者Bundle里面，然后再传递。
通过Android的API，我们知道有两种选择，即在传递对象时，需要对我们的对象进行 [Parcelable](https://developer.android.com/reference/android/os/Parcelable.html) 或者[Serializable化](https://developer.android.com/reference/java/io/Serializable.html)。作为Java开发者，相信大家对Serializable 机制有一定了解，那为什么还需要 Parcelable呢？

为了回答这个问题，让我们分别来看看这两者的差异。

#### Serializable, 简单易用

```java
// access modifiers, accessors and constructors omitted for brevity
public class SerializableDeveloper implements Serializable
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;

    static class Skill implements Serializable {
        String name;
        boolean programmingRelated;
    }
}
```

serializable的迷人之处在于你只需要对某个类以及它的属性实现Serializable 接口即可。Serializable 接口是一种标识接口（[marker interface](http://javapapers.com/core-java/abstract-and-interface-core-java-2/what-is-a-java-marker-interface/)），这意味着无需实现方法，Java便会对这个对象进行高效的序列化操作。

这种方法的缺点是使用了反射，序列化的过程较慢。这种机制会在序列化的时候创建许多的临时对象，容易触发垃圾回收。

<!--more-->

#### Parcelable, 速度至上

```java
// access modifiers, accessors and regular constructors ommited for brevity
class ParcelableDeveloper implements Parcelable {
    String name;
    int yearsOfExperience;
    List<Skill> skillSet;
    float favoriteFloat;

    ParcelableDeveloper(Parcel in) {
        this.name = in.readString();
        this.yearsOfExperience = in.readInt();
        this.skillSet = new ArrayList<Skill>();
        in.readTypedList(skillSet, Skill.CREATOR);
        this.favoriteFloat = in.readFloat();
    }

    void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(yearsOfExperience);
        dest.writeTypedList(skillSet);
        dest.writeFloat(favoriteFloat);
    }

    int describeContents() {
        return 0;
    }

    static final Parcelable.Creator<ParcelableDeveloper> CREATOR
            = new Parcelable.Creator<ParcelableDeveloper>() {

        ParcelableDeveloper createFromParcel(Parcel in) {
            return new ParcelableDeveloper(in);
        }

        ParcelableDeveloper[] newArray(int size) {
            return new ParcelableDeveloper[size];
        }
    };

    static class Skill implements Parcelable {
        String name;
        boolean programmingRelated;

        Skill(Parcel in) {
            this.name = in.readString();
            this.programmingRelated = (in.readInt() == 1);
        }

        @Override
        void writeToParcel(Parcel dest, int flags) {
            dest.writeString(name);
            dest.writeInt(programmingRelated ? 1 : 0);
        }

        static final Parcelable.Creator<Skill> CREATOR
            = new Parcelable.Creator<Skill>() {

            Skill createFromParcel(Parcel in) {
                return new Skill(in);
            }

            Skill[] newArray(int size) {
                return new Skill[size];
            }
        };

        @Override
        int describeContents() {
            return 0;
        }
    }
}
```

根据[Google 工程师](http://stackoverflow.com/questions/3611843/is-using-serializable-in-android-bad/3612364#3612364)的说法，这些代码将会运行地特别快。原因之一就是我们已经清楚地知道了序列化的过程，而不需要使用反射来推断。同时为了更快地进行序列化，对象的代码也需要高度优化。

因此，很明显实现Parcelable并不容易。实现Parcelable接口需要写大量的模板代码，这使得对象代码变得难以阅读和维护。

#### 速度测试

当然，我们还是想知道到底Parcelable相对于Serializable要快多少。

#### 测试方法

  * 通过将一个对象放到一个bundle里面然后调用Bundle#writeToParcel(Parcel, int)方法来模拟传递对象给一个activity的过程，然后再把这个对象取出来。

  * 在一个循环里面运行1000 次。

  * 两种方法分别运行10次来减少内存整理，cpu被其他应用占用等情况的干扰。

  * 参与测试的对象就是上面代码中的SerializableDeveloper 和 ParcelableDeveloper。

  * 在多种Android软硬件环境上进行测试

      - LG Nexus 4 – Android 4.2.2

      - Samsung Nexus 10 – Android 4.2.2

      - HTC Desire Z – Android 2.3.3

#### 结果

![](/images/categories/android/android_notes/003_android_handler_leaks/parcelable-vs-serializable.png)

  * NEXUS 10

    Serializable: 1.0004ms,  Parcelable: 0.0850ms – 提升10.16倍。

  * NEXUS 4

    Serializable: 1.8539ms – Parcelable: 0.1824ms – 提升11.80倍。

  * DESIRE Z

    Serializable: 5.1224ms – Parcelable: 0.2938ms – 提升17.36倍。

由此可以得出: Parcelable 比 Serializable快了10多倍。有趣的是，即使在Nexus 10这样性能强悍的硬件上，一个相当简单的对象的序列化和反序列化的过程要花将近一毫秒。

#### 总结

如果你想成为一个优秀的软件工程师，你需要多花点时间来实现 Parcelable ，因为这将会为你对象的序列化过程快10多倍，而且占用较少的资源。

但是大多数情况下， Serializable 的龟速不会太引人注目。你想偷点懒就用它吧，不过要记得serialization是一个比较耗资源的操作，尽量少使用。

如果你想要传递一个包含许多对象的列表，那么整个序列化的过程的时间开销可能会超过一秒，这会让屏幕转向的时候变得很卡顿。



转载自：[Android系统中Parcelable和Serializable的区别](http://greenrobot.me/devpost/android-parcelable-serializable/)

本文翻译自：[Parcelable vs Serializable](http://www.developerphil.com/parcelable-vs-serializable/)
