---
layout: post
title: "Android四大组件"
category: "笔记"
tags: "Android"
keywords: ["Android","Service","生命周期","Content Provider", "BroadcastReceiver"]
description: "Android四大基本组件介绍与生命周期"
---
{% include JB/setup %}

### Android四大组件

Android 四大基本组件分别是`Activity`(活动)，`Service`(服务)，`Content Provider`(内容提供者)，`BroadcastReceiver`(广播接收器)。

#### Activity(活动)

应用程序中，一个activity通常就是一个单独的屏幕，它上面可以显示一些控件，监听并响应用户的事件。

`Activity`之间通过`Intent`进行通信。在`Intent`的描述结构中，有两个最重要的部分：动作(`ACTION`)和动作对应的数据。

典型的动作类型有：`MAIN`、`VIEW`、`PICK`、`EDIT`等。动作对应的数据则以`URI`形式进行表示。例如：要查看一个人的联系方式，需要创建一个动作类型为`VIEW`的intent，以及表示这个人的`URI`。

与`Activity`有关系的一个类叫做`IntentFilter`。相对于intent作为一个有效的做某事的请求而言，`IntentFilter`则用于描述一个activity(或者`IntentReceiver`)能够操作哪些类型的intent。一个activity如果要显示一个人的联系方式时，需要声明一个`IntentFilter`，这个`IntentFilter`要知道怎么去处理`VIEW`动作和表示一个人的`URI`。`IntentFilter`需要在`AndroidManifest.xml`文件中定义。通过解析各种intent，从可以一个屏幕导航到另一个屏幕。当向前导航时，activity 将会调用`startActivity(Intent myIntent)`方法。然后，系统会在所有已安装的应用程序中定义好的`IntentFilter` 中查找，找到最匹配`myIntent` 的`Intent` 对应的activity。新的activity 接收到`myIntent` 的通知后，开始运行。当`startActivity(...)` 方法被调用将触发解析myIntent 的动作，这个机制提供了两个好处：

+ 1.`Activities` 能够重复利用从其它组件中以Intent 的形式产生的一个请求；
+ 2.`Activities` 可以在任何时候被一个具有相同IntentFilter 的新的Activity 取代。

`AndroidManifest.xml`文件中含有如下过滤器的`Activity`组件为默认启动类，当程序启动时系统自动调用该`Activity`：

