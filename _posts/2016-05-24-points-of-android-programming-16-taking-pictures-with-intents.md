---
layout: post
title: "第16章 使用Intent拍照"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Intent","CriminalIntent"]
description: "使用Intents拍照"
---
{% include JB/setup %}

#### 16.使用Intents拍照

+ 外部存储
	+ `Context`的基本文件与路径方法：

		| Method | Purpose |
		|---|---|
		|`File getFilesDir()`|获取`/data/data/<packagename>/files`目录|
		|`FileInputStream openFileInput(String name)`|打开现有文件进行读取|
		|`FileOutputStream openFileOutput(String name, int mode)`|打开文件进行写入，如不存在就创建它|
		|`File getDir(String name, int mode)`|获取`/data/data/<packagename>/`目录的子目录(如不存在就先创建它)|
		|`String[] fileList()`|获取`/data/data/<packagename>/files`目录下的文件列表。可与其他方法配合使用，例如`openFileInput(String)`|
		|`File getCacheDir()`|获取`/data/data/<packagename>/cache`目录。应注意及时清理该目录，并节约使用空间|

	+ `Context`的外部文件与路径方法：

		| Method | Purpose |
		|---|---|
		|`File getExternalCacheDir()`|Returns a handle to a cache folder in primary external storage. 与`getCacheDir()`类似|
		|`File[] getExternalCacheDirs()`|Returns cache folders for multiple external storage locations.|
		|`File getExternalFilesDir(String)`|Returns a handle to a folder on primary external storage in which to store regular files. |
		|`File[] getExternalFilesDirs(String)`|与`getExternalFilesDir(String)`一样|
		|`File[] getExternalMediaDirs()`|Returns handles to all the external folders Android makes available for storing media – pictures, movies, and music. |

+ 指明照片的存储地址
	+ 生成照片的文件名(`Crime.java`)

		```java
		...
		public void setSuspect(String suspect) {
			mSuspect = suspect;
		}

		public String getPhotoFilename() {
			return "IMG_" + getId().toString() + ".jpg";
		}
		```

	+ 生成照片文件的路径名(`CrimeLab.java`)

		```java
        public class CrimeLab {
            ...
            public Crime getCrime(UUID id) {
                ...
            }

            public File getPhotoFile(Crime crime) {
                File externalFilesDir = mContext
                        .getExternalFilesDir(Environment.DIRECTORY_PICTURES);

                // 确认是否有外部存储空间
                if (externalFilesDir == null) {
                    return null;
                }

                // 生成的是指向该位置的`File`对象
                return new File(externalFilesDir, crime.getPhotoFilename());
            }
            ...
        }
	```

	+ 获取照片文件的文件地址(`CrimeFragment.java`)

		```java
        ...
        private Crime mCrime;
        private File mPhotoFile;
        private EditText mTitleField;
        ...
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            UUID crimeId = (UUID) getArguments().getSerializable(ARG_CRIME_ID);
            mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);

            mPhotoFile = CrimeLab.get(getActivity()).getPhotoFile(mCrime);
        }
        ...
		```

+ 添加外部存储空间许可&声明特性(`AndroidManifest.xml`)

	```xml
    <manifest
        xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.bignerdranch.android.criminalintent" >
        <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
                         android:maxSdkVersion="18"/>
        <uses-feature android:name="android.hardware.camera"
                  	android:required="false"/>
        ...
	</manifest>
	```

+ 启动一个拍照intent (`CrimeFragment.java`)

	```java
    ...
    private static final int REQUEST_DATE = 0;
    private static final int REQUEST_CONTACT = 1;
    private static final int REQUEST_PHOTO= 2;
    ...
    @Override
    public View onCreateView(...) {
        ...
        mPhotoButton = (ImageButton) v.findViewById(R.id.crime_camera);
        final Intent captureImage = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

        boolean canTakePhoto = mPhotoFile != null &&
            captureImage.resolveActivity(packageManager) != null;
        // 不能拍照则禁用按钮
        mPhotoButton.setEnabled(canTakePhoto);

        if (canTakePhoto) {
            Uri uri = Uri.fromFile(mPhotoFile);
            captureImage.putExtra(MediaStore.EXTRA_OUTPUT, uri);
        }

        mPhotoButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivityForResult(captureImage, REQUEST_PHOTO);
            }
        });

        mPhotoView = (ImageView) v.findViewById(R.id.crime_photo);

        return v;
    }
	```

+ 按比例显示位图`Bitmaps`。当照片保存之后，需要取出，载入并将它显示在用户面前。需要将其载入成一个大小合理的`Bitmap`对象。使用`BitmapFactory`从一个文件中获得一个`Bitmap`(存储无压缩像素的一个简单对象)。需要将`bitmap`按比例缩小，先扫描出文件的具体大小，在决定缩小倍数，最后按缩小倍数重新获得一个`Bitmap`对象。
	+ 新建一个图片处理工具类(`PictureUtils.java`)

		```java
        public class PictureUtils {

            // 保守的比例方法
        	public static Bitmap getScaledBitmap(String path, Activity activity) {
                Point size = new Point();
                // 确认屏幕的大小
                activity.getWindowManager().getDefaultDisplay()
                        .getSize(size);

                return getScaledBitmap(path, size.x, size.y);
            }

            public static Bitmap getScaledBitmap(String path, int destWidth, int destHeight) {
                // Read in the dimensions of the image on disk
                BitmapFactory.Options options = new BitmapFactory.Options();
                options.inJustDecodeBounds = true;
                BitmapFactory.decodeFile(path, options);

                float srcWidth = options.outWidth;
                float srcHeight = options.outHeight;

                // Figure out how much to scale down by
                int inSampleSize = 1;
                if (srcHeight > destHeight || srcWidth > destWidth) {
                    if (srcWidth > srcHeight) {
                        inSampleSize = Math.round(srcHeight / destHeight);
                    } else {
                        inSampleSize = Math.round(srcWidth / destWidth);
                    }
                }

                options = new BitmapFactory.Options();
                options.inSampleSize = inSampleSize;

                // Read in and create final bitmap
                return BitmapFactory.decodeFile(path, options);
            }
        }
		```

	+ 将`Bitmap`添加到相应crime的`ImageView`视图中(`CrimeFragment.java`)

		```java
		...
        @Override
        public View onCreateView(...) {
            ...
            mPhotoButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    startActivityForResult(captureImage, REQUEST_PHOTO);
                }
            });
            mPhotoView = (ImageView) v.findViewById(R.id.crime_photo);

            updatePhotoView();

            return v;
        }

        @Override
        public void onActivityResult(int requestCode, int resultCode, Intent data) {
            if (resultCode != Activity.RESULT_OK) {
                return;
            }

            if (requestCode == REQUEST_DATE) {
                ...
            } else if (requestCode == REQUEST_CONTACT && data != null) {
                ...

            } else if (requestCode == REQUEST_PHOTO) {
                updatePhotoView();
            }
        }
        ...
        private String getCrimeReport() {
            ...
        }

        private void updatePhotoView() {
            if (mPhotoFile == null || !mPhotoFile.exists()) {
                mPhotoView.setImageDrawable(null);
            } else {
                Bitmap bitmap = PictureUtils.getScaledBitmap(
                        mPhotoFile.getPath(), getActivity());
                mPhotoView.setImageBitmap(bitmap);
            }
        }
        ...
		```

+ 挑战练习：点击图片后显示图片放大后的对话框；使用ViewTreeObserver实现高效图片载入









































































