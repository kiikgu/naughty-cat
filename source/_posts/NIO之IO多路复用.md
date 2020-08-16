---
title: NIO之IO多路复用
date: 2018-07-26 22:03:22
categories: [nio]
tags: [NIO,blocking I/O,nonblocking I/O,I/O multipexer, signal driven I/O, aynchronous I/O,阻塞I/O,非阻塞I/O,I/O多路复用,异步I/O]
toc: true

---

NIO中的Selector是对IO多路复用的抽象，理解IO多路复用可以帮助我们理解Java Selector机制的实现，本文主要介绍IO多路复用技术，几种IO模型，Unix对这几种IO模型的实现机制。

<!--more-->

### 简单IO场景
一个线程串行执行两个IO操作：读本地文件，请求远端server。当线程执行第一个操作时发生了block，而此时Server返回消息。那么这种情况下线程只有等待block结束，执行完第一个IO操作后，才会接受到server传回的信息。显然这种模式不能及时处理多个IO操作，需要一种机制能够监控到每个IO操作是否处于ready状态，对与ready状态的IO能迅速处理，IO多路复用技术就是这样的机制。

### I/O模型
Unix下有五种I/O模型：阻塞I/O(blocking I/O)、非阻塞I/O(nonblocking I/O)、I/O多路复用(select and poll)、信号驱动I/O(signal driven I/O)、异步I/O(aynchronous I/O)，下面通过分析read操作来理解这几种I/O模型。<br>

read操作涉及到两个不同的阶段：

 * 数据准备阶段。这个阶段等待数据包到达，当数据包到达后会被拷贝到内核缓冲区。
 * 数据拷贝阶段。这个阶段数据从内核缓冲区拷贝到应用线程缓冲区。

 #### 阻塞I/O（blocking I/O）
 我们上文提到的IO场景，就是阻塞I/O。当应用进程执行read操作时会一直block，直到数据准备好并且从内核缓冲区拷贝到应用缓冲区，如下图所示：
 
 ![blocking I/O](/img/20180726/blocking_io.png)
 
 * 应用调用recvfrom，进入内核态
 * 内核发现数据没有准备好，阻塞应用进程，等待数据到达
 * 数据到达，从内核缓冲区拷贝到应用缓冲区
 * 唤醒应用进程，处理数据
 
我们看到阻塞I/O模型下，整个过程应用进程一直处于block状态，直到数据准备好并且从内核缓冲区拷贝到应用缓冲区(或者发生interrupt)。

#### 非阻塞I/O（nonblocking I/O)
非阻塞I/O模式下，应用调用recvfrom后会立即收到返回，不会被阻塞，如下图所示：

![nonblocking I/O](/img/20180726/nonblocking_io.png)

当处于非阻塞I/O模式下(socket设置为nonblocking)，在数据准备阶段调用recvfrom，内核会立即返回EWOULDBOCK错误。应用进程会轮询recvfrom，直到数据准备好，在轮询阶段不会发生block，但是浪费CPU时间。

#### I/O多路复用（I/O Multiplexing）
I/O多路复用模型下，应用先调用select或poll等待多个I/O操作ready状态，应用进程会一直block直到有I/O操作处于ready状态，然后调用recvfrom获取数据，如下图所示：

![I/O multipexing](/img/20180726/multipexing_io.png)

I/O多路复用模型下，会调用两次系统调用：select和recvfrom，它的优势是可以同时处理多个I/O操作，当有I/O操作准备就绪时就执行。当然，如果处理单个I/O操作，性能显然比不上阻塞I/O。对于处于多I/O操作，还有一种多线程block I/O模型，为每个I/O操作创建一个线程。多线程block I/O模型下，要为每一次请求创建一个线程去处理，很容易造成线程过多导致系统崩溃。

#### 信号驱动I/O模型（signal-driven I/O模型）
信号驱动I/O模型下，应用进程告诉内核：当I/O操作就绪时通过SIGIO通知，我收到通知后回去读取数据，如下图所示：

![signal-driven I/O](/img/20180726/signal_driven_io.png)

* 设置socket支持signal-driven I/O，调用sigaction注册信号通知handler
* 等待数据就绪，此时进程可处理其它事情
* 数据准备就绪，内核发出SIGIO信号
* 应用进程通过handler处理信号，执行recvfrom读取数据
* 信号驱动I/O模型下，在等待数据就绪阶段，应用进程不会block，可以执行其它操作。

#### 异步I/O模型（asynchronous I/O）

