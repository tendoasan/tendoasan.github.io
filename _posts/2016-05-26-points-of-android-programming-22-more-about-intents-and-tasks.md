---
layout: post
title: "第22章 深入学习intent和任务"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Intent","NerdLauncher","Task"]
description: "深入学习intent和任务"
---
{% include JB/setup %}

#### 22.深入学习intent和任务

+ 创建`NerdLauncher`启动器应用。
	+ 布局文件，使用`RecyclerView`显示启动应用列表。(`layout/fragment_nerd_launcher.xml`)
	+ Activity。(`NerdLauncherActivity.java`)

		```java
        public class NerdLauncherActivity extends SingleFragmentActivityAppCompatActivity {
            @Override
            protected Fragment createFragment() {
                return NerdLauncherFragment.newInstance();
            }
        }
		```

	+ Fragment。(`NerdLauncherFragment.java`)

		```java
        public class NerdLauncherFragment extends Fragment {
            private static final String TAG = "NerdLauncherFragment";
            private RecyclerView mRecyclerView;

            public static NerdLauncherFragment newInstance() {
                return new NerdLauncherFragment();
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                View v = inflater.inflate(R.layout.fragment_nerd_launcher, container, false);
                mRecyclerView = (RecyclerView) v
                                .findViewById(R.id.fragment_nerd_launcher_recycler_view);
                mRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
                setupAdapter();
                return v;
            }

            private void setupAdapter() {
                Intent startupIntent = new Intent(Intent.ACTION_MAIN);
                startupIntent.addCategory(Intent.CATEGORY_LAUNCHER);

                PackageManager pm = getActivity().getPackageManager();
                // 查询所有可启动的 activity 的标签及其他元数据
                // 这些数据都包含在 ResolceInfo对象中。
                List<ResolveInfo> activities = pm.queryIntentActivities(startupIntent, 0);
                // 将所有 ResolveInfo对象按字母顺序排序
                Collections.sort(activities, new Comparator<ResolveInfo>() {
                    public int compare(ResolveInfo a, ResolveInfo b) {
                        PackageManager pm = getActivity().getPackageManager();
                        return String.CASE_INSENSITIVE_ORDER.compare(
                                a.loadLabel(pm).toString(),
                                b.loadLabel(pm).toString());
                    }
                });
                Log.i(TAG, "Found " + activities.size() + " activities.");
				// 设置 Adapter
                mRecyclerView.setAdapter(new ActivityAdapter(activities));
            }
        }
		```

	+ 创建`ActivityAdapter`。(`NerdLauncherFragment.java`)

		```java
		public class NerdLauncherFragment extends Fragment {
			...
			private void setupAdapter() {
				...
			}

			private class ActivityHolder extends RecyclerView.ViewHolder {
				private ResolveInfo mResolveInfo;
				private TextView mNameTextView;

				public ActivityHolder(View itemView) {
					super(itemView);
					mNameTextView = (TextView) itemView;
				}

				public void bindActivity(ResolveInfo resolveInfo) {
					mResolveInfo = resolveInfo;
					PackageManager pm = getActivity().getPackageManager();
					// 从 ResolveInfo对象中取出启动activity的标签
					String appName = mResolveInfo.loadLabel(pm).toString();
					mNameTextView.setText(appName);
				}
			}

			private class ActivityAdapter extends RecyclerView.Adapter<ActivityHolder> {
				private final List<ResolveInfo> mActivities;

				public ActivityAdapter(List<ResolveInfo> activities) {
					mActivities = activities;
				}

				@Override
				public ActivityHolder onCreateViewHolder(ViewGroup parent, int viewType) {
					LayoutInflater layoutInflater = LayoutInflater.from(getActivity());
					View view = layoutInflater
							.inflate(android.R.layout.simple_list_item_1, parent, false);
					return new ActivityHolder(view);
				}

				@Override
				public void onBindViewHolder(ActivityHolder activityHolder, int position) {
					ResolveInfo resolveInfo = mActivities.get(position);
					activityHolder.bindActivity(resolveInfo);
				}

				@Override
				public int getItemCount() {
					return mActivities.size();
				}
			}
		}
		```

+ 在运行时创建显式Intents。为`ActivityHolder`添加单击事件，利用activity的`ActivityInfo`新建一个显式intent。(`NerdLauncherFragment.java`)

	```java
    ...
    private class ActivityHolder extends RecyclerView.ViewHolder
            implements View.OnClickListener {
        private ResolveInfo mResolveInfo;
        private TextView mNameTextView;

        public ActivityHolder(View itemView) {
            super(itemView);
            mNameTextView = (TextView) itemView;
            mNameTextView.setOnClickListener(this);
        }

        public void bindActivity(ResolveInfo resolveInfo) {
            ...
        }

        @Override
        public void onClick(View v) {
            ActivityInfo activityInfo = mResolveInfo.activityInfo;

            Intent i = new Intent(Intent.ACTION_MAIN)
                    .setClassName(activityInfo.applicationInfo.packageName,
                            activityInfo.name);

            startActivity(i);
        }
    }
	...
	```

	+ 以往创建显式intent的方式：

		```java
		public Intent(Context packageContext, Class<?> cls);
		```

		该构造方法使用传入的参数来获取`Intent`需要的`ComponentName`。`ComponentName`由包名和类名共同组成。传入`Activity`和`Class`创建`Intent`时，构造方法会通过`Activity`类自行确定全路径包名。

	+ 在这里使用`setClassName(...)`方法自动创建组件名，然后创建一个显式intent。

		```java
		public Intent setClassName(String packageName, String className);
		```

