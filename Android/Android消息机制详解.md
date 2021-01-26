# 开发者营地|Android消息机制详解

## 一、Handler基础

### 1. Handler简介

在Android中使用消息机制，通常就是指Handler机制。Handler是Android消息机制的上层接口。

Handler的使用过程很简单，通过它可以轻松地将一个任务切换到Handler所在的线程中去执行。通常情况下，Handler的使用场景就是更新UI。 

示例：

```java
public class Activity extends android.app.Activity {
    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            System.out.println(msg.what);
        }
    };
    @Override
    public void onCreate(Bundle savedInstanceState, PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                //...耗时操作
                Message message = Message.obtain();
                message.what = 1;
                mHandler.sendMessage(message);
            }
        }).start();
    }
}
```

在子线程中，进行耗时操作，执行完操作后，发送消息，通知主线程更新UI。这便是消息机制的典型应用场景。

### 2. Handler模型

Android消息机制中的五大概念：

- `ThreadLocal`：当前线程存储的数据仅能从当前线程取出。
- `MessageQueue`：具有时间优先级的消息队列（单链表）。因为单链表在插入和删除上比较有优势。主要功能向消息池投递消息`MessageQueue.enqueueMessage`和取走消息池的消息`MessageQueue.next`。
- `Looper`：保存在ThreadLocal中的轮询消息队列，不断循环执行`Looper.loop`，若有新的消息到来，从MessageQueue中读取消息，按分发机制将消息分发给目标处理者。
- `Handler`：具体处理逻辑的地方，主要功能向消息池发送各种消息事件`Handler.sendMessage`和处理相应消息事件`Handler.handleMessage`。
- `Message`：需要传递的消息，可以传递数据。

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095e29e507a873?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 3. Handler架构

**消息机制的运行流程**：

在子线程执行完耗时操作，当Handler发送消息时，将会调用`MessageQueue.enqueueMessage`，向消息队列中添加消息。当通过`Looper.loop`开启循环后，会不断地从线程池中读取消息，即调用`MessageQueue.next`，然后调用目标Handler（即发送该消息的Handler）的`dispatchMessage`方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用`handleMessage`方法，接收消息，处理消息。

