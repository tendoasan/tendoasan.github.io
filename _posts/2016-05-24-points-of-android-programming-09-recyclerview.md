---
layout: post
title: "第09章 RecyclerView"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","RecyclerView","CriminalIntent"]
description: "使用布局与组件创建用户界面"
---
{% include JB/setup %}

#### 9.使用RecyclerView显示列表

+ 谨慎地使用单例，当Android从内存中移除应用时，单例即会消失。

+ `CrimeLab`单例不是数据长期存储的解决办法，但是它可以让App有一个数据的管理者，并且提供简单的方式使得数据在控制层之间传输。

+ RecyclerView和它的ViewHolders：

![]({{ IMAGE_PATH }}/RecyclerView和它的ViewHolders.jpg)

+ 典型的`RecyclerView-Adapter`对话:
	+ 首先，`RecyclerView`调用`adapter`的`getItemCount()`方法来获得列表中的对象个数。
	+ 然后，`RecyclerView`调用`adapter`的`createViewHolder(ViewGroup, int)`方法来创建一个`ViewHolder`，并传入一个要显示的`View`。
	+ 最后, `RecyclerView`调用`onBindViewHolder(ViewHolder, int)`方法，传入一个`ViewHolder`和`position`。`adapter`查询模型层数据中`position`位置的数据，并将其与 `ViewHolder`的`View`绑定。`adapter`用模型对象的数据相应地来填充`View`。 这一步骤完成后, `RecyclerView`将会在屏幕显示一个列表。

![]({{ IMAGE_PATH }}/RecyclerView-Adapter的对话.jpg)

+ 使用RecyclerView：
	+ `RecyclerView`自己不会创建`Views`，它总是通过创建`ViewHolders`来载入`itemViews`。`ViewHolder`填充`itemView`的不同部分来更加简单高效地展现一个`Crime`。
	+ `RecyclerView`的作用只是回收TextViews，通过一个`Adapter`子类和一个`ViewHolder`子类将它们放置在屏幕上，放置工作委托`LayoutManager`来完成。
	+ `LayoutManager` 管理items的位置，同时也定义了列表的的滚动方式。`LayoutManager` 对于`RecyclerView`是必须的。

+ 一个`adapter`是`RecyclerView`和`RecyclerView`将要展现的`data set`之间的一个控制器，它负责如下工作：
	+ 创建必要的`ViewHolders`
	+ 从绑定模型层的数据到`ViewHolders`

+ 创建一个`adapter`，首先需要定义一个`RecyclerView.Adapter`的子类，该adapter子类将会包裹(`wrap`)`CrimeLab`中的crimes列表。当`RecyclerView`需要展示一个视图对象时，它会与它的`adapter`展开一次对话。

+ 实现一个`Adapter`和`ViewHolder`：

	```java
	public class CrimeListFragment extends Fragment {
		...
		private class CrimeHolder extends RecyclerView.ViewHolder {
			private TextView mTitleTV;
			private TextView mDateTV;
			private CheckBox mSolvedCB;

			public CrimeHolder(View v) {
				super(v);

				mTitleTV = (TextView) v.findViewById(R.id.list_item_crime_title_tv);
				mDateTV = (TextView) v.findViewById(R.id.list_item_crime_date_tv);
				mSolvedCB = (CheckBox) v.findViewById(R.id.list_item_crime_solved_cb);
			}
		}

		private class CrimeAdapter extends RecyclerView.Adapter<CrimeHolder> {
			private List<Crime> mCrimes;

			public CrimeAdapter(List<Crime> crimes) {
				mCrimes = crimes;
			}

			@Override
			public CrimeHolder onCreateViewHolder(ViewGroup parent, int viewType) {
				LayoutInflater layoutInflater = LayoutInflater.from(getActivity());
				View view = layoutInflater.inflate(R.layout.list_item, parent, false);
				return new CrimeHolder(view);
			}

			@Override
			public void onBindViewHolder(CrimeHolder holder, int position) {
				Crime crime = mCrimes.get(position);
				holder.mTitleTV.setText(crime.getTitle());
				holder.mDateTV.setText(mCrime.getDate().toString());
				holder.mSolvedCB.setChecked(mCrime.isSolved());
			}

			@Override
			public int getItemCount() {
				return mCrimes.size();
			}
		}
	}
	```

	+ 当`RecyclerView`需要一个新的`View`来展示一个item时，调用`onCreateViewHolder()`方法。在这个方法中，新建了一个`View`，并将其传入了一个`ViewHolder`。
	+ 之后，调用`onBindViewHolder()`方法，将模型对象与`ViewHolder`的一个`View`绑定。通过`position`来找到模型对象的数据，并将数据映射到`View`中。

+ 连接`Adapter`和`RecyclerView`:

	```java
	public class CrimeListFragment extends Fragment {
		private RecyclerView mCrimeRV;
		private CrimeAdapter mAdapter;

		@Override
		public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
			View v = inflater.inflate(R.layout.fragment_crime_list, container, false);
			mCrimeRV = (RecyclerView) v.findViewById(R.id.crime_rv);
			mCrimeRV.setLayoutManager(new LinearLayoutManager(getActivity()));

			CrimeLab crimeLab = CrimeLab.get(getActivity());
			List<Crime> crimes = crimeLab.getCrimes();
			mAdapter = new CrimeAdapter(crimes);

			mCrimeRV.setAdapter(mAdapter);
		}
	}
	```