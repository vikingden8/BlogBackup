---
title: 避免Android中Context引起的内存泄露
date: 2016-12-30 17:03:48
tags:
categories: "Android学习笔记"
---

Context是我们在编写Android程序经常使用到的对象，意思为上下文对象。 常用的有Activity的Context还是有Application的Context。Activity用来展示活动界面，包含了很多的视图，而视图又含有图片，文字等资源。在Android中内存泄露很容易出现，而持有很多对象内存占用的Activity更加容易出现内存泄露，开发者需要特别注意这个问题。

本文讲介绍Android中Context，更具体的说是Activity内存泄露的情况，以及如何避免Activity内存泄露，加速应用性能。  


### Drawable引起的内存泄露  

Drawable引起内存泄露这个问题是比较隐晦，难以察觉的。在阅读了Romain Guy的[Avoiding memory leaks](http://android-developers.blogspot.com.tr/2009/01/avoiding-memory-leaks.html)，结合grepcode查看源码才明白了。  

<!--more-->

在Android系统中，当我们进行了屏幕旋转，默认情况下，会销毁掉当前的Activity，并创建一个新的Activity并保持之前的状态。在这个过程中，Android系统会重新加载程序的UI视图和资源。假设我们有一个程序用到了一个很大的Bitmap图像，我们不想每次屏幕旋转时都重新加载这个Bitmap对象，最简单的办法就是将这个Bitmap对象使用static修饰。  

```java
    private static Drawable sBackground;

    @Override
    protected void onCreate(Bundle state) {
      super.onCreate(state);

      TextView label = new TextView(this);
      label.setText("Leaks are bad");

      if (sBackground == null) {
        sBackground = getDrawable(R.drawable.large_bitmap);
      }
      label.setBackgroundDrawable(sBackground);

      setContentView(label);
    }
```

但是上面的方法在屏幕旋转时有可能引起内存泄露，无论是咋一看还是仔细看这段代码，都很难发现哪里引起了内存泄露。

当一个Drawable绑定到了View上，实际上这个View对象就会成为这个Drawable的一个callback成员变量，上面的例子中静态的sBackground持有TextView对象lable的引用，而lable持有Activity的引用，而Activity会持有其他更多对象的引用。sBackground生命周期要长于Activity。当屏幕旋转时，Activity无法被销毁，这样就产生了内存泄露问题。

2.3.7及以下版本Drawable的setCallback方法的实现：

```java
public final void setCallback(Callback cb) {
    mCallback = cb;
}
```

好在从4.0.1开始，引入了弱引用处理这个问题，弱引用在GC回收时，不会阻止GC回收其指向的对象，避免了内存泄露问题:  

```java
public final void setCallback(Callback cb) {
    mCallback = new WeakReference<Callback>(cb);
}
```

### 单例引起的内存泄露  

单例是我们比较简单常用的一种设计模式,然而如果单例使用不当也会导致内存泄露。 比如这样一个例子,我们使用饿汉式初始化单例，AppSettings我们需要持有一个Context作为成员变量，如果我们按照下面的实现其实是有问题。  

```java
public class AppSettings {    
    private Context mAppContext;
    private static AppSettings sInstance = new AppSettings();

    //some other codes
    public static AppSettings getInstance() {
      return sInstance;
    }

    public final void setup(Context context) {
        mAppContext = context;
    }
}
```


sInstance作为静态对象，其生命周期要长于普通的对象，其中也包含Activity，当我们进行屏幕旋转，默认情况下，系统会销毁当前Activity，然后当前的Activity被一个单例持有，导致垃圾回收器无法进行回收，进而产生了内存泄露。

解决的方法就是不持有Activity的引用，而是持有Application的Context引用。代码如下修改:
```java
public final void setup(Context context) {
    mAppContext = context.getApplicationContext();
}
```


通常我们想要获取Context对象，主要有以下四种方法:

  * View.getContext,返回当前View对象的Context对象，通常是当前正在展示的Activity对象。  

  * Activity.getApplicationContext,获取当前Activity所在的(应用)进程的Context对象，通常我们使用Context对象时，要优先考虑这个全局的进程Context。  

  * ContextWrapper.getBaseContext():用来获取一个ContextWrapper进行装饰之前的Context，可以使用这个方法，这个方法在实际开发中使用并不多，也不建议使用。  

  * Activity.this 返回当前的Activity实例，如果是UI控件需要使用Activity作为Context对象，但是默认的Toast实际上使用ApplicationContext也可以。  

### 避免内存泄露须谨记  


  * 不要让生命周期长于Activity的对象持有到Activity的引用

  * 尽量使用Application的Context而不是Activity的Context

  * 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用（具体可以查看细话Java：”失效”的private修饰符了解）。如果使用静态内部类，将外部实例引用作为弱引用持有。

  * 垃圾回收不能解决内存泄露，了解Android中垃圾回收机制  


转载自：  
[避免Android中Context引起的内存泄露](http://droidyue.com/blog/2015/04/12/avoid-memory-leaks-on-context-in-android/)  
### 引申阅读  

[1、Difference between getContext() , getApplicationContext() , getBaseContext() and “this”](http://stackoverflow.com/questions/10641144/difference-between-getcontext-getapplicationcontext-getbasecontext-and)  

[2、What's the difference between the various methods to get a Context?](http://stackoverflow.com/questions/1026973/whats-the-difference-between-the-various-methods-to-get-a-context)
