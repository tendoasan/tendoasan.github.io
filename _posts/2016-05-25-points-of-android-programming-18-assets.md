---
layout: post
title: "第18章 Assets"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Assets","BeatBox"]
description: "Assets"
---
{% include JB/setup %}


#### 18.Assets

+ 建立BeatBox项目。
	+ 添加`RecyclerView`依赖。
	+ 创建`RecyclerView`布局文件。(`res/layout/fragment_beat_box.xml`)

		```xml
		<?xml version="1.0" encoding="utf-8"?>
		<android.support.v7.widget.RecyclerView
			xmlns:android="http://schemas.android.com/apk/res/android"
			android:id="@+id/fragment_beat_box_recycler_view"
			android:layout_width="match_parent"
			android:layout_height="match_parent"/>
		```

	+ 为列表项目`sound`创建布局文件。(`res/layout/list_item_sound.xml`)

		```xml
		<?xml version="1.0" encoding="utf-8"?>
		<Button
			xmlns:android="http://schemas.android.com/apk/res/android"
			xmlns:tools="http://schemas.android.com/tools"
			android:id="@+id/list_item_sound_button"
			android:layout_width="match_parent"
			android:layout_height="120dp"
			tools:text="Sound name"/>
		```

	+ 创建`BeatBoxFragment`。(`BeatBoxFragment.java`)

		```java
        public class BeatBoxFragment extends Fragment {
            public static BeatBoxFragment newInstance() {
                return new BeatBoxFragment();
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                View view = inflater.inflate(R.layout.fragment_beat_box, container, false);
                RecyclerView recyclerView = (RecyclerView)view
                    .findViewById(R.id.fragment_beat_box_recycler_view);

                LayoutManager layoutManager = new GridLayoutManager(getActivity(), 3);
                recyclerView.setLayoutManager(layoutManager);

                recyclerView.setAdapter(new SoundAdapter());

                return view;
            }

            private class SoundHolder extends RecyclerView.ViewHolder {
                private Button mButton;

                public SoundHolder(LayoutInflater inflater, ViewGroup container) {
                    super(inflater.inflate(R.layout.list_item_sound, container, false));

                    mButton = (Button)itemView.findViewById(R.id.list_item_sound_button);
                }
            }

            private class SoundAdapter extends RecyclerView.Adapter<SoundHolder> {
                @Override
                public SoundHolder onCreateViewHolder(ViewGroup parent, int viewType) {
                    LayoutInflater inflater = LayoutInflater.from(getActivity());
                    return new SoundHolder(inflater, parent);
                }

                @Override
                public void onBindViewHolder(SoundHolder soundHolder, int position) {}

                @Override
                public int getItemCount() {
                    return 0;
                }
            }
        }
		```

	+ 添加`BeatBoxFragment`给`BeatBoxActivity`托管。(`BeatBoxActivity.java`)

		```java
		public class BeatBoxActivity extends SingleFragmentActivity {
			@Override
				protected Fragment createFragment() {
					return BeatBoxFragment.newInstance();
				}
		}
		```

+ 导入Assets资源。
	+ 在项目的`app module`上右键，选择`New → Folder → Assets Folder`，`Change Folder Location`和`Target Source Set`使用默认结果。
	+ 在`assets`目录上右键，选择`New → Directory`，命名为`sample_sounds`，存放需要展示的`.wav`文件。

+ 获取Assets资源。使用`AssetManager` 查找，追踪，获取以及调用assets。可以从`Context`直接获得`AssetManager`。
	+ 新建`Sound`模型类，通过传入文件的路径名来获取文件名。(`Sound.java`)

		```java
        public class Sound {
            private String mAssetPath;
            private String mName;

            public Sound(String assetPath) {
                mAssetPath = assetPath;
                String[] components = assetPath.split("/");
                String filename = components[components.length - 1];
                mName = filename.replace(".wav", "");
            }

            public String getAssetPath() {
                return mAssetPath;
            }

            public String getName() {
                return mName;
            }
        }
		```

	+ 新建`BeatBox`模型类，封装一个`AssetManager`，获取assets文件夹的文件名列表。(`BeatBox.java`)

		```java
        public class BeatBox {
            private static final String TAG = "BeatBox";
            private static final String SOUNDS_FOLDER = "sample_sounds";
            private AssetManager mAssets;
            private List<Sound> mSounds = new ArrayList<>();

            public BeatBox(Context context) {
                mAssets = context.getAssets();
                loadSounds();
            }

            private void loadSounds() {
            	// 获得该目录下所有文件的路径名所组成的字符串数组。
                String[] soundNames;
                try {
                    soundNames = mAssets.list(SOUNDS_FOLDER);
                    Log.i(TAG, "Found " + soundNames.length + " sounds");
                } catch (IOException ioe) {
                    Log.e(TAG, "Could not list assets", ioe);
                    return;
                }

                // 通过路径名来新建Sound对象
                for (String filename : soundNames) {
                    String assetPath = SOUNDS_FOLDER + "/" + filename;
                    Sound sound = new Sound(assetPath);
                    mSounds.add(sound);
                }
            }

            public List<Sound> getSounds() {
                return mSounds;
            }
        }
		```

	+ 在`BeatBoxFragment`中修改`SounHolder`来绑定`Sound`，然后在`SoundAdapter`传入`Sounds`。(`BeatBoxFragment.java`)

		```java
        private class SoundHolder extends RecyclerView.ViewHolder {
            private Button mButton;
            private Sound mSound;

            public SoundHolder(LayoutInflater inflater, ViewGroup container) {
                ...
            }

            public void bindSound(Sound sound) {
                mSound = sound;
                mButton.setText(mSound.getName());
            }
        }
        ...
        private class SoundAdapter extends RecyclerView.Adapter<SoundHolder> {
            private List<Sound> mSounds;

            public SoundAdapter(List<Sound> sounds) {
                mSounds = sounds;
            }
            ...
            @Override
            public void onBindViewHolder(SoundHolder soundHolder, int position) {
                Sound sound = mSounds.get(position);
                soundHolder.bindSound(sound);
            }

            @Override
            public int getItemCount() {
                return mSounds.size();
            }
        }
		```

	+ 在`BeatBoxFragment`中创建`BeatBox`实例，通过`BeatBox`实例来获得所有`Sound`的文件名列表(`getSounds()`)。(`BeatBoxFragment.java`)

		```java
        public class BeatBoxFragment extends Fragment {

            private BeatBox mBeatBox;

            public static BeatBoxFragment newInstance() {
                return new BeatBoxFragment();
            }

            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);

                mBeatBox = new BeatBox(getActivity());
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                View view = inflater.inflate(R.layout.fragment_beat_box, container, false);

                RecyclerView recyclerView = (RecyclerView)view
                    .findViewById(R.id.fragment_beat_box_recycler_view);
                recyclerView.setLayoutManager(new GridLayoutManager(getActivity(), 3));

                recyclerView.setAdapter(new SoundAdapter(mBeatBox.getSounds()));

                return view;
            }
            ...
        }
		```







































