![](http://upload-images.jianshu.io/upload_images/3985563-d7da4f5ba49f6887.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

**MessageQueue，Handler和Looper三者之间的关系**：

每个线程中只能存在一个Looper，Looper是保存在ThreadLocal中的。

主线程（UI线程）已经创建了一个Looper，所以在主线程中不需要再创建Looper，但是在其他线程中需要创建Looper。

每个线程中可以有多个Handler，即一个Looper可以处理来自多个Handler的消息。

Looper中维护一个MessageQueue，来维护消息队列，消息队列中的Message可以来自不同的Handler。

![](http://upload-images.jianshu.io/upload_images/3985563-88a27b5906166c63.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

下面是消息机制的整体架构图，接下来我们将慢慢解剖整个架构。

![](http://upload-images.jianshu.io/upload_images/3985563-6c25004471646c1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

从中我们可以看出：  

Looper有一个MessageQueue消息队列；  

MessageQueue有一组待处理的Message；  

Message中记录发送和处理消息的Handler；  

Handler中有Looper和MessageQueue。

接下来我们通过源码来分析Handler的原理。

## 二、Handler源码

### 1. Looper

要想使用消息机制，首先要创建一个Looper。  

#### 1.1 初始化Looper  

##### 1.1.1 普通线程初始化 

相关代码如下：

```java
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

	final MessageQueue mQueue;
	final Thread mThread;	

	public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
```

`prepare()`无参情况下，默认调用`prepare(true)`表示的是这个Looper可以退出，而对于false的情况则表示当前Looper不可以退出。

`sThreadLocal.get() != null`这里看出，只能创建一个，不能重复创建Looper，否则会抛出RuntimeException。

然后创建Looper，并保存在ThreadLocal。

其中ThreadLocal是线程本地存储区（Thread Local Storage，简称为TLS），每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的TLS区域。  

Looper中创建了MessageQueue，且保存了MessageQueue的引用。

```java
	// True if the message queue can be quit.
    private final boolean mQuitAllowed;	
	private long mPtr; // used by native code
    MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }

	private native static long nativeInit();
```
MessageQueue的初始化过程中保存了是否能退出这一状态，且调用了`nativeInit()`这一native层的函数，保存返回的指针引用。

##### 1.1.2 主线程初始化 

主线程中不需要自己创建Looper，这是由于在程序启动的时候，系统已经帮我们自动调用了`Looper.prepare()`方法。查看ActivityThread中的`main()`方法，代码如下所示：

```java
    public static void main(String[] args) {
        ...
        Looper.prepareMainLooper();
        
        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
        	sMainThreadHandler = thread.getHandler();
        }
        
        Looper.loop();
        ...
    }
```

Android主线程的`Looper#prepareMainLooper()`方法，我们看下其实现。

```java
    private static Looper sMainLooper;  // guarded by Looper.class
	
	public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }

    public static Looper getMainLooper() {
        synchronized (Looper.class) {
            return sMainLooper;
        }
    }
```

这里首先也是调用了`prepare(false)`方法，false表示当前Looper不可以退出，主线程的Handler是不允许退出的。

> `Looper.loop()` 是个死循环，后面的代码正常情况不会执行。因为主线程不允许退出，退出就意味 APP 要挂。

通过`myLooper()`方法获取主线程的Looper，赋值给sMainLooper。

我们注意到`Looper#getMainLooper()`方法供我们获取主线程的Looper。

#### 1.2 开启Looper

```java
public static void loop() {
    final Looper me = myLooper();  //获取TLS存储的Looper对象 
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;  //获取Looper对象中的消息队列

    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) { //进入loop的主循环方法
        Message msg = queue.next(); //可能会阻塞,因为next()方法可能会无限循环
        if (msg == null) { //消息为空，则退出循环
            return;
        }

        Printer logging = me.mLogging;  //默认为null，可通过setMessageLogging()方法来指定输出，用于debug功能
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
		
        final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
        final long end;
        
        try {
            msg.target.dispatchMessage(msg);//获取msg的目标Handler，然后用于分发Message 
            end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        if (slowDispatchThresholdMs > 0) {
            final long time = end - start;
            if (time > slowDispatchThresholdMs) {
                Slog.w(TAG, "...“);
            }
        }        

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
         	Log.wtf(TAG, "...”);
        }
        msg.recycleUnchecked(); 
    }
}
```

`loop()`进入循环模式，不断重复下面的操作，直到消息为空时退出循环:  

- 读取MessageQueue的下一条Message（关于next\(\)，后面详细介绍）;  

- 把Message分发给相应的target（Handler）。

**当next()取出下一条消息时，队列中已经没有消息时，next()会无限循环，产生阻塞。等待MessageQueue中加入消息，然后重新唤醒。**

#### 1.3 结束Looper

```java
    public void quit() {
        mQueue.quit(false);
    }

    public void quitSafely() {
        mQueue.quit(true);
    }

    void quit(boolean safe) {
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }

        synchronized (this) {
            if (mQuitting) {
                return;
            }
            mQuitting = true;

            if (safe) {
                removeAllFutureMessagesLocked();  //安全移除消息，移除未到当前时间的消息
            } else {
                removeAllMessagesLocked();		  //移除消息
            }
            // We can assume mPtr != 0 because mQuitting was previously false.
            nativeWake(mPtr);
        }
    }

    private native static void nativeWake(long ptr);
```

可以看到`quit()`和`quitSafely()`对应立即结束和安全结束方法。

特别地，对于主线程，在App退出时，ActivityThread中的 mH(Handler)收到消息后，才会执行退出。

```java
//ActivityThread.java
case EXIT_APPLICATION:
    if (mInitialApplication != null) {
        mInitialApplication.onTerminate();
    }
    Looper.myLooper().quit();
    break;
```

手动退出主线程Looper，便会抛出异常：

```
 Caused by: java.lang.IllegalStateException: Main thread not allowed to quit.
```

主线程不允许退出，一旦退出就意味着程序挂了。

### 2. Handler

#### 2.1 创建Handler

```java
    final Looper mLooper;
    final MessageQueue mQueue;
    final Callback mCallback;
    final boolean mAsynchronous;

    public Handler() {
        this(null, false);
    }

    public Handler(Callback callback, boolean async) {
        ...
        //必须先执行Looper.prepare()，才能获取Looper对象，否则为null.
        mLooper = Looper.myLooper();  //从当前线程的TLS中获取Looper对象
        if (mLooper == null) {
            throw new RuntimeException("Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue; //消息队列，来自Looper对象
        mCallback = callback;  //回调方法，默认为null
        mAsynchronous = async; //设置消息是否为异步处理方式
    }

    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```

对于Handler的无参构造方法，默认采用当前线程TLS中的Looper对象，并且callback回调方法为null，且消息为同步处理方式。只要执行的`Looper.prepare()`方法，那么便可以获取有效的Looper对象。

#### 2.2 其他构造方法

```java
    public Handler() {
        this(null, false);
    }

    public Handler(boolean async) {
        this(null, async);
    }

       public Handler(Callback callback) {
        this(callback, false);
    } 

    public Handler(Callback callback, boolean async){
        ...//同上
    }
```

以上，默认采用当前线程的Looper构建Handler。

如果要指定Looper，有如下构造方法。

```java
    public Handler(Looper looper) {
        this(looper, null, false);
    }

    public Handler(Looper looper, Callback callback) {
        this(looper, callback, false);
    }

    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }  
```

### 3. 发送消息

发送消息有几种方式，但是归根结底都是调用了`sendMessageAtTime()`方法。

在子线程中通过Handler的post\(\)方式或send\(\)方式发送消息，最终都是调用了`sendMessageAtTime()`方法。

#### 3.1 post方法

```java
    public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
    public final boolean postAtTime(Runnable r, long uptimeMillis)
    {
        return sendMessageAtTime(getPostMessage(r), uptimeMillis);
    }
    public final boolean postAtTime(Runnable r, Object token, long uptimeMillis)
    {
        return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
    }
    public final boolean postDelayed(Runnable r, long delayMillis)
    {
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }

    private static Message getPostMessage(Runnable r, Object token) {
        Message m = Message.obtain();
        m.obj = token;
        m.callback = r;
        return m;
    }
```

#### 3.2 send方法

```java
    public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
    public final boolean sendEmptyMessage(int what)
    {
        return sendEmptyMessageDelayed(what, 0);
    } 
    public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageDelayed(msg, delayMillis);
    }
    public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageAtTime(msg, uptimeMillis);
    }
    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```

#### 3.3 runOnUiThread\()

就连子线程中调用Activity中的runOnUiThread()中更新UI，其实也是发送消息通知主线程更新UI，最终也会调用`sendMessageAtTime()`方法。

```java
 public final void runOnUiThread(Runnable action) {
        if (Thread.currentThread() != mUiThread) {
            mHandler.post(action);
        } else {
            action.run();
        }
    }
```

如果当前的线程不等于UI线程\(主线程\)，就去调用Handler的post\(\)方法，最终会调用`sendMessageAtTime()`方法。否则就直接调用Runnable对象的run\(\)方法。

#### 3.4 sendMessageAtTime\(\) 

```java
 public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
       //其中mQueue是消息队列，从Looper中获取的
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        //调用enqueueMessage方法
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

```java
 private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        //调用MessageQueue的enqueueMessage方法
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

可以看到sendMessageAtTime\(\)\`方法的作用很简单，就是调用MessageQueue的enqueueMessage\(\)方法，往消息队列中添加一个消息。  

下面来看enqueueMessage\(\)方法的具体执行逻辑。  

#### 3.5 enqueueMessage()

```java
boolean enqueueMessage(Message msg, long when) {
    // 每一个Message必须有一个target
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }
    synchronized (this) {
        if (mQuitting) {  //正在退出时，回收msg，加入到消息池
            ...
            msg.recycle();
            return false;
        }
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            //p为null(代表MessageQueue没有消息） 或者msg的触发时间是队列中最早的， 则进入该该分支
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked; 
        } else {
            //将消息按时间顺序插入到MessageQueue。一般地，不需要唤醒事件队列，除非
            //消息队头存在barrier，并且同时Message是队列中最早的异步消息。
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
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
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

MessageQueue是按照Message触发时间的先后顺序排列的，队头的消息是将要最早触发的消息。

当有消息需要加入消息队列时，会从队列头开始遍历，直到找到消息应该插入的合适位置，以保证所有消息的时间顺序。

### 4. 获取消息

当发送了消息后，在MessageQueue维护了消息队列，然后在Looper中通过`loop()`方法，不断地获取消息。上面对`loop()`方法进行了介绍，其中最重要的是调用了`queue.next()`方法,通过该方法来提取下一条信息。下面我们来看一下`next()`方法的具体流程。  

#### 4.1 next()

```java
private native void nativePollOnce(long ptr, int timeoutMillis); /*non-static for callbacks*/

Message next() {
    final long ptr = mPtr;
    if (ptr == 0) { //当消息循环已经退出，则直接返回
        return null;
    }
    int pendingIdleHandlerCount = -1; // 循环迭代的首次为-1
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        
        //阻塞操作，当等待nextPollTimeoutMillis时长，或者消息队列被唤醒，都会返回
        nativePollOnce(ptr, nextPollTimeoutMillis);
        
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                //当消息Handler为空时，查询MessageQueue中的下一条异步消息msg，为空则退出循环。
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    //当异步消息触发时间大于当前时间，则设置下一次轮询的超时时长
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 获取一条消息，并返回
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    //设置消息的使用状态，即flags |= FLAG_IN_USE
                    msg.markInUse();
                    return msg;   //成功地获取MessageQueue中的下一条即将要执行的消息
                }
            } else {
                //没有消息
                nextPollTimeoutMillis = -1;
            }
         	//消息正在退出，返回null
            if (mQuitting) {
                dispose();
                return null;
            }
            //IdleHandler的处理
            ......
    }
}
```

**nativePollOnce**是阻塞操作，其中nextPollTimeoutMillis代表下一个消息到来前，还需要等待的时长；当nextPollTimeoutMillis = -1时，表示消息队列中无消息，会一直等待下去。  

可以看出`next()`方法根据消息的触发时间，获取下一条需要执行的消息,队列中消息为空时，则会进行阻塞操作。

### 5. 分发消息

在loop\(\)方法中，获取到下一条消息后，执行`msg.target.dispatchMessage(msg)`，来分发消息到目标Handler对象。  下面就来具体看下`dispatchMessage(msg)`方法的执行流程。  

#### 5.1 dispatchMessage()

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        //当Message存在回调方法，回调msg.callback.run()方法；
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //当Handler存在Callback成员变量时，回调方法handleMessage()；
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //Handler自身的回调方法handleMessage()
        handleMessage(msg);
    }
}

private static void handleCallback(Message message) {
   message.callback.run();
}
```

**分发消息流程**  ：

- 当Message的`msg.callback`不为空时，则回调方法`msg.callback.run()`;  

- 当Handler的`mCallback`不为空时，则回调方法`mCallback.handleMessage(msg)`；  

- 最后调用Handler自身的回调方法`handleMessage()`，该方法默认为空，Handler子类通过覆写该方法来完成具体的逻辑。

**消息分发的优先级**  ：

- Message的回调方法：`message.callback.run()`，优先级最高；  

- Handler中Callback的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；  

- Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。

- 对于很多情况下，消息分发后的处理方法是第3种情况，即`Handler.handleMessage()`，一般地往往通过覆写该方法从而实现自己的业务逻辑。

### 6. 移除消息

removeAllMessagesLocked()，实现如下:

```java
    private void removeAllMessagesLocked() {
        Message p = mMessages;
        while (p != null) {
            Message n = p.next;
            p.recycleUnchecked();
            p = n;
        }
        mMessages = null;
    }
```

立即删除消息，不管消息是否在当前时间应该执行。

removeAllFutureMessagesLocked()，实现如下:

````java
    private void removeAllFutureMessagesLocked() {
        final long now = SystemClock.uptimeMillis();
        Message p = mMessages;
        if (p != null) {
            if (p.when > now) {
                removeAllMessagesLocked();
            } else {
                Message n;
                for (;;) {
                    n = p.next;
                    if (n == null) {
                        return;
                    }
                    if (n.when > now) {
                        break;
                    }
                    p = n;
                }
                p.next = null;
                do {
                    p = n;
                    n = p.next;
                    p.recycleUnchecked();
                } while (n != null);
            }
        }
    }
````

等待当前时间应该执行的消息执行完成后删除所有消息。

removeCallbacksAndMessages()，实现如下:

```java
    //Handler
	public final void removeCallbacksAndMessages(Object token) {
        mQueue.removeCallbacksAndMessages(this, token);
    }

	//MessageQueue
    void removeCallbacksAndMessages(Handler h, Object object) {
        if (h == null) {
            return;
        }

        synchronized (this) {
            Message p = mMessages;

            // Remove all messages at front.
            while (p != null && p.target == h
                    && (object == null || p.obj == object)) {
                Message n = p.next;
                mMessages = n;
                p.recycleUnchecked();
                p = n;
            }

            // Remove all messages after front.
            while (p != null) {
                Message n = p.next;
                if (n != null) {
                    if (n.target == h && (object == null || n.obj == object)) {
                        Message nn = n.next;
                        n.recycleUnchecked();
                        p.next = nn;
                        continue;
                    }
                }
                p = n;
            }
        }
    }
```

移除所有的回调和消息。

### 7. 总结

以上便是消息机制的原理，以及从源码角度来解析消息机制的运行过程。可以简单地用下图来理解。

![](http://upload-images.jianshu.io/upload_images/3985563-b3295b67a2b0477f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)



## 三、IdleHandler

IdleHandler 可以用来提升性能，主要用在我们希望能够在当前线程消息队列空闲时做些事情，不过最好不要做耗时操作。具体用法如下。

```java
//getMainLooper().myQueue()或者Looper.myQueue()
Looper.myQueue().addIdleHandler(new IdleHandler() {  
    @Override  
    public boolean queueIdle() {  
        //你要处理的事情
        return false;    
    }  
});
```

关于 IdleHandler 在 MessageQueue 与 Looper 和 Handler 的关系原理源码分析如下：

```csharp
/**
 * 获取当前线程队列使用Looper.myQueue()，获取主线程队列可用getMainLooper().myQueue()
 */
public final class MessageQueue {
    ......
	Message mMessages;
    private final ArrayList<IdleHandler> mIdleHandlers = new ArrayList<IdleHandler>();
    
    /**
     * 当前队列将进入阻塞等待消息时调用该接口回调，即队列空闲
     */
    public static interface IdleHandler {
        /**
         * 返回true就是单次回调后不删除，下次进入空闲时继续回调该方法，false只回调单次。
         */
        boolean queueIdle();
    }

    /**
     * 判断当前队列是不是空闲的，辅助方法
     */
    public boolean isIdle() {
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            return mMessages == null || now < mMessages.when;
        }
    }

    /**
     * 添加一个IdleHandler到队列，如果IdleHandler接口方法返回false则执行完会自动删除，
     * 否则需要手动removeIdleHandler。
     */
    public void addIdleHandler(@NonNull IdleHandler handler) {
        if (handler == null) {
            throw new NullPointerException("Can't add a null IdleHandler");
        }
        synchronized (this) {
            mIdleHandlers.add(handler);
        }
    }

    /**
     * <p>This method is safe to call from any thread.
     * 删除一个之前添加的 IdleHandler。
     */
    public void removeIdleHandler(@NonNull IdleHandler handler) {
        synchronized (this) {
            mIdleHandlers.remove(handler);
        }
    }
    ......
    //Looper的prepare()方法会通过ThreadLocal准备当前线程的MessageQueue实例，
    //然后在loop()方法中死循环调用当前队列的next()方法获取Message。
    Message next() {
        ......
        for (;;) {
            ......
            nativePollOnce(ptr, nextPollTimeoutMillis);
            synchronized (this) {
                ......
                //把通过addIdleHandler添加的IdleHandler转成数组存起来在mPendingIdleHandlers中
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            //循环遍历所有IdleHandler
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    //调用IdleHandler接口的queueIdle方法并获取返回值。
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }
                //如果IdleHandler接口的queueIdle方法返回false说明只执行一次需要删除。
                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }
            ......
        }
    }
}
```

## 四、同步屏障机制

Message分为3中：普通消息（同步消息）、屏障消息（同步屏障）和异步消息。

我们通常使用的都是普通消息，而屏障消息就是在消息队列中插入一个屏障，在屏障之后的所有普通消息都会被挡着，不能被处理。不过异步消息却例外，屏障不会挡住异步消息，因此可以这样认为：屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809160420302.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N0YXJ0X21hbw==,size_16,color_FFFFFF,t_70)

### 1. 插入屏障消息

同步屏障是通过`MessageQueue#postSyncBarrier`方法插入到消息队列的。

