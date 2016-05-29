---
layout: post
title: "第19章 Audio Playback"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Audio","BeatBox","Soundpool"]
description: "Audio Playback"
---
{% include JB/setup %}

#### 19.使用SoundPool来播放音频

+ `Sound`模型添加`mSoundId`属性。(`Sound.java`)

	```java
    public class Sound {
        ...
        private Integer mSoundId;

        public Sound(String assetPath) {
            ...
        }
        ...
        public Integer getSoundId() {
            return mSoundId;
        }

        public void setSoundId(Integer soundId) {
            mSoundId = soundId;
        }
    }
	```

+ `BeatBox`类中添加`SoundPool`实例来管理`Sounds`，添加`loadSound()`方法将`Sound`加载进`SoundPool`，添加`play()`方法播放`SoundPool`中的`Sound`。

	```java
    public class BeatBox {
        ...
        private static final int MAX_SOUNDS = 5;
        ...
        private SoundPool mSoundPool;

        public BeatBox(Context context){
            mAssets = context.getAssets();
            // 初始化 mSoundPool
            mSoundPool = new SoundPool(MAX_SOUNDS, AudioManager.STREAM_MUSIC, 0);
            loadSounds();
        }

        public void play(Sound sound){
            Integer soundId = sound.getSoundId();
            if (sound == null){
                return;
            }
            mSoundPool.play(soundId, 1.0f, 1.0f, 1, 0, 1.0f);
        }
        // 释放 Sound
        public void release(){
            mSoundPool.release();
        }

        private void loadSounds(){
            String[] soundNames;
            try{
                soundNames = mAssets.list(SOUNDS_FOLDER);
            } catch (IOException ioe){
                Log.e(TAG, "Could not list assets", ioe);
                return;
            }

            for (String filename : soundNames){
                try {
                    String assetPath = SOUNDS_FOLDER + "/" + filename;
                    Sound sound = new Sound(assetPath);
                    // 将所有 Sound 都加载到 SoundPool 中
                    load(sound);
                    mSounds.add(sound);
                } catch (IOException e) {
                    Log.e(TAG, "Could not load sound " + filename, e);
                }
            }
        }
        // 加载 Sound 到 SoundPool 中
        private void load(Sound sound) throws IOException{
            AssetFileDescriptor afd = mAssets.openFd(sound.getAssetPath());
            int soundId = mSoundPool.load(afd, 1);
            sound.setSoundId(soundId);
        }

        public List<Sound> getSounds(){
            return mSounds;
        }
    }
	```

	+ `mSoundPool.load(AssetFileDescriptor, int)`方法，将一个文件加载进`SoundPool`，返回一个整型ID，用于标记该文件，方便追踪，播放以及释放。
	+ `SoundPool.play(int, float, float, int, int, float)`方法有六个参数，分别为:
		- the sound ID;
		- volume on the left;
		- volume on the right;
		- priority (which is ignored),
		- whether the audio should loop;
		- and playback rate.

+ 在`BeatBoxFragment`中设置监听器播放以及释放`Sound`。(`BeatBoxFragment.java`)

	```java
	...
    private class SoundHolder extends RecyclerView.ViewHolder
            implements View.OnClickListener {
        private Button mButton;
        private Sound mSound;

        public SoundHolder(LayoutInflater inflater, ViewGroup container) {
            super(inflater.inflate(R.layout.list_item_sound, parent, false));
            mButton = (Button)itemView.findViewById(R.id.button);
            mButton.setOnClickListener(this);
        }

        public void bindSound(Sound sound) {
            mSound = sound;
            mButton.setText(mSound.getName());
        }

        @Override
        public void onClick(View v) {
            mBeatBox.play(mSound);
        }
    }
	...
	@Override
    public void onDestroy() {
        super.onDestroy();
        mBeatBox.release();
    }
    ...
	```

+ 播放过程中的屏幕旋转问题。设置`retainInstance`特性来解决。(`BeatBoxFragment.java`)

	```java
    ...
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
		// 设置发生改变后保存fragment实例
        setRetainInstance(true);

        mBeatBox = new BeatBox(getActivity());
    }
    ...
	```
	+ 旋转过程中保存的UI fragment。

	![]({{ IMAGE_PATH }}/旋转过程中保存的UI fragment.jpg)








































