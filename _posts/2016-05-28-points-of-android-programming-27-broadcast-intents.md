---
layout: post
title: "第27章 广播Intents"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Broadcast","PhotoGallery"," Intents"]
description: "Broadcast Intents"
---
{% include JB/setup %}

#### 27.Broadcast Intents

+ broadcast intent的工作原理类似之前的intent，唯一不同的是broadcast intent可同时被多个组件接收。普通intent和broadcast intent：

	![]({{IMAGE_PATH}}/普通intent和broadcast intent.jpg)

+ 接收一个系统广播：随设备重启而唤醒的定时器。
	+ 新建一个单独的`receiver`。(`StartupReceiver.java`)

		```java
		public class StartupReceiver extends BroadcastReceiver{
			private static final String TAG = "StartupReceiver";

			@Override
			public void onReceive(Context context, Intent intent) {
				Log.i(TAG, "Received broadcast intent: " + intent.getAction());
			}
		}
		```

	+ 登记关联receiver，配置使用权限监听`BOOT_COMPLETED`操作。(`AndroidManifest.xml`)

		```xml
		<manifest ...>
			<uses-permission android:name="android.permission.INTERNET"/>
			<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
			<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
			<application
				...>
				<activity
					android:name=".PhotoGalleryActivity"
					android:label="@string/app_name">
					...
				</activity>

				<service android:name=".PollService"/>

				<receiver android:name=".StartupReceiver">
					<intent-filter>
						<action android:name="android.intent.action.BOOT_COMPLETED"/>
					</intent-filter>
				</receiver>
			</application>
		</manifest>
		```

	+ 如何使用 receiver
		+ broadcast receiver的存在如此短暂，因此它的作用有限。例如，我们无法使用任何异步API或登记任何监听器，因为onReceive(Context,Intent)方法刚运行完，receiver就不存在了。onReceive(Context,Intent)方法同样运行在主线程上，因此不能在该方法内做一些耗时的重度任务，如网络连接或数据的永久存储等。
		+ 对于轻型任务代码的运行而言，receiver非常有用。系统重启后，定时运行的定时器也需进行重置。
		+ receiver需要知道定时器的启停状态。在PollService类中添加一个preference常量，用于存储状态信息。(`QueryPreferences.java`)

			```java
			public class QueryPreferences {
				private static final String PREF_SEARCH_QUERY = "searchQuery";
				private static final String PREF_LAST_RESULT_ID = "lastResultId";
				private static final String PREF_IS_ALARM_ON = "isAlarmOn";
				...
				public static void setLastResultId(Context context, String lastResultId) {
					...
				}

				public static boolean isAlarmOn(Context context) {
					return PreferenceManager.getDefaultSharedPreferences(context)
							.getBoolean(PREF_IS_ALARM_ON, false);
				}

				public static void setAlarmOn(Context context, boolean isOn) {
					PreferenceManager.getDefaultSharedPreferences(context)
							.edit()
							.putBoolean(PREF_IS_ALARM_ON, isOn)
							.apply();
				}
			}
			```

		+ 定时器打开后，存储开关状态(`PollService.java`)

			```java
			public class PollService extends IntentService {
				...
				public static void setServiceAlarm(Context context, boolean isOn) {
					...
					if (isOn) {
						alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME,
								SystemClock.elapsedRealtime(), POLL_INTERVAL, pi);
					} else {
						alarmManager.cancel(pi);
						pi.cancel();
					}

					QueryPreferences.setAlarmOn(context, isOn);
				}
				...
			}
			```

		+ 接收到重启广播后启动定时器。(`StartupReceiver.java`)

			```java
			public class StartupReceiver extends BroadcastReceiver{
				private static final String TAG = "StartupReceiver";

				@Override
				public void onReceive(Context context, Intent intent) {
					Log.i(TAG, "Received broadcast intent: " + intent.getAction());

					boolean isOn = QueryPreferences.isAlarmOn(context);
					PollService.setServiceAlarm(context, isOn);
				}
			}
			```

