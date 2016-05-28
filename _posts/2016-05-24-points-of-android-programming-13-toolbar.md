---
layout: post
title: "第13章 Toolbar"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Toolbar","CriminalIntent"]
description: "操作栏"
---
{% include JB/setup %}

#### 13.操作栏

+ 添加AppCompat依赖，使用AppCompat 主题，确保所有activity都是`AppCompatActivity`的子类。
+ 菜单布局文件(`res/menu/fragment_crime_list.xml`)

	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<menu
    	xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
        <item
			android:id="@+id/menu_item_new_crime"
			android:icon="@android:drawable/ic_menu_add"
			android:title="@string/new_crime"
			app:showAsAction="ifRoom|withText"/>
	</menu>
	```

+ 创建菜单(`CrimeListFragment.java`)：

	```java
    ...
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setHasOptionsMenu(true);
    }

    @Override
    public View onCreateView(...){
        ...
    }
    ...
    @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        super.onCreateOptionsMenu(menu, inflater);
        inflater.inflate(R.menu.fragment_crime_list, menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.menu_item_new_crime:
                Crime crime = new Crime();
                CrimeLab.get(getActivity()).addCrime(crime);
                Intent i = CrimePagerActivity.newIntent(getActivity(), crime.getId());
                startActivity(i);
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }
	```

+ 实现层级式导航--开启返回按钮(`AndroidManifest.xml`)

	```xml
    ...
    <activity
        android:name=".CrimePagerActivity"
        android:label="@string/app_name"
        android:parentActivityName=".CrimeListActivity">
    </activity>
	```

	+ 原理：

		```java
		Intent intent = new Intent(this, CrimeListActivity.class);
		intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
		startActivity(intent);
		finish();
		```

+ 添加可选菜单项，用于显示或隐藏副标题。
	+ 添加显示副标题菜单(`res/menu/fragment_crime_list.xml`)

		```xml
		<?xml version="1.0" encoding="utf-8"?>
		<menu xmlns:android="http://schemas.android.com/apk/res/android"
			  xmlns:app="http://schemas.android.com/apk/res-auto">
			<item
				android:id="@+id/menu_item_new_crime"
				android:icon="@android:drawable/ic_menu_add"
				android:title="@string/new_crime"
				app:showAsAction="ifRoom|withText"/>

			<item
				android:id="@+id/menu_item_show_subtitle"
				android:title="@string/show_subtitle"
				app:showAsAction="ifRoom"/>
		</menu>
		```

	+ 响应菜单操作(`CrimeListFragment.java`)：

		```java
		public class CrimeListFragment extends Fragment {
			...
			private boolean mSubtitleVisible;
			...
			@Override
			public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
				super.onCreateOptionsMenu(menu, inflater);
				inflater.inflate(R.menu.fragment_crime_list, menu);

				MenuItem subtitleItem = menu.findItem(R.id.menu_item_show_subtitle);
				if (mSubtitleVisible) {
					subtitleItem.setTitle(R.string.hide_subtitle);
				} else {
					subtitleItem.setTitle(R.string.show_subtitle);
				}
			}

			@Override
			public boolean onOptionsItemSelected(MenuItem item) {
				switch (item.getItemId()) {
					case R.id.menu_item_new_crime:
						...
					case R.id.menu_item_show_subtitle:

						mSubtitleVisible = !mSubtitleVisible;
						getActivity().invalidateOptionsMenu();
						updateSubtitle();

						return true;
					default:
						return super.onOptionsItemSelected(item);
				}
			}

			private void updateSubtitle() {
				CrimeLab crimeLab = CrimeLab.get(getActivity());
				int crimeCount = crimeLab.getCrimes().size();
				String subtitle = getString(R.string.subtitle_format, crimeCount);

				if (!mSubtitleVisible) {
					subtitle = null;
				}

				AppCompatActivity activity = (AppCompatActivity) getActivity();
				activity.getSupportActionBar().setSubtitle(subtitle);
			}
		}
		```

	+ 补充：当创建了一个新的`crime`后(crime个数+1)，回到`CrimeListActivity`，副标题也要随之刷新。所以需要修改一下`CrimeListFragment`的`updateUI()`方法：

		```java
		private void updateUI() {
			CrimeLab crimeLab = CrimeLab.get(getActivity());
			List<Crime> crimes = crimeLab.getCrimes();
			if (mAdapter == null) {
				mAdapter = new CrimeAdapter(crimes);
				mCrimeRecyclerView.setAdapter(mAdapter);
			} else {
				mAdapter.notifyDataSetChanged();
			}
				updateSubtitle();
			}
		```

	+ 应对旋转问题(`CrimeListFragment.java`)

		```java
		public class CrimeListFragment extends Fragment {
			private static final String SAVED_SUBTITLE_VISIBLE = "subtitle";
			...
			@Override
			public View onCreateView(LayoutInflater inflater, ViewGroup container,
									 Bundle savedInstanceState) {
				...
				if (savedInstanceState != null) {
					mSubtitleVisible = savedInstanceState.getBoolean(SAVED_SUBTITLE_VISIBLE);
				}

				updateUI();

				return view;
			}

			@Override
			public void onResume() {
				...
			}

			@Override
			public void onSaveInstanceState(Bundle outState) {
				super.onSaveInstanceState(outState);
				outState.putBoolean(SAVED_SUBTITLE_VISIBLE, mSubtitleVisible);
			}
		}
		```

+ 挑战练习：删除Crimes，Plural String Resources，An Empty View for the RecyclerView。




































































