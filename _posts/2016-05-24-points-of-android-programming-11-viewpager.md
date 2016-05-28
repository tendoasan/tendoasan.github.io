---
layout: post
title: "第11章 ViewPager"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","ViewPager","CriminalIntent"]
description: "使用ViewPager"
---
{% include JB/setup %}

#### 11.使用ViewPager

+ 创建`CrimePagerActivity`：
	+ 布局文件：

		```xml
		<android.support.v4.view.ViewPager
			xmlns:android="http://schemas.android.com/apk/res/android"
			android:id="@+id/activity_crime_pager_view_pager"
			android:layout_width="match_parent"
			android:layout_height="match_parent">
		</android.support.v4.view.ViewPager>
		```

	+ Java Class文件：

		```java
		public class CrimePagerActivity extends FragmentActivity {
			@Override
			protected void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);
				setContentView(R.layout.activity_crime_pager);
			}
		}
		```

	+ 在`AndroidManifest.xml`文件中添加`CrimePagerActivity`。

+ 一个`ViewPager`需要一个`PagerAdapter`，`FragmentStatePagerAdapter`对二者间的配合支持实际归结为两个简单方法的使用，即`getCount()`和`getItem(int)`。调用`getItem(int)`方法获取`crime`数组指定位置的`Crime`时，它会返回一个已配置的用于显示指定位置`crime`信息的`CrimeFragment`：

	```java
    public class CrimePagerActivity extends FragmentActivity {
        private ViewPager mViewPager;
        private List<Crime> mCrimes;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_crime_pager);

            mViewPager = (ViewPager) findViewById(R.id.activity_crime_pager_view_pager);
            mCrimes = CrimeLab.get(this).getCrimes();
            FragmentManager fm = getSupportFragmentManager();

            mViewPager.setAdapter(new FragmentStatePagerAdapter(fm) {
                @Override
                public Fragment getItem(int position) {
                    Crime crime = mCrimes.get(position);
                    return CrimeFragment.newInstance(crime.getId());
                }

                @Override
                public int getCount() {
                    return mCrimes.size();
                }
            });
        }
    }
	```

+ 集成`CrimePagerActivity`：
	+ 添加`newIntent()`方法：

		```java
		public class CrimePagerActivity extends FragmentActivity {
			private static final String EXTRA_CRIME_ID = "extra.crime_id";

			private ViewPager mViewPager;
			private List<Crime> mCrimes;

			public static Intent newIntent(Context packageContext, UUID crimeId) {
				Intent i = new Intent(packageContext, CrimePagerActivity.class);
				i.putExtra(EXTRA_CRIME_ID, crimeId);
				return i;
			}

			@Override
			protected void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);
				setContentView(R.layout.activity_crime_pager);

				UUID crimeId = (UUID) getIntent().getSerializableExtra(EXTRA_CRIME_ID);
				...
			}
		}
		```

	+ 在`CrimeListFragment`的itemView监听器中改成启动`CrimePagerActivity`：

		```java
		private class CrimeHolder extends RecyclerView.ViewHolder
				implements View.OnClickListener {
			...
			@Override
			public void onClick(View v) {
				Intent i = CrimePagerActivity.newIntent(getActivity(), mCrime.getId());
				startActivity(i);
			}
		}
		```

	+ `ViewPager`在他的`PagerAdapter`中默认显示第一个item。使用`ViewPager`的`setCurrentItem(int)`方法来设置`ViewPager`要默认显示的item。

		```java
		public class CrimePagerActivity extends FragmentActivity {
			@Override
			public void onCreate(Bundle savedInstanceState) {
				...
				FragmentManager fm = getSupportFragmentManager();
				mViewPager.setAdapter(new FragmentStatePagerAdapter(fm) {
					...
				});

				for (int i = 0; i < mCrimes.size(); i++) {
					if (mCrimes.get(i).getId().equals(crimeId)) {
						mViewPager.setCurrentItem(i);
						break;
					}
				}
			}
		}
		```

+ `FragmentStatePagerAdapter` vs. `FragmentPagerAdapter`
+ How `ViewPager` Really Works
