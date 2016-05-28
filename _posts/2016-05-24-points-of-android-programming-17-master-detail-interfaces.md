---
layout: post
title: "第17章 Master-Detail用户界面"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Mater-Detail","CriminalIntent"]
description: "使用Intents拍照"
---
{% include JB/setup %}

#### 17.Master-Detail用户界面

+ 在手机设备上，CrimeListActivity生成的是单版面（`single-pane`）布局。而在平板设备上，为同时显示主从视图，我们需要它生成双版面（`two-pane`）布局。在双版面布局中，CrimeListActivity将同时托管CrimeListFragment和CrimeFragment。

![]({{ IMAGE_PATH }}/不同类型的布局.jpg)

+ 要实现双版面布局，需执行如下操作步骤：
	+ 修改SingleFragmentActivity，不再硬编码实例化布局；
	+ 创建包含两个fragment容器的布局；
	+ 修改CrimeListActivity，实现在手机设备上实例化单版面布局，而在平板设备上实例化双版面布局。

+ 修改SingleFragmentActivity(`SingleFragmentActivity.java`)

	```java
    public abstract class SingleFragmentActivity extends AppCompatActivity {
        protected abstract Fragment createFragment();

        @LayoutRes
        protected int getLayoutResId() {
            return R.layout.activity_fragment;
        }

        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);

            setContentView(getLayoutResId());
            FragmentManager fm = getSupportFragmentManager();
            Fragment fragment = fm.findFragmentById(R.id.fragment_container);

            if (fragment == null) {
                fragment = createFragment();
                fm.beginTransaction()
                    .add(R.id.fragment_container, fragment)
                    .commit();
            }
        }
    }
	```

+ 创建包含两个fragment容器的布局(`activity_twopane.xml`)

	```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:divider="?android:attr/dividerHorizontal"
        android:showDividers="middle"
        android:orientation="horizontal">
        <FrameLayout
            android:id="@+id/fragment_container"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1">
        </FrameLayout>
        <FrameLayout
            android:id="@+id/detail_fragment_container"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="3">
        </FrameLayout>
    </LinearLayout>
	```

+ 使用别名资源。(`res/values/refs.xml`)

	```xml
    <resources>
      <item name="activity_masterdetail"
            type="layout">
            @layout/activity_fragment
      </item>
    </resources>
	```

	+ 创建平板设备专用可选资源。（`res/values-sw600dp/refs.xml`）

		```xml
		<resources>
			<item name="activity_masterdetail"
					type="layout">
					@layout/activity_twopane
			</item>
		</resources>
		```

	+ 将`CrimeListActivity`的布局资源ID设为别名资源ID。(`CrimeListActivity.java`)

		```java
		@Override
		protected int getLayoutResId() {
			return R.layout.activity_masterdetail;
		}
		```

+ 托管activity实现fragment定义的回调接口(`Callbacks`)，履行托管fragment的任务。
	+ 回调接口定义了fragment委托给托管activity处理的工作任务。任何打算托管目标fragment的activity必须实现这些定义的接口。
	+ 要实现一个fragment的Callbacks接口：
		+ 首先定义一个成员变量存放实现Callbacks接口的对象。
		+ 然后将托管activity强制类型转换为Callbacks对象并赋值给Callbacks类型变量。该操作在fragment的`onAttach()`生命周期方法中处理。
		+ 在fragment的`onDetach()`方法中将Callbacks变量设置为null。
	+ 在`CrimeListFragment`中定义回调接口：

		```java
		public class CrimeListFragment extends Fragment {
			...
			private boolean mSubtitleVisible;
			private Callbacks mCallbacks;

			/**
			 * 托管Activity需要的接口.
			 */
			public interface Callbacks {
				void onCrimeSelected(Crime crime);
			}

			@Override
			public void onAttach(Activity activity) {
				super.onAttach(activity);
				// 将托管activity强制转换为Callbacks类型
				mCallbacks = (Callbacks) activity;
			}
			...
			@Override
			public void onDetach() {
				super.onDetach();
				mCallbacks = null;
			}
			...
		}
		```

	+ Activity实现回调接口（`CrimeListActivity.java`）

		```java
		public class CrimeListActivity extends SingleFragmentActivity
			implements CrimeListFragment.Callbacks {

			@Override
			protected Fragment createFragment() {
				return new CrimeListFragment();
			}

			@Override
			protected int getLayoutResId() {
				return R.layout.activity_twopane;
			}

			// fragment委托的工作任务
			@Override
			public void onCrimeSelected(Crime crime) {
				if (findViewById(R.id.detail_fragment_container) == null) {
					Intent intent = CrimePagerActivity.newIntent(this, crime.getId());
					startActivity(intent);
				} else {
					Fragment newDetail = CrimeFragment.newInstance(crime.getId());

					getSupportFragmentManager().beginTransaction()
							.replace(R.id.detail_fragment_container, newDetail)
							.commit();
				}
			}
		}
		```

	+ 在`CrimeListFragment`中，在启动新的`CrimePagerActivity`的地方，调用`onCrimeSelected(Crime)`回调方法：

		```java
        ...
        @Override
        public boolean onOptionsItemSelected(MenuItem item) {
            switch (item.getItemId()) {
                case R.id.menu_item_new_crime:
                    Crime crime = new Crime();
                    CrimeLab.get(getActivity()).addCrime(crime);
                 	updateUI();

                    mCallbacks.onCrimeSelected(crime);

                    return true;
                case R.id.menu_item_show_subtitle:
                    ...
                default:
                    return super.onOptionsItemSelected(item);
            }
        }
        ...
        private class CrimeHolder extends RecyclerView.ViewHolder
                implements View.OnClickListener {
            ...
            @Override
            public void onClick(View v) {

                mCallbacks.onCrimeSelected(mCrime);

            }
            ...
        }
        ...
		```