```java
//MessageQueue#postSyncBarrier
public int postSyncBarrier() {
    return postSyncBarrier(SystemClock.uptimeMillis());
}

 private int postSyncBarrier(long when) {
        synchronized (this) {
            final int token = mNextBarrierToken++;
            //1、屏障消息和普通消息的区别是屏障消息没有tartget。
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;
            //2、根据时间顺序将屏障插入到消息链表中适当的位置
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            //3、返回一个序号，通过这个序号可以撤销屏障
            return token;
        }
    }
```

postSyncBarrier方法就是用来插入一个屏障到消息队列的，可以看到它很简单，从这个方法我们可以知道如下：

- 屏障消息和普通消息的区别在于屏障没有tartget，普通消息有target是因为它需要将消息分发给对应的target，而屏障不需要被分发，它就是用来挡住普通消息来保证异步消息优先处理的。
- 屏障和普通消息一样可以根据时间来插入到消息队列中的适当位置，并且只会挡住它后面的同步消息的分发。
- postSyncBarrier返回一个int类型的数值，通过这个数值可以撤销屏障。
- postSyncBarrier方法是私有的，如果我们想调用它就得使用反射。
- 插入普通消息会唤醒消息队列，但是插入屏障不会。

### 2. 屏障消息的工作原理

通过postSyncBarrier方法屏障就被插入到消息队列中了，那么屏障是如何挡住普通消息只允许异步消息通过的呢？我们知道MessageQueue是通过next方法来获取消息的。

