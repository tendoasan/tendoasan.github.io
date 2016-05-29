---
layout: post
title: "第21章 XML Drawables"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","XML","BeatBox","Drawable"]
description: "XML Drawable"
---
{% include JB/setup %}

#### 21.XML Drawables

+ Android把任何可绘制在屏幕上的图形图像都称为`drawable`。drawable可以是一种抽象的图形、一个继承Drawable类的子类，或者是一张位图图像。下列三种drawable文件都是在`XML`文件中定义。
	+ state list drawables
	+ shape drawables
	+ layer list drawables

+ Shape Drawables
	+ 新建一个`shape drawable`文件(深蓝色圆形)：(`res/drawable/button_beat_box_normal.xml`)

		```xml
        <shape xmlns:android="http://schemas.android.com/apk/res/android"
                android:shape="oval">
            <solid android:color="@color/dark_blue"/>
        </shape>
		```

	+ 作为背景添加给`BeatBoxButton`样式。(`res/values/styles.xml`)

		```xml
        <resources>
            <style name="AppTheme" parent="Theme.AppCompat">
                ...
            </style>
            <style name="BeatBoxButton" parent="android:style/Widget.Holo.Button">
                <item name="android:background">@drawable/button_beat_box_normal</item>
            </style>
        </resources>
		```

+ State List Drawables。可根据关联View的不同状态显示不同的drawable。
	+ 新建一个`shape drawable`(红色圆形)文件用于在按钮被按下时显示。(`res/drawable/button_beat_box_pressed.xml`)

		```xml
        <shape xmlns:android="http://schemas.android.com/apk/res/android"
            android:shape="oval">
            <solid android:color="@color/red"/>
        </shape>
		```

	+ 新建一个`state list drawable`文件(`res/drawable/button_beat_box.xml`)

		```xml
        <selector xmlns:android="http://schemas.android.com/apk/res/android">
            <item android:drawable="@drawable/button_beat_box_pressed"
                android:state_pressed="true"/>
            <item android:drawable="@drawable/button_beat_box_normal" />
        </selector>
		```

	+ 将`BeatBoxButton`样式的背景改为新的`state list drawable`文件。(`res/values/styles.xml`)

		```xml
        <resources>
            <style name="AppTheme" parent="Theme.AppCompat">
                ...
            </style>
            <style name="BeatBoxButton" parent="android:style/Widget.Holo.Button">
                <item name="android:background">@drawable/button_beat_box</item>
            </style>
        </resources>
		```

+ Layer List Drawables。将多个`XML drawable`文件组合成一个。给按钮加一个黑环，当按钮被按下时显示。(`res/drawable/button_beat_box_pressed.xml`)

	```xml
	<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
  		<item>
    		<shape android:shape="oval">
				<solid android:color="@color/red"/>
    		</shape>
  		</item>
  		<item>
    		<shape android:shape="oval">
				<stroke
        			android:width="4dp"
        			android:color="@color/dark_red"/>
    		</shape>
  		</item>
	</layer-list>
	```




















































