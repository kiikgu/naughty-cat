---
title: Java Map介绍
date: 2017-10-09 06:05:25
categories: [java]
tags: [Map,java,集合]
toc: true
---

简单介绍下Map接口
<!--more-->

### Java Map介绍

#### 简介
Map是key-value映射集合，并且key是唯一不重复的，一个key至多映射一个value；Map是函数的抽象。Map接口包含的方法如下：

* 基本方法（put、get、remove、containsKey、contansValue、size、isEmpty）
* 批量方法（putAll、clear）
* 集合视图 (keySet、entrySet、values)
* Java 8新增方法（getOrDefault、forEach、replace、replaceAll、compute、computeIfAbsent、computeIfPresent、merge）

Java包含3种基本的Map实现：HashMap、TreeMap、LinkListMap。

* 三种实现都是非线程安全的
* HashMap， 内部通过Hash表实现，无序、key-value可以为null，Map.Entry可以setValue
* TreeMap， 通过红黑树实现，按照key值自然排序或提供的Camparator方法排序、key值不能为null，Map.Entry不能setValue
* LinkedListMap， 内部由HashTable和双端链表实现，以插入顺序排列，同一个key重复插入不影响顺序、key-value可以为null

#### map集合视图
map可以转换为3种集合视图：

* keySet - map中的所有key组成的set
* values - map中的所有value组成的集合，不是set
* entrySet - map中的key-value键值对组成的set

集合视图是遍历map的唯一途径，操作3个集合视图的remove, clear方法都会直接影响到map

#### Java 8特性
##### stream

```
	public static void test1() {
        List<Person> persons = Arrays.asList(new Person("p1", 10), new Person("p2", 12), new Person("p3", 10));
        
        //按照年龄划分
        Map<Integer, List<Person>> map = persons.stream().collect(Collectors.groupingBy(Person::getAge));
        System.out.println(map);
        
        //按年龄换分后，求总年龄
        Map<Integer, Integer> sumMap = persons.stream().collect(Collectors.groupingBy(Person::getAge, Collectors.summingInt(Person::getAge)));
        System.out.println(sumMap);

        //按照条件划分
        Map<Boolean, List<Person>> map1 = persons.stream().collect(Collectors.groupingBy(s -> s.getAge() == 10));
        System.out.println(map1);
    }

    static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public int getAge() {
            return age;
        }


运行结果：      
{10=[Person{name='p1', age=10}, Person{name='p3', age=10}], 12=[Person{name='p2', age=12}]}
{10=20, 12=12}
{false=[Person{name='p2', age=12}], true=[Person{name='p1', age=10}, Person{name='p3', age=10}]}

```

##### getOrDefault

```
default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
 }

```

获取指定默认值

```
Map<String, String> m = new HashMap<>();
m.put("t1", "1");
m.put("t2", "2");

System.out.println(m.getOrDefault("t2", "default"));
System.out.println(m.getOrDefault("t3", "default"));

运行结果：
2
default
```

##### forEach

```
Map<String, String> m = new HashMap<>();
m.put("t1", "1");
m.put("t2", "2");

default void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
        action.accept(k, v);
    }
}

```

循环执行action

```
Map<String, String> m = new HashMap<>();
m.put("t1", "1");
m.put("t2", "2");

m.forEach((k, v)-> {
    System.out.println("key:" + k + ", value:" + v);
});

运行结果：
key:t1, value:1
key:t2, value:2
```

##### replace

```
default V replace(K key, V value) {
    V curValue;
    if (((curValue = get(key)) != null) || containsKey(key)) {
        curValue = put(key, value);
    }
    return curValue;
}

default boolean replace(K key, V oldValue, V newValue) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, oldValue) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    put(key, newValue);
    return true;
}
```

元素替换

```
Map<String, String> m = new HashMap<>();
m.put("t1", "1");
m.put("t2", "2");

System.out.println(m.replace("t3", "3"));
System.out.println(m.get("t3"));

System.out.println(m.replace("t2", "2-1"));
System.out.println(m.get("t2"));

System.out.println(m.replace("t1", "1", "1-1"));
System.out.println(m.get("t1"));

运行结果：
null
null
2
2-1
true
1-1
```

##### replaceAll
```
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    Objects.requireNonNull(function);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }

        // ise thrown from function is not a cme.
        v = function.apply(k, v);

        try {
            entry.setValue(v);
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
    }
}
```

所有元素以相同规则替换

```
Map<String, String> m = new HashMap<>();
m.put("t1", "1");
m.put("t2", "2");

m.replaceAll((x, y)-> x + "----" + y);
m.forEach((k, v)-> {
    System.out.println("key:" + k + ", value:" + v);
});

运行结果：
key:t1, value:t1----1-1
key:t2, value:t2----2-1

```
##### computeIfAbsent/computeIfPresent

```
default V computeIfAbsent(K key,
        Function<? super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) {
            put(key, newValue);
            return newValue;
        }
    }

    return v;
}
```
如果key不存在时，执行maping方法生成value

```
default V computeIfPresent(K key,
        BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue != null) {
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}
```
如果key存在时，执行maping方法生成新的value

```
Map<String, String> m = new HashMap<>();
m.put("t1", "1");
m.put("t2", "2");

m.computeIfAbsent("t3", (x) -> "1111");
System.out.println(m.get("t3"));

m.computeIfPresent("t3", (x, y) -> null);
System.out.println(m.get("t3"));

运行结果：
1111
null

```

#### 总结
本文大概讲解了Map接口的一些特点，及相关的使用技巧。

### 参考
[http://docs.oracle.com/javase/tutorial/collections/interfaces/map.html](http://docs.oracle.com/javase/tutorial/collections/interfaces/map.html)

[http://docs.oracle.com/javase/tutorial/collections/streams/index.html](http://docs.oracle.com/javase/tutorial/collections/streams/index.html)

[https://segmentfault.com/q/1010000000630486](https://segmentfault.com/q/1010000000630486)

[http://ifeve.com/sun-misc-unsafe/](http://ifeve.com/sun-misc-unsafe/)

[http://www.cnblogs.com/skywang12345/p/3245399.html](http://www.cnblogs.com/skywang12345/p/3245399.html)
