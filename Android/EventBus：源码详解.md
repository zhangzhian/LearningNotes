# EventBus：源码详解

关于EventBus的基本使用已经在[《EventBus：基本使用详解》](https://blog.csdn.net/baidu_32237719/article/details/103863489)一文中进行了详细的阐述。

接下来，通过这篇文章来分析其源码。

> 源码基于**eventbus:3.1.1**版本

首先我们回顾一下主要流程：

1.注册订阅

```java
EventBus.getDefault().register(this);
```

2.事件处理

```java
@Subscribe(threadMode = ThreadMode.MainThread)
    public void  onNewsEvent(NewsEvent event){
        String message = event.getMessage();
        mTv_message.setText(message);
    }
```

3.发布事件

```java
 EventBus.getDefault().post(new NewsEvent("你好！"));
```

以上是EventBus的基本使用。接下来我们查看源码实现。

## 一、Subscribe注解

EventBus3.0 开始用`Subscribe`注解配置事件订阅方法，不再使用方法名了，例如：

```csharp
@Subscribe
public void handleEvent(String event) {
    // do something
}
```

其中事件类型可以是 Java 中已有的类型或者我们自定义的类型。

具体看下`Subscribe`注解的实现：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Subscribe {
    // 指定事件订阅方法的线程模式，即在那个线程执行事件订阅方法处理事件，默认为POSTING
    ThreadMode threadMode() default ThreadMode.POSTING;
    // 是否支持粘性事件，默认为false
    boolean sticky() default false;
    // 指定事件订阅方法的优先级，默认为0，如果多个事件订阅方法可以接收相同事件的，则优先级高的先接收到事件
    int priority() default 0;
}
```

所以在使用`Subscribe`注解时可以根据需求指定`threadMode`、`sticky`、`priority`三个属性。

其中`threadMode`属性有如下几个可选值：

- **ThreadMode.POSTING**，默认的线程模式，在那个线程发送事件就在对应线程处理事件，避免了线程切换，效率高。
- **ThreadMode.MAIN**，如在主线程（UI线程）发送事件，则直接在主线程处理事件；如果在子线程发送事件，则先将事件入队列，然后通过 Handler 切换到主线程，依次处理事件。
- **ThreadMode.MAIN_ORDERED**，无论在那个线程发送事件，都先将事件入队列，然后通过 Handler 切换到主线程，依次处理事件。
- **ThreadMode.BACKGROUND**，如果在主线程发送事件，则先将事件入队列，然后通过线程池依次处理事件；如果在子线程发送事件，则直接在发送事件的线程处理事件。
- **ThreadMode.ASYNC**，无论在那个线程发送事件，都将事件入队列，然后通过线程池处理。

源码如下：

```java
public enum ThreadMode {
    /**
     * Subscriber will be called directly in the same thread, which is posting the event. This is the default. Event delivery
     * implies the least overhead because it avoids thread switching completely. Thus this is the recommended mode for
     * simple tasks that are known to complete in a very short time without requiring the main thread. Event handlers
     * using this mode must return quickly to avoid blocking the posting thread, which may be the main thread.
     */
    POSTING,

    /**
     * On Android, subscriber will be called in Android's main thread (UI thread). If the posting thread is
     * the main thread, subscriber methods will be called directly, blocking the posting thread. Otherwise the event
     * is queued for delivery (non-blocking). Subscribers using this mode must return quickly to avoid blocking the main thread.
     * If not on Android, behaves the same as {@link #POSTING}.
     */
    MAIN,

    /**
     * On Android, subscriber will be called in Android's main thread (UI thread). Different from {@link #MAIN},
     * the event will always be queued for delivery. This ensures that the post call is non-blocking.
     */
    MAIN_ORDERED,

    /**
     * On Android, subscriber will be called in a background thread. If posting thread is not the main thread, subscriber methods
     * will be called directly in the posting thread. If the posting thread is the main thread, EventBus uses a single
     * background thread, that will deliver all its events sequentially. Subscribers using this mode should try to
     * return quickly to avoid blocking the background thread. If not on Android, always uses a background thread.
     */
    BACKGROUND,

    /**
     * Subscriber will be called in a separate thread. This is always independent from the posting thread and the
     * main thread. Posting events never wait for subscriber methods using this mode. Subscriber methods should
     * use this mode if their execution might take some time, e.g. for network access. Avoid triggering a large number
     * of long running asynchronous subscriber methods at the same time to limit the number of concurrent threads. EventBus
     * uses a thread pool to efficiently reuse threads from completed asynchronous subscriber notifications.
     */
    ASYNC
}
```

## 二、注册事件订阅方法

注册事件的方式如下：

```java
EventBus.getDefault().register(this);
```

### 1. 获取实例

其中`getDefault()`是一个单例模式，保证当前只有一个`EventBus`实例：

```java
    static volatile EventBus defaultInstance;
	/** Convenience singleton for apps using a process-wide EventBus instance. */
    public static EventBus getDefault() {
        EventBus instance = defaultInstance;
        if (instance == null) {
            synchronized (EventBus.class) {
                instance = EventBus.defaultInstance;
                if (instance == null) {
                    instance = EventBus.defaultInstance = new EventBus();
                }
            }
        }
        return instance;
    }
```

继续看`new EventBus()`，在这里又调用了`EventBus`的另一个构造函数来完成初始化：

```java
private static final EventBusBuilder DEFAULT_BUILDER = new EventBusBuilder();

public EventBus() {
	this(DEFAULT_BUILDER);
}

EventBus(EventBusBuilder builder) {
        logger = builder.getLogger();
        subscriptionsByEventType = new HashMap<>();
        typesBySubscriber = new HashMap<>();
        stickyEvents = new ConcurrentHashMap<>();
        mainThreadSupport = builder.getMainThreadSupport();
        mainThreadPoster = mainThreadSupport != null ? mainThreadSupport.createPoster(this) : null;
        backgroundPoster = new BackgroundPoster(this);
        asyncPoster = new AsyncPoster(this);
        indexCount = builder.subscriberInfoIndexes != null ? builder.subscriberInfoIndexes.size() : 0;
        subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
                builder.strictMethodVerification, builder.ignoreGeneratedIndex);
        logSubscriberExceptions = builder.logSubscriberExceptions;
        logNoSubscriberMessages = builder.logNoSubscriberMessages;
        sendSubscriberExceptionEvent = builder.sendSubscriberExceptionEvent;
        sendNoSubscriberEvent = builder.sendNoSubscriberEvent;
        throwSubscriberException = builder.throwSubscriberException;
        eventInheritance = builder.eventInheritance;
        executorService = builder.executorService;
    }
