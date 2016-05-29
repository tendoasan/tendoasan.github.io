---
layout: post
title: "第24章 Looper与Handler"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Looper","PhotoGallery","Handler"]
description: "Looper, Handler与HandlerThread"
---
{% include JB/setup %}

#### 24.Loopers, Handlers 与 HandlerThread

+ 设置 RecyclerView 以显示图片
	+ 首先，为gallery图片项创建一个名为`gallery_item.xml`的布局文件。该布局将包含一个`ImageView`组件。(`res/layout/gallery_item.xml`)

		```xml
        <ImageView xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/fragment_photo_gallery_image_view"
            android:layout_width="match_parent"
            android:layout_height="120dp"
            android:layout_gravity="center"
            android:scaleType="centerCrop">
        </ImageView>
		```

	+ 更新`PhotoHolder`和`PhotoAdapter`。(`PhotoGalleryFragment.java`)

		```java
        ...
        private class PhotoHolder extends RecyclerView.ViewHolder {
            private TextView mTitleTextView ImageView mItemImageView;

            public PhotoHolder(View itemView) {
                super(itemView);
                mItemImageView = (ImageView) itemView
                        .findViewById(R.id.fragment_photo_gallery_image_view);
            }

            public void bindDrawable(Drawable drawable) {
                mItemImageView.setImageDrawable(drawable);
            }
        }
        ...
        private class PhotoAdapter extends RecyclerView.Adapter<PhotoHolder> {
        	...
            @Override
            public PhotoHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
                LayoutInflater inflater = LayoutInflater.from(getActivity());
                View view = inflater.inflate(R.layout.gallery_item, viewGroup, false);
                return new PhotoHolder(view);
            }
            ...
            @Override
            public void onBindViewHolder(PhotoHolder photoHolder, int position) {
              	GalleryItem galleryItem = mGalleryItems.get(position);
                // 默认图片
              	Drawable placeholder = getResources()
                		.getDrawable(R.drawable.bill_up_close);
              	photoHolder.bindDrawable(placeholder);
            }
            ...
        }
		...
		```

+ 批量下载缩略图。
	+ 一次性下载全部缩略图存在两个问题。首先，下载比较耗时，而且在下载完成前，UI都无法完成更新；其次，缩略图的保存也是个问题。
	+ 由于此类问题的存在，实际开发的应用通常会选择仅在需要显示图片时才去下载。显然，RecyclerView及其adapter应负责实现按需下载。
	+ AsyncTask是获得后台线程的最简单方式，但它基本上不适用于重复且长时间运行的任务。
	+ 为代替AsyncTask的使用，创建一个专用的后台线程。这是实现按需下载的最常用方式。

+ 与主线程通信。
	+ Android系统中，线程使用的收件箱叫做消息队列（`message queue`）。使用消息队列的线程叫做消息循环（`message loop`）。消息循环会不断循环检查队列上是否有新消息。
	+ 消息循环由一个线程和一个`looper`组成。`Looper`对象管理着线程的消息队列。
	+ 主线程也是一个消息循环，因此具有一个`looper`。主线程的所有工作都是由其`looper`完成的。`looper`不断从消息队列中抓取消息，然后完成消息指定的任务。
	+ 创建一个同样是消息循环的后台线程。使用一个HandlerThread类准备需要的looper。

