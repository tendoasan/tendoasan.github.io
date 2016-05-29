---
layout: post
title: "第29章 定制视图和触摸事件"
category: "Android编程权威指南"
tags: "Android"
keywords: ["Android","Touch","DragAndDraw","Event"]
description: "定制视图和触摸事件"
---
{% include JB/setup %}

#### 29.定制视图和触摸事件

+ 创建DragAndDraw项目
	+ 创建DragAndDrawActivity。(`DragAndDrawActivity.java`)

		```java
        public class DragAndDrawActivity extends AppCompatActivity SingleFragmentActivity {
            @Override
            public Fragment createFragment() {
                return DragAndDrawFragment.newInstance();
            }
        }
		```

	+ 创建DragAndDrawFragment。(`DragAndDrawFragment.java`)

		```java
        public class DragAndDrawFragment extends Fragment {
            public static DragAndDrawFragment newInstance() {
                return new DragAndDrawFragment();
            }

            @Override
            public View onCreateView(LayoutInflater inflater, ViewGroup container,
                                     Bundle savedInstanceState) {
                View v = inflater.inflate(R.layout.fragment_drag_and_draw, container, false);
                return v;
            }
        }
		```

+ 创建定制视图。
	+ 创建定制视图所需的三大步骤。
		+ 选择超类。对于简单定制视图而言，View是一个空白画布，因此是最常见的选择。而对于聚合定制视图，我们应选择合适的布局类
		+ 继承选定的超类，并至少覆盖一个超类构造方法。或者创建自己的构造方法，并在其中调用超类的构造方法。
		+ 覆盖其他关键方法，以定制视图行为。

	+ 以`View`为超类，新建B`oxDrawingView`类。 (`BoxDrawingView.java`)

		```java
        public class BoxDrawingView extends View {
            // Used when creating the view in code
            public BoxDrawingView(Context context) {
                this(context, null);
            }

            // Used when inflating the view from XML
            public BoxDrawingView(Context context, AttributeSet attrs) {
                super(context, attrs);
            }
        }
		```

	+ 更新布局文件。(`fragment_drag_and_draw.xml`)

		```xml
        <com.bignerdranch.android.draganddraw.BoxDrawingView
            xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>
		```

+ 处理触摸事件。
	+ 监听触摸事件的一种方式是使用以下View方法，设置一个触摸事件监听器:

		```java
		public void setOnTouchListener(View.OnTouchListener l)
		```

		该方法的工作方式与`setOnClickListener(View.OnClickListener)`相同。实现`View.OnTouchListener`接口，供触摸事件发生时调用。
	+ 因为定制视图是View的子类，因此可走捷径直接覆盖以下View方法：

		```java
		public boolean onTouchEvent(MotionEvent event)
		```

	+ 该方法可以接收一个`MotionEvent`类实例，而`MotionEvent`类可用来描述包括位置和动作的触摸事件。动作则用来描述事件所处的阶段。

		|动作常量 | 动作描述|
		|---|---|
		|ACTION_DOWN |	用户手指触摸到屏幕|
		|ACTION_MOVE |	用户在屏幕上移动手指|
		|ACTION_UP	| 用户手指离开屏幕|
		|ACTION_CANCEL	| 父视图拦截了触摸事件|


	+ 在`onTouchEvent(...)`实现方法中，可使用以下`MotionEvent`方法，查看动作值：

		```java
		public final int getAction()
		```

	+ 在`BoxDrawingView`中，添加一个日志tag，然后实现`onTouchEvent(...)`方法记录可能发生的四个不同动作。(`BoxDrawingView.java`)

		```java
        public class BoxDrawingView extends View {
            private static final String TAG = "BoxDrawingView";
            ...
            @Override
            public boolean onTouchEvent(MotionEvent event) {
                PointF current = new PointF(event.getX(), event.getY());
                String action = "";

                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        action = "ACTION_DOWN";
                        break;
                    case MotionEvent.ACTION_MOVE:
                        action = "ACTION_MOVE";
                        break;
                    case MotionEvent.ACTION_UP:
                        action = "ACTION_UP";
                        break;
                    case MotionEvent.ACTION_CANCEL:
                        action = "ACTION_CANCEL";
                        break;
                }

                Log.i(TAG, action + " at x=" + current.x +
                        ", y=" + current.y);

                return true;
            }
        }
		```

