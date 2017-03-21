---
title: How to use Android's Job Scheduler
date: 2017-03-21 21:09:17
tags:
categories: "Android学习笔记"
---

One of the really cool things about Lollipop is the new _JobScheduler_ API. Not only is this API exciting for developers but end users should also be excited. This surprisingly easy to use API lets your app schedule a job to take place according to a number of parameters. The cool thing about this is that you can easily set parameters that will save the end user lots of battery life, giving your users one less reason to uninstall your app. There are three main parts to this API: _JobInfo_, _JobService_ and _JobScheduler_.

## JobInfo and Available Parameters

Parameters are defined by the _JobInfo_ class and constructed using the _JobInfo.Builder_. The following are the different parameters that you can set.

  * Back-off Criteria is the policy that kicks in when a job is finished and a retry is requested. You can set the initial back-off time and if it is linear or exponential. The default is 30 sec and exponential._setBackoffCriteria(long initialBackoffMillis, int backoffPolicy)_

  * A Bundle of extras. This lets you send specific data to your job. _setExtras(PersistableBundle extras)_

  * A minimum amount of time your job should be delayed. _setMinimumLatency(long minLatencyMillis)_

  * A maximum amount of time to wait to execute your job. If you hit this time your job will be executed immediately regardless of your other parameters. _setOverrideDeadline(long maxExecutionDelayMillis)_

  * If you want this job repeated, you can specify the interval between repeats. You are guaranteed to be executed within an interval but not at what point during that interval. This can sometimes lead to jobs being run closely together. _setPeriodic(long intervalMillis)_

  * You can persist the job across boot. This requires the RECEIVE_BOOT_COMPLETED permission added to your manifest. _setPersisted(boolean isPersisted)_

  * The network type you want the device to have when your job is executed. You can choose between none, any and unmetered (Wifi). _setRequiredNetworkType(int networkType)_

  * Whether or not the device should be charging. _setRequiresCharging(boolean requiresCharging)_

  * If the device should be idle when running the job. This is a great time to do resource heavy jobs. _setRequiresDeviceIdle(boolean requiresDeviceIdle)_

  If we were the PlayStore we would probably want to update apps when:

    * on wifi to save the user data.

    * the device is idle so that we don’t slow down the user’s device experience. No more updating when the screen turns on!

    * the device is charging, allowing us to save the user battery life during the day.
<!--more-->
```java
ComponentName serviceName = new ComponentName(context, MyJobService.class);
JobInfo jobInfo = new JobInfo.Builder(JOB_ID, serviceName)
        .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
        .setRequiresDeviceIdle(true)
        .setRequiresCharging(true)
        .build();
```

What’s MyJobService.class? Don’t freak out, we’re getting to that now.

## JobService

The _JobService_ is the actual service that is going to run your job. This service has different methods to implement than normal services. The first method is _onStartJob(JobParameters params)_. This method is what gets called when the _JobScheduler_ decides to run your job based on its parameters. You can get the jobId from the _JobParameters_ and you will have to hold on to these parameters to finish the job later. The next method is _onStopJob(JobParameters params)_. This will get called when your parameters are no longer being met. In our previous example this would happen when the user switches off of wifi, unplugs or turns the screen on their device. This means that we want to stop our job as soon as we can; in this case we wouldn’t update any more apps.

There are three very important things to know when using a _JobService_.

  * The _JobService_ runs on the main thread. It is your responsibility to move your work off thread. If the user tries to open your app while a job is running on the main thread they might get an Android Not Responding error (ANR).

  * You must finish your job when it is complete. The _JobScheduler_ keeps a wake lock for your job. If you don’t call _jobFinished(JobParameters params, boolean needsReschedule)_ with the _JobParameters_ from _onStartJob(JobParameters params)_ the _JobScheduler_ will keep a wake lock for your app and burn the devices battery. Even worse is that the battery history will blame your app. Here at Two Toasters we call that type of bug a 1 star, uninstall.

  * You have to register your job service in the _AndroidManifest_. If you do not, the system will not be able to find your service as a component and it will not start your jobs. You’ll never even know as this does not produce an error.

```xml
<application
    .... stuff ....
    >

    <service
        android:name=".MyJobService"
        android:permission="android.permission.BIND_JOB_SERVICE"
        android:exported="true"/>
</application>
```

```java
public class MyJobService extends JobService {

    private UpdateAppsAsyncTask updateTask = new UpdateAppsAsyncTask();

    @Override
    public boolean onStartJob(JobParameters params) {
        // Note: this is preformed on the main thread.

        updateTask.execute(params);

        return true;
    }

    // Stopping jobs if our job requires change.

    @Override
    public boolean onStopJob(JobParameters params) {
        // Note: return true to reschedule this job.

        boolean shouldReschedule = updateTask.stopJob(params);

        return shouldReschedule;
    }

    private class UpdateAppsAsyncTask extends AsyncTask<JobParameters, Void, JobParameters[]> {


        @Override
        protected JobParameters[] doInBackground(JobParameters... params) {

          // Do updating and stopping logical here.
          return params;
        }

        @Override
        protected void onPostExecute(JobParameters[] result) {
            for (JobParameters params : result) {
                if (!hasJobBeenStopped(params)) {
                    jobFinished(params, false);
                }
            }
        }

        private boolean hasJobBeenStopped(JobParameters params) {
            // Logic for checking stop.
        }

        public boolean stopJob(JobParameters params) {
            // Logic for stopping a job. return true if job should be rescheduled.
        }

    }
}
```

Of course this is just stubbed out and you can handle your off thread logic however you so wish. Just remember that however you handle threading you have to call _jobFinished(JobParameters params, boolean needsReschedule)_ when you’re done.

## JobScheduler, wait… its how easy?

This part is really darn easy. We now have our _JobInfo_ and our _JobService_ so it is time to schedule our job! All we have to do is get the _JobService_ the same way you would get any system service and hand it our _JobInfo_ with the _schedule(JobInfo job)_ method. Noob tip: your activity is a context object.

```java
JobScheduler scheduler = (JobScheduler) context.getSystemService(Context.JOB_SCHEDULER_SERVICE);
int result = scheduler.schedule(jobInfo);
if (result == JobScheduler.RESULT_SUCCESS) Log.d(TAG, "Job scheduled successfully!");
```

## Conclusion

Not too bad eh? Way easier than setting up a _SyncAdapter_ plus it’s more powerful and flexible than the _AlarmManager_! If you want to check out the code in action, head to our [Github](https://github.com/twotoasters/toastdroid/tree/master/chris/how_to_use_jobscheduler) repo and build it yourself! Thanks for the read and as always, keep on toasting.

Original From:[How to use Android's Job Scheduler](http://toastdroid.com/2015/02/21/how-to-use-androids-job-scheduler/)
