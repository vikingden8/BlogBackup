---
title: CTS Development
date: 2017-03-25 20:15:53
tags:
categories: "Android Sources Website"
---

### Initializing your Repo client

Follow the [instructions](http://source.android.com/source/downloading.html) to get and build the Android source code but specify a particular CTS branch name, for example-b android-5.0_r2, when issuing the repo init command. This assures your CTS changes will be included in the next CTS release and beyond.

### Building and running CTS

Execute the following commands to build CTS and start the interactive CTS console:

>Note: You may supply one of these other values for TARGET_PRODUCT to build for different architectures: aosp_x86_64 or aosp_mips

```sh
cd /path/to/android/root
make cts -j32 TARGET_PRODUCT=aosp_arm64
cts-tradefed
```

<!--more-->

At the cts-tf console, enter e.g.:

```sh
run cts --plan CTS
```

### Writing CTS tests

CTS tests use JUnit and the Android testing APIs. Review the Testing and Instrumentation tutorial while perusing the existing tests under the cts/tests directory. You will see that CTS tests mostly follow the same conventions used in other Android tests.

Since CTS runs across many production devices, the tests must follow these rules:

  * Must take into account varying screen sizes, orientations, and keyboard layouts.

  * Only use public API methods. In other words, avoid all classes, methods, and fields that are annotated with the "hide" annotation.

  * Avoid relying upon particular view layouts or depend on the dimensions of assets that may not be on some device.

  * Don't rely upon root privileges.

### Test naming and location

Most CTS test cases target a specific class in the Android API. These tests have Java package names with a cts suffix and class names with the Test suffix. Each test case consists of multiple tests, where each test usually exercises a particular method of the class being tested. These tests are arranged in a directory structure where tests are grouped into different categories such as "widgets" and "views."

For example, the CTS test for the Java package android.widget.TextView is android.widget.cts.TextViewTest with its Java package name as android.widget.cts and its class name as TextViewTest.

  * __Java package name__

  The Java package name for the CTS tests is the package name of the class that the test is testing, followed by ".cts". For our example, the package name would be android.widget.cts.

  * __Class name__

  The class name for CTS tests is name of the class that the test is testing with "Test" appended. (For example, if a test is targeting TextView, the class name should be TextViewTest.)

  * __Module name (CTS v2 only)__

  CTS v2 organizes tests by module. The module name is usually the second string of the Java package name (in our example, widget), although it does not have to be.

The directory structure and sample code depend on whether you are using CTS v1 or CTS v2.

#### CTS v1

For Android 6.0 and earlier, use CTS v1. For CTS v1, the sample code is at cts/tests/tests/example.

The directory structure in CTS v1 tests looks like this:

```xml
cts/
  tests/
    tests/
      package-name/
        Android.mk
        AndroidManifest.xml
        src/
          android/
            package-name/
              SampleDeviceActivity.java
              cts/
                SampleDeviceTest.java
```

#### CTS v2

For Android 7.0 and later, use CTS v2. For details, see the sample test in AOSP.

The CTS v2 directory structure looks like this:

```xml
cts/
  tests/
    module-name/
      Android.mk
      AndroidManifest.xml
      src/
        android/
          package-name/
            SampleDeviceActivity.java
            cts/
              SampleDeviceTest.java

```

### New sample packages

When adding new tests, there may not be an existing directory to place your test. In those cases, you'll need to create the directory and copy the appropriate sample files.

#### CTS v1

If you are using CTS v1, refer to the example under _cts/tests/tests/example_ and create a new directory. Also, make sure to add your new package's module name from its Android.mk to _CTS_COVERAGE_TEST_CASE_LIST_ in _cts/CtsTestCaseList.mk_. This Makefile is used by _build/core/tasks/cts.mk_ to combine all the tests together to create the final CTS package.

#### CTS v2

Use the sample test _/cts/tests/sample/_ to quick start your new test module with following steps:

  * Run this command to create the test directory and copy sample files to it:

  ```sh
  $ mkdir cts/tests/module-name && cp -r cts/tests/sample/* cts/tests/module-name
  ```

  * Navigate to _cts/tests/module-name_ and substitute all instances of "[Ss]ample" following the recommended naming convention from above.

  * Update _SampleDeviceActivity_ to exercise the feature you're testing.

  * Update _SampleDeviceTest_ to ensure the activity succeeds or logs its errors.

#### Additional directories

Other Android directories such as assets, jni, libs and res can also be added. To add JNI code, create a directory in the root of the project next to src with the native code and an Android.mk in it.

The makefile typically contains the following settings:

```sh
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := libCtsSample_jni

# don't include this package in any target
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := list of source code files
LOCAL_C_INCLUDES := $(JNI_H_INCLUDE)

# Tag this module as a cts test artifact
LOCAL_COMPATIBILITY_SUITE := cts
LOCAL_SHARED_LIBRARIES := libnativehelper
LOCAL_SDK_VERSION := current
include $(BUILD_SHARED_LIBRARY)
```
#### Android.mk file

Finally, the Android.mk file in the root of the project will need to be modified to build the native code and depend on it, as shown below:

```sh
# All tests should include android.test.runner.
LOCAL_JAVA_LIBRARIES := android.test.runner

# Includes the jni code as a shared library
LOCAL_JNI_SHARED_LIBRARIES := libCtsSample_jni

# Include for InstrumentationCtsTestRunner
LOCAL_STATIC_JAVA_LIBRARIES := ctstestrunner...
LOCAL_SDK_VERSION := currentinclude $(BUILD_CTS_PACKAGE)

#Tells make to look in subdirectories for more make files to include
include $(call all-makefiles-under,$(LOCAL_PATH))
```

### Fix or remove tests

Besides adding new tests there are other ways to contribute to CTS: Fix or remove tests annotated with "BrokenTest" or "KnownFailure."

### Submitting your changes

Follow the Submitting Patches workflow to contribute changes to CTS. A reviewer will be assigned to your change, and your change should be reviewed shortly!

### Release schedule and branch information

CTS releases follow this schedule.

>Note: This schedule is tentative and may be updated from time to time as CTS for the given Android version matures.

![](/images/categories/android/android-sources/008/cts_release_schedule.png)

#### Important Dates during month of the release

  * __End of 1st Week__: Code Freeze. At this point, submissions on the current branch will no longer be accepted and will not be included in the next version of CTS. Once we have chosen a candidate for release, the branch will again be open and accepting new submissions.

  * __Second or third week__: CTS is published in the Android Open Source Project (AOSP).

#### Auto-merge flow

CTS development branches have been set up so that changes submitted to each branch will automatically merge as below:
jb-dev-> jb-mr1.1-cts-dev -> jb-mr2-cts-dev -> kitkat-cts-dev -> lollipop-cts-dev -> lollipop-mr1-cts-dev -> marshmallow-cts-dev -> nougat-cts-dev -> <private-development-branch for Android N MR1>

If a changelist (CL) fails to merge correctly, the author of the CL will get an email with instructions on how to resolve the conflict. In most of the cases, the author of the CL can use the instructions to skip the auto-merge of the conflicting CL.
