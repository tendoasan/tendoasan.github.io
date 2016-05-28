---
layout: post
title: "第15章 Implicit Intents"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Implicit","Intents","CriminalIntent"]
description: "隐式intent"
---
{% include JB/setup %}

#### 15.隐式intent

+ 使用格式化字符串
	+ 使用占位符(`strings.xml`)

		```xml
		<string name="crime_suspect_text">Choose Suspect</string>
		<string name="crime_report_text">Send Crime Report</string>
		<string name="crime_report">
			%1$s! The crime was discovered on %2$s. %3$s, and %4$s
		</string>
		<string name="crime_report_solved">The case is solved</string>
		<string name="crime_report_unsolved">The case is not solved</string>
		<string name="crime_report_no_suspect">there is no suspect.</string>
		<string name="crime_report_suspect">the suspect is %s.</string>
		<string name="crime_report_subject">CriminalIntent Crime Report</string>
		<string name="send_report">Send crime report via</string>
		```

	+ 与占位符对应的方法调用语句(`CrimeFragment.java`)

		```java
        ...
        private void updateDate() {
            mDateButton.setText(mCrime.getDate().toString());
        }

        private String getCrimeReport() {
            String solvedString = null;
            if (mCrime.isSolved()) {
                solvedString = getString(R.string.crime_report_solved);
            } else {
                solvedString = getString(R.string.crime_report_unsolved);
            }

            String dateFormat = "EEE, MMM dd";
            String dateString = DateFormat.format(dateFormat, mCrime.getDate()).toString();

            String suspect = mCrime.getSuspect();
            if (suspect == null) {
                suspect = getString(R.string.crime_report_no_suspect);
            } else {
            	// <string name="crime_report_suspect">the suspect is %s.</string>
                // 第二个参数即格式化字符串%s表示的内容
                suspect = getString(R.string.crime_report_suspect, suspect);
            }

			/* <string name="crime_report">
    				%1$s! The crime was discovered on %2$s. %3$s, and %4$s
  			</string>
            */
            String report = getString(R.string.crime_report,
                mCrime.getTitle(), dateString, solvedString, suspect);

            return report;
        }
		```

+ 使用隐式Intent
	+ 典型隐式intent的组成：
		+ 要执行的操作(`action`)。通常以`Intent`类中的常量来表示。例如，要访问查看某个`URL`，可以使用`Intent.ACTION_VIEW`；要发送邮件，可以使用`Intent.ACTION_SEND`。
		+ 要访问数据的位置(`location`)。这可能是设备以外的资源，如某个网页的`URL`，也可能是指向某个文件的`URI`，或者是指向`ContentProvider`中某条记录的某个**内容**URI（`content URI`）。
		+ 操作涉及的数据类型(`type`)。这指的是`MIME`形式的数据类型，如`text/html`或`audio/mpeg3`。如果一个`intent`包含某类数据的位置，那么通常可以从中推测出数据的类型。
		+ 可选类别(`categoties`)。如果操作用于描述具体要做什么，那么类别通常用来描述我们是何时、何地或者说如何使用某个activity的。Android的`android.intent.category.LAUNCHER`类别表明，activity应该显示在顶级应用启动器中。而`android.intent.category.INFO`类别表明，虽然activity向用户显示了包信息，但它不应该显示在启动器中。
		+ 一个用来查看某个网址的简单隐式intent会包括一个Intent.ACTION_VIEW操作，以及某个具体URL网址的uri数据。
		+ 基于以上信息，操作系统将启动适用应用的适用activity（如果有多个适用应用可选，用户可自行如何选择）。通过配置文件中的intent过滤器设置，activity会对外宣称自己是适合处理`ACTION_VIEW`的activity：

			```xml
			<activity
				android:name=".BrowserActivity"
				android:label="@string/app_name" >
				<intent-filter>
					<action android:name="android.intent.action.VIEW" />
					<category android:name="android.intent.category.DEFAULT" />
					<data android:scheme="http" android:host="www.tendoasan.com" />
				</intent-filter>
			</activity>
			```

	+ 发送一个crime报告(`CrimeFragment.java`)

		```java
        private Crime mCrime;
        private EditText mTitleField;
        private Button mDateButton;
        private CheckBox mSolvedCheckbox;
        private Button mReportButton;
        ...
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            ...
            mReportButton = (Button) v.findViewById(R.id.crime_report);
            mReportButton.setOnClickListener(new View.OnClickListener() {
                public void onClick(View v) {
                    Intent i = new Intent(Intent.ACTION_SEND);
                    i.setType("text/plain");
                    i.putExtra(Intent.EXTRA_TEXT, getCrimeReport());
                    i.putExtra(Intent.EXTRA_SUBJECT,
                    		getString(R.string.crime_report_subject));
                    // 使用选择器
                    i = Intent.createChooser(i, getString(R.string.send_report));
                    startActivity(i);
                }
            });
            return v;
        }
		```

	+ 获取联系人信息。创建另一个隐式intent，实现让用户从联系人应用里选择嫌疑人，操作为`Intent.ACTION_PICK`。联系人数据获取位置为`ContactsContract.Contacts.CONTENT_URI`。(`CrimeFragment.java`)

		```java
        ...
        private static final int REQUEST_DATE = 0;
        private static final int REQUEST_CONTACT = 1;
        ...
        private CheckBox mSolvedCheckbox;
        private Button mSuspectButton;
        ...
        @Override
        public View onCreateView(...) {
            ...
            final Intent pickContact = new Intent(Intent.ACTION_PICK,
                    ContactsContract.Contacts.CONTENT_URI);

            mSuspectButton = (Button) v.findViewById(R.id.crime_suspect);
            mSuspectButton.setOnClickListener(new View.OnClickListener() {
                public void onClick(View v) {
                    startActivityForResult(pickContact, REQUEST_CONTACT);
                }
            });

            if (mCrime.getSuspect() != null) {
                mSuspectButton.setText(mCrime.getSuspect());
            }

            // 调用PackageManager确认是否有联系人APP，没有则禁用按钮
            PackageManager packageManager = getActivity().getPackageManager();
            if (packageManager.resolveActivity(pickContact,
                    PackageManager.MATCH_DEFAULT_ONLY) == null) {
                mSuspectButton.setEnabled(false);
            }
            return v;
        }

        @Override
        public void onActivityResult(int requestCode, int resultCode, Intent data) {
            if (resultCode != Activity.RESULT_OK) {
                return;
            }

            if (requestCode == REQUEST_DATE) {
                ...
                updateDate();
            } else if (requestCode == REQUEST_CONTACT && data != null) {
                Uri contactUri = data.getData();
                // Specify which fields you want your query to return
                // values for.
                String[] queryFields = new String[] {
                        ContactsContract.Contacts.DISPLAY_NAME
                };
                // Perform your query - the contactUri is like a "where"
                // clause here
                Cursor c = getActivity().getContentResolver()
                        .query(contactUri, queryFields, null, null, null);

                try {
                    // 确认返回结果
                    if (c.getCount() == 0) {
                        return;
                    }

                    //取出第一行的第一列数据即是嫌疑人的名字
                    c.moveToFirst();
                    String suspect = c.getString(0);
                    mCrime.setSuspect(suspect);
                    mSuspectButton.setText(suspect);
                } finally {
                    c.close();
                }
            }
        }
		```

+ 挑战练习：ShareCompat，新建一个直接打嫌疑人电话的隐式Intent。
































































































