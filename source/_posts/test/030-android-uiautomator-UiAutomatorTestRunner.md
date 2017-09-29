---
title: Android UiAutomator UiAutomatorTestRunner
date: 2017-09-29 8:31:48
tags:
categories: "测试开发"
---

Commands类说完了,其中提及到RunTestCommand这个命令是整个命令执行的核心,因为我们所需要执行的用例就是靠它来执行起来的,但是到了最后,我们才发现,其实它也只是一个数据的收集者而已,它把执行指令中传入的参数进行一轮检查后,成功传入给[UiAutomatorTestRunner](https://github.com/android/platform_frameworks_base/blob/master/cmds/uiautomator/library/testrunner-src/com/android/uiautomator/testrunner/UiAutomatorTestRunner.java)后,就完成了自己的本职工作,接下来的活又由Runner自己去定义和完成.

<!--more-->

### run()方法

```java
public void run(List<String> testClasses, Bundle params, boolean debug, boolean monkey) {
    Thread.setDefaultUncaughtExceptionHandler(new UncaughtExceptionHandler() {
        @Override
        public void uncaughtException(Thread thread, Throwable ex) {
            Log.e(LOGTAG, "uncaught exception", ex);
            Bundle results = new Bundle();
            results.putString("shortMsg", ex.getClass().getName());
            results.putString("longMsg", ex.getMessage());
            mWatcher.instrumentationFinished(null, 0, results);
            // bailing on uncaught exception
            System.exit(EXIT_EXCEPTION);
        }
    });

    mTestClasses = testClasses;
    mParams = params;
    mDebug = debug;
    mMonkey = monkey;
    start();
    System.exit(EXIT_OK);
}
```

run方法首先设置线程的异常捕获，然后把RunTestCommand中传过来的参数赋值，然后调用内部的start()方法．

```java
/**
 * Called after all test classes are in place, ready to test
 */
protected void start() {
    //首先创建一个测试用例收集器
    TestCaseCollector collector = getTestCaseCollector(this.getClass().getClassLoader());
    try {
        collector.addTestClasses(mTestClasses);
    } catch (ClassNotFoundException e) {
        // will be caught by uncaught handler
        throw new RuntimeException(e.getMessage(), e);
    }
    //如果是debug模式则等待调试器开启
    if (mDebug) {
        Debug.waitForDebugger();
    }
    //设置线程为守护线程，防止被jvm杀死
    mHandlerThread = new HandlerThread(HANDLER_THREAD_NAME);
    mHandlerThread.setDaemon(true);
    mHandlerThread.start();
    //构建UiAutomationShellWrapper对象
    UiAutomationShellWrapper automationWrapper = new UiAutomationShellWrapper();
    automationWrapper.connect();
    //创建开始计时的对象
    long startTime = SystemClock.uptimeMillis();
    TestResult testRunResult = new TestResult();
    ResultReporter resultPrinter;
    String outputFormat = mParams.getString("outputFormat");
    //获取所有的测试用例
    List<TestCase> testCases = collector.getTestCases();
    Bundle testRunOutput = new Bundle();
    //测试结果的输出格式
    if ("simple".equals(outputFormat)) {
        resultPrinter = new SimpleResultPrinter(System.out, true);
    } else {
        resultPrinter = new WatcherResultPrinter(testCases.size());
    }
    try {
        //是否以Monkey的方式运行
        automationWrapper.setRunAsMonkey(mMonkey);
        //初始化UiDevice
        mUiDevice = UiDevice.getInstance();
        mUiDevice.initialize(new ShellUiAutomatorBridge(automationWrapper.getUiAutomation()));
        //设置uiautomator内部的调试方式，如果是文件或者所有（文件和logcat),则需要有设置保存的文件名称参数
        String traceType = mParams.getString("traceOutputMode");
        if(traceType != null) {
            Tracer.Mode mode = Tracer.Mode.valueOf(Tracer.Mode.class, traceType);
            if (mode == Tracer.Mode.FILE || mode == Tracer.Mode.ALL) {
                String filename = mParams.getString("traceLogFilename");
                if (filename == null) {
                    throw new RuntimeException("Name of log file not specified. " +
                            "Please specify it using traceLogFilename parameter");
                }
                Tracer.getInstance().setOutputFilename(filename);
            }
            //启动Tracer
            Tracer.getInstance().setOutputMode(mode);
        }
        //添加打印case执行的结果信息
        // add test listeners
        testRunResult.addListener(resultPrinter);
        // add all custom listeners
        //添加其他的自定义测试结果信息监听器
        for (TestListener listener : mTestListeners) {
            testRunResult.addListener(listener);
        }
        //　开始分别运行所有的测试用例
        // run tests for realz!
        for (TestCase testCase : testCases) {
            prepareTestCase(testCase);
            testCase.run(testRunResult);
        }
    } catch (Throwable t) {
        // catch all exceptions so a more verbose error message can be outputted
        resultPrinter.printUnexpectedError(t);
        testRunOutput.putString("shortMsg", t.getMessage());
    } finally {
      　//打印测试的结果信息，并关闭所有的资源
        long runTime = SystemClock.uptimeMillis() - startTime;
        resultPrinter.print(testRunResult, runTime, testRunOutput);
        automationWrapper.disconnect();
        automationWrapper.setRunAsMonkey(false);
        mHandlerThread.quit();
    }
}
```

上面的start()方法能根据给定的测试类来添加所有的测试用例：

```java
protected TestCaseCollector getTestCaseCollector(ClassLoader classLoader) {
    return new TestCaseCollector(classLoader, getTestCaseFilter());
}

/**
 * Returns an object which determines if the class and its methods should be
 * accepted into the test suite.
 * @return
 */
public UiAutomatorTestCaseFilter getTestCaseFilter() {
    return new UiAutomatorTestCaseFilter();
}
```

这个下一节介绍，先来看下两个定义的测试结果信息监听器：

```java
private interface ResultReporter extends TestListener {
    public void print(TestResult result, long runTime, Bundle testOutput);
    public void printUnexpectedError(Throwable t);
}

/**
 * Class that produces the same output as JUnit when running from command line. Can be
 * used when default UiAutomator output is too verbose.
 */
private class SimpleResultPrinter extends ResultPrinter implements ResultReporter {
    private final boolean mFullOutput;
    public SimpleResultPrinter(PrintStream writer, boolean fullOutput) {
        super(writer);
        mFullOutput = fullOutput;
    }

    @Override
    public void print(TestResult result, long runTime, Bundle testOutput) {
        printHeader(runTime);
        if (mFullOutput) {
            printErrors(result);
            printFailures(result);
        }
        printFooter(result);
    }

    @Override
    public void printUnexpectedError(Throwable t) {
        if (mFullOutput) {
            getWriter().printf("Test run aborted due to unexpected exeption: %s",
                    t.getMessage());
            t.printStackTrace(getWriter());
        }
    }
}
```

SimpleResultPrinter是一个简单的测试结果打印类，ResultReporter接口定义了两个方法，print为打印正常的测试结果信息，如测试成功，或测试失败；printUnexpectedError为在测试出现了未捕获抛出异常后的处理操作．

```java
// Copy & pasted from InstrumentationTestRunner.WatcherResultPrinter
private class WatcherResultPrinter implements ResultReporter {

    private static final String REPORT_KEY_NUM_TOTAL = "numtests";
    private static final String REPORT_KEY_NAME_CLASS = "class";
    private static final String REPORT_KEY_NUM_CURRENT = "current";
    private static final String REPORT_KEY_NAME_TEST = "test";
    private static final String REPORT_KEY_NUM_ITERATIONS = "numiterations";
    private static final String REPORT_VALUE_ID = "UiAutomatorTestRunner";
    private static final String REPORT_KEY_STACK = "stack";

    private static final int REPORT_VALUE_RESULT_START = 1;
    private static final int REPORT_VALUE_RESULT_ERROR = -1;
    private static final int REPORT_VALUE_RESULT_FAILURE = -2;

    private final Bundle mResultTemplate;
    Bundle mTestResult;
    int mTestNum = 0;
    int mTestResultCode = 0;
    String mTestClass = null;

    private final SimpleResultPrinter mPrinter;
    private final ByteArrayOutputStream mStream;
    private final PrintStream mWriter;

    //构建函数需传入此次测试的用例总数
    public WatcherResultPrinter(int numTests) {
        mResultTemplate = new Bundle();
        mResultTemplate.putString(Instrumentation.REPORT_KEY_IDENTIFIER, REPORT_VALUE_ID);
        mResultTemplate.putInt(REPORT_KEY_NUM_TOTAL, numTests);

        mStream = new ByteArrayOutputStream();
        mWriter = new PrintStream(mStream);
        mPrinter = new SimpleResultPrinter(mWriter, false);
    }

    /**
     * send a status for the start of a each test, so long tests can be seen
     * as "running"
     */
     //开始执行测试用例时调用
    @Override
    public void startTest(Test test) {
        String testClass = test.getClass().getName();
        String testName = ((TestCase) test).getName();
        mTestResult = new Bundle(mResultTemplate);
        mTestResult.putString(REPORT_KEY_NAME_CLASS, testClass);
        mTestResult.putString(REPORT_KEY_NAME_TEST, testName);
        mTestResult.putInt(REPORT_KEY_NUM_CURRENT, ++mTestNum);
        // pretty printing
        if (testClass != null && !testClass.equals(mTestClass)) {
            mTestResult.putString(Instrumentation.REPORT_KEY_STREAMRESULT,
                    String.format("\n%s:", testClass));
            mTestClass = testClass;
        } else {
            mTestResult.putString(Instrumentation.REPORT_KEY_STREAMRESULT, "");
        }

        Method testMethod = null;
        try {
            testMethod = test.getClass().getMethod(testName);
            // Report total number of iterations, if test is repetitive
            if (testMethod.isAnnotationPresent(RepetitiveTest.class)) {
                int numIterations = testMethod.getAnnotation(RepetitiveTest.class)
                        .numIterations();
                mTestResult.putInt(REPORT_KEY_NUM_ITERATIONS, numIterations);
            }
        } catch (NoSuchMethodException e) {
            // ignore- the test with given name does not exist. Will be
            // handled during test
            // execution
        }
        //发送信息打印输出
        mAutomationSupport.sendStatus(REPORT_VALUE_RESULT_START, mTestResult);
        mTestResultCode = 0;

        mPrinter.startTest(test);
    }

    //测试错误时调用
    @Override
    public void addError(Test test, Throwable t) {
        mTestResult.putString(REPORT_KEY_STACK, BaseTestRunner.getFilteredTrace(t));
        mTestResultCode = REPORT_VALUE_RESULT_ERROR;
        // pretty printing
        mTestResult.putString(Instrumentation.REPORT_KEY_STREAMRESULT,
            String.format("\nError in %s:\n%s",
                ((TestCase)test).getName(), BaseTestRunner.getFilteredTrace(t)));

        mPrinter.addError(test, t);
    }

    //测试失败时调用
    @Override
    public void addFailure(Test test, AssertionFailedError t) {
        mTestResult.putString(REPORT_KEY_STACK, BaseTestRunner.getFilteredTrace(t));
        mTestResultCode = REPORT_VALUE_RESULT_FAILURE;
        // pretty printing
        mTestResult.putString(Instrumentation.REPORT_KEY_STREAMRESULT,
            String.format("\nFailure in %s:\n%s",
                ((TestCase)test).getName(), BaseTestRunner.getFilteredTrace(t)));

        mPrinter.addFailure(test, t);
    }

    //测试结束时调用
    @Override
    public void endTest(Test test) {
        if (mTestResultCode == 0) {
            mTestResult.putString(Instrumentation.REPORT_KEY_STREAMRESULT, ".");
        }
        mAutomationSupport.sendStatus(mTestResultCode, mTestResult);

        mPrinter.endTest(test);
    }

    //整个测试结束时调用
    @Override
    public void print(TestResult result, long runTime, Bundle testOutput) {
        mPrinter.print(result, runTime, testOutput);
        testOutput.putString(Instrumentation.REPORT_KEY_STREAMRESULT,
              String.format("\nTest results for %s=%s",
              getClass().getSimpleName(),
              mStream.toString()));
        mWriter.close();
        mAutomationSupport.sendStatus(Activity.RESULT_OK, testOutput);
    }

    //测试执行过程中有未捕获的异常抛出时调用
    @Override
    public void printUnexpectedError(Throwable t) {
        mWriter.println(String.format("Test run aborted due to unexpected exception: %s",
                t.getMessage()));
        t.printStackTrace(mWriter);
    }
}
```
