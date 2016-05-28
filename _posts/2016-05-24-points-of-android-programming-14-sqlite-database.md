---
layout: post
title: "第14章 SQLite Database"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","SQLite","Database",CriminalIntent"]
description: "使用SQLite 本地数据库"
---
{% include JB/setup %}

#### 14.使用SQLite 本地数据库

+ 定义一个Schema(`CrimeDbSchema.java`)

	```java
	public class CrimeDbSchema {
        public static final class CrimeTable {
			public static final String NAME = "crimes";

			public static final class Cols {
				public static final String UUID = "uuid";
				public static final String TITLE = "title";
                public static final String DATE = "date";
                public static final String SOLVED = "solved";
            }
        }
    }
	```

+ 建立初始化本地数据库
	+ 创建一个`SQLiteOpenHelper`的子类`CrimeBaseHelper`来操作一个数据库:

		```java
        public class CrimeBaseHelper extends SQLiteOpenHelper {
            private static final int VERSION = 1;
            private static final String DATABASE_NAME = "crimeBase.db";

            public CrimeBaseHelper(Context context) {
                super(context, DATABASE_NAME, null, VERSION);
            }

            @Override
            public void onCreate(SQLiteDatabase db) {
            }

            @Override
            public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            }
        }
		```

	+ 导入`CrimeDbSchema.CrimeTable`，为本地数据库建表：(CrimeBaseHelper.java)

		```java
        @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("create table " + CrimeTable.NAME + "(" +
                    " _id integer primary key autoincrement, " +
                    CrimeTable.Cols.UUID + ", " +
                    CrimeTable.Cols.TITLE + ", " +
                    CrimeTable.Cols.DATE + ", " +
                    CrimeTable.Cols.SOLVED +
                    ")"
            );
        }
		```

	+ 修改`CrimeLab`，使用数据库管理数据(CrimeLab.java)

		```java
        public class CrimeLab {
            private static CrimeLab sCrimeLab;

            private Context mContext;
            private SQLiteDatabase mDatabase;

            public static CrimeLab get(Context context) {
                ...
            }

            private CrimeLab(Context context) {
                mContext = context.getApplicationContext();
                mDatabase = new CrimeBaseHelper(mContext)
                        .getWritableDatabase();
            }

			// 读取：使用CursorWrapper读取所有Crimes
			public List<Crime> getCrimes() {
                List<Crime> crimes = new ArrayList<>();
				CrimeCursorWrapper cursor = queryCrimes(null, null);
				try {
        			cursor.moveToFirst();
        			while (!cursor.isAfterLast()) {
            			crimes.add(cursor.getCrime());
            			cursor.moveToNext();
        			}
    			} finally {
        			cursor.close();
    			}
    			return crimes;
			}

			// 读取：使用CursorWrapper读取特定Crime
            public Crime getCrime(UUID id) {
                CrimeCursorWrapper cursor = queryCrimes(
            		CrimeTable.Cols.UUID + " = ?",
            		new String[] { id.toString() }
    			);

    			try {
        			if (cursor.getCount() == 0) {
            			return null;
        			}
        			cursor.moveToFirst();
        			return cursor.getCrime();
    			} finally {
        			cursor.close();
    			}
            }

			// 写入：插入行
            public void addCrime(Crime c) {
            	ContentValues values = getContentValues(c);
    			mDatabase.insert(CrimeTable.NAME, null, values);
            }

			// 写入：更新行
            public void updateCrime(Crime crime) {
                String uuidString = crime.getId().toString();
                ContentValues values = getContentValues(crime);

                mDatabase.update(CrimeTable.NAME, values,
                        CrimeTable.Cols.UUID + " = ?",
                        new String[] { uuidString });
			}

			// 使用ContentValues方便写入数据库
            private static ContentValues getContentValues(Crime crime) {
                ContentValues values = new ContentValues();
                values.put(CrimeTable.Cols.UUID, crime.getId().toString());
                values.put(CrimeTable.Cols.TITLE, crime.getTitle());
                values.put(CrimeTable.Cols.DATE, crime.getDate().getTime());
                values.put(CrimeTable.Cols.SOLVED, crime.isSolved() ? 1 : 0);

                return values;
        	}

			// 使用Cursor方便读取数据库
            private CrimeCursorWrapper queryCrimes(String whereClause, String[] whereArgs) {
    			Cursor cursor = mDatabase.query(
            		CrimeTable.NAME,
            		null, // Columns - null selects all columns
            		whereClause,
            		whereArgs,
            		null, // groupBy
            		null, // having
            		null  // orderBy
    			);
    			return new CrimeCursorWrapper(cursor);
			}
        }
		```

	+ 使用`Cursor`从数据库中读取数据：

		```java
        String uuidString = cursor.getString(cursor.getColumnIndex(CrimeTable.Cols.UUID));
        String title = cursor.getString(cursor.getColumnIndex(CrimeTable.Cols.TITLE));
        long date = cursor.getLong(cursor.getColumnIndex(CrimeTable.Cols.DATE));
        int isSolved = cursor.getInt(cursor.getColumnIndex(CrimeTable.Cols.SOLVED));
		```

	+ 使用`CursorWrapper`的子类封装`Cursor`，方便添加新方法(`CrimeCursorWrapper.java`)。

		```java
		public class CrimeCursorWrapper extends CursorWrapper {
			public CrimeCursorWrapper(Cursor cursor) {
				super(cursor);
			}

			public Crime getCrime() {
				String uuidString = getString(getColumnIndex(CrimeTable.Cols.UUID));
				String title = getString(getColumnIndex(CrimeTable.Cols.TITLE));
				long date = getLong(getColumnIndex(CrimeTable.Cols.DATE));
				int isSolved = getInt(getColumnIndex(CrimeTable.Cols.SOLVED));

				// Crime的构造器`public Crime(UUID id){};`
				Crime crime = new Crime(UUID.fromString(uuidString));
				crime.setTitle(title);
				crime.setDate(new Date(date));
				crime.setSolved(isSolved != 0);

				return crime;;
			}
		}
		```

	+ 在`CrimeFragment`中刷新`CrimeLab`：

		```java
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            UUID crimeId = (UUID) getArguments().getSerializable(ARG_CRIME_ID);
            mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
        }

        @Override
        public void onPause() {
            super.onPause();

			// 即修改完成后，离开fragment时更新修改了的crime
            CrimeLab.get(getActivity()).updateCrime(mCrime);
        }
		```

	+ 在`CrimeListFragment`中刷新模型数据，即是在`CrimeAdapter`中刷新`Crimes`：

		```java
		...
		private class CrimeAdapter extends RecyclerView.Adapter<CrimeHolder> {
			...
			@Override
			public int getItemCount() {
				return mCrimes.size();
			}

			// // 刷新Adapter中的模型数据
			public void setCrimes(List<Crime> crimes) {
				mCrimes = crimes;
			}
		}

		// public方便双版面布局中CrimeFragment发生修改后能及时调用该方法刷新
		public void updateUI() {
			CrimeLab crimeLab = CrimeLab.get(getActivity());
			List<Crime> crimes = crimeLab.getCrimes();

			if (mAdapter == null) {
				mAdapter = new CrimeAdapter(crimes);
				mCrimeRecyclerView.setAdapter(mAdapter);
			} else {
				// 刷新Adapter中的模型数据
				mAdapter.setCrimes(crimes);
				mAdapter.notifyDataSetChanged();
			}
			updateSubtitle();
		}
		...
```

+ The Application Context

+ 挑战练习：删除Crimes，



