```java
Message next() {
			//1、如果有消息被插入到消息队列或者超时时间到，就被唤醒，否则阻塞在这。
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {        
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {//2、遇到屏障  msg.target == null
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());//3、遍历消息链表找到最近的一条异步消息
                }
                if (msg != null) {
                	//4、如果找到异步消息
                    if (now < msg.when) {//异步消息还没到处理时间，就再等会（超时时间）
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        //异步消息到了处理时间，就从链表移除，返回它。
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // 如果没有异步消息就一直休眠，等待被唤醒。
                    nextPollTimeoutMillis = -1;
                }
			//。。。。
        }
    }
```

可以看到，在注释2如果碰到屏障就遍历整个消息链表找到最近的一条异步消息，在遍历的过程中只有异步消息才会被处理执行到 if (msg != null){}中的代码。可以看到通过这种方式就挡住了所有的普通消息。

### 3. 发送异步消息

Handler有几个构造方法，可以传入async标志为true，这样构造的Handler发送的消息就是异步消息。不过可以看到，这些构造函数都是hide的，正常我们是不能调用的，不过利用反射机制可以使用@hide方法。

```java
     /**
      * @hide
      */
    public Handler(boolean async) {}

    /**
     * @hide
     */
    public Handler(Callback callback, boolean async) { }

    /**
     * @hide
     */
    public Handler(Looper looper, Callback callback, boolean async) {}
```

