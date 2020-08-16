---
title: LinkedHashMap详解
date: 2017-11-09 08:41:19
categories: [java]
tags: [LinkedHashMap,java,集合]
toc: true
---

LinkedHashMap继承自HashMap, 它在HashMap的结构之上，将所有的节点连接起来，组成双向链表，从而具有了新的特性，本文将详细介绍LinkedHashMap具有的新特性及其实现。
<!--more-->

**本文分析的HashMap源码来自：jdk1.8**

### LinkedHashMap结构
![LinkedHashMap结构](/img/20171108/LinkedHashMap结构.png)

![LinkedHashMap继承结构](/img/20171108/LinkedHashMap_extend.png)

![LinkedHashMap节点](/img/20171108/LinkedHashMap_node.png)

相对于HashMap，LinkedHashMap多了双向链表结构，包括类属性增加头尾节点引用：head、tail，节点中也增加了前后关联节点的引用：before和after。

### 有序
基于链表，LinkedHashMap实现了两种元素排序功能：插入顺序、访问顺序。
#### insertion-order
LinkedHashMap在将节点插入Hash表的同时，也会按照插入顺序把双向链表给构建起来。
![LinkedHashMap newNode](/img/20171108/LinkedHashMap_newNode.png)

![LinkedHashMap linkNodeLast](/img/20171108/LinkedHashMap_linkLast.png)

#### access-order
在创建LinkedHashMap对象时，指定accessOrder属性，元素会按照访问顺序排列，最少使用的元素排在前面，最近使用的原始排在后面。

![访问顺序排列](/img/20171108/LinkedHashMap_accessOrder.png)

LinkedHashMap通过3个方法来实现accessOrder功能：afterNodeRemoval、afterNodeInsertion、afterNodeAccess。

![accessOrder实现](/img/20171108/LinkdedHashMap_accessOrder实现.png)

LinkedHashMap可以改变元素访问顺序的方法包括：put、putIfAbsent、get、getOrDefault、compute, computeIfAbsent、computeIfPresent、merge，这些方法都会改变LinkedHashMap的结构，导致fast-fail。

LinkedHashMap可以用来实现LRU cache，从上图中的afterNodeInsertion方法可以看到，当插入新元素时，如果removeEldestEntry判断出需要删除eldest元素，则执行删除操作。默认，removeEldestEntry总是返回false，可以重写该方法实现LRU功能。

### 性能

add、remove、contains等基本操作：O(1) <br>
遍历操作开销：与LinkedHashMap大小，即元素个数相关；而HashMap遍历操作开销要比LinkedHashMap开销大，它与HashMap的Capacity相关。

总体来说，LinkedHashMap的性能是介于HashMap和TreeMap之间；由于LinkedHashMap维护了双端链表，进行元素操作时，需要增加额外的开销，性能开销的回报是元素有序。

### 参考
[https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html)

版权声明：本文为博主原创文章，转载请注明出处