---
title: Android ddmlib执行shell命令分析
date: 2017-07-23 22:24:22
tags:
categories: "测试开发"
---

在使用ddmlib时，我们可以通过IDevice对象执行具体的shell命令，那么是通过什么实现的呢？

首先我们找到IDevice.java源码，路径：[IDevice](https://android.googlesource.com/platform/tools/base/+/master/ddmlib/src/main/java/com/android/ddmlib/IDevice.java).
找到其中执行shell命令的方法定义：

```java
/**
 * @deprecated Use {@link #executeShellCommand(String, IShellOutputReceiver, long, java.util.concurrent.TimeUnit)}.
 */
@Deprecated
void executeShellCommand(String command, IShellOutputReceiver receiver,
        int maxTimeToOutputResponse)
        throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
        IOException;
/**
 * Executes a shell command on the device, and sends the result to a <var>receiver</var>
 * <p/>This is similar to calling
 * <code>executeShellCommand(command, receiver, DdmPreferences.getTimeOut())</code>.
 *
 * @param command the shell command to execute
 * @param receiver the {@link IShellOutputReceiver} that will receives the output of the shell
 *            command
 * @throws TimeoutException in case of timeout on the connection.
 * @throws AdbCommandRejectedException if adb rejects the command
 * @throws ShellCommandUnresponsiveException in case the shell command doesn't send output
 *            for a given time.
 * @throws IOException in case of I/O error on the connection.
 *
 * @see #executeShellCommand(String, IShellOutputReceiver, int)
 * @see DdmPreferences#getTimeOut()
 */
void executeShellCommand(String command, IShellOutputReceiver receiver)
        throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
        IOException;
```

<!--more-->

command代表的是要执行的shell命令
receiver代表的是执行的shell命令输出，如果不需要输出的化，可以为[NullOutputReceiver](https://android.googlesource.com/platform/tools/base/+/master/ddmlib/src/main/java/com/android/ddmlib/NullOutputReceiver.java)

上面的两个参数都是两个方法相同的，其中第一个方法还有一个maxTimeToOutputResponse参数，代表的是当前执行的命令最后的输出响应时间，如果在指定的时间内命令没有任何输出的话，则会抛出ShellCommandUnresponsiveException异常。

其实发现，第一个方法标记为了Deprecated，也就是说不建议使用，在doc中说明建议使用方法 _executeShellCommand(String, IShellOutputReceiver, long, java.util.concurrent.TimeUnit)_，但是在IDevice方法中并没有发现上述方法的定义，可以看到IDevice其实是继承自IShellEnabledDevice：

```java
/**
 *  A Device. It can be a physical device or an emulator.
 */
public interface IDevice extends IShellEnabledDevice {
...
}
```

上述的方法肯定在IShellEnabledDevice接口类中有定义，打开，果然如此：

```java
/**
 * Executes a shell command on the device, and sends the result to a <var>receiver</var>.
 * <p/><var>maxTimeToOutputResponse</var> is used as a maximum waiting time when expecting the
 * command output from the device.<br>
 * At any time, if the shell command does not output anything for a period longer than
 * <var>maxTimeToOutputResponse</var>, then the method will throw
 * {@link ShellCommandUnresponsiveException}.
 * <p/>For commands like log output, a <var>maxTimeToOutputResponse</var> value of 0, meaning
 * that the method will never throw and will block until the receiver's
 * {@link IShellOutputReceiver#isCancelled()} returns <code>true</code>, should be
 * used.
 *
 * @param command the shell command to execute
 * @param receiver the {@link IShellOutputReceiver} that will receives the output of the shell
 *            command
 * @param maxTimeToOutputResponse the maximum amount of time during which the command is allowed
 *            to not output any response. A value of 0 means the method will wait forever
 *            (until the <var>receiver</var> cancels the execution) for command output and
 *            never throw.
 * @param maxTimeUnits Units for non-zero {@code maxTimeToOutputResponse} values.
 * @throws TimeoutException in case of timeout on the connection when sending the command.
 * @throws AdbCommandRejectedException if adb rejects the command.
 * @throws ShellCommandUnresponsiveException in case the shell command doesn't send any output
 *            for a period longer than <var>maxTimeToOutputResponse</var>.
 * @throws IOException in case of I/O error on the connection.
 *
 * @see DdmPreferences#getTimeOut()
 */
void executeShellCommand(String command, IShellOutputReceiver receiver,
        long maxTimeToOutputResponse, TimeUnit maxTimeUnits)
        throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
        IOException;
```

相对于IDevice中的连个executeShellCommand重载方法，此方法在参数签名上多了个maxTimeUnits也就是第三个参数maxTimeToOutputResponse的单位，之前版本默认是ms。

IDevice只是一个接口，肯定会有具体的实现，其实具体的实现类就是[Device](https://android.googlesource.com/platform/tools/base/+/master/ddmlib/src/main/java/com/android/ddmlib/Device.java)。来看一下Device中对 _executeShellCommand_ 三个重载方法的具体实现：

```java
@Override
public void executeShellCommand(String command, IShellOutputReceiver receiver)
        throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
        IOException {
    AdbHelper.executeRemoteCommand(AndroidDebugBridge.getSocketAddress(), command, this,
            receiver, DdmPreferences.getTimeOut());
}
@Override
public void executeShellCommand(String command, IShellOutputReceiver receiver,
        int maxTimeToOutputResponse)
        throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
        IOException {
    AdbHelper.executeRemoteCommand(AndroidDebugBridge.getSocketAddress(), command, this,
            receiver, maxTimeToOutputResponse);
}
@Override
public void executeShellCommand(String command, IShellOutputReceiver receiver,
        long maxTimeToOutputResponse, TimeUnit maxTimeUnits)
        throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
        IOException {
    AdbHelper.executeRemoteCommand(AndroidDebugBridge.getSocketAddress(), command, this,
            receiver, maxTimeToOutputResponse, maxTimeUnits);
}
```

这三个方法统一都是调用 _[AdbHelper](https://android.googlesource.com/platform/tools/base/+/master/ddmlib/src/main/java/com/android/ddmlib/AdbHelper.java)_ 中的 _executeRemoteCommand_ 方法，查看下其源码实现：

```java
/**
 * @deprecated Use {@link #executeRemoteCommand(java.net.InetSocketAddress, String, IDevice, IShellOutputReceiver, long, java.util.concurrent.TimeUnit)}.
 */
@Deprecated
static void executeRemoteCommand(InetSocketAddress adbSockAddr,
    String command, IDevice device, IShellOutputReceiver rcvr, int maxTimeToOutputResponse)
    throws TimeoutException, AdbCommandRejectedException, ShellCommandUnresponsiveException,
    IOException {
    executeRemoteCommand(adbSockAddr, command, device, rcvr, maxTimeToOutputResponse,
            TimeUnit.MILLISECONDS);
}
/**
 * Executes a shell command on the device and retrieve the output. The output is
 * handed to <var>rcvr</var> as it arrives.
 *
 * @param adbSockAddr the {@link InetSocketAddress} to adb.
 * @param command the shell command to execute
 * @param device the {@link IDevice} on which to execute the command.
 * @param rcvr the {@link IShellOutputReceiver} that will receives the output of the shell
 *            command
 * @param maxTimeToOutputResponse max time between command output. If more time passes
 *            between command output, the method will throw
 *            {@link ShellCommandUnresponsiveException}. A value of 0 means the method will
 *            wait forever for command output and never throw.
 * @param maxTimeUnits Units for non-zero {@code maxTimeToOutputResponse} values.
 * @throws TimeoutException in case of timeout on the connection when sending the command.
 * @throws AdbCommandRejectedException if adb rejects the command
 * @throws ShellCommandUnresponsiveException in case the shell command doesn't send any output
 *            for a period longer than <var>maxTimeToOutputResponse</var>.
 * @throws IOException in case of I/O error on the connection.
 *
 * @see DdmPreferences#getTimeOut()
 */
static void executeRemoteCommand(InetSocketAddress adbSockAddr,
    String command, IDevice device, IShellOutputReceiver rcvr, long maxTimeToOutputResponse,
    TimeUnit maxTimeUnits) throws TimeoutException, AdbCommandRejectedException,
    ShellCommandUnresponsiveException, IOException {
    executeRemoteCommand(adbSockAddr, AdbService.SHELL, command, device, rcvr, maxTimeToOutputResponse,
            maxTimeUnits, null /* inputStream */);
}
/**
 * Identify which adb service the command should target.
 */
public enum AdbService {
    /**
     * the shell service
     */
    SHELL,
    /**
     * The exec service.
     */
    EXEC
}
/**
 * Executes a remote command on the device and retrieve the output. The output is
 * handed to <var>rcvr</var> as it arrives. The command is execute by the remote service
 * identified by the adbService parameter.
 *
 * @param adbSockAddr the {@link InetSocketAddress} to adb.
 * @param adbService the {@link com.android.ddmlib.AdbHelper.AdbService} to use to run the
 *                   command.
 * @param command the shell command to execute
 * @param device the {@link IDevice} on which to execute the command.
 * @param rcvr the {@link IShellOutputReceiver} that will receives the output of the shell
 *            command
 * @param maxTimeToOutputResponse max time between command output. If more time passes
 *            between command output, the method will throw
 *            {@link ShellCommandUnresponsiveException}. A value of 0 means the method will
 *            wait forever for command output and never throw.
 * @param maxTimeUnits Units for non-zero {@code maxTimeToOutputResponse} values.
 * @param is a optional {@link InputStream} to be streamed up after invoking the command
 *           and before retrieving the response.
 * @throws TimeoutException in case of timeout on the connection when sending the command.
 * @throws AdbCommandRejectedException if adb rejects the command
 * @throws ShellCommandUnresponsiveException in case the shell command doesn't send any output
 *            for a period longer than <var>maxTimeToOutputResponse</var>.
 * @throws IOException in case of I/O error on the connection.
 *
 * @see DdmPreferences#getTimeOut()
 */
static void executeRemoteCommand(InetSocketAddress adbSockAddr, AdbService adbService,
        String command, IDevice device, IShellOutputReceiver rcvr, long maxTimeToOutputResponse,
        TimeUnit maxTimeUnits,
        @Nullable InputStream is) throws TimeoutException, AdbCommandRejectedException,
        ShellCommandUnresponsiveException, IOException {
    //如果我们这设置了maxTimeToOutputResponse，则maxTimeUnits不应为null
    //然后通过TTimeUnit统一转换为ms
    long maxTimeToOutputMs = 0;
    if (maxTimeToOutputResponse > 0) {
        if (maxTimeUnits == null) {
            throw new NullPointerException("Time unit must not be null for non-zero max.");
        }
        maxTimeToOutputMs = maxTimeUnits.toMillis(maxTimeToOutputResponse);
    }
    Log.v("ddms", "execute: running " + command);
    //具体的命令执行逻辑
    SocketChannel adbChan = null;
    try {
        //首先打开adb的socket通道，并设置为不阻塞
        adbChan = SocketChannel.open(adbSockAddr);
        adbChan.configureBlocking(false);
        // if the device is not -1, then we first tell adb we're looking to
        // talk
        // to a specific device
        //设置要执行的设备
        setDevice(adbChan, device);

        byte[] request = formAdbRequest(adbService.name().toLowerCase() + ":" + command); //$NON-NLS-1$
        write(adbChan, request);

        AdbResponse resp = readAdbResponse(adbChan, false /* readDiagString */);
        if (!resp.okay) {
            Log.e("ddms", "ADB rejected shell command (" + command + "): " + resp.message);
            throw new AdbCommandRejectedException(resp.message);
        }

        //组装执行shell命令的adb请求字节数组
        byte[] data = new byte[16384];
        // stream the input file if present.
        if (is != null) {
            int read;
            while ((read = is.read(data)) != -1) {
                ByteBuffer buf = ByteBuffer.wrap(data, 0, read);
                int written = 0;
                while (buf.hasRemaining()) {
                    written += adbChan.write(buf);
                }
                if (written != read) {
                    Log.e("ddms",
                            "ADB write inconsistency, wrote " + written + "expected " + read);
                    throw new AdbCommandRejectedException("write failed");
                }
            }
        }
        ByteBuffer buf = ByteBuffer.wrap(data);
        buf.clear();
        long timeToResponseCount = 0;
        //循环读取shell命令的输出
        while (true) {
            int count;
            if (rcvr != null && rcvr.isCancelled()) {
                Log.v("ddms", "execute: cancelled");
                break;
            }
            count = adbChan.read(buf);
            if (count < 0) {
                //shell命令已执行完毕
                // we're at the end, we flush the output
                rcvr.flush();
                Log.v("ddms", "execute '" + command + "' on '" + device + "' : EOF hit. Read: "
                        + count);
                break;
            } else if (count == 0) {
                //当每次读到的内容为空时，检查是否超时
                try {
                    int wait = WAIT_TIME * 5;
                    timeToResponseCount += wait;
                    if (maxTimeToOutputMs > 0 && timeToResponseCount > maxTimeToOutputMs) {
                        throw new ShellCommandUnresponsiveException();
                    }
                    Thread.sleep(wait);
                } catch (InterruptedException ie) {
                }
            } else {
                //读到的内容不为空，首先重置超时时间的计数器，然后写入到IShellOutputReceiver中
                // reset timeout
                timeToResponseCount = 0;
                // send data to receiver if present
                if (rcvr != null) {
                    rcvr.addOutput(buf.array(), buf.arrayOffset(), buf.position());
                }
                buf.rewind();
            }
        }
    } finally {
        if (adbChan != null) {
            adbChan.close();
        }
        Log.v("ddms", "execute: returning");
    }
}

```

解析都写在注释中了，这里需要注意的就是，当使用ddmlib开发工具时，经常会碰到 [ShellCommandUnresponsiveException](https://android.googlesource.com/platform/tools/base/+/master/ddmlib/src/main/java/com/android/ddmlib/ShellCommandUnresponsiveException.java) 异常出现，也就是执行shell命令超时了。这个异常类的定义如下：

```java
/**
 * Exception thrown when a shell command executed on a device takes too long to send its output.
 * <p/>The command may not actually be unresponsive, it just has spent too much time not outputting
 * any thing to the console.
 */
public class ShellCommandUnresponsiveException extends Exception {
    private static final long serialVersionUID = 1L;
}
```

也就是说有时候并不是命令无响应，而是执行的命令长时间无输出也会导致此异常的输出。如果是第二种情况，可以在执行命令时指定 _maxTimeToOutputResponse_ 为 **0** 即可。

那如果没有设置 _maxTimeToOutputResponse_，默认值就是从 _[DdmPreferences](https://android.googlesource.com/platform/tools/base/+/master/ddmlib/src/main/java/com/android/ddmlib/DdmPreferences.java)_ 的 _getTimeOut()_ 得到，查看下其源码：

```java
/**
 * Returns the timeout to be used in adb connections (milliseconds).
 */
public static int getTimeOut() {
    return sTimeOut;
}
/**
 * Sets the timeout value for adb connection.
 * <p/>This change takes effect for newly created connections only.
 * @param timeOut the timeout value (milliseconds).
 */
public static void setTimeOut(int timeOut) {
    sTimeOut = timeOut;
}
```

其实就是读取的类局部变量 _sTimeOut_ 的值，查看下其的定义：

```java
   ...
   /** Default timeout values for adb connection (milliseconds) */
   public static final int DEFAULT_TIMEOUT = 5000; // standard delay, in ms
   ...
   private static int sTimeOut = DEFAULT_TIMEOUT;
   ...
```

从上可以看出，如果不设置 _maxTimeToOutputResponse_，默认就是5秒的超时。如果我们想统一为每一条shell命令设置超时的时间，可以通过 _DdmPreferences_ 的 _setTimeOut(int timeOut)_ 方法指定即可。