当调用`handler.sendMessage(msg)`发送消息，最终会走到：

```java
	private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);//把消息设置为异步消息
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
    public boolean isAsynchronous() {
        return (flags & FLAG_ASYNCHRONOUS) != 0;
    }
```

可以看到如果这个handler的mAsynchronous为true就把消息设置为异步消息，设置异步消息其实也就是设置msg内部的一个标志。而这个mAsynchronous就是构造handler时传入的async。除此之外，还有一个公开的方法：

```java
	    Message message=Message.obtain();
        message.setAsynchronous(true);
        handler.sendMessage(message);
```

在发送消息时通过 `message.setAsynchronous(true)`将消息设为异步的，这个方法是公开的，我们可以正常使用。

### 4. 移除屏障

移除屏障可以通过MessageQueue的removeSyncBarrier方法：

```java
	//注释已经写的很清楚了，就是通过插入同步屏障时返回的token 来移除屏障
	/**
     * Removes a synchronization barrier.
     *
     * @param token The synchronization barrier token that was returned by
     * {@link #postSyncBarrier}.
     *
     * @throws IllegalStateException if the barrier was not found.
     *
     * @hide
     */
    public void removeSyncBarrier(int token) {
        // Remove a sync barrier token from the queue.
        // If the queue is no longer stalled by a barrier then wake it.
        synchronized (this) {
            Message prev = null;
            Message p = mMessages;
            //找到token对应的屏障
            while (p != null && (p.target != null || p.arg1 != token)) {
                prev = p;
                p = p.next;
            }
            final boolean needWake;
            //从消息链表中移除
            if (prev != null) {
                prev.next = p.next;
                needWake = false;
            } else {
                mMessages = p.next;
                needWake = mMessages == null || mMessages.target != null;
            }
            //回收这个Message到对象池中。
            p.recycleUnchecked();
			// If the loop is quitting then it is already awake.
            // We can assume mPtr != 0 when mQuitting is false.
            if (needWake && !mQuitting) {
                nativeWake(mPtr);//唤醒消息队列
            }
    }
```