```xml
<intent-filter>
	<action android:name="android.intent.action.MAIN" />
	<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

---

#### Service(服务)

一个`Service` 是一段长生命周期的，没有用户界面的程序。可以用来开发诸如监控类的后台程序。

比较好的一个例子就是一个正在从播放列表中播放歌曲的媒体播放器。在一个媒体播放器的应用中，应该会有多个activity，让使用者可以选择歌曲并播放歌曲。然而，音乐重放(`playback`)这个功能并没有对应的activity，因为使用者当然会认为在导航到其它屏幕时音乐应该还在播放的。在这个例子中，媒体播放器这个activity 会使用`Context.startService()`来启动一个service，从而可以在后台保持音乐的播放。同时，系统也将保持这个service 一直执行，直到这个service 运行结束。另外，还可以通过使用`Context.bindService()`方法，连接到一个service 上（如果这个service 还没有运行将启动它）。当连接到一个service 之后，还可以调用service 提供的接口与它进行通讯。拿媒体播放器这个例子来说，还可以进行暂停、重播等操作。

**`Service`使用步骤如下：**

+ 继承`Service`类；
+ 在`AndroidManifast.xml`配置清单文件中的`<application>`节点里对服务进行配置：

    ```xml
    <service name=".SMSService"/>
    ```

服务不能自己运行,需要通过`Contex.startService()`或`Contex.bindService()`来启动服务：

+ 通过`startService()`方法启动的服务于调用者(activity)没有关系，即使调用者关闭了，服务仍然在运行。如果想要停止服务，要调用`Context.stopService()`，此时系统会调用`onDestory()`。使用此方法启动时，服务首次启动系统依次调用服务的`onCreate()-->onStart()`，如果服务已经启动再次调用的话只会触发`onStart()`方法；

+ 使用`bindService()`启动的服务与调用者绑定，只要调用者关闭，服务就终止。使用此方法启动时，服务首次启动系统依次调用服务的`onCreate()-->onBind()`，如果服务已经启动再次调用不会再触发这两个方法。调用者退出时系统会依次调用服务的`onUnbind()-->onDestory()`。如果想要主动解除绑定可使用`Contex.unbindService()`，系统依次调用`onUnbind()-->onDestory()`。

---

#### Content Provider(内容提供者)

`Android`平台提供了`Content Provider`类，使一个应用程序的指定数据集可以提供给其他应用程序。这些数据可以存储在文件系统中、在一个SQLite数据库中、或任何其他合理的方式中。

其他应用可以通过`ContentResolver`类从该内容提供者中获取或存入数据。

只有需要在多个应用程序间共享数据时才需要内容提供者。例如，通讯录数据被多个应用程序使用，且必须存储在一个内容提供者中。

内容提供者的好处就是可以统一数据访问方式。

`Android`系统自带的内容提供者(顶级的表示数据库名，非顶级的都是表名)，在SDK文档的 [android.provider Java](http://developer.android.com/reference/android/provider/package-summary.html) 包中都有介绍。

        ├────Browser
        ├────CallLog
        ├────Contacts
        │		├────Groups
        │		├────People
        │		├────Phones
        │		└────Photos
        ├────Images
        │		└────Thumbnails
        ├────MediaStore
        │		├────Albums
        │		├────Artists
        │		├────Audio
        │		├────Genres
        │		└────Playlists
        ├────Settings
        └────Video

+ `CallLog`：地址和接收到的电话信息

+ `Contact.People.Phones`：存储电话号码

+ `Setting.System`：系统设置和偏好设置

**使用`Content Provider`对外共享数据的步骤：**

+ 1).继承`ContentProvider`类并根据需求重写以下方法:

	```java
    public boolean onCreate();//处理初始化操作

	/**
	* 插入数据到内容提供者(允许其他应用向你的应用中插入数据时重写)
	* @param uri
	* @param initialValues 插入的数据
	* @return
	*/
	public Uri insert(Uri uri, ContentValues initialValues);

	/**
	* 从内容提供者中删除数据(允许其他应用删除你应用的数据时重写)
	* @param uri
	* @param selection 条件语句
	* @param selectionArgs 参数
	* @return
	*/
	public int delete(Uri uri, String selection, String[] selectionArgs);

	/**
	* 更新内容提供者已存在的数据(允许其他应用更新你应用的数据时重写)
	* @param uri
	* @param values 更新的数据
	* @param selection 条件语句
	* @param selectionArgs 参数
	* @return
	*/
	public int update(Uri uri, ContentValues values, String selection,
			String[] selectionArgs);

	/**
	* 返回数据给调用者(允许其他应用从你的应用中获取数据时重写)
	* @param uri
	* @param projection 列名
	* @param selection 条件语句
	* @param selectionArgs 参数
	* @param sortOrder 排序
	* @return
	*/
	public Cursor query(Uri uri, String[] projection, String selection,
                     String[] selectionArgs, String sortOrder) ;

	/**
	* 用于返回当前Uri所代表数据的MIME类型
	* 如果操作的数据为集合类型(多条数据),那么返回的类型字符串应该为vnd.android.cursor.dir/开头
	* 例如要得到所有person记录的Uri为content://com.tendoasan.provider.personprovider/person,
	* 那么返回的MIME类型字符串应该为"vnd.android.cursor.dir/person"
	* 如果操作的数据为单一数据,那么返回的类型字符串应该为vnd.android.cursor.item/开头
	* 例如要得到id为10的person记录的Uri为content://com.tendoasan.provider.personprovider/person/10,
	* 那么返回的MIME类型字符串应该为"vnd.android.cursor.item/person"
	* @param uri
	*/
	public String getType(Uri uri)
	```

	这些方法中的`Uri`参数,得到后需要进行解析，然后做相应处理。`Uri`表示要操作的数据，包含两部分信息:
	+ 1.需要操作的`ContentProvider`；
	+ 2.对`ContentProvider`中的什么数据进行操作。
	一个`Uri`格式：`结构头://authorities(域名)/路径`(要操作的数据，根据业务而定)

        ```uri
        content://com.tendoasan.provider.personprovider/person/10
        ```

	`ContentProvider`的结构头已经由`Android`规定为`content://`；
	`authorities`用于唯一标识这个`Content Provider`程序，外部调用者可以根据这个找到它；
	`路径`表示具体要操作的数据,路径的构建根据业务而定。路径格式如下：
		+ 要操作`person`表行号为`10`的记录,可以这样构建`/person/10`
		+ 要操作`person`表的所有记录,可以这样构建`/person`

