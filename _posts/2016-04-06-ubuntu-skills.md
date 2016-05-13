---
layout: post
title: "ubuntu 常用操作"
category: "技能"
tags: ["ubuntu","操作"]
keywords: ["ubuntu","操作"]
description: "ubuntu 常用操作"
---

{% include JB/setup %}

## 优化操作

---

## 1.修改grub2等待时间
修改`/etc/default/grub`文件和`/etc/grub.d/`目录下的文件
打开第一个文件，修改如下代码:

	GRUB_TIMEOUT=0

之后打开`/etc/grub.d/30_os-prober`文件，修改如下代码:

	set timeout_style=menu
	if [ "\${timeout}" = 0 ]; then
	  set timeout=0
	fi

更新以上修改:

	update-grub

之后重启电脑

## 2.给文件添加可执行权限
`+` 添加某个权限

`x` 可执行

执行如下命令后，终端下查看该文件名呈绿色。

	chmod +x 某文件

## 3.安装`.deb`文件

	sudo dpkg -i xxxx.deb

## 4.删除目录

	sudo rm -rf 目录

## 5.执行studio.sh

	./studio.sh

## 6.Axel命令使用方法

	axel 参数 文件下载地址

可选参数:

	-n 指定线程数
	-o 指定另存为目录
	-s 指定每秒的最大比特数
	-q 静默模式

如从`Diahosting`下载`lnmp`安装包指定10个线程，存到`/tmp/`：

	axel -n 10 -o /tmp/ http://soft.vpser.net/lnmp/lnmp0.7-full.tar.gz

如果下载过程中下载中断可以再执行下载命令即可恢复上次的下载进度。


## 解压缩命令

---

## ZIP
zip的优点就是可以在不同的操作系统平台，比如Linux， Windows以及Mac OS上使用。缺点就是支持的压缩率不是很高，而tar.gz和tar.gz2在压缩率方面做得非常好。

使用下列命令压缩一个目录：

	zip -r archive_name.zip directory_to_compress

解压一个zip文档：

	unzip archive_name.zip

## TAR
TAR的好处就是只消耗非常少的CPU以及时间去打包文件，仅仅是一个打包工具，并不负责压缩。

使用下列命令打包一个目录：

	tar -cvf archive_name.tar directory_to_compress

解包：

	tar -xvf archive_name.tar.gz

上面这个解包命令将会将文档解开在当前目录下面。也可以指定解包的路径：

	tar -xvf archive_name.tar -C /tmp/extract_here/

## TAR.GZ
TAR.GZ在压缩时不会占用太多CPU的，而且可以得到一个非常理想的压缩率。

使用下列命令压缩一个目录：

	tar -zcvf archive_name.tar.gz directory_to_compress

解压缩：

	tar -zxvf archive_name.tar.gz

上面这个解包命令将会将文档解开在当前目录下面。也可以指定解包路径：
`tar -zxvf archive_name.tar.gz -C /tmp/extract_here/`

## TAR.BZ2
TAR.BZ2是上述所有方式中压缩率最好的。但是，它比前面的方式要占用更多的CPU与时间。

使用下列命令压缩一个目录：

	tar -jcvf archive_name.tar.bz2 directory_to_compress

上面这个解包命令将会将文档解开在当前目录下面。也可以指定解包路径：

	tar -jxvf archive_name.tar.bz2 -C /tmp/extract_here




