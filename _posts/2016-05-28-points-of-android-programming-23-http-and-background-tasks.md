---
layout: post
title: "第23章 HTTP与后台任务"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","HTTP","PhotoGallery","AsyncTask"]
description: "HTTP与后台任务"
---
{% include JB/setup %}

#### 23.HTTP与后台任务

+ 创建`PhotoGallery`项目。
	+ 建立Activity。(`PhotoGalleryActivity.java`)

		```java
        public class PhotoGalleryActivity extends SingleFragmentActivity {
            @Override
            public Fragment createFragment() {
                return PhotoGalleryFragment.newInstance();
            }
        }
		```

	+ `RecyclerView`布局。(`layout/fragment_photo_gallery.xml`)

		```xml
        <android.support.v7.widget.RecyclerView
            xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/fragment_photo_gallery_recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </android.support.v7.widget.RecyclerView>
		```

	+ Fragment的基本骨架。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            private RecyclerView mPhotoRecyclerView;

            public static PhotoGalleryFragment newInstance() {
                return new PhotoGalleryFragment();
            }

            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setRetainInstance(true);
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                    Bundle savedInstanceState) {
                View v = inflater.inflate(R.layout.fragment_photo_gallery, container, false);

                mPhotoRecyclerView = (RecyclerView) v
                    .findViewById(R.id.fragment_photo_gallery_recycler_view);
                mPhotoRecyclerView.setLayoutManager(new GridLayoutManager(getActivity(), 3));

                return v;
            }
        }
		```

+ 基本的网络连接。
	+ 处理网络连接的专用类，基本网络连接代码。(`FlickrFetchr.java`)

		```java
        public class FlickrFetchr {
        	// 从指定 URL 获取原始数据并返回一个字节流数组
            public byte[] getUrlBytes(String urlSpec) throws IOException {
                URL url = new URL(urlSpec);
                HttpURLConnection connection = (HttpURLConnection)url.openConnection();

                try {
                    ByteArrayOutputStream out = new ByteArrayOutputStream();
                    InputStream in = connection.getInputStream();

                    if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {
                        throw new IOException(connection.getResponseMessage() +
                                ": with " +
                                urlSpec);
                    }

                    int bytesRead = 0;
                    byte[] buffer = new byte[1024];
                    while ((bytesRead = in.read(buffer)) > 0) {
                        out.write(buffer, 0, bytesRead);
                    }
                    out.close();
                    return out.toByteArray();
                } finally {
                    connection.disconnect();
                }
            }
			// 将 getUrlBytes(String) 方法返回的结果转换为 String
            public String getUrlString(String urlSpec) throws IOException {
                return new String(getUrlBytes(urlSpec));
            }
        }
		```

        + 在`getUrlBytes(String)`方法中，根据传入的字符串参数，如`http://www.google.com`，首先创建一个URL对象。然后调用`openConnection()`方法创建一个指向要访问URL的`连接对象`。`URL.openConnection()`方法默认返回的是`URLConnection`对象，但我们要连接的是`http URL`，因此需将其强制类型转换为HttpURLConnection对象。随后，我们得以调用它的`getInputStream()`、`getResponseCode()`等方法。
        + `HttpURLConnection`对象虽然提供了一个连接，但只有在调用`getInputStream()`方法时（如果是POST请求，则调用`getOutputStream()`方法），它才会真正连接到指定的URL地址。在此之前我们无法获得有效的返回代码。
        + 一旦创建了URL并打开了网络连接，我们便可循环调用`read()`方法读取网络连接到的数据，直到取完为止。只要还有数据存在，InputStream类便可不断输出字节流数据。数据全部输出后，关闭网络连接，并将读取的数据写入`ByteArrayOutputStream`字节数组中。
	+ 添加网络使用权限（`AndroidManifest.xml`）

		```xml
        <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.bignerdranch.android.photogallery"
            android:versionCode="1"
            android:versionName="1.0" >

            <uses-sdk
                android:minSdkVersion="8"
                android:targetSdkVersion="15" />

            <uses-permission android:name="android.permission.INTERNET" />
            ...
        </manifest>
		```

