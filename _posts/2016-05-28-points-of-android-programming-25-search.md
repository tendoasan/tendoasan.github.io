---
layout: post
title: "第25章 搜索"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Search","PhotoGallery"]
description: "搜索"
---
{% include JB/setup %}

#### 25.搜索

+ 搜索 `Flickr` 网站。
	+ 添加`Flickr`搜索方法。

		```java
        public class FlickrFetchr {
            private static final String TAG = "FlickrFetchr";

            private static final String API_KEY = "yourApiKeyHere";
            private static final String FETCH_RECENTS_METHOD = "flickr.photos.getRecent";
            private static final String SEARCH_METHOD = "flickr.photos.search";
            private static final Uri ENDPOINT = Uri
                    .parse("https://api.flickr.com/services/rest/")
                    .buildUpon()
                    .appendQueryParameter("api_key", API_KEY)
                    .appendQueryParameter("format", "json")
                    .appendQueryParameter("nojsoncallback", "1")
                    .appendQueryParameter("extras", "url_s")
                    .build();
            ...
            // 获取图片的方法
            public List<GalleryItem> fetchRecentPhotos() {
                String url = buildUrl(FETCH_RECENTS_METHOD, null);
                return downloadGalleryItems(url);
            }
			// 搜索图片的方法
            public List<GalleryItem> searchPhotos(String query) {
                String url = buildUrl(SEARCH_METHOD, query);
                return downloadGalleryItems(url);
            }
            // 将 fetchItems()更名为downloadGalleryItems(String url)
            public List<GalleryItem> downloadGalleryItems(String url) {
                List<GalleryItem> items = new ArrayList<>();
                try {

                    String jsonString = getUrlString(url);
                    ...
                } catch (IOException ioe) {
                    Log.e(TAG, "Failed to fetch items", ioe);
                } catch (JSONException je) {
                    Log.e(TAG, "Failed to parse JSON", je);
                }
                return items;
            }
            // 构建 Url的专用方法
            private String buildUrl(String method, String query) {
                Uri.Builder uriBuilder = ENDPOINT.buildUpon()
                        .appendQueryParameter("method", method);

                if (method.equals(SEARCH_METHOD)) {
                    uriBuilder.appendQueryParameter("text", query);
                }

                return uriBuilder.build().toString();
            }
            ...
        }
		```

	+ 根据`query`的值来调用不同方法。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            ...
            private class FetchItemsTask extends AsyncTask<Void,Void,List<GalleryItem>> {
                @Override
                protected List<GalleryItem> doInBackground(Void... params) {
                    return new FlickrFetchr().fetchItems();
                    String query = "robot"; // Just for testing

                    if (query == null) {
                        return new FlickrFetchr().fetchRecentPhotos();
                    } else {
                        return new FlickrFetchr().searchPhotos(query);
                    }
                }

                @Override
                protected void onPostExecute(List<GalleryItem> items) {
                    mItems = items;
                    setupAdapter();
                }
            }
        }
		```

+ 使用搜索视图。
	+ 搜索视图(`SearchView`)是一个`action view`，可以将搜索界面都包含在操作栏里面。
	+ 添加搜索菜单项。(`res/menu/fragment_photo_gallery.xml`)

		```xml
        <?xml version="1.0" encoding="utf-8"?>
        <menu xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:app="http://schemas.android.com/apk/res-auto">

            <item android:id="@+id/menu_item_search"
                  android:title="@string/search"
                  app:actionViewClass="android.support.v7.widget.SearchView"
                  app:showAsAction="ifRoom" />

            <item android:id="@+id/menu_item_clear"
                  android:title="@string/clear_search"
                  app:showAsAction="never" />
        </menu>
		```

	+ 添加搜索字符串。(`res/values/strings.xml`)

		```xml
        <resources>
                ...
                <string name="search">Search</string>
                <string name="clear_search">Clear Search</string>
        </resources>
		```

	+ 添加选项菜单回调方法。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            ...
            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setRetainInstance(true);

                setHasOptionsMenu(true);
                ...
            }
            ...
            @Override
            public void onDestroy() {
                ...
            }

            @Override
            public void onCreateOptionsMenu(Menu menu, MenuInflater menuInflater) {
                super.onCreateOptionsMenu(menu, menuInflater);
                menuInflater.inflate(R.menu.fragment_photo_gallery, menu);
            }
            ...
        }
		```