+ 2).在`AndroidManifest.xml`中使用`<provider>`对`ContentProvider`进行配置注册。`ContentProvider`采用`authoritie`作为唯一标识,方便其他应用能找到。

    ```xml
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <!-- authorities属性命名建议:公司名.provider.SomeProvider-->
        <provider android:name=".PersonProvider"
            android:authorities="com.tendoasan.provider.personprovider"/>
            ...
    </application>
    ```

---

#### BroadcastReceiver(广播接收器)

应用可以使用`BroadcastReceive`对外部事件进行过滤，只对感兴趣的外部事件(如当电话呼入时，或者数据网络可用时)进行接收并做出响应。广播接收器没有用户界面，但是它们可以通过启动一个`Activity`或`Service` 来响应它们收到的信息，或者用`NotificationManager` 来通知用户。通知可以用多种方式来吸引用户的注意力──闪动背灯、震动、播放声音等。一般来说是在状态栏上放一个持久的图标，用户可以打开它并获取消息。

广播类型：

+ 普通广播，通过`Context.sendBroadcast(Intent myIntent)`发送的广播；

+ 有序广播，通过`Context.sendOrderedBroadcast(intent, receiverPermission)`发送的广播。该方法第二个参数`receiverPermission`决定该广播的级别。级别数值是在 -1000 到 1000 之间，值越大，则发送的优先级越高；广播接收者接收广播时的级别可通过`intentfilter`中的`priority`进行设置，设为`2147483647`时优先级最高。同级别广播接收者接收广播的先后是随机的，之后再到级别低的接收者收到广播。高级别或同级别的广播接收者先接收到广播后，可以通过`abortBroadcast()`方法来截断广播，得使其他的接收者无法收到该广播；

+ 异步广播，通过`Context.sendStickyBroadcast(Intent myIntent)`发送的广播。还有`sendStickyOrderedBroadcast(intent, resultReceiver, scheduler,  initialCode, initialData, initialExtras)`方法，该方法同时具有有序广播和异步广播的特性。发送异步广播需要权限：

    ```xml
    <uses-permission android:name="android.permission.BROADCAST_STICKY" />
    ```

    接收并处理完`Intent`后，广播依然存在，直到调用`removeStickyBroadcast(intent)`主动把它去掉。

+ 需要注意的是：发送广播时的`intent`参数与`Context.startActivity()`启动起来的`Intent`不同，前者可以被多个订阅它的广播接收器调用，而后者只能被一个`Activity`或`Service`调用。

**监听广播`Intent`(定义广播接收器)步骤**：

+ 1.继承`BroadCastReceiver`类，重写`onReceive()`方法，广播接收器仅在执行这个方法时处于活跃状态。当`onReceive()`返回后，广播接收器即为失活状态。注意：为了保证用户交互过程的流畅,一些费时的操作要放到线程里，如`SMSBroadcastReceiver`类；

