# OkHttp：源码详解之核心流程（一）

> 源码基于okhttp3 java版本：3.14.x

OkHttp整体架构：

![okhttp](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC8zLzI1LzE3MTExMTY4NmYxNDc1NWE?x-oss-process=image/format,png#pic_center)

首先，我们回顾一下基本的okhttp请求流程：

Get异步请求：

```java
 private void asynchronousGetRequests() {
        String url = "https://wwww.baidu.com";
        OkHttpClient okHttpClient = new OkHttpClient();
        final Request request = new Request.Builder()
                .url(url)
                .get()
                .build();
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
            }
        });
    }
```

Get同步请求：

```java
 private void synchronizedGetRequests() {
        String url = "https://wwww.baidu.com";
        OkHttpClient okHttpClient = new OkHttpClient();
        final Request request = new Request.Builder()
                .url(url)
                .get()
                .build();
        final Call call = okHttpClient.newCall(request);
        new Thread(() -> {
            try {
                //直接execute call
                Response response = call.execute();
                String result = response.body().string();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
```

**主要步骤：**

- 先创建OkHttpClient和Request实例

- 使用OkHttpClient和Request的实例来创建Call

- 根据调度方式的不同执行请求

> 同步请求和异步请求的主要区别在call的调用方式上。

接下我们以Get的请求的流程为例，开始分析OkHttp源码。

## 一、请求的创建

### 构建OkHttpClient

首先我们分析OkHttpClient的构建

```java
OkHttpClient okHttpClient = new OkHttpClient();
```

查看OkHttpClient源码

```java
  public OkHttpClient() {
    this(new Builder());
  }

  OkHttpClient(Builder builder) {
    this.dispatcher = builder.dispatcher;
    this.proxy = builder.proxy;
    this.protocols = builder.protocols;
    this.connectionSpecs = builder.connectionSpecs;
    this.interceptors = Util.immutableList(builder.interceptors);
    this.networkInterceptors = Util.immutableList(builder.networkInterceptors);
    this.eventListenerFactory = builder.eventListenerFactory;
    this.proxySelector = builder.proxySelector;
    this.cookieJar = builder.cookieJar;
    this.cache = builder.cache;
    this.internalCache = builder.internalCache;
    this.socketFactory = builder.socketFactory;

    boolean isTLS = false;
    for (ConnectionSpec spec : connectionSpecs) {
      isTLS = isTLS || spec.isTls();
    }

    if (builder.sslSocketFactory != null || !isTLS) {
      this.sslSocketFactory = builder.sslSocketFactory;
      this.certificateChainCleaner = builder.certificateChainCleaner;
    } else {
      X509TrustManager trustManager = Util.platformTrustManager();
      this.sslSocketFactory = newSslSocketFactory(trustManager);
      this.certificateChainCleaner = CertificateChainCleaner.get(trustManager);
    }

    if (sslSocketFactory != null) {
      Platform.get().configureSslSocketFactory(sslSocketFactory);
    }

    this.hostnameVerifier = builder.hostnameVerifier;
    this.certificatePinner = builder.certificatePinner.withCertificateChainCleaner(
        certificateChainCleaner);
    this.proxyAuthenticator = builder.proxyAuthenticator;
    this.authenticator = builder.authenticator;
    this.connectionPool = builder.connectionPool;
    this.dns = builder.dns;
    this.followSslRedirects = builder.followSslRedirects;
    this.followRedirects = builder.followRedirects;
    this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
    this.callTimeout = builder.callTimeout;
    this.connectTimeout = builder.connectTimeout;
    this.readTimeout = builder.readTimeout;
    this.writeTimeout = builder.writeTimeout;
    this.pingInterval = builder.pingInterval;

    if (interceptors.contains(null)) {
      throw new IllegalStateException("Null interceptor: " + interceptors);
    }
    if (networkInterceptors.contains(null)) {
      throw new IllegalStateException("Null network interceptor: " + networkInterceptors);
    }
  }
```

 OkHttpClient的构造方法中调用了`OkHttpClient(Builder builder)`,传入了`new Build()`。

`OkHttpClient(Builder builder)`中主要是设置了build传入的属性。

我们看一下Builder的源码，Builder为OkHttpClient的内部类

```java
  public static final class Builder {
    Dispatcher dispatcher;  //调度器，执行异步请求时的策略
    @Nullable Proxy proxy;  //代理设置
    List<Protocol> protocols; //实现的HTTP协议List
    List<ConnectionSpec> connectionSpecs; //TLS版本与连接协议
    final List<Interceptor> interceptors = new ArrayList<>(); //拦截器
    final List<Interceptor> networkInterceptors = new ArrayList<>();  //网络拦截器
    EventListener.Factory eventListenerFactory; //监听器的创建工厂
    ProxySelector proxySelector; //代理选择器
    CookieJar cookieJar; //cookie
    @Nullable Cache cache;  //缓存
    @Nullable InternalCache internalCache;//内部使用的缓存接口
    SocketFactory socketFactory; //socket 工厂
    @Nullable SSLSocketFactory sslSocketFactory;
    @Nullable CertificateChainCleaner certificateChainCleaner;
    HostnameVerifier hostnameVerifier; //主机name验证
    CertificatePinner certificatePinner; //证书
    Authenticator proxyAuthenticator; //代理验证
    Authenticator authenticator; //验证
    ConnectionPool connectionPool;  //连接池
    Dns dns;  //dns域名
    boolean followSslRedirects;//追踪ssl重定向
    boolean followRedirects;//追踪请求重定向
    boolean retryOnConnectionFailure;//连接失败时是否重试
    int callTimeout;//cell超时设置
    int connectTimeout;//连接超时设置
    int readTimeout;//读超时设置
    int writeTimeout;//写超时设置
    int pingInterval;//ping间隔时间

    public Builder() {
      dispatcher = new Dispatcher();
      protocols = DEFAULT_PROTOCOLS;
      connectionSpecs = DEFAULT_CONNECTION_SPECS;
      eventListenerFactory = EventListener.factory(EventListener.NONE);
      proxySelector = ProxySelector.getDefault();
      if (proxySelector == null) {
        proxySelector = new NullProxySelector();
      }
      cookieJar = CookieJar.NO_COOKIES;
      socketFactory = SocketFactory.getDefault();
      hostnameVerifier = OkHostnameVerifier.INSTANCE;
      certificatePinner = CertificatePinner.DEFAULT;
      proxyAuthenticator = Authenticator.NONE;
      authenticator = Authenticator.NONE;
      connectionPool = new ConnectionPool();
      dns = Dns.SYSTEM;
      followSslRedirects = true;
      followRedirects = true;
      retryOnConnectionFailure = true;
      callTimeout = 0;
      connectTimeout = 10_000;
      readTimeout = 10_000;
      writeTimeout = 10_000;
      pingInterval = 0;
    }
	
	Builder(OkHttpClient okHttpClient) {
		...
	}
	...
	public OkHttpClient build() {
      return new OkHttpClient(this);
    }
}
```

在Builder构造方法中，设置了各个属性的默认值，在源码中还提供可以对属性可以进行配置的函数。

这里用到了构造者模式，若不了解的话推荐阅读[建造者模式（Bulider模式）详解](http://c.biancheng.net/view/1354.html)

综上直接`new OkHttpClient()`实例，配置项就是Builder构造方法中默认值，设置了请求的全局参数。

### 构建Request

接下来我们看Request的源码，对应如下:

```java
Request request = new Request.Builder()
                .url(url)
                .get()
                .build();
```

源码如下：

```java
public final class Request {
  final HttpUrl url;
  final String method;
  final Headers headers;
  final @Nullable RequestBody body;
  final Map<Class<?>, Object> tags;

  private volatile @Nullable CacheControl cacheControl; // Lazily initialized.

  Request(Builder builder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers.build();
    this.body = builder.body;
    this.tags = Util.immutableMap(builder.tags);
  }

  ...
	
  public static class Builder {
    @Nullable HttpUrl url;      // 请求访问的 url
    String method;              // 请求方式
    Headers.Builder headers;    // header
    @Nullable RequestBody body; // body

    /** A mutable map of tags, or an immutable empty map if we don't have any. */
    Map<Class<?>, Object> tags = Collections.emptyMap();

    public Builder() {
      this.method = "GET";
      this.headers = new Headers.Builder();
    }

    Builder(Request request) {
      this.url = request.url;
      this.method = request.method;
      this.body = request.body;
      this.tags = request.tags.isEmpty()
          ? Collections.emptyMap()
          : new LinkedHashMap<>(request.tags);
      this.headers = request.headers.newBuilder();
    }
    
    ...
    
    public Request build() {
      if (url == null) throw new IllegalStateException("url == null");
      return new Request(this);
    }
}
```

我们可以看到Request的创建，也是使用建造者模式。

加下来看一下url方法和请求方法：

```java
    public Builder url(HttpUrl url) {
      if (url == null) throw new NullPointerException("url == null");
      this.url = url;
      return this;
    }

    public Builder url(String url) {
      if (url == null) throw new NullPointerException("url == null");

      // Silently replace web socket URLs with HTTP URLs.
      if (url.regionMatches(true, 0, "ws:", 0, 3)) {
        url = "http:" + url.substring(3);
      } else if (url.regionMatches(true, 0, "wss:", 0, 4)) {
        url = "https:" + url.substring(4);
      }

      return url(HttpUrl.get(url));
    }

    public Builder url(URL url) {
      if (url == null) throw new NullPointerException("url == null");
      return url(HttpUrl.get(url.toString()));
    }
    
    public Builder get() {
      return method("GET", null);
    }

    public Builder head() {
      return method("HEAD", null);
    }

    public Builder post(RequestBody body) {
      return method("POST", body);
    }

    public Builder delete(@Nullable RequestBody body) {
      return method("DELETE", body);
    }

    public Builder delete() {
      return delete(Util.EMPTY_REQUEST);
    }

    public Builder put(RequestBody body) {
      return method("PUT", body);
    }

    public Builder patch(RequestBody body) {
      return method("PATCH", body);
    }

    //method方法内部对请求方式和请求体进行了校验
    public Builder method(String method, @Nullable RequestBody body) {
      if (method == null) throw new NullPointerException("method == null");
      if (method.length() == 0) throw new IllegalArgumentException("method.length() == 0");
      if (body != null && !HttpMethod.permitsRequestBody(method)) {
        throw new IllegalArgumentException("method " + method + " must not have a request body.");
      }
      if (body == null && HttpMethod.requiresRequestBody(method)) {
        throw new IllegalArgumentException("method " + method + " must have a request body.");
      }
      this.method = method;
      this.body = body;
      return this;
    }
```

get()和post(RequestBody body)等请求方式都是对method方法的包装，method方法内部对请求方式和请求体进行了校验，比如get请求不能有请求体、post请求必须要请求体等。

Request主要是设定了具体的请求。

### 构建Call

```java
Call call = okHttpClient.newCall(request);
```

接着看HttpClient的newCall方法：

```java
  @Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
  }
```

在newCall实例化了**RealCall**。

RealCall的newRealCall方法：

```java
  static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    // Transmitter意为发射器，是应用层和网络层的桥梁
    // 在进行连接、真正发出请求和读取响应中起到很重要的作用
    call.transmitter = new Transmitter(client, call);
    return call;
  }

  private RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    this.client = client;
    this.originalRequest = originalRequest;
    this.forWebSocket = forWebSocket;
  }

  ...
```

RealCall就是准备执行的请求，是对接口Call的实现。内部持有OkHttpClient实例、Request实例。

并且这里还创建了**Transmitter**给RealCall的transmitter赋值。

**Transmitter意为发射器，是应用层和网络层的桥梁**，在进行 **连接**、真正发出请求和读取响应中起到很重要的作用，看下构造方法：

```java
  public Transmitter(OkHttpClient client, Call call) {
    //保存了OkHttpClient、连接池、call、事件监听器、call超时时间。
    this.client = client;
    this.connectionPool = Internal.instance.realConnectionPool(client.connectionPool());
    this.call = call;
    this.eventListener = client.eventListenerFactory().create(call);
    this.timeout.timeout(client.callTimeoutMillis(), MILLISECONDS);
  }
```

连接池我们先注意一下，留个印象，后面回详细讲解。

`client.connectionPool()`即client的build中默认构建的connectionPool：

```java
  public ConnectionPool() {
    //默认的连接池，最大连接5，保持Alive持续时间5min
    this(5, 5, TimeUnit.MINUTES);
  }
```

RealCall实现了Call接口。我们再看一下**Call接口**：

```java
public interface Call extends Cloneable {
  /** Returns the original request that initiated this call. */
  Request request();//请求

  Response execute() throws IOException;//同步

  void enqueue(Callback responseCallback);//异步

  void cancel();//取消请求

  boolean isExecuted();//是否在请求过程中

  boolean isCanceled();//是否取消

  Timeout timeout();

  Call clone();

  //工厂接口
  interface Factory {
    Call newCall(Request request);
  }
}
```

主要是定义请求的执行动作和状态。RealCall对Call的具体实现，在后面执行流程中说明。

OkHttpClient，Request，Call三个部分的创建都已经讲解完毕，接下来我们看是OkHttp是如何进行事件调度的。

## 二、调度 

### 同步

```java
Response response = call.execute();
```

上面以及分析过，这里call的实现是RealCall，即调用RealCall的execute方法：

```java
  @Override public Response execute() throws IOException {
    synchronized (this) {
      //如果已经执行，就会抛出异常。这就是一个请求只能执行一次的原因。
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    //超时计时开始
    transmitter.timeoutEnter();
    //回调请求监听器的请求开始
    transmitter.callStart();
    try {
      //加入同步请求的调度队列
      client.dispatcher().executed(this);
      //通过拦截链获取request的response
      return getResponseWithInterceptorChain();
    } finally {
      //完成请求
      client.dispatcher().finished(this);
    }
  }
```

流程的具体描述见注释。这里我们先看一下`client.dispatcher().executed(this);`

```java
  //正在进行的同步请求
  /** Running synchronous calls. Includes canceled calls that haven't finished yet. */
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

  synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }
```

请求放入一个双端队列runningSyncCalls中，表示正在执行的同步请求。

getResponseWithInterceptorChain()的返回结果是Response，**同步请求真正的请求流程是在getResponseWithInterceptorChain()方法中**。这个我们稍后和异步请求一起看。

请求结束，会走Dispatcher的finished方法：

```java
  //同步请求执行结束
  void finished(RealCall call) {
    finished(runningSyncCalls, call);
  }

  private <T> void finished(Deque<T> calls, T call) {
    Runnable idleCallback;
    synchronized (this) {
      //移除队列中的cell
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      idleCallback = this.idleCallback;
    }
    //继续执行其他异步请求
    boolean isRunning = promoteAndExecute();
    //触发空闲线程执行
    if (!isRunning && idleCallback != null) {
      idleCallback.run();
    }
  }
```

这里需要我们注意`promoteAndExecute()`，是去执行后面的异步请求，在异步部分会进行讲解。

综上，同步请求的流程我们大概有了了解，接下来就看异步请求。

### 异步

```java
call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
            }
        });
```

RealCall的enqueue函数源码：

```java
  @Override public void enqueue(Callback responseCallback) {
    synchronized (this) {
      //如果已经执行，就会抛出异常。这就是一个请求只能执行一次的原因。
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    //回调请求监听器的请求开始
    transmitter.callStart();
    //用AsyncCall封装Callback，由dispatcher添加到就绪队列
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
  }

```

Dispatcher的enqueue方法，参数接受的是AsyncCall，AsyncCall继承NamedRunnable，NamedRunnable实现自Runnable，即AsyncCall就是个Runnable。接下来我们看下这部分源码：

AsyncCall是RealCall的内部类

```java
  //AsyncCall就是一个Runnable
  final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;
    //目前每个主机（域名）有多少个会话call
    private volatile AtomicInteger callsPerHost = new AtomicInteger(0);

    AsyncCall(Callback responseCallback) {
      super("OkHttp %s", redactedUrl());
      this.responseCallback = responseCallback;
    }
    ...
  }
```

NamedRunnable 实现了Runnable

```java
public abstract class NamedRunnable implements Runnable {
  //记录了当前线程名字的Runnable
  protected final String name;

  public NamedRunnable(String format, Object... args) {
    this.name = Util.format(format, args);
  }

  @Override public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

  protected abstract void execute();
}
```

可以看到run方法中调用了抽象的execute()，**AsyncCall的run方法实际就是其实现的execute()**。

回到RealCall#enqueue方法，调用了**Dispatcher#enqueue**方法。源码如下：

```java
  //即将要进行的异步请求
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

  //正在进行的异步请求
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

  void enqueue(AsyncCall call) {
    synchronized (this) {
      //添加请求到异步队列
      readyAsyncCalls.add(call);
        
      if (!call.get().forWebSocket) {
        // 判断当前请求是否已经存在
        AsyncCall existingCall = findExistingCallWithHost(call.host());
        // 如果当前请求已经存在，则复用之前的线程计数，不进行递增
        if (existingCall != null) call.reuseCallsPerHostFrom(existingCall);
      }
    }
    //执行请求
    promoteAndExecute();
  }

  @Nullable private AsyncCall findExistingCallWithHost(String host) {
    //遍历runningAsyncCalls或者readyAsyncCalls中找到相同host的请求
    for (AsyncCall existingCall : runningAsyncCalls) {
      if (existingCall.host().equals(host)) return existingCall;
    }
    for (AsyncCall existingCall : readyAsyncCalls) {
      if (existingCall.host().equals(host)) return existingCall;
    }
    return null;
  }
```

- 请求放入双端队列readyAsyncCalls中，表示等待执行的异步请求。

- 从正在执行的请求runningAsyncCalls 或 等待执行的请求readyAsyncCalls 中找到是相同host的请求，把AsyncCall的callsPerHost重用给当前请求。

callsPerHost看名字感觉像是 拥有相同host的请求的数量，并且注意到类型是AtomicInteger。

```java
    //目前每个主机（域名）有多少个会话call
    private volatile AtomicInteger callsPerHost = new AtomicInteger(0);
    
    void reuseCallsPerHostFrom(AsyncCall other) {
      this.callsPerHost = other.callsPerHost;
    }
```

**相同host的请求是共享callsPerHost的**，为了后面判断host并发做准备。

`promoteAndExecute()`前面同步请求结束时候调用过，这里我们看一下代码：

```java
  private int maxRequests = 64;
  private int maxRequestsPerHost = 5;
  
  private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));
    //可执行的AsyncCall
    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;//没啥用
    synchronized (this) {
      // 遍历准备要执行的请求队列
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();
        // 当前正在执行的请求个数大于最大请求个数64时，则取消请求
        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        // 当前主机的连接数超过5个时，则跳过当前请求
        if (asyncCall.callsPerHost().get() >= maxRequestsPerHost) continue; // Host max capacity.
        // 从等待队列中移除
        i.remove();
        // cell计数器+1
        asyncCall.callsPerHost().incrementAndGet();
        // 把AsyncCall保存起来
        executableCalls.add(asyncCall);
        // 添加请求到正在执行的队列中
        runningAsyncCalls.add(asyncCall);
      }
      //正在执行的异步/同步 请求数 >0
      isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      //执行请求
      //executorService返回了一个线程池
      asyncCall.executeOn(executorService());
    }

    return isRunning;
  }
```

异步请求并发数达到64、相同host的异步请求达到5，都不会再继续请求。

遍历完后 把executableCalls中的请求都执行`asyncCall.executeOn(executorService());`

我们先看`executorService()`函数：

```java
  public synchronized ExecutorService executorService() {
    if (executorService == null) {
      //核心线程数为0，空闲线程等待60秒超时后，线程会被清空；最大线程数无限制，已有Dispatcher会限制请求数了。
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }

```

创建返回了一个请求执行的线程池。

接下来看`asyncCall.executeOn`方法：

```java
    void executeOn(ExecutorService executorService) {
      assert (!Thread.holdsLock(client.dispatcher()));
      boolean success = false;
      try {
        //线程池运行Runnable
        //实际是调用AsyncCall#execute
        //AsyncCall继承抽象的NamedRunnable实现Runnable接口，run方法在NamedRunnable中，调用了抽象execute
        executorService.execute(this);
        success = true;
      } catch (RejectedExecutionException e) {
        InterruptedIOException ioException = new InterruptedIOException("executor rejected");
        ioException.initCause(e);
        transmitter.noMoreExchanges(ioException);
        //RejectedExecutionException失败回调
        responseCallback.onFailure(RealCall.this, ioException);
      } finally {
        if (!success) {
          //异常结束
          client.dispatcher().finished(this); // This call is no longer running!
        }
      }
    }
```

使用类似CachedThreadPool的**线程池**执行请求RealCall。

先看一下`client.dispatcher().finished(this)`代码：

```java
  //异步请求执行结束
  void finished(AsyncCall call) {
    call.callsPerHost().decrementAndGet();
    finished(runningAsyncCalls, call);
  }
 
  private <T> void finished(Deque<T> calls, T call) {
    Runnable idleCallback;
    synchronized (this) {
      //移除队列中的cell
      if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      idleCallback = this.idleCallback;
    }
    //继续执行其他异步请求
    boolean isRunning = promoteAndExecute();
    //触发空闲线程执行
    if (!isRunning && idleCallback != null) {
      idleCallback.run();
    }
  }
```

**注意：**和同步相比多了一句话`call.callsPerHost().decrementAndGet();`callsPerHost-1。同步请求不涉及到callsPerHost的加减。

根据之前我们的分析，`executorService.execute(this);`会调用AsyncCall的`execute()`方法

```java
    //run方法
    @Override protected void execute() {
      boolean signalledCallback = false;
      //超时计时开始
      transmitter.timeoutEnter();
      try {
        //通过拦截器链得到Response
        Response response = getResponseWithInterceptorChain();
        signalledCallback = true;
        //Http请求执行成功，回调
        responseCallback.onResponse(RealCall.this, response);
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          //IOException失败回调
          responseCallback.onFailure(RealCall.this, e);
        }
      } catch (Throwable t) {
        cancel();
        if (!signalledCallback) {
          IOException canceledException = new IOException("canceled due to " + t);
          canceledException.addSuppressed(t);
          // 其他Throwable失败回调
          responseCallback.onFailure(RealCall.this, canceledException);
        }
        throw t;
      } finally {
        //结束，callsPerHost减1 runningAsyncCalls移除AsyncCall，继续执行其他请求
        client.dispatcher().finished(this);
      }
    }
  }
```

可以看到这里使用`getResponseWithInterceptorChain()`获取请求的执行结果Response。与上面的同步请求方式一致。所以我们接下来在下一节重点分析`getResponseWithInterceptorChain()`方法。

使用responseCallback调用回调函数，也就是我们在回调函数中写的逻辑。

最后请求结束也是调用了dispatcher的finish方法，上面以及分析过了。

**总结：**

1. **同步请求**：RealCall使用Dispatcher存入runningSyncCalls，然后使用getResponseWithInterceptorChain()获取结果，最后调用Dispatcher的finish方法结束请求。
2. **异步请求**：RealCall使用Dispatcher存入readyAsyncCalls，获得host并发数，使用promoteAndExecute()方法在 控制异步并发 的策略基础上，使用 线程池 执行异步请求（并发控制有包括 最大并发数64、host最大并发数5）。异步请求的执行同样使用getResponseWithInterceptorChain()，获得结果后回调出去。最后调用Dispatcher的finish方法结束请求。

**Dispatcher**：调度器，主要是异步请求的并发控制、把异步请求放入线程池执行，实现方法是promoteAndExecute()。 promoteAndExecute()有两处调用：添加异步请求时、同步/异步请求 结束时。

综上，我们的整个调度流程分析完毕了，接下来分析执行过程，即`getResponseWithInterceptorChain()`方法。

## 三、执行

无论同步还是异步请求，最终的执行都是在RealCall的getResponseWithInterceptorChain()方法，只不过异步请求需要先通过Dispatcher进行并发控制和线程池处理。

```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());//OkHttpClient.Build#addInterceptor添加自定义拦截器
    interceptors.add(new RetryAndFollowUpInterceptor(client));//重试和重定向拦截器
    interceptors.add(new BridgeInterceptor(client.cookieJar()));//封装Request、Response的拦截器
    interceptors.add(new CacheInterceptor(client.internalCache()));//缓存处理相关的拦截器
    interceptors.add(new ConnectInterceptor(client));//连接服务的拦截器，真正的网络请求在这里实现
    if (!forWebSocket) {
      //OkHttpClient.Build#addNetworkInterceptor添加自定义网络网络拦截器
      //在ConnectInterceptor后面，此时网络连接已准备好
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));//负责写请求和读响应的拦截器
    //拦截器链
    Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    boolean calledNoMoreExchanges = false;
    try {
      //拦截器链 开始执行第一个index = 0 的拦截器
      Response response = chain.proceed(originalRequest);
      if (transmitter.isCanceled()) {
        closeQuietly(response);
        throw new IOException("Canceled");
      }
      return response;
    } catch (IOException e) {
      calledNoMoreExchanges = true;
      throw transmitter.noMoreExchanges(e);
    } finally {
      if (!calledNoMoreExchanges) {
        transmitter.noMoreExchanges(null);
      }
    }
  }
```

**在interceptors中依次添加了七类拦截器：**

- 应用拦截器（外部配置）client.interceptors()、
- 重试和重定向拦截器RetryAndFollowUpInterceptor、
- 桥拦截器BridgeInterceptor、
- 缓存拦截器CacheInterceptor、
- 连接拦截器ConnectInterceptor、
- 网络拦截器（外部配置）client.networkInterceptors()、
- 请求服务拦截器CallServerInterceptor，

可以看到应用拦截器（最早添加）比网络拦截器要早添加到interceptors中。可以和前面《OkHttp：进阶详解》中所讲的两者特点和区别相互呼应。宏观上看，应用拦截器获取了网络请求的最终结果，网络拦截器则监听请求过程的每个流程。

该小节我们只关注整个流程，从整体上理解拦截器的工作过程，具体的实现留到后面讲解。

我们知道所有的拦截器都是实现了Interceptor接口。接下来让我们看看Interceptor的源码：

```java
//拦截器
public interface Interceptor {

  //把拦截器链传递下去，实现请求的处理过程，返回response
  Response intercept(Chain chain) throws IOException;

  //拦截器链，负责把所有拦截器串联起来
  interface Chain {
    Request request();

    //Chain的核心方法，进行拦截器的层次调用
    Response proceed(Request request) throws IOException;

    //返回请求执行的 连接. 仅网络拦截器可用; 应用拦截器就是null.
    @Nullable Connection connection();

    Call call();

    int connectTimeoutMillis();

    Chain withConnectTimeout(int timeout, TimeUnit unit);

    int readTimeoutMillis();

    Chain withReadTimeout(int timeout, TimeUnit unit);

    int writeTimeoutMillis();

    Chain withWriteTimeout(int timeout, TimeUnit unit);
  }
}

```

Interceptor是个接口类，只有一个intercept方法，参数是Chain对象。内部接口类Chain拦截器链，有个proceed方法，参数是Request对象，返回值是Response，这个方法的实现就是请求的处理过程了。

Chain的唯一实现类就是RealInterceptorChain，负责把所有拦截器串联起来，在实例化RealInterceptorChain时，index赋值是0。

```java
    //拦截器链
    Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());
```

RealInterceptorChain#proceed函数是拦截链执行并获取结果的函数，所以我们要看一下其实现。

```java
     Response response = chain.proceed(originalRequest);
```

代码如下：

```java
  @Override public Response proceed(Request request) throws IOException {
    return proceed(request, transmitter, exchange);
  }

  public Response proceed(Request request, Transmitter transmitter, @Nullable Exchange exchange)
      throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();

    calls++;
	
    ...     
    // 传入index + 1，可以访问下一个拦截器
    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(interceptors, transmitter, exchange, index + 1, request, call, connectTimeout, readTimeout, writeTimeout);
    // 获得索引为index的拦截器
    Interceptor interceptor = interceptors.get(index);
    // 执行第一个拦截器，同时传入next（RealInterceptorChain），此时索引为index+1
    Response response = interceptor.intercept(next);
    ...
        
    //等所有拦截器处理完，就能返回Response了
    return response;
  }
```

我们现在只关注核心逻辑，RealInterceptorChain#proceed中再次创建RealInterceptorChain，定义为next，与RealCall#getResponseWithInterceptorChain()函数中RealInterceptorChain的创建不同的是传入了index + 1，其他参数和当前实例是一样的：

```java
RealInterceptorChain next = new RealInterceptorChain(interceptors, transmitter, exchange, index + 1, request, call, connectTimeout, readTimeout, writeTimeout);
```

接下来获取了第index个拦截器：

```java
Interceptor interceptor = interceptors.get(index)
```

调用intercept方法，传入了next，这一步就是去执行index拦截器的`Response intercept(Chain chain)`方法：

```java
 Response response = interceptor.intercept(next)
```

假设我们没有应用拦截器，RetryAndFollowUpInterceptor即为第一个拦截器，RetryAndFollowUpInterceptor#intercept方法看一下：

```java
public final class RetryAndFollowUpInterceptor implements Interceptor {

  @Override public Response intercept(Chain chain) throws IOException {
    Request request = chain.request();
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Transmitter transmitter = realChain.transmitter();

    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      //处理request
      ...
      Response response;
      boolean success = false;
      try {
        // 执行其他(下一个->下一个->...)拦截器的功能，获取Response；
        response = realChain.proceed(request, transmitter, null);
        success = true;
      } catch (RouteException e) {
        ...
      } catch (IOException e) {
        ...
      } finally {
        ...
      }
	  //处理response
      ...
    }
  ...
}
```

我们只关心核心流程，其他的先不看

```java
response = realChain.proceed(request, transmitter, null);
```

接上面的分析，getResponseWithInterceptorChain()中RealInterceptorChain的proceed方法最终会调用RetryAndFollowUpInterceptor（当前例子的第一个拦截器）的intercept()。RetryAndFollowUpInterceptor的intercept方法会通过chain.proceed调用（这里的chain是前面传进来的索引值为index+1的拦截器链，所以get的时候就跳过了已经取过的拦截器）继续调用下一个拦截器（BridgeInterceptor）的intercept，层层传递。

相应的，最后一个拦截器CallServerInterceptor返回后，函数层层返回，最终获取到了response。

**除了最后一个拦截器CallServerInterceptor之外，所有拦截器的intercept方法都调用了 传入chain的proceed方法**。每个拦截器在chain的proceed方法前后处理了自己负责的工作。

逻辑总结如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517210957331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hmeTg5NzE2MTM=,size_16,color_FFFFFF,t_70#pic_center)

OkHttp的RealCall#getResponseWithInterceptorChain()执行流程的整体核心流程如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517192938866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hmeTg5NzE2MTM=,size_16,color_FFFFFF,t_70#pic_center#pic_center)

**总结：**

1. 拦截器链：把原始请求 request **依次** 传入到每个拦截器。拦截器处理后把response **反向** 依次回传。
2. 拦截器：可以对request进行处理，然后调用index+1的拦截器链proceed方法，获取下一个拦截器处理的结果，接着自己也可以处理这个结果，即：处理request、chain.proceed、处理response。

| 拦截器                      | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| 应用拦截器                  | 处理原始请求和最终的响应：可以添加自定义header、通用参数、参数加密、网关接入等等。 |
| RetryAndFollowUpInterceptor | 处理错误重试和重定向                                         |
| BridgeInterceptor           | 应用层和网络层的桥接拦截器，主要工作是为请求添加cookie、添加固定的header，比如Host、Content-Length、Content-Type、User-Agent等等，然后保存响应结果的cookie，如果响应使用gzip压缩过，则还需要进行解压。 |
| CacheInterceptor            | 缓存拦截器，获取缓存、更新缓存。如果命中缓存则不会发起网络请求。 |
| ConnectInterceptor          | 连接拦截器，内部会维护一个连接池，负责连接复用、创建连接（三次握手等等）、释放连接以及创建连接上的socket流。 |
| 网络拦截器                  | 用户自定义拦截器，通常用于监控网络层的数据传输。             |
| CallServerInterceptor       | 请求拦截器，在前置准备工作完成后，真正发起网络请求，进行IO读写。 |



---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)