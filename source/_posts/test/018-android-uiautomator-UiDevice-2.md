---
title: Android UiAutomator UiDevice（二）
date: 2017-09-11 21:50:18
tags:
categories: "测试开发"
---

本节主要是对UiDevice的一些按键方法进行分析。

先看下Home，Back，Menu键的实现，源码如下：

```java
/**
 * Simulates a short press on the MENU button.
 * @return true if successful, else return false
 * @since API Level 16
 */
public boolean pressMenu() {
    Tracer.trace();
    waitForIdle();
    return getAutomatorBridge().getInteractionController().sendKeyAndWaitForEvent(
            KeyEvent.KEYCODE_MENU, 0, AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED,
            KEY_PRESS_EVENT_TIMEOUT);
}

/**
 * Simulates a short press on the BACK button.
 * @return true if successful, else return false
 * @since API Level 16
 */
public boolean pressBack() {
    Tracer.trace();
    waitForIdle();
    return getAutomatorBridge().getInteractionController().sendKeyAndWaitForEvent(
            KeyEvent.KEYCODE_BACK, 0, AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED,
            KEY_PRESS_EVENT_TIMEOUT);
}

/**
 * Simulates a short press on the HOME button.
 * @return true if successful, else return false
 * @since API Level 16
 */
public boolean pressHome() {
    Tracer.trace();
    waitForIdle();
    return getAutomatorBridge().getInteractionController().sendKeyAndWaitForEvent(
            KeyEvent.KEYCODE_HOME, 0, AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED,
            KEY_PRESS_EVENT_TIMEOUT);
}
```
<!--more-->

可以看出是调用了InteractionController类中的sendKeyAndWaitForEvent方法，追踪下代码：

```java
/**
 * Send keys and blocks until the first specified accessibility event.
 *
 * Most key presses will cause some UI change to occur. If the device is busy, this will
 * block until the device begins to process the key press at which point the call returns
 * and normal wait for idle processing may begin. If no events are detected for the
 * timeout period specified, the call will return anyway with false.
 *
 * @param keyCode
 * @param metaState
 * @param eventType
 * @param timeout
 * @return true if events is received, otherwise false.
 */
public boolean sendKeyAndWaitForEvent(final int keyCode, final int metaState,
        final int eventType, long timeout) {
    Runnable command = new Runnable() {
        @Override
        public void run() {
            final long eventTime = SystemClock.uptimeMillis();
            KeyEvent downEvent = new KeyEvent(eventTime, eventTime, KeyEvent.ACTION_DOWN,
                    keyCode, 0, metaState, KeyCharacterMap.VIRTUAL_KEYBOARD, 0, 0,
                    InputDevice.SOURCE_KEYBOARD);
            if (injectEventSync(downEvent)) {
                KeyEvent upEvent = new KeyEvent(eventTime, eventTime, KeyEvent.ACTION_UP,
                        keyCode, 0, metaState, KeyCharacterMap.VIRTUAL_KEYBOARD, 0, 0,
                        InputDevice.SOURCE_KEYBOARD);
                injectEventSync(upEvent);
            }
        }
    };

    return runAndWaitForEvents(command, new WaitForAnyEventPredicate(eventType), timeout)
            != null;
}
```

sendKeyAndWaitForEvent其实就是模拟一个KeyEvent事件，上面构造出了这个KeyCode的ACTION_DOWN和ACTION_UP的KeyEvent，分别通过injectEventSync执行，在执行的时候会等待屏幕的内容出现变化，直至超时，默认超时时间KEY_PRESS_EVENT_TIMEOUT为一秒.

injectEventSync方法调用了UiAutomatorBridge的injectInputEvent方法，

```java
private boolean injectEventSync(InputEvent event) {
    return mUiAutomatorBridge.injectInputEvent(event, true);
}
```

追踪代码：

```java
public boolean injectInputEvent(InputEvent event, boolean sync) {
    return mUiAutomation.injectInputEvent(event, sync);
}
```

最终调用的是UiAutomation的injectInputEvent，且第二个参数是否同步设置为了true，也就是会等KeyEvent阻塞执行完成后才返回。

