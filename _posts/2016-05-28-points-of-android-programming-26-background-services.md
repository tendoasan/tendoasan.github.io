---
layout: post
title: "第26章 后台服务"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Service","PhotoGallery"]
description: "后台服务"
---
{% include JB/setup %}

#### 26.后台服务

+ 创建IntentService。
	+ 使用`IntentService`创建服务(`Service`)。IntentService并不是Android提供的唯一服务，但却可能是最常用的。创建一个名为`PollService`的IntentService子类，作为用于查询搜索结果的服务。(`PollService.java`)

		```java
        public class PollService extends IntentService {
            private static final String TAG = "PollService";

            public static Intent newIntent(Context context) {
                return new Intent(context, PollService.class);
            }

            public PollService() {
                super(TAG);
            }

            @Override
            protected void onHandleIntent(Intent intent) {
                Log.i(TAG, "Received an intent: " + intent);
            }
        }
		```

        + `IntentService`也是一个`context`（Service是Context的子类），并能够响应`intent`。
        + 服务的`intent`又称作命令（`command`）。每一条命令都要求服务完成某项具体的任务。根据服务的种类不同，服务执行命令的方式也不尽相同。IntentService执行命令的方式：

        	![]({{ IMAGE_PATH }}/IntentService执行命令的方式.jpg)

        + IntentService逐个执行命令队列里的命令。接收到首个命令时，IntentService即完成启动，并触发一个后台线程，然后将命令放入队列。
        + 随后，IntentService继续按顺序执行每一条命令，并同时为每一条命令在后台线程上调用`onHandleIntent(Intent)`方法。新进命令总是放置在队列尾部。最后，执行完队列中全部命令后，服务也随即停止并被销毁。
        + 以上描述仅适用于IntentService。
	+ 在`manifest`配置文件中添加服务并添加网络获取权限。（`AndroidManifest.xml`）

		```xml
        <manifest xmlns:android="http://schemas.android.com/apk/res/android"						package="com.bignerdranch.android.photogallery" >
        	...
            <uses-permission android:name="android.permission.INTERNET" />
            <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
            <application
                ... >
                <activity
                    android:name=".PhotoGalleryActivity"
                    android:label="@string/app_name" >
                    ...
                </activity>
                <service android:name=".PollService" />
            </application>
        </manifest>
		```

	+ 在`PhotoGalleryFragment`中启动服务。

		```java
        public class PhotoGalleryFragment extends Fragment {
            private static final String TAG = "PhotoGalleryFragment";
            ...
            @Override
            public void onCreate(Bundle savedInstanceState) {
                ...
                updateItems();

                Intent i = PollService.newIntent(getActivity());
                getActivity().startService(i);

                Handler responseHandler = new Handler();
                mThumbnailDownloader = new ThumbnailDownloader<>(responseHandler);
                ...
            }
            ...
        }
		```

+ 服务的作用。
	+ 服务就是Android应用的后台。用户无需关心后台发生的一切。即使前台关闭，activity长时间停止运行，后台服务依然可以持续不断地执行工作任务。
	+ 服务可以完成，但activity却做不到的事情：在用户离开当前应用去别处时，服务依然可以在后台运行。
+ 后台网络连接的安全。
	+ 服务将在后台查询`Flickr`网站。为保证后台网络连接的安全性，需进一步完善代码。Android为用户提供了关闭后台应用网络连接的功能。对于非常耗电的应用而言，这项功能可极大地改善手机的续航能力。
	+ 所以在后台连接网络时，需使用`ConnectivityManager`确认网络连接是否可用。(`PollService.java`)

		```java
        public class PollService extends IntentService {
            private static final String TAG = "PollService";
            ...
            @Override
            protected void onHandleIntent(Intent intent) {
                if (!isNetworkAvailableAndConnected()) {
                    return;
                }

                Log.i(TAG, "Received an intent: " + intent);
            }

            private boolean isNetworkAvailableAndConnected() {
                ConnectivityManager cm =
                        (ConnectivityManager) getSystemService(CONNECTIVITY_SERVICE);

                boolean isNetworkAvailable = cm.getActiveNetworkInfo() != null;
                boolean isNetworkConnected = isNetworkAvailable &&
                        cm.getActiveNetworkInfo().isConnected();

                return isNetworkConnected;
            }
        }
		```

		+ 调用`ConnectivityManager.getActiveNetworkInfo()`返回为`null`则表示后台服务没有可用网络，调用该方法需要声明网络连接权限。
		+ 如果后台服务有可用网络，则上述方法将返回一个`android.net.NetworkInfo`的实例，表示目前的网络连接状况。