### 5. 实战

测试代码如下：

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private Handler handler;
    private int token;

    public static final int MESSAGE_TYPE_SYNC=1;
    public static final int MESSAGE_TYPE_ASYN=2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initHandler();
        initListener();
    }

    private void initHandler() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Looper.prepare();
                handler=new Handler(){
                    @Override
                    public void handleMessage(Message msg) {
                        if (msg.what == MESSAGE_TYPE_SYNC){
                            Log.d("MainActivity","收到普通消息");
                        }else if (msg.what == MESSAGE_TYPE_ASYN){
                            Log.d("MainActivity","收到异步消息");
                        }
                    }
                };
                Looper.loop();
            }
        }).start();
    }

    private void initListener() {
        findViewById(R.id.btn_postSyncBarrier).setOnClickListener(this);
        findViewById(R.id.btn_removeSyncBarrier).setOnClickListener(this);
        findViewById(R.id.btn_postSyncMessage).setOnClickListener(this);
        findViewById(R.id.btn_postAsynMessage).setOnClickListener(this);
    }

    //往消息队列插入同步屏障
    @RequiresApi(api = Build.VERSION_CODES.M)
    public void sendSyncBarrier(){
        try {
            Log.d("MainActivity","插入同步屏障");
            MessageQueue queue=handler.getLooper().getQueue();
            Method method=MessageQueue.class.getDeclaredMethod("postSyncBarrier");
            method.setAccessible(true);
            token= (int) method.invoke(queue);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //移除屏障
    @RequiresApi(api = Build.VERSION_CODES.M)
    public void removeSyncBarrier(){
        try {
            Log.d("MainActivity","移除屏障");
            MessageQueue queue=handler.getLooper().getQueue();
            Method method=MessageQueue.class.getDeclaredMethod("removeSyncBarrier",int.class);
            method.setAccessible(true);
            method.invoke(queue,token);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //往消息队列插入普通消息
    public void sendSyncMessage(){
        Log.d("MainActivity","插入普通消息");
        Message message= Message.obtain();
        message.what=MESSAGE_TYPE_SYNC;
        handler.sendMessageDelayed(message,1000);
    }

    //往消息队列插入异步消息
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP_MR1)
    private void sendAsynMessage() {
        Log.d("MainActivity","插入异步消息");
        Message message=Message.obtain();
        message.what=MESSAGE_TYPE_ASYN;
        message.setAsynchronous(true);
        handler.sendMessageDelayed(message,1000);
    }

    @RequiresApi(api = Build.VERSION_CODES.M)
    @Override
    public void onClick(View v) {
        int id=v.getId();
        if (id == R.id.btn_postSyncBarrier) {
            sendSyncBarrier();
        }else if (id == R.id.btn_removeSyncBarrier) {
            removeSyncBarrier();
        }else if (id == R.id.btn_postSyncMessage) {
            sendSyncMessage();
        }else if (id == R.id.btn_postAsynMessage){
            sendAsynMessage();
        }
    }
    
}
```

## 五、Handler扩展

### 1. 内存泄漏

Handler 允许我们发送**延时消息**，如果在延时期间用户关闭了 Activity，那么该 Activity 会泄露。

这个泄露是因为 Message 会持有 Handler，而又因为 **Java 的特性，内部类会持有外部类**，使得 Activity 会被 Handler 持有，这样最终就导致 Activity 泄露。

解决该问题的最有效的方法是：**将 Handler 定义成静态的内部类，在内部持有 Activity 的弱引用，并及时移除所有消息**。

```java
private static class SafeHandler extends Handler {

    private WeakReference<HandlerActivity> ref;

    public SafeHandler(HandlerActivity activity) {
        this.ref = new WeakReference(activity);
    }

    @Override
    public void handleMessage(final Message msg) {
        HandlerActivity activity = ref.get();
        if (activity != null) {
            activity.handleMessage(msg);
        }
    }
}
```

并且再在 `Activity.onDestroy()` 前移除消息，加一层保障：

```java
@Override
protected void onDestroy() {
  safeHandler.removeCallbacksAndMessages(null);
  super.onDestroy();
}
```

注意：单纯的在 `onDestroy` 移除消息并不保险，因为 `onDestroy` 并不一定执行。

### 2. 创建 Message 实例的最佳方式

由于 Handler 极为常用，所以为了节省开销，Android 给 Message 设计了回收机制，所以我们在使用的时候尽量复用 Message ，减少内存消耗。

- 通过 Message 的静态方法 `Message.obtain()`  获取，源码如下：

```java
    private static final Object sPoolSync = new Object();
    private static Message sPool;
    private static int sPoolSize = 0;

    private static final int MAX_POOL_SIZE = 50;

    public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
   }

   public static Message obtain(Message orig) {
        Message m = obtain();
        m.what = orig.what;
        m.arg1 = orig.arg1;
        m.arg2 = orig.arg2;
        m.obj = orig.obj;
        m.replyTo = orig.replyTo;
        m.sendingUid = orig.sendingUid;
        if (orig.data != null) {
            m.data = new Bundle(orig.data);
        }
        m.target = orig.target;
        m.callback = orig.callback;

        return m;
    }

    public static Message obtain(Handler h) {
        Message m = obtain();
        m.target = h;

        return m;
    }

    public static Message obtain(Handler h, Runnable callback) {
        Message m = obtain();
        m.target = h;
        m.callback = callback;

        return m;
    }

    public static Message obtain(Handler h, int what) {
        Message m = obtain();
        m.target = h;
        m.what = what;

        return m;
    }


    public static Message obtain(Handler h, int what, Object obj) {
        Message m = obtain();
        m.target = h;
        m.what = what;
        m.obj = obj;

        return m;
    }

    public static Message obtain(Handler h, int what, int arg1, int arg2) {
        Message m = obtain();
        m.target = h;
        m.what = what;
        m.arg1 = arg1;
        m.arg2 = arg2;

        return m;
    }

    public static Message obtain(Handler h, int what,
            int arg1, int arg2, Object obj) {
        Message m = obtain();
        m.target = h;
        m.what = what;
        m.arg1 = arg1;
        m.arg2 = arg2;
        m.obj = obj;

        return m;
    }
```

从sPool中取一个Message，复制后返回。

- 通过 Handler 的公有方法 `handler.obtainMessage()` ，源码如下：

```java
    public final Message obtainMessage()
    {
        return Message.obtain(this);
    }

    public final Message obtainMessage(int what)
    {
        return Message.obtain(this, what);
    }
    
    public final Message obtainMessage(int what, Object obj)
    {
        return Message.obtain(this, what, obj);
    }

    public final Message obtainMessage(int what, int arg1, int arg2)
    {
        return Message.obtain(this, what, arg1, arg2);
    }
    
    public final Message obtainMessage(int what, int arg1, int arg2, Object obj)
    {
        return Message.obtain(this, what, arg1, arg2, obj);
    }
```

可以看到是其实就是第一个方式的封装。

### 3. 子线程使用Handler

示例代码：

```java
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_three);
        new Thread(new Runnable() {
            @Override
            public void run() {
                //创建Looper，MessageQueue
                Looper.prepare();
                new Handler().post(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(HandlerActivity.this,"toast",Toast.LENGTH_LONG).show();
                    }
                });
                //开始处理消息
                Looper.loop();
            }
        }).start();
    }