+ 任务与后退栈。
	+ 在每一个运行的应用中，Android都使用任务(`Task`)来跟踪用户的状态。任务是用户比较关心的activity栈。栈底部的activity通常称为`基 activity`。栈顶的activity是用户可以看到的activity。用户点击后退键时，栈顶activity会弹出栈外。如果当前屏幕上显示的是基activity，点击后退键，系统将退回主屏幕。
	+ 不影响各个任务的状态，任务管理器可以让我们在任务间切换。例如，如果启动进入联系人应用，然后切换到Twitter应用查看信息，这时，我们将启动两个任务。如果再切换回联系人应用，我们在两项任务中所处的状态位置会被保存下来。
	+ 有时，我们需要在当前任务中启动activity。而有时又需要在新任务中启动activity。

		![]({{ IMAGE_PATH }}/任务与后退栈.jpg)

	+ 默认情况下，新activity都在当前任务中启动。在CriminalIntent应用中，无论什么时候启动新activity，它都会被添加到当前任务中。即使要启动的activity不属于CriminalIntent应用，它同样也是在当前任务中启动。启动activity发送crime报告就是这样的一个例子。在当前任务中启动activity的好处是，用户可以在任务内而不是在应用层级间导航返回。
	+ 当前，从NerdLauncher应用启动的任何activity都会添加到NerdLauncher的任务中。
	+ 在新任务中启动activity。这样，用户可以在运行的应用间自由切换。为启动新activity时启动新任务，需要为intent添加一个`新任务`标志（`NerdLauncherFragment.java`）

		```java
        Intent i = new Intent(Intent.ACTION_MAIN);
        i.setClassName(activityInfo.applicationInfo.packageName, activityInfo.name);

        i.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

        startActivity(i);
		```

+ 使用 NerdLauncher 应用作为设备主屏幕
	+ 在`AndroidManifest.xml`文件中，在intent主过滤器添加以下类别定义。

		```xml
		<intent-filter>
			<action android:name="android.intent.action.MAIN" />
			<category android:name="android.intent.category.LAUNCHER" />
			<category android:name="android.intent.category.HOME" />
			<category android:name="android.intent.category.DEFAULT" />
		</intent-filter>
		```

	+ 通过添加`HOME`和`DEFAULT`类别定义，NerdLauncher应用的activity会成为可选的主界面。点击Home键，可以看到，在弹出的界面选择对话框中，NerdLauncher变成了主界面可选项。（如果已设置NerdLauncher应用为主界面，然后需要恢复到系统默认设置，可以选择`Settings→Applications→Manage Applications`菜单项，找到NerdLauncher应用并清除它的Launch by default选项。）

+ 挑战练习：应用图标和任务重排。

+ 进程与任务
	+ 对象需要内存和虚拟机的支持才能存在。进程是操作系统创建的供应用对象生存以及应用运行的地方。
	+ 进程通常占用由操作系统管理着的系统资源，如内存、网络端口以及打开的文件等。进程还拥有至少一个（可能多个）执行线程。在Android系统中，进程总会有一个运行的**虚拟机**。
	+ 尽管存在着未知的异常情况，但总的来说，Android世界里的每个应用组件都仅与一个进程相关联。应用伴随着自己的进程一起完成创建，该进程同时也是应用中所有组件的默认进程。
	+ 虽然，组件可以指派给不同的进程，但我们推荐使用默认进程。如果确实需要在不同进程中运行应用组件，通常也可以借助多线程来达到目的。相比多进程的使用，Android多线程的使用更加简单。
	+ 每一个activity实例都仅存在于一个进程和一个任务中。这也是进程与任务的唯一类似的地方。任务只包含activity，这些activity通常来自于不同应用。而进程则包含了应用的全部运行代码和对象。
	+ 进程与任务很容易让人混淆，主要原因在于它们不仅在概念上有某种重叠，而且通常都是以它们所属应用的名称被人提及的。例如，从NerdLauncher启动器中启动CriminalIntent应用时，操作系统创建了一个CriminalIntent进程以及一个以CrimeListActivity为`基activity`的新任务。在任务管理器中，我们可以看到标签为CriminalIntent的任务。
	+ activity赖以生存的任务和进程有可能会不同。例如，在CriminalIntent应用中启动联系人应用选择嫌疑人时，虽然联系人activity是在CriminalIntent任务中启动的，但它是在联系人应用的进程中运行的，如下图。

		![]({{ IMAGE_PATH }}/任务与进程.jpg)

	+ 这也意味着，用户点击后退键在不同activity间导航时，他或她可能还没意识到他们正在进程 间切换。























































