异步I/O模型与信号驱动模型的不同之处是信号通知点，信号驱动模型是在数据准备就绪时告诉应用程序I/O操作可以初始话了，应用程序调用过程包含拷贝数据阶段，而异步I/O模型是在数据拷贝完成后告诉应用程序来取数据。

![asynchronous I/O](/img/20180726/asynchronous_io.png)

#### I/O模型对比

![I/O模型对比](/img/20180726/io_compare.png)

前四种模型的不同点是数据准备阶段时的行为，在数据拷贝阶段应用进程会阻塞；异步I/O模型不同，它同时处理两个阶段完才会通知应用来去数据，应用进程不会发生阻塞，因此具有更高的效率。

#### 同步I/O vs 一步I/O
>* 同步I/O：在I/O操作过程中，I/O操作会导致请求进程阻塞，信号驱动模型也属于同步I/O，因为在recvfrom调用阶段请求进行会阻塞。
>* 异步I/O：I/O操作过程中，I/O操作不会导致请求进程阻塞。

### I/O多路复用实现
前述章节中我们介绍了几种I/O模型，接下来介绍一下Unix系统中I/O多路复用模型的实现: select，pselect，poll，epoll，kqueue。select, pselect, poll是POSIX标准定义的接口；epoll是Linux下实现的select/poll增强版本；kqueue是FreeBSD系统上开发的一个高性能的事件通知接口，功能类似epoll。

#### select
select告诉kernel所关注的I/O描述符列表，返回就绪状态的描述符个数，select函数声明如下：

![select声明](/img/20180726/select.png)

应用进程调用select会一直阻塞，除非有描述符处于就绪状态或者等待超时；超时时间有三种状态：

* null：一直阻塞直到有描述符处于就绪状态
* 0: 立即返回
* 正值：等待时间

select实现了I/O多路复用，但是它有一下缺陷：

* 描述符个数有限制
* 返回信息有限，应用只能感知到有描述符处于就绪状态，程序需要通过轮询去定位就绪状态的描述符，当监控大量描述符时很低效
* 只支持水平触发I/O事件

#### pselect
![pselect声明](/img/20180726/pselect.png)

相对于select，pselect的变更有两点：

* pselect使用timespec能支持纳秒级别的超时时间，select只支持毫秒级别的超时时间
* pselect参数增加signal mask，用来过滤掉某些当前不关注的描述符

#### poll
poll功能类似select，只是参数增加了额外信息，接口定义如下：

![poll声明](/img/20180726/poll.png)

fdarray是需要监控的描述符数组，数组元素是pollfd结构，它包含描述符，检测事件，发生事件，结构如下：

![pollfd结构](/img/20180726/pollfd.png)

events指定针对描述符的检测时间，revents保存这些事件的检测结果。每个bit代表一个事件，所有事件类型如下：

![event类型](/img/20180726/poll_event.png)

#### epoll/kqueue

epoll和kqueue功能类似，其中epoll是select和poll的改进版本，它的优势如下：

* 没有描述符个数的限制
* 只会返回就绪状态的描述符列表，不会像select/poll一样进行全表扫描
* 使用mmap避免数据从内核态到用户态的拷贝

### 总结和得到

本篇主要介绍了几种I/O模型，通过这几种模型我们可以了解到同步I/O和异步I/O，同时也能准确理解I/O复用这一概念；接着重点介绍了I/O复用的几种实现机制：select/pselect/poll/epoll/kqueue。Java NIO就是基于I/O复用实现的I/O框架，先理解这些基础概念有助于我们更好地理解NIO。

### 参考资料

[https://notes.shichao.io/unp/ch6/#io-models](https://notes.shichao.io/unp/ch6/#io-models)
[https://notes.shichao.io/unp/ch6/#select-function](https://notes.shichao.io/unp/ch6/#select-function)
[https://notes.shichao.io/unp/ch6/#pselect-function](https://notes.shichao.io/unp/ch6/#pselect-function)
[https://notes.shichao.io/unp/ch6/#poll-function](https://notes.shichao.io/unp/ch6/#poll-function)
[https://wiki.linuxfoundation.org/networking/kevent](https://wiki.linuxfoundation.org/networking/kevent)
[https://www.cnblogs.com/linganxiong/p/5583415.html](https://www.cnblogs.com/linganxiong/p/5583415.html)
[https://baike.baidu.com/item/epoll/10738144](https://baike.baidu.com/item/epoll/10738144)
[https://www.cnblogs.com/FG123/p/5256553.html](https://www.cnblogs.com/FG123/p/5256553.html)
[https://blog.csdn.net/summerhust/article/details/18260117](https://blog.csdn.net/summerhust/article/details/18260117)