+ 2.注册该广播接收者。注册方法分为：`AndroidManifest.xml`文件静态注册和程序动态注册。

	+ 静态注册,，下面的priority表示接收广播的级别`2147483647`为最高优先级：

        ```xml
        <receiver android:name=".SMSBroadcastReceiver" >
          <intent-filter android:priority = "2147483647" >
            <action android:name="android.provider.Telephony.SMS_RECEIVED" />
          </intent-filter>
        </receiver >
        ```

	+ 动态注册，一般在`Activity`可交互时，在`onResume()`内注册`BroadcastReceiver`：

        ```java
        IntentFilter intentFilter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
        registerReceiver(mBatteryInfoReceiver ,intentFilter);

        //反注册
        unregisterReceiver(receiver);
        ```

		需要注意的是：
		+ 广播接收器生命周期只有十秒左右，如果在`onReceive()`内做超过十秒内的事情，就会报`ANR(Application No Response)`程序无响应的错误信息。如果需要完成一项比较耗时的工作，应该通过发送`Intent`给`Service`，由`Service`来完成。这里不能使用子线程来解决，因为 `BroadcastReceiver`的生命周期很短，子线程可能还没有结束`BroadcastReceiver` 就先结束了。`BroadcastReceiver`一旦结束，此时`BroadcastReceiver`的所在进程很容易在系统需要内存时被优先杀死，因为它属于空进程(没有任何活动组件的进程)。如果`BroadcastReceiver`的宿主进程被杀死， 那么正在工作的子线程也会被杀死，所以采用子线程来解决是不可靠的。
		+ 动态注册广播接收器还有一个特点，就是当用来注册的`Activity`关掉后，广播也就失效了。静态注册则无需担心广播接收器是否被关闭，因为只要设备是开启状态，那么广播接收器就是打开着的。也就是说哪怕应用本身未启动，该应用订阅的广播在触发时也会对它起作用。

系统常见广播`Intent`，如开机启动、电池电量变化、时间改变等广播。

---

#### 四大基本组件小结：

##### 注册

四大基本组件都需要注册才能使用。每个`Activity`、`Service`、`Content Provider`都需要在`AndroidManifest.xml`文件中进行配置。`AndroidManifest.xml`文件中未进行声明的活动、服务以及内容提供者在系统中为不可见状态，从而也就不可用。`BroadcastReceiver`的注册分为静态注册和动态注册。需要注意的是：在`AndroidManifest.xml`文件中进行配置的广播接收器会随系统的启动而一直处于活跃状态，只要接收到感兴趣的广播就会触发（即使应用没有运行）。

在`AndroidManifest.xml`文件中进行注册的格式如下：

+ `<activity>`元素的`name`属性指定了实现了该`activity`的`Activity`的子类。`icon` 和`label`属性指向了包含展示给用户的此`activity`的图标和标签的资源文件。

+ `<service>`元素用于声明服务。

+ `<receiver>`元素用于声明广播接收器。

+ `<provider>`元素用于声明内容提供者。

##### 激活

+ 活动的激活：通过传递一个`Intent`对象至`Context.startActivity()`或`Activity.startActivityForResult()`以启动（或指定新工作给）一个`activity`。相应的`activity`可以通过调用`getIntent()`方法来查看激活它的`intent`。如果`activity`期望它所启动的那个`activity`返回一个结果，它会调用`startActivityForResult()`来启动另一个`activity`。比如说，如果它启动了另一个`activity` 以使用户挑选一张照片，它需要知道哪张照片被选中了。另一个`activity`应该将结果封装在一个`Intent`对象中，并传递给启动它的`activity`的`onActivityResult()`方法中。

