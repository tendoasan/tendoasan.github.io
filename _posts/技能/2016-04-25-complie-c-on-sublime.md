---
layout: post
title: Sublime Text 3 下配置C&C++编译环境
category: 技能
tags: Sublime
keywords: C,环境,Sublime
description: 
---

Sublime text3提供了构建功能，它的构建系统（Build systems）可以运行一段外部命令，还可以捕获输出并显示。
要在sublime text3中实现C/C\++代码的编译和运行，在本质上说也是调用外部的命令，Windows中也可以理解为执行一段cmd命令。
目前C/C\++编译器最流行的就是gcc和g\++。
安装编译器是后面所有工作的基础，如果没有编译器，后面的一切都无从谈起。在windows下使用gcc和g++，是通过安装MinGW实现的。


# 编译环境

---

## 安装MinGW

[MinGW](http://www.mingw.org/)是Minimalist GNU on Windows的首字母缩写，安装后就可以使用很多的GNU工具。GNU（GNU’s Not Unix）是linux中的一个著名的项目，包含了gcc\g\++\gdb等工具。也就是说，安装MinGw后，就可以使用gcc和g\++命令了。

解压版的[MinGW](http://pan.baidu.com/s/1gd5YzVP)

这是使用 `codeblocks-13.12mingw-setup` 安装后复制出来的。
解压后，可以在 `MinGW/bin` 目录下找到gcc.exe和g\++.exe。把MinGW文件夹放到c盘根目录即可。

## 配置环境变量

右键计算机->属性->高级系统设置->环境变量，双击path，把gcc的路径`C:\MinGW\bin`添加进去。要注意前后的英文分号。


## 在cmd中使用gcc

假设有一个`hello.c`文件在Z盘的work目录下。首先要在cmd中进入此目录。方法可以是在work目录空白处按住Shift点击鼠标右键，选择“在此处打开命令窗口”；也可以使用cd命令进入。
gcc的一般格式是：

	gcc -Wall 源文件名 -o 可执行文件名
	gcc -Wall hello.c -o hello

成功编译生成了可执行文件`hello.exe`后就可以在cmd里运行了。


# Sublime Text 3 下的编译系统

---

## 默认的编译配置文件

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

### C语言

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


### C++

gcc虽然可以编译C\++代码，但是不能进行C\++的连接函数库操作。所以针对C\++代码一般使用g\++来编译。
方法和上面的c语言的配置一样，只要把配置文件中的gcc改为g\++ ，`source.c`改为`source.c++` ，保存文件名`c.sublime-build`改为`c++.sublime-build`就可以了。
这里增加了`-std=c++11` 选项，是按照C\++11标准进行编译，不需要的话可以去掉，配置文件如下：

    {
        "encoding": "utf-8",
        "working_dir": "$file_path",
        "shell_cmd": "g++ -Wall -std=c++11 \"$file_name\" -o \"$file_base_name\"",
        "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
        "selector": "source.c++",

        "variants":
        [
            {
            "name": "Run",
                "shell_cmd": "g++ -Wall -std=c++11 \"$file\" -o \"$file_base_name\" && start cmd /c \"\"${file_path}/${file_base_name}\" & pause\""
            }
        ]
    }

这个配置文件编译的时候也会运行：

	g++ -Wall -std=c++0x $file_name -o $file_base_name && cmd /c ${file_path}/${file_base_name}

如果只想编译，可以把&&后面去掉就可以了。
实际上，可以利用Varians ，来配置多个不同的编译命令。例如下面的配置文件有编译 ，捕获输出运行，cmd运行三种。

    {
        "encoding": "utf-8",
        "working_dir": "$file_path",
        "shell_cmd": "g++ -Wall -std=c++11 \"$file_name\" -o \"$file_base_name\"",
        "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
        "selector": "source.c++",

        "variants":
        [
            {
            "name": "Run in sublime",
                "shell_cmd": "g++ -Wall -std=c++11 \"$file_name\" -o \"$file_base_name\" && cmd /c \"${file_path}/${file_base_name}\""
            },
            {
            "name": "CMD Run",
                "shell_cmd": "g++ -Wall -std=c++11 \"$file\" -o \"$file_base_name\" && start cmd /c \"\"${file_path}/${file_base_name}\" & pause\""
            }
        ]
    }


# 其他

---

## 使用makefile编译多个文件

sublime可以使用makefile来编译多个文件，以便支持稍大一点的工程项目。只要在侧边栏中打开相关的文件夹，确保文件夹中包含makefile文件。此时按下Ctrl+Shift+B ，会有make的选项，点击执行就可以了。


## sublime-build编译系统配置文件

这个编译文件是JSON文件，遵循JSON的语法。JSON 数据的书写格式是：

	“名称”: “值”

例如：

	“firstName” : “John”

值中如果还有双引号要用转义  \” 来表示。

| 名称 | 含义 |
|--------|--------|
|working_dir	|运行cmd是会先切换到working_dir指定的工作目录|
|cmd	|包括命令及其参数。如果不指定绝对路径，外部程序会在你系统的:const:PATH 环境变量中搜索。|
|shell_cmd	|相当于shell:true的cmd ，cmd可以通过shell运行。|
|file_regex	|该选项用Perl的正则表达式来捕获构建系统的错误输出到sublime的窗口。|
|selector	|在选定 Tools->Build System->Automatic 时根据这个自动选择编译系统。|
|variants	|用来替代主构建系统的备选。例如Run命令。会显示在tool的命令中。|
|name	|只在variants下面有，设置命令的名称，例如Run。|



## 支持的变量

只列举了用到的：

| 变量 | 含义 |
|--------|--------|
|$file_path	|当前文件所在目录路径, `e.g., C:\Files`.|
|$file	|当前文件的详细路径, `e.g., C:\Files\Chapter1.txt`.|
|$file_name	|文件全名（含扩展名）, `e.g., Chapter1.txt`.|
|$file_extension	|当前文件扩展名, `e.g., txt`.|
|$file_base_name	|当前文件名（不包括扩展名）, `e.g., Document`.|



变量的使用可以直接使用，也可以使用花括号括起来，例如：`${project_name}`
还可以使用:设置默认值：`${project_name:Default}`

文章来源：http://www.yalewoo.com/sublime_text_3_gcc.html
