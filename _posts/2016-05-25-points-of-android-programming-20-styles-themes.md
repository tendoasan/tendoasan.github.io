---
layout: post
title: "第20章 样式与主题"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","样式","BeatBox","主题"]
description: "Audio Playback"
---
{% include JB/setup %}

#### 20.样式与主题

+ 样式。
	+ 新建一个`BeatBoxButton`的样式。(`res/values/styles.xml`)

		```xml
		<resources>
			<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
			</style>
			<style name="BeatBoxButton">
				<item name="android:background">@color/dark_blue</item>
			</style>
		</resources>
		```

	+ 在`Button`布局文件中应用这个样式。(`res/layout/list_item_sound.xml`)

		```xml
		<Button xmlns:android="http://schemas.android.com/apk/res/android"
			xmlns:tools="http://schemas.android.com/tools"
			style="@style/BeatBoxButton"
			android:id="@+id/button"
			android:layout_width="match_parent"
			android:layout_height="120dp"
			tools:text="Sound name"/>
		```

	+ 样式继承的两种写法。
		+ `BeatBoxButton.Strong`。(`res/layout/styles.xml`)

			```xml
			...
			<style name="BeatBoxButton">
				<item name="android:background">@color/dark_blue</item>
			</style>
			<style name="BeatBoxButton.Strong">
				<item name="android:textStyle">bold</item>
			</style>
			...
			```

		+ `parent="@style/BeatBoxButton"`。

			```xml
			...
			<style name="BeatBoxButton">
				<item name="android:background">@color/dark_blue</item>
			</style>
			<style name="StrongBeatBoxButton" parent="@style/BeatBoxButton">
				<item name="android:textStyle">bold</item>
			</style>
			···
			```

+ 主题。
	+ 修改主题为`dark theme`。(`res/values/styles.xml`)

		```xml
        <resources>
            <style name="AppTheme" parent="Theme.AppCompat">
            </style>
        ...
        </resources>
		```

	+ 添加主题颜色，重载主题属性`android:colorBackground`和`android:buttonStyle`。(`res/values/styles.xml`)

		```xml
        <resources>
            <style name="AppTheme" parent="Theme.AppCompat">
                <item name="colorPrimary">@color/red</item>
                <item name="colorPrimaryDark">@color/dark_red</item>
                <item name="colorAccent">@color/gray</item>

                <item name="android:colorBackground">@color/soothing_blue</item>
                <item name="android:buttonStyle">@style/BeatBoxButton</item>
            </style>

            <style name="BeatBoxButton" parent="android:style/Widget.Holo.Button">
                <item name="android:background">@color/dark_blue</item>
            </style>
        </resources>
		```

	+ 当需要引用一个主题中的资源时，使用`?`注解。

		```xml
		<Button xmlns:android="http://schemas.android.com/apk/res/android"
			...
			android:background="?attr/colorAccent"
			tools:text="Sound name"/>
		```

+ 挑战练习：一个合适的`Base Theme`





























































