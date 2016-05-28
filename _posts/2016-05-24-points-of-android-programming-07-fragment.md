---
layout: post
title: "第07章 Fragment"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Fragment","CriminalIntent"]
description: "UI Fragment 与Fragment 管理器"
---
{% include JB/setup %}

#### 7.UI Fragment 与Fragment 管理器

+ 生成独有ID：`mId = UUID.randomUUID();`
+ 为托管UI fragment，activity必须做到：
	+ 在布局中为fragment 的视图安排位置；
	+ 管理fragment 实例的生命周期。
+ Fragment的生命周期图解：

![]({{ IMAGE_PATH }}/Fragment的生命周期图解.jpg)

+ 在activity 中托管一个UI fragment，有如下两种方式：
	+ 添加fragment 到activity 布局中；
	+ 在activity 代码中添加fragment。
+ 创建一个UI fragment的步骤如下所示：
	+ 通过定义布局文件中的组件，组装界面；
	+ 创建fragment 类并设置其视图为定义的布局；
	+ 通过代码的方式，连接布局文件中生成的组件。
+ FragmentManager 类负责管理fragment 并将它们的视图添加到activity 的视图层级结构中。FragmentManager类具体管理的是：
	+ fragment队列；
	+ fragment事务的回退栈
+ FragmentManager 关系图：

	![]({{ IMAGE_PATH }}/FragmentManager 关系图.jpg)

+ 要通过代码的方式将fragment添加到activity 中，可直接调用activity 的FragmentManager：

	```java
	FragmentManager fm = getSupportFragmentManager();
	```

+ 创建并提交一个fragment 事务：

	```java
	if (fragment == null) {
		fragment = new CrimeFragment();
		fm.beginTransaction()
			.add(R.id.fragmentContainer, fragment)
			.commit();
	}
	```

	+ fragment 事务被用来`添加`、`移除`、`附加`、`分离`或`替换`fragment队列中的fragment。
	+ FragmentManager 管理着fragment 事务的回退栈。`FragmentManager.beginTransaction()`方法创建并返回`FragmentTransaction`实例。
	+ `FragmentTransaction`类使用了一个`fluent interface`接口方法，通过该方法配置`FragmentTransaction`返回`FragmentTransaction`类对象，而不是void，由此可得到一个`FragmentTransaction`队列。
	+ 因此上述代码可解读为：“创建一个新的fragment事务，加入一个添加操作，然后提交该事物。” `add(...)`方法是整个事务的核心部分，并含有两个参数，即`容器视图资源ID`和新创建的`CrimeFragment`。容器视图资源ID即定义在`activity_crime.xml`中的 `FrameLayout`组件的资源ID。容器视图资源ID主要有两点作用：
		+ 告知`FragmentManager` fragment视图应该出现在activity视图的什么地方；
		+ 是FragmentManager队列中fragment的唯一标识符。
		如需从FragmentManager中获取CrimeFragment，即可使用容器视图资源ID：

			```java
			Fragment fragment = fm.findFragmentById(R.id.fragmentContainer);
			```

+ activity 的`FragmentManager`负责调用队列中fragment的生命周期方法。添加fragment供`FragmentManager`管理时，`onAttach(Activity)`、`onCreate(Bundle)`以及`onCreateView(...)`方法会被调用。
	+ 托管`activity的onCreate(...)`方法执行后，`onActivityCreated(...)`方法也会被调用。因为我们正在向`CrimeActivity.onCreate(...)`方法中添加CrimeFragment，所以fragment被添加后，该方法会被调用。
	+ 在activity处于停止、暂停或运行状态下时，，`FragmentManager`会立即驱使fragment快速跟上activity的步伐，直到与activity的最新状态保持同步。例如，向处于运行状态的activity中添加fragment时，以下fragment生命周期方法会被依次调用：`onAttach(Activity)`、`onCreate(Bundle)`、`onCreateView(...)`、`onActivityCreated(Bundle)`、`onStart()`，以及`onResume()`方法。
	+ 只要fragment的状态与activity的状态保持了同步，托管activity的`FragmentManager`便会继续调用其他生命周期方法以继续保持fragment与activity的状态一致，而几乎就在同时，它接收到了从操作系统发出的相应调用。但fragment方法究竟是在activity方法之前还是之后调用的这一点是无法保证的。

+ `AUF（Always Use Fragments）`原则，即“总是使用fragment”原则。