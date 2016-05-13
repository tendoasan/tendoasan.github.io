---
layout: post
title: "ST3配置C编译环境"
category: "技能"
tags: "Sublime"
keywords: ["C","环境","Sublime"]
description: "ST3配置C编译环境"
---
{% include JB/setup %}

## 编译环境

---

### 安装MinGW

[MinGW](http://www.mingw.org/)是Minimalist GNU on Windows的首字母缩写，安装后就可以使用很多的GNU工具。GNU（GNU’s Not Unix）是linux中的一个著名的项目，包含了gcc\g\++\gdb等工具。也就是说，安装MinGw后，就可以使用gcc和g\++命令了。

解压版的[MinGW](http://pan.baidu.com/s/1gd5YzVP)

这是使用 `codeblocks-13.12mingw-setup` 安装后复制出来的。
解压后，可以在 `MinGW/bin` 目录下找到gcc.exe和g\++.exe。把MinGW文件夹放到c盘根目录即可。

### 配置环境变量

右键计算机->属性->高级系统设置->环境变量，双击path，把gcc的路径`C:\MinGW\bin`添加进去。要注意前后的英文分号。

### 在cmd中使用gcc

假设有一个`hello.c`文件在Z盘的work目录下。首先要在cmd中进入此目录。方法可以是在work目录空白处按住Shift点击鼠标右键，选择“在此处打开命令窗口”；也可以使用cd命令进入。
gcc的一般格式是：

	gcc -Wall 源文件名 -o 可执行文件名
	gcc -Wall hello.c -o hello

成功编译生成了可执行文件`hello.exe`后就可以在cmd里运行了。

## Sublime Text 3 下的编译系统

---

### 默认的编译配置文件

在Sublime的安装目录的Packages文件夹中，有个文件叫`C++.sublime-package`。
这个实际上是zip的压缩包包含了C\++的默认系统设置，修改后缀名为zip后解压，可以在里面找到`C++ Single File.sublime-build`文件，内容如下：

    {
        "shell_cmd": "g++ \"${file}\" -o \"${file_path}/${file_base_name}\"",
        "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
        "working_dir": "${file_path}",
        "selector": "source.c, source.c++",

        "variants":
        [
            {
                "name": "Run",
                "shell_cmd": "g++ \"${file}\" -o \"${file_path}/${file_base_name}\" && \"${file_path}/${file_base_name}\""
            }
        ]

这是JSON格式的配置文件，可以看到 selector部分确实是C和C\++都选择的。
建议把用户配置放到用户文件夹下，来代替默认的编译配置。


## 新建编译系统

---

选择`tool –> Build System –> New Build System`
然后输入以下代码：

    {
        "working_dir": "$file_path",
        "cmd": "gcc -Wall \"$file_name\" -o \"$file_base_name\"",
        "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
        "selector": "source.c",

        "variants":
        [
            {
            "name": "Run",
                "shell_cmd": "gcc -Wall \"$file\" -o \"$file_base_name\" && start cmd /c \"\"${file_path}/${file_base_name}\" & pause\""
            }
        ]
    }

按Ctrl+s保存，会自动打开user目录（`Sublime Text 3\Packages\User`），修改文件名为 `C.sublime-build`，保存在此目录。
这时候，可以在`Tools -> Build System`下看到刚才新建的C，选中后就可以使用了。
Build System中除了选择具体的编译系统，还可以选择第一个：Automatic 自动选择，会根据打开的文件后缀自动选择。

用sublime打开.c文件，按Ctrl+Shift+B，第一个c就是对应执行配置文件中的第三行：

	gcc -Wall $file_name -o $file_base_name

作用是编译。

第二个c-Run对应后面的命令：

	gcc -Wall $file -o $file_base_name && start cmd /c \”${file_path}/${file_base_name} & pause\” 

作用是是在新的cmd窗口运行。这样就可以对scanf等函数进行输入了。

文章来源：[http://www.yalewoo.com/sublime_text_3_gcc.html](http://www.yalewoo.com/sublime_text_3_gcc.html)
