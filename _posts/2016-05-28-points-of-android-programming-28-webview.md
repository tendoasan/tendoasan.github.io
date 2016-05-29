---
layout: post
title: "第28章 网页浏览与WebView"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","WebView","PhotoGallery"]
description: "网页浏览与WebView"
---
{% include JB/setup %}

#### 28.网页浏览与WebView

+ Flickr 数据
	+ 为每一张图片的页面创建一个`URL`: `http://www.flickr.com/photos/user-id/photo-id`
	+ `photo-id`与从`JSON`中获取的id属性值相同，即GalleryItem的`mId`，JSON数据中的owner属性则对应`user ID`。
	+ 获取图片页面Url的代码 (`GalleryItem.java`)

		```java
        public class GalleryItem {
            ...
            private String mUrl;
            private String mOwner;
            ...
            public void setUrl(String url) {
                mUrl = url;
            }

            public String getOwner() {
                return mOwner;
            }

            public void setOwner(String owner) {
                mOwner = owner;
            }

            public Uri getPhotoPageUri() {
                return Uri.parse("http://www.flickr.com/photos/")
                        .buildUpon()
                        .appendPath(mOwner)
                        .appendPath(mId)
                        .build();
            }

            @Override
            public String toString() {
                return mCaption;
            }
        }
		```

	+ 获取owner属性(`FlickrFetchr.java`)

		```java
        public class FlickrFetchr {
            ...
            private void parseItems(List<GalleryItem> items, JSONObject jsonBody)
                    throws IOException, JSONException {
                ...
                for (int i = 0; i < photoJsonArray.length(); i++) {
                    JSONObject photoJsonObject = photoJsonArray.getJSONObject(i);
                    ...
                    if (!photoJsonObject.has("url_s")) {
                        continue;
                    }
                    item.setUrl(photoJsonObject.getString("url_s"));
                    // 设置Owner属性
                    item.setOwner(photoJsonObject.getString("owner"));
                    items.add(item);
                }
            }
        }
		```

