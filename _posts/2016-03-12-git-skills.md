---
layout: default
title: Git 基本操作
---

## 1.配置身份命令：
	git config --global user.name "Tendoasan"
	git config --global user.email "tjuywb@gmail.com"

## 2.创建代码仓库(Repository)
在项目的目录下面，输入：

`git init`

之后会在根目录下生成一个隐藏的`.git`文件夹

## 3.提交本地代码
添加想要提交的代码(分别为添加某文件，添加某目录下所有文件，添加所有文件)：

    git add AndroidManifest.xml
	git add src
	git add .

提交代码(通过`-m`参数来加上提交的描述信息)：

	git commit -m "First commit"

## 4.忽略文件
创建`.gitignore`文件，指定文件或目录排除在版本控制之外，可以使用通配符"*"

	touch .gitignore

文件内容(忽略`bin`目录和`gen`目录)：

	bin/
	gen/

## 5.查看文件修改情况
在项目的根目录下输入：

	git status

当代码文件发生更改，查看所有文件的更改内容：

	git diff

查看特定文件的更改内容：

	git diff src/com/example/providertest/MainActivity.java

未提交的情况(没执行过`add`命令)下，撤销修改：

	git checkout src/com/example/providertest/MainActivity.java

对于已添加的文件，要撤销修改，先取消添加：

	git reset HEAD src/com/example/providertest/MainActivity.java

## 6.查看提交记录
查看历史提交信息：

	git log

查看具体一条记录（指定该记录的`id`，并加上-1表示显示一行）：

	git log 98e88caffd8315287d6dab83b592dd32a7ad8e4d -1

查看具体修改的内容，加上`-p`参数(减号代表删除部分，加号代表添加的部分)：

	git log 98e88caffd8315287d6dab83b592dd32a7ad8e4d -1 -p

## 7.版本控制
查看当前版本库中有哪些分支：

	git branch -a

创建一个分支：

	git branch version1.0

切换到新建分支：

	git checkout version1.0

把`version1.0`分支上修改并提交的内容合并到`master`分支上(可能存在代码冲突)：

	git checkout master
	git merge version1.0

## 8.与远程版本库协作
一个远程版本库的`Git`地址:`https://github.com/example/test.git`
下载远程版本库：

	git clone https://github.com/example/test.git

将本地修改的内容同步到远程版本库上：

	git push origin master

将远程版本库上的修改同步到本地(1)：

	git fetch origin master

同步下来的代码不会合并到任何分支上去，会存放在一个`origin/master`上，可通过`diff`命令查看修改内容：

	git diff origin/master

调用`merge`命令将此分支上的修改合并到主分支：

	git merge origin/master

将远程版本库上的修改同步到本地(2)：

	git pull origin master

相当于`fetch`和`merge`命令合并
