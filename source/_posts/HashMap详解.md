---
title: HashMap详解
date: 2017-10-19 07:24:57
categories: [java]
tags: [HashMap,java,集合]
toc: true
---
**本文分析的HashMap源码来自：jdk1.8**

HashMap是我们在编程中常用的数据结构，本篇主要从HashMap的几个特性出发，分析其内部实现原理

* capacity和load_factor
* put/get
* keySet方法
* fast-fail
* 序列化/反序列化

<!--more-->

### 概述

HashMap是我们在编程中常用的数据结构，本篇主要从HashMap的几个特性出发，深入分析其内部实现原理。

* capacity和load_factor
* put/get
* keySet方法
* fast-fail

#### capacity和load_factor
capacity和load\_factor分别表示HashMap的容量和负载因子，简单来说，容量是指HashMap能装多少元素，load\_factor指HashMap所能承受的负载有多大，就好比一个能装20L水的桶，全部装满会的话会增加我们操作的难度。同样，如果把HashMap装满的话，会降低HashMap的性能，所以当元素个数达到负载因子规定的阈值时，HashMap会进行扩容，容量扩大一倍。<br>

**HashMap结构**

为了探究capcacity和load_factor的含义，我们来看下HashMap的结构。HashMap内部实现是Hash表：<br>
![hash表结构图](/img/20171018/hashmap结构图.png)

图中可以看出，hash表是由数组和链表组成，数组的每个元素是链表的头结点，这样一个链表叫做一个桶，桶的个数就是HashMap的capacity，即是数组的长度；当hash表中的元素个数超过capacity * load\_factor时，hash表就要扩容。根据经验值load\_factor=0.75时，HashMap的性能和空间使用率最优。

**指定capacity和load_factor**

![HashMap构造函数](/img/20171018/hashMap构造函数1.png)

可以通过指定capacity和load_factor来构造HashMap。为了方便，HashMap的capacity取值总是2的n次方。这里用了一个很有意思的算法，求大于一个给定数值的最小2的n次方数。

![tableSizeFor方法](/img/20171018/hashMap_tableSizeFor.png)

![tableSizeFor方法原理图](/img/20171018/hashMap_tableSizeFor_detail.png)

**resize**

当HashMap元素个数大于阈值时，HashMap的容量会扩大一倍，部分元素会重新hash到新的桶中。
![HashMap resize重新hash算法](/img/20171018/hashMap_resize.png)

* 首先创建新的hash表
* 遍历旧hash表，将每个桶中的元素重新hash到新的hash表中

新的hash表的桶数是旧hash表的一倍，旧hash表中同一个桶的所有元素会被划分到newTable[n]和newTable[n+oldCap]这两个桶中，如上图hash表扩容后：

![重新hash](/img/20171018/hashMap_resize_detail.png)

（TreeNode就不在此处展开讨论了）

#### put/get

**put**

![HashMap put](/img/20171018/hashMap_putVal.png)

put的过程很好理解，传入参数为(key，value), 首先根据key值算出hash值，然后再key的hash值再hash到某一个桶，最后遍历链表，如果key已经存在则更新value，并返回旧的value，否则插入新节点。 <br>
这里有个问题：在算key的hash值时，为什么不直接用key.hashCode()？让我们看一下HashMap中求key得hash值算法。

![HashMap hash](/img/20171018/hashMap_hash.png)

HashMap中桶的个数总是2的次方，如果直接使用key的hashCode作为hash值进行模运算，那么只有低几位能决定元素归属于哪个桶，这样很容易导致元素分布不均匀；HashMap通过高16位和低16位异或操作来抵消这种影响。 这里也可以看到HashMap对key为null的支持，当key为null时，对应的hash值为0<br>

代码中可以看到，当某个桶元素个数超过TREEIFY_THRESHOLD时，链表存储会被转换为红黑树，来优化性能，关于HashMap中红黑树的应用到后面细讲，这里就不再继续展开。

如果执行的是插入新节点，在最后还要判断，是否需要扩容。

**get**

![HashMap get](/img/20171018/hashMap_getNode.png)

get过程：先根据key算出hash，然后找到相应的桶，最后遍历桶中的元素比较key值，找出目标元素。如果元素分布均匀，往往元素位于桶的首位，查询速度很快；如果元素分布不均匀，HashMap通过把链表转换为红黑树，来提高查询性能。