+ 查找最新返回结果。
	+ 后台服务会一直查看最新的返回结果，因此它需知道最近一次获取的结果(最近获取图片ID)。使用SharedPreferences保存结果值。(`QueryPreferences.java`)

		```java
        public class QueryPreferences {

            private static final String PREF_SEARCH_QUERY = "searchQuery";
            private static final String PREF_LAST_RESULT_ID = "lastResultId";

            public static String getStoredQuery(Context context) {
                ...
            }

            public static void setStoredQuery(Context context, String query) {
                ...
            }

            public static String getLastResultId(Context context) {
                return PreferenceManager.getDefaultSharedPreferences(context)
                        .getString(PREF_LAST_RESULT_ID, null);
            }

            public static void setLastResultId(Context context, String lastResultId) {
                PreferenceManager.getDefaultSharedPreferences(context)
                        .edit()
                        .putString(PREF_LAST_RESULT_ID, lastResultId)
                        .apply();
            }
        }
		```

	+ 更新服务代码，执行以下任务：
		+ 从默认SharedPreferences中获取当前查询结果以及上一次结果ID；
		+ 使用FlickrFetchr类获取最新结果集；
		+ 如果有结果返回，抓取结果的第一条；
		+ 检查确认是否不同于上一次结果ID；
		+ 将第一条结果保存回SharedPreferences。

			```java
			public class PollService extends IntentService {
				private static final String TAG = "PollService";
				...
				@Override
				protected void onHandleIntent(Intent intent) {
					...
					Log.i(TAG, "Received an intent: " + intent);
					String query = QueryPreferences.getStoredQuery(this);
					String lastResultId = QueryPreferences.getLastResultId(this);
					List<GalleryItem> items;

					if (query == null) {
						items = new FlickrFetchr().fetchRecentPhotos();
					} else {
						items = new FlickrFetchr().searchPhotos(query);
					}

					if (items.size() == 0) {
						return;
					}

					String resultId = items.get(0).getId();
					if (resultId.equals(lastResultId)) {
						Log.i(TAG, "Got an old result: " + resultId);
					} else {
						Log.i(TAG, "Got a new result: " + resultId);
					}
					QueryPreferences.setLastResultId(this, resultId);
				}
				...
			}
			```