+ 简单方式: 使用隐式Intents
	+ 当item被点击时创建隐式intent (`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends VisibleFragment {
            ...
            private class PhotoHolder extends RecyclerView.ViewHolder
                     implements View.OnClickListener {
                private ImageView mItemImageView;
                private GalleryItem mGalleryItem;

                public PhotoHolder(View itemView) {
                    super(itemView);

                    mItemImageView = (ImageView) itemView
                        .findViewById(R.id.fragment_photo_gallery_image_view);
                    // 设置监听器
                    itemView.setOnClickListener(this);
                }

                public void bindDrawable(Drawable drawable) {
                    mItemImageView.setImageDrawable(drawable);
                }

                public void bindGalleryItem(GalleryItem galleryItem) {
                    mGalleryItem = galleryItem;
                }

                @Override
                public void onClick(View v) {
                    Intent i = new Intent(Intent.ACTION_VIEW, mGalleryItem.getPhotoPageUri());
                    startActivity(i);
                }
            }
            ...
        }
		```

	+ 在`PhotoAdapter.onBindViewHolder(…)`中，将`GalleryItem`与`PhotoHolder`绑定。(`PhotoGalleryFragment.java`)

		```java
        ...
        private class PhotoAdapter extends RecyclerView.Adapter<PhotoHolder> {
            ...
            @Override
            public void onBindViewHolder(PhotoHolder photoHolder, int position) {
                GalleryItem galleryItem = mGalleryItems.get(position);
                photoHolder.bindGalleryItem(galleryItem);
                Drawable placeholder = getResources().getDrawable(R.drawable.bill_up_close);
                photoHolder.bindDrawable(placeholder);
                mThumbnailDownloader.queueThumbnail(photoHolder, galleryItem.getUrl());
            }
            ...
        }
        ...
		```

+ 复杂方式: 使用WebView
	+ 用WebView在用户交互界面显示网页内容。
	+ 创建`WebView`。
		+ 布局文件 (`res/layout/fragment_photo_page.xml`)

			```xml
			<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
							android:layout_width="match_parent"
							android:layout_height="match_parent">
				<WebView
					android:id="@+id/fragment_photo_page_web_view"
					android:layout_width="match_parent"
					android:layout_height="wrap_content"
					android:layout_alignParentBottom="true"
					android:layout_alignParentTop="true">
				</WebView>
			</RelativeLayout>
			```

		+ 网页浏览 fragment (`PhotoPageFragment.java`)

			```java
			public class PhotoPageFragment extends VisibleFragment {
				private static final String ARG_URI = "photo_page_url";
				private Uri mUri;
				private WebView mWebView;

				public static PhotoPageFragment newInstance(Uri uri) {
					Bundle args = new Bundle();
					args.putParcelable(ARG_URI, uri);

					PhotoPageFragment fragment = new PhotoPageFragment();
					fragment.setArguments(args);
					return fragment;
				}

				@Override
				public void onCreate(Bundle savedInstanceState) {
					super.onCreate(savedInstanceState);

					mUri = getArguments().getParcelable(ARG_URI);
				}

				@Override
				public View onCreateView(LayoutInflater inflater, ViewGroup container,
										 Bundle savedInstanceState) {
					View v = inflater.inflate(R.layout.fragment_photo_page, container, false);

					mWebView = (WebView) v.findViewById(R.id.fragment_photo_page_web_view);

					return v;
				}
			}
			```

		+ 托管 activity (`PhotoPageActivity.java`)

			```java
			public class PhotoPageActivity extends SingleFragmentActivity {
				public static Intent newIntent(Context context, Uri photoPageUri) {
					Intent i = new Intent(context, PhotoPageActivity.class);
					i.setData(photoPageUri);
					return i;
				}

				@Override
				protected Fragment createFragment() {
					return PhotoPageFragment.newInstance(getIntent().getData());
				}
			}
			```

		+ 在`PhotoGalleryFragment`中启动新的activity (`PhotoGalleryFragment.java`)

			```java
			public class PhotoGalleryFragment extends VisibleFragment {
				...
				private class PhotoHolder extends RecyclerView.ViewHolder
						implements View.OnClickListener{
					...
					@Override
					public void onClick(View v) {
						Intent i = PhotoPageActivity
							.newIntent(getActivity(), mGalleryItem.getPhotoPageUri());
						startActivity(i);
					}
				}
				...
			}
			```

		+ 配置新activity (`AndroidManifest.xml`)

			```xml
			<manifest ... >
				...
				<application
					...>
					<activity
						android:name=".PhotoGalleryActivity"
						android:label="@string/app_name" >
						...
					</activity>
					<activity android:name=".PhotoPageActivity" />
					<service android:name=".PollService" />
					...
				</application>
			</manifest>
			```

	+ 为了使得`WebView`成功显示`Flickr`的图片页面，需要做如下三件事。
		+ 告诉WebView要打开的URL。
		+ 启用JavaScript。
		+ 覆盖WebViewClient类的`shouldOverrideUrlLoading(WebView,String)`方法，并返回false值。
	(`PhotoPageFragment.java`)

			```java
			public class PhotoPageFragment extends VisibleFragment {
				...
				@Override
				public View onCreateView(LayoutInflater inflater, ViewGroup container,
										 Bundle savedInstanceState) {
					View v = inflater.inflate(R.layout.fragment_photo_page, container, false);
					mWebView = (WebView) v.findViewById(R.id.fragment_photo_page_web_view);

					mWebView.getSettings().setJavaScriptEnabled(true);
					mWebView.setWebViewClient(new WebViewClient() {
						public boolean shouldOverrideUrlLoading(WebView view, String url) {
							return false;
						}
					});
					mWebView.loadUrl(mUri.toString());

					return v;
				}
			}
			```

		+ 加载URL网页必须等WebView配置完成后进行，因此这一操作最后完成。在此之前，首先调用`getSettings()`方法获得WebSettings实例，再调用`WebSettings.setJavaScriptEnabled(true)`方法，从而完成JavaScript的启用。WebSettings是修改WebView配置的三种途径之一。它还有其他一些可设置属性，如用户代理字符串和显示文字大小。
		+ 然后，是配置`WebViewClient`。WebViewClient是一个事件接口。通过提供自己实现的WebViewClient，可响应各种渲染事件。例如，可检测渲染器何时开始从特定URL加载图片，或决定是否需要向服务器重新提交POST请求。
		+ `WebViewClient`有多个方法可供覆盖，其中大多数用不到。然而，我们必须覆盖它的`shouldOverrideUrlLoading(WebView,String)`默认方法。当有新的URL加载到WebView（譬如说点击某个链接），该方法会决定下一步的行动。如返回true值，意即“不要处理这个URL，我自己来。”如返回false值，意即“WebView，去加载这个URL，我不会对它做任何处理。”

+ 使用WebChromeClient优化WebView的显示
	+ 添加标题和进度条(`fragment_photo_page.xml`)

		```xml
        <RelativeLayout ...>
            <ProgressBar
                android:id="@+id/fragment_photo_page_progress_bar"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_alignParentTop="true"
                android:visibility="gone"
                style="?android:attr/progressBarStyleHorizontal"
                android:background="?attr/colorPrimary"/>
            <WebView
                android:id="@+id/fragment_photo_page_web_view"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:layout_alignParentBottom="true"
                android:layout_below="@id/fragment_photo_page_progress_bar" />
        </RelativeLayout>
		```

	+ 引用并显示ProgressBar和TextView视图非常简单。但要将它们与WebView关联起来，还需使用`WebView: WebChromeClient`的第二个回调方法。如果说WebViewClient是响应渲染事件的接口，那么WebChromeClient就是一个响应那些改变浏览器中装饰元素的事件接口。这包括JavaScript警告信息、网页图标、状态条加载，以及当前网页标题的刷新。在`onCreateView(...)`方法中，编写代码实现WebChromeClient的关联使用 (`PhotoPageFragment.java`)

		```java
        public class PhotoPageFragment extends VisibleFragment {
            ...
            private WebView mWebView;
            **private ProgressBar mProgressBar;**
            ...
            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                View v = inflater.inflate(R.layout.fragment_photo_page, container, false);
                mProgressBar =
                        (ProgressBar)v.findViewById(R.id.fragment_photo_page_progress_bar);
                mProgressBar.setMax(100); // WebChromeClient reports in range 0-100

                mWebView = (WebView) v.findViewById(R.id.fragment_photo_page_web_view);
                mWebView.getSettings().setJavaScriptEnabled(true);

                mWebView.setWebChromeClient(new WebChromeClient() {
                    public void onProgressChanged(WebView webView, int newProgress) {
                        if (newProgress == 100) {
                            mProgressBar.setVisibility(View.GONE);
                        } else {
                            mProgressBar.setVisibility(View.VISIBLE);
                            mProgressBar.setProgress(newProgress);
                        }
                    }

                    public void onReceivedTitle(WebView webView, String title) {
                        AppCompatActivity activity = (AppCompatActivity) getActivity();
                        activity.getSupportActionBar().setSubtitle(title);
                    }
                });

                mWebView.setWebViewClient(new WebViewClient() {
                    ...
                });
                mWebView.loadUrl(mUri.toString());

                return v;
            }
        }
		```

		+ 进度条和标题栏的更新都有各自的回调方法，即`onProgressChanged(WebView,int)`和`onReceivedTitle(WebView,String)`方法。从`onProgressChanged(WebView,int)`方法收到的网页加载进度是一个从0到100的整数值。如果值是100，说明网页已完成加载，因此需设置进度条可见性为View.INVISIBLE，将ProgressBar视图隐藏起来

+ 处理WebView的旋转问题。
	+ 告诉`PhotoPageActivity`自己处理配置变更。 (`AndroidManifest.xml`)

		```xml
        <manifest ... >
          ...
              <activity
                  android:name=".PhotoPageActivity"
                  android:configChanges="keyboardHidden|orientation|screenSize" />
              ...
        </manifest>
		```

+ 挑战练习：后退按钮访问浏览历史；支持Non-HTTP链接。













































































