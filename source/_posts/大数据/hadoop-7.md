---
title: hadoop(7)
date: 2018-07-19 09:21:17
tags: hadoop
---
sqoop

<!--more-->

## Sqoop伪分布式环境搭建
第一步：解压sqoop压缩包sqoop-1.99.7-bin-hadoop200.tar.gz
1)	执行解压命令
2)	查看解压目录
3)	配置环境变量，配置完毕之后，需要执行命令:source /etc/profile，使其生效,如图25所示
第二步：配置sqoop配置文件
1）确认是否把Hadoop的安装目录配置到环境变量中 ,如果没有需要在/etc/profile中添加
export HADOOP_HOME=/simple/Hadoop-2.7.2
2）配置第三方jar引用路径
export SQOOP_SERVER_EXTRA_LIB=$SQOOP_HOME/extra 
4）	复制sqoop/conf/sqoop-env-template.sh重命名为sqoop-env.sh
5）	修改sqoop-env.sh的属性
6）	修改${SQOOP_HOME}/bin/configure-sqoop里面的配置
注释掉HCatalog等不用的组件
第三步： 验证是否安装成功
1）	启动hadoop，输入 start-all.sh可全部启动。
2）	启动mysql服务器 
3）	启动sqoop，在${sqoop}的bin目录下有一个sqoop,执行` ./sqoop`
4）	输入命令链接mysql数据库，列出数据库列表

	./sqoop list-databases --connect jdbc:mysql://localhost:3306/ --username root --password 123456

## 总结

通过这次实验，我在虚拟机上分别搭建了Hadoop伪分布式环境、Hive伪分布式环境以及Sqoop伪分布式环境，对Hadoop大数据生态环境下的三个组件有了一定的了解。
在一台计算机上搭建hadoop环境，让hadoop的核心：hdfs和yarn运行在同一台机器上，虽然没有达到真实的分布式存储和分布式计算环境，但是能让我们认识到hadoop每个组件的作用，学习mapreduce的分布式计算的过程，能让我对hadoop整个体系有个初步的了解。
通过Hive伪分布式的搭建，对Hive有了初步的认识和了解，Hive可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 
在搭建Sqoop伪分布式的过程中，认识到sqoop可以将一个关系型数据库（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中，是一个强大的在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递的工具。
实验二使用MapReduce实现对HDFS上的日志文件处理并把处理结果输出到HDFS，让我对mapreduce的数据处理能力有了一定的认识，也对mapreduce进行离线计算的开发有了一定理解。MapReduce提供了一个庞大但设计精良的并行计算软件框架，能自动完成计算任务的并行化处理，自动划分计算数据和计算任务，在集群节点上自动分配和执行任务以及收集计算结果，将数据分布存储、数据通信、容错处理等并行计算涉及到的很多系统底层的复杂细节交由系统负责处理，大大减少了软件开发人员的负担。