+ 创建并启动后台线程。
	+ 继承`HandlerThread`类，创建一个名为`ThumbnailDownloader`的新类。`ThumbnailDownloader`类需要使用某些对象来标识每一次下载。因此，在类创建对话框，通过`ThumbnailDownloader<T>`的命名，为其提供一个`T`泛型参数。然后，再添加一个构造方法以及一个名为`queueThumbnail()`的存根方法。初始线程代码（`ThumbnailDownloader.java`）

		```java
        public class ThumbnailDownloader<T> extends HandlerThread {
            private static final String TAG = "ThumbnailDownloader";

            public ThumbnailDownloader() {
                super(TAG);
            }

            public void queueThumbnail(T target, String url) {
                Log.i(TAG, "Got an URL: " + url);
            }
        }
		```

	+ 在`PhotoGalleryFragment`中，添加`ThumbnailDownloader`类型的成员变量，使用`PhotoHolder`作为泛型参数，该`Holder`是下载图片最终要显示的地方。然后，在`onCreate(...)`方法中，创建并启动线程。最后，覆盖`onDestroy()`方法退出线程。

		```java
        public class PhotoGalleryFragment extends Fragment {
            private static final String TAG = "PhotoGalleryFragment";
            private RecyclerView mPhotoRecyclerView;
            private List<GalleryItem> mItems = new ArrayList<>();

            private ThumbnailDownloader<PhotoHolder> mThumbnailDownloader;
            ...
            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setRetainInstance(true);
                new FetchItemsTask().execute();

                mThumbnailDownloader = new ThumbnailDownloader<>();
                mThumbnailDownloader.start();
                mThumbnailDownloader.getLooper();
                Log.i(TAG, "Background thread started");
            }

            @Override
            public View onCreateView(...) {
                ...
            }

            @Override
            public void onDestroy() {
                super.onDestroy();
                mThumbnailDownloader.quit();
                Log.i(TAG, "Background thread destroyed");
            }
            ...
        }
		```

		+ 在`ThumbnailDownloader`线程上，`getLooper()`方法是在`start()`方法之后调用的。这是一种保证线程就绪的处理方式。
		+ 结束线程的`quit()`方法是在`onDestroy()`方法内完成调用的。这非常关键。如不终止`HandlerThread`，它会一直运行下去。
	+ 在`GalleryItemAdapter.onBindViewHolder`方法中，使用`position`参数定位获取正确的`GalleryItem`，然后调用线程的`queueThumbnail()`方法，并传入`ImageView`和`gallery`图片项的`URL`。

		```java
        public class PhotoGalleryFragment extends Fragment {
          ...
          private class PhotoAdapter extends RecyclerView.Adapter<PhotoHolder> {
            ...
            @Override
            public void onBindViewHolder(PhotoHolder photoHolder, int position) {
                GalleryItem galleryItem = mGalleryItems.get(position);
                Drawable placeholder = getResources().getDrawable(R.drawable.bill_up_close);
                photoHolder.bindDrawable(placeholder);

                mThumbnailDownloader.queueThumbnail(photoHolder, galleryItem.getUrl());
            }
            ...
          }
          ...
        }
		```

+ Message 与 message Handler。
	+ 消息是Message类的一个实例，包含有好几个实例变量。其中有三个需在实现时定义：
		+ `what` 用户定义的int型消息代码，用来描述消息；
		+ `obj` 随消息发送的用户指定对象；
		+ `target` 处理消息的Handler。
		Message的目标是Handler类的一个实例。Handler可看作是“message handler”的简称。Message在创建时，会自动与一个Handler相关联。Message在准备处理状态下，Handler是负责让消息处理行为发生的对象。
	+ Handler。要处理消息以及消息指定的任务，首先需要一个消息Handler实例。Handler不仅仅是处理Message的目标（target），也是创建和发布Message的接口。
		+ Looper拥有Message对象的收件箱，所以Message必须在Looper上发布或读取。基于Looper和Message的这种关系，为与Looper协同工作，Handler总是引用着它。
		+ 一个Handler仅与一个Looper相关联，一个Message也仅与一个目标Handler（也称作Message目标）相关联。Looper拥有着整个Message队列。

			![]({{ IMAGE_PATH }}/Looper,Handler,HandlerThread与Message.jpg)

		+ 多个Handler可与一个Looper相关联。这意味着一个Handler的Message可能与另一个Handler的Message存放在同一消息队列中。

			![]({{ IMAGE_PATH }}/多个Handler对应一个Looper.jpg)

