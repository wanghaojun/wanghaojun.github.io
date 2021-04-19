---
title: 使用OrderedDict实现LRU算法
date: 2021-03-19 13:06:49
tags: [算法,python]
---

本文主要介绍LRU机制和python中的OrderedDict字典，以及如何使用OrderedDict实现LRU算法。对应leetcode的第 146 题「LRU缓存机制」

<!--more-->

#### LRU

LRU是一种缓存淘汰策略，全称是Least Recently Used，就是说我们认为最近访问过的缓存数据是有用的，很长时间没有使用的是无用的，如果缓存满了，应该优先删除那行很久没有使用过的数据。

#### 有序字典OrderedDict

python中常用的字典是dict，OrderedDict类似于常规字典，但是有一些与排序操作相关的额外功能。

**实现**

```python
class collections.OrderedDict([items])
```

返回一个dict子类的实例，它具有专门用于重新排列字典顺序的方法。

**主要方法**：

- `popitem`(*last=True*)

  有序字典的 `popitem()`方法移除并返回一个 (key, value) 键值对。 如果 *last* 值为真，则按 LIFO 后进先出的顺序返回键值对，否则就按 FIFO 先进先出的顺序返回键值对。

- `move_to_end`(*key*, *last=True*)

  将现有 *key* 移动到有序字典的任一端。 如果 *last* 为真值（默认）则将元素移至末尾；如果 *last* 为假值则将元素移至开头。

#### LRU算法实现

力扣第 146 题「LRU缓存机制」就是设计数据结构：

首先要接收一个 `capacity` 参数作为缓存的最大容量，然后实现两个 API，一个是 `put(key, val)` 方法存入键值对，另一个是 `get(key)` 方法获取 `key` 对应的 `val`，如果 `key` 不存在则返回 -1。

LRU 缓存算法的核心数据结构就是哈希链表，双向链表和哈希表的结合体。

自己实现的LRU算法：

```python
class Node:
    def __init__(self, key: int, val: int):
        self.key = key
        self.val = val
        self.next = None
        self.prev = None

class DoubleList:

    def __init__(self):
        self.head = Node(0,0)
        self.tail = Node(0,0)
        self.head.next = self.tail
        self.tail.prev = self.head
        self.size = 0

    # 尾部添加节点
    def addLast(self, x: Node):
        x.prev = self.tail.prev
        x.next = self.tail
        self.tail.prev.next = x
        self.tail.prev = x
        self.size += 1

    #删除链表节点
    def remove(self, x: Node):
        x.prev.next = x.next
        x.next.prev = x.prev
        self.size -= 1


    #删除链表第一个节点并返回
    def removeFirst(self) -> Node:
        if self.head.next == self.tail:
            return None
        first = self.head.next
        self.remove(first)
        return first


class LRUCache:

    def __init__(self, c: int):
        self.capacity = c
        self.map = {int: Node}
        self.cache = DoubleList()

    # 将某个key提升为最近使用的
    def makeRecently(self, key: int):
        x = self.map[key]
        self.cache.remove(x)
        self.cache.addLast(x)

    # 添加最近使用的节点
    def addRecent(self, key: int, val: int):
        x = Node(key,val)
        self.cache.addLast(x)
        self.map[key] = x

    # 删除某个key
    def removeKey(self, key: int):
        x = self.map[key]
        self.cache.remove(x)
        self.map.pop(key)


    # 删除最久未使用的
    def removeLastRecently(self):
        x = self.cache.removeFirst()
        if x:
            self.map.pop(x.key)

    def get(self, key: int) -> int:
        if key in self.map:
            self.makeRecently(key)
            return self.map[key].val
        else:
            return -1

    def put(self,key: int, val :int):
        if key in self.map:
            self.map[key].val = val
            self.makeRecently(key)
            # self.removeKey(key)
            # self.addRecent(key,val)
            return None
        if (self.capacity == self.cache.size):
            self.removeLastRecently()
        self.addRecent(key,val)

```

使用OrderedDict实现的LRU算法：

```python
class LRUCache:

    def __init__(self, capacity: int):
        self.cache = collections.OrderedDict()
        self.capacity = capacity

    def get(self, key: int) -> int:
        if key in self.cache:
            self.cache.move_to_end(key)
            return self.cache[key]
        else: return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache[key] = value
            self.cache.move_to_end(key)
            return None
        if len(self.cache) == self.capacity:
            self.cache.popitem(last=False)
        self.cache[key] = value
```



