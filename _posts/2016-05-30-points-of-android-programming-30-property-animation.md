---
layout: post
title: "第30章 动画"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Animation","Sunset"]
description: "Property Animation"
---
{% include JB/setup %}

#### 30.Property Animation

+ 构建场景。
	+ 新建`Sunset`项目。`minSdkVersion`为16。
	+ 添加颜色。(`res/values/colors.xml`)

		```xml
        <resources>
            <color name="bright_sun">#fcfcb7</color>
            <color name="blue_sky">#1e7ac7</color>
            <color name="sunset_sky">#ec8100</color>
            <color name="night_sky">#05192e</color>
            <color name="sea">#224869</color>
        </resources>
		```

	+ 添加太阳的`XML drawable`文件。(`res/drawable/sun.xml`)

		```xml
        <shape xmlns:android="http://schemas.android.com/apk/res/android"
            android:shape="oval">
            <solid android:color="@color/bright_sun" />
        </shape>
		```

	+ 设置布局文件。(`res/layout/fragment_sunset.xml`)

		```xml
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:orientation="vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
            <FrameLayout
                android:id="@+id/sky"
                android:layout_width="match_parent"
                android:layout_height="0dp"
                android:layout_weight="0.61"
                android:background="@color/blue_sky">
                <ImageView
                    android:id="@+id/sun"
                    android:layout_width="100dp"
                    android:layout_height="100dp"
                    android:layout_gravity="center"
                    android:src="@drawable/sun"
                    />
            </FrameLayout>

            <View
                android:layout_width="match_parent"
                android:layout_height="0dp"
                android:layout_weight="0.39"
                android:background="@color/sea"
                />
        </LinearLayout>
		```

	+ 创建`SunsetFragment` (`SunsetFragment.java`)

		```java
        public class SunsetFragment extends Fragment {
            public static SunsetFragment newInstance() {
                return new SunsetFragment();
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                    Bundle savedInstanceState) {
                View view = inflater.inflate(R.layout.fragment_sunset, container, false);

                return view;
            }
		}
		```

	+ 设置托管的`SunsetActivity`。 (`SunsetActivity.java`)

		```java
        public class SunsetActivity extends AppcompatActivity {

            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_fragment);

                FragmentManager fm = getSupportFragmentManager();
                Fragment fragment = fm.findFragmentById(R.id.fragment_container);

                if(fragment == null){
                    fragment = SunsetFragment.newInstance();
                    fm.beginTransaction()
                            .add(R.id.fragment_container, fragment)
                            .commit();
                }
            }
        }
		```

+ 简单的`Property`动画
	+ 获得所有视图的引用。(`SunsetFragment.java`)

		```java
        public class SunsetFragment extends Fragment {
            private View mSceneView;
            private View mSunView;
            private View mSkyView;

            public static SunsetFragment newInstance() {
                return new SunsetFragment();
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                   Bundle savedInstanceState) {
                View view = inflater.inflate(R.layout.fragment_sunset, container, false);

                mSceneView = view;
                mSunView = view.findViewById(R.id.sun);
                mSkyView = view.findViewById(R.id.sky);

                return view;
            }
        }
		```

	+ 将`mSunView`平滑移动至刚好被`sea`淹没。(即`mSunView`的顶部刚刚好与`sea`的顶部相切)。 方法是：将`mSunView`顶部的位置`translating`为其父视图的底部。
		+ 第一步，确定动画的起始位置。 新建`startAnimation()`方法。 (`SunsetFragment.java`)

			```java
            @Override
            public View onCreateView(...) {
                ...
            }

            private void startAnimation() {
                float sunYStart = mSunView.getTop();
                float sunYEnd = mSkyView.getHeight();
            }
			```

		+ 第二步，运行一个 `ObjectAnimator` 来演示动画。(`SunsetFragment.java`)

			```java
            private void startAnimation() {
                float sunYStart = mSunView.getTop();
                float sunYEnd = mSkyView.getHeight();

                ObjectAnimator heightAnimator = ObjectAnimator
                        .ofFloat(mSunView, "y", sunYStart, sunYEnd)
                        .setDuration(3000);

                heightAnimator.start();
            }
			```

		+ 最后为场景设置监听器，可触发 `startAnimation()`。 (`SunsetFragment.java`)

			```java
            public View onCreateView(...) {
                View view = inflater.inflate(R.layout.fragment_sunset, container, false);

                mSceneView = view;
                mSunView = view.findViewById(R.id.sun);
                mSkyView = view.findViewById(R.id.sky);

                mSceneView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        startAnimation();
                    }
                });

                return view;
            }
			```