#### keySet

视图是遍历Map的唯一途径，Map提供了3个视图：keySet、values、entrySet，本文就以keySet实现为例，探索HashMap的视图实现方法。

![keySet 概览](/img/20171018/hashMap_keySet_overiew.png)

keySet是HashMap所有的key组成的set，通过keySet可以遍历、删除HashMap中的元素。HashMap通过如上图所示的三个内部类，来实现keySet的功能。

![HashMap KeySet](/img/20171018/hashMap_keySet_detail.png)

代码中可以看到，对KeySet的操作其实是直接操作的HashMap，所以clear、remove操作直接影响到HashMap结构。

**视图的遍历**

![HashMap KeyIterator](/img/20171018/hashMap_keySet_iterator.png)

上图的代码可以看到，keySet、values、entrySet三个视图的遍历分别对应三个不同的Iterator，分别是：KeyIterator、ValueIterator、EntryIterator。三个Iterator都继承自HashIterator，并实现Iterator接口。为什么要继承HashIterator类？

![HashMap HashIterator](/img/20171018/hashMap_hashIterator.png)

这三个Iterator除了next()实现不同，其它实现都一样，所以把对Iterator实现中的公共部分提出来，抽象成一个公共的HashIterator类。

#### fast-fail

HashMap不是线程安全的，当遍历HashMap过程中，如果其它线程更改了HashMap的结构，那么Iterator会抛出ConcurrentModificationException，导致遍历中断。只有当前的iterator调用remove方法修改HashMap的结构时，才不会抛出上述异常。

为什么要有fast-fail?

如果没有fast-fail, 处在多线程环境时，多个线程同时去修改HashMap的结构，那么遍历改HashMap时的结果是不可预测且多种多样的，这对程序来说是一场灾难。所以，HashMap的设计者能够迅速让程序检测出HashMap结构的变化，从而消除这种未知性！

![HashMap modCount](/img/20171018/hashMap_modCount.png)

HashMap通过modCount字段记录HashMap结构变化测试，即通过modCount记录HashMap的版本，当插入或者删除元素发生时，modCount就会加1，这时iterator检测到modCount的变化，就会抛出异常。

![HashMap iterator modCount](/img/20171018/hashMap_iterator_modCount.png)

当单线程遍历时，调用iterator.remove()方法，会去同步当前的modCount。

![HashMap iterator remove modCount](/img/20171018/hashMap_iterator_remove_modCount.png)

#### 序列化/反序列化
分析HashMap源码时，发现HashMap中的很多变量都是transient的，那么这些transient变量在序列化时不会被保存下来，HashMap为什么要这么做？它是通过什么方式实现序列化、反序列化的呢？

![HashMap trasient变量](/img/20171018/hashMap_serializable.png)

HashMap中key的hash值使用hashCode()方法生成，而hashCode()方法是native方法，在不同JVM实现上面可能不同。如果HashMap的变量不是transient，在序列化和反序列化时HashMap的内部结构是不会发生变化的，如果执行反序列化机器的hashCode()的返回值跟序列化机器返回值不同，那么整个HashMap就会发生错乱。

**HashMap实现序列化反序列化**

![HashMap 序列化](/img/20171018/hashMap_serializable_001.png)

![HashMap 序列化](/img/20171018/hashMap_serializable_002.png)

![HashMap 反序列化](/img/20171018/hashMap_serializable_003.png)

HashMap通过实现writeObject()、readObject()方法来实现序列化和反序列化，序列化时保存非transient变量、capacity、size及所有的key-value，反序列化时读取这些变量重新构造新的Hash表，并存储key-value值。

关于序列化反序列化请参考：{% post_link serialize和deserialize %}


### 总结
本文主要从HashMap的知识点点作为切入点，分析了HashMap的实现，通过这篇文章可以了解HashMap的capacity和load_factor、put/get、keySet方法、fast-fail、序列化/反序列化，及相关实现的细节。

### 参考

[http://hllvm.group.iteye.com/group/topic/45517](http://hllvm.group.iteye.com/group/topic/45517)

[https://stackoverflow.com/questions/1313922/step-through-jdk-source-code-in-intellij-idea](https://stackoverflow.com/questions/1313922/step-through-jdk-source-code-in-intellij-idea)

[https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)

版权声明：本文为博主原创文章，转载请注明出处