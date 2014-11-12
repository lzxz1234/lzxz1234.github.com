---
layout: post
title: 不依赖堆内存的异步计数器
category : java
tags : [Java, Concurrent]
---
{% include JB/setup %}

几乎所有的系统中都存在异步计数器，它们可能被用于收集数据，也有可能用于线程同步等。`JAVA` 在基于栈的异步计数器上已做了相当不错的支持。

但有时候你需要在不同的进程间同步计数。

## 如何构造线程间的计数器 ##

#### 数据库 ###
这是能想到的第一种实现方式，数据库的序列足矣在不同进程间同步。所有的同步操作都交给数据库处理。但我们知道这会极大增加数据库的负载（网络、锁等）。

#### 单独服务 ###
你可以开发一个服务或者中间件来实现这项功能。但这仍然会有网络延迟、编码解码等负载。

#### 内存映射文件 ###
这就是今天的主角。

## 多线程计数器中的难点 ##

### 数据可见性 ###
一个线程做的修改应该是对所有的线程都是可见的。这可以通过内存映射文件解决。操作系统和 `JAVA` 内存机制保证它的可行性。

### 线程安全 ###
计数器同时被很多线程修改，所以线程安全就是一个很大的问题。[Compare-and-swap](http://en.wikipedia.org/wiki/Compare-and-swap "Compare-and-swap") 能解决多线程写的问题。但能在内存之外使用 `CAS` 吗？答案是YES。通过内存映射和 `Unsafe` 就能实现内存外的 `CAS` 操作。

现在就让我们看下怎么实现。

## 解决方案 ##

### 如何获取内存地址 ###
因为 `MappedByteBuffer` 使用了 `DirectByteBuffer`，所以通过获取内存中的虚地址然后使用 `unsafe` 实现 `CAS` 操作是可行的。代码如下:

{% highlight java linenos %}
FileChannel fc = new RandomAccessFile(fileName, "rw").getChannel();
// Map 8 bytes for long value.
mem = fc.map(FileChannel.MapMode.READ_WRITE, 0, 8);
startAddress = ((DirectBuffer) mem).address();
{% endhighlight %}

这段代码申请了 8 字节的内存映射文件并获取到了它的虚地址。使用这个地址我们就能实现对文件的读写操作了。

### 如何线程安全的读写文件 ###
{% highlight java linenos %}
public boolean increment() {
	long orignalValue = readVolatile(startAddress);
	long value = convert(orignalValue);
	return UnsafeUtils.unsafe.compareAndSwapLong(null, 
			startAddress,orignalValue, convert(value + 1));
}

public long get() {
	long orignalValue = readVolatile(startAddress);
	return convert(orignalValue);
}

// Only unaligned is implemented
private static long readVolatile(long position) {
	if (UnsafeUtils.unaligned()) {
		return UnsafeUtils.unsafe.getLongVolatile(null, position);
	}
	throw new UnsupportedOperationException();
}

private static long convert(long a) {
	if (UnsafeUtils.unaligned()) {
		return (UnsafeUtils.nativeByteOrder ? a : Long.reverseBytes(a));
	}
	throw new UnsupportedOperationException();
}
{% endhighlight %}

需要注意的就就是 `readVolatile` 和 `increment`。`readVolatile` 直接从内存获得了读数，`increment` 使用
了 `unsafe` 对从 `MemoryByteBuffer` 获取的地址进行了 `CAS` 操作。

## 结论 ##
- 内存映射文件是很强大的，用它能实现很多强大的工具，像不依赖堆的集合，IPC，不依赖堆内存的线程同步等。
- 内存映射文件开启了无 GC 编程模式的大门。

转载注明出处：[{{page.title}}]({{permalink}})

[原文链接](http://ashkrit.blogspot.com/2014/03/off-heap-concurrent-counter.html "http://ashkrit.blogspot.com/2014/03/off-heap-concurrent-counter.html")