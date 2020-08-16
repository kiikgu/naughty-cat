---
title: Java集合概述
date: 2017-09-17 14:55:49
categories: [java]
tags: [java,集合]
toc: true
---

Java集合框架提供了适合各种场景的接口和实现，以及操作集合的各种算法，提高我们的编程效率，大大提高了程序的复用率。
<!--more-->

### 面对的问题
&nbsp;&nbsp;&nbsp;&nbsp;在Java集合框架还未出现的“洪荒”年代，要实现某个算法，比如：排序、搜索，一般步骤如下：

1. 实现一种数据结构，存储和获取元素
2. 保证数据结构功能的正确性、性能满足需要
3. 针对数据结构写相应的算法逻辑

我们只是想关注算法的实现，但是在实现算法之前又不得不设计实现一个算法需要的数据结构，它不但要满足基本的功能，对于特定的算法，还要考虑其性能，除此之外还面临以下问题：

1. 面对不同的场景，要实现不同的数据结构
2. 不可避免自己的实现出现bug及性能问题
3. 大家都在重复造轮子
4. 想用别人的实现，要去学习相应的API
5. 更换数据结构底层实现，可能要对程序有很大的改动

为什么不能有一些通用的数据结构，在需要时直接拿来用？于是Java集合框架出现了！

### 框架优势
毋庸置疑，Java集合框架的出现是必然事件，极大减轻了编程的负担。Java集合框架的特点：

1. 提供了一组“小且精”的接口
2. 提供了适合各种接口实现
3. 提供了Abstract实现，方便扩展
4. 提供了针对接口的一些常用算法和工具，如：排序，搜索等

她能带给我们的好处：

1. 不用再重复造轮子，专注我们关注的功能
2. Java集合提供的实现能够涵盖大部分场景，且性能足够强大，可以放心使用
3. 重用代码，减轻学习负担
4. 基于接口编程，可以很方便地替换内部实现，容易扩展现有的实现类
5. 很方便实现不同API之间的互通，不用进行对象间的适配

### 核心接口

![java核心接口](/img/20170917/colls-coreInterfaces.gif)

Java集合框架核心接口是框架的基础，包含Collection和Map两个分支。

#### Collection接口

![collection接口](/img/20170917/colls-interfaces.png)
![map接口](/img/20170917/map-interfaces.png)

* 每个接口都是通用的
* 接口“小而美”, 很多集合修改方法可选择性实现，调用不支持的修改方法抛出UnSupportedException
* 不同的接口具有不同的特性：modified/unmodified、mutable/immutable、fixed-size/variable-size、random access/sequence access

**1. Collection**

集合代表一组元素，根据元素可重复、不重复、有序、无序等特性，集合可以分为多种类型。Collection是对所有集合公共特性的抽象。将方法中的参数类型定义为Collection，可以让方法操作所有集合。

**2. Set**

* 元素不可重复的集合

**3. List**

* 有序且允许元素重复的集合
* 可以通过下标随机存取元素

**4. Queue**

* 元素按照优先级有序排列的集合
* 元素存取操作发生在集合的尾部和头部，头部取出元素，尾部插入元素
* 同等优先级下集合获取元素的顺序：FIFO

**5. Deque**

* 元素按照优先级排列的集合
* 可以在集合的两端插入、获取、删除元素
* 同等优先级下集合获取元素的顺序，可以是FIFO、LIFO

**6. Map**

* k-v映射的集合，且key不能重复

**7. SortedSet**

* 继承自Set且元素有序的集合

**8. SortedMap**

* 继承自Map且Key值有序的k-v映射集合

#### 接口实现
![接口实现](/img/20170917/colls-implements.png)

* 普通的接口实现都是unsynchronized
* Collections提供synchronize方法将集合变为synchronized
* 集合遍历时支持fail-fast
* 提供了Abstract类方便扩展：AbstractCollection, AbstractSet, AbstractList, AbstractSequentialListh和 AbstractMap


### 并发集合
#### 并发集合接口
![并发集合接口](/img/20170917/con-colls-interfaces.png)

#### 并发集合实现
![并发集合实现](/img/20170917/con-colls-implements.png)


### 总结
这一节首先介绍了Java集合框架的使命，提高了编程效率，且大大提高了程序的复用率；接着介绍了集合框架的构成，它是有三部分构成：一组小而美的接口，各种集合实现以及操作集合的常用算法；最后，简单介绍了集合的核心接口，及每个接口所代表集合的特点。

### 参考
[http://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html](http://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
[http://docs.oracle.com/javase/tutorial/collections/interfaces/index.html](http://docs.oracle.com/javase/tutorial/collections/interfaces/index.html)