+ 服务的激活：通过传递一个`Intent`对象至`Context.startService()`或`Context.bindService()`，前者`Android`调用服务的`onStart()`方法并将`Intent对象传递给它，后者`Android`调用服务的`onBind()`方法将这个`Intent`对象传递给它。

+ 内容提供者的激活：当接收到`ContentResolver` 发出的请求后，内容提供者被激活。而其它三种组件──活动、服务和广播接收器被一种叫做`Intent`的异步消息所激活。

+ 发送广播：可以通过传递一个`Intent`对象至`Context.sendBroadcast()`、`Context.sendOrderedBroadcast()`或`Context.sendStickyBroadcast()`。`Android`会调用所有对此广播有兴趣的广播接收器的`onReceive()`方法，将`Intent`传递给它们。

##### 关闭

+ 活动关闭：可以通过调用它的`finish()`方法来关闭它。

+ 服务关闭：对于通过`startService()`方法启动的服务要调用`Context.stopService()`方法关闭它；对于使用`bindService()`方法启动的服务要调用`Contex.unbindService ()`方法关闭它。

+ 内容提供者仅在响应`ContentResolver`提出请求的时候激活。而一个广播接收器仅在响应广播信息的时候激活。所以，没有必要去显式地关闭这些组件。

---

### 四大基本组件的生命周期

介绍生命周期之前，先提一下任务(`Task`)的概念。

任务其实就是`Activity`的栈，它由一个或多个`Activity`组成，共同完成一个完整的用户体验，换句话说任务就是`应用程序`。栈底的是启动整个任务的`Activity`，栈顶的是当前运行的用户可以交互的`Activity`，当一个`activity`启动另外一个的时候，新的`activity`就被压入栈，并成为当前运行的`activity`。而前一个`activity`仍保持在栈之中。当用户按下`BACK`键的时候，当前`activity`出栈，而前一个`activity`恢复为当前运行的`activity`。栈中保存的其实是对象，栈中的`Activity`永远不会重排，只会压入或弹出，所以如果发生了诸如需要多个地图浏览器的情况，就会使得一个任务中出现多个同一`Activity`子类的实例同时存在。

任务中的所有`activity`是作为一个整体进行移动的。整个的任务（即`activity`栈）可以移到前台，或退至后台。举个例子说，比如当前任务在栈中存有四个`activity`──三个在当前`activity`之下。当用户按下`HOME`键的时候，回到了应用程序加载器，然后选择了一个新的应用程序（也就是一个新任务）。则当前任务遁入后台，而新任务的根`activity`显示出来。然后，过了一会儿，用户再次回到了应用程序加载器而又选择了前一个应用程序（上一个任务）。于是那个任务，带着它栈中所有的四个`activity`，再一次的到了前台。当用户按下`BACK`键的时候，屏幕不会显示出用户刚才离开的`activity`（上一个任务的根`activity`）。取而代之的是，当前任务的栈中最上面的`activity`被弹出，而同一任务中的上一个`activity`显示了出来。

`Activity`栈：先进后出规则

`Android`系统是一个多任务(`Multi-Task`)的操作系统，可以在用手机听音乐的同时，也执行其他多个程序。每多执行一个应用程序，就会多耗费一些系统内存，当同时执行的程序过多，或是关闭的程序没有正确释放掉内存，系统就会觉得越来越慢，甚至不稳定。

为了解决这个问题，`Android`引入了一个新的机制--生命周期(`Life Cycle`)。

`Android`应用程序的生命周期是由`Android`框架进行管理，而不是由应用程序直接控制的。通常，每一个应用程序（入口一般会是一个`Activity`的`onCreate()`方法），都会产生一个进程(`Process`)。当系统内存即将不足的时候，会依照优先级自动进行进程(`process`)的回收。不管是使用者或开发者，都无法确定应用程序何时会被系统回收，所以为了很好地防止数据丢失和其他问题，了解生命周期很重要。

#### 活动的生命周期

活动的生命周期图：

![]({{ IMAGE_PATH }}/活动的生命周期图.jpg)

`Activity`整个生命周期拥有4种状态，7个重要方法和3个嵌套循环。

**四种状态：**

+ 活动(`Active/Running`)状态。当`Activity`运行在屏幕前台(处于当前任务活动栈的最上面)，此时它获取了焦点能响应用户的操作，属于运行状态，同一个时刻只会有一个`Activity`处于活动(`Active`)或运行(`Running`)状态；

+ 暂停(`Paused`)状态。当`Activity`失去焦点但仍对用户可见(如在它之上有另一个透明的`Activity`或`Toast`、`AlertDialog`等弹出窗口时)它处于暂停状态。暂停的`Activity`仍然是存活状态(它保留着所有的状态和成员信息并保持和窗口管理器的连接)，但是当系统内存极小时仍可以被系统杀掉；

+ 停止(`Stopped`)状态。完全被另一个`Activity`遮挡时处于停止状态，它仍然保留着所有的状态和成员信息，只是对用户不可见，当其他地方需要内存时它往往被系统优先杀掉；

+ 非活动(`Dead`)状态。`Activity`尚未被启动或已经被手动终止，或已经被系统回收时处于非活动的状态。要手动终止`Activity`，可以在程序中调用`finish()`方法。如果是（按根据内存不足时的回收规则）被系统回收，可能是因为内存不足了。内存不足时，`Dalvak`虚拟机会根据其内存回收规则来回收内存：
	+ 先回收与其他`Activity`或`Service/Intent Receiver`无关的进程(即优先回收独立的`Activity`)。因此建议，一些耗时的后台操作，最好通过`Service`的形式来完成。
	+ 不可见(处于`Stopped`状态的)`Activity`。
	+ `Service`进程(除非真的没有内存可用时会被销毁)。
	+ 非活动的可见的(`Paused`状态的)`Activity`。
	+ 当前正在运行（`Active`/`Running`状态的）`Activity`。

**七个重要方法：**

当`Activity`从一种状态进入另一状态时系统会自动调用下面相应的方法来通知用户这种变化。

+ 当`Activity`第一次被实例化的时，系统会调用：

    ```java
	onCreate(Bundle savedInstanceState);
    ```

	整个生命周期只调用一次这个方法。通常用于初始化设置:
	+ 为`Activity`设置所要使用的布局文件；
	+ 为按钮绑定监听器等静态的设置操作。

+ 当`Activity`可见，但是没有获得用户焦点不能交互时，系统会调用：

	```java
	onStart();
	```

+ 当`Activity`已经停止，然后重新被启动时，系统会调用：

	```java
	onRestart();
	```

+ 当`Activity`可见，且获得用户焦点能交互时，系统会调用：

	```java
	onResume();
	```

+ 当系统设置发生改变时(按下`HOME`键，按下电源按键关闭屏幕，横竖屏切换等情况)，需要记录当前`activity`的临时状态时，重写方法：

    ```java
    onSaveInstanceState(Bundle outState)
    ```

	当该`activity`再次被实例化时会通过`onCreate(Bundle savedInstanceState)`将已经保存的临时状态数据传入。

+ 存储持久数据需要使用`onPause()`方法：

    ```java
    onPause();
    ```

+ 当`Activity`被新的A`ctivity`完全覆盖不可见时被系统调用

	```java
	onStop();
	```

+ 当`Activity`(用户调用`finish()`或系统由于内存不足)被系统销毁杀掉时系统调用：

	```java
	onDestroy();
	```

	整个生命周期只调用一次，用来释放`onCreate()`方法中创建的资源，如结束线程等。

**三个嵌套循环：**

+ `Activity`完整的生命周期：从第一次调用`onCreate()`开始直到调用`onDestroy()`结束；

+ `Activity`的可视生命周期：从调用`onStart()`到相应地调用`onStop()`。在这两个方法之间，可以保持显示`Activity`所需要的资源。如在`onStart()`中注册一个广播接收器监听影响`UI`的改变，然后在`onStop()`中注销；

+ `Activity`的前台生命周期：从调用`onResume()`到相应地调用`onPause()`。

举例说明：

例1：有3个`Acitivity`，分别用`One`(应用启动时的主`Activity`)，`Two`(透明的)和`Three`(全屏的)表示。

+ 启动第一个界面`Activity One`时，对应周期方法的调用次序是：
`onCreate (ONE) ---> onStart (ONE) ---> onResume(ONE)`

+ 点击`打开透明Activity Two`按钮时，对应周期方法的调用次序是：
`onPause(ONE) ---> onCreate(TWO) ---> onStart(TWO) ---> onResume(TWO)`

+ 再点击`Back`键回到第一个界面，`Two`会被销毁，对应周期方法的调用次序是：
`onPause(TWO) ---> onActivityResult(ONE) ---> onResume(ONE) ---> onStop(TWO) ---> onDestroy(TWO)`

+ 点击`打开全屏Activity`按钮时，对应周期方法的调用次序是：
`onPause(ONE) ---> onCreate(Three) ---> onStart(Three) ---> onResume(Three) ---> onStop(ONE)`

+ 再点击`Back`键回到第一个界面，`Three`会被销毁，对应周期方法的调用次序是：
`onPause(Three) ---> onActivityResult(ONE) ---> onRestart(ONE) ---> onStart(ONE) ---> onResume(ONE) ---> onStop(Three) ---> onDestroy(Three)`

+ 再点击`Back`键退出应用时，对应周期方法的调用次序是：
`onPause(ONE) ---> onStop(ONE) ---> onDestroy(ONE)`

例2：横竖屏切换时，`Activity`的生命周期。

+ 1、新建一个`Activity`，打印各个生命周期方法。

+ 2、运行`Activity`，得到如下信息：

        onCreate-->
        onStart-->
        onResume-->

+ 3、切换成横屏时：

        onSaveInstanceState-->
        onPause-->
        onStop-->
        onDestroy-->
        onCreate-->
        onStart-->
        onRestoreInstanceState-->
        onResume-->

+ 4、切换回竖屏时，发现打印了两次相同的日记：

        onSaveInstanceState-->
        onPause-->
        onStop-->
        onDestroy-->
        onCreate-->
        onStart-->
        onRestoreInstanceState-->
        onResume-->
        onSaveInstanceState-->
        onPause-->
        onStop-->
        onDestroy-->
        onCreate-->
        onStart-->
        onRestoreInstanceState-->
        onResume-->

+ 5、修改`AndroidManifest.xml`文件，为该`Activity`添加`android:configChanges="orientation"`，执行步骤3：

        onSaveInstanceState-->
        onPause-->
        onStop-->
        onDestroy-->
        onCreate-->
        onStart-->
        onRestoreInstanceState-->
        onResume-->

+ 6、再执行步骤4，发现不会再打印相同信息，但多打印了一行`onConfigChanged`

        onSaveInstanceState-->
        onPause-->
        onStop-->
        onDestroy-->
        onCreate-->
        onStart-->
        onRestoreInstanceState-->
        onResume-->
        onConfigurationChanged-->

+ 7、把步骤5的`android:configChanges="orientation"`改成 `android:configChanges="orientation|keyboardHidden"`，执行步骤3，就只打印了`onConfigChanged`：

		onConfigurationChanged-->

+ 8、执行步骤4：

        onConfigurationChanged-->
        onConfigurationChanged-->

总结：

+ 1、不设置`Activity`的`android:configChanges`时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次；

+ 2、设置`Activity`的`android:configChanges="orientation"`时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次；

+ 3、设置`Activity`的`android:configChanges="orientation|keyboardHidden"`时，切屏不会重新调用各个生命周期，只会执行`onConfigurationChanged`方法。

补充：

+ 当前`Activity`产生事件，弹出`Toast`和`AlertDialog`的时候，`Activity`的生命周期不会有改变；
+ `Activity`运行时按下`HOME`键(跟被完全覆盖是一样的)：`onSaveInstanceState --> onPause --> onStop`，再次进入激活状态时：`onRestart -->onStart--->onResume`。
+ `onCreate(Bundle savedInstanceState)`与`onSaveInstanceState(Bundle savedInstanceState)`配合使用，见如下代码，达到显示`activity`被系统杀死前的状态：

	```java
	@Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (null != savedInstanceState) {
            String _userid = savedInstanceState.getString("StrUserId");
            String _uid = savedInstanceState.getString("StrUid");
            String _serverid = savedInstanceState.getString("StrServerId");
            String _servername = savedInstanceState.getString("StrServerName");
            int _rate = savedInstanceState.getInt("StrRate");
            //updateUserId(_userid);
            //updateUId(_uid);
            //updateServerId(_serverid);
            //updateUserServer(_servername);
            //updateRate(_rate);
        }
    }

    @Override
    protected void onSaveInstanceState(Bundle savedInstanceState) {
        super.onSaveInstanceState(savedInstanceState);
        savedInstanceState.putString("StrUserId", getUserId());
        savedInstanceState.putString("StrUid", getUId());
        savedInstanceState.putString("StrServerId", getServerId());
        savedInstanceState.putString("StrServerName", getServerName());
        savedInstanceState.putInt("StrRate", getRate());
    }
	```

#### 服务的生命周期

服务的生命周期图：

![]({{ IMAGE_PATH }}/服务的生命周期图.jpg)

`Service`完整的生命周期：从调用`onCreate()`开始直到调用`onDestroy()`结束。

`Service`的两种使用方法：

+ 调用`Context.startService()`启动，调用`Context.stopService()`结束；

+ 调用`Context.bindService()`方法建立，调用`Context.unbindService()`关闭。

`Service`的重要生命周期方法：

+ 当用户调用`startService(`)或`bindService()`时，`Service`第一次被实例化的时，系统会调用：

    ```java
    void onCreate()
    ```

	整个生命周期只调用一次这个方法，通常用于初始化设置。注意：多次调用`startService()`或`bindService()`方法不会多次触发`onCreate()方法。

+ 当用户调用`stopService()`或`unbindService()`来停止服务时，被系统调用：

	```java
	void onDestroy()
	```

	整个生命周期只调用一次，用来释放`onCreate()`方法中创建的资源。

+ 通过`startService()`方法启动的服务：
初始化结束后系统会调用该方法，用于处理传递给`startService()`的`Intent`对象。如音乐服务会打开`Intent`来探明将要播放哪首音乐，并开始播放。

	```java
	void onStart(Intent intent)
	```

	多次调用`startService()`方法会多次触发`onStart()`方法。

+ 通过`bindService()`方法启动的服务：
初始化结束后系统会调用该方法，用来绑定传递给`bindService()`的`Intent` 对象。

	```java
	IBinder onBind(Intent intent)
	```

	多次调用`bindService()`时，如果该服务已启动则不会再触发此方法。

+ 用户调用`unbindService()`时系统调用此方法，`Intent`对象同样传递给该方法：

	```java
	boolean onUnbind(Intent intent)
	```

+ 如果有新的客户端连接至该服务，只有当旧的调用`onUnbind()`后，新的才会调用该方法：

	```java
	void onRebind(Intent intent)
	```

#### 广播接收器的生命周期

广播接收器的生命周期只有十秒左右，如果在`onReceive()`内做超过十秒内的事情，就会报`ANR(Application No Response)`程序无响应的错误信息。它的生命周期为从回调`onReceive()`方法开始到该方法返回结果后结束。

---

参考文章：[http://www.cnblogs.com/bravestarrhu/archive/2012/05/02/2479461.html](http://www.cnblogs.com/bravestarrhu/archive/2012/05/02/2479461.html)








































