+ 使用Handler。
	+ 消息的目标Handler通常不需要手动设置。一个比较理想的方式是，调用`Handler.obtainMessage(...)`方法创建信息并传入其他消息字段，然后该方法自动完成目标Handler的设置。
	+ 为避免创建新的Message对象，`Handler.obtainMessage(...)`方法会从公共循环池里获取消息。因此相比创建新实例，这样有效率多了。
	+ 一旦取得Message，就调用`sendToTarget()`方法将其发送给它的Handler。紧接着Handler会将Message放置在Looper消息队列的尾部。

		![]({{ IMAGE_PATH }}/创建并发送Message.jpg)

	+ `PhotoGallery`应用中，在`queueThumbnail()`实现方法中获取并发送消息给它的目标。消息的`what`属性是一个定义为`MESSAGE_DOWNLOAD`的常量。消息的`obj`属性是一个`T`，这里指由`adapter`传入`queueThumbnail()`方法的`PhotoHolder`。
	+ Looper取得消息队列中的特定消息后，会将它发送给消息目标去处理。消息一般是在目标的`Handler.handleMessage(...)`实现方法中进行处理的。
	+ 这里，`handleMessage(...)`实现方法将使用`FlickrFetchr`从`URL`下载图片字节，然后再转换为位图。
	+ 获取、发送以及处理消息。(ThumbnailDownloader.java)

		```java
        public class ThumbnailDownloader<T> extends HandlerThread {
            private static final String TAG = "ThumbnailDownloader";
            private static final int MESSAGE_DOWNLOAD = 0;
            private Handler mRequestHandler;
            private ConcurrentMap<T,String> mRequestMap = new ConcurrentHashMap<>();

            public ThumbnailDownloader(Handler responseHandler) {
                super(TAG);
            }

            @Override
            protected void onLooperPrepared() {
                mRequestHandler = new Handler() {
                    @Override
                    public void handleMessage(Message msg) {
                        if (msg.what == MESSAGE_DOWNLOAD) {
                            T target = (T) msg.obj;
                            Log.i(TAG, "Got a request for URL: " + mRequestMap.get(target));
                            handleRequest(target);
                        }
                    }
                };
            }

            public void queueThumbnail(T target, String url) {
                Log.i(TAG, "Got a URL: " + url);

                if (url == null) {
                    mRequestMap.remove(target);
                } else {
                    mRequestMap.put(target, url);
                    mRequestHandler.obtainMessage(MESSAGE_DOWNLOAD, target)
                            .sendToTarget();
                }
            }

            private void handleRequest(final T target) {
                try {
                    final String url = mRequestMap.get(target);

                    if (url == null) {
                        return;
                    }

                    byte[] bitmapBytes = new FlickrFetchr().getUrlBytes(url);
                    final Bitmap bitmap = BitmapFactory
                            .decodeByteArray(bitmapBytes, 0, bitmapBytes.length);
                    Log.i(TAG, "Bitmap created");

                } catch (IOException ioe) {
                    Log.e(TAG, "Error downloading image", ioe);
                }
            }
        }
		```

		+ `MESSAGE_DOWNLOAD`用来标识`下载请求`的消息。
		+ `mRequestHandler`将持有`Handler`的一个引用，负责在后台线程添加并处理`下载请求`的消息。
		+ `mRequsetMap`是一个`ConcurrentHashMap`的实例，后者是一个`线程安全`版的`HashMap`。这里使用下载请求的标识对象Target作为键，可以利用这个存储和取出与具体请求相关的`URL`。
		+ 在`queueThumbnail()`方法中，将传入的`Target-URL`键值对放入map中。然后以Target为obj获取一条消息，并发送出去以存放到消息队列中。
		+ 在`onLooperPrepared()`方法内，我们在Handler子类中实现了`Handler.handleMessage(...)`方法。因为`HandlerThread.onLooperPrepared()`方法的调用发生在Looper第一次检查消息队列之前，所以该方法成了我们创建Handler实现的好地方。
		+ 在`Handler.handleMessage(...)`方法中，检查消息类型，获取`Target`，然后将其传递给`handleRequest(...)`方法，这个方法作为一个帮助方法来处理下载动作。
		+ `handleMessage(...)`方法是下载动作发生的地方。这里，确认URL已存在后，将它传递给`FlickrFetchr.getUrlBytes(...)`方法。
		+ 最后，使用`BitmapFactory`将`getUrlBytes(...)`返回的字节数组转换为位图。