```

`DEFAULT_BUILDER`就是一个默认的EventBusBuilder：

```java
public class EventBusBuilder {
    //使用了一个CachedThreadPool 
    private final static ExecutorService DEFAULT_EXECUTOR_SERVICE = Executors.newCachedThreadPool();

    boolean logSubscriberExceptions = true;
    boolean logNoSubscriberMessages = true;
    boolean sendSubscriberExceptionEvent = true;
    boolean sendNoSubscriberEvent = true;
    boolean throwSubscriberException;
    boolean eventInheritance = true;
    boolean ignoreGeneratedIndex;
    boolean strictMethodVerification;
    ExecutorService executorService = DEFAULT_EXECUTOR_SERVICE;
    List<Class<?>> skipMethodVerificationForClasses;
    List<SubscriberInfoIndex> subscriberInfoIndexes;
    Logger logger;
    MainThreadSupport mainThreadSupport;
}
```

可以看到这里使用了newCachedThreadPool线程池，先留意一下。

如果有需要的话，我们也可以通过配置`EventBusBuilder`来更改`EventBus`的属性，例如用如下方式注册事件：

```java
EventBus.builder()
        .eventInheritance(false)
        .logSubscriberExceptions(false)
        .build()
        .register(this);
```

### 2. 注册

首先我们关注一下EventBus类的变量声明：

```java
    /** Map<订阅事件类型, 订阅该事件类型的订阅（包含订阅者和该订阅者的订阅方法）集合> */
    private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
    /** Map<订阅者, 订阅事件类型集合> */
    private final Map<Object, List<Class<?>>> typesBySubscriber;
    /** Map<订阅事件类型,订阅事件实例对象>. */
    private final Map<Class<?>, Object> stickyEvents;

	private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
        @Override
        protected PostingThreadState initialValue() {
            return new PostingThreadState();
        }
    };
```

我们看到有三个Map，从Map的定义和名称上我们猜测和事件的存储相关，具体后面会看。

同时还使用到了ThreadLocal保持线程独有的一些数据。

接下来我们步入正题，看`regiser`：

```java
    public void register(Object subscriber) {
        // 订阅者类型
        Class<?> subscriberClass = subscriber.getClass();
        // 获取订阅者全部的响应函数信息
        // 根据Class查找当前类中订阅了事件的方法集合，即使用了Subscribe注解、有public修饰符、一个参数的方法
        // SubscriberMethod类主要封装了符合条件方法的相关信息：
        // Method对象、线程模式、事件类型、优先级、是否是粘性事等
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            //循环每一个事件响应函数，执行 subscribe()方法，更新订阅相关信息
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```

可以看到`register()`方法主要分为查找和注册两部分。

##### 查找

首先来看查找的过程，从`findSubscriberMethods()`开始：

```java
    //<订阅者,对应的方法>
    private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();

    List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
        // METHOD_CACHE是一个ConcurrentHashMap，
        // 直接保存了subscriberClass和对应SubscriberMethod的集合，以提高注册效率，赋值重复查找。
        List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
        if (subscriberMethods != null) {
            return subscriberMethods;
        }
        // 由于使用了默认的EventBusBuilder
        // 则ignoreGeneratedIndex属性默认为false，即不忽略注解生成器
        if (ignoreGeneratedIndex) {
            subscriberMethods = findUsingReflection(subscriberClass);
        } else {
            subscriberMethods = findUsingInfo(subscriberClass);
        }
        // 如果对应类中没有符合条件的方法，则抛出异常
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass
                    + " and its super classes have no public methods with the @Subscribe annotation");
        } else {
            // 保存查找到的订阅事件的方法
            METHOD_CACHE.put(subscriberClass, subscriberMethods);
            return subscriberMethods;
        }
    }


