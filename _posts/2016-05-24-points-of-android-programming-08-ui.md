---
layout: post
title: "第08章 UI"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","UI","CriminalIntent"]
description: "使用布局与组件创建用户界面"
---
{% include JB/setup %}

#### 8.使用布局与组件创建用户界面

+ 样式（style）是XML资源文件，含有用来描述组件行为和外观的属性定义。例如，下列样式资源就是用来配置组件，使其显示的文字大小大于正常值的一段代码。

	```xml
	<style name="BigTextStyle">
		<item name="android:textSize">20sp</item>
		<item name="android:layout_margin">3dp</item>
	</style>
	```

	将属性定义添加并保存在`res/values/`目录下的样式文件中，然后在布局文件中以`@style/my_own_style（样式文件名）`的形式引用它们。

+ 视图最常见的属性有：
	+ 文字大小（text size），指设备上显示的文字像素高度；
	+ 边距（margin），指定视图组件间的距离；
	+ 内边距（padding），指定视图外边框与其内容间的距离。

+ dp & sp：
	+ dp：`density-independent pixel`的缩写形式，意为密度无关像素。在设置边距、内边距或任何不打算按像素值指定尺寸的情况下，通常都使用dp这种单位。1dp单位在设备屏幕上总是等于`1/160英寸`。
	+ sp：`scale-independent pixel`的缩写形式，意为缩放无关像素。我们通常会使用sp来设置屏幕上的字体大小。

+ Android开发设计原则：使用`16dp`单位值设定边距尺寸。该单位值的设定遵循了Android的`48dp调和`设计原则。访问网址：[http://developer.android.com/design/index.html](http://developer.android.com/design/index.html)，可查看Android所有的开发设计原则。

+ 布局参数：名称以`layout_`开头的属性作用于组件的父组件，这些属性统称为布局参数。它们会告知父布局如何在内部安排自己的子元素。即使布局对象（如LinearLayout）是布局的根元素，它仍然是一个带有布局参数的子组件。

+ 如果一个组件只存在于一个布局上，则需先在代码中进行空值检查，确认当前方向的组件存在后，再调用相关方法：

	```java
    Button landscapeOnlyButton = (Button)v.findViewById(R.id.landscapeOnlyButton);
    if (landscapeOnlyButton != null) {
    	// Set it up
    }
	```

+ 挑战练习：日期格式化，使用`android.text.format.DateFormat`类。