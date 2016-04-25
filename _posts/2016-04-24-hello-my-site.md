---
layout: post
title: What the Rice?
---
###Taking Pictures with Intents
1. A Place for Your Photo
	- Build out a place for your photo to live: an `ImageView` to display the photo and a `Button` to press to take a new photo. Camera view layout and Title layout (`res/layout/view_camera_and_title.xml`)
            <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="16dp"
                android:layout_marginRight="16dp">
                <LinearLayout
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:layout_marginRight="4dp">
                    <ImageView
                        android:id="@+id/crime_photo"
                        android:layout_width="80dp"
                        android:layout_height="80dp"
                        android:scaleType="centerInside"
                        android:background="@android:color/darker_gray"
                        android:cropToPadding="true"/>
                    <ImageButton
                        android:id="@+id/crime_camera"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:src="@android:drawable/ic_menu_camera"/>
                </LinearLayout>
                <LinearLayout
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:layout_weight="1">
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="@string/crime_title_label"
                        style="?android:listSeparatorTextViewStyle"/>
                    <EditText
                        android:id="@+id/crime_title"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginRight="16dp"
                        android:hint="@string/crime_title_hint"/>
                </LinearLayout>
            </LinearLayout>
	- Including camera layout (`portrait & landscape`)
	- Adding instance variables (`CrimeFragment.java`)
            ...
            private CheckBox mSolvedCheckbox;
            private Button mSuspectButton;
            **private ImageButton mPhotoButton;
            private ImageView mPhotoView;**
            ...
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                ...
                **mPhotoButton = (ImageButton) v.findViewById(R.id.crime_camera);
                mPhotoView = (ImageView) v.findViewById(R.id.crime_photo);**
                return v;
            }
            ...
2. External Storage
	- Basic file and directory methods in `Context`
| Method | Purpose |
|--------|--------|
|`File getFilesDir()`|Returns a handle to the directory for private application files.|
|`FileInputStream openFileInput(String name)`|Opens an existing file for input (relative to the files directory).|
|`FileOutputStream openFileOutput(String name, int mode)`|Opens a file for output, possibly creating it (relative to the files directory).|
|`File getDir(String name, int mode)`|Gets (and possibly creates) a subdirectory within the files directory.|
|`String[] fileList()`|Gets a list of file names in the main files directory, such as for use with openFileInput(String).|
|`File getCacheDir()`|Returns a handle to a directory you can use specifically for storing cache files. You should take care to keep this directory tidy and use as little space as possible.|

	- External file and directory methods in `Context`
| Method | Purpose |
|--------|--------|
|`File getExternalCacheDir()`|Returns a handle to a cache folder in primary external storage. Treat it like you do `getCacheDir()`, except a little more carefully. Android is even less likely to clean up this folder than the private storage one.|
|`File[] getExternalCacheDirs()`|Returns cache folders for multiple external storage locations.|
|`File getExternalFilesDir(String)`|Returns a handle to a folder on primary external storage in which to store regular files. If you pass in a type String, you can access a specific subfolder dedicated to a particular type of content. Type constants are defined in Environment, where they are prefixed with `DIRECTORY_`. For example, pictures go in `Environment.DIRECTORY_PICTURES`.|
|`File[] getExternalFilesDirs(String)`|Same as `getExternalFilesDir(String)`, but returns all possible file folders for the given type.|
|`File[] getExternalMediaDirs()`|Returns handles to all the external folders Android makes available for storing media – pictures, movies, and music. What makes this different from calling `getExternalFilesDir(Environment.DIRECTORY_PICTURES)` is that the media scanner automatically scans this folder. The media scanner makes files available to applications that play music, or browse movies and photos, so anything that you put in a folder returned by `getExternalMediaDirs()` will automatically appear in those apps.|