```

`findSubscriberMethods()`流程很清晰，即先从缓存中查找，如果找到则直接返回，否则去做下一步的查找过程，然后缓存查找到的集合，根据上边的注释可知`findUsingInfo()`方法会被调用：

```java
    private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
        FindState findState = prepareFindState();
        findState.initForSubscriber(subscriberClass);
        // 初始状态下findState.clazz就是subscriberClass
        while (findState.clazz != null) {
            findState.subscriberInfo = getSubscriberInfo(findState);
            // 条件不成立
            if (findState.subscriberInfo != null) {
                SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
                for (SubscriberMethod subscriberMethod : array) {
                    if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                        findState.subscriberMethods.add(subscriberMethod);
                    }
                }
            } else {
                // 通过反射查找订阅事件的方法
                findUsingReflectionInSingleClass(findState);
            }
            // 修改findState.clazz为subscriberClass的父类Class，即需要遍历父类
            findState.moveToSuperclass();
        }
        // 查找到的方法保存在了FindState实例的subscriberMethods集合中。
        // 使用subscriberMethods构建一个新的List<SubscriberMethod>
        // 释放掉findState
        return getMethodsAndRelease(findState);
    }
```

`findUsingInfo()`方法会在当前要注册的类以及其父类中查找订阅事件的方法，这里出现了一个`FindState`类，它是`SubscriberMethodFinder`的内部类，用来辅助查找订阅事件的方法，具体的查找过程在`findUsingReflectionInSingleClass()`方法，它主要通过反射查找订阅事件的方法：

```java
    private void findUsingReflectionInSingleClass(FindState findState) {
        Method[] methods;
        try {
            // This is faster than getMethods, especially when subscribers are fat classes like Activities
            methods = findState.clazz.getDeclaredMethods();
        } catch (Throwable th) {
            // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
            try {
                methods = findState.clazz.getMethods();
            } catch (LinkageError error) { // super class of NoClassDefFoundError to be a bit more broad...
                String msg = "Could not inspect methods of " + findState.clazz.getName();
                if (ignoreGeneratedIndex) {
                    msg += ". Please consider using EventBus annotation processor to avoid reflection.";
                } else {
                    msg += ". Please make this class visible to EventBus annotation processor to avoid reflection.";
                }
                throw new EventBusException(msg, error);
            }
            findState.skipSuperClasses = true;
        }
        // 循环遍历当前类的方法，筛选出符合条件的
        for (Method method : methods) {
            // 获得方法的修饰符
            int modifiers = method.getModifiers();
            // 如果是public类型，但非abstract、static等
            if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                // 获得当前方法所有参数的类型
                Class<?>[] parameterTypes = method.getParameterTypes();
                // 如果当前方法只有一个参数
                if (parameterTypes.length == 1) {
                    Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                    // 如果当前方法使用了Subscribe注解
                    if (subscribeAnnotation != null) {
                        // 得到该参数的类型
                        Class<?> eventType = parameterTypes[0];
                        // checkAdd()方法用来判断FindState的anyMethodByEventType map
                        // 是否已经添加过以当前eventType为key的键值对，没添加过则返回true
                        if (findState.checkAdd(method, eventType)) {
                            // 得到Subscribe注解的threadMode属性值，即线程模式
                            ThreadMode threadMode = subscribeAnnotation.threadMode();
                            // 创建一个SubscriberMethod对象，并添加到subscriberMethods集合
                            findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                    subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                        }
                    }
                } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                    String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                    throw new EventBusException("@Subscribe method " + methodName +
                            "must have exactly 1 parameter but has " + parameterTypes.length);
                }
            } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException(methodName +
                        " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
            }
        }
    }
