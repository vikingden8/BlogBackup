---
title: Android UiAutomator UiDevice（一）
date: 2017-09-09 21:47:21
tags:
categories: "测试开发"
---

>UiDevice provides access to state information about the device. You can also use this class to simulate user actions on the device, such as pressing the d-pad or pressing the Home and Menu buttons.

上面是官方文档对UiDevice的定义，简单来说主要有两个用处：第一，提供获取设备状态信息的访问；第二，提供模拟用户动作，比如d
-pad按键或者点击Home和Menu菜单按键。

<!--more-->

```java
// reference to self
private static UiDevice sDevice;

private UiDevice() {
    /* hide constructor */
}

...

/**
 * Retrieves a singleton instance of UiDevice
 *
 * @return UiDevice instance
 * @since API Level 16
 */
public static UiDevice getInstance() {
    if (sDevice == null) {
        sDevice = new UiDevice();
    }
    return sDevice;
}

```

UiDevice的访问设计成了单例模式，在我们的测试用例代码中可以通过UiDevice.getInstance()来获取。

```java
/**
 * Enables or disables layout hierarchy compression.
 *
 * If compression is enabled, the layout hierarchy derived from the Acessibility
 * framework will only contain nodes that are important for uiautomator
 * testing. Any unnecessary surrounding layout nodes that make viewing
 * and searching the hierarchy inefficient are removed.
 *
 * @param compressed true to enable compression; else, false to disable
 * @since API Level 18
 */
public void setCompressedLayoutHeirarchy(boolean compressed) {
    getAutomatorBridge().setCompressedLayoutHierarchy(compressed);
}
```

setCompressedLayoutHeirarchy通过此方法来设置在获取当前界面对象的时候是否采用压缩布局，也就是Accessibility框架只提供对uiautomator有用的测试控件节点。
