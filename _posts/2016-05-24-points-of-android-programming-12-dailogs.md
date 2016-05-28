---
layout: post
title: "第12章 Dialogs"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Dialogs","CriminalIntent"]
description: "对话框"
---
{% include JB/setup %}

#### 12.对话框

+ 添加AppCompat 库。

+ 要实现对话框显示，首先应完成以下任务：
	+ 创建`DatePickeFragment`类；
	+ 创建`AlertDialog`；
	+ 通过`FragmentManager`在屏幕上显示对话框。

+ 为了使用一个弹出对话框(`AlertDialog`)，最好的方法是将其包裹在`DialogFragment`的一个实例当中。

+ 使用`DialogFragment`的一个子类`DatePickerFragment`，来创建一个显示日期选择组件的弹出对话框：

	```java
    public class DatePickerFragment extends DialogFragment {
        @Override
        public Dialog onCreateDialog(Bundle savedInstanceState) {
            return new AlertDialog.Builder(getActivity())
                .setTitle(R.string.date_picker_title)
                .setPositiveButton(android.R.string.ok, null)
                .create();
        }
    }
	```

+ 要将`DialogFragment`添加给`FragmentManager`管理并放置到屏幕上，可调用`fragment`实例的`show(FragmentManager manager, String tag)`方法：

	```java
    public class CrimeFragment extends Fragment {
        private static final String ARG_CRIME_ID = "crime_id";
        private static final String DIALOG_DATE = "DialogDate";
        ...
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                        Bundle savedInstanceState) {
            ...
            mDateButton = (Button) v.findViewById(R.id.crime_date);
            mDateButton.setText(mCrime.getDate().toString());
            mDateButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    FragmentManager manager = getFragmentManager();
                    DatePickerFragment dialog = new DatePickerFragment();
                    dialog.show(manager, DIALOG_DATE);
                }
            });

            mSolvedCheckBox = (CheckBox) v.findViewById(R.id.crime_solved);
            ...
            return v;
        }
        ...
    }
	```

+ 设置对话框的显示内容
	+ 日期选择组件的布局(`layout/dialog_date.xml`)

		```xml
		<DatePicker
			xmlns:android="http://schemas.android.com/apk/res/android"
			android:id="@+id/dialog_date_date_picker"
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:calendarViewShown="false">
		</DatePicker>
		```

	+ 添加组件到弹出对话框(使用`public AlertDialog.Builder setView(View view);`方法)(`DatePickerFragment.java`)

		```java
		@Override
		public Dialog onCreateDialog(Bundle savedInstanceState) {
			View v = LayoutInflater.from(getActivity())
				.inflate(R.layout.dialog_date, null);

			return new AlertDialog.Builder(getActivity())
				.setView(v)
				.setTitle(R.string.date_picker_title)
				.setPositiveButton(android.R.string.ok, null)
				.create();
		}
		```

