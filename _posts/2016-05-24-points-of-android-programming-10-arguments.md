---
layout: post
title: "第10章 Arguments"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Arguments","CriminalIntent"]
description: "使用Fragment Arguments"
---
{% include JB/setup %}

#### 10.使用Fragment Arguments

+ 在一个`Fragment`中启动一个`Activity`的一般方法：

	```java
    public class CrimeListFragment extends Fragment {
        ...
        @Override
        public void onClick(View v) {
            Intent intent = new Intent(getActivity(), CrimeActivity.class);
            startActivity(intent);
        }
        ...
    }
	```

+ 添加extra：

	```java
    public class CrimeActivity extends SingleFragmentActivity {
        public static final String EXTRA_CRIME_ID = "extra_crime_id";

        public static Intent newIntent(Context packageContext, UUID crimeId) {
            // 在启动CrimeActivity的intent中添加crimeId
            Intent intent = new Intent(packageContext, CrimeActivity.class);
            intent.putExtra(EXTRA_CRIME_ID, crimeId);
            return intent;
        }
        ...
    }
	```

+ 新的启动方法：

	```java
	public class CrimeListFragment extends Fragment {
		...
		@Override
		public void onClick(View v) {
			Intent intent = CrimeActivity.newIntent(getActivity(), mCrime.getId());
			startActivity(intent);
		}
		...
	}
	```

+ `CrimeFragment`从`CrimeActivity`中取出extra：
	+ 简单一点，直接在`CrimeFragment`中获取`CrimeActivity`的`intent`，并从中取出`extra`，随之取出`Crime`，最后用`CrimeFragment`的视图展示`Crime`的数据：

		```java
		public class CrimeFragment extends Fragment {
			...
			public void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);

				UUID crimeId = (UUID) getActivity().getIntent()
									.getSerializableExtra(CrimeActivity.EXTRA_CRIME_ID);
				mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
			}
			...
		}
	```

	+ 复杂一点，使用`Fragment Arguments`。每个`fragment`实例都有一个相应的`Bundle`对象，其中可以包含`key-values`对，与`Activity`中`Intent`的`extra`类似。其中的每一对`key-pair`都称之为`argument`。创建方法：

		```java
		Bundle args = new Bundle();
		args.putSerializable(EXTRA_MY_OBJECT, myObject);
		args.putInt(EXTRA_MY_INT, myInt);
		args.putCharSequence(EXTRA_MY_STRING, myString);
		```

	+ **首先**，将`arguments bundle`附加到`fragment`上，可以调用`Fragment.setArguments(Bundle)`。附加时机在一个`fragment`被创建后，被添加给一个`activity`之前。为了处理好这个时机问题，一个常见的做法即是给`Fragment`添加一个`newInstance()`静态方法。该方法创建一个`fragment`实例，并且设置`fragment`的`arguments`：

		```java
		public class CrimeFragment extends Fragment {

			private static final String ARG_CRIME_ID = "crime_id";
			...
			public static CrimeFragment newInstance(UUID crimeId) {
				Bundle args = new Bundle();
				args.putSerializable(ARG_CRIME_ID, crimeId);

				CrimeFragment fragment = new CrimeFragment();
				fragment.setArguments(args);
				return fragment;
			}
			...
		}
	```

	+ **然后**，回到`CrimeActivity`，在`createFragment()`方法内部，从`CrimeActivity`的`intent`中取出`extra`，并传入`CrimeFragment.newInstance(UUID)`方法中，从而创建一个`CrimeFragment`实例：

		```java
		public class CrimeActivity extends SingleFragmentActivity {
			private static final String EXTRA_CRIME_ID = "extra_crime_id";
			...
			@Override
			protected Fragment createFragment() {
				UUID crimeId = (UUID) getIntent().getSerializableExtra(EXTRA_CRIME_ID);
				return CrimeFragment.newInstance(crimeId);
			}
			...
		}
		```

	+ **最后**，在`CrimeFragment`中调用`getArguments()`方法来取出`arguments`。

		```java
		public class CrimeFragment extends Fragment {

			private static final String ARG_CRIME_ID = "crime_id";
			...
			public static CrimeFragment newInstance(UUID crimeId) {
				...
			}
			@Override
			public void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);
				UUID crimeId = (UUID) getArguments().getSerializable(ARG_CRIME_ID);
				mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
			}
		}
		```

+ 重新载入显示列表项：
	+ 如果模型层保存的数据发生改变，应通知`RecyclerView`的`Adapter`，以便其即是获取最新数据并重新加载显示列表项。
	+ 当从`CrimeListFragment`启动`CrimeActivity`实例后，`CrimeActivity`被置于回退栈顶。这导致原先处于栈顶的`CrimeListActivity`实例被暂停(`pause`)并停止(`stop`)。
	+ 当用户按了`Back`按钮返回到列表时，`CrimeActivity`随即被弹出栈外并被销毁。与此同时，`CrimeListActivity`被重新启动(`start`)并恢复运行(`resume`)。
	+ 当`CrimeListActivity`恢复运行状态之后，操作系统会向它发出调用`onResume()`生命周期方法的指令。`CrimeListActivity`接到指令后，它的`FragmentManager`会调用当前被activity托管的fragment的`onResume()`方法。这里，CrimeListFragment即唯一的目标fragment。
	+ 一般来说，要保证fragment视图得到刷新，在`onResume()`方法内更新代码是最安全的选择。
在`CrimeListFragment`中，覆盖`onResume()`方法刷新显示列表项：

		```java
		public class CrimeListFragment extends Fragment {
		...
			@Override
			public View onCreateView(...) {
				...
			}

			@Override
			public void onResume() {
				super.onResume();
				updateUI();
			}

			private void updateUI() {
				CrimeLab crimeLab = CrimeLab.get(getActivity());
				List<Crime> crimes = crimeLab.getCrimes();

				if (mAdapter == null) {
					mAdapter = new CrimeAdapter(crimes);
					mCrimeRecyclerView.setAdapter(mAdapter);
				} else {
					mAdapter.notifyDataSetChanged();
				}j
			}
		...
		}
		```

+ 获取`Fragments`的`Results`。使用`Fragment.startActivityForResult(…)`与 `Fragment.onActivityResult(…)`方法来处理`Results`：

	```java
    public class CrimeListFragment extends Fragment {
        private static final int REQUEST_CRIME = 1;
        ...
        private class CrimeHolder extends RecyclerView.ViewHolder
                            implements View.OnClickListener {
        	...
            @Override
            public void onClick(View v) {
                Intent i = CrimeActivity.newIntent(getActivity(), mCrime.getId());
                startActivityForResult(i, REQUEST_CRIME);
            }
        }

        @Override
        public void onActivityResult(int requestCode, int resultCode, Intent data) {
            if (requestCode == REQUEST_CRIME) {
                // Handle result
            }
        }
        ...
    }
	```

+ `fragment`实例可以从`activity`实例中接受一个`result`，但是它没有自己的`result`。只有`activity`才有`result`。所以尽管`Fragment`有它自己的`startActivityForResult(…)`和 `onActivityResult(…)`方法，却没有任何`setResult(…)`方法。解决办法就是告诉托管`fragment`的`activity`返回一个值。比如：

	```java
    public class CrimeFragment extends Fragment {
        ...
        public void returnResult() {
            getActivity().setResult(Activity.RESULT_OK, null);
        }
    }
	```

+ 挑战练习：高效的`RecyclerView`重新载入。发现item改变的具体位置，并且重新载入相应的item。
