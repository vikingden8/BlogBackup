---
title: Android面试一天一题（Day 3）
date: 2017-01-10 22:33:05
tags:
categories: "Android面试"
---

#### 面试题： 如何判断Activity是否在运行？

如下这场景我相信很多人都遇到过，这段话也是从某个帖子截取出来的：

>从Activity A 启动一个线程去进行网络上传操作，在A中设立一个回调函数，当上传操作完成以后，在A的这个回调函数中会弹出一个对话框，用来显示回调信息。可是当上传的过程还在进行的时候，我按下back键，A的activity 被销毁了，可是那个上传的线程还在进行，当那个线程结束后，本来应该在A中弹出一个对话框，可是由于A已经不存在了，系统就会报错提示，“将对话框显示在不存在的页面上”，然后程序就挂掉了。

我看到过很多人用Handler来充当上面所提到的“回调函数”，即工作线程完成工作后，通过主线程的Handler处理UI更新，如弹出提示Dialog。可能有些人没有弄明白，Activity都被系统销毁了，工作线程怎么还能调它的变量呢？其实所谓的Activity销毁只是不再受系统的AMS控制，但Activity这个对象的实例还是存在于内存中的，具体什么时候真正把这个对象实例也销毁（回收）了，就要看内存回收机制了，哪怕是这个实例没有可达的引用了也不一定会马上回收。

<!--more-->

针对这种用Handler更新UI的情况，我们需要在操作UI前判断一下此Activity是否已被销毁。很多人可能都用过isFinishing来判断，用多了就会发现好象不太准，为什么呢，看一下它的源代码：

```java
    /**
     * Check to see whether this activity is in the process of finishing,
     * either because you called {@link #finish} on it or someone else
     * has requested that it finished.  This is often used in
     * {@link #onPause} to determine whether the activity is simply pausing or
     * completely finishing.
     *
     * @return If the activity is finishing, returns true; else returns false.
     *
     * @see #finish
     */
    public boolean isFinishing() {
        return mFinished;
    }

    /**
     * Returns true if the final {@link #onDestroy()} call has been made
     * on the Activity, so this instance is now dead.
     */
    public boolean isDestroyed() {
        return mDestroyed;
    }
```

而mFinished是在finish()中被赋值的，也就是说只有通过调用finish()结束的Activity，mFinished的值才会被置为true。所以有时候Activity的生命周期没有按我们预想的来走时（如内存紧张时），会出现判断出错的情况。

看看Google工程师是怎么判断的（来源于Android源码中的Call应用，AsyncTask中的onPostExecute片段）：

```java
@Override
protected void onPostExecute(Void result) {
    final Activity activity = progressDialog.getOwnerActivity();

    if (activity == null || activity.isDestroyed() || activity.isFinishing()) {
        return;
    }

    if (progressDialog != null && progressDialog.isShowing()) {
        progressDialog.dismiss();
    }
}
```

多了一个isDestroyed()的判断。

#### 小结

如果对方没听说过isFinishing函数，那可以让他从自己的角度看如何解决这个问题，正好可以看看他的逻辑思维是否清晰合理。工作中往往会遇到，一些求职者由于之前是做其他方面刚转Android开发，对Android的了解还不够，但有很强理解和学习能力，通过引导发现他可以快速的得到合理的解决方案的话，我一般都很乐意要这样的人。
