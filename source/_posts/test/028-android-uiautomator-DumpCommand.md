---
title: Android UiAutomator DumpCommand
date: 2017-09-26 19:31:48
tags:
categories: "测试开发"
---

DumpCommand是4个命令中的一个，它同样是uiautomator的一个非常重要的辅助命令，就比如说，我们在定位组件位置的时候，需要获取当前图片，还需要获取当前界面的dump文件，才能准确地知道每个组件的特性，每个组件之间的关系继承等等，这也是uiautomator框架中不可缺少的一部分。

<!--more-->

```java
/**
 * Implementation of the dump subcommand
 *
 * This creates an XML dump of current UI hierarchy
 */
public class DumpCommand extends Command {

    private static final File DEFAULT_DUMP_FILE = new File(
            Environment.getLegacyExternalStorageDirectory(), "window_dump.xml");

    public DumpCommand() {
        super("dump");
    }

    @Override
    public String shortHelp() {
        return "creates an XML dump of current UI hierarchy";
    }

    @Override
    public String detailedOptions() {
        return "    dump [--verbose][file]\n"
            + "      [--compressed]: dumps compressed layout information.\n"
            + "      [file]: the location where the dumped XML should be stored, default is\n      "
            + DEFAULT_DUMP_FILE.getAbsolutePath() + "\n";
    }

    @Override
    public void run(String[] args) {
        File dumpFile = DEFAULT_DUMP_FILE;
        boolean verboseMode = true;

        for (String arg : args) {
            if (arg.equals("--compressed"))
                verboseMode = false;
            else if (!arg.startsWith("-")) {
                dumpFile = new File(arg);
            }
        }

        UiAutomationShellWrapper automationWrapper = new UiAutomationShellWrapper();
        automationWrapper.connect();
        if (verboseMode) {
            // default
            automationWrapper.setCompressedLayoutHierarchy(false);
        } else {
            automationWrapper.setCompressedLayoutHierarchy(true);
        }

        // It appears that the bridge needs time to be ready. Making calls to the
        // bridge immediately after connecting seems to cause exceptions. So let's also
        // do a wait for idle in case the app is busy.
        try {
            UiAutomation uiAutomation = automationWrapper.getUiAutomation();
            uiAutomation.waitForIdle(1000, 1000 * 10);
            AccessibilityNodeInfo info = uiAutomation.getRootInActiveWindow();
            if (info == null) {
                System.err.println("ERROR: null root node returned by UiTestAutomationBridge.");
                return;
            }

            Display display =
                    DisplayManagerGlobal.getInstance().getRealDisplay(Display.DEFAULT_DISPLAY);
            int rotation = display.getRotation();
            Point size = new Point();
            display.getSize(size);
            AccessibilityNodeInfoDumper.dumpWindowToFile(info, dumpFile, rotation, size.x, size.y);
        } catch (TimeoutException re) {
            System.err.println("ERROR: could not get idle state.");
            return;
        } finally {
            automationWrapper.disconnect();
        }
        System.out.println(
                String.format("UI hierchary dumped to: %s", dumpFile.getAbsolutePath()));
    }
}
```

DumpCommand有两个参数，第一个--compressed指定是否dump压缩的布局信息，第二个即为dump后的控件信息保存的文件路径．

DEFAULT_DUMP_FILE的作用是如果命令中没有包含自定义的文件名，则会调用默认的文件名先来创建一个File对象。

构建UiAutomationShellWrapper对象后通过connect方法连接UiAutomation，然后使用方法setCompressedLayoutHierarchy设置是否使用压缩布局．

在dump之前，程序会等当前界面idle下来，也就是静止画面．会有10*1000ms的超时，然后通过uiAutomation的getRootInActiveWindow获取当前界面的根视图，如果不为空，则通过AccessibilityNodeInfoDumper的dumpWindowToFile进行真正的控件信息解析并保存到xml文件中．

在最终，所有try方法都安全地执行完成后，我们在最后的finally中还会与uiautomation对象完成断开连接的操作。在终端输出告知用户，这个文件存储的位置，该命令执行顺利完成。
