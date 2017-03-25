---
title: Interpreting CTS results
date: 2017-03-25 18:54:37
tags:
categories: "Android Sources Website"
---

The CTS test results are placed in the file:

```sh
$CTS_ROOT/android-cts/repository/results/<start_time>.zip
```

If you have built the CTS yourself, $CTS_ROOT will resemble the path _out/host/linux-x86/cts_ but differ by platform. This reflects the path where you have uncompressed the prebuilt official CTS downloaded from this site.

Inside the zip, the _testResult.xml_ file contains the actual results. Open this file in any web browser (HTML5 compatible browser recommended) to view the test results.

If _testResult.xml_ displays a blank page when using the Chrome browser, change your browser configuration to enable the __--allow-file-access-from-files__ command line flag.

<!--more-->

### Reading the test results

The details of the test results depend on which version of CTS you are using:

  * CTS v1 for Android 6.0 and earlier

  * CTS v2 for Android 7.0 and later

>Note: The results are provided to help you ensure the software remains compatible throughout the development process and act as a common format for communicating the compatibility status of your device with other parties.

### Device Information

In CTS v6.0 and earlier, select Device Information (link above Test Summary) to view details about the device, firmware (make, model, firmware build, platform), and device hardware (screen resolution, keypad, screen type). (CTS v7.0 does not display device information.)

### Test Summary

The Test Summary section provides executed test plan details, like the CTS plan name and execution start and end times. It also presents an aggregate summary of the number of tests that passed, failed, timed out, or could not be executed.

  CTS v1 sample test summary

  ![](cts-test-summary.png)

  CTS v2 sample test summary

  ![](cts-v2-test-summary.png)

### Test Report

The next section, the CTS test report, provides a summary of tests passed per package.

This is followed by details of the the actual tests that were executed. The report lists the test package, test suite, test case, and the executed tests. It shows the result of the test execution—pass, fail, timed out, or not executed. In the event of a test failure details are provided to help diagnose the cause.

Further, the stack trace of the failure is available in the XML file but is not included in the report to ensure brevity—viewing the XML file with a text editor should provide details of the test failure (search for the <Test> tag corresponding to the failed test and look within it for the <StackTrace> tag).

  CTS v1 sample test report

  ![](cts-test-report.png)

  CTS v2 sample test report

  ![](cts-v2-test-report.png)

### Reviewing test_result.xml for incomplete test modules

To determine the number of incomplete modules in a given test session, run command 'list results'. The count of Modules Completed and Total Modules are listed for each previous session. To determine which modules are complete vs. incomplete, open the test_result.xml file and read the value of the "done" attribute for each module in the result report. Modules with value done = "false" have not run to completion.
