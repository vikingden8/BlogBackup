---
title: Android视频录制－－屏幕录制
date: 2017-07-18 17:45:44
tags:
categories: "Android学习笔记"
---

简单说一下，一个视频的生成，最少要有以下两步：

* 视频的采集，比如摄像头，比如我们讲的MediaProjection，这一步最终的输出，通常是一个流

* 视频的编码压缩，这一步是对第一步中获取到的流做处理，编码可能采用硬编码，比如h264，也可能采用软编码，自己写编码逻辑，最终生成的是一个解码器（也就是我们通常说的播放器）可以解码（播放）的视频文件（比如mp4）

所以MediaProjection其实帮我们实现了第一步，也就是视频的采集，我们还需要自己来实现视频的编码。所幸Google给我们提供了另外一个类MediaCodec来实现视频的硬编码，而不需要我们自己写太多的逻辑。

废话不多说，直接上代码，首先，我们需要在开始编码之前，先做一下准备，定义我们要编码的格式等信息：

<!--more-->


```Java
//MediaFormat这个类是用来定义视频格式相关信息的
//video/avc,这里的avc是高级视频编码Advanced Video Coding
//mWidth和mHeight是视频的尺寸，这个尺寸不能超过视频采集时采集到的尺寸，否则会直接crash
MediaFormat format = MediaFormat.createVideoFormat("video/avc", mWidth, mHeight);

//COLOR_FormatSurface这里表明数据将是一个graphicbuffer元数据
format.setInteger(MediaFormat.KEY_COLOR_FORMAT,  MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface);  

//设置码率，通常码率越高，视频越清晰，但是对应的视频也越大，这个值我默认设置成了2000000，也就是通常所说的2M，这已经不低了，如果你不想录制这么清晰的，你可以设置成500000，也就是500k         
format.setInteger(MediaFormat.KEY_BIT_RATE, mBitRate);

//设置帧率，通常这个值越高，视频会显得越流畅，一般默认我设置成30，你最低可以设置成24，不要低于这个值，低于24会明显卡顿
format.setInteger(MediaFormat.KEY_FRAME_RATE, FRAME_RATE);

//IFRAME_INTERVAL是指的帧间隔，这是个很有意思的值，它指的是，关键帧的间隔时间。通常情况下，你设置成多少问题都不大。
//比如你设置成10，那就是10秒一个关键帧。但是，如果你有需求要做视频的预览，那你最好设置成1
//因为如果你设置成10，那你会发现，10秒内的预览都是一个截图
format.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, IFRAME_INTERVAL);

//创建一个MediaCodec的实例
MediaCodec mEncoder = MediaCodec.createEncoderByType("video/avc");

//定义这个实例的格式，也就是上面我们定义的format，其他参数不用过于关注
mEncoder.configure(format, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);

//这一步非常关键，它设置的，是MediaCodec的编码源，也就是说，我要告诉mEncoder，你给我解码哪些流。
//很出乎大家的意料，MediaCodec并没有要求我们传一个流文件进去，而是要求我们指定一个surface
//而这个surface，其实就是我们在上一讲MediaProjection中用来展示屏幕采集数据的surface
mSurface = mEncoder.createInputSurface();
mEncoder.start();
```

关于上面的mSurface，定义和使用的代码：

```java
Surface mSurface;
mMediaProjection.createVirtualDisplay(TAG + "-display",
                    mWidth, mHeight, mDpi, DisplayManager.VIRTUAL_DISPLAY_FLAG_PUBLIC,
                    mSurface, null, null);
```

可以看到，通过上面mSurface的串联，我们把mMediaProjection的输出内容放到了mSurface里面，而mSurface正是mEncoder的输入源，这样就完成了对mMediaProjection输出内容的编码，也就是屏幕采集数据的编码。

现在我们搞定编码的输入源（mSurface）问题了，下一步我们需要把编码后的内容输出到一个文件中去：

```Java
public class AvcEncoder {

private MediaCodec mediaCodec;
private BufferedOutputStream outputStream;

public AvcEncoder() {
    File f = new File(Environment.getExternalStorageDirectory(), "Download/video_encoded.264");
    touch (f);
    try {
        outputStream = new BufferedOutputStream(new FileOutputStream(f));
        Log.i("AvcEncoder", "outputStream initialized");
    } catch (Exception e){
        e.printStackTrace();
    }

    mediaCodec = MediaCodec.createEncoderByType("video/avc");
    MediaFormat mediaFormat = MediaFormat.createVideoFormat("video/avc", 320, 240);
    mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, 125000);
    mediaFormat.setInteger(MediaFormat.KEY_FRAME_RATE, 15);
    mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Planar);
    mediaFormat.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, 5);
    mediaCodec.configure(mediaFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);
    mediaCodec.start();
}

public void close() {
    try {
        mediaCodec.stop();
        mediaCodec.release();
        outputStream.flush();
        outputStream.close();
    } catch (Exception e){
        e.printStackTrace();
    }
}

// called from Camera.setPreviewCallbackWithBuffer(...) in other class
public void offerEncoder(byte[] input) {
    try {
        ByteBuffer[] inputBuffers = mediaCodec.getInputBuffers();
        ByteBuffer[] outputBuffers = mediaCodec.getOutputBuffers();
        int inputBufferIndex = mediaCodec.dequeueInputBuffer(-1);
        if (inputBufferIndex >= 0) {
            ByteBuffer inputBuffer = inputBuffers[inputBufferIndex];
            inputBuffer.clear();
            inputBuffer.put(input);
            mediaCodec.queueInputBuffer(inputBufferIndex, 0, input.length, 0, 0);
        }

        MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
        int outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo,0);
        while (outputBufferIndex >= 0) {
            ByteBuffer outputBuffer = outputBuffers[outputBufferIndex];
            byte[] outData = new byte[bufferInfo.size];
            outputBuffer.get(outData);
            outputStream.write(outData, 0, outData.length);
            Log.i("AvcEncoder", outData.length + " bytes written");

            mediaCodec.releaseOutputBuffer(outputBufferIndex, false);
            outputBufferIndex = mediaCodec.dequeueOutputBuffer(bufferInfo, 0);

        }
    } catch (Throwable t) {
        t.printStackTrace();
    }

}
```



原文地址：[Android视频录制－－屏幕录制](http://blog.csdn.net/l00149133/article/details/50483327)
