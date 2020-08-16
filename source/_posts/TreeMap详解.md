---
title: TreeMap详解
date: 2017-11-17 08:41:34
categories: [java]
tags: [TreeMap,java,集合]
toc: true
---

TreeMap不仅实现了Map接口，而且实现了SortedMap和NavigableMap接口。因此，除了具有Map的功能，还具有SortedMap和NavigableMap接口定义的功能。本文主要先介绍TreeMap新增功能，然后再分析其具体实现。
<!--more-->

### TreeMap继承结构

![TreeMap继承结构](/img/20171118/TreeMap_struct.png)

#### SortedMap接口

##### 特点

* 元素根据Key值进行排序，顺序是有Key的自然顺序或传入的Comparator方法决定
* 所有插入Map的元素都是可比较的(实现Comparable接口或能够被Comparator比较)
* 两个元素的比较结果必须与equal()方法一致，即当且仅当a.compare(b) == 0时，a.equals(b) == true 

##### 包含方法
![SortedMap方法](/img/20171118/SortedMap_方法.png)

SortedMap接口继承了Map接口，添加了6个新方法定义：comparator、subMap、headMap、tailMap、firstKey、lastKey；重新定义了3个继承自Map接口的方法：keySet、values、entrySet。

**Comparator<? super K> comparator()**

返回SortedMap用来对key做比较的comparator，如果没有则返回null。

**SortedMap<K,V> subMap(K fromKey, K toKey)**

获取SortedMap的一个子Map

* 子Map由所有key值在***[fromKey, toKey)***区间的键值对组成
* 对子Map的修改会影响到父Map，反之亦然
* 子Map插入区间[fromKey, toKey)以外的元素时，会抛出IllegalArgumentException

**SortedMap<K,V> headMap(K toKey)**

返回SortedMap的一个子Map

* 子Map由所有***key<toKey***的键值对组成
* 其它特点同subMap方法

**SortedMap<K,V> tailMap(K fromKey)**

返回SortedMap的一个子Map

* 子Map由所有***key>=fromKey***的键值对组成
* 其它特点同subMap方法

**K firstKey()**

返回SortedMap中第一个元素

**K lastKey()**

返回SortedMap中最后一个元素

**keySet、values、entrySet**

* 返回结果集根据Key值进行升序排列
* 不支持add、addAll方法
* 其它功能同Map

##### Tips
所有SortedMap接口的实现类都应该实现以下4个构造方法：

* 无参构造函数
* 仅包含一个Comparator类型参数的构造函数
* 仅包含一个Map类型的构造函数
* 仅包含一个SortedMap的构造函数

#### NavigableMap接口

![NavigableMap方法](/img/20171118/NavigableMap_方法.png)

NavigableMap接口扩展了SortedMap接口，增加了如下功能：

* 提供更多区间查询功能：lowerEntry、floorEntry、ceilingEntry、higherEntry等
* 提供了按Key值进行正序、倒序访问的方式，descendingMap方法返回Key值为倒序的Map
* 提供了获取首位元素的方法：firstEntry、pollFirstEntry、lastEntry、pollLastEntry
* 不支持Entry.setValue方法

### 内部实现及性能

TreeMap内部实现结构为红黑树，可参考：{% post_link 详解红黑树 %}。<br> TreeMap的containsKey，get，put及remove操作的时间复杂度是log(n)


### 参考
[https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html](https://docs.oracle.com/javase/8/docs/api/java/util/TreeMap.html)