```

到此`register()`方法中`findSubscriberMethods()`流程就分析完了，我们已经找到了当前注册类及其父类中订阅事件的方法的集合。

##### 注册

接下来分析具体的注册流程，即`register()`中的`subscribe()`方法：

```java
    // Must be called in synchronized block
    private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
        //获取订阅的事件类型
        Class<?> eventType = subscriberMethod.eventType;
        //把通过register()订阅的订阅者包装成Subscription对象以及当前的注册者的所有subscriberMethod
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
        //获取订阅该事件的订阅者集合
        // subscriptionsByEventType是一个HashMap，保存了以eventType为key,Subscription对象集合为value的键值对
        // 先查找subscriptionsByEventType是否存在以当前eventType为key的值
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        //订阅集合为空，创建新的集合，并把newSubscription 加入
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            //集合中已经有该订阅，抛出异常。不能重复订阅
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }
        //把新的订阅按照优先级加入到订阅集合中。
        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

        // 根据订阅者，获得该订阅者订阅的此事件类型集合
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        //如果事件集合为空，创建新的集合，并加入新订阅的事件类型。
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        //如果事件类型集合不为空，加入新订阅的此事件类型
        subscribedEvents.add(eventType);
        //该事件是stick=true。特殊处理.
        if (subscriberMethod.sticky) {
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                //循环获得每个stickyEvent事件
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    //是该类的父类
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        //该事件类型最新的事件发送给当前订阅者。
                        Object stickyEvent = entry.getValue();
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }
```

第一步：通过subscriptionsByEventType得到该事件类型所有订阅信息队列，根据优先级将当前订阅信息插入到订阅队列subscriptionsByEventType中；

第二步：在typesBySubscriber中得到当前订阅者订阅的所有该事件类型队列，将此事件类型保存到队列typesBySubscriber中，用于后续取消订阅；

第三步：检查这个事件是否是 Sticky 事件，如果是则从stickyEvents事件保存队列中取出该事件类型最后一个事件发送给当前订阅者。



### 3. 取消注册

接下来看，EventBus 如何取消注册：

```java
EventBus.getDefault().unregister(this);
```

核心的方法就是`unregister()`：

```java
    public synchronized void unregister(Object subscriber) {
        // 获取该订阅者所有的订阅事件类型集合
        List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
        if (subscribedTypes != null) {
            // 遍历参数类型集合，释放之前缓存的当前类中的Subscription
            for (Class<?> eventType : subscribedTypes) {
                unsubscribeByEventType(subscriber, eventType);
            }
            // 从typesBySubscriber删除该<订阅者对象,订阅事件类类型集合>
            typesBySubscriber.remove(subscriber);
        } else {
            logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + subscriber.getClass());
        }
    }

```

继续看`unsubscribeByEventType()`方法：

```java
    private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
        // 获取订阅事件类型对应的订阅信息集合.
        List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions != null) {
            int size = subscriptions.size();
            // 遍历Subscription集合
            for (int i = 0; i < size; i++) {
                Subscription subscription = subscriptions.get(i);
                // 从订阅者集合中删除特定的订阅者.
                if (subscription.subscriber == subscriber) {
                    subscription.active = false;
                    subscriptions.remove(i);
                    i--;
                    size--;
                }
            }
        }
    }