+ 使用 `AsyncTask` 在后台线程上运行代码
	+ 调用并测试新添加的网络连接代码。因为Android禁止在主线程中发生任何网络连接行为，所以应创建一个后台线程，然后在该线程中运行代码。一般线程与主线程：

		![]({{ IMAGE_PATH }}/一般线程与主线程.jpg)

	+ 使用后台线程最简便的方式是使用`AsyncTask`工具类。AsyncTask创建后台线程后，我们便可在该线程上调用`doInBackground(...)`方法运行代码。
	+ 在`PhotoGalleryFragment.java`中，添加一个名为`FetchItemsTask`的内部类。覆盖`AsyncTask.doInBackground(...)`方法，从目标网站获取数据并记录下日志。

		```java
        public class PhotoGalleryFragment extends Fragment {
            private static final String TAG = "PhotoGalleryFragment";
            private RecyclerView mPhotoRecyclerView;
            ...
            private class FetchItemsTask extends AsyncTask<Void,Void,Void> {
                @Override
                protected Void doInBackground(Void... params) {
                    try {
                        String result = new FlickrFetchr().getUrl("http://www.google.com");
                        Log.i(TAG, "Fetched contents of URL: " + result);
                    } catch (IOException ioe) {
                        Log.e(TAG, "Failed to fetch URL: ", ioe);
                    }
                    return null;
                }
            }
        }
		```

	+ 在`PhotoGalleryFragment.onCreate(...)`方法中，调用`FetchItemsTask`新实例的`execute()`方法。

		```java
        public class PhotoGalleryFragment extends Fragment {
            private static final String TAG = "PhotoGalleryFragment";
            private RecyclerView mPhotoRecyclerView;

            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setRetainInstance(true);

                new FetchItemsTask().execute();
            }
            ...
        }
		```

+ 从Flickr获取`JSON`数据。
	+ 在网络连接类里面添加一些常量和`fetchItems()`方法。(`FlickrFetchr.java`)

		```java
        public class FlickrFetchr {
            private static final String TAG = "FlickrFetchr";
            private static final String API_KEY = "yourApiKeyHere";
            ...
            String getUrlString(String urlSpec) throws IOException {
                return new String(getUrlBytes(urlSpec));
            }

            public void fetchItems() {
                try {
                    String url = Uri.parse("https://api.flickr.com/services/rest/")
                            .buildUpon()
                            .appendQueryParameter("method", "flickr.photos.getRecent")
                            .appendQueryParameter("api_key", API_KEY)
                            .appendQueryParameter("format", "json")
                            .appendQueryParameter("nojsoncallback", "1")
                            .appendQueryParameter("extras", "url_s")
                            .build().toString();
                    String jsonString = getUrlString(url);
                    Log.i(TAG, "Received JSON: " + jsonString);
                } catch (IOException ioe) {
                    Log.e(TAG, "Failed to fetch items", ioe);
                }
            }
        }
		```

	+ 修改`PhotoGalleryFragment`的后台运行任务的内容。

		```java
        public class PhotoGalleryFragment extends Fragment {
            ...
            private class FetchItemsTask extends AsyncTask<Void,Void,Void> {
                  @Override
                  protected Void doInBackground(Void... params) {
                      new FlickrFetchr().fetchItems();
                      return null;
                }
            }
        }
		```

	+ 为解析`JSON`文本创建模型类`GalleryItem`。

		```java
        public class GalleryItem {
            private String mCaption;
            private String mId;
            private String mUrl;

            @Override
            public String toString() {
                return mCaption;
            }
            ...
        }
		```