+ 使用不同的 `interpolators`
	+ 使用一个`TimeInterpolator`来增加一个加速的效果。 `TimeInterpolator`作用是改变从`A`到`B`的动画的运行方式。使用`AccelerateInterpolator`来达成太阳开始下落时加速的效果。(`SunsetFragment.java`)

		```java
        private void startAnimation() {
            float sunYStart = mSunView.getTop();
            float sunYEnd = mSkyView.getHeight();

            ObjectAnimator heightAnimator = ObjectAnimator
                    .ofFloat(mSunView, "y", sunYStart, sunYEnd)
                    .setDuration(3000);
            heightAnimator.setInterpolator(new AccelerateInterpolator());

            heightAnimator.start();
        }
		```

+ 颜色估值
	+ 获取`Sunset`的颜色引用。 (`SunsetFragment.java`)

		```java
        ...
        private View mSkyView;

        private int mBlueSkyColor;
        private int mSunsetSkyColor;
        private int mNightSkyColor;
        ...
        public View onCreateView(...) {
            ...
            mSkyView = view.findViewById(R.id.sky);

			Resources resources = getResources();
            mBlueSkyColor = resources.getColor(R.color.blue_sky);
            mSunsetSkyColor = resources.getColor(R.color.sunset_sky);
            mNightSkyColor = resources.getColor(R.color.night_sky);

            mSceneView.setOnClickListener(new View.OnClickListener() {
                ...
            });

            return view;
        }
		```

	+ 在`startAnimation()`添加一个附属动画，使得`sky`的颜色从 `mBlueSkyColor` 变化为 `mSunsetSkyColor`. (`SunsetFragment.java`)

		```java
        private void startAnimation() {
            float sunYStart = mSunView.getTop();
            float sunYEnd = mSkyView.getHeight();

            ObjectAnimator heightAnimator = ObjectAnimator
                    .ofFloat(mSunView, "y", sunYStart, sunYEnd)
                    .setDuration(3000);
            heightAnimator.setInterpolator(new AccelerateInterpolator());

            ObjectAnimator sunsetSkyAnimator = ObjectAnimator
                    .ofInt(mSkyView, "backgroundColor", mBlueSkyColor, mSunsetSkyColor)
                    .setDuration(3000);

            heightAnimator.start();
            sunsetSkyAnimator.start();
        }
		```

	+ 一个`TypeEvaluator`能告诉`ObjectAnimator`变化过程中的具体值是什么，比如起始值和终端值之间的四分之一为多少。`TypeEvaluator`的子类`ArgbEvaluator`可以告诉`ObjectAnimator`颜色值的变化范围与具体值，使得颜色能平滑过渡。(`SunsetFragment.java`)

		```java
        private void startAnimation() {
            float sunYStart = mSunView.getTop();
            float sunYEnd = mSkyView.getHeight();

            ObjectAnimator heightAnimator = ObjectAnimator
                    .ofFloat(mSunView, "y", sunYStart, sunYEnd)
                    .setDuration(3000);
            heightAnimator.setInterpolator(new AccelerateInterpolator());

            ObjectAnimator sunsetSkyAnimator = ObjectAnimator
                    .ofInt(mSkyView, "backgroundColor", mBlueSkyColor, mSunsetSkyColor)
                    .setDuration(3000);
            sunsetSkyAnimator.setEvaluator(new ArgbEvaluator());

            heightAnimator.start();
            sunsetSkyAnimator.start();
        }
		```

+ 组合动画
	+ 使用`AnimatorSet`将动画组合播放。
	+ 第一步，添加黑夜动画。(`SunsetFragment.java`)

		```java
		private void startAnimation() {
			...
			sunsetSkyAnimator.setEvaluator(new ArgbEvaluator());

			ObjectAnimator nightSkyAnimator = ObjectAnimator
					.ofInt(mSkyView, "backgroundColor", mSunsetSkyColor, mNightSkyColor)
					.setDuration(1500);
			nightSkyAnimator.setEvaluator(new ArgbEvaluator());
		}
		```

	+ 第二步，创建并运行一个`AnimatorSet`. (`SunsetFragment.java`)

		```java
        private void startAnimation() {
            ...
            ObjectAnimator nightSkyAnimator = ObjectAnimator
                    .ofInt(mSkyView, "backgroundColor", mSunsetSkyColor, mNightSkyColor)
                    .setDuration(1500);
            nightSkyAnimator.setEvaluator(new ArgbEvaluator());

            AnimatorSet animatorSet = new AnimatorSet();
            animatorSet
                    .play(heightAnimator)
                    .with(sunsetSkyAnimator)
                    .before(nightSkyAnimator);
            animatorSet.start();
        }
		```














































