```

添加Looper.prepare()创建Looper，同时调用Looper.loop()方法开始处理消息。

> 在所有事情处理完成后应该调用quit方法来终止消息循环，否则这个子线程就会一直处于循环等待的状态，因此不需要的时候终止Looper，调用`Looper.myLooper().quit()`。

### 4. 妙用 Looper 机制

我们可以利用 Looper 的机制来帮助我们做一些事情：

- 将 Runnable post 到主线程执行；

- 利用 Looper 判断当前线程是否是主线程。

示例代码如下：

```java
public final class MainThread {

    private MainThread() {
    }

    private static final Handler HANDLER = new Handler(Looper.getMainLooper());
	
    //将 Runnable post 到主线程执行
    public static void run(@NonNull Runnable runnable) {
        if (isMainThread()) {
            runnable.run();
        }else{
            HANDLER.post(runnable);
        }
    }

    public static boolean isMainThread() {
        //利用 Looper 判断当前线程是否是主线程。
        return Looper.myLooper() == Looper.getMainLooper();
    }

}
```

### 5. epoll机制

Looper会在线程中不断的检索消息，如果是子线程的Looper死循环，一旦任务完成，用户应该手动退出，而不是让其一直休眠等待。而对于主线程，我们是绝不希望会被运行一段时间自己就退出，所以通过死循环来保证不会被退出。

主线程的死循环并不会特别消耗 CPU 资源。这就涉及到 Linux  pipe/epoll机制：简单说就是在主线程的 MessageQueue 没有消息时，便阻塞在 loop 的 `queue.next()` 中的 `nativePollOnce(`) 方法里，此时主线程会释放 CPU 资源进入休眠状态，直到下个消息到达或者有事务发生，通过往 pipe 管道写端写入数据来唤醒主线程工作。

这里采用的 epoll 机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。

[聊聊IO多路复用之select,poll,epoll详解](https://www.jianshu.com/p/dfd940e7fca2)

### 6.  ThreadLocal

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定线程存储数据，数据存储后，只能在指定的线程可以获取到存储的数据，对于其他线程则无法获取到数据。一般来说，当数据是以线程作为作用域并且不同线程有不同副本的时候，就可以考虑使用ThreadLocal。

Java中的实现是下面这样，Java 的实现里面也有一个 Map，叫做 ThreadLocalMap，不过持有 ThreadLocalMap 的不是 ThreadLocal，而是 Thread。Thread 这个类内部有一个私有属性 threadLocals，其类型就是 ThreadLocalMap，ThreadLocalMap 的 Key 是 ThreadLocal。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dc583d20e32447e962d91f663d89429~tplv-k3u1fbpfcp-watermark.image)

#### 6.1 整体

精简之后的代码如下：

```java
class Thread {
    //内部持有ThreadLocalMap
    ThreadLocal.ThreadLocalMap threadLocals;
}
class ThreadLocal<T>{
    public T get() {
        //首先获取线程持有的ThreadLocalMap
        ThreadLocalMap map = Thread.currentThread().threadLocals;
        //在ThreadLocalMap中查找变量
        Entry e = map.getEntry(this);
        return e.value;  
    }
    
	public void set(T value) {
        //首先获取线程持有的ThreadLocalMap
        ThreadLocalMap map = Thread.currentThread().threadLocals;
        //在ThreadLocalMap中设置变量
        if (map != null)
            map.set(this, value);
        else
            //创建ThreadLocalMap
            createMap(t, value);
    }
    
  static class ThreadLocalMap{
    //内部是数组而不是Map
    Entry[] table;
    //根据ThreadLocal查找Entry
    Entry getEntry(ThreadLocal key){
      //省略查找逻辑
    }
    //Entry定义
    static class Entry extends WeakReference<ThreadLocal<?>>{
		Object value;
		Entry(ThreadLocal<?> k, Object v) {
			super(k);
            value = v;
		}
    }
  }
}
```

#### 6.2 get

```java
    //ThreadLocal
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

   ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

	//ThreadLocal.ThreadLocalMap
    private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);
        Entry e = table[i];
        if (e != null && e.get() == key)
            return e;
        else
            return getEntryAfterMiss(key, i, e);
    }
