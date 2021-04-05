---
title: JDK配置
date: 2017-11-03 11:32:50
tags: [java,环境搭建]
---
大二开了Java课，上机课，机房环境竟然都没配jdk，看来以后少不了配jdk，于是我整理一下配置JDK的步骤。

<!--more-->

[http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html][1]

这是jdk官网，应该是J2SE，我们上课用的SE，据说大二小学期会用J2EE。下载，然后安装，一路点下来就好。不过要看一下安装目录，这个下面配环境变量会用到。

接下来是配置环境变量

 1. 系统变量→新建 JAVA_HOME 变量 。（本人是C:\Program Files\Java\jdk1.8.0_102）
 2. 系统变量→寻找 Path 变量→编辑
    
    在变量值最后输入 

    %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
    
    （注意原来Path的变量值末尾有没有;号，如果没有，先输入;号再输入上面的代码）
 3. 系统变量→新建 CLASSPATH 变量
    
    变量值填写   

    .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar

   （注意最前面有一点）
    
    jdk的环境变量这就配好了，运行cmd 输入 java -version 如果显示版本信息 则说明安装和配置成功。

ps：配置失败可能是变量名字打错了，或者输入了中文符，仔细检查一下，改正后，需要重启命令行，再试一下。

其实有一个非常简单配置jdk的方法，在我的这篇文章中：window下的包管理器chocolatey。
 只需要在管理员模式的jdk下，输入choco install jdk 就ok了，它会帮你安装jdk，并且配置好环境变量。


  [1]: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html