---
title: Android面试一天一题（Day 4）
date: 2017-01-10 22:41:53
tags:
categories: "Android面试"
---

#### 面试题：自定义View的状态是如何保存的？

Android有一套标准的做法，做过自定义View的人很容易遇到这个问题，因为Activity转屏，或Home键到后台很容易在被系统销毁，恢复时我们肯定是希望看到View保持之前状态。

提示：系统内存紧张时会主动销毁这类Activity，做为开发我们可以利用DDMS的stop process模拟这一动作而不必等到内存紧张时，如下图所示：

![](/images/categories/android/android_interview/ddms_kill_process.png)

<!--more-->

这个标准的做法可以随便从一个Android自带的控件中看到，如TextView的源代码:

```java
   /**
    * User interface state that is stored by TextView for implementing
    * {@link View#onSaveInstanceState}.
    */
   public static class SavedState extends BaseSavedState {
       int selStart;
       int selEnd;
       CharSequence text;
       boolean frozenWithFocus;
       CharSequence error;

       SavedState(Parcelable superState) {
           super(superState);
       }

       @Override
       public void writeToParcel(Parcel out, int flags) {
           super.writeToParcel(out, flags);
           out.writeInt(selStart);
           out.writeInt(selEnd);
           out.writeInt(frozenWithFocus ? 1 : 0);
         }
     }
```

BaseSavedState是View的一个内部静态类，从代码上我们也很容易看出是把控件的属性（如selStart）打包到Parcel容器，Activity的onSaveInstanceState、onRestoreInstanceState最终也会调用到控件的这两个同名方法。

Parcel相关的问题以后专门再讲，现在我们可以焦点先放在状态保存上，先看一下Activity的状态如何保存的：

![](/images/categories/android/android_interview/activity_state.png)

注：无法保证系统会在销毁Activity前一定调用onSaveInstanceState，例如用户使用“返回” 按退出 Activity 时，因为用户的行为是在显式关闭 Activity，所以不会调用onSaveInstanceState。

> 如果系统调用onSaveInstanceState，那么它是在onStop还是在onPause之前执行呢？

可以肯定的是它会在调用 onStop之前，但是是不是在onPuase之前就不能确认了，要看情况，官方文档在说明这个执行顺序时用了“可能”这个词。

#### 小结

Activity类的onSaveInstanceState默认实现会恢复Activity的状态，默认实现会为布局中的每个View调用相应的 onSaveInstanceState方法，让每个View都能保存自身的信息。

这里需要注意一个细节：想要保存View的状态，需要在XML布局文件中提供一个唯一的ID（android:id），如果没有设置这个ID的话，View控件的onSaveInstanceState是不会被调用的。
