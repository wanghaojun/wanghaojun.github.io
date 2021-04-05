---
title: java基础-2021春招
date: 2021-04-05 14:44:03
tags: [面试,java]
---

java相关基础知识：String & StringBuffer & StringBuild、Map、Array & ArrayList、集合、线程、Synchronized、Object

<!--more-->

##### String StringBuffer & StringBuild

当对字符串进行修改的时候，需要使用 StringBuffer 和 StringBuilder 类。和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。

在使用 StringBuffer 类时，每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象，所以如果需要对字符串进行修改推荐使用 StringBuffer。

StringBuilder 类在 Java 5 中被提出，它和 StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。

由于 StringBuilder 相较于 StringBuffer 有速度优势，所以多数情况下建议使用 StringBuilder 类。

##### Map

**HashMap** 

Hashmap 是一个最常用的 Map,它根据键的 HashCode 值存储数据,根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。 HashMap 最多只允许一条记录的键为 Null;允许多条记录的值为 Null；HashMap 不支持线程的同步，即任一时刻可以有多 个线程同时写 HashMap；可能会导致数据的不一致。如果需要同步，可以用 Collections 的 synchronizedMap 方法使 HashMap 具有同步的能力，或者使用 ConcurrentHashMap。

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

##### 集合

![](http://img.wanghaojun.cn//other/20210330101050.png)

Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。Collection 接口又有 3 种子类型，List、Set 和 Queue，再下面是一些抽象类，最后是具体实现类，常用的有 ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap、LinkedHashMap 等等。

集合体系框架如下图：

![](http://img.wanghaojun.cn//other/20210330101221.png)

##### 线程

**线程的生命周期**

![](http://img.wanghaojun.cn//other/20210330142153.png)

- 新建状态:

  使用 **new** 关键字和 **Thread** 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 **start()** 这个线程。

- 就绪状态:

  当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。

- 运行状态:

  如果就绪状态的线程获取 CPU 资源，就可以执行 **run()**，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。

- 阻塞状态:

  如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：

  - 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入到等待阻塞状态。
  - 同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。
  - 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。

- 死亡状态:

  一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

**线程的优先级**

每一个 Java 线程都有一个优先级，这样有助于操作系统确定线程的调度顺序。

Java 线程的优先级是一个整数，其取值范围是 1 （Thread.MIN_PRIORITY ） - 10 （Thread.MAX_PRIORITY ）。

默认情况下，每一个线程都会分配一个优先级 NORM_PRIORITY（5）。

具有较高优先级的线程对程序更重要，并且应该在低优先级的线程之前分配处理器资源。但是，线程优先级不能保证线程执行的顺序，而且非常依赖于平台。

**线程创建**

Java 提供了三种创建线程的方法：

- 通过实现 Runnable 接口；
- 通过继承 Thread 类本身；
- 通过 Callable 和 Future 创建线程。

##### Synchronized

synchronized 是 Java 的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证 在同一时刻最多只有一个线程执行该段代码。JDK1.5 以后引入了自旋锁、锁粗化、轻量级锁， 偏向锁来有优化关键字的性能。

##### Object

所有类的父类，如果一个类没有继承其它类，也会默认继承Object的类

方法：

- Object()默认构造方法。
- clone() 创建并返回此对象的一个副本。
- equals(Object obj) 指 示某个其他对象是否与此对象“相等”。
- finalize()当垃圾回收器确定不存在对该对象的更多引 用时，由对象的垃圾回收器调用此方法。
- getClass()返回一个对象的运行时类。
- hashCode()返回 该对象的哈希码值。
- notify()唤醒在此对象监视器上等待的单个线程。
- notifyAll()唤醒在此 对象监视器上等待的所有线程。
- toString()返回该对象的字符串表示。
- wait()导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。
- wait(long timeout)导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超 过指定的时间量。
- wait(long timeout, int nanos) 导致当前的线程等待，直到其他线程调用此 对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某 个实际时间量。

