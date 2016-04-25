---
layout: post
title: Hello My Site!
---
##1.输入法不能正常拼音的问题
打开终端，输入:
`ibus-daemon -drx`

##2.右键打开终端
打开终端，输入:
`sudo apt-get install nautilus-open-terminal`

##3.修改grub2等待时间
修改`/etc/default/grub`文件和`/etc/grub.d/`目录下的文件
打开第一个文件，修改如下代码:
`GRUB_TIMEOUT=0`
之后打开`/etc/grub.d/30_os-prober`文件，修改如下代码:
```gedit
set timeout_style=menu
if [ "\${timeout}" = 0 ]; then
  set timeout=0
fi
```
更新以上修改:
`update-grub`
之后重启电脑

##4.给文件添加可执行权限
`+` 添加某个权限
`x` 可执行
`chmod +x 某文件`，执行后命令行下查看该文件成绿色。

##5.删除目录
`sudo rm -rf 目录`

##6.执行studio.sh
`./studio.sh`

##7.Axel命令使用方法
`axel 参数 文件下载地址`
可选参数:
```
-n 指定线程数
-o 指定另存为目录
-s 指定每秒的最大比特数
-q 静默模式
```
如从`Diahosting`下载`lnmp`安装包指定10个线程，存到`/tmp/`：`axel -n 10 -o /tmp/ http://soft.vpser.net/lnmp/lnmp0.7-full.tar.gz`

如果下载过程中下载中断可以再执行下载命令即可恢复上次的下载进度。

##8.ubuntu 14.0 更新源列表失败， Hash校验不符？
首先备份源列表(for sure):
`sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup`
而后用gedit或其他编辑器打开:
`gksu gedit /etc/apt/sources.list`
从下面列表中选择合适的源，替换掉文件中所有的内容，保存编辑好的文件:
`http://wiki.ubuntu.org.cn/%E6%BA%90%E5%88%97%E8%A1%A8`
 注意：一定要选对版本
然后，刷新列表:
`sudo apt-get update`

##9.打开缓存失败
`sudo rm /var/lib/apt/lists/* -vf `
`sudo apt-get update`

##10.关闭内部错误提醒
`sudo gedit  /etc/default/apport `
把enabled=1的值改成0 即
`enabled=0 `










