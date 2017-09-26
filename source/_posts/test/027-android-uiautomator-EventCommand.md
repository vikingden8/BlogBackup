---
title: Android UiAutomator EventsCommand
date: 2017-09-26 7:31:48
tags:
categories: "测试开发"
---

这节介绍的是EventCommand类的具体作用

```java
/**
 * Implementation of the events subcommand
 *
 * Prints out accessibility events until process is stopped.
 */
public class EventsCommand extends Command {

    private Object mQuitLock = new Object();

    public EventsCommand() {
        super("events");
    }

    @Override
    public String shortHelp() {
        return "prints out accessibility events until terminated";
    }

    @Override
    public String detailedOptions() {
        return null;
    }

    @Override
    public void run(String[] args) {
        UiAutomationShellWrapper automationWrapper = new UiAutomationShellWrapper();
        automationWrapper.connect();
        automationWrapper.getUiAutomation().setOnAccessibilityEventListener(
                new OnAccessibilityEventListener() {
            @Override
            public void onAccessibilityEvent(AccessibilityEvent event) {
                SimpleDateFormat formatter = new SimpleDateFormat("MM-dd HH:mm:ss.SSS");
                System.out.println(String.format("%s %s",
                        formatter.format(new Date()), event.toString()));
            }
        });
        // there's really no way to stop, essentially we just block indefinitely here and wait
        // for user to press Ctrl+C
        synchronized (mQuitLock) {
            try {
                mQuitLock.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        automationWrapper.disconnect();
    }
}
```

<!--more-->

* EventsCommand继承自Launcher中的Command类，然后复写类父类中的抽象方法；

* run方法中主要构建了一个UiAutomationShellWrapper对象，然后设置了Accessibility的事件监听，在监听器里面格式化打印出了具体的AccessibilityEvent；

* 阻塞直到程序按下Ctrl+C，然后调用UiAutomationShellWrapper的disconnect方法断开和UiAutomation的连接．