+ 使用 `AlarmManager` 延迟运行服务。
	+ 为保证服务在后台的切实可用，当没有activity在运行时，需通过某种方式在后台执行一些任务。比如说，设置一个5分钟间隔的定时器。`AlarmManager`是可以发送Intent的系统服务。
	+ 使用`PendingIntent`告诉`AlarmManager`发送什么样的intent。可以使用PendingIntent打包intent：“我想启动PollService服务。”然后，将其发送给系统中的其他部件，如`AlarmManager`。
	+ 在`PollService`中，实现一个启停定时器的`setServiceAlarm(Context,boolean)`方法该方法是一个静态方法。这样，可使定时器代码和与之相关的代码都放置在PollService类中，但同时又允许其他系统部件调用它。通常会从前端的fragment或其他控制层代码中启停定时器。(`PollService.java`)

		```java
        public class PollService extends IntentService {
            private static final String TAG = "PollService";

            private static final int POLL_INTERVAL = 1000 * 60; // 60 seconds

            public static Intent newIntent(Context context) {
                return new Intent(context, PollService.class);
            }

            public static void setServiceAlarm(Context context, boolean isOn) {
                Intent i = PollService.newIntent(context);
                PendingIntent pi = PendingIntent.getService(context, 0, i, 0);

                AlarmManager alarmManager = (AlarmManager)
                        context.getSystemService(Context.ALARM_SERVICE);

                if (isOn) {
                    alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME,
                            SystemClock.elapsedRealtime(), POLL_INTERVAL, pi);
                } else {
                    alarmManager.cancel(pi);
                    pi.cancel();
                }
            }
            ...
        }
		```

		+ 首先通过调用`PendingIntent.getService(...)`方法，创建一个用来启动PollService的`PendingIntent`。`PendingIntent.getService(...)`方法打包了一个`Context.startService(Intent)`方法的调用。它有四个参数：一个用来发送intent的Context、一个区分PendingIntent来源的请求代码，待发送的Intent对象以及一组用来决定如何创建PendingIntent的标志符。
		+ 接下来，需要设置或取消定时器。设置定时器可调用`AlarmManager.setRepeating(...)`方法。该方法同样具有四个参数：一个描述定时器时间基准的常量、定时器运行的开始时间、定时器循环的时间间隔以及一个到时要发送的PendingIntent。
		+ 取消定时器可调用`AlarmManager.cancel(PendingIntent)`方法。通常，也需同步取消PendingIntent。取消PendingIntent的操作有助于跟踪定时器状态的。
	+ 启动定时器。(`PhotoGalleryFragment`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            private static final String TAG = "PhotoGalleryFragment";
            ...
            @Override
            public void onCreate(Bundle savedInstanceState) {
                ...
                updateItems();

                PollService.setServiceAlarm(getActivity(), true);

                Handler responseHandler = new Handler();
                mThumbnailDownloader = new ThumbnailDownloader<>(responseHandler);
                ...
            }
            ...
        }
		```

+ 使用`PendingIntent`管理定时器。
	+ `PendingIntent`是一种`token`对象。调用`PendingIntent.getService(...)`方法获取PendingIntent时，我们告诉操作系统：“请记住，我需要使用startService(Intent)方法发送这个intent。”随后，调用PendingIntent对象的`send()`方法时，操作系统会按照我们的要求发送原来封装的intent。
	+ PendingIntent真正精妙的地方在于，将PendingIntent token交给其他应用使用时，它是代表当前应用发送token对象的。另外，PendingIntent本身存在于操作系统而不是token里，因此实际上是我们在控制着它。如果不顾及别人感受的话，也可以在交给别人一个PendingIntent对象后，立即撤销它，让send()方法啥也做不了。
	+ 如果使用同一个intent请求PendingIntent两次，得到的PendingIntent仍会是同一个。我们可借此测试某个PendingIntent是否已存在，或撤销已发出的PendingIntent。
	+ 一个`PendingIntent`只能登记一个定时器。这也是`isOn`值为false时，`setServiceAlarm(Context,boolean)`方法的工作原理：首先调用`AlarmManager.cancel(PendingIntent)`方法撤销`PendingIntent`的定时器，然后撤销PendingIntent。
	+ 既然撤销定时器也随即撤消了PendingIntent，可通过检查PendingIntent是否存在，确认定时器激活与否。具体代码实现时，传入`PendingIntent.FLAG_NO_CREATE`标志给`PendingIntent.getService(...)`方法即可。该标志表示如果PendingIntent不存在，则返回null值，而不是创建它。
	+ 添加一个名为`isServiceAlarmOn(Context)`的方法，并传入`PendingIntent.FLAG_NO_CREATE`标志，以判断定时器的启停状态。(`PollService.java`)

		```java
        public class PollService extends IntentService {
            ...
            public static void setServiceAlarm(Context context, boolean isOn) {
                ...
            }

            public static boolean isServiceAlarmOn(Context context) {
                Intent i = PollService.newIntent(context);
                PendingIntent pi = PendingIntent
                        .getService(context, 0, i, PendingIntent.FLAG_NO_CREATE);
                return pi != null;
            }
            ...
        }
		```

		+ 这里的PendingIntent仅用于设置定时器，因此PendingIntent空值表示定时器还未设置。

+ 控制定时器。
	+ 添加服务开关（`menu/fragment_photo_gallery.xml`）

		```xml
		<menu xmlns:android="http://schemas.android.com/apk/res/android"
			  xmlns:app="http://schemas.android.com/apk/res-auto">
			<item android:id="@+id/menu_item_search"
				  ... />
			<item android:id="@+id/menu_item_clear"
				  ... />

			<item android:id="@+id/menu_item_toggle_polling"
				  android:title="@string/start_polling"
				  app:showAsAction="ifRoom" />
		</menu>
		```

	+ 添加polling字符串资源（`res/values/strings.xml`）

		```xml
		<resources>
			...
			<string name="search">Search</string>
			<string name="clear_search">Clear Search</string>
			<string name="start_polling">Start polling</string>
			<string name="stop_polling">Stop polling</string>
			<string name="new_pictures_title">New PhotoGallery Pictures</string>
			<string name="new_pictures_text">You have new pictures in PhotoGallery.</string>
		</resources>
		```

	+ 代码实现。(`PhotoGalleryFragment.java`)

		```java
        private static final String TAG = "PhotoGalleryFragment";
        ...
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            ...
            updateItems();
            Handler responseHandler = new Handler();
            ...
        }
        ...
        @Override
        public boolean onOptionsItemSelected(MenuItem item) {
            switch (item.getItemId()) {
                case R.id.menu_item_clear:
                    QueryPreferences.setStoredQuery(getActivity(), null);
                    updateItems();
                    return true;
                case R.id.menu_item_toggle_polling:
                    boolean shouldStartAlarm = !PollService.isServiceAlarmOn(getActivity());
                    // 按下开关后，改变定时器的开关设置
                    PollService.setServiceAlarm(getActivity(), shouldStartAlarm);
                    // 定时器开关设置改变后，告诉OS重新绘制操作栏
                    getActivity().invalidateOptionsMenu();
                    return true;
                default:
                    return super.onOptionsItemSelected(item);
            }
        }
        ...
        @Override
        public void onCreateOptionsMenu(Menu menu, MenuInflater menuInflater) {
            ...
            MenuItem toggleItem = menu.findItem(R.id.menu_item_toggle_polling);
            // 根据定时器的开关来设置显示的菜单内容。
            if (PollService.isServiceAlarmOn(getActivity())) {
                toggleItem.setTitle(R.string.stop_polling);
            } else {
                toggleItem.setTitle(R.string.start_polling);
            }
        }
        ...
		```

+ 通知信息。
	+ 通知信息（`notification`）是指显示在通知抽屉上的消息条目，用户可向下滑动屏幕读取。
	+ 为发送通知信息，首先需创建一个`Notification`对象。Notification需使用构造对象完成创建，应至少具备：
		+ 首次显示通知信息时，在状态栏上显示的ticker text；
		+ ticker text消失后，在状态栏上显示的图标；
		+ 代表通知信息自身，在通知抽屉中显示的视图；
		+ 用户点击抽屉中的通知信息，触发PendingIntent。
	+ 完成`Notification`对象的创建后，可调用`NotificationManager`系统服务的`notify(int, Notification)`方法发送它。
	+ 添加代码代码实现让PollService通知新结果信息给用户。(`PollService.java`)

		```java
        ...
        @Override
        protected void onHandleIntent(Intent intent) {
            ...
            String resultId = items.get(0).getId();
            if (resultId.equals(lastResultId)) {
                Log.i(TAG, "Got an old result: " + resultId);
            } else {
                Log.i(TAG, "Got a new result: " + resultId);

                Resources resources = getResources();
                Intent i = PhotoGalleryActivity.newIntent(this);
                PendingIntent pi = PendingIntent.getActivity(this, 0, i, 0);

                Notification notification = new NotificationCompat.Builder(this)
                        .setTicker(resources.getString(R.string.new_pictures_title))
                        .setSmallIcon(android.R.drawable.ic_menu_report_image)
                        .setContentTitle(resources.getString(R.string.new_pictures_title))
                        .setContentText(resources.getString(R.string.new_pictures_text))
                        .setContentIntent(pi)
                        .setAutoCancel(true)
                        .build();

                NotificationManagerCompat notificationManager =
                        NotificationManagerCompat.from(this);
                notificationManager.notify(0, notification);
            }
            QueryPreferences.setLastResultId(this, resultId);
        }
        ...
		```

		+ 首先，调用`setTicker(CharSequence`)和`setSmallIcon(int)`方法，配置ticker text和小图标。
		+ 然后配置Notification在下拉抽屉中的外观。虽然可以定制Notification视图的显示外观和样式，但使用带有图标、标题以及文字显示区域的标准视图要相对更容易些。图标的值来自于`setSmallIcon(int)`方法，而设置标题和显示文字，需分别调用`setContentTitle(CharSequence)`和`setContentText(CharSequence)`方法。
		+ 接下来，须指定用户点击Notification消息时所触发的动作行为。与AlarmManager类似，这里通过使用PendingIntent来完成指定任务。用户在下拉抽屉中点击Notification消息时，传入`setContentIntent(PendingIntent)`方法的PendingIntent将会被发送。调用`setAutoCancel(true)`方法可调整上述行为。使用`setAutoCancel(true)`设置方法，用户点击Notification消息后，也可将该消息从消息抽屉中删除。
		+ 最后，调用`NotificationManager.notify(...)`方法。传入的整数参数是通知消息的标识符，在整个应用中该值应该是唯一的。如使用同一ID发送两条消息，则第二条消息会替换掉第一条消息。这也是进度条或其他动态视觉效果的实现方式。

+ 深入学习：服务细节内容。
	+ 服务的能与不能
		+ 与activity一样，服务是一个提供了生命周期回调方法的应用组件。而且，这些回调方法同样也会在主UI线程上运行。
		+ 初始创建的服务不会在后台线程上运行任何代码。这也是推荐使用IntentService的最主要原因。大多重要服务都需要某种后台线程，而IntentService已提供了一套标准实现代码。
	+ 服务的生命周期
		通过startService(Intent)方法启动的服务，其生命周期很简单，并具有四种生命周期回调方法。
		+ onCreate(...)方法。服务创建时调用。
		+ onStartCommand(Intent,int,int)方法。每次组件通过startService(Intent)方法启动服务时调用一次。两个整数参数，一个是一组标识符，一个是启动ID。标识符用来表示当前intent发送究竟是一次重新发送，还是一次从没成功过的发送。每次调用onStartCommand(Intent,int,int)方法，启动ID都会不同。因此，启动ID也可用于区分不同的命令。
		+ onDestroy()方法。服务不再需要时调用。通常是在服务停止后。

		还有一个问题：服务是如何停止的？根据所开发服务的具体类型，有多种方式可以停止服务。服务的类型由onStartCommand(...)方法的返回值决定，可能的服务类型有`Service.START_NOT_STICKY`、`START_REDELIVER_INTENT`和`START_STICKY`等。

	+ non-sticky服务
		+ IntentService是一种non-sticky服务。non-sticky服务在服务自己认为已完成任务时停止。为获得non-sticky服务，应返回START_NOT_STICKY或START_REDELIVER_INTENT。
		+ 通过调用stopSelf()或stopSelf(int)方法，我们告诉Android任务已完成。stopSelf()是一个无条件方法。不管onStartCommand(...)方法调用多少次，该方法总是会成功停止服务。
		+ IntentService使用的是stopSelf(int)方法。该方法需要来自于onStartCommand(...)方法的启动ID。只有在接收到最新启动ID后，该方法才会停止服务。（这也是IntentService工作的后台实现部分。）
		+ 返回START_NOT_STICKY和START_REDELIVER_INTENT有什么不同呢？区别就在于，如果系统需要在服务完成任务之前关闭它，则服务的具体表现会有所不同。START_NOT_STICKY型服务会被关闭。而START_REDELIVER_INTENT型服务，则会在可用资源不再吃紧时，尝试再次启动服务。
		+ 根据操作于应用的重要程度，在START_NOT_STICKY和START_REDELIVER_INTENT之间做出选择。如服务并不重要，则选择START_NOT_STICKY。PhotoGallery应用中，服务根据定时器的设定重复运行。如发生方法调用失败，也不会产生严重后果，因此应选择START_NOT_STICKY，同时，它也是IntentService的默认行为。我们也可调用IntentService.setIntentRedelivery(true)方法，切换使用START_REDELIVER_INTENT。

	+ sticky服务
		+ sticky 服务会持续运行，直到外部组件调用Context.stopService(Intent)方法让它停止为止。为启动sticky服务，应返回START_STICKY。
		+ sticky服务启动后会持续运行，直到某个组件调用Context.stopService(Intent)方法为止。如因某种原因需终止服务，可传入一个null intent给onStartCommand(...)方法，实现服务的重启。
		+ sticky服务适用于长时间运行的服务，如音乐播放器这种启动后一直保持运行状态，直到用户主动停止的服务。即使是这样，也应考虑一种使用non-sticky服务的替代架构方案。sticky服务的管理很不方便，因为判断服务是否已启动会比较困难。

	+ 绑定服务
		+ 除以上各类型服务外，也可使用bindService(Intent,ServiceConnection,int)方法绑定一个服务。通过服务绑定可连接到一个服务并直接调用它的方法。ServiceConnection是代表服务绑定的一个对象，并负责接收全部绑定回调方法。
		+ 在fragment中，绑定代码示例如下：

			```java
			private ServiceConnection mServiceConnection = new ServiceConnection() {
				public void onServiceConnected(ComponentName className, IBinder service) {
					// Used to communicate with the service
					MyBinder binder = (MyBinder)service;
				}

				public void onServiceDisconnected(ComponentName className) {
				}
			};

			@Override
			public void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);

				Intent i = new Intent(c, MyService.class);
				c.bindService(i, mServiceConnection, 0);
			}

			@Override
			public void onDestroy() {
				super.onDestroy();
					getActivity().getApplicationContext().unbindService(mServiceConnection);
			}
			```

		+ 对服务来说，绑定引入了另外两个生命周期回调方法：
			+ `onBind(Intent)`方法。绑定服务时调用，返回来自`ServiceConnection.onService Connected(ComponentName,IBinder)`方法的IBinder对象。
		+ `onUnbind(Intent)`方法。服务绑定终止时调用。

	+ 本地服务绑定
		+ MyBinder是一种怎样的对象呢？如果服务是一个本地服务，MyBinde就可能是本地进程中一个简单的Java对象。通常，MyBinde用于提供一个句柄，以便直接调用服务方法：

			```java
			private class MyBinder extends IBinder {
				public MyService getService() {
					return MyService.this;
				}
			}
			@Override
			public void onBind(Intent intent) {
				return new MyBinder();
			}
			```

		+ 这种模式看上去让人激动。这是Android系统中唯一一处支持组件间直接对话的地方。不过，并不推荐此种模式。服务是高效的单例，与仅使用一个单例相比，使用此种模式则显现不出优势。

	+ 远程服务绑定
		+ 绑定更适用于远程服务，因为它们赋予了其他进程的应用调用服务方法的能力。创建远程绑定服务属于高级主题，可查阅AIDL或Messager类，了解更多相关内容。

+　深入学习：JobScheduler和JobServices；Sync Adapters












































































