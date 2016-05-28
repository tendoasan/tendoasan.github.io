---
layout: post
title: "MarkDown文件中引用图片"
category: "技能"
tags: ["MarkDown"]
keywords: ["MarkDown","图片"]
description: "MarkDown文件中引用图片"
---
{% include JB/setup %}

### 利用Github存储图片

- 将markdown文件需要用到的图片放到git仓库中，发布到github上

- 访问github仓库： [tendoasan/MarkdownPhotos](https://github.com/tendoasan/MarkDownPhotos)
- 访问图片： [MarkdownPhotos/Avatar.jpg](https://github.com/tendoasan/MarkDownPhotos/blob/master/Res/Avatar.jpg)
- 点击Raw按钮，拷贝链接地址：
	https://raw.githubusercontent.com/tendoasan/MarkDownPhotos/master/Res/Avatar.jpg

- 在Markdown文件中引用图片：

        ![Avatar](链接地址)


### 引用本地存储图片

- 使用相对路径插入图片。比如你把一个叫做·test.png·的图片和·*.md·文件放在一起，那么你就可以用这种方式插入图片：

		![](test.png)

- 如果不想放在同一层级,那么就可以这样插入:

		![](foldername/test.png)
		// 表示引用同层级一个叫做"foldername"的文件夹中的test.png图片