+ 跟踪运动事件
	+ `BoxDrawingView`主要用于在屏幕上绘制矩形框。要实现这一目标，有几个问题需要解决。
	+ 首先，要定义一个矩形框，需知道：
		+ 原始坐标点（手指的初始位置）；
		+ 当前坐标点（手指的当前位置）。
	+ 其次，定义一个矩形框，还需追踪记录来自多个MotionEvent的数据。这些数据将会保存在Box对象中。
	+ 新建一个Box类，用于表示一个矩形框的定义数据。(`Box.java`)

		```java
        public class Box {
            private PointF mOrigin;
            private PointF mCurrent;

            public Box(PointF origin) {
                mOrigin = origin;
                mCurrent = origin;
            }

            public PointF getCurrent() {
                return mCurrent;
            }

            public void setCurrent(PointF current) {
                mCurrent = current;
            }

            public PointF getOrigin() {
                return mOrigin;
            }
        }
		```

	+ 回到`BoxDrawingView`类中，使用新的Box对象跟踪绘制状态。(`BoxDrawingView.java`)

		```java
        public class BoxDrawingView extends View {
            public static final String TAG = "BoxDrawingView";

            private Box mCurrentBox;
            private List<Box> mBoxen = new ArrayList<>();
            ...
            @Override
            public boolean onTouchEvent(MotionEvent event) {
                PointF current = new PointF(event.getX(), event.getY());
                String action = "";

                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        action = "ACTION_DOWN";
                        **// Reset drawing state
                        mCurrentBox = new Box(current);
                        mBoxen.add(mCurrentBox);**
                        break;
                    case MotionEvent.ACTION_MOVE:
                        action = "ACTION_MOVE";
                        **if (mCurrentBox != null) {
                            mCurrentBox.setCurrent(current);
                            invalidate();
                        }**
                        break;
                    case MotionEvent.ACTION_UP:
                        action = "ACTION_UP";
                        **mCurrentBox = null;**
                        break;
                    case MotionEvent.ACTION_CANCEL:
                        action = "ACTION_CANCEL";
                        **mCurrentBox = null;**
                        break;
                }

                Log.i(TAG, action + " at x=" + current.x +
                        ", y=" + current.y);

                return true;
            }
        }
		```

		+ 只要接收到`ACTION_DOWN`动作事件，我们都以事件原始坐标新建Box对象并赋值给mCurrentBox，然后再添加到矩形框数组中。
		+ 用户手指在屏幕上移动时，mCurrentBox.mCurrent会得到更新。而在取消触摸事件或用户手指离开屏幕时，我们应清空mCurrentBox以结束屏幕绘制。已完成的Box会安全地存储在数组中，但它们再也不会受任何动作事件影响了。
		+ 注意`ACTION_MOVE`事件发生时调用的`invalidate()`方法。该方法会强制BoxDrawingView重新绘制自己。这样，用户在屏幕上拖曳时就能实时看到矩形框。这同时也引出我们接下来的任务：在屏幕上绘制矩形框。

+ `onDraw(...)`方法内的图形绘制
	+ 应用启动时，所有视图都处于无效状态。也就是说，视图还没有绘制到屏幕上。为解决这个问题，Android调用了顶级View视图的`draw()`方法。这将引起自上而下的链式调用反应，视图完成自我绘制，然后是子视图的自我绘制，再然后是子视图的子视图的自我绘制，如此调用下去直至继承结构的末端。当继承结构中的所有视图都完成自我绘制后，最顶级View视图也就不再无效了。
	+ 为参与这种绘制，可覆盖以下View方法：

		```java
		protected void onDraw(Canvas canvas)
		```

	+ 前面，在`onTouchEvent(...)`方法中响应`ACTION_MOVE`动作时，我们调用invalidate()方法再次让BoxDrawingView处于失效状态。这迫使它重新完成自我绘制，并再次调用`onDraw(...)`方法。
	+ Canvas和Paint是Android系统的两大绘制类。
		+ Canvas类具有需要的所有绘制操作。其方法可决定绘制的位置及图形，例如线条、圆形、字词、矩形等。
		+ Paint类决定如何进行绘制操作。其方法可指定绘制图形的特征，例如是否填充图形、使用什么字体绘制、线条是什么颜色等。
	+ 在`BoxDrawingView.java`的`XML构造方法`中创建两个Paint对象。

		```java
        public class BoxDrawingView extends View {
            private static final String TAG = "BoxDrawingView";

            private Box mCurrentBox;
            private List<Box> mBoxen = new ArrayList<>();

            private Paint mBoxPaint;
            private Paint mBackgroundPaint;
            ...
            // Used when inflating the view from XML
            public BoxDrawingView(Context context, AttributeSet attrs) {
                super(context, attrs);

                // Paint the boxes a nice semitransparent red (ARGB)
                mBoxPaint = new Paint();
                mBoxPaint.setColor(0x22ff0000);

                // Paint the background off-white
                mBackgroundPaint = new Paint();
                mBackgroundPaint.setColor(0xfff8efe0);
            }
        }
		```

	+ 有了Paint对象的支持，现在可将矩形框绘制到屏幕上了 (`BoxDrawingView.java`)

		```java
        public BoxDrawingView(Context context, AttributeSet attrs) {
            ...
        }

        @Override
        protected void onDraw(Canvas canvas) {
            // Fill the background
            canvas.drawPaint(mBackgroundPaint);

            for (Box box : mBoxen) {
                float left = Math.min(box.getOrigin().x, box.getCurrent().x);
                float right = Math.max(box.getOrigin().x, box.getCurrent().x);
                float top = Math.min(box.getOrigin().y, box.getCurrent().y);
                float bottom = Math.max(box.getOrigin().y, box.getCurrent().y);

                canvas.drawRect(left, top, right, bottom, mBoxPaint);
            }
        }
		```

		+ 以上代码的第一部分简单直接：使用米白背景paint，填充canvas以衬托矩形框。
		+ 然后，针对矩形框数组中的每一个矩形框，通过其两点坐标，确定矩形框上下左右的位置。绘制时，左端和顶端的值将作为最小值，右端和底端的值作为最大值。
		+ 完成位置坐标值计算后，调用`Canvas.drawRect(...)`方法，在屏幕上绘制红色的矩形框。

+ 挑战练习：设备旋转问题；box旋转问题。

























































































































