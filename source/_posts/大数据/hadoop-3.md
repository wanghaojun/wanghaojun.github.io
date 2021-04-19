---
title: hadoop(3)  
date: 2018-07-15 08:31:53
tags: [大数据]
---

MapReduce 简介
采用“分而治之”的策略，设计理念是“计算向数据靠拢”

<!--more-->

## MapReduce 不擅长
* 实时计算
* 流式计算
* DGA计算

## Map

* 计算尽量在map阶段
* 将小数据集成进一步解析成一批<key,value>对，输入map函数中进行处理
* 每一个输入的<k1,v1>都会输入一批<k2,v2>,<k2,v2>是计算的中间结果
* map的数目取决于split的数目，理想的分片大小是一个HDFS块大小

## Reduce

* 输入的中间结果<k2,List<v2>>中的List<v2>表示是一批属于同一个K2的value

## shuffle
map：
输入->map->缓存（内存中环形缓冲区）-> 80%时溢写（分区（哈希分区，发生在内存阶段）、排序（发生在内存阶段）、合并（优化措施，减少磁盘io））->磁盘文件归并 -> reduce阶段

reduce：
归并 -> reduce -> 输出

## 优化：
数据本地化：hadoop尽可能在数据所驻留的HDFS节点上运行map task
减少磁盘io
减少网络传输

Hadoop部署jar包

	hadoop jar wc.jar com.upc.wcdriver <输入路径> <输出路径>

Hadoop Maven依赖

	 <repositories>
	    <repository>
	        <id>alimaven</id>
	        <name>aliyun maven</name>
	        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	    </repository>
	</repositories>
	<dependencies>
	    <dependency>
	        <groupId>org.apache.hadoop</groupId>
	        <artifactId>hadoop-common</artifactId>
	        <version>2.8.3</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.hadoop</groupId>
	        <artifactId>hadoop-hdfs</artifactId>
	        <version>2.8.3</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.hadoop</groupId>
	        <artifactId>hadoop-client</artifactId>
	        <version>2.8.3</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.hadoop</groupId>
	        <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
	        <version>2.8.3</version>
	    </dependency>
	    <dependency>
	        <groupId>org.apache.hadoop</groupId>
	        <artifactId>hadoop-mapreduce-client-common</artifactId>
	        <version>2.8.3</version>
	    </dependency>
	    <dependency>
	        <groupId>junit</groupId>
	        <artifactId>junit</artifactId>
	        <version>3.8.1</version>
	        <scope>test</scope>
	    </dependency>
	</dependencies>