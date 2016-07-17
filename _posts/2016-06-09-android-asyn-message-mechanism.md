---
layout: post
title: "Android异步消息处理机制"
category: "笔记"
tags: "Android"
keywords: ["Android","异步消息","Looper","Handler", "Message"]
description: "深入理解Looper、Handler&Message"
---
{% include JB/setup %}

#### 深入理解Looper、Handler、Message三者关系

---

本节出自：[张鸿洋的博客](http://blog.csdn.net/lmj623565791/article/details/38377229)

#### 1、概述

Handler 、 Looper 、Message 这三者都是与 Android 异步消息处理线程相关的概念。那么，什么叫异步消息处理线程呢？
异步消息处理线程启动后会进入一个无限的循环体之中，每循环一次，从其内部的消息队列中取出一个消息，然后回调相应的消息处理函数，执行完成一个消息后则继续循环。若消息队列为空，线程则会阻塞等待。
说了这一堆，那么和Handler 、 Looper 、Message有啥关系？其实Looper负责的就是创建一个MessageQueue，然后进入一个无限循环体不断从该MessageQueue中读取消息，而消息的创建者就是一个或多个Handler 。

#### 2、源码解析

+ 1、Looper

	对于Looper主要是`prepare()`和`loop()`两个方法。
	首先看`prepare()`方法：

	```java
	private static void prepare() {
		if (sThreadLocal.get() != null) {
			throw new RuntimeException("Only one Looper may be created per thread");
		}
		sThreadLocal.set(new Looper(true));
	}
	```

	`sThreadLocal`是一个ThreadLocal对象，可以在一个线程中存储变量。可以看到，在倒数第2行，将一个Looper的实例放入了ThreadLocal，并且2-4行判断了`sThreadLocal`是否为null，否则抛出异常。这也就说明了`Looper.prepare()`方法不能被调用两次，同时也保证了一个线程中只有一个Looper实例。

	下面看Looper的构造方法：

	```java
	private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
		mRun = true;
        mThread = Thread.currentThread();
    }
	```

	在构造方法中，创建了一个`MessageQueue`（消息队列）。
	然后我们看`loop()`方法：

	```java
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }

        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
	```

	第2行的`myLooper()`方法：

	```java
	public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
	```

	该方法直接返回了`sThreadLocal`存储的Looper实例，如果me为null则抛出异常，也就是说`loop()`方法必须在`prepare()`方法之后运行。
	第6行(`me.mQueue`)：拿到该looper实例中的mQueue（消息队列）。
	第13到45行(`for(;;){}`)：就进入了我们所说的无限循环。
	第14行(`queue.next()`)：取出一条消息，如果没有消息则阻塞。
	第27行：调用 `msg.target.dispatchMessage(msg)`方法，即将消息交给msg的target的`dispatchMessage()`方法去处理。Msg的target是什么呢？其实就是handler对象，下面会进行分析。
	第44行：释放消息占据的资源。

	因此，Looper的主要作用：
	+ 1、与当前线程绑定，保证一个线程只会有一个Looper实例，同时一个Looper实例也只有一个MessageQueue。
	+ 2、`loop()`方法，不断从MessageQueue中去取出消息，交给消息的target属性的`dispatchMessage()`方法去处理。

	好了，我们的异步消息处理线程已经有了消息队列（MessageQueue），也有了在无限循环体中取出消息的`Looper`，现在缺的就是发送消息的对象了，于是乎：Handler登场了。

+ 2、Handler

	使用Handler之前，我们都是初始化一个实例，比如用于更新UI线程，我们会在声明的时候直接初始化，或者在`onCreate()`方法中初始化Handler实例。所以我们首先看Handler的构造方法，看其如何与MessageQueue联系上的，它在子线程中发送的消息（一般发送消息都在非UI线程）是怎么发送到MessageQueue中的。

	```java
	public Handler() {
        this(null, false);
    }

	public Handler(Callback callback, boolean async) {
		if (FIND_POTENTIAL_LEAKS) {
			final Class<? extends Handler> klass = getClass();
			if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) && (klass.getModifiers() & Modifier.STATIC) == 0) {
				Log.w(TAG, "The following Handler class should be static or leaks might occur: " + klass.getCanonicalName());
				}
			}

			mLooper = Looper.myLooper();
			if (mLooper == null) {
				throw new RuntimeException(
					"Can't create handler inside thread that has not called Looper.prepare()");
			}
			mQueue = mLooper.mQueue;
			mCallback = callback;
			mAsynchronous = async;
		}
	```

	第14行(`Looper.myLooper()`)：获取了当前线程保存的Looper实例，然后在第19行(`mLooper.mQueue`)又获取了这个Looper实例中保存的MessageQueue（消息队列），这样就保证了handler的实例与我们Looper实例中MessageQueue关联上了。

	然后看我们最常用的`sendMessage()`方法：

	```java
	public final boolean sendMessage(Message msg){
        return sendMessageDelayed(msg, 0);
    }

    public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageDelayed(msg, delayMillis);
    }

    public final boolean sendMessageDelayed(Message msg, long delayMillis){
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
	```

	辗转反侧最后调用了`sendMessageAtTime()`，在此方法内部又直接获取MessageQueue，然后调用了`enqueueMessage()`方法，我们再来看看此方法：

	```java
	private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
	```

	`enqueueMessage()`中首先为`msg.target`赋值为this，【如果大家还记得Looper的`loop()`方法会取出每个msg，然后交给`msg.target.dispatchMessage(msg)`去处理消息】，也就是把当前的handler作为msg的target属性。最终会调用queue的`enqueueMessage()`的方法，也就是说handler发出的消息，最终会保存到消息队列中去。

	现在已经很清楚了Looper会调用`prepare()`和`loop()`方法，在当前执行的线程中保存一个Looper实例，这个实例会保存一个MessageQueue对象，然后当前线程进入一个无限循环中去，不断从MessageQueue中读取Handler发来的消息。然后再回调创建这个消息的handler中的`dispathMessage()`方法，下面我们赶快去看一看这个方法：

	```java
	public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
	```

	可以看到，第10行(`handleMessage(msg)`)，调用了`handleMessage()`方法，下面我们去看这个方法：

	```java
	/**
     * Subclasses must implement this to receive messages.
     */
    public void handleMessage(Message msg) {
    }
	```

	可以看到这是一个空方法，为什么呢？因为消息的最终回调是由我们控制的，我们在创建handler实例的时候都是复写`handleMessage()`方法，然后根据`msg.what`进行消息处理。
	例如：

	```java
	private Handler mHandler = new Handler(){
        public void handleMessage(android.os.Message msg){
            switch (msg.what){
            	case value:
                	break;
            	default:
                	break;
            }
        };
    };
	```

	到此，这个流程已经解释完毕，让我们首先总结一下：
	+ 1、首先`Looper.prepare()`在本线程中保存一个Looper实例，然后该实例中保存一个MessageQueue对象；因为`Looper.prepare()`在一个线程中只能调用一次，所以MessageQueue在一个线程中只会存在一个。
	+ 2、`Looper.loop()`会让当前线程进入一个无限循环，不断地从MessageQueue的实例中读取消息，然后回调`msg.target.dispatchMessage(msg)`方法。
	+ 3、Handler的构造方法，首先会得到当前线程中保存的Looper实例，进而与Looper实例中的MessageQueue相关联。
	+ 4、Handler的`sendMessage()`方法，会给msg的target赋值为handler自身，然后加入MessageQueue中。
	+ 5、在构造Handler实例时，我们会重写`handleMessage()`方法，也就是`msg.target.dispatchMessage(msg)`最终调用的方法。

	好了，总结完成，大家可能还会问，那么在Activity中，我们并没有显式地调用`Looper.prepare()`和`Looper.loop()`方法，为啥Handler可以成功创建呢？这是因为在Activity的启动代码中，已经在当前UI线程调用了`Looper.prepare()`和`Looper.loop()`方法(`当前线程也是一个消息循环`)。

+ 3、Handler post

	今天有人问我，你说Handler的`post()`方法创建的线程和UI线程有什么关系？
	其实这个问题也是出现这篇博客的原因之一。这里需要说明，有时候为了方便，我们会直接写如下代码：

	```java
	mHandler.post(new Runnable(){
		@Override
		public void run(){
			Log.e("TAG", Thread.currentThread().getName());
			mTxt.setText("yoxi");
		}
	});
	```

	然后`run()`方法中可以写更新UI的代码，其实这个Runnable并没有创建什么线程，而是发送了一条消息，下面看源码：

	```java
	public final boolean post(Runnable r){
       	return  sendMessageDelayed(getPostMessage(r), 0);
    }
	...
	private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
	```

	可以看到，在`getPostMessage()`中，得到了一个Message对象，然后将我们创建的Runable对象作为callback属性，赋值给了此Message对象。
	注：产生一个Message对象，可以new ，也可以使用`Message.obtain()`方法；两者都可以，但是更建议使用`obtain()`方法，因为Message内部维护了一个Message池用于Message的复用，避免使用new 重新分配内存。
	`post()`方法中的`sendMessageDelayed(getPostMessage(r), 0)`方法，最终和`handler.sendMessage()`方法一样，调用了`sendMessageAtTime()`，然后调用了`enqueueMessage()`方法，给`msg.target`赋值为`handler`，最终加入MessagQueue。
	可以看到，这里msg的callback和target都有值，那么会执行哪个呢？
	其实上面已经贴过代码，就是`dispatchMessage()`方法：

	```java
	public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
	```

	第2行(`(msg.callback != null) `)，如果不为null，则执行callback回调，也就是我们的Runnable对象。

	好了，关于Looper , Handler , Message 这三者关系上面已经叙述的非常清楚了。最后来张图解：

	![Looper、Handler和Message.png](https://raw.githubusercontent.com/tendoasan/MarkDownPhotos/master/Res/Looper%E3%80%81Handler%E5%92%8CMessage.jpg)

+ 4、后话

其实Handler不仅可以更新UI，你完全可以在一个子线程中去创建一个Handler，然后使用这个handler实例在任何其他线程中发送消息，最终处理消息的代码都会在你创建Handler实例的线程中运行。

```java
new Thread(){
	private Handler handler;
	public void run(){
		Looper.prepare();
		handler = new Handler(){
			public void handleMessage(android.os.Message msg){
				Log.e("TAG",Thread.currentThread().getName());
			};
        };
	}
}
```

Android不仅给我们提供了异步消息处理机制让我们更好的完成UI的更新，其实也为我们提供了异步消息处理机制代码的参考~~不仅能够知道原理，最好还可以将此设计用到其他的非Android项目中去~~