3. Designating a picture location
	- Adding `filename-derived` property (`Crime.java`)
            ...
            public void setSuspect(String suspect) {
                mSuspect = suspect;
            }
            **public String getPhotoFilename() {
                return "IMG_" + getId().toString() + ".jpg";
            }**
	- Finding photo file location (`CrimeLab.java`)
            ...
            public Crime getCrime(UUID id) {
                ...
            }
            **public File getPhotoFile(Crime crime) {
                File externalFilesDir = mContext
                        .getExternalFilesDir(Environment.DIRECTORY_PICTURES);
                if (externalFilesDir == null) {
                    return null;
                }
                return new File(externalFilesDir, crime.getPhotoFilename());
            }**
            ...
		- This code does not create any files on the filesystem. It only returns `File objects` that **point to the right locations**. It does perform one check: it verifies that there is external storage to save them to.
4. Using a Camera Intent
	- Grabbing photo file location (`CrimeFragment.java`)
            ...
            private Crime mCrime;
            **private File mPhotoFile;**
            private EditText mTitleField;
            ...
            @Override
            public void onCreate(Bundle savedInstanceState) {
                ...
                mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);
                **mPhotoFile = CrimeLab.get(getActivity()).getPhotoFile(mCrime);**
            }
            ...
	- The camera intent is defined in `MediaStore`, Android’s lord and master of all things media related. You will send an intent with an action of `MediaStore.ACTION_IMAGE_CAPTURE`, and Android will fire up a camera activity and take a picture for you.
5. External storage permission
	- Since `Context.getExternalFilesDir(String)` returns a folder that is specific to your app, it makes sense that you would want to be able to read and write files that live there. So on `Android 4.4 (API 19)` and up, you do not need this permission for this folder. (But you still need it for other kinds of external storage.) Requesting external storage permission (`AndroidManifest.xml`)
            <manifest xmlns:android="http://schemas.android.com/apk/res/android"
                package="com.bignerdranch.android.criminalintent" >
                <uses-permission
                    android:name="android.permission.READ_EXTERNAL_STORAGE"
                    android:maxSdkVersion="18"/>
                ...
6. Firing the intent
	- The action you want is called `ACTION_CAPTURE_IMAGE`, and it is defined in the `MediaStore` class. `MediaStore` defines the public interfaces used in Android for interacting with common media – images, videos, and music. This includes the `image capture intent`, which fires up the camera.
	- By default, `ACTION_CAPTURE_IMAGE` will dutifully fire up the camera application and take a picture, but it will not be a `full-resolution` picture. Instead, it will take a small resolution `thumbnail picture`, and stick the whole thing inside the `Intent` object returned in `onActivityResult(…)`.
	- For a full-resolution output, you need to tell it where to save the image on the filesystem. This can be done by passing a `Uri` pointing to where you want to save the file in `MediaStore.EXTRA_OUTPUT`.
	- Write an implicit intent to ask for a new picture to be taken into the location saved in `mPhotoFile`. Add code to ensure that the button is disabled if there is no camera app, or if there is no location at which to save the photo.
	- Firing a camera intent (`CrimeFragment.java`)
            ...
            private static final int REQUEST_DATE = 0;
            private static final int REQUEST_CONTACT = 1;
            **private static final int REQUEST_PHOTO= 2;**
            ...
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                ...
                mPhotoButton = (ImageButton) v.findViewById(R.id.crime_camera);
                **final Intent captureImage = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);**

                boolean canTakePhoto = mPhotoFile != null &&
                        captureImage.resolveActivity(packageManager) != null;
                mPhotoButton.setEnabled(canTakePhoto);

                if (canTakePhoto) {
                    Uri uri = Uri.fromFile(mPhotoFile);
                    **captureImage.putExtra(MediaStore.EXTRA_OUTPUT, uri);**
                }

                mPhotoButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        **startActivityForResult(captureImage, REQUEST_PHOTO);**
                    }
                });

                mPhotoView = (ImageView) v.findViewById(R.id.crime_photo);

                return v;
            }