+ 解析`JSON`文本。
	+ `JSON`文本的层次结构。

		![]({{ IMAGE_PATH }}/JSON的层次结构.jpg)

	+ 将`JSON`字符串转换成`JSONObject`。

		```java
        public class FlickrFetchr {
            private static final String TAG = "FlickrFetchr";
            ...
            public void fetchItems() {
                try {
                    ...
                    Log.i(TAG, "Received JSON: " + jsonString);

                    JSONObject jsonBody = new JSONObject(jsonString);

                } catch (JSONException je){
                    Log.e(TAG, "Failed to parse JSON", je);
                } catch (IOException ioe) {
                    Log.e(TAG, "Failed to fetch items", ioe);
                }
            }
        }
		```

	+ 从`JSONObject`解析出相应数据保存到`GalleryItem`的实例的相应成员变量中。

		```java
        public class FlickrFetchr {
            private static final String TAG = "FlickrFetchr";
            ...
            public void fetchItems() {
                ...
            }

            private void parseItems(List<GalleryItem> items, JSONObject jsonBody)
                    throws IOException, JSONException {
                // 解析出"photos"对象
                JSONObject photosJsonObject = jsonBody.getJSONObject("photos");
                // 解析出"photo"数组
                JSONArray photoJsonArray = photosJsonObject.getJSONArray("photo");

                for (int i = 0; i < photoJsonArray.length(); i++) {
                    JSONObject photoJsonObject = photoJsonArray.getJSONObject(i);

                    GalleryItem item = new GalleryItem();
                    item.setId(photoJsonObject.getString("id"));
                    item.setCaption(photoJsonObject.getString("title"));

                    if (!photoJsonObject.has("url_s")) {
                        continue;
                    }

                    item.setUrl(photoJsonObject.getString("url_s"));
                    items.add(item);
                }
            }
        }
		```

	+ 在`fetchItems()`方法中解析JSON文本。(`FlickrFetchr.java`)

		```java
        public void List<GalleryItem> fetchItems() {

            List<GalleryItem> items = new ArrayList<>();

            try {
                String url = ...;
                String jsonString = getUrlString(url);
                Log.i(TAG, "Received JSON: " + jsonString);
                JSONObject jsonBody = new JSONObject(jsonString);

                parseItems(items, jsonBody);

            } catch (JSONException je) {
                    Log.e(TAG, "Failed to parse JSON", je);
            } catch (IOException ioe) {
                Log.e(TAG, "Failed to fetch items", ioe);
            }
            // 提取到的数据存储在GalleryItem的实例中
            return items;
        }
		```

