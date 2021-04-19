---
title: hadoop-4
date: 2018-07-16 08:32:41
tags: [大数据]
---
Hadoop2 & Yarn  & hive

<!--more-->

Yarn的目标时实现“一个集群多个框架”,实现一个集群上的不用应用负载。

## 调度器
Yarn提供了接口用于实现可拔插的调度器
YARN有两个预定的调度器
* Fair Scheduler
* Capacity Scheduler
策略：
* FIFO：先进先出
* Capacity： 小人物单独通道
* Fair：新的作业提交，划分一半资源

## Kafka
高吞吐量的分布式发布订阅消息系统，用户通过Kafka系统可以发布大量的消息，同时实时订阅消费消息
在大数据生态系统中，Kafka作为数据交换枢纽，不同类型的分布式系统，可以统一接入Kafka，实现Hadoop各个组件间高效的数据交换

## Hive
基于Hadoop的数据仓库工具，它将Hdfs中的结构化数据映射为数据表，并且将类SQL脚本转化为MapReduce作业。定义了简单的类SQL查询语言，成为HQL。
安装前提：
* jdk 1.7+
* hadoop -2.2.0+
* Mysql数据库 v5.6.x
* Hive安装在与Mysql同一台机器上

#### 环境配置
1.解压完毕hive压缩包后，切换目录到/simple/hive-0.12.0目录并查看下面的文件列表。

2.在/simple/hive-0.12.0目录下执行命令：cd conf切换到conf目录并查看列表，执行命令：cp hive-env.sh.template hive-env.sh。

3.在/simple/hive-0.12.0/conf目录下执行：vim hive-env.sh并编辑内容。

4.在/simple/hive-0.12.0目录下执行命令：cd conf切换到conf目录并查看列表，执行命令：mv hive-default.xml.template  hive-site.xml。

5.完成上一步操作之后，此时需要修改hive-site.xml文件的内容，由于hive-site.xml中内容较多，我们需要在本地打开文件进行删除文件中的内容，单击桌面Computer->Filesystem->simple->hive-0.12.0->conf,右击hive-site.xml文件选择Open With gedit进行编译，删除
所有内容，此操作会比较耗时，操作完之后再终端执行命令：vim hive-site.xml之后并查看内容，注意：mysql url路径地址的ip地址根据本机情况进行修改。


	<configuration>
	  <property>
	    <name>javax.jdo.option.ConnectionURL</name>
	    <value>jdbc:mysql://localhost:3306/hive</value> 
	  </property>
	  <property>
	    <name>javax.jdo.option.ConnectionDriverName</name>
	    <value>com.mysql.jdbc.Driver</value>
	  </property>
	  <property>
	    <name>javax.jdo.option.ConnectionUserName</name>
	    <value>root</value>
	  </property>
	  <property>
	    <name>javax.jdo.option.ConnectionPassword</name>
	    <value>123456</value>
	  </property>
	</configuration>


6.完成上一步之后，在目录/simple/hive-0.12.0/bin下面，修改文件hive-config.sh，增加以下内容：
	
	export JAVA_HOME=/simple/jdk1.7.0_79
	export HIVE_HOME=/simple/hive-0.12.0
	export HADOOP_HOME=/simple/hadoop-2.4.1。

7.在命令终端任意目录下，执行命令：vim /etc/profile然后编辑内容，进行hive环境变量的配置。配置完毕后执行命令source /etc/profile进行刷新。

8.配置完环境变量之后，可以执行hive命令，进入./hive进入hive shell，环境表示安装配置成功。

ps： 在测试的过程中有可能文件权限问题，对应目录主要有两个：
       1)hdfs://192.168.0.201:9000/tmp
       2)/tmp
   可以通过如下命令修改hdfs上的tmp和本地tmp文件夹权限的修改：
hdfs dfs -chmod -R 1777 /tmp //hdfs上的文件权限
chmod -R 1777 /tmp //linux文件权限。 

#### hive 数据模型
内部表 命名空间对应文件系统的一个目录
外部表
分区表
分桶表

#### 创建表的语法
数据类型
运算符

#### 简单的查询语法

#### 使用函数
内置函数
分析函数（窗口函数）
自定义函数

#### 使用批处理模式
.hql脚本

#### 保存结果