+ 传递Handler
	+ HandlerThread能在主线程上完成任务的一种方式是，让主线程将其自身的Handler传递给HandlerThread。
	+ 主线程是一个拥有handler和Looper的消息循环。主线程上创建的Handler会自动与它的Looper相关联。可以将主线程上创建的Handler传递给另一线程。传递出去的Handler与创建它的线程Looper始终保持着联系。因此，任何已传出的Handler负责处理的消息都将在主线程的消息队列中处理。
	+ 这看上去像在使用ThumbnailDownloader的Handler，实现在主线程上安排后台线程上的任务。

		![]({{ IMAGE_PATH }}/从主线程安排ThumbnailDownloader上的任务.jpg)

	+ 反过来，也可从后台线程使用与主线程关联的Handler，安排要在主线程上完成的任务。

		![]({{ IMAGE_PATH }}/从ThumbnailDownloader线程上规划主线程上执行的任务.jpg)

	+ 在`ThumbnailDownloader`中，添加上述`mResponseHandler`变量，以存放来自于主线程的Handler。然后，以一个接受Handler的构造方法替换原有构造方法，并设置变量的值，最后新增一个用来通信的监听器接口。

		```java
        public class ThumbnailDownloader<T> extends HandlerThread {
            private static final String TAG = "ThumbnailDownloader";
            private static final int MESSAGE_DOWNLOAD = 0;
            private Handler mRequestHandler;
            private ConcurrentMap<T,String> mRequestMap = new ConcurrentHashMap<>();

            private Handler mResponseHandler;
            private ThumbnailDownloadListener<T> mThumbnailDownloadListener;

            public interface ThumbnailDownloadListener<T> {
                void onThumbnailDownloaded(T target, Bitmap thumbnail);
            }

            public void setThumbnailDownloadListener(ThumbnailDownloadListener<T> listener) {
                mThumbnailDownloadListener = listener;
            }

            public ThumbnailDownloader(Handler responseHandler) {
                super(TAG);
                mResponseHandler = responseHandler;
            }
            ...
        }
		```

	+ 修改`PhotoGalleryFragment`，将Handler传递给`ThumbnailDownloader`，并设置Listener，将返回的BitMap设置给ImageView。记住，Handler默认与当前线程的Looper相关联。该Handler是在`onCreate(...)`方法中创建的，因此它将与主线程的Looper相关联。创建responseHandler。

		```java
        ...
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setRetainInstance(true);
            new FetchItemsTask().execute();

            Handler responseHandler = new Handler();
            mThumbnailDownloader = new ThumbnailDownloader<>(responseHandler);
            mThumbnailDownloader.setThumbnailDownloadListener(
                new ThumbnailDownloader.ThumbnailDownloadListener<PhotoHolder>() {
                    @Override
                    public void onThumbnailDownloaded(PhotoHolder photoHolder, Bitmap bitmap) {
                        Drawable drawable = new BitmapDrawable(getResources(), bitmap);
                        photoHolder.bindDrawable(drawable);
                    }
                }
            );
            mThumbnailDownloader.start();
            mThumbnailDownloader.getLooper();
            Log.i(TAG, "Background thread started");
        }
		```

		+ 通过mResponseHandler，ThumbnailDownloader能够访问与主线程Looper绑定的Handler。同时，它的Listener会使用返回的Bitmap执行UI更新操作。
	+ 要返回定制Message给主线程，需要另一个Handler子类，以及一个`handleMessage(...)`覆盖方法。这里，使用另一种方便的Handler方法——`post(Runnable)`。`Handler.post(Runnable)`是一个发送Message的便利方法。具体使用如下：

		```java
        Runnable myRunnable = new Runnable() {
            public void run() {
                /* Your code here */
            }
        };
        Message m = mHandler.obtainMessage();
        m.callback = myRunnable;
		```

		+ Message具有回调方法时，使用回调方法中的Runnable，而非其Handler目标来实现运行。
	+ 在`ThumbnailDownloader.handleRequest()`方法中，添加代码：

		```java
        public class ThumbnailDownloader<T> extends HandlerThread {
            ...
            private Handler mResponseHandler;
            private ThumbnailDownloadListener<T> mThumbnailDownloadListener;
            ...
            private void handleRequest(final T target) {
                try {
                    final String url = mRequestMap.get(target);

                    if (url == null) {
                        return;
                    }

                    byte[] bitmapBytes = new FlickrFetchr().getUrlBytes(url);
                    final Bitmap bitmap = BitmapFactory
                            .decodeByteArray(bitmapBytes, 0, bitmapBytes.length);
                    Log.i(TAG, "Bitmap created");

                    mResponseHandler.post(new Runnable() {
                        public void run() {
                            if (mRequestMap.get(target) != url) {
                                return;
                            }

                            mRequestMap.remove(target);
                            mThumbnailDownloadListener.onThumbnailDownloaded(target, bitmap);
                        }
                    });
                } catch (IOException ioe) {
                    Log.e(TAG, "Error downloading image", ioe);
                }
            }
        }
		```

		+ 因为`mResponseHandler`与主线程的Looper相关联，所以UI更新代码也是在主线程中完成的。
		+ 首先，再次检查了requestMap。这很有必要，因为RecyclerView会循环使用它的视图。ThumbnailDownloader完成Bitmap下载后，Recycler可能已经循环使用了ImageView，并继续请求一个不同的URL。该检查可保证每个`PhotoHolder`都能获取到正确的图片，即使中间发生了其他请求也无妨。
		+ 最后，从requestMap中删除`PhotoHolder-URL`键值对，然后将bitmap设置到`PhotoHolder`上。
	+ 如果用户旋转屏幕，因ImageView视图的失效，ThumbnailDownloader则可能会挂起。如果点击这些ImageView，就可能发生异常。新增下列方法清除队列外的所有下载请求。(`ThumbnailDownloader.java`)

		```java
        public class ThumbnailDownloader<T> extends HandlerThread {
            ...
            public void queueThumbnail(T target, String url) {
                ...
            }

            public void clearQueue() {
                mRequestHandler.removeMessages(MESSAGE_DOWNLOAD);
            }

            private void handleRequest(final T target) {
                ...
            }
        }
		```

	+ 既然视图已销毁，在`PhotoGalleryFragment`中添加下载器清除代码。

		```java
        @Override
        public void onDestroyView() {
            super.onDestroyView();
            mThumbnailThread.clearQueue();
        }
		```