```java
/**
 * A method for injecting an arbitrary input event.
 * <p>
 * <strong>Note:</strong> It is caller's responsibility to recycle the event.
 * </p>
 * @param event The event to inject.
 * @param sync Whether to inject the event synchronously.
 * @return Whether event injection succeeded.
 */
public boolean injectInputEvent(InputEvent event, boolean sync) {
    synchronized (mLock) {
        throwIfNotConnectedLocked();
    }
    try {
        if (DEBUG) {
            Log.i(LOG_TAG, "Injecting: " + event + " sync: " + sync);
        }
        // Calling out without a lock held.
        return mUiAutomationConnection.injectInputEvent(event, sync);
    } catch (RemoteException re) {
        Log.e(LOG_TAG, "Error while injecting input event!", re);
    }
    return false;
}
```

mUiAutomationConnection是IUiAutomationConnection，是一个AIDL。

```java
/**
 * This interface contains privileged operations a shell program can perform
 * on behalf of an instrumentation that it runs. These operations require
 * special permissions which the shell user has but the instrumentation does
 * not. Running privileged operations by the shell user on behalf of an
 * instrumentation is needed for running UiTestCases.
 *
 * {@hide}
 */
interface IUiAutomationConnection {
    void connect(IAccessibilityServiceClient client, int flags);
    void disconnect();
    boolean injectInputEvent(in InputEvent event, boolean sync);
    boolean setRotation(int rotation);
    Bitmap takeScreenshot(int width, int height);
    boolean clearWindowContentFrameStats(int windowId);
    WindowContentFrameStats getWindowContentFrameStats(int windowId);
    void clearWindowAnimationFrameStats();
    WindowAnimationFrameStats getWindowAnimationFrameStats();
    void executeShellCommand(String command, in ParcelFileDescriptor fd);
    void grantRuntimePermission(String packageName, String permission, int userId);
    void revokeRuntimePermission(String packageName, String permission, int userId);
    // Called from the system process.
    oneway void shutdown();
}
```

是一个接口的话肯定有实现对象，其实就是UiAutomationConnection，查看其实现：

```java
@Override
public boolean injectInputEvent(InputEvent event, boolean sync) {
    synchronized (mLock) {
        throwIfCalledByNotTrustedUidLocked();
        throwIfShutdownLocked();
        throwIfNotConnectedLocked();
    }
    final int mode = (sync) ? InputManager.INJECT_INPUT_EVENT_MODE_WAIT_FOR_FINISH
            : InputManager.INJECT_INPUT_EVENT_MODE_ASYNC;
    final long identity = Binder.clearCallingIdentity();
    try {
        return InputManager.getInstance().injectInputEvent(event, mode);
    } finally {
        Binder.restoreCallingIdentity(identity);
    }
}
```

原来在底层就是通过InputManager的injectInputEvent实现的KeyEvent。

除了Home，Back，Menu按键事件之外，我们再看下其他的事件：

```java
/**
* Simulates a short press on the SEARCH button.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressSearch() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_SEARCH);
}
/**
* Simulates a short press on the CENTER button.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressDPadCenter() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_DPAD_CENTER);
}
/**
* Simulates a short press on the DOWN button.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressDPadDown() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_DPAD_DOWN);
}
/**
* Simulates a short press on the UP button.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressDPadUp() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_DPAD_UP);
}
/**
* Simulates a short press on the LEFT button.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressDPadLeft() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_DPAD_LEFT);
}
/**
* Simulates a short press on the RIGHT button.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressDPadRight() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_DPAD_RIGHT);
}
/**
* Simulates a short press on the DELETE key.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressDelete() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_DEL);
}
/**
* Simulates a short press on the ENTER key.
* @return true if successful, else return false
* @since API Level 16
*/
public boolean pressEnter() {
   Tracer.trace();
   return pressKeyCode(KeyEvent.KEYCODE_ENTER);
}
```

上面的几类Key事件都是通过pressKeyCode方法来实现的，只是每一个事件的KeyCode不同而已，我们看下pressKeyCode方法的实现：

```java
/**
 * Simulates a short press using a key code.
 *
 * See {@link KeyEvent}
 * @return true if successful, else return false
 * @since API Level 16
 */
public boolean pressKeyCode(int keyCode) {
    Tracer.trace(keyCode);
    waitForIdle();
    return getAutomatorBridge().getInteractionController().sendKey(keyCode, 0);
}
/**
 * Simulates a short press using a key code.
 *
 * See {@link KeyEvent}.
 * @param keyCode the key code of the event.
 * @param metaState an integer in which each bit set to 1 represents a pressed meta key
 * @return true if successful, else return false
 * @since API Level 16
 */
public boolean pressKeyCode(int keyCode, int metaState) {
    Tracer.trace(keyCode, metaState);
    waitForIdle();
    return getAutomatorBridge().getInteractionController().sendKey(keyCode, metaState);
}

```

