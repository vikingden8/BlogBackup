---
title: Android UiAutomator AccessibilityNodeInfoDumper
date: 2017-09-04 10:02:14
tags:
categories: "测试开发"
---

之前一直很好奇UiAutomatorViewer中选中某一个控件后出现的控件的信息是在哪里获取的，以及UiObject中的className，text，id或者content description等是在哪里得到的，其实可以通过今天的这个AccessibilityNodeInfoDumper类可以看到获取的过程。

```java
/**
 * Using {@link AccessibilityNodeInfo} this method will walk the layout hierarchy
 * and generates an xml dump into the /data/local/window_dump.xml
 * @param root The root accessibility node.
 * @param rotation The rotaion of current display
 * @param width The pixel width of current display
 * @param height The pixel height of current display
 */
public static void dumpWindowToFile(AccessibilityNodeInfo root, int rotation,
        int width, int height) {
    File baseDir = new File(Environment.getDataDirectory(), "local");
    if (!baseDir.exists()) {
        baseDir.mkdir();
        baseDir.setExecutable(true, false);
        baseDir.setWritable(true, false);
        baseDir.setReadable(true, false);
    }
    dumpWindowToFile(root,
            new File(new File(Environment.getDataDirectory(), "local"), "window_dump.xml"),
            rotation, width, height);
}
```

<!--more-->

首先看到这个dumpWindowToFile方法，这个方法的主要用途是遍历当前设备的界面布局，并且生成一个xml文件在设备的/data/local/window_dump.xml处。

参数为四个，root是当前界面的root控件，Accessibility其实是提供给有障碍访问的人提供的框架，rotation是当前设备的屏幕方向，width和height为设备的宽和高。

上方法的第一步是监测local目录是否存在，不存在的话，则创建这个目录并设置相关的权限。

然后调用另外一个重载的dumpWindowToFile方法来执行具体的dump操作。

```java
/**
 * Using {@link AccessibilityNodeInfo} this method will walk the layout hierarchy
 * and generates an xml dump to the location specified by <code>dumpFile</code>
 * @param root The root accessibility node.
 * @param dumpFile The file to dump to.
 * @param rotation The rotaion of current display
 * @param width The pixel width of current display
 * @param height The pixel height of current display
 */
public static void dumpWindowToFile(AccessibilityNodeInfo root, File dumpFile, int rotation,
        int width, int height) {
    if (root == null) {
        return;
    }
    final long startTime = SystemClock.uptimeMillis();
    try {
        FileWriter writer = new FileWriter(dumpFile);
        XmlSerializer serializer = Xml.newSerializer();
        StringWriter stringWriter = new StringWriter();
        serializer.setOutput(stringWriter);
        serializer.startDocument("UTF-8", true);
        serializer.startTag("", "hierarchy");
        serializer.attribute("", "rotation", Integer.toString(rotation));
        dumpNodeRec(root, serializer, 0, width, height);
        serializer.endTag("", "hierarchy");
        serializer.endDocument();
        writer.write(stringWriter.toString());
        writer.close();
    } catch (IOException e) {
        Log.e(LOGTAG, "failed to dump window to file", e);
    }
    final long endTime = SystemClock.uptimeMillis();
    Log.w(LOGTAG, "Fetch time: " + (endTime - startTime) + "ms");
}
```

在dump之前，先创建一个XML的序列化对象，然后调用dumpNodeRec方法循环dump界面的控件信息：

```java
private static void dumpNodeRec(AccessibilityNodeInfo node, XmlSerializer serializer,int index,
        int width, int height) throws IOException {
    serializer.startTag("", "node");
    if (!nafExcludedClass(node) && !nafCheck(node))
        serializer.attribute("", "NAF", Boolean.toString(true));
    serializer.attribute("", "index", Integer.toString(index));
    serializer.attribute("", "text", safeCharSeqToString(node.getText()));
    serializer.attribute("", "resource-id", safeCharSeqToString(node.getViewIdResourceName()));
    serializer.attribute("", "class", safeCharSeqToString(node.getClassName()));
    serializer.attribute("", "package", safeCharSeqToString(node.getPackageName()));
    serializer.attribute("", "content-desc", safeCharSeqToString(node.getContentDescription()));
    serializer.attribute("", "checkable", Boolean.toString(node.isCheckable()));
    serializer.attribute("", "checked", Boolean.toString(node.isChecked()));
    serializer.attribute("", "clickable", Boolean.toString(node.isClickable()));
    serializer.attribute("", "enabled", Boolean.toString(node.isEnabled()));
    serializer.attribute("", "focusable", Boolean.toString(node.isFocusable()));
    serializer.attribute("", "focused", Boolean.toString(node.isFocused()));
    serializer.attribute("", "scrollable", Boolean.toString(node.isScrollable()));
    serializer.attribute("", "long-clickable", Boolean.toString(node.isLongClickable()));
    serializer.attribute("", "password", Boolean.toString(node.isPassword()));
    serializer.attribute("", "selected", Boolean.toString(node.isSelected()));
    serializer.attribute("", "bounds", AccessibilityNodeInfoHelper.getVisibleBoundsInScreen(
            node, width, height).toShortString());
    int count = node.getChildCount();
    for (int i = 0; i < count; i++) {
        AccessibilityNodeInfo child = node.getChild(i);
        if (child != null) {
            if (child.isVisibleToUser()) {
                dumpNodeRec(child, serializer, i, width, height);
                child.recycle();
            } else {
                Log.i(LOGTAG, String.format("Skipping invisible child: %s", child.toString()));
            }
        } else {
            Log.i(LOGTAG, String.format("Null child %d/%d, parent: %s",
                    i, count, node.toString()));
        }
    }
    serializer.endTag("", "node");
}
```

通过这个方法可以看出 ，原来控件的信息都是通过AccessibilityNodeInfo对象来获取的，如getText(),getViewIdResourceName(),getClassName()等。

但是这个有一个要注意，里面有一个判断：

```java
if (!nafExcludedClass(node) && !nafCheck(node))
    serializer.attribute("", "NAF", Boolean.toString(true));
```

即控件信息的NAF值设置，NAF指的是 **Not Accessibility Friendly**，也就是访问不友好的控件，比如一个可点击的TextView的属性是可点击的并且设置了可用，但是这个控件的Content Description和Text都为空，则认为是不友好访问的。在前面的文章中其实也有介绍到，这里不再重复。
