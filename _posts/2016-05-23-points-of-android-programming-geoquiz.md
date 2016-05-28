---
layout: post
title: "第01-06章 GeoQuiz"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","要点","GeoQuiz"]
description: "GeoQuiz"
---
{% include JB/setup %}

#### 1.第一个Android 应用

+ Toast方法：

	```java
	public static Toast makeText(Context context, int resId, int duration);
	```

其中`Context`变量使得`Toast`类能够找到并使用`string`的资源ID。

+ 在构建过程中，`Android tools`将资源，代码与`AndroidManifest.xml(包含应用的元数据)`文件打包成一个`.apk`文件。
+ 作为构建过程中的一部分，`aapt(Android Asset Packaging Tool)`将布局(`layout`)文件编译成一个更加复杂的格式。
+ `Android build tools`。Windows命令行下使用`gradle`构建项目：

	```cmd
	> gradlew.bat tasks
	> gradlew.bat installDebug
	```

#### 2.Android 与MVC 设计模式

+ 模型对象(`Model`)，用于存储着应用的数据和业务逻辑。模型类通常被设计用来映射与应用相关的一些事物。
+ 视图对象(`View`)，知道如何在屏幕上绘制自己以及如何响应用户的输入，比如用户的触摸操作等。一个简单的经验法则是：凡是能够在屏幕上看见的对象，就是视图对象。
+ 控制对象(`Controller`)，包含了应用的逻辑单元，是视图与模型对象的联系纽带。控制对象被设计用来响应由视图对象触发的各类事件，此外还用来管理模型对象和视图层间的数据流动。在Android里，控制器通常是Activity、Fragment或Service的一个子类。
+ MVC 数据控制流与用户交互：

	![]({{ IMAGE_PATH }}/MVC 数据控制流与用户交互.jpg)

#### 3.Activity 的生命周期

+ Activity 的每一个实例都有一个生命周期。在其生命周期内，activity 在`running`, `paused`和`stopped`三种可能的状态间进行转换。每次状态发生变化时，都有一个Activity 方法将状态改变的消息通知给activity。
+ Activity 的状态图解：

	![]({{ IMAGE_PATH }}/activity_state.jpg)

+ 通常，一个activity 通过覆盖`onCreate(...)`方法来准备一下用户界面的相关工作：
	+ 实例化组件并将组件放置在屏幕上(`setContentView(int)`)
	+ 引用已实例化的组件
	+ 为组件设置监听器以处理用户交互
	+ 访问外部模型数据

+ `Android`会在适时的时机去调用`onCreate(...)`方法或任何其他Activity生命周期方法。
+ 输出日志信息：

	```java
	public static int d(String tag, String msg);
	```

+ 使用`@Override`注解，要求编译器保证当前类具有你即将要覆盖的方法。
+ `FrameLayout`是一种最简单的`ViewGroup`组件，它不以特定方式安排其子视图的位置。其子视图的位置排列都是由它们各自的`layout_gravity`属性决定的。
+ 在设备运行时发生配置变更时，例如设备旋转，需采用某种方式保存以前的数据。其中一种方式就是覆盖Activity的方法`onSaveInstanceState(...)`：

	```java
	private static final String KEY_INDEX = "index";
	@Override
	protected void onSaveInstanceState(Bundle savedInstanceState){
		super.onSaveInstanceState(savedInstanceState);
		savedInstanceState.putInt(KEY_INDEX, mIntData);
	}
	```

	系统会在`onPause()`, `onStop()`和`onDestroy()`方法之前调用此方法。
	之后在`onCreate(...)`方法中检查存储的`bundle`信息：

	```java
	@Override
	public void onCreate(Bundle savedInstanceState) {
		if (savedInstanceState != null) {
				mIntData = savedInstanceState.getInt(KEY_INDEX, 0);
			}
			super.onCreate(savedInstanceState);
		}
	}
	```

+ 完整的Activity 生命周期：

	![]({{ IMAGE_PATH }}/完整的Activity 生命周期.jpg)

#### 4.调试Android 应用

+ 遇到运行异常时，记住在`LogCat`中寻找最后一个异常及其栈追踪的第一行(改行对应着源代码)。这里即是问题发生的地方，也是寻找问题答案的最佳起始点。
+ `Android Lint`基于对Android框架知识的掌握，能够更深入地检查代码，找出编译器无法发现的问题。

#### 5.第二个activity

+ 创建一个activity 一般至少涉及到三个文件：`Java class`文件、`XML layout`文件和应用的`AndroidManifest.xml`文件。
+ 一个activity启动另一个activity的最简单方式是：

	```java
	public void startActivity(Intent intent);
	```