+ 从 `AsyncTask` 回到主线程。
	+ 首先，为托管`GalleryItem`定义`ViewHolder`(仅显示文本信息)。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {

            private static final String TAG = "PhotoGalleryFragment";
            ...
            private class PhotoHolder extends RecyclerView.ViewHolder {
                private TextView mTitleTextView;

                public PhotoHolder(View itemView) {
                    super(itemView);

                    mTitleTextView = (TextView) itemView;
                }

                public void bindGalleryItem(GalleryItem item) {
                    mTitleTextView.setText(item.toString());
                }
            }

            private class FetchItemsTask extends AsyncTask<Void,Void,Void> {
                ...
            }
        }
		```

	+ 然后，添加一个`RecyclerView.Adapter`，通过`GalleryItems`的一个列表来提供`PhotoHolders`。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {

            private static final String TAG = "PhotoGalleryFragment";
            ...
            private class PhotoHolder extends RecyclerView.ViewHolder {
                ...
            }

            private class PhotoAdapter extends RecyclerView.Adapter<PhotoHolder> {

                private List<GalleryItem> mGalleryItems;

                public PhotoAdapter(List<GalleryItem> galleryItems) {
                    mGalleryItems = galleryItems;
                }

                @Override
                public PhotoHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
                    TextView textView = new TextView(getActivity());
                    return new PhotoHolder(textView);
                }

                @Override
                public void onBindViewHolder(PhotoHolder photoHolder, int position) {
                    GalleryItem galleryItem = mGalleryItems.get(position);
                    photoHolder.bindGalleryItem(galleryItem);
                }

                @Override
                public int getItemCount() {
                    return mGalleryItems.size();
                }
            }
            ...
        }
		```

	+ 之后，设置`RecyclerView`的`Adapter`。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {

            private static final String TAG = "PhotoGalleryFragment";
            private RecyclerView mPhotoRecyclerView;

            private List<GalleryItem> mItems = new ArrayList<>();
            ...
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                View v = inflater.inflate(R.layout.fragment_photo_gallery, container, false);
                mPhotoRecyclerView = (RecyclerView) v
                    .findViewById(R.id.fragment_photo_gallery_recycler_view);
                mPhotoRecyclerView.setLayoutManager(new GridLayoutManager(getActivity(), 3));

                setupAdapter();

                return v;
            }

            private void setupAdapter() {
                if (isAdded()) {
                    mPhotoRecyclerView.setAdapter(new PhotoAdapter(mItems));
                }
            }
            ...
        }
		```

	+ 最后，修改`FetchItemsTask`异步任务。(`PhotoGalleryFragment.java`)

		```java
        ...
        private class FetchItemsTask extends AsyncTask<Void,Void, List<GalleryItem>> {
            @Override
            protected List<GalleryItem> doInBackground(Void... params) {
                return new FlickrFetchr().fetchItems();
            }

            @Override
            protected void onPostExecute(List<GalleryItem> items) {
                mItems = items;
                setupAdapter();
            }
        }
        ...
		```

		+ 首先，改变`FetchItemsTask`类第三个泛型参数的类型。该参数是`AsyncTask`返回结果的数据类型。它设置了`doInBackground(...)`方法返回结果的类型，以及`onPostExecute(...)`方法输入参数的数据类型。
		+ 其次，让`doInBackground(...)`方法返回`GalleryItem`类型的数据列表，将数组列表数据传递给`onPostExecute(...)`方法。
		+ 最后，添加`onPostExecute(...)`方法的实现代码。该方法接收从`doInBackground(...)`方法获取的列表数据，并将返回数据放入`mItems`变量，然后调用`setupAdapter()`方法更新`RecyclerView`视图的`adapter`。

+ 深入学习：再探 AsyncTask
	+ AsyncTask的另外两个类型参数。
	+ 第一个类型参数可指定输入参数的类型。可参考以下示例使用该参数：

		```java
        AsyncTask<String,Void,Void> task = new AsyncTask<String,Void,Void>() {
            public Void doInBackground(String... params) {
                for (String parameter : params) {
                    Log.i(TAG, "Received parameter: " + parameter);
                }
                return null;
            }
        };
        task.execute("First parameter", "Second parameter", "Etc.");
		```

		输入参数传入`execute(...)`方法（可接受一个或多个参数）。然后，这些变量参数再传递给`doInBackground(...)`方法。
	+ 第二个类型参数可指定发送进度更新需要的类型。以下为示例代码：

		```java
        final ProgressBar progressBar = /* A determinate progress bar */;
        progressBar.setMax(100);

        AsyncTask<Integer,Integer,Void> task = new AsyncTask<Integer,Integer,Void>() {
            public Void doInBackground(Integer... params) {
                for (Integer progress : params) {
                    publishProgress(progress);
                    Thread.sleep(1000);
                }
            }

            public void onProgressUpdate(Integer... params) {
                int progress = params[0];
                progressBar.setProgress(progress);
            }
        };
        task.execute(25, 50, 75, 100);
		```

		进度更新通常发生在执行的后台进程中。在后台进程中，我们无法完成必要的UI更新。因此AsyncTask提供了`publishProgress(...)和onProgressUpdate(...)`两个方法。
		+ 其工作方式如下：在后台线程中，我们从`doInBackground(...)`方法中调用`publishProgress(...)`方法。这样`onProgressUpdate(...)`方法便能够在UI线程上被调用。因此我们可在`onProgressUpdate(...)`方法中执行UI更新，但我们必须在`doInBackground(...)`方法中使用`publishProgress(...)`方法对它们进行管理控制。

+ 清理AsyncTask
	+ 在一些复杂的使用场景下，需将AsyncTask赋值给实例变量。一旦能够掌控它，就可以随时调用`AsyncTask.cancel(boolean)`方法，撤销运行中的AsyncTask。
	+ `AsyncTask.cancel(boolean)`方法有两种工作模式：粗暴的和温和的。如调用温和的`cancel(false)`方法，该方法会设置`isCancelled()`的状态为true。随后，AsyncTask会检查`doInBackground(...)`方法中的`isCancelled()状态，然后选择提前结束运行。
	+ 然而，如调用粗暴的`cancel(true)`方法，它会直接终止`doInBackground(...)`方法当前所在的线程。`AsyncTask.cancel(true)`方法停止AsyncTask的方式简单粗暴，如果可能，应尽量避免使用此种方式。

+ AsyncTask的替代：AsyncTaskLoader

+ 挑战练习：Gson，分页，动态改变图片显示的列数。

























