---
layout: article
title: android消息队列源码分析
tags: android
---

### 背景

Handler是android中比较重要的机制，也是日常开发中也会经常遇到。另外，也是面试过程中必问的问题之一，所以不论在何种场景下都非常重要。为了加深了解，需要从源码角度去看看消息队列是如何实现的。

<!--more-->

### 源码分析

#### handler
在开发中我们最经常接触的就是handler中的post()方法，post()方法实现很简单，调用了另一个方法。

```java
public final boolean post(Runnable r) {
   return sendMessageDelayed(getPostMessage(r), 0);
}
```

其中getPostMessage(r)是获取一个message，message的实现稍后看。
```java
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```

接下来是一串调用，代码如下

```java
public final boolean sendMessageDelayed(Message msg, long delayMillis) {
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

最终调用到sendMessageAtTime()这个方法，最终调用了enqueueMessage()，看方法名应该是讲message加入消息队列。

```java
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

在enqueueMessage()中调用了enqueueMessage.enqueueMessage()方法，这里真正将message加入到MessageQueue的消息队列中。

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

#### Message
上面提到Message，Message是在消息队列中传递的对象，具体的成员变量如下

```java
public int what; //调用方自定义消息码，用来传递简单的事件

int flags; //当前Message的标记位，有三种FLAG_IN_USE，FLAG_ASYNCHRONOUS和FLAGS_TO_CLEAR_ON_COPY_FROM

long when; //消息执行的时间

Handler target; //和当前Message绑定的Handler

Runnable callback; //消息处理时的回调方法

Message next; //下一个Message，这里可以看出Message可以成为一个链表，Message中有一个Message Pool，最大数为50
```

#### MessageQueue

在Handler的最后一步中，调用了MessageQueue中的enqueueMessage()方法，下面看看具体是怎么实现的。当有消息需要进入队列的时候，如果当前MessageQueue中的Message如果为空，则将入队列的Message设为队列的第一个消息；如果Message不为空，则将新入队列的Message设为当前Message的next。

```java
boolean enqueueMessage(Message msg, long when) {
	//这里可以看到如果Message没有绑定handler的话会直接抛出异常
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }
    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        //mMessages为空，将入队列的Message设为队首消息
        if (p == null || when == 0 || when < p.when) {
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            //如果mMessages不为空，则将mMessages的next设为传入的Message
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (; ; ) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p;
            prev.next = msg;
        }

    }
    return true;
}
```

#### Looper

上面的代码完成一次将Message传入MessageQueue中的过程，那么系统中是如何读取这些消息的呢？在使用Handler的时候，如果不在主线程，需要调用如下代码

```
Looper.prepare()
Looper.loop()
```
分别来看看这两个方法做了什么事情。可以看到调用了prepare()会新建一个Looper，并存放在ThreadLocal中，代码如下

```
private static void prepare(boolean quitAllowed) {
	if (sThreadLocal.get() != null) {
		throw new RuntimeException("Only one Looper may be created per thread");
	}
	sThreadLocal.set(new Looper(quitAllowed));
}

```

之后需要调用的loop()会真正开始消息队列的循环，会遍历Message中所有的消息，并通过dispatchMessage()分发Message，下面是核心代码。

```java
public static void loop() {
	final Looper me = myLooper();
	//这里可以看到在调用loop()之前，必须调用prepare()方法，否则会抛出异常，因为无法指定在某个线程中去循环消息队列
	if (me == null) {
		throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
	}
	final MessageQueue queue = me.mQueue;

	for (;;) {
		Message msg = queue.next();
		if (msg == null) {
			return;
		}
		try {
			msg.target.dispatchMessage(msg);
		} finally {
		
		}
       
       msg.recycleUnchecked();
    }
}
```

以上就是android中消息队列从发出Message到消费的完成过程。

#### ActivityThread

从学Java开始，我们在写"hello, world"的时候知道，执行代码需要一个入口方法，就是```main(String[] args)```方法，那么在android中有没有这个main方法呢？这就需要对activity的执行过程有一定了解。
在android中确实有main()方法，在ActivityThread中，不要被这个类名迷惑，这个类和Thread无关，ActivityThread的代码相当长，直接看main()方法，代码如下。

```java
public static void main(String[] args) {
	Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
		
	CloseGuard.setEnabled(false);
		
	Environment.initForCurrentUser();
		
	EventLogger.setReporter(new EventLoggingReporter());
		
	final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
	
	TrustedCertificateStore.setDefaultUserDirectory(configDir);
		
	Process.setArgV0("<pre-initialized>");
		
	Looper.prepareMainLooper();
		
	ActivityThread thread = new ActivityThread();
	thread.attach(false);
		
	if (sMainThreadHandler == null) {
		sMainThreadHandler = thread.getHandler();
	}
		
	if (false) {
		Looper.myLooper().setMessageLogging(new
				LogPrinter(Log.DEBUG, "ActivityThread"));
	}
		
	Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
	Looper.loop();
		
	throw new RuntimeException("Main thread loop unexpectedly exited");
}
```
在这里可以看到调用了```Looper.prepareMainLooper()```和```Looper.loop()```，其中prepareMainLooper()与prepare()一致，前者是生成一个主线程的Looper，二后者可以不在主线程。这里就是主线程的消息队列初始化的地方，我们常做的异步线程处理数据，然后post到主线程的操作，就是讲Message加入到这个队列中的。

### 小结

以上就是MainLooper如何创建，以及整个消息队列的工作机制，一张图简单总结一下

![](https://upload-images.jianshu.io/upload_images/150061-d869830822794750.png?imageMogr2/auto-orient/)
