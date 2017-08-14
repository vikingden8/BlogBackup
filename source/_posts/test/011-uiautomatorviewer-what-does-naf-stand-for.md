---
title: What does NAF stand for on Android UiAutomatorViewer
date: 2017-08-14 10:15:18
tags:
categories: "测试开发"
---


**NAF** means **"Not Accessibility Friendly"**

These are UI elements which are apparently interactive, but which don't have accessibility affordances like content descriptions.

From the source for [AccessibilityNodeInfoDumper.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/AccessibilityNodeInfoDumper.java) (part of uiautomator):

<!--more-->

```java
/**
 * We're looking for UI controls that are enabled, clickable but have no
 * text nor content-description. Such controls configuration indicate an
 * interactive control is present in the UI and is most likely not
 * accessibility friendly. We refer to such controls here as NAF controls
 * (Not Accessibility Friendly)
 *
 * @param node
 * @return false if a node fails the check, true if all is OK
 */
private static boolean nafCheck(AccessibilityNodeInfo node) {
    boolean isNaf = node.isClickable() && node.isEnabled()
            && safeCharSeqToString(node.getContentDescription()).isEmpty()
            && safeCharSeqToString(node.getText()).isEmpty();
    if (!isNaf)
        return true;
    // check children since sometimes the containing element is clickable
    // and NAF but a child's text or description is available. Will assume
    // such layout as fine.
    return childNafCheck(node);
}
/**
 * This should be used when it's already determined that the node is NAF and
 * a further check of its children is in order. A node maybe a container
 * such as LinerLayout and may be set to be clickable but have no text or
 * content description but it is counting on one of its children to fulfill
 * the requirement for being accessibility friendly by having one or more of
 * its children fill the text or content-description. Such a combination is
 * considered by this dumper as acceptable for accessibility.
 *
 * @param node
 * @return false if node fails the check.
 */
private static boolean childNafCheck(AccessibilityNodeInfo node) {
    int childCount = node.getChildCount();
    for (int x = 0; x < childCount; x++) {
        AccessibilityNodeInfo childNode = node.getChild(x);
        if (childNode == null) {
            Log.i(LOGTAG, String.format("Null child %d/%d, parent: %s",
                    x, childCount, node.toString()));
            continue;
        }
        if (!safeCharSeqToString(childNode.getContentDescription()).isEmpty()
                || !safeCharSeqToString(childNode.getText()).isEmpty())
            return true;
        if (childNafCheck(childNode))
            return true;
    }
    return false;
}
private static String safeCharSeqToString(CharSequence cs) {
    if (cs == null)
        return "";
    else {
        return stripInvalidXMLChars(cs);
    }
}
```

### References

[uiautomatorviewer - What does NAF stand for?](https://stackoverflow.com/questions/25435878/uiautomatorviewer-what-does-naf-stand-for)

[AccessibilityNodeInfoDumper.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/AccessibilityNodeInfoDumper.java)