+ fragment之间的数据传递。
	+ 在`DatePickerFragment`中添加`newInstance(Date)`，即显示对话框的时候需要传入`mDate`。

		```java
		public class DatePickerFragment extends DialogFragment {
			private static final String ARG_DATE = "date";
			private DatePicker mDatePicker;

			public static DatePickerFragment newInstance(Date date) {
				Bundle args = new Bundle();
				args.putSerializable(ARG_DATE, date);

				DatePickerFragment fragment = new DatePickerFragment();
				fragment.setArguments(args);
				return fragment;
			}
			...
		}
		```

	+ 在`CrimeFragment`中，使用`DatePickerFragment.newInstance(Date)`方法来创建实例。

		```java
		@Override
		public View onCreateView(LayoutInflater inflater, ViewGroup parent,
				Bundle savedInstanceState) {
			...
			mDateButton = (Button)v.findViewById(R.id.crime_date);
			mDateButton.setOnClickListener(new View.OnClickListener() {
				public void onClick(View v) {
					FragmentManager fm = getActivity().getSupportFragmentManager();
					DatePickerFragment dialog = DatePickerFragment
									.newInstance(mCrime.getDate());
					dialog.show(fm, DIALOG_DATE);
				}
			});
			return v;
		}
		```

	+ 使用`Date`来初始化`DatePickerFragment`中的`DatePicker`：

		```java
		@Override
		public Dialog onCreateDialog(Bundle savedInstanceState) {
			Date date = (Date) getArguments().getSerializable(ARG_DATE);
			Calendar calendar = Calendar.getInstance();
			calendar.setTime(date);
			int year = calendar.get(Calendar.YEAR);
			int month = calendar.get(Calendar.MONTH);
			int day = calendar.get(Calendar.DAY_OF_MONTH);

			View v = LayoutInflater.from(getActivity())
							.inflate(R.layout.dialog_date, null);

			mDatePicker = (DatePicker)v.findViewById(R.id.dialog_date_date_picker);
			mDatePicker.init(year, month, day, null);

			return new AlertDialog.Builder(getActivity())
					.setView(v)
					.setTitle(R.string.date_picker_title)
					.setPositiveButton(android.R.string.ok, null)
					.create();
		}
		```

	+ 返回数据给`CrimeFragment`：
		+ 设置目标fragment。可将`CrimeFragment`设置成`DatePickerFragment`的目标 fragment。要建立这种关联，可调用以下Fragment方法：

			```java
			public void setTargetFragment(Fragment fragment, int requestCode);
			```

			该方法接受`目标fragment`以及一个类似于传入`startActivityForResult(...)`方法的`请求代码`作为参数。随后，`目标fragment`可使用该`请求代码`通知是哪一个fragment在返回数据信息。
			`目标fragment`以及`请求代码`由`FragmentManager`负责跟踪记录，我们可调用fragment（设置目标fragment的fragment）的`getTargetFragment()`和`getTargetRequestCode()`方法获取它们。
	+ 在`CrimeFragment.java`中，创建一个请求代码常量，然后将`CrimeFragment`设为`DatePickerFragment`实例的目标fragment：

		```java
        public class CrimeFragment extends Fragment {
            private static final String ARG_CRIME_ID = "crime_id";
            private static final String DIALOG_DATE = "DialogDate";

            private static final int REQUEST_DATE = 0;
            ...
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup parent, Bundle savedInstanceState) {
                ...
                mDateButton.setOnClickListener(new View.OnClickListener() {
                    public void onClick(View v) {
                        FragmentManager fm = getFragmentManager();
                        DatePickerFragment dialog = DatePickerFragment
                                        .newInstance(mCrime.getDate());
                        dialog.setTargetFragment(CrimeFragment.this, REQUEST_DATE);
                        dialog.show(manager, DIALOG_DATE);
                    }
                });
                return v;
            }
            ...
        }
		```

	+ 处理由同一activity托管的两个fragment间的数据返回时，可借用`Fragment.onActivityResult(...)`方法。因此，直接调用目标fragment的`Fragment.onActivityResult(...)`方法，即可实现数据的回传。该方法有我们需要的信息：
		+ 一个与传入`setTargetFragment(...)`方法相匹配的请求代码，用以告知目标fragment返回结果来自于哪里。
		+ 一个决定下一步该采取什么行动的结果代码。
		+ 一个含有extra数据信息的Intent。

	+ 在`DatePickerFragment`类中，新建一个`sendResult(...)`私有方法。通过该方法，创建一个intent，将日期数据作为extra附加到intent上。最后调用`CrimeFragment.onActivityResult(...)方法`。在`onCreateDialog(...)`方法中，取代`setPositiveButton(...)`的null参数，实现一个`DialogInterface.OnClickListener`监听器接口。然后在监听器接口的onClick(...)方法中，调用新建的`sendResult(...)`私有方法并传入结果代码：

		```java
		public class DatePickerFragment extends DialogFragment {

			public static final String EXTRA_DATE = "extra.date";

			private static final String ARG_DATE = "date";
			...
			@Override
			public Dialog onCreateDialog(Bundle savedInstanceState) {
				...
				return new AlertDialog.Builder(getActivity())
					.setView(v)
					.setTitle(R.string.date_picker_title)
					.setPositiveButton(android.R.string.ok,
						new DialogInterface.OnClickListener() {
							@Override
							public void onClick(DialogInterface dialog, int which) {
								int year = mDatePicker.getYear();
								int month = mDatePicker.getMonth();
								int day = mDatePicker.getDayOfMonth();
								Date date = new GregorianCalendar(year, month, day).getTime();
								sendResult(Activity.RESULT_OK, date);
							}
				})
				.create();
			}

			private void sendResult(int resultCode, Date date) {
				if (getTargetFragment() == null) {
					return;
				}

				Intent intent = new Intent();
				intent.putExtra(EXTRA_DATE, date);

				getTargetFragment()
						.onActivityResult(getTargetRequestCode(), resultCode, intent);
			}
		}
		```

	+ 响应DatePicker对话框。在`CrimeFragment`中，覆盖`onActivityResult(...)`方法，从`extra`中获取日期数据，设置对应`Crime`的记录日期，然后刷新日期按钮的显示。

		```java
		public class CrimeFragment extends Fragment {
			...
			@Override
			public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
				View v = inflater.inflate(R.layout.fragment_crime, container, false);
				...
				mDateButton = (Button) v.findViewById(R.id.crime_date);
				updateDate();
				...
			}

			@Override
			public void onActivityResult(int requestCode, int resultCode, Intent data) {
				if (resultCode != Activity.RESULT_OK) {
					return;
				}

				if (requestCode == REQUEST_DATE) {
					Date date = (Date) data.getSerializableExtra(DatePickerFragment.EXTRA_DATE);
					mCrime.setDate(date);
					updateDate();
				}
			}

			private void updateDate() {
				mDateButton.setText(mCrime.getDate().toString());
			}
		}
		```