7. Scaling and Displaying `Bitmaps`
	- With that, you are successfully taking pictures. Your image will be saved to a file on the filesystem for you to use. Your next step will be to take this file, load it up, and show it to the user. To do this, you need to load it into **a reasonably sized Bitmap object**. To get a `Bitmap` from a file, all you need to do is use the `BitmapFactory` class:
			Bitmap bitmap = BitmapFactory.decodeFile(mPhotoFile.getPath());
	- A `Bitmap` is a simple object that stores literal pixel data. That means that even if the original file was compressed, there is no compression in the `Bitmap` itself. So a 16 megapixel 24-bit camera image which might only be a 5 Mb JPG would blow up to 48 Mb loaded into a `Bitmap object (!)`.
	- You will need to `scale the bitmap down` by hand. You can do this by `first scanning the file to see how big it is`, `next figuring out how much you need to scale it` by to fit it in a given area, and `finally rereading the file to create a scaled-down Bitmap object`
	- Create a new class called `PictureUtils.java` for this new method, and add a static method to it called `getScaledBitmap(String, int, int)`.
            public class PictureUtils {
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
                        if (srcWidth > srcHeight) {inSampleSize = Math.round(srcHeight / destHeight);
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
        - When your fragment initially starts up, you will not know how big `PhotoView` is. Until a layout pass happens, views do not have `dimensions` on screen. The first layout pass happens after `onCreate(…)`, `onStart()`, and `onResume()` initially run, which is why `PhotoView` does not know how big it is.
        - Two solutions to this problem: **either you can wait until a layout pass happens, or you can use a conservative estimate.** The conservative estimate approach is less efficient, but more straightforward. Write another static method called `getScaledBitmap(String, Activity)` to scale a `Bitmap` for a particular `Activity’s` size.
                public class PictureUtils {
                    public static Bitmap getScaledBitmap(String path, Activity activity) {
                        **Point size = new Point();
                        activity.getWindowManager().getDefaultDisplay()
                                .getSize(size);**

                        return getScaledBitmap(path, size.x, size.y);
                    }
                    ...
        - This method checks to see how big the screen is, and then scales the image down to that size. The `ImageView` you load into will always be smaller than this size, so this is a very conservative estimate.
	- Next, to load this `Bitmap` into your `ImageView` add a method to `CrimeFragment` to update `mPhotoView`.
        - Updating `mPhotoView` (`CrimeFragment.java)`
                ...
                private String getCrimeReport() {
                    ...
                }
                **private void updatePhotoView() {
                    if (mPhotoFile == null || !mPhotoFile.exists()) {
                        mPhotoView.setImageDrawable(null);
                    } else {
                        Bitmap bitmap = PictureUtils.getScaledBitmap(
                                mPhotoFile.getPath(), getActivity());
                        mPhotoView.setImageBitmap(bitmap);
                    }
                }**
				...
		- Calling `updatePhotoView()` (`CrimeFragment.java`)
                    mPhotoButton.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            startActivityForResult(captureImage, REQUEST_PHOTO);
                        }
                    });
                    mPhotoView = (ImageView) v.findViewById(R.id.crime_photo);
                    **updatePhotoView();**
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
                        **updatePhotoView();**
                    }
                }
8. Declaring Features
	- When your app `uses a feature like the camera, or NFC, or any other feature` that may vary from device to device, it is strongly recommended that you `tell Android about it`. This allows other apps (like the Google Play store) to refuse to install your app if it uses a feature the device does not support. Adding uses-feature tag (`AndroidManifest.xml`)
            <?xml version="1.0" encoding="utf-8"?>
            <manifest xmlns:android="http://schemas.android.com/apk/res/android"
                package="com.bignerdranch.android.criminalintent" >
                <uses-permission
                	android:name="android.permission.READ_EXTERNAL_STORAGE"
                    android:maxSdkVersion="18"/>
                <uses-feature
                	android:name="android.hardware.camera"
                    android:required="false"/>
                ...
		- Passing in `android:required="false"` tells Android that your app can work fine without the camera, but that some parts will be disabled as a result.










