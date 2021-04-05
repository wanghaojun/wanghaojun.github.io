---
title: hadoop(1)
date: 2018-07-13 09:06:13
tags: hadoop 
---

##大数据简介
* 数据产生方式： 运营式系统——用户原创——传感器感知
* 大数据特征：体量大、多样化、快速化、价值密度低

<!--more-->

##Hadoop:
* Hadoop common
* Hadoop Distribute File System(HDFS)
* Hadoop YARN  工作分配+资源调度 操作系统的作用
* Hadoop MapReduce  分布式离线计算

unbuntu 下jdk的配置

	JAVA_HOME=/home/whj/Desktop/jdk1.8.0_171
	export PATH =$JAVA_HOME/bin:$PATH
	
	source /etc/profile

	sudo update-alternatives --install /usr/bin/java java /home/whj/Desktop/jdk1.8.0_171/bin/java 300  
	sudo update-alternatives --install /usr/bin/javac javac /home/whj/Desktop/jdk1.8.0_171/bin/javac 300  
	sudo update-alternatives --install /usr/bin/jar jar /home/whj/Desktop/jdk1.8.0_171/bin/jar 300   
	sudo update-alternatives --install /usr/bin/javah javah /home/whj/Desktop/jdk1.8.0_171/bin/javah 300   
	sudo update-alternatives --install /usr/bin/javap javap /home/whj/Desktop/jdk1.8.0_171/bin/javap 300

core-site.xml

	<property>
		<name>fs.default.name</name>
		<value>hdfs://192.168.159.128:9000</value>
	</property>

	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://192.168.159.128:9000</value>
	</property>

	<property>
		<name>hadoop.tmp.dir</name>
		<value>/home/whj/Desktop/hadoop-2.8.3/tmp</value>
	</property>

hdfs-site.xml

		<property>
			<name>dfs.replication</name>
			<value>1</value>
		</property>

		<property>
			<name>dfs.data.dir</name>
			<value>/home/whj/Desktop/hadoop-2.8.3/hdfs/name</value>
		</property>

		<property>
			<name>dfs.data.dir</name>
			<value>/home/whj/Desktop/hadoop-2.8.3/hdfs/data</value>
		</property>

mapred-site.xml

	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>

yarn-site.xml

	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>192.168.159.128</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>