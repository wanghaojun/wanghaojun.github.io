---
title: win10环境配置
date: 2021-05-12 11:42:41
tags: [环境搭建]
---

记录一下win10需要配置的环境

<!--more-->

#### 命令行

- [chocolatey](https://chocolatey.org/install)  特别好用的windows环境下的包管理器，安装完成后，python,java,git,maven,nodejs等环境都可以在这里配置

- [windows terminal](https://docs.microsoft.com/zh-cn/windows/terminal/get-started)，感觉UI要比powershell好看，可以结合wsl一起使用

- [gsudo](https://github.com/gerardog/gsudo) 添加管理员版本的shell：

  安装gsudo，在管理员页面下使用chocolatey安装：

  ```powershell
  choco install gsudo
  ```

  然后可以在windows terminal中新建一个powershell的配置，在命令行那一栏使用

  ```
  gsudo.exe powershell.exe
  ```

  图标可以用这个[windows terminal icon](https://i.imgur.com/Giuj3FT.png)。

- [wsl](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10) 适用于 Linux 的 Windows 子系统。

#### 开发环境

- anaconda 建议[清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/)
- jdk 建议直接用Chocolatey安装
- nodejs 直接用Chocolatey安装

#### IDE

- pycharm-python
- idea-java

#### 文本编辑

- sublime text 直接用Chocolatey安装
- Typora markdown编辑器 
- Gaaiho Reader pdf阅读器
- 知云文献翻译 英语渣渣的学术必备 也是pdf阅读器
- [幕布](https://mubu.com/) 大纲笔记，还是蛮好用的
- xmind 思维导图 
- onenote for windows10 笔记工具

#### 其它

- potplayer  视频播放软件
- xshell 服务器远程连接
- mathpix 数学公式识别
- PicGo 上传图床工具
- snipaste 贴图截图工具
- ev录屏 录屏软件