pressKeyCode有两个重载的方法，都是调用InteractionController类中的sendKey方法：

```java
public boolean sendKey(int keyCode, int metaState) {
    if (DEBUG) {
        Log.d(LOG_TAG, "sendKey (" + keyCode + ", " + metaState + ")");
    }
    final long eventTime = SystemClock.uptimeMillis();
    KeyEvent downEvent = new KeyEvent(eventTime, eventTime, KeyEvent.ACTION_DOWN,
            keyCode, 0, metaState, KeyCharacterMap.VIRTUAL_KEYBOARD, 0, 0,
            InputDevice.SOURCE_KEYBOARD);
    if (injectEventSync(downEvent)) {
        KeyEvent upEvent = new KeyEvent(eventTime, eventTime, KeyEvent.ACTION_UP,
                keyCode, 0, metaState, KeyCharacterMap.VIRTUAL_KEYBOARD, 0, 0,
                InputDevice.SOURCE_KEYBOARD);
        if(injectEventSync(upEvent)) {
            return true;
        }
    }
    return false;
}
```

实现其实跟之前Home，Back，Menu类似，只是这些KeyCode的事件没有等待界面的是否有更新的逻辑，最终也还是调用的injectEventSync方法，等待事件同步执行完成，并且返回事件是否执行的状态。

UiDevice还有几个通用的方法，比如打开消息通知栏，打开快速设置栏，打开最近任务栏：

```java
/**
 * Simulates a short press on the Recent Apps button.
 *
 * @return true if successful, else return false
 * @throws RemoteException
 * @since API Level 16
 */
public boolean pressRecentApps() throws RemoteException {
    Tracer.trace();
    waitForIdle();
    return getAutomatorBridge().getInteractionController().toggleRecentApps();
}
/**
 * Opens the notification shade.
 *
 * @return true if successful, else return false
 * @since API Level 18
 */
public boolean openNotification() {
    Tracer.trace();
    waitForIdle();
    return  getAutomatorBridge().getInteractionController().openNotification();
}
/**
 * Opens the Quick Settings shade.
 *
 * @return true if successful, else return false
 * @since API Level 18
 */
public boolean openQuickSettings() {
    Tracer.trace();
    waitForIdle();
    return getAutomatorBridge().getInteractionController().openNotification();
}
```
可以看到其分别调用了InteractionController的toggleRecentApps，openNotification，openNotification方法：

```java
/**
 * Simulates a short press on the Recent Apps button.
 *
 * @return true if successful, else return false
 * @since API Level 18
 */
public boolean toggleRecentApps() {
    return mUiAutomatorBridge.performGlobalAction(
            AccessibilityService.GLOBAL_ACTION_RECENTS);
}
/**
 * Opens the notification shade
 *
 * @return true if successful, else return false
 * @since API Level 18
 */
public boolean openNotification() {
    return mUiAutomatorBridge.performGlobalAction(
            AccessibilityService.GLOBAL_ACTION_NOTIFICATIONS);
}
/**
 * Opens the quick settings shade
 *
 * @return true if successful, else return false
 * @since API Level 18
 */
public boolean openQuickSettings() {
    return mUiAutomatorBridge.performGlobalAction(
            AccessibilityService.GLOBAL_ACTION_QUICK_SETTINGS);
}
```

而这三个方法分别调用了UiAutomatorBridge类的performGlobalAction方法，只是传入了不同的TAG来区分。

```java
public boolean performGlobalAction(int action) {
    return mUiAutomation.performGlobalAction(action);
}
```

再次到UiAutomation的performGlobalAction方法：

