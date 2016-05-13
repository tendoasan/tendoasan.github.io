---
layout: post
title: Sublime Text 3 下配置Java编译环境
category: 技能
tags: Sublime
keywords: Java,环境,Sublime
description: 
---

## 新建Java编译系统

1.打开 菜单栏>工具>编译系统>新编译系统
删除原来的内容，输入下面的内容保存为`Java.sublime-build`

	{
		"shell_cmd": "runJava.bat \"$file\"",
		"file_regex": "^(...*?):([0-9]*):?([0-9]*)",
		"selector": "source.java",
		//"encoding": "GBK"  设置文件编码为GBK
		"encoding": "UTF-8"   //设置文件编码为UTF-8
	}

如果保存不成功，重新以管理员身份打开sublime重复上面的操作。

2.新建一个文本文档输入下面内容保存为`runJava.bat`，然后复制这个文件到对应的jdk下的bin目录。默认的就是`C:\Program Files\Java\jdk1.8.0_45\bin`目录。

	@ECHO OFF
	cd %~dp1
	ECHO Compiling %~nx1.......
	IF EXIST %~n1.class (
	DEL %~n1.class
	)
	javac %~nx1
	IF EXIST %~n1.class (
	ECHO -----------OUTPUT-----------
	java %~n1
	)

3.写好java文件，选择编译系统为Java，Ctrl + b运行。

	public class hello{
		public static void main(String args[]){
			System.out.println("Hello Java 测试中文"); //中文注释
		}
	}