+ 过滤前台通知消息
	+ 应用的通知消息虽然工作良好，但在打开应用后，我们依然会收到通知消息。可以利用broadcast intent来解决这个问题。
	+ 发送 broadcast intent。要发送broadcast intent，只需创建一个intent，并传入`sendBroadcast(Intent)`方法即可。这里，需要通过`sendBroadcast(Intent)`方法广播自己定义的操作（action），因此还需要定义一个操作常量。(`PollService.java`)

		```java
		public class PollService extends IntentService {
			private static final String TAG = "PollService";
			private static final long POLL_INTERVAL = AlarmManager.INTERVAL_FIFTEEN_MINUTES;

			public static final String ACTION_SHOW_NOTIFICATION =
					"com.tendoasan.android.photogallery.SHOW_NOTIFICATION";
			...
			@Override
			protected void onHandleIntent(Intent intent) {
				...
				String resultId = items.get(0).getId();
				if (resultId.equals(lastResultId)) {
					Log.i(TAG, "Got an old result: " + resultId);
				} else {
					...
					NotificationManagerCompat notificationManager =
							NotificationManagerCompat.from(this);
					notificationManager.notify(0, notification);
					// 发生消息之后发送广播
					sendBroadcast(new Intent(ACTION_SHOW_NOTIFICATION));
				}
				QueryPreferences.setLastResultId(this, resultId);
			}
			...
		}
		```

	+ 动态 broadcast receiver
		+ receiver在不断接收intent的同时，还需要知晓PhotoGalleryFragment的存在状态。使用动态broadcast receiver可解决该问题。
		+ 如果receiver声明在manifest配置文件里，且仅限应用内部使用，则可在receiver标签上添加一个`android:exported="false"`属性。这样，系统中的其他应用就再也无法接触到该receiver。另外，也可创建自己的使用权限。这通常通过在`AndroidManifest.xml`中添加一个permission标签来完成。(`AndroidManifest.xml`)

			```xml
			<manifest ...>
				<permission android:name="com.tendoasan.android.photogallery.PRIVATE"
					android:protectionLevel="signature" />

				<uses-permission android:name="android.permission.INTERNET" />
				<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
				<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
				<uses-permission android:name="com.tendoasan.android.photogallery.PRIVATE" />

				<application
					... >
					...
				</application>
			</manifest>
			```

		+ 发送带有权限的broadcast （`PollService.java`）

			```java
			public class PollService extends IntentService {
				...
				public static final String ACTION_SHOW_NOTIFICATION =
						"com.tendoasan.android.photogallery.SHOW_NOTIFICATION";
				public static final String PERM_PRIVATE =
						"com.tendoasan.android.photogallery.PRIVATE";

				public static Intent newIntent(Context context) {
					return new Intent(context, PollService.class);
				}
				...
				@Override
				protected void onHandleIntent(Intent intent) {
					...
					String resultId = items.get(0).getId();
					if (resultId.equals(lastResultId)) {
						Log.i(TAG, "Got an old result: " + resultId);
					} else {
						...
						notificationManager.notify(0, notification);
						sendBroadcast(new Intent(ACTION_SHOW_NOTIFICATION), PERM_PRIVATE);
					}
					QueryPreferences.setLastResultId(this, resultId);
				}
				...
			}
			```

		+ 要在代码中登记receiver，可调用`registerReceiver(BroadcastReceiver, IntentFilter)`方法；取消登记时，则调用`unregisterReceiver(BroadcastReceiver)`方法。在`registerReceiver(...)`和`unregisterReceiver(...)`方法中，需要同一个实例，因此需要将receiver赋值给一个实例变量。
		+ 以Fragment为超类，新建一个`VisibleFragment`抽象类该类是一个隐藏前台通知的通用fragment。

			```java
			public abstract class VisibleFragment extends Fragment {
				private static final String TAG = "VisibleFragment";

				@Override
				public void onStart() {
					super.onStart();
					IntentFilter filter = new IntentFilter(PollService.ACTION_SHOW_NOTIFICATION);
					getActivity().registerReceiver(mOnShowNotification, filter, PollService.PERM_PRIVATE, null);
				}

				@Override
				public void onStop() {
					super.onStop();
					getActivity().unregisterReceiver(mOnShowNotification);
				}

				private BroadcastReceiver mOnShowNotification = new BroadcastReceiver() {
					@Override
					public void onReceive(Context context, Intent intent) {
						Toast.makeText(getActivity(),
								"Got a broadcast:" + intent.getAction(),
								Toast.LENGTH_LONG)
							 .show();
					}
				};
			}
			```

		+ 任何使用XML定义的IntentFilter，均可以代码的方式完成定义。要配置以代码方式创建的IntentFilter，直接调用`addCategory(String)`、`addAction(String)`和`addDataPath(String)`等方法。
		+ 使用完后，动态登记的broadcast receiver必须能够自我清除。通常，如果在启动生命周期方法中登记了receiver，则需在相应的停止方法中调用`Context.unregisterReceiver(BroadcastReceiver)`方法。
	+ 修改`PhotoGalleryFragment`，调整其父类为`VisibleFragment`。