+ 在启动activity 以前，`ActivityManager`会检查确认指定的Class 是否已在manifest 配置文件中声明。如已完成声明，则启动activity，应用正常运行。反之，则抛出`ActivityNotFoundException`。
+ 显式intent：通过指定Context 与Class 对象，然后调用intent 的构造方法来创建Intent，即为显式intent。
+ 当一个应用的activity 如需启动另一个应用的activity，可通过创建隐式intent 来处理。
+ extra 是一种`key-value`结构。将extra 数据信息添加给intent，我们需要调用`Intent.putExtra(...)`方法。例如：

	```Java
	public Intent putExtra(String name, boolean value);
	```

	`Intent.putExtra(...)`有两个参数。一个参数是固定为`String` 类型的`key`，另一个参数值可以是多种数据类型。从`extra`获取数据，使用如下方法：

	```java
	public boolean getBooleanExtra(String name, boolean defaultValue);
	```

	第一个参数是`extra` 的名字，即`key`。`getBooleanExtra(...)`方法的第二个参数是指定默认值，它在无法获得有效key值时使用。
+ `Activity.getIntent()`返回的Intent就是启动activity的intent. 也就是`startActivity(Intent)`中传入的Intent。
+ 若需要从子activity 获取返回信息时，可调用以下Activity 方法：

	```java
	public void startActivityForResult(Intent intent, int requestCode);
	```

	该方法的第一个参数同前述的intent。第二个参数是请求代码。请求代码是先发送给子activity，然后再返回给父activity 的用户定义整数值。当一个activity启动多个不同类型的子activity，且需要判断区分消息回馈方时，我们通常会用到该请求代码。
+ 实现子activity 发送返回信息给父activity，有以下两种方法可供调用：

	```java
    public final void setResult(int resultCode);
    public final void setResult(int resultCode, Intent data);
	```

	通常来说，参数resultCode可以是以下两个预定义常量中的任何一个：

	```java
    Activity.RESULT_OK；
    Activity.RESULT_CANCELED;
	```

	（如需自己定义结果代码，还可使用另一个常量：`RESULT_FIRST_USER`）
	+ 在父activity 需要依据子activity 的完成结果采取不同操作时，设置结果代码很有帮助。例如，假设子activity 有一个OK 按钮及一个Cancel 按钮，并且为每个按钮的单击动作分别设置了不同的结果代码。根据不同的结果代码，父activity会采取不同的操作。
	+ 子activity 可以不调用`setResult(...)` 方法。如不需要区分附加在intent 上的结果或其他信息，可让操作系统发送默认的结果代码。如果子activity 是以调用`startActivityForResult(...)`方法启动的，结果代码则总是会返回给父activity。在没有调用`setResult(...)`方法的情况下，如果用户单击了后退按钮，父activity则会收到`Activity.RESULT_CANCELED`的结果代码。
+ 用户单击`Show Answer`按钮时，`CheatActivity(子)` 调用`setResult(int, Intent)` 方法将结果代码以及`intent` 打包。然后，在用户单击后退键回到`QuizActivity(父)`时，`ActivityManager`调用父activity的以下方法：

	```java
	protected void onActivityResult(int requestCode, int resultCode, Intent data);
	```

	该方法的参数来自于`QuizActivity`的原始请求代码以及传入`SetResult(...)`方法的结果代码和intent。
+ GeoQuiz 应用内部的交互时序图：

	![]({{ IMAGE_PATH }}/GeoQuiz应用内部的交互时序图.jpg)

#### 6.Android SDK 版本与兼容

+ SDK最低版本，manifest是操作系统用来与应用交互的元数据。以最低版本设置值为标准，操作系统会拒绝将应用安装在系统版本低于标准的设备上。
+ SDK目标版本，目标版本的设定值可告知Android：应用是设计给哪个API级别去运行的。大多数情况下，目标版本即最新发布的Android版本。
+ SDK编译版本，该设置不会出现在manifest配置文件里。SDK最低版本和目标版本会通知给操作系统，而SDK编译版本是我们和编译器之间的私有信息。Android 的特色功能是通过SDK 中的类和方法展现的。在编译代码时，SDK编译版本或编译目标指定具体要使用的系统版本。编译目标的最佳选择为最新的API 级别。
+ 检查Android设备版本的条件语句：

	```java
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    	...
    }
	```

+ 使用注解向`Android Lint`声明版本信息

	```java
    @TargetApi(11)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_quiz);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            ActionBar actionBar = getActionBar();
            actionBar.setSubtitle("TFFTT");
        }
    }
	```

+ Android开发者文档: [http://developer.android.com/](http://developer.android.com/)。文档分为三大部分，即`设计`、`开发`和`发布`。设计部分的文档包括应用UI设计的模式和原则。开发部分包括SDK文档和培训资料。发布部分告知我们如何在GooglePlay商店上或通过开放发布模式准备并发布应用。
开发部分可细分为四大块内容：
	+ Android培训，初级和中级开发者的培训模块，包括可下载的示例代码；
	+ API使用指导，基于主题的应用组件、特色功能详述以及它们的最佳实践；
	+ 参考文档，SDK中类、方法、接口、属性常量等可搜索、交叉链接的参考文档；
	+ 开发工具，开发工具的描述及下载链接。
	无需联网也可查看文档。浏览下载SDK的文件系统，会发现有一个docs目录，该目录包含了全部的Android开发者文档内容。