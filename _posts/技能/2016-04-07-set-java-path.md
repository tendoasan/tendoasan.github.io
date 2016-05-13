---
layout: post
title: Java 环境配置
category: 技能
tags: Java
keywords: Java,环境
description: 
---

# ubuntu下

## 1.源码包准备
首先到官网下载`jdk1.7`。

[下载链接](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

下载到主目录。

## 2.解压源码包
通过终端在`usr/local'目录下新建java文件夹，命令行:

	sudo mkdir /usr/local/java

将下载到的压缩包拷贝到java文件夹中，在主目录下打开终端后，命令行:

	sudo cp jdk-7u79-linux-x64.tar.gz /usr/local/java

然后进入java目录，命令行:

	cd /usr/local/java

解压压缩包，命令行:

	sudo tar xvf jdk-7u79-linux-x64.tar.gz

解压成功后删除压缩包，命令行:

	sudo rm jdk-7u79-linux-x64.tar.gz

## 3.设置jdk环境变量
这里采用全局设置方法，它是所有用户共用的环境变量

	sudo gedit ~/.bashrc

打开文件后在末尾添加

	export JAVA_HOME=/usr/local/java/jdk1.7.0_79
	export JRE_HOME=${JAVA_HOME}/jre
	export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
	export PATH=${JAVA_HOME}/bin:$PATH

注意等号两侧不要加入空格。
更新：

	source ~/.bashrc

## 4.检验是否安装成功
打开终端输入：

	java -version

显示结果：

	java version "1.7.0_79"
	Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
	Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

---

# windows 7下

## 1.安装Java

首先需要下载Java开发工具包JDK，

[下载地址](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

点击对应的下载按钮开始下载，JDK的安装根据提示进行，之后继续安装JRE。

安装JDK可以自定义安装目录等信息，例如选择安装目录为：`C:\Program Files\Java\jdk1.7.0 `。


## 2.配置环境变量

安装完成后，点击我的计算机，右键属性→“高级系统设置“→”环境变量”→“系统变量“下设置如下三个属性值：

变量名：`JAVA_HOME`

变量值：`C:\Program Files\Java\jdk1.7.0`

变量名：`Path`

变量值：`%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;`

变量名：`CLASSPATH`

变量值：`.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar; //记得前面有个"."`


这是java的环境配置，配置完成后直接启动eclipse，它会自动完成java环境的配置。

注意：如果使用1.5以上版本的JDK，不用设置CLASSPATH环境变量，也可以正常编译和运行Java程序。


## 3.测试JDK是否安装成功

- "开始"→"运行"，键入"cmd"；

- 键入命令"java -version"，"java"，"javac"几个命令，出现画面，说明环境变量配置成功；