+ 使用 bordered broadcast 接收结果
	+ 使用有序broadcast intent实现双向通信。有序broadcast允许多个broadcast receiver依序处理broadcast intent。另外，通过传入一个名为result receiver的特别broadcast receiver，有序broadcast还可实现让broadcast的发送者接收broadcast接收者发送的返回结果。

		![]({{ IMAGE_PATH }}/有序broadcast intent.jpg)

	+ 从接收方来看，获得了一个特别工具：一套改变接收者返回值的方法。这里需要取消通知信息。可通过一个简单的整数结果码，将此需要告知信息发送者。使用`setResultCode(int)`方法，设置结果码为`Activity.RESULT_CANCELED`。
	+ 修改`VisibleFragment`类，将取消通知的信息发送给`SHOW_NOTIFICATION`的发送者。

		```java
        private BroadcastReceiver mOnShowNotification = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                // If we receive this, we're visible, so cancel
                // the notification
                Log.i(TAG, "canceling notification");
                setResultCode(Activity.RESULT_CANCELED);
            }
        };
		```

	+ 既然此处只需发送YES或NO标志，因此使用int结果码即可。如需返回更多复杂数据，可使用`setResultData(String)`或`setResultExtras(Bundle)`方法。如需设置所有三个参数值，可调用`setResult(int,String,Bundle)`方法。设定返回值后，每个后续接收者均可看到或修改返回值。
	+ 为让以上方法发挥作用，broadcast必须有序。在`PollService`中，编写一个可发送有序broadcast的新方法。该方法打包一个Notification调用，然后作为一个 broadcast发出。只要通知信息还没被撤消，可指定一个result receiver发出打包的Notification。

		```java
		...
        public static final String REQUEST_CODE = "REQUEST_CODE";
        public static final String NOTIFICATION = "NOTIFICATION";
        ...
        @Override
        protected void onHandleIntent(Intent intent) {
            ...
            String resultId = items.get(0).getId();
            if (resultId.equals(lastResultId)) {
                Log.i(TAG, "Got an old result: " + resultId);
            } else {
                Log.i(TAG, "Got a new result: " + resultId);
                ...
                Notification notification = ...;

                showBackgroundNotification(0, notification);
            }
            QueryPreferences.setLastResultId(this, resultId);
        }
        private void showBackgroundNotification(int requestCode, Notification notification) {
            Intent i = new Intent(ACTION_SHOW_NOTIFICATION);
            i.putExtra("REQUEST_CODE", requestCode);
            i.putExtra("NOTIFICATION", notification);
            sendOrderedBroadcast(i, PERM_PRIVATE, null, null,
                Activity.RESULT_OK, null, null);
        }
		```

		+ 除了在`sendBroadcast(Intent,String)`方法中使用的参数外，`Context.sendOrderedBroadcast(Intent,String,BroadcastReceiver,Handler,int,String,Bundle)`方法还有另外五个参数，依次为：一个result receiver、一个支持result receiver运行的Handler、结果代码初始值、结果数据以及有序broadcast的结果附加内容。
		+ result receiver比较特殊，只有在所有有序broadcast intent的接收者结束运行后，它才开始运行。虽然有时可使用result receiver接收broadcast和发送通知对象，但此处该方法行不通。目标broadcast intent通常是在PollService对象消亡之前发出的，也就是说broadcast receiver可能也被销毁了。
		+ 因此，最终的broadcast receiver需要保持独立运行。
	+ 以BroadcastReceiver为父类，新建一个NotificationReceiver类。(`NotificationReceiver.java`)

		```java
        public class NotificationReceiver extends BroadcastReceiver {
            private static final String TAG = "NotificationReceiver";

            @Override
            public void onReceive(Context c, Intent i) {
                Log.i(TAG, "received result: " + getResultCode());
                if (getResultCode() != Activity.RESULT_OK) {
                    // A foreground activity cancelled the broadcast
                    return;
                }

                int requestCode = i.getIntExtra(PollService.REQUEST_CODE, 0);
                Notification notification = (Notification)
                        i.getParcelableExtra(PollService.NOTIFICATION);

                NotificationManagerCompat notificationManager =
                        NotificationManagerCompat.from(c);
                notificationManager.notify(requestCode, notification);
            }
        }
		```

	+ 注册notification receiver (`AndroidManifest.xml`)

		```xml
		<manifest ...>
			...
			<application
				... >
				...
				<receiver android:name=".StartupReceiver">
					<intent-filter>
						<action android:name="android.intent.action.BOOT_COMPLETED" />
					</intent-filter>
				</receiver>
				<receiver android:name=".NotificationReceiver"
					android:exported="false">
					<intent-filter
						android:priority="-999">
						<action
						android:name="com.tendoasan.android.photogallery.SHOW_NOTIFICATION" />
					</intent-filter>
				</receiver>
			</application>
		</manifest>
		```

+ receiver 与长时运行任务
	+ 如不想受限于主线程的时间限制，并希望broadcast intent可触发一个长时运行任务，该怎么做呢？
	+ 有两种方式可以选择。
		+ 将任务交给服务去处理，然后再通过broadcast receiver启动服务。这也是我们推荐的方式。服务可以运行很久，直到完成需要处理的任务。同时服务可将请求放在队列中，然后依次进行处理，或按其自认为合适的方式管理全部任务请求。
		+ 使用`BroadcastReceiver.goAsync()`方法。该方法返回一个`BroadcastReceiver.PendingResult`对象，随后，我们可使用该对象提供结果。因此，可将PendingResult交给AsyncTask去执行长时运行的任务，然后再调用PendingResult的方法响应broadcast。
	+ BroadcastReceiver.goAsync()方法有两处弊端。首先它不支持旧设备。其次，它不够灵活：我们仍需快速响应broadcast，并且与使用服务相比，没什么架构模式好选择。
	+ goAsync()方法并非一无是处：可通过该方法的调用，完成有序broadcast的结果设置。如果真的要使用它，应注意不要耗时过长。

+ 深入探究：使用event bus处理 Local Events，借助EventBus&RxJava第三方库；探测fragment的可见性。






































