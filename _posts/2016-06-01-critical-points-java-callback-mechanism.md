---
layout: post
title: "Android中的回调机制"
category: "笔记"
tags: "Android"
keywords: ["Android","Java","Callback"]
description: "Android中的回调机制"
---
{% include JB/setup %}

所谓回调：就是`A`类中调用`B`类中的某个方法`method_b`，然后`B`类中反过来调用`A`类中的方法`method_a`，`method_a`这个方法就叫回调方法。

比较经典的回调方式：

+ `class A`实现接口`CallBack callback` ——**背景1**
+ `class A`中包含一个`class B`的引用`b` ——**背景2**
+ `class B`有一个参数为`callback`的方法`method_b(CallBack callback)` ——**背景3**
+ `A`的对象`a`调用`B`的方法 `method_b(CallBack callback)` ——`A`类调用`B`类的某个方法`mehod_b`
+ 然后`b`就可以在`method_b(CallBack callback)`方法中调用`A`的方法 ——`B`类调用`A`类的某个方法`method_a`

#### 异步回调

用打电话的例子解释异步回调。
有一天小王遇到一个很难的问题，问题是“1 + 1 = ?”，就打电话问小李，小李一下子也不知道，就跟小王说，等我办完手上的事情，就去想想答案，小王也不会傻傻地拿着电话去等小李的答案吧，于是小王就对小李说，我还要去逛街，你知道了答案就打我电话告诉我，于是挂了电话，自己办自己的事情，过了一个小时，小李打了小王的电话，告诉他答案是2。

+ 定义回调接口：

	```java
	public interface CallBack {
		/**
		 * 小李知道答案时，需要调用的告诉小王的函数，也就是回调函数
		 * @param result 是答案
		 */
		public void solve(String result);
	}

	```

+ 定义`A`类(小王)：

	```java
	/**
	 * 这个是小王
	 * 实现了一个回调接口CallBack，相当于--->背景1
	 */
	public class Wang implements CallBack {
		private static final String TAG = "Wang";

		/**
		 * 小李对象的引用
		 * 相当于--->背景2
		 */
		private Li mLi;

		/**
		 * 小王的构造方法，持有小李的引用
		 * @param li
		 */
		public Wang(Li li){
			mLi = li;
		}

		/**
		 * 小王通过这个方法去问小李一个问题
		 * @param question 就是小王要问的问题，1 + 1 = ?
		 */
		public void askQuestion(final String question){
			// 这里用一个线程就是异步
			new Thread(new Runnable() {
				@Override
				public void run() {
					/**
					 * 小王调用小李中的方法，在这里注册回调接口
					 * 这就相当于A类中调用B类的方法method_b
					 */
					mLi.executeMessage(Wang.this, question);
				}
			}).start();

			// 小王问完问题挂掉电话就去干其他事情了，比如逛街去了
			play();
		}

		public void play(){
			Log.d(TAG, "我要逛街去了");
		}

		/**
		 * 小李知道答案后，调用此方法告诉小王，就是所谓的小王的回调方法
		 * @param result 是答案
		 */
		@Override
		public void solve(String result) {
			Log.d(TAG, "小李告诉小王的答案是--->" + result);
		}
	}
	```

+ 定义`B`类(小李)：

	```java
	/**
	 * 这个就是小李
	 */
	public class Li {
		private static final String TAG = "Li";

		/**
		 * 相当于B类有参数为CallBack callBack的method_b，相当于--->背景3
		 * @param callBack
		 * @param question 小王问的问题
		 */
		public void executeMessage(CallBack callBack, String question){
			Log.d(TAG, "小王问的问题--->" + question);

			// 模拟小李办自己的事情需要很长时间
			for (int i = 0; i < 10000; i++){}

			/**
			 * 小李办完自己的事情之后想到了答案是2
			 */
			String result = "答案是2";

			/**
			 * 于是就打电话告诉小王，调用小王中的方法
			 * 这就相当于B类反过来调用A的方法method_a
			 */
			callBack.solve(result);
		}
	}
	```

+ 测试：

	```java
	/**
	 * 测试类
	 */
	public class Test {
		public static void main(String[] args){
			/**
			 * new 一个小李
			 */
			Li li = new Li();

			/**
			 * new 一个小王
			 */
			Wang wang = new Wang(li);

			/**
			 * 小王问小李问题
			 */
			String question = "1 + 1 = ?";
			wang.askQuestion(question);
		}
	}
	```

---

#### 同步回调

通过`Android View`的`onClick()`方法来分析同步回调。

`onclick()`是一个回调方法，当用户点击`View`就会执行这个方法，这里用`Button`来举例。

+ 回调接口(源码)：

	```java
	/**
	 * 这个是View的一个回调接口
	 * Interface definition for a callback to be invoked when a view is clicked.
	 */
	public interface OnClickListener {
		/**
		 * Called when a view has been clicked.
		 * @param v The view that was clicked.
		 */
		void onclick(View v);
	}
	```

+ 定义`A`类：

	```java
	/**
	 * 这个就相当于Class A
	 * 实现了 OnClickListener接口--->背景1
	 */
	public class MainActivity extends Activity implements OnClickListener {

		/**
		 * Class A 包含Class B的引用--->背景2
		 */
		private Button mButton;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			mButton = (Button)findViewById(R.id.add_topic_button);

			/**
			 * Class A 调用View的方法，而Button extends View--->A类调用B类的某个方法method_b
			 */
			mButton.setOnClickListener(this);
		}

		/**
		 * 用户点击Button时调用的回调函数，你可以做你要做的事(响应事件)
		 * 这里我做的是用Toast提示OnClick
		 * @param v The view that was clicked.
		 */
		@Override
		public void onclick(View v) {
			Toast.makeText(getApplication(), "OnClick", Toast.LENGTH_LONG).show();
		}
	}
	```

+ `B`类(`Button`的父类`View`)的`method_b`方法(`setOnclickListener()`方法(源码))：

	```java
	/**
	 * 这个View就相当于B类
	 */
	public class View implements Drawable.Callback, KeyEvent.Callback, AccessibilityEventSource {
		/**
		 * Listener used to dispatch click events.
		 * This field should be made private, so it is hidden from the SDK.
		 * {@hide}
		 */
		protected OnClickListener mOnClickListener;

		/**
		 * setOnClickListener()的参数是OnClickListener接口-->背景3
		 * Register a callback to be invoked when this view is clicked. If this view is not
		 * clickable, it becomes clickable.
		 *
		 * @param l The callback that will run
		 *
		 * @see #setClickable(boolean)
		 */

		public void setOnClickListener(OnClickListener l) {
			if (!isClickable()) {
				setClickable(true);
			}
			mOnClickListener = l;
		}

		/**
		 * Call this view's OnClickListener, if it is defined.
		 *
		 * @return True there was an assigned OnClickListener that was called, false
		 *         otherwise is returned.
		 */
		public boolean performClick() {
			sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

			if (mOnClickListener != null) {
				playSoundEffect(SoundEffectConstants.CLICK);

				//这个不就是相当于B类调用A类的某个方法method_a，这个method_a就是所谓的回调方法
				mOnClickListener.onClick(this);
				return true;
			}

			return false;
		}
	}
	```

这个就是`Android`典型的回调机制。线程`run()`也是一个回调方法，当执行`Thread`的`start()`方法就会回调这个`run()`方法，还有处理消息都比较经典等等。

---

参考文章：[xiaanming的博客](http://blog.csdn.net/xiaanming/article/details/17483273)。







