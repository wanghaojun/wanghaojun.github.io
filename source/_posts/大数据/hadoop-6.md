---
title: hadoop-6
date: 2018-07-18 08:26:56
tags: hadoop
---

spark：
大数据的分析引擎
基于mapreduce，但是对mapreduce进行了扩展，以有效地用于更多类型的计算，包括交互式查询和流处理。
主要特性是内存中的集群计算
<!--more-->

## spark组件
* spark core
* spark SQL
* spark Streaming
* MLlib( Machine Learning Library)
* GraphX

## spark  VS MapReduce
数据存储结构：弹性分布式数据集RDD，对数据进行运算和Cahce
编程范式：DAG：Transformation + Action
中间数据：内存中
Task：以线程的方式维护
 
## 类似
Flink + Beam

## Spark集群
spark基于standalone集群搭建，standalone是主从结构，分master,worker
首先，简单说明下Master、Worker、Application三种角色。
Application：带有自己需要的mem和cpu资源量，会在master里排队，最后被分发到worker上执行。app的启动是去各个worker遍历，获取可用的cpu，然后去各个worker launch executor。
Worker：每台slave起一个(也可以起多个)，默认或被设置cpu和mem数，并在内存里做加减维护资源剩余量。Worker同时负责拉起本地的executor backend，即执行进程。
Master：接受Worker、app的注册，为app执行资源分配。Master和Worker本质上都是一个带Actor的进程

## RDD
saveAsTextFile：将数据以文本格式输出到指定目录
def saveAsTextFile(path: String)
saveAsObjectFile：写入文件为SequenceFile格式
def saveAsObjectFile(path: String)

## RDD Transformation的Lazy特性
Transformation中Lazy在action执行前是不会进行计算的，只是记录下当前要做的事情。
Action结果会返回给driver
可以避免产生各种众多的中间数据.
Transformation操作：得到一个新的RDD，比如从数据源生成一个新的RDD，从RDD生成一个新的RDD
哪些属于Transformation：map(func)，mapPartitions(func)，sample(withReplacement,faction,seed)，filter(func)，flatMap(func),distinct([numTasks])等
Action操作：action是得到一个值，或者一个结果
哪些属于Action: reduce(func),collect(),count(),first(),take(n),takeSample(withReplacement，num，seed),takeOrdered(n, [ordering]),saveAsTextFile（path）

可以通过持久化机制避免这种重复计算的开始
持久化方法：
* persist()  MEMEORY_ONLY MEMORY_AND_DISK unpersist()
* cache()  持久化到内存中
* heckpoint() 持久化到外存中，可以用来备份

cache：RDD的cache()方法其实调用的就是persist方法，缓存策略均为MEMORY_ONLY
persist：可以通过persist方法手工设定StorageLevel来满足工程需要的存储级别
`def cache(): RDD[T]`
`def persist(): RDD[T]`
`def persist(newLevel: StorageLevel): RDD[T]`
`org.apache.spark.storage.StorageLevel = StorageLevel(disk=false, memory=false, offheap=false, deserialized=false, replication=1)`
存储级别如下：
NONE (default)
`DISK_ONLY`
`DISK_ONLY_2`
`MEMORY_ONLY (default for cache() operation)`
`MEMORY_ONLY_2`
`MEMORY_ONLY_SER`
`MEMORY_ONLY_SER_2`
`MEMORY_AND_DISK`
`MEMORY_AND_DISK_2`
`MEMORY_AND_DISK_SER`
`MEMORY_AND_DISK_SER_2`
`OFF_HEAP`



## 算子
* foreach(f)：foreach用于遍历RDD,将函数f应用于每一个元素
   def foreach(f: T => Unit)
* collect：将RDD转换成list，结果以List的方式输出
   def collect(): Array[T]
* count：对RDD的元素进行统计，返回个数
  def count(): Long
* top：返回最大的k个元素，返回List的形式
  def top(num: Int)(implicit ord: Ordering[T]): Array[T]
* take：返回数据的前k个元素，返回List的形式
  def take(num: Int): Array[T]