```

所以在`unregister()`方法中，释放了`typesBySubscriber`、`subscriptionsByEventType`中缓存的资源。

## 三、发送事件

当发送一个事件的时候，我们可以通过如下方式：

```java
EventBus.getDefault().post("Hello World!")
```

可以看到，发送事件就是通过`post()`方法完成的：

```java
public void post(Object event) {
    // currentPostingThreadState是一个PostingThreadState类型的ThreadLocal(ThreadLocal<PostingThreadState>)
    // 获取当前线程的Posting状态
    PostingThreadState postingState = currentPostingThreadState.get();
    // 获取当前线程的事件队列.
    List<Object> eventQueue = postingState.eventQueue;
    // 将当前事件添加到其事件队列
    eventQueue.add(event);
    // 判断新加入的事件是否在分发中，否的话执行，默认false
    if (!postingState.isPosting) {
        // 是否为主线程
        postingState.isMainThread = isMainThread();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            // 循环处理当前线程eventQueue中的每一个event对象
            while (!eventQueue.isEmpty()) {
                // 先移除事件，然后发送单个事件
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            // 处理完知乎重置postingState一些标识信息
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```

所以`post()`方法先将发送的事件保存的事件队列，然后通过循环出队列，将事件交给`postSingleEvent()`方法处理：

```java
   private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        //分发事件的类型
        Class<?> eventClass = event.getClass();
        boolean subscriptionFound = false;
        //默认为true，响应订阅事件的父类事件
        if (eventInheritance) {
            //找出当前订阅事件类类型eventClass的所有父类的类类型和其实现的接口的类类型
            List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
            int countTypes = eventTypes.size();
            // 遍历Class集合，继续处理事件
            for (int h = 0; h < countTypes; h++) {
                Class<?> clazz = eventTypes.get(h);
                //发布每个事件到每个订阅者
                subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
            }
        } else {
            subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
        }
        if (!subscriptionFound) {
            if (logNoSubscriberMessages) {
                logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
            }
            if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                    eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }
```

`postSingleEvent()`方法中，根据`eventInheritance`属性，决定是否向上遍历事件的父类型，然后用`postSingleEventForEventType()`方法进一步处理事件：

```java
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        // 获取订阅事件类类型对应的订阅者信息集合(register函数时构造的集合)
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    // 如果已订阅了对应类型的事件
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
            // 记录事件
            postingState.event = event;
            // 记录对应的订阅（含订阅者和订阅方法）
            postingState.subscription = subscription;
            boolean aborted;
            try {
                // 最终的事件处理
                // 发布订阅事件给订阅函数
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}
```

`postSingleEventForEventType()`方法核心就是遍历发送的事件类型对应的Subscription集合，然后调用`postToSubscription()`方法处理事件。

## 四、处理事件

接上一节，`postToSubscription`方法为事件处理的流程，内部会根据订阅事件方法的线程模式，间接或直接的以发送的事件为参数，通过反射执行订阅事件的方法。如下：

```java
   private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        // 判断订阅事件方法的线程模式
        switch (subscription.subscriberMethod.threadMode) {
            // 默认的线程模式，在那个线程发送事件就在那个线程处理事件
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            // 在主线程处理事件
            case MAIN:
                // 如果在主线程发送事件，则直接在主线程通过反射处理事件
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    // 如果是在子线程发送事件，则将事件入队列，通过Handler切换到主线程执行处理事件
                    // mainThreadPoster 不为空
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case MAIN_ORDERED:
                // 无论在那个线程发送事件，都先将事件入队列，然后通过 Handler 切换到主线程，依次处理事件。
                // mainThreadPoster 不为空
                if (mainThreadPoster != null) {
                    mainThreadPoster.enqueue(subscription, event);
                } else {
                    // temporary: technically not correct as poster not decoupled from subscriber
                    invokeSubscriber(subscription, event);
                }
                break;
            case BACKGROUND:
                // 如果在主线程发送事件，则先将事件入队列，然后通过线程池依次处理事件
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    // 如果在子线程发送事件，则直接在发送事件的线程通过反射处理事件
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                // 无论在那个线程发送事件，都将事件入队列，然后通过线程池处理。
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }
```

可以看到，`postToSubscription()`方法就是根据订阅事件方法的线程模式、以及发送事件的线程来判断如何处理事件，至于处理方式主要有两种：

### 1.invokeSubscriber

一种是在相应线程直接通过`invokeSubscriber()`方法，用反射来执行订阅事件的方法，这样发送出去的事件就被订阅者接收并做相应处理了：

```java
    void invokeSubscriber(Subscription subscription, Object event) {
        try {
           	 //反射
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
        } catch (InvocationTargetException e) {
            handleSubscriberException(subscription, event, e.getCause());
        } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
        }
    }
```

另外一种是先将事件入队列（其实底层是一个List），然后做进一步处理

### 2.mainThreadPoster.enqueue

我们首先以`mainThreadPoster.enqueue(subscription, event)`为例简单的分析下，其中`mainThreadPoster`是`HandlerPoster`类的一个实例，来看该类的主要实现：

`mainThreadPoster`定义如下：

```java
private final Poster mainThreadPoster;

interface Poster {

    /**
     * Enqueue an event to be posted for a particular subscription.
     *
     * @param subscription Subscription which will receive the event.
     * @param event        Event that will be posted to subscribers.
     */
    void enqueue(Subscription subscription, Object event);
}

```

`Poster`为一个接口，此处的实现为`HandlerPoster`:

```java
public class HandlerPoster extends Handler implements Poster {

    private final PendingPostQueue queue;
    private final int maxMillisInsideHandleMessage;
    private final EventBus eventBus;
    private boolean handlerActive;

    protected HandlerPoster(EventBus eventBus, Looper looper, int maxMillisInsideHandleMessage) {
        super(looper);
        this.eventBus = eventBus;
        this.maxMillisInsideHandleMessage = maxMillisInsideHandleMessage;
        queue = new PendingPostQueue();
    }

    public void enqueue(Subscription subscription, Object event) {
        // 用subscription和event封装一个PendingPost对象
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        synchronized (this) {
            // 入队列
            queue.enqueue(pendingPost);
            if (!handlerActive) {
                handlerActive = true;
                // 发送开始处理事件的消息，handleMessage()方法将被执行，完成从子线程到主线程的切换
                if (!sendMessage(obtainMessage())) {
                    throw new EventBusException("Could not send handler message");
                }
            }
        }
    }

    @Override
    public void handleMessage(Message msg) {
        boolean rescheduled = false;
        try {
            long started = SystemClock.uptimeMillis();
            // 死循环遍历队列
            while (true) {
                // 出队列
                PendingPost pendingPost = queue.poll();
                if (pendingPost == null) {
                    synchronized (this) {
                        // Check again, this time in synchronized
                        pendingPost = queue.poll();
                        if (pendingPost == null) {
                            handlerActive = false;
                            return;
                        }
                    }
                }
                // 处理pendingPost
                eventBus.invokeSubscriber(pendingPost);
                long timeInMethod = SystemClock.uptimeMillis() - started;
                if (timeInMethod >= maxMillisInsideHandleMessage) {
                    if (!sendMessage(obtainMessage())) {
                        throw new EventBusException("Could not send handler message");
                    }
                    rescheduled = true;
                    return;
                }
            }
        } finally {
            handlerActive = rescheduled;
        }
    }
}
```

所以`HandlerPoster`的`enqueue()`方法主要就是将`subscription`、`event`对象封装成一个`PendingPost`对象，然后保存到队列里，之后通过`Handler`切换到主线程，在`handleMessage()`方法将中将`PendingPost`对象循环出队列，交给`invokeSubscriber()`方法进一步处理：

```java
// EventBus#invokeSubscriber
void invokeSubscriber(PendingPost pendingPost) {
        Object event = pendingPost.event;
        Subscription subscription = pendingPost.subscription;
        // 释放pendingPost引用的资源
        PendingPost.releasePendingPost(pendingPost);
        if (subscription.active) {
            // 用反射来执行订阅事件的方法
            invokeSubscriber(subscription, event);
        }
    }
  
```

这个方法主要就是从`pendingPost`中取出之前保存的`event`、`subscription`，然后用反射来执行订阅事件的方法，又回到了第一种处理方式。所以`mainThreadPoster.enqueue(subscription, event)`的核心就是先将将事件入队列，然后通过Handler从子线程切换到主线程中去处理事件。

`backgroundPoster.enqueue()`和`asyncPoster.enqueue`也类似，内部都是先将事件入队列，然后再出队列，但是会通过线程池去进一步处理事件。

### 3. backgroundPoster.enqueue

`backgroundPoster.enqueue(subscription, event)`分析如下：

```java
    private final BackgroundPoster backgroundPoster;
```

`backgroundPoster`为`Poster`接口，此处的实现为`BackgroundPoster`:

```java
final class BackgroundPoster implements Runnable, Poster {

    private final PendingPostQueue queue;
    private final EventBus eventBus;

    private volatile boolean executorRunning;

    BackgroundPoster(EventBus eventBus) {
        this.eventBus = eventBus;
        queue = new PendingPostQueue();
    }

    public void enqueue(Subscription subscription, Object event) {
        //用subscription和event封装一个PendingPost对象
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        synchronized (this) {
            // 入队列
            queue.enqueue(pendingPost);
            if (!executorRunning) {
                executorRunning = true;
                //获取线程池并执行，线程池执行run方法，接下来看run方法
                eventBus.getExecutorService().execute(this);
            }
        }
    }

    @Override
    public void run() {
        try {
            try {
                while (true) {
                    //wait 1ms
                    PendingPost pendingPost = queue.poll(1000);
                    if (pendingPost == null) {
                        synchronized (this) {
                            // Check again, this time in synchronized
                            pendingPost = queue.poll();
                            if (pendingPost == null) {
                                executorRunning = false;
                                return;
                            }
                        }
                    }
                    //反射执行
                    eventBus.invokeSubscriber(pendingPost);
                }
            } catch (InterruptedException e) {
                eventBus.getLogger().log(Level.WARNING, Thread.currentThread().getName() + " was interruppted", e);
            }
        } finally {
            executorRunning = false;
        }
    }

}
```

这里使用了线程池，可以看到为CachedThreadPool。

```java
   //使用了一个CachedThreadPool
    private final static ExecutorService DEFAULT_EXECUTOR_SERVICE = Executors.newCachedThreadPool();
    
ExecutorService executorService = DEFAULT_EXECUTOR_SERVICE;

```

### 4.  asyncPoster.enqueue

`asyncPoster.enqueue(subscription, event)`分析如下：

```java
   private final AsyncPoster asyncPoster;
```

`asyncPoster`为`Poster`接口，此处的实现为`AsyncPoster`:

```java
class AsyncPoster implements Runnable, Poster {

    private final PendingPostQueue queue;
    private final EventBus eventBus;

    AsyncPoster(EventBus eventBus) {
        this.eventBus = eventBus;
        queue = new PendingPostQueue();
    }

    public void enqueue(Subscription subscription, Object event) {
        //用subscription和event封装一个PendingPost对象
        PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
        // 入队列
        queue.enqueue(pendingPost);
        //获取线程池并执行，线程池执行run方法，接下来看run方法
        eventBus.getExecutorService().execute(this);
    }

    @Override
    public void run() {
        PendingPost pendingPost = queue.poll();
        if(pendingPost == null) {
            throw new IllegalStateException("No pending post available");
        }
        //反射执行
        eventBus.invokeSubscriber(pendingPost);
    }

}
```

综上，处理事件分析基本完成。

## 五、 粘性事件

一般情况，我们使用 EventBus 都是准备好订阅事件的方法，然后注册事件，最后在发送事件，即要先有事件的接收者。但粘性事件却恰恰相反，我们可以先发送事件，后续再准备订阅事件的方法、注册事件。

### 1.发送事件

由于这种差异，我们分析粘性事件原理时，先从事件发送开始，发送一个粘性事件通过如下方式：

```java
EventBus.getDefault().postSticky("Hello World!");
```

实现如下：

```java
   public void postSticky(Object event) {
        synchronized (stickyEvents) {
            stickyEvents.put(event.getClass(), event);
        }
        // Should be posted after it is putted, in case the subscriber wants to remove immediately
        post(event);
    }

```

`postSticky()`方法主要做了两件事，先将事件类型和对应事件保存到`stickyEvents`中，方便后续使用；然后执行`post(event)`继续发送事件，这个`post()`方法就是之前发送的`post()`方法。所以，如果在发送粘性事件前，已经有了对应类型事件的订阅者，即便是非粘性的，依然可以接收到发送出的粘性事件。

### 2.注册事件

发送完粘性事件后，再准备订阅粘性事件的方法，并完成注册。核心的注册事件流程还是我们之前的`register()`方法中的`subscribe()`方法，前边分析`subscribe()`方法时，有一段没有分析的代码，就是用来处理粘性事件的：

```java
    private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
		......
        //该事件是stick=true。特殊处理.
        if (subscriberMethod.sticky) {
            // 默认为true，表示是否向上查找事件的父类
            if (eventInheritance) {
                // stickyEvents就是发送粘性事件时，保存了事件类型和对应事件
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                //循环获得每个stickyEvent事件
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    //是该类的父类
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        //该事件类型最新的事件
                        Object stickyEvent = entry.getValue();
                        //发送给当前订阅者，处理粘性事件
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }
```

可以看到，处理粘性事件就是在 EventBus 注册时，遍历`stickyEvents`，如果当前要注册的事件订阅方法是粘性的，并且该方法接收的事件类型和`stickyEvents`中某个事件类型相同或者是其父类，则取出`stickyEvents`中对应事件类型的具体事件，做进一步处理。

继续看`checkPostStickyEventToSubscription()`处理方法：

```java
    private void checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent) {
        if (stickyEvent != null) {
            // If the subscriber is trying to abort the event, it will fail (event is not tracked in posting state)
            // --> Strange corner case, which we don't take care of here.
            postToSubscription(newSubscription, stickyEvent, isMainThread());
        }
    }
```

最终还是通过`postToSubscription()`方法完成粘性事件的处理，这就是粘性事件的整个处理流程。

## 六、Subscriber Index

回顾之前分析的 EventBus 注册事件流程，主要是在项目运行时通过反射来查找订事件的方法信息，这也是默认的实现，如果项目中有大量的订阅事件的方法，必然会对项目运行时的性能产生影响。其实除了在项目运行时通过反射查找订阅事件的方法信息，EventBus 还提供了在项目编译时通过[注解处理器](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fgreenrobot%2FEventBus%2Fblob%2Fmaster%2FEventBusAnnotationProcessor%2Fsrc%2Forg%2Fgreenrobot%2Feventbus%2Fannotationprocessor%2FEventBusAnnotationProcessor.java)查找订阅事件方法信息的方式，生成一个辅助的索引类来保存这些信息，这个索引类就是[Subscriber Index](https://link.jianshu.com?t=http%3A%2F%2Fgreenrobot.org%2Feventbus%2Fdocumentation%2Fsubscriber-index%2F)，其实和 ButterKnife的原理类似。

要在项目编译时查找订阅事件的方法信息，首先要在 app 的 build.gradle 中加入如下配置：

```groovy
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                // 根据项目实际情况，指定辅助索引类的名称和包名
                arguments = [ eventBusIndex : 'com.example.myapp.MyEventBusIndex' ]
            }
        }
    }
}
dependencies {
    compile 'org.greenrobot:eventbus:3.1.1'
    // 引入注解处理器
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.1.1'
}
```

然后在项目的 Application 中添加如下配置，以生成一个默认的 EventBus 单例：

```java
EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
```

之后的用法就和我们平时使用 EventBus 一样了。当项目编译时会在生成对应的`MyEventBusIndex`类：

对应的源码如下：

```java
public class MyEventBusIndex implements SubscriberInfoIndex {
    private static final Map<Class<?>, SubscriberInfo> SUBSCRIBER_INDEX;