```

#### 6.3 set

```java
    //ThreadLocal
	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

   ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

	//ThreadLocal.ThreadLocalMap
	ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }

    private void set(ThreadLocal<?> key, Object value) {

        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);

        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
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

在Java的实现方案中，ThreadLocal仅仅只是一个代理工具类，内部并不持有任何线程相关的数据，所有和线程相关的数据都存储在Thread里。

这样的设计从数据的亲缘性上来讲，ThreadLocalMap属于Thread也更加合理。**所以ThreadLocal的get方法，其实就是拿到每个线程独有的ThreadLocalMap**。

还有一个原因，就是**不容易产生内存泄漏**。Thread持有ThreadLocalMap，而且ThreadLocalMap里对ThreadLocal的引用还是弱引用，所以只要Thread对象可以被回收，那么ThreadLocalMap就能被回收。**Java的实现方案虽然看上去复杂一些，但是更安全**。

但是，如果在线程池中使用ThreadLocal可能会导致内存泄漏，原因是线程池中线程的存活时间太长，往往和程序都是同生共死的，这就意味着Thread持有的ThreadLocalMap一直都不会被回收，再加上ThreadLocalMap中的Entry对ThreadLocal是弱引用，所以只要ThreadLocal结束了自己的生命周期是可以被回收掉的。**但是Entry中的Value却是被Entry强引用的，所以即便Value的生命周期结束了，Value也是无法被回收的，从而导致内存泄漏**。

所以我们可以通过`try{} finally{}`方案来手动释放资源。


```java
    ExecutorService pools;
    ThreadLocal tl;

    pools.execute(()->{
      //ThreadLocal增加变量
      tl.set(obj);
      try {
        // 省略业务逻辑代码
      }finally {
        //手动清理ThreadLocal 
        tl.remove();
      }
    });
```

### 7. Handler锁相关

`enqueueMessage()`通过synchronized关键字保证线程安全。同时`messagequeue.next()`内部也会通过synchronized加锁，确保取的时候线程安全，同时插入也会加锁。





---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)



