---
title: FileChannel之异步关闭和中断机制
date: 2018-06-27 22:17:40
categories: [nio]
tags: [nio, FileChannel]
toc: [true]

---

FileChannel是程序与文件的连接通道，程序可通过FileChannel对文件进行I/O操作。除了具有Channel基本的读写功能外，FileChannel还具有异步关闭和中断机制。当一个线程对FileChannel执行I/O操作且处于阻塞状态时，其它线程可以通过关闭Channel来结束该线程的阻塞状态；同样，如果中断阻塞线程，那么Channel会被关闭。本文将讲解FileChannel异步关闭和中断机制的实现。
<!--more-->

### 类图一览
FileChannel异步关闭和中断机制是通过继承AbstractInterruptibleChannel类实现，类继承图如下：
![FileChannelImpl实现类图](/img/20180627/fileChannel_class.png)

### open标志和close()方法

channel的异步关闭实现很简单，提供一个open字段表示channel的开/闭状态，并且对外提供一个close()方法用于对open字段进行修改。implCloseChannel()方法实现了具体的关闭操作，对于FileChannelImpl，implCloseChannel执行的操作是：释放所有文件锁，终止阻塞在channel上的线程，关闭文件。

![FileChannelImpl close()](/img/20180627/fileChannel_close.png)

![FileChannelImpl close()](/img/20180627/fileChannel_impClose.png)

threads.signalAndWait()会调用NativeThread.signal()方法关闭阻塞线程。NativeThread.signal()是native方法，感兴趣的童鞋可以自行查看JVM源码。<br>
当线程A阻塞在某个I/O操作上，此时线程B调用了close()方法，name线程A会发生什么情况呢？答案是如果线程A的操作没有完成时，会抛出AsynchronousCloseException，异常的抛出是在end()方法中进行的，end()方法在接下来会讲到。

### begin()和end()
AbstractInterruptibleChannel中的begin()和end()方法是实现可Channel可中断机制的基础。FileChannel的I/O操作遵循以下编程模式：

```
boolean completed = false;
try {
	begin();
	completed = ....;    //执行阻塞I/O操作
	return ...;
} finally {
	end(completed);
}
```

在执行阻塞I/O操作前，begin()方法负责向调用线程注入Channel包含的Interruptible对象，I/O操作执行后把注入的Interruptible对象置为null，可以猜到Interruptible对象的作用是什么了吧。当执行I/O操作的线程处于阻塞状态时，调用线程的interrupt()中断阻塞线程，阻塞线程设置中断位，并且会调用被注入进来的Interruptible对象的interrupt()方法，Interruptible.interrupt()方法就是负责关闭Channel。<br>
总结来说，begin()方法负责把Channel中的Interruptible对象注入I/O调用线程，I/O调用线程被中断时，会回调Interruptible.interrupt()方法关闭Channel。

![AbstractInterruptibleChannel begin()](/img/20180627/fileChannel_begin.png)

![Thread interrupt()](/img/20180627/thread_interrupt.png)

### 总结与得到

* Channel的异步关闭功能通过open状态字段+状态更改方法close()方式实现
* Channel的中断关闭机制通过向调用线程注入对象+回调方式实现
* 思考此类使用场景