    static {
        SUBSCRIBER_INDEX = new HashMap<Class<?>, SubscriberInfo>();

        putIndex(new SimpleSubscriberInfo(MainActivity.class, true, new SubscriberMethodInfo[] {
            new SubscriberMethodInfo("changeText", String.class),
        }));

    }

    private static void putIndex(SubscriberInfo info) {
        SUBSCRIBER_INDEX.put(info.getSubscriberClass(), info);
    }

    @Override
    public SubscriberInfo getSubscriberInfo(Class<?> subscriberClass) {
        SubscriberInfo info = SUBSCRIBER_INDEX.get(subscriberClass);
        if (info != null) {
            return info;
        } else {
            return null;
        }
    }
}
```

其中`SUBSCRIBER_INDEX`是一个HashMap，保存了当前注册类的 Class 类型和其中事件订阅方法的信息。

接下来分析下使用 Subscriber Index 时 EventBus 的注册流程，我们先分析：

```java
EventBus.builder().addIndex(new MyEventBusIndex()).installDefaultEventBus();
```

首先创建一个`EventBusBuilder`，然后通过`addIndex()`方法添加索引类的实例：

```java
public EventBusBuilder addIndex(SubscriberInfoIndex index) {
        if (subscriberInfoIndexes == null) {
            subscriberInfoIndexes = new ArrayList<>();
        }
        subscriberInfoIndexes.add(index);
        return this;
    }
