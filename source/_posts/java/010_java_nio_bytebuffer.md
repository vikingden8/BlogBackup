---
title: 图解ByteBuffer
date: 2017-03-04 22:17:55
tags:
categories: "Java学习笔记"
---

ByteBuffer前前后后看过好几次了，实际使用也用了一些，总觉得条理不够清晰。

《程序员的思维修炼》一本书讲过，主动学习，要比单纯看资料效果来的好，所以干脆写个详细点的文章来记录一下。

### 概述

ByteBuffer是NIO里用得最多的Buffer，它包含两个实现方式：HeapByteBuffer是基于Java堆的实现，而DirectByteBuffer则使用了unsafe的API进行了堆外的实现。这里只说HeapByteBuffer。

### 使用

ByteBuffer最核心的方法是 _put(byte)_ 和 _get()_。分别是往ByteBuffer里写一个字节，和读一个字节。值得注意的是，ByteBuffer的读写模式是分开的，正常的应用场景是：往ByteBuffer里写一些数据，然后flip()，然后再读出来。

这里插两个Channel方面的对象，以便更好的理解Buffer。ReadableByteChannel是一个从Channel中读取数据，并保存到ByteBuffer的接口，它包含一个方法：

```java
public int read(ByteBuffer dst) throws IOException;
```

WritableByteChannel则是从ByteBuffer中读取数据，并输出到Channel的接口：

```java
public int write(ByteBuffer src) throws IOException;
```

那么，一个ByteBuffer的使用过程是这样的：

```java
byteBuffer = ByteBuffer.allocate(N);
//读取数据，写入byteBuffer
readableByteChannel.read(byteBuffer);
//变读为写
byteBuffer.flip();
//读取byteBuffer，写入数据
writableByteChannel.write(byteBuffer);
```

看到这里，一般都不太明白flip()干了什么事，先从ByteBuffer结构说起：

#### ByteBuffer内部字段

**byte[] buff**

buff即内部用于缓存的数组。

**position**

当前读取的位置。

**mark**

为某一读过的位置做标记，便于某些时候回退到该位置。

**capacity**

初始化时候的容量。

**limit**

读写的上限，limit<=capacity。

### 图解

**put**

写模式下，往buffer里写一个字节，并把postion移动一位。写模式下，一般limit与capacity相等。

![](/images/categories/java/010/bytebuffer_write.png)

**flip**

写完数据，需要开始读的时候，将postion复位到0，并将limit设为当前postion。

![](/images/categories/java/010/bytebuffer_flip.png)

**get**

从buffer里读一个字节，并把postion移动一位。上限是limit，即写入数据的最后位置。

![](/images/categories/java/010/bytebuffer_get.png)

**clear**

将position置为0，并不清除buffer内容

![](/images/categories/java/010/bytebuffer_clear.png)
