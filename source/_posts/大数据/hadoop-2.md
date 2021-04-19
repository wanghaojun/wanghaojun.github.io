---
title: hadoop(2)
date: 2018-07-14 08:58:48
tags: [大数据]
---
HDFS:分布式文件存储系统

<!--more-->

## hdfs设计前提与目标：
* 硬件容错
* 流式数据访问
* 超大规模数据集
* 简单一致性模型
* 移动计算比移动数据便宜
* 跨异构硬件和软件平台的可移植性

## hdfs的主要组件

### NameNode 
* 存储元数据
* 元数据保存在内存中
* 保存文件，block，datanode之间的映射关系
* 镜像文件FsImage:目录树，加载到内存中
* 修改日志EditLog:记录了所有针对文件的创建、删除、重命名等操作
* 元数据Metadata:存储目录树，记录块信息
* NameNode启动时，把EditLog的事务都应用到FsImage的内存表现上，进入保护模式，只允许访问不允许操作。

### DataNode
*  存储文件内容
* 文件内容保存在磁盘中
* 维护了block id到datanode的文件的映射关系
* 管理存储在它们所运行节点上的数据
* 按照客户端的请求，执行文件系统的读写操作
* 根据NameNode的指令执行block的创建，删除和配分。
* 每隔3秒发送一次心跳报告给NameNode，同时NameNode将命令响应给datanode

### SecondaryNameNode 
相当于NameNode的秘书，在达到一定限度后（一个小时，或者一百万条），使用http请求下载NameNode的editlog与fsimage，然后在本地合并，使用httppost将fsimage ckpt复制到NameNode。辅助的作用，能够备份，防止NameNode过大

## HDFS特性与优点
* 横向线性扩展 半年到两年扩充一次
* 自动冗余  三倍冗余
* 高容错性
* 可构建在廉价商用机
* 并发处理

## HDFS缺点
* 命名空间的限制 不适合存放小文件
* 性能问题
* 不适合低延迟
* 无法高效存储大量小文件
* 不支持多用户写入及任意修改文件

## HDFS 常用命令

	hadoop fs -ls <path> 指定文件的详细信息
	hadoop fs  -mkdir <path> 创建文件
	hadoop fs -copyFromLocal <localsrc> <dst> 把本地文件上传到hdfs
	hadoop fs -put  <localsrc> <dst> 把本地文件上传到hdfs
	hadoop fs -get  <dst> <local> 从hdfs上下载文件
	hadoop fs -copyToLocal <dst> <localsrc> 从hdfs上下载文件

### HDFS safemode



 