```

即把生成的索引类的实例保存在`subscriberInfoIndexes`集合中，然后用`installDefaultEventBus()`创建默认的 EventBus实例：

```java
public EventBus installDefaultEventBus() {
        synchronized (EventBus.class) {
            if (EventBus.defaultInstance != null) {
                throw new EventBusException("Default instance already exists." +
                        " It may be only set once before it's used the first time to ensure consistent behavior.");
            }
            EventBus.defaultInstance = build();
            return EventBus.defaultInstance;
        }
    }
```

```java
public EventBus build() {
        // this 代表当前EventBusBuilder对象
        return new EventBus(this);
    }
```

即用当前`EventBusBuilder`对象创建一个 EventBus 实例，这样我们通过`EventBusBuilder`配置的 Subscriber Index 也就传递到了EventBus实例中，然后赋值给EventBus的 `defaultInstance`成员变量。之前我们在分析 EventBus 的`getDefault()`方法时已经见到了`defaultInstance`：

```java
public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
```

所以在 Application 中生成了 EventBus 的默认单例，这样就保证了在项目其它地方执`EventBus.getDefault()`就能得到唯一的 EventBus 实例。

之前在分析注册流程时有一个方法`findUsingInfo()`：

```java
private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
        FindState findState = prepareFindState();
        findState.initForSubscriber(subscriberClass);
        while (findState.clazz != null) {
            // 查找SubscriberInfo
            findState.subscriberInfo = getSubscriberInfo(findState);
            // 条件成立
            if (findState.subscriberInfo != null) {
                // 获得当前注册类中所有订阅了事件的方法
                SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
                for (SubscriberMethod subscriberMethod : array) {
                    // findState.checkAdd()之前已经分析过了，即是否在FindState的anyMethodByEventType已经添加过以当前eventType为key的键值对，没添加过返回true
                    if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                        // 将subscriberMethod对象添加到subscriberMethods集合
                        findState.subscriberMethods.add(subscriberMethod);
                    }
                }
            } else {
                findUsingReflectionInSingleClass(findState);
            }
            findState.moveToSuperclass();
        }
        return getMethodsAndRelease(findState);
    }
