---
title: Using the JobScheduler API on Android Lollipop
date: 2017-3-21 09:34:25
tags:
categories: "Android学习笔记"
---

In this tutorial, you will learn how to use the _JobScheduler_ API available in Android Lollipop. The _JobScheduler_ API allows developers to create jobs that execute in the background when certain conditions are met.

### Introduction

When working with Android, there will be occasions where you will want to run a task at a later point in time or under certain conditions, such as when a device is plugged into a power source or connected to a Wi-Fi network. Thankfully with API 21, known by most people as Android Lollipop, Google has provided a new component known as the _JobScheduler_ API to handle this very scenario.

The _JobScheduler_ API performs an operation for your application when a set of predefined conditions are met. Unlike the _AlarmManager_ class, the timing isn't exact. In addition, the _JobScheduler_ API is able to batch various jobs to run together. This allows your app to perform the given task while being considerate of the device's battery at the cost of timing control.

In this article, you will learn more about the _JobScheduler_ API and the JobService class by using them to run a simple background task in an Android application. The code for this tutorial is available on [GitHub](https://github.com/tutsplus/Android-JobSchedulerAPI).

#### Creating the Job Service

To start, you're going to want to create a new Android project with a minimum required API of 21, because the _JobScheduler_ API was added in the most recent version of Android and, at the time of writing, is not backwards compatible through a support library.

Assuming you're using Android Studio, after you've hit the finished button for the new project, you should have a bare-bones "Hello World" application. The first step you're going to take with this project is to create a new Java class. To keep things simple, let's name it _JobSchedulerService_ and extend the _JobService_ class, which requires that two methods be created _onStartJob(JobParameters params)_ and _onStopJob(JobParameters params)_.

<!--more-->

```java
public class JobSchedulerService extends JobService {

    @Override
    public boolean onStartJob(JobParameters params) {

        return false;
    }

    @Override
    public boolean onStopJob(JobParameters params) {

        return false;
    }

}
```

_onStartJob(JobParameters params)_ is the method that you must use when you begin your task, because it is what the system uses to trigger jobs that have already been scheduled. As you can see, the method returns a boolean value. If the return value is false, the system assumes that whatever task has run did not take long and is done by the time the method returns. If the return value is true, then the system assumes that the task is going to take some time and the burden falls on you, the developer, to tell the system when the given task is complete by calling _jobFinished(JobParameters params, boolean needsRescheduled)_.

_onStopJob(JobParameters params)_ is used by the system to cancel pending tasks when a cancel request is received. It's important to note that if _onStartJob(JobParameters params)_ returns false, the system assumes there are no jobs currently running when a cancel request is received. In other words, it simply won't call _onStopJob(JobParameters params)_.

One thing to note is that the job service runs on your application's main thread. This means that you have to use another thread, a handler, or an asynchronous task to run longer tasks to not block the main thread. Because multithreading techniques are beyond the scope of this tutorial, let's keep it simple and implement a handler to run our task in the _JobSchedulerService_ class.

```java
private Handler mJobHandler = new Handler( new Handler.Callback() {

    @Override
    public boolean handleMessage( Message msg ) {
        Toast.makeText( getApplicationContext(),
            "JobService task running", Toast.LENGTH_SHORT )
            .show();
        jobFinished( (JobParameters) msg.obj, false );
        return true;
    }

} );
```

In the handler, you implement the _handleMessage(Message msg)_ method that is a part of Handler instance and have it run your task's logic. In this case, we're keeping things very simple and post a Toast message from the application, though this is where you would put your logic for things like syncing data.

When the task is done, you need to call _jobFinished(JobParameters params, boolean needsRescheduled)_ to let the system know that you're done with that task and that it can begin queuing up the next operation. If you don't do this, your jobs will only run once and your application will not be allowed to perform additional jobs.

The two parameters that _jobFinished(JobParameters params, boolean needsRescheduled)_ takes are the JobParameters that were passed to the JobService class in the _onStartJob(JobParameters params)_ method and a boolean value that lets the system know if it should reschedule the job based on the original requirements of the job. This boolean value is useful to understand, because it is how you handle the situations where your task is unable to complete because of other issues, such as a failed network call.

With the Handler instance created, you can go ahead and start implementing the _onStartJob(JobParameters params)_ and _onStopJob(JobParameters params)_ methods to control your tasks. You'll notice that in the following code snippet, the _onStartJob(JobParameters params)_ method returns true. This is because you're going to use a Handler instance to control your operation, which means that it could take longer to finish than the _onStartJob(JobParameters params)_ method. By returning true, you're letting the application know that you will manually call the _jobFinished(JobParameters params, boolean needsRescheduled)_ method. You'll also notice that the number 1 is being passed to the Handler instance. This is the identifier that you're going to use for referencing the job.

```java
@Override
public boolean onStartJob(JobParameters params) {
    mJobHandler.sendMessage( Message.obtain( mJobHandler, 1, params ) );
    return true;
}

@Override
public boolean onStopJob(JobParameters params) {
    mJobHandler.removeMessages( 1 );
    return false;
}
```

Once you're done with the Java portion of the _JobSchedulerService_ class, you need to go into _AndroidManifest.xml_ and add a node for the service so that your application has permission to bind and use this class as a JobService.

```xml
<service android:name=".JobSchedulerService"
    android:permission="android.permission.BIND_JOB_SERVICE" />
```

#### Creating the Job Scheduler

With _JobSchedulerService_ class finished, we can start looking at how your application will interact with the _JobScheduler_ API. The first thing you will need to do is create a JobScheduler object, called _mJobScheduler_ in the sample code, and initialize it by getting an instance of the system service _JOB_SCHEDULER_SERVICE_. In the sample application, this is done in the MainActivity class.

```java
mJobScheduler = (JobScheduler)
    getSystemService( Context.JOB_SCHEDULER_SERVICE );
```

When you want to create your scheduled task, you can use the _JobInfo.Builder_ to construct a _JobInfo_ object that gets passed to your service. To create a JobInfo object, _JobInfo.Builder_ accepts two parameters. The first is the identifier of the job that you will run and the second is the component name of the service that you will use with the JobScheduler API.

```java
JobInfo.Builder builder = new JobInfo.Builder( 1,
        new ComponentName( getPackageName(),
            JobSchedulerService.class.getName() ) );
```

This builder allows you to set many different options for controlling when your job will execute. The following code snippet shows how you could set your task to run periodically every three seconds.

```java
builder.setPeriodic( 3000 );
```

Other methods include:

  * _setMinimumLatency(long minLatencyMillis)_: This makes your job not launch until the stated number of milliseconds have passed. This is incompatible with _setPeriodic(long time)_ and will cause an exception to be thrown if they are both used.

  * _setOverrideDeadline(long maxExecutionDelayMillis)_: This will set a deadline for your job. Even if other requirements are not met, your task will start approximately when the stated time has passed. Like _setMinimumLatency(long time)_, this function is mutually exclusive with _setPeriodic(long time)_ and will cause an exception to be thrown if they are both used.

  * _setPersisted(boolean isPersisted)_: This function tells the system whether your task should continue to exist after the device has been rebooted.

  * _setRequiredNetworkType(int networkType)_: This function will tell your job that it can only start if the device is on a specific kind of network. The default is _JobInfo.NETWORK_TYPE_NONE_, meaning that the task can run whether there is network connectivity or not. The other two available types are JobInfo.NETWORK_TYPE_ANY, which requires some type of network connection available for the job to run, and JobInfo._NETWORK_TYPE_UNMETERED_, which requires that the device be on a non-cellular network.

  * _setRequiresCharging(boolean requiresCharging)_: Using this function will tell your application that the job should not start until the device has started charging.

  * _setRequiresDeviceIdle(boolean requiresDeviceIdle)_: This tells your job to not start unless the user is not using their device and they have not used it for some time.

It's important to note that _setRequiredNetworkType(int networkType)_, _setRequiresCharging(boolean requireCharging)_ and _setRequiresDeviceIdle(boolean requireIdle)_ may cause your job to never start unless _setOverrideDeadline(long time)_ is also set, allowing your job to run even if conditions are not met. Once the preferred conditions are stated, you can build the JobInfo object and send it to your JobScheduler object as shown below.

```java
if( mJobScheduler.schedule( builder.build() ) <= 0 ) {
    //If something goes wrong
}
```

You'll notice that the schedule operation returns an integer. If schedule fails, it will return a value of zero or less, corresponding to an error code. Otherwise it will return the job identifier that we defined in the JobInfo.Builder.

If your application requires that you stop a specific or all jobs, you can do so by calling _cancel(int jobId)_ or _cancelAll()_ on the JobScheduler object.

```java
mJobScheduler.cancelAll();
```

You should now be able to use the _JobScheduler_ API with your own applications to batch jobs and run background operations.

### Conclusion

In this article, you've learned how to implement a JobService subclass that uses a Handler object to run background tasks for your application. You've also learned how to use the JobInfo.Builder to set requirements for when your service should run. Using these, you should be able to improve how your own applications operate while being mindful of power consumption.

Original Site:[Using the JobScheduler API on Android Lollipop](https://code.tutsplus.com/tutorials/using-the-jobscheduler-api-on-android-lollipop--cms-23562)