+ 响应搜索视图的用户交互。添加`Search.OnQueryTextListener`监听器，封装异步任务的执行(`PhotoGalleryFragment.java`)

	```java
    public class PhotoGalleryFragment extends Fragment {
        ...
        @Override
        public void onCreate(Bundle savedInstanceState) {
            ...
            setHasOptionsMenu(true);
            updateItems();
            ...
            Log.i(TAG, "Background thread started");
        }
        ...
        @Override
        public void onCreateOptionsMenu(Menu menu, MenuInflater menuInflater) {
            super.onCreateOptionsMenu(menu, menuInflater);
            menuInflater.inflate(R.menu.fragment_photo_gallery, menu);

            MenuItem searchItem = menu.findItem(R.id.menu_item_search);
            final SearchView searchView = (SearchView) searchItem.getActionView();

            searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
                @Override
                public boolean onQueryTextSubmit(String s) {
                    Log.d(TAG, "QueryTextSubmit: " + s);
                    updateItems();
                    return true;
                }

                @Override
                public boolean onQueryTextChange(String s) {
                    Log.d(TAG, "QueryTextChange: " + s);
                    return false;
                }
            });
        }

        private void updateItems() {
            new FetchItemsTask().execute();
        }
        ...
    }
	```

+ 使用`shared preferences`实现轻量级数据存储。
	+ 新建`QueryPreferences`类保存搜索词条(`QueryPreferences.java`)

		```java
        public class QueryPreferences {
            private static final String PREF_SEARCH_QUERY = "searchQuery";

            public static String getStoredQuery(Context context) {
                return PreferenceManager.getDefaultSharedPreferences(context)
                        .getString(PREF_SEARCH_QUERY, null);
            }

            public static void setStoredQuery(Context context, String query) {
                PreferenceManager.getDefaultSharedPreferences(context)
                        .edit()
                        .putString(PREF_SEARCH_QUERY, query)
                        .apply();
            }
        }
		```

	+ 当用户提交一个新的搜索词条时，保存该词条。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            ...
            @Override
            public void onCreateOptionsMenu(Menu menu, MenuInflater menuInflater) {
                ...
                searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
                    @Override
                    public boolean onQueryTextSubmit(String s) {
                        Log.d(TAG, "QueryTextSubmit: " + s);
                        // 保存词条
                        QueryPreferences.setStoredQuery(getActivity(), s);
                        updateItems();
                        return true;
                    }

                    @Override
                    public boolean onQueryTextChange(String s) {
                        Log.d(TAG, "QueryTextChange: " + s);
                        return false;
                    }
                });
            }
            ...
        }
		```

	+ 当用户选择菜单栏的`Clear Search`时，清除保存的词条。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            ...
            @Override
            public void onCreateOptionsMenu(Menu menu, MenuInflater menuInflater) {
                ...
            }

            @Override
            public boolean onOptionsItemSelected(MenuItem item) {
                switch (item.getItemId()) {
                    case R.id.menu_item_clear:
                        // 清除词条
                        QueryPreferences.setStoredQuery(getActivity(), null);
                        updateItems();
                        return true;
                    default:
                        return super.onOptionsItemSelected(item);
                }
            }
            ...
        }
		```

	+ 在异步任务`FetchItemsTask`中取出保存的词条。(`PhotoGalleryFragment.java`)

		```java
        public class PhotoGalleryFragment extends Fragment {
            ...
            private void updateItems() {
                // 取出存储的词条
                String query = QueryPreferences.getStoredQuery(getActivity());
                new FetchItemsTask(query).execute();
            }
            ...
            private class FetchItemsTask extends AsyncTask<Void,Void,List<GalleryItem>> {
                private String mQuery;

                public FetchItemsTask(String query) {
                    mQuery = query;
                }

                @Override
                protected List<GalleryItem> doInBackground(Void... params) {
                    if (mQuery == null) {
                        return new FlickrFetchr().fetchRecentPhotos();
                    } else {
                        return new FlickrFetchr().searchPhotos(querymQuery);
                    }
                }

                @Override
                protected void onPostExecute(List<GalleryItem> items) {
                    mItems = items;
                    setupAdapter();
                }
            }
        }
		```

+ 完善应用。打开搜索视图时预填充查询词条。(`PhotoGalleryFragment.java`)

	```java
    public class PhotoGalleryFragment extends Fragment {
      ...
      @Override
      public void onCreateOptionsMenu(Menu menu, MenuInflater menuInflater) {
          ...
          searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
              ...
          });

          searchView.setOnSearchClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View v) {
                  String query = QueryPreferences.getStoredQuery(getActivity());
                  searchView.setQuery(query, false);
              }
          });
      }
      ...
    }
	```

+ 挑战练习：搜索词条提交后关闭键盘，折叠`SearchView`；词条提交后，清空`RecyclerView`，显示一个`loading indicator`(`indeterminate progress bar`)，当`JSON`数据下载完成后，进度提示关闭。














































































