```

由于我们现在使用了 Subscriber Index 所以不会通过`findUsingReflectionInSingleClass()`来反射解析订阅事件的方法。我们重点来看`getSubscriberInfo()`都做了些什么：

```java
private SubscriberInfo getSubscriberInfo(FindState findState) {
        // 该条件不成立
        if (findState.subscriberInfo != null && findState.subscriberInfo.getSuperSubscriberInfo() != null) {
            SubscriberInfo superclassInfo = findState.subscriberInfo.getSuperSubscriberInfo();
            if (findState.clazz == superclassInfo.getSubscriberClass()) {
                return superclassInfo;
            }
        }
        // 该条件成立
        if (subscriberInfoIndexes != null) {
            // 遍历索引类实例集合
            for (SubscriberInfoIndex index : subscriberInfoIndexes) {
                // 根据注册类的 Class 类查找SubscriberInfo
                SubscriberInfo info = index.getSubscriberInfo(findState.clazz);
                if (info != null) {
                    return info;
                }
            }
        }
        return null;
    }
```

`subscriberInfoIndexes`就是在前边`addIndex()`方法中创建的，保存了项目中的索引类实例，即`MyEventBusIndex`的实例，继续看索引类的`getSubscriberInfo()`方法，来到了`MyEventBusIndex`类中：

```kotlin
@Override
    public SubscriberInfo getSubscriberInfo(Class<?> subscriberClass) {
        SubscriberInfo info = SUBSCRIBER_INDEX.get(subscriberClass);
        if (info != null) {
            return info;
        } else {
            return null;
        }
    }
```

即根据注册类的 Class 类型从 `SUBSCRIBER_INDEX` 查找对应的`SubscriberInfo`，如果我们在注册类中定义了订阅事件的方法，则 `info`不为空，进而上边`findUsingInfo()`方法中`findState.subscriberInfo != null`成立，到这里主要的内容就分析完了，其它的和之前的注册流程一样。

所以 Subscriber Index 的核心就是项目编译时使用注解处理器生成保存事件订阅方法信息的索引类，然后项目运行时将索引类实例设置到 EventBus 中，这样当注册 EventBus 时，从索引类取出当前注册类对应的事件订阅方法信息，以完成最终的注册，避免了运行时反射处理的过程，所以在性能上会有质的提高。项目中可以根据实际的需求决定是否使用 Subscriber Index。

## 七、小结

结合上边的分析，我们可以总结出`register`、`post`、`unregister`的核心流程：

![img](https:////upload-images.jianshu.io/upload_images/1633070-2cc84089d0fff8dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/854/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/1633070-3637c94ce7ff4b07.png?imageMogr2/auto-orient/strip|imageView2/2/w/688/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/1633070-b3b3a17a97d95d02.png?imageMogr2/auto-orient/strip|imageView2/2/w/609/format/webp)



到这里 EventBus 几个重要的流程就分析完了，整体的设计思路还是值得我们学习的。和 Android 自带的广播相比，使用简单、同时又兼顾了性能。但如果在项目中滥用的话，各种逻辑交叉在一起，也可能会给后期的维护带来困难。



---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)