```java
/**
* Performs a global action. Such an action can be performed at any moment
* regardless of the current application or user location in that application.
* For example going back, going home, opening recents, etc.
*
* @param action The action to perform.
* @return Whether the action was successfully performed.
*
* @see android.accessibilityservice.AccessibilityService#GLOBAL_ACTION_BACK
* @see android.accessibilityservice.AccessibilityService#GLOBAL_ACTION_HOME
* @see android.accessibilityservice.AccessibilityService#GLOBAL_ACTION_NOTIFICATIONS
* @see android.accessibilityservice.AccessibilityService#GLOBAL_ACTION_RECENTS
*/
public final boolean performGlobalAction(int action) {
   final IAccessibilityServiceConnection connection;
   synchronized (mLock) {
       throwIfNotConnectedLocked();
       connection = AccessibilityInteractionClient.getInstance()
               .getConnection(mConnectionId);
   }
   // Calling out without a lock held.
   if (connection != null) {
       try {
           return connection.performGlobalAction(action);
       } catch (RemoteException re) {
           Log.w(LOG_TAG, "Error while calling performGlobalAction", re);
       }
   }
   return false;
}
```

调用了IAccessibilityServiceConnection的performGlobalAction，同样的IAccessibilityServiceConnection只是接口定义，我们查看其实现类的具体实现，通过查阅可以直到AccessibilityManagerService是其实现接口：

```java
/**
 * This class is instantiated by the system as a system level service and can be
 * accessed only by the system. The task of this service is to be a centralized
 * event dispatch for {@link AccessibilityEvent}s generated across all processes
 * on the device. Events are dispatched to {@link AccessibilityService}s.
 */
public class AccessibilityManagerService extends IAccessibilityManager.Stub {

  @Override
  public boolean performGlobalAction(int action) {
      synchronized (mLock) {
          if (!isCalledForCurrentUserLocked()) {
              return false;
          }
      }
      final long identity = Binder.clearCallingIdentity();
      try {
          mPowerManager.userActivity(SystemClock.uptimeMillis(),
                  PowerManager.USER_ACTIVITY_EVENT_ACCESSIBILITY, 0);
          switch (action) {
              case AccessibilityService.GLOBAL_ACTION_BACK: {
                  sendDownAndUpKeyEvents(KeyEvent.KEYCODE_BACK);
              } return true;
              case AccessibilityService.GLOBAL_ACTION_HOME: {
                  sendDownAndUpKeyEvents(KeyEvent.KEYCODE_HOME);
              } return true;
              case AccessibilityService.GLOBAL_ACTION_RECENTS: {
                  return openRecents();
              }
              case AccessibilityService.GLOBAL_ACTION_NOTIFICATIONS: {
                  expandNotifications();
              } return true;
              case AccessibilityService.GLOBAL_ACTION_QUICK_SETTINGS: {
                  expandQuickSettings();
              } return true;
              case AccessibilityService.GLOBAL_ACTION_POWER_DIALOG: {
                  showGlobalActions();
              } return true;
              case AccessibilityService.GLOBAL_ACTION_TOGGLE_SPLIT_SCREEN: {
                  toggleSplitScreen();
              } return true;
          }
          return false;
      } finally {
          Binder.restoreCallingIdentity(identity);
      }
  }

  private void sendDownAndUpKeyEvents(int keyCode) {
            final long token = Binder.clearCallingIdentity();
            // Inject down.
            final long downTime = SystemClock.uptimeMillis();
            KeyEvent down = KeyEvent.obtain(downTime, downTime, KeyEvent.ACTION_DOWN, keyCode, 0, 0,
                    KeyCharacterMap.VIRTUAL_KEYBOARD, 0, KeyEvent.FLAG_FROM_SYSTEM,
                    InputDevice.SOURCE_KEYBOARD, null);
            InputManager.getInstance().injectInputEvent(down,
                    InputManager.INJECT_INPUT_EVENT_MODE_ASYNC);
            down.recycle();
            // Inject up.
            final long upTime = SystemClock.uptimeMillis();
            KeyEvent up = KeyEvent.obtain(downTime, upTime, KeyEvent.ACTION_UP, keyCode, 0, 0,
                    KeyCharacterMap.VIRTUAL_KEYBOARD, 0, KeyEvent.FLAG_FROM_SYSTEM,
                    InputDevice.SOURCE_KEYBOARD, null);
            InputManager.getInstance().injectInputEvent(up,
                    InputManager.INJECT_INPUT_EVENT_MODE_ASYNC);
            up.recycle();
            Binder.restoreCallingIdentity(token);
        }

        private void expandNotifications() {
            final long token = Binder.clearCallingIdentity();
            StatusBarManager statusBarManager = (StatusBarManager) mContext.getSystemService(
                    android.app.Service.STATUS_BAR_SERVICE);
            statusBarManager.expandNotificationsPanel();
            Binder.restoreCallingIdentity(token);
        }

        private void expandQuickSettings() {
            final long token = Binder.clearCallingIdentity();
            StatusBarManager statusBarManager = (StatusBarManager) mContext.getSystemService(
                    android.app.Service.STATUS_BAR_SERVICE);
            statusBarManager.expandSettingsPanel();
            Binder.restoreCallingIdentity(token);
        }

        private boolean openRecents() {
            final long token = Binder.clearCallingIdentity();
            try {
                StatusBarManagerInternal statusBarService = LocalServices.getService(
                        StatusBarManagerInternal.class);
                if (statusBarService == null) {
                    return false;
                }
                statusBarService.toggleRecentApps();
            } finally {
                Binder.restoreCallingIdentity(token);
            }
            return true;
        }
}

```

