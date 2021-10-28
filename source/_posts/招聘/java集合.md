---
title: java集合
date: 2021-05-25 19:52:09
tags: [面试,java]
---

介绍java集合。

<!--more-->

##### 集合

![](http://img.wanghaojun.cn//other/20210330101050.png)

Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。Collection 接口又有 3 种子类型，List、Set 和 Queue，再下面是一些抽象类，最后是具体实现类，常用的有 ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap、LinkedHashMap 等等。

集合体系框架如下图：

![](http://img.wanghaojun.cn//other/20210330101221.png)

##### Map

**HashMap** 

Hashmap 是一个最常用的 Map,它根据键的 HashCode 值存储数据,根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。 HashMap 最多只允许一条记录的键为 Null;允许多条记录的值为 Null；HashMap 不支持线程的同步，即任一时刻可以有多 个线程同时写 HashMap；可能会导致数据的不一致。如果需要同步，可以用 Collections 的 synchronizedMap 方法使 HashMap 具有同步的能力，或者使用 ConcurrentHashMap。

更加详细的hashmap可以参考[这篇文章](https://tech.meituan.com/2016/06/24/java-hashmap.html)

**HashTable**

与 HashMap 类似,它继承自 Dictionary 类，不同的是:它不允许记录的键或者值为空;它支持线程的同步，即任一时刻只有一个线程能写 Hashtable,因此也导致了 Hashtable 在写入时会比较慢。

**LinkedHashMap** 

HashMap 的一个子类，保存了记录的插入顺序，在用 Iterator 遍历 LinkedHashMap 时，先得到的记录肯定是先插入的.也可以在构造时用带参数，按照应用次数排 序。在遍历的时候会比 HashMap 慢，不过有种情况例外，当 HashMap 容量很大，实际数据较少时， 遍历起来可能会比 LinkedHashMap 慢，因为 LinkedHashMap 的遍历速度只和实际数据有关，和 容量无关，而 HashMap 的遍历速度和他的容量有关。

**TreeMap** 

实现 SortMap 接口，能够把它保存的记录根据键排序,默认是按键值的升序排序， 也可以指定排序的比较器，当用 Iterator 遍历 ，得到的记录是排过序的

一般情况下，我们用的最多的是 HashMap,在 Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么 TreeMap 会更好。如果需要输出的 顺序和输入的相同,那么用 LinkedHashMap 可以实现,它还可以按读取顺序来排列。

HashMap 不支持线程同步，即任一时刻可以有多个线程同时写 HashMap，可能会导致数据的不一致性。如果需要同步，可以用 Collections 的 synchronizedMap 方法使 HashMap 具有同步的能力。

##### Array & ArrayList

Array 可以包含基本类型和对象类型，ArrayList 只能包含对象类型。 Array 大小是固定的，ArrayList 的大小是动态变化的。 ArrayList 提供了更多的方法和特性，比如：addAll()，removeAll()，iterator()等等。