+ 深入学习：AsyncTask 与Thread
	+ AsyncTask的工作方式不适用于本章的使用场景。它主要应用于那些短暂且较少重复的任务。上一章的实现代码才是AsyncTask大展身手的地方。如果创建了大量的AsyncTask，或者长时间运行了AsyncTask，那么很可能是做了错误的选择。
	+ 在Android 3.2系统版本中，AsyncTask的内部实现发生了重大变化。自Android 3.2版本起，AsyncTask不再为每一个AsyncTask实例单独创建一个线程。相反，它使用一个Executor在单一的后台线程上运行所有AsyncTask的后台任务。这意味着每个AsyncTask都需要排队逐个运行。显然，长时间运行的AsyncTask会阻塞其他AsyncTask。
	+ 使用一个线程池executor虽然可安全地并发运行多个AsyncTask，但不推荐这么做。如果真的考虑这么做，最好自己处理线程相关的工作，必要时可使用Handler与主线程通信。

+ 深入学习：使用`Picasso`([http://square.github.io/picasso/](http://square.github.io/picasso/))第三方库来解决图片下载问题。

+ 挑战练习：预加载以及缓存
	+ 为接近完美的即时性，大多实际应用都通过以下两种方式增强自己的代码：
		+ 增加一个缓存层
		+ 预加载图片
	+ 缓存指存储一定数目Bitmap对象的地方。这样，即使不再使用这些对象，它们也依然存储在那里。缓存的存储空间有限，因此，在缓存空间用完的情况下，需要某种策略对保存的对象做一定的取舍。许多缓存机制使用一种叫做`LRU`（`least recently used`，最近最少使用）的存储策略。基于该种策略，当存储空间用尽时，缓存将清除最近最少使用的对象。
	+ Android支持库中的`LruCache`类实现了LRU缓存策略。作为第一个挑战练习，请使用LruCache为ThumbnailDownloader增加简单的缓存功能。这样，每次完成下载Bitmap时，将其存入缓存中。然后，准备下载新图片时，首先查看缓存，确认它是否存在。
	+ 缓存实现完成后，即可使用它进行预加载。预加载是指在实际使用对象前，预先就将处理对象加载到缓存中。这样，在显示Bitmap时，就不会存在下载延迟。



