继续追踪下去，都是调用了StatusBarManager或者StatusBarManagerInternal中的具体方法：

```java
/**
 * Expand the notifications panel.
 */
public void expandNotificationsPanel() {
    try {
        final IStatusBarService svc = getService();
        if (svc != null) {
            svc.expandNotificationsPanel();
        }
    } catch (RemoteException ex) {
        throw ex.rethrowFromSystemServer();
    }
}
/**
  * Expand the settings panel.
  */
 public void expandSettingsPanel() {
     expandSettingsPanel(null);
 }
 /**
  * Expand the settings panel and open a subPanel, pass null to just open the settings panel.
  */
 public void expandSettingsPanel(String subPanel) {
     try {
         final IStatusBarService svc = getService();
         if (svc != null) {
             svc.expandSettingsPanel(subPanel);
         }
     } catch (RemoteException ex) {
         throw ex.rethrowFromSystemServer();
     }
 }
```

转到了IStatusBarService的实现类StatusBarManagerService中的方法：

```java
// ================================================================================
// From IStatusBarService
// ================================================================================
public void expandNotificationsPanel() {
    enforceExpandStatusBar();
    if (mBar != null) {
        try {
            mBar.animateExpandNotificationsPanel();
        } catch (RemoteException ex) {
        }
    }
}
public void collapsePanels() {
    enforceExpandStatusBar();
    if (mBar != null) {
        try {
            mBar.animateCollapsePanels();
        } catch (RemoteException ex) {
        }
    }
}
public void expandSettingsPanel() {
    enforceExpandStatusBar();
    if (mBar != null) {
        try {
            mBar.animateExpandSettingsPanel();
        } catch (RemoteException ex) {
        }
    }
}

private void enforceExpandStatusBar() {
    mContext.enforceCallingOrSelfPermission(android.Manifest.permission.EXPAND_STATUS_BAR,
            "StatusBarManagerService");
}

```

先调用enforceExpandStatusBar方法，然后执行StatusBar中的打开各自功能动画，至此UiDevice的所有按键相关已分析完毕。

源码链接：

* [UiDevice.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiDevice.java)

* [InteractionController.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/InteractionController.java)

* [UiAutomatorBridge.java](https://android.googlesource.com/platform/frameworks/testing/+/master/uiautomator/library/core-src/com/android/uiautomator/core/UiAutomatorBridge.java)

* [UiAutomationConnection.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/UiAutomationConnection.java)

* [IUiAutomationConnection.aidl](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/IUiAutomationConnection.aidl)

* [UiAutomation.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/UiAutomation.java)

* [AccessibilityService.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/accessibilityservice/AccessibilityService.java)

* [IAccessibilityServiceConnection.aidl](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/accessibilityservice/IAccessibilityServiceConnection.aidl)

* [AccessibilityManagerService.java](https://android.googlesource.com/platform/frameworks/base/+/master/services/accessibility/java/com/android/server/accessibility/AccessibilityManagerService.java)

* [StatusBarManager.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/StatusBarManager.java)

* [IStatusBarService.aidl](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/com/android/internal/statusbar/IStatusBarService.aidl)

* [StatusBarManagerService.java](https://android.googlesource.com/platform/frameworks/base/+/a029ea1/services/java/com/android/server/StatusBarManagerService.java)