+ 实现`CrimeFragment.Callbacks`回调接口，使得`Master`界面在`Detail`界面发生改变后能立即刷新。
	+ 添加CrimeFragment回调接口。

		```java
        ...
        private ImageButton mPhotoButton;
        private ImageView mPhotoView;
        private Callbacks mCallbacks;

        /**
         * 托管Activity需要实现的接口
         */
        public interface Callbacks {
            void onCrimeUpdated(Crime crime);
        }
        ...
        @Override
        public void onAttach(Activity activity) {
            super.onAttach(activity);
            mCallbacks = (Callbacks)activity;
        }
        ...
        @Override
        public void onDetach() {
            super.onDetach();
            mCallbacks = null;
        }
        ...
		```

	+ 托管activity执行fragment的委托任务(`CrimeListActivity.java`)

		```java
        public class CrimeListActivity extends SingleFragmentActivity
            implements CrimeListFragment.Callbacks, CrimeFragment.Callbacks {
            ...
            public void onCrimeUpdated(Crime crime) {
                FragmentManager fm = getSupportFragmentManager()
                CrimeListFragment listFragment = (CrimeListFragment)
                        fm.findFragmentById(R.id.fragment_container);

                listFragment.updateUI();
            }
        }
		```

	+ 当`CrimeFragment`完成对数据的修改后需要刷新数据库，添加刷新方法。之后在所有数据发生修改的场合，调用刷新方法。(`CrimeFragment.java`)

		```java
        ...
        @Override
        public View onCreateView(...) {
            ...
            mTitleField.addTextChangedListener(new TextWatcher() {
                ...
                @Override
                public void onTextChanged(CharSequence s, int start, int before, int count) {
                    mCrime.setTitle(s.toString());
					// 修改标题需要刷新
                    updateCrime();
                }
                ...
            });
            ...
            mSolvedCheckbox.setOnCheckedChangeListener(new OnCheckedChangeListener() {
                @Override
                public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
                    mCrime.setSolved(isChecked);
					// 修改确认框需要刷新
                    updateCrime();
                }
            });
            ...
        }
        ...
        // 选择日期，联系人和照片后同样也要刷新，即使双版面的`Master`版面不显示这些。
        @Override
        public void onActivityResult(...) {
            if (requestCode == REQUEST_DATE) {
                ...
                mCrime.setDate(date);
                updateCrime();
                updateDate();
            } else if (requestCode == REQUEST_CONTACT && data != null) {
                ...
                try {
                    ...
                    String suspect = c.getString(0);
                    mCrime.setSuspect(suspect);
                    updateCrime();
                    mSuspectButton.setText(suspect);
                } finally {
                    c.close();
                }
            } else if (requestCode == REQUEST_PHOTO) {
                updateCrime();
                updatePhotoView();
            }
        }

        private void updateCrime() {
            CrimeLab.get(getActivity()).updateCrime(mCrime);
            mCallbacks.onCrimeUpdated(mCrime);
        }

        private void updateDate() {
            mDateButton.setText(mCrime.getDate().toString());
        }
        ...
		```





























































































