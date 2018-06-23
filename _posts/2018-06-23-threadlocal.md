---
layout: article
title: ThreadLocal源码分析
key: 20180623
tags: android
---

## ThreadLocal

### 背景
在android的消息队列中，每个线程都可以创建一个Looper，那么在android中是如何管理Looper和线程的关系呢？在Looper.prepare()方法中可以看到，这里通过了一个ThreadLocal进行每个线程中Looper的管理。

```java
private static void prepare(boolean quitAllowed) {
	if (sThreadLocal.get() != null) {
		throw new RuntimeException("Only one Looper may be created per thread");
	}
	sThreadLocal.set(new Looper(quitAllowed));
}
```

ThreadLocal是怎样工作的，是怎样保证不同线程中变量存取正确的，下面从源码的角度看一看。

<!--more-->

### ThreadLocal

下面从源码角度来看看ThreadLocal实现了什么样的功能，从set()方法看起，在set()方法中首先获取了当前调用set()方法的线程，然后通过这个线程获取了一个ThreadLocalMap对象，如果存在就把当前的ThreadLocal和需要储存的value放到这个ThreadLocalMap对象中，如果不存在就新建一个ThreadLocalMap把当前thread和value存进去。代码如下

```java
public void set(T value) {
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null)
		map.set(this, value);
	else
		createMap(t, value);
}
```
这里看到creatMap()方法并没有以当前的ThreadLocal为key进行存储，看看createMap()就不奇怪了，在createMap()方法中还是通过当前的ThreadLocal进行存储的，代码如下

```java
void createMap(Thread t, T firstValue) {
	t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

有set()就会有get()，下面来看看get()方法。首先还是先通过当前线程thread对象获取ThreadLocalMap，如果map存在再通过当前ThreadLocal作为key进行查找，如果存在的话就返回对应的值，如果不存在则进行一次对value的初始化，代码如下

```java
public T get() {
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null) {
		ThreadLocalMap.Entry e = map.getEntry(this);
		if (e != null) {
			@SuppressWarnings("unchecked")
			T result = (T)e.value;
			return result;
		}
	}
	return setInitialValue();
}
```

可以看到调用get()方法后，如果当前ThreadLocal对应的value不存在，就会进行一次初始化，在setInitialValue()中第一步就对value进行了初始化，初始化的同时，还是根据当前线程去生成了一个ThreadLocalMap，并进行和set()方法一样的初始化，然后返回value，代码如下

```java
private T setInitialValue() {
	T value = initialValue();
	Thread t = Thread.currentThread();
	ThreadLocalMap map = getMap(t);
	if (map != null)
		map.set(this, value);
	else
		createMap(t, value);
	return value;
}
```

在初始化value时调用了initialValue()，这里直接暴力的返回了null，但是可以看到这个方法是protected的，应该是方便继承时自己定义初始化值使用的，如果直接重写setInitialValue()是非常不安全的。initialValue()代码如下

```java
protected T initialValue() {
	return null;
}
```

上述方法还会调到getMap()方法，这个方法也很简单，直接返回和当前thread绑定的threadLocals，代码如下。

```java
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```

以上就是ThreadLocal存取数据的主要代码，上面一直提到ThreadLocalMap，下面来看看它究竟是什么。

#### ThreadLocalMap
ThreadLocalMap是ThreadLocal的一个静态内部类，从命名上猜测，它的作用应该是类似于Java中的Map。首先在它的内部定义了一个Entry类，定义了一个key-value格式的映射，key是ThreadLocal，value是任意类型，代码如下

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
	
	Object value;
	
	Entry(ThreadLocal<?> k, Object v) {
		super(k);
		value = v;
	}
}
```

下面看看ThreadLocalMap中default的构造方法，这个构造方法只有在ThreadLocalMap第一次创建的时候调用，即调用createMap()方法时调用，可以看到在构造方法内对Entry进行了初始化，其中的table是一个Entry数组，并传出一个默认为16的长度，通过ThreadLocal计算出一个hash值，并存在数组中下表的位置，这里和HashMap的实现原理类似，代码如下。

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
	table = new Entry[INITIAL_CAPACITY];
	int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
	table[i] = new Entry(firstKey, firstValue);
	size = 1;
	setThreshold(INITIAL_CAPACITY);
}
```

在ThreadLocal中获取entry时类似HashMap的取值方法，通过Hash值计算出下标，从数组中取出对应entry，代码如下

```java
private Entry getEntry(ThreadLocal<?> key) {
	int i = key.threadLocalHashCode & (table.length - 1);
	Entry e = table[i];
	if (e != null && e.get() == key)
		return e;
	else
		return getEntryAfterMiss(key, i, e);
}
```

set()的时候也很简单，如果key存在则替换value，不存在则通过replaceStaleEntry()去新增一个，代码如下

```java
private void set(ThreadLocal<?> key, Object value) {
	
	Entry[] tab = table;
	int len = tab.length;
	int i = key.threadLocalHashCode & (len-1);
	
	for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
		ThreadLocal<?> k = e.get();
		
		if (k == key) {
		    e.value = value;
		    return;
		}
		
		if (k == null) {
		    replaceStaleEntry(key, value, i);
		    return;
		}
	}
	
	tab[i] = new Entry(key, value);
	int sz = ++size;
	if (!cleanSomeSlots(i, sz) && sz >= threshold)
		rehash();
}
```

### 小结
以上就是ThreadLocal中主要的几个方法，通过ThreadLocal和value进行映射，对数据进行存储，用起来十分简单和方便，在处理多线程中变量储存的时候可以起到很好的效果。
