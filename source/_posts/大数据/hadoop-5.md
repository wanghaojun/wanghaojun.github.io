---
title: hadoop-5
date: 2018-07-17 08:35:10
tags: hadoop
---
Spark 支持的语言：Scala java python R

Scala
“可扩展的语言” 面向对象+面向函数，运行在JVM上，可以与java兼容，调用java库，
支持脚本，支持交互式编程，支持带main方法的应用程序

<!--more-->

#### 脚本
后缀名为.scala,运行命令
	
	scala <文件>

#### 应用程序
编译：

	scals -c <文件>

生成.class文件，之后与脚本运行方式相同

#### 交互式
进入解释器：
	
	>  scala
	val a = 3  // val指a是常量
	val a:Int = 3  
	var a = 3 //var 指a是变量
	//scala会自动推断类型，可以不指定类型

能声明为val的就不要声明为var

	:paste //可输入多行命令

#### 数据类型
unit = void
any = 任何

#### 函数定义
	
	def info (name:String,age:int):unit{
		println(name+age);
	}

使用位置参数

	info("aaa",21)
使用命名参数 
	
	info(name="aaa",age=21)


