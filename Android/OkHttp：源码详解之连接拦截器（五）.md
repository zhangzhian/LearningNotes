# OkHttp：源码详解之连接拦截器（五）

> 源码基于okhttp3 java版本：3.14.x

前面分析了RetryAndFollowUpInterceptor、BridgeInterceptor、CacheInterceptor，三个拦截器，它们在请求建立连接之前做了一些预处理。请求经过这三个拦截器后，接下要分析剩下的两个拦截器：ConnectInterceptor、CallServerInterceptor，分别负责 **连接建立**、**请求服务读写**。本文先分析ConnectInterceptor拦截器。

## ConnectInterceptor

连接拦截器ConnectInterceptor代码实现如下：

```java
public final class ConnectInterceptor implements Interceptor {
  public final OkHttpClient client;

  public ConnectInterceptor(OkHttpClient client) {
    this.client = client;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    Transmitter transmitter = realChain.transmitter();

    // We need the network to satisfy this request. Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");

    // 实现了复杂的网络请求，Exchange用于下一个拦截器CallServerInterceptor进行网络请求使用；
    /*
        1：调用transmitter.newExchange方法；
        2：通过Transmitter的ExchangeFinder调用了find方法；
        3：ExchangeFinder里调用了findHealthyConnection方法；
        4：ExchangeFinder里findHealthyConnection调用了findConnection方法创建了RealConnection；
        5：调用了RealConnection的connect方法，实现了TCP + TLS 握手，底层通过socket来实现的；
        6: 通过RealConnection的newCodec方法创建了两个协议类，一个是Http1ExchangeCodec，对应着HTTP1.1，
        一个是Http2ExchangeCodec，对应着HTTP2.0；
    */
    //创建一个交换器Exchange
    Exchange exchange = transmitter.newExchange(chain, doExtensiveHealthChecks);
    // 执行后续的拦截器逻辑,注意是三个参数
    return realChain.proceed(request, transmitter, exchange);
  }
}
```

看起来代码很少很简单，实际上实现很复杂。

从代码流程上看：

- 通过`realChain.transmitter()`获取Transmitter

- 使用`transmitter.newExchange`获取Exchange实例
- Exchange实例作为参数调用拦截器链的proceed的方法。

### Transmitter

**Exchange（意为交换）主要作用就是真正的IO操作：写入请求、读取响应**（会在下一个拦截器做介绍）。

实际上获取Exchange实例的逻辑处理都封装在Transmitter中了。前面的文章提到过Transmitter，它是“发射器”，是把请求从应用端发射到网络层，它持有请求的连接、请求、响应和流，一个请求对应一个Transmitter实例，一个数据流。

我们前面分析过Transmitter的创建，再来回顾一下。

在RealCall#newRealCall的时候，创建了Transmitter的示例并赋值给call.transmitter。

```java
  static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    // Transmitter意为发射器，是应用层和网络层的桥梁
    // 在进行连接、真正发出请求和读取响应中起到很重要的作用
    call.transmitter = new Transmitter(client, call);
    return call;
  }
```

Transmitter的构造方法实现如下：

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

下面就看下它的**newExchange方法**：

```java
  Exchange newExchange(Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
    synchronized (connectionPool) {
      if (noMoreExchanges) {
        throw new IllegalStateException("released");
      }
      if (exchange != null) {
        throw new IllegalStateException("cannot make a new request because the previous response "
            + "is still open: please call response.close()");
      }
    }
    //RetryAndFollowUpInterceptor#intercept中调用this#prepareToConnect()方法,初始化exchangeFinder
    //使用exchangeFinder的find方法获取到了ExchangeCodec实例，然后作为参数构建了Exchange实例，并返回
    ExchangeCodec codec = exchangeFinder.find(client, chain, doExtensiveHealthChecks);
    Exchange result = new Exchange(this, call, eventListener, exchangeFinder, codec);

    synchronized (connectionPool) {
      this.exchange = result;
      this.exchangeRequestDone = false;
      this.exchangeResponseDone = false;
      return result;
    }
  }

```

使用exchangeFinder的find方法获取到了ExchangeCodec实例，然后作为参数构建了Exchange实例，并返回。

注意到这个方法里涉及了连接池RealConnectionPool、交换寻找器ExchangeFinder、交换编解码ExchangeCodec、交换管理Exchange这几个类。

- RealConnectionPool，连接池，负责管理请求的连接，包括新建、复用、关闭，理解上类似线程池。
- ExchangeCodec，接口类，负责真正的IO操作—写请求、读响应，实现类有Http1ExchangeCodec、Http2ExchangeCodec，分别对应HTTP1.1协议、HTTP2.0协议。
- Exchange，管理IO操作，可以理解为 数据流，是ExchangeCodec的包装，增加了事件回调；一个请求对应一个Exchange实例。传给下个拦截器CallServerInterceptor使用。
- ExchangeFinder，（从连接池中）寻找可用TCP连接，然后通过连接得到ExchangeCodec。

### Exchange/ExchangeFinder

这里我们注意到了exchangeFinder，是在RetryAndFollowUpInterceptor#intercept中调用this#prepareToConnect()方法，初始化的。

```java
  public void prepareToConnect(Request request) {
    if (this.request != null) {
      if (sameConnection(this.request.url(), request.url()) && exchangeFinder.hasRouteToTry()) {
        return; // Already ready.//已有相同连接
      }
      if (exchange != null) throw new IllegalStateException();

      if (exchangeFinder != null) {
        maybeReleaseConnection(null, true);
        exchangeFinder = null;
      }
    }

    this.request = request;
    //创建ExchangeFinder，目的是为获取连接做准备
    //ExchangeFinder是交换查找器，作用是获取请求的连接
    //connectionPool是在OkHttpClient.Build->new ConnectionPool->new RealConnectionPool创建的
    //createAddress方法返回的Address
    //注意:构造函数中创建了RouteSelector
    this.exchangeFinder = new ExchangeFinder(this, connectionPool, createAddress(request.url()),
        call, eventListener);
  }
```

先看下createAddress方法：

```java
  private Address createAddress(HttpUrl url) {
    SSLSocketFactory sslSocketFactory = null;
    HostnameVerifier hostnameVerifier = null;
    CertificatePinner certificatePinner = null;
    if (url.isHttps()) {
      sslSocketFactory = client.sslSocketFactory();
      hostnameVerifier = client.hostnameVerifier();
      certificatePinner = client.certificatePinner();
    }
    //使用url和client配置 创建一个Address实例
    //Address意思是指向服务的连接的地址，可以理解为请求地址及其配置
    //复用连接：相同Address的HTTP请求 共享 相同的连接
    return new Address(url.host(), url.port(), client.dns(), client.socketFactory(),
        sslSocketFactory, hostnameVerifier, certificatePinner, client.proxyAuthenticator(),
        client.proxy(), client.protocols(), client.connectionSpecs(), client.proxySelector());
  }
```

使用url和client配置 创建一个Address实例。Address意思是指向服务的连接的地址，可以理解为请求地址及其配置。Address有一个重要作用：**相同Address的HTTP请求 共享 相同的连接**。这可以作为前面提到的 HTTP1.1和HTTP2.0 **复用连接** 的请求的判断。

查看ExchangeFinder的构造函数：

```java
  ExchangeFinder(Transmitter transmitter, RealConnectionPool connectionPool,
      Address address, Call call, EventListener eventListener) {
    this.transmitter = transmitter;
    this.connectionPool = connectionPool;
    this.address = address;
    this.call = call;
    this.eventListener = eventListener;
    //创建RouteSelector
    this.routeSelector = new RouteSelector(
        address, connectionPool.routeDatabase, call, eventListener);
  }

```

在ExchangeFinder中创建了RouteSelector，后续我们会讲到。

回头看exchangeFinder的find方法

```java
  public ExchangeCodec find(
      OkHttpClient client, Interceptor.Chain chain, boolean doExtensiveHealthChecks) {
    int connectTimeout = chain.connectTimeoutMillis();
    int readTimeout = chain.readTimeoutMillis();
    int writeTimeout = chain.writeTimeoutMillis();
    int pingIntervalMillis = client.pingIntervalMillis();
    boolean connectionRetryEnabled = client.retryOnConnectionFailure();

    try {
      //找到一个健康的连接
      RealConnection resultConnection = findHealthyConnection(connectTimeout, readTimeout,
          writeTimeout, pingIntervalMillis, connectionRetryEnabled, doExtensiveHealthChecks);
      //利用连接实例化ExchangeCodec对象，如果是HTTP/2返回Http2ExchangeCodec，否则返回Http1ExchangeCodec
      return resultConnection.newCodec(client, chain);
    } catch (RouteException e) {
      trackFailure();
      throw e;
    } catch (IOException e) {
      trackFailure();
      throw new RouteException(e);
    }
  }
```

RealConnection的newCodec方法如下：

```java
ExchangeCodec newCodec(OkHttpClient client, Interceptor.Chain chain) throws SocketException {
  // http2Connection不为空就创建Http2ExchangeCodec，否则是Http1ExchangeCodec
  if (http2Connection != null) {
    return new Http2ExchangeCodec(client, this, chain, http2Connection);
  } else {
    socket.setSoTimeout(chain.readTimeoutMillis());
    source.timeout().timeout(chain.readTimeoutMillis(), MILLISECONDS);
    sink.timeout().timeout(chain.writeTimeoutMillis(), MILLISECONDS);
    return new Http1ExchangeCodec(client, this, source, sink);
  }
}
```

主要就是通过findHealthyConnection方法获取连接RealConnection实例，然后用RealConnection的newCodec方法获取了ExchangeCodec实例，如果是HTTP/2返回Http2ExchangeCodec，否则返回Http1ExchangeCodec，然后返回。

接下来重点看findHealthyConnection方法的实现：

```java
  private RealConnection findHealthyConnection(int connectTimeout, int readTimeout,
      int writeTimeout, int pingIntervalMillis, boolean connectionRetryEnabled,
      boolean doExtensiveHealthChecks) throws IOException {
    while (true) {
      //找连接
      RealConnection candidate = findConnection(connectTimeout, readTimeout, writeTimeout,
          pingIntervalMillis, connectionRetryEnabled);

      // 是新连接 且不是HTTP2.0 就不用检查
      // If this is a brand new connection, we can skip the extensive health checks.
      synchronized (connectionPool) {
        if (candidate.successCount == 0 && !candidate.isMultiplexed()) {
          return candidate;
        }
      }

      // 检查不健康，继续找
      // Do a (potentially slow) check to confirm that the pooled connection is still good. If it
      // isn't, take it out of the pool and start again.
      if (!candidate.isHealthy(doExtensiveHealthChecks)) {
        //标记不可用 noNewExchanges = true
        candidate.noNewExchanges();
        continue;
      }

      return candidate;
    }//...while
  }

```

循环寻找连接：如果是不健康的连接，标记不可用（标记后会移除，后面讲连接池会讲到），然后继续找。健康是指连接可以承载新的数据流，socket是连接状态。

再查看findConnection方法：

```java
  //为承载新的数据流 寻找 连接。寻找顺序是 已分配的连接、连接池、新建连接
  private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
      int pingIntervalMillis, boolean connectionRetryEnabled) throws IOException {
    boolean foundPooledConnection = false;
    RealConnection result = null;
    Route selectedRoute = null;
    RealConnection releasedConnection;
    Socket toClose;
    synchronized (connectionPool) {
      //请求已被取消（Call的cancel方法->transmitter的cancel方法），抛异常
      if (transmitter.isCanceled()) throw new IOException("Canceled");
      hasStreamFailure = false; // This is a fresh attempt.

      // 尝试使用 已给数据流分配的连接.（例如重定向请求时，可以复用上次请求的连接）
      releasedConnection = transmitter.connection;
      //有已分配的连接，但已经被限制承载新的数据流，就尝试释放掉（如果连接上已没有数据流），并返回待关闭的socket。
      toClose = transmitter.connection != null && transmitter.connection.noNewExchanges
          ? transmitter.releaseConnectionNoEvents()
          : null;

      if (transmitter.connection != null) {
        // 不为空，说明上面没有释放掉，那么此连接可用
        result = transmitter.connection;
        releasedConnection = null;
      }

      if (result == null) {
        // 尝试从连接池中获取可用的连接
        if (connectionPool.transmitterAcquirePooledConnection(address, transmitter, null, false)) {
          foundPooledConnection = true;
          result = transmitter.connection;
        } else if (nextRouteToTry != null) {
          // 如果获取不到，那么久从Route里面获取
          selectedRoute = nextRouteToTry;
          nextRouteToTry = null;
        } else if (retryCurrentRoute()) {
          // 如果当前的路由是重试的路由，那么就从路由里面获取
          selectedRoute = transmitter.connection.route();
        }
      }
    }
    closeQuietly(toClose);//（如果有）关闭待关闭的socket

    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);//（如果有）回调连接释放事件
    }
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);//(如果有）回调（从连接池）获取连接事件
    }
    if (result != null) {
      // 如果获取到连接，那么就直接返回结果
      return result;
    }

    // 如果前面没有获取到连接，那么这里就通过RouteSelector先获取到Route，然后再获取Connection。是阻塞操作。
    boolean newRouteSelection = false;
    if (selectedRoute == null && (routeSelection == null || !routeSelection.hasNext())) {
      newRouteSelection = true;
      // 通过RouteSelector获取Route
      // routeSelector在RetryAndFollowUpInterceptor#prepareToConnect的new ExchangeFinder()中创建了
      // next()获取routeSelection
      routeSelection = routeSelector.next();
    }

    List<Route> routes = null;
    synchronized (connectionPool) {
      if (transmitter.isCanceled()) throw new IOException("Canceled");

      if (newRouteSelection) {
        // 通过RouteSelector拿到Route集合（IP地址），再次尝试从缓存池中获取连接，看看是否有可以复用的连接；
        routes = routeSelection.getAll();
        if (connectionPool.transmitterAcquirePooledConnection(
            address, transmitter, routes, false)) {
          foundPooledConnection = true;
          result = transmitter.connection;
        }
      }
      //第二次连接池也没找到
      if (!foundPooledConnection) {
        // 如果上面没有获取到，那么就创建一个新的连接
        if (selectedRoute == null) {
          selectedRoute = routeSelection.next();
        }

        result = new RealConnection(connectionPool, selectedRoute);
        connectingConnection = result;
      }
    }

    // 如果第二次从连接池的尝试成功了，结束，因为连接池中的连接是已经和服务器建立连接的
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);
      return result;
    }
    // 开始TCP握手和TSL握手，这是一个阻塞的过程
    result.connect(connectTimeout, readTimeout, writeTimeout, pingIntervalMillis,
        connectionRetryEnabled, call, eventListener);
    connectionPool.routeDatabase.connected(result.route());

    Socket socket = null;
    synchronized (connectionPool) {
      connectingConnection = null;
      // 第三次，也是最后一次尝试从连接池获取，注意最后一个参数为true，即要求 多路复用（http2.0）
      // 意思是，如果本次是http2.0，那么为了保证 多路复用性，
      // （因为上面的握手操作不是线程安全）会再次确认连接池中此时是否已有同样连接
      if (connectionPool.transmitterAcquirePooledConnection(address, transmitter, routes, true)) {
        // 如果获取到，就关闭我们创建里的连接，返回获取的连接
        result.noNewExchanges = true;
        socket = result.socket();
        result = transmitter.connection;

        // 那么这个刚刚连接成功的路由 就可以 用作下次 尝试的路由
        nextRouteToTry = selectedRoute;
      } else {
        // 将连接成功的RealConnection放到缓存池中，用于后续复用
        connectionPool.put(result);
        //transmitter和连接关联起来
        transmitter.acquireConnectionNoEvents(result);
      }
    }
    closeQuietly(socket);//如果刚刚建立的连接没用到，就关闭

    eventListener.connectionAcquired(call, result);
    return result;
  }
```

代码看着很长，已经加了注释，方法目的就是为承载新的数据流寻找连接。寻找顺序是 已分配的连接、连接池、新建连接。如下：

1. 首先会尝试使用 已给数据流分配的连接。（已分配连接的情况例如重定向时的再次请求，说明上次已经有了连接）
2. 若没有 已分配的可用连接，就尝试从连接池中 匹配获取。因为此时没有路由信息，所以匹配条件：address一致——host、port、代理等一致，且 匹配的连接可以接受新的数据流。
3. 若从连接池没有获取到，则取下一个代理的路由信息（多个Route，即多个IP地址），再次尝试从连接池获取，此时可能因为连接合并而匹配到。
4. 若第二次也没有获取到，就创建RealConnection实例，进行TCP + TLS 握手，与服务端建立连接。
5. 此时为了确保Http2.0连接的多路复用性，会第三次从连接池匹配。因为新建立的连接的握手过程是非线程安全的，所以此时可能连接池新存入了相同的连接。
6. 第三次若匹配到，就使用已有连接，释放刚刚新建的连接；若未匹配到，则把新连接存入连接池并返回。

![findConnection方法流程](https://img-blog.csdnimg.cn/20200612152339877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hmeTg5NzE2MTM=,size_16,color_FFFFFF,t_70#pic_center)



### RouteSelector

首先我们来看Route类

```java
public final class Route {
  final Address address;
  final Proxy proxy;//代理
  final InetSocketAddress inetSocketAddress;//连接目标地址

  public Route(Address address, Proxy proxy, InetSocketAddress inetSocketAddress) {
    if (address == null) {
      throw new NullPointerException("address == null");
    }
    if (proxy == null) {
      throw new NullPointerException("proxy == null");
    }
    if (inetSocketAddress == null) {
      throw new NullPointerException("inetSocketAddress == null");
    }
    this.address = address;
    this.proxy = proxy;
    this.inetSocketAddress = inetSocketAddress;
  }
    ...
}
```

**Route，通过代理服务器信息 proxy、连接目标地址 InetSocketAddress 来描述一条 连接服务器的具体路由**。

- proxy代理：可以为客户端显式配置代理服务器。否则，将使用ProxySelector代理选择器。可能会返回多个代理。
- IP地址：无论是直连还是代理，打开socket连接都需要IP地址。 DNS服务可能返回多个IP地址尝试。

上面分析的findConnection方法中是使用routeSelection.getAll()获取Route集合routes，而routeSelection是通过routeSelector.next()获取，routeSelector是在ExchangeFinder的构造方法内创建的，也就是说routeSelector在RetryAndFollowUpInterceptor中就创建了，那么我们看下RouteSelector：

```java
  RouteSelector(Address address, RouteDatabase routeDatabase, Call call,
      EventListener eventListener) {
    this.address = address;
    this.routeDatabase = routeDatabase;//注意:这个是连接池中的路由黑名单（连接失败的路由）
    this.call = call;
    this.eventListener = eventListener;
    //收集代理服务器
    resetNextProxy(address.url(), address.proxy());
  }

  private void resetNextProxy(HttpUrl url, Proxy proxy) {
    if (proxy != null) {
      // 若指定了代理，那么就这一个。（就是初始化OkhttpClient时配置的）
      // If the user specifies a proxy, try that and only that.
      proxies = Collections.singletonList(proxy);
    } else {
      // 没配置就使用ProxySelector获取代理
      // （若初始化OkhttpClient时没有配置ProxySelector，会使用系统默认的）
      // Try each of the ProxySelector choices until one connection succeeds.
      List<Proxy> proxiesOrNull = address.proxySelector().select(url.uri());
      proxies = proxiesOrNull != null && !proxiesOrNull.isEmpty()
          ? Util.immutableList(proxiesOrNull)
          : Util.immutableList(Proxy.NO_PROXY);
    }
    nextProxyIndex = 0;
  }
```

注意到RouteSelector的构造方法中传入了routeDatabase，是连接失败的路由黑名单（后面连接池也会讲到），并使用resetNextProxy方法获取代理服务器列表：若没有指定proxy就是用ProxySelector获取proxy列表（若没有配置ProxySelector会使用系统默认）。接着看next方法：

```java
public Selection next() throws IOException {
  if (!hasNext()) {
    throw new NoSuchElementException();
  }
  //还有下一个
  // Compute the next set of routes to attempt.
  List<Route> routes = new ArrayList<>();
  while (hasNextProxy()) {
    // Postponed routes are always tried last. For example, if we have 2 proxies and all the
    // routes for proxy1 should be postponed, we'll move to proxy2. Only after we've exhausted
    // all the good routes will we attempt the postponed routes.
    //获取下一个代理Proxy的代理去尝试
    //nextProxy->resetNextInetSocketAddress->inetSocketAddresses.add
    Proxy proxy = nextProxy();
    // 遍历Proxy经DNS后的所有IP地址，组装成Route
    for (int i = 0, size = inetSocketAddresses.size(); i < size; i++) {
      Route route = new Route(address, proxy, inetSocketAddresses.get(i));
      //此路由在黑名单中，存起来最后尝试
      if (routeDatabase.shouldPostpone(route)) {
        postponedRoutes.add(route);
      } else {
        //保持起来
        routes.add(route);
      }
    }

    if (!routes.isEmpty()) {
      break;
    }
  }
  // 若没有拿到路由，就尝试上面存的黑名单的路由
  if (routes.isEmpty()) {
    // We've exhausted all Proxies so fallback to the postponed routes.
    routes.addAll(postponedRoutes);
    postponedRoutes.clear();
  }

  //routes包装成Selection返回
  return new Selection(routes);
}
```

next方法主要就是获取下一个代理Proxy的代理信息，即多个路由。具体是在resetNextInetSocketAddress方法中实现，主要是对代理服务地址进行DNS解析获取多个IP地址，这里就不展开了，具体可以参考[OkHttp中的代理和路由](https://www.jianshu.com/p/63ba15d8877a)。

### ConnectionPool

ConnectionPool，即连接池，用于管理http1.1/http2.0连接重用，以减少网络延迟。相同Address的http请求可以共享一个连接，ConnectionPool就是实现了连接的复用。

OkHttpClient初始化时，实例化了ConnectionPool

```java
 connectionPool = new ConnectionPool();
```

代码如下：

```java
public final class ConnectionPool {
  final RealConnectionPool delegate;

  /**
   * Create a new connection pool with tuning parameters appropriate for a single-user application.
   * The tuning parameters in this pool are subject to change in future OkHttp releases. Currently
   * this pool holds up to 5 idle connections which will be evicted after 5 minutes of inactivity.
   */
  public ConnectionPool() {
    //默认的连接池，最大连接5，保持Alive持续时间5min
    this(5, 5, TimeUnit.MINUTES);
  }

  public ConnectionPool(int maxIdleConnections, long keepAliveDuration, TimeUnit timeUnit) {
    this.delegate = new RealConnectionPool(maxIdleConnections, keepAliveDuration, timeUnit);
  }
  //返回空闲连接数
  /** Returns the number of idle connections in the pool. */
  public int idleConnectionCount() {
    return delegate.idleConnectionCount();
  }
  //返回池子中的连接数
  /** Returns total number of connections in the pool. */
  public int connectionCount() {
    return delegate.connectionCount();
  }
  //关闭并移除所以空闲连接
  /** Close and remove all idle connections in the pool. */
  public void evictAll() {
    delegate.evictAll();
  }
}

```

ConnectionPool默认配置是最大空闲连接数5，最大空闲时间5分钟（即一个连接空闲时间超过5分钟就移除），我们也可以在初始化okhttpClient时进行不同的配置。

需要注意的是ConnectionPool是用于应用层，实际管理者是RealConnectionPool。RealConnectionPool是okhttp内部真实管理连接的地方。用到了代理模式。

RealConnectionPool实现如下：

```java
public final class RealConnectionPool {

  //线程池，用于清理过期的连接。一个连接池最多运行一个线程。
  private static final Executor executor = new ThreadPoolExecutor(0 /* corePoolSize */,
      Integer.MAX_VALUE /* maximumPoolSize */, 60L /* keepAliveTime */, TimeUnit.SECONDS,
      new SynchronousQueue<>(), Util.threadFactory("OkHttp ConnectionPool", true));

  /** The maximum number of idle connections for each address. */
  //ConnectionPool的构造方法
  //每个ip地址的最大空闲连接数，为5个
  private final int maxIdleConnections;
  //空闲连接存活时间，为5分钟
  private final long keepAliveDurationNs;

  //连接队列
  private final Deque<RealConnection> connections = new ArrayDeque<>();
  final RouteDatabase routeDatabase = new RouteDatabase();
  boolean cleanupRunning;

  public RealConnectionPool(int maxIdleConnections, long keepAliveDuration, TimeUnit timeUnit) {
    this.maxIdleConnections = maxIdleConnections;
    this.keepAliveDurationNs = timeUnit.toNanos(keepAliveDuration);

    // Put a floor on the keep alive duration, otherwise cleanup will spin loop.
    if (keepAliveDuration <= 0) {
      throw new IllegalArgumentException("keepAliveDuration <= 0: " + keepAliveDuration);
    }
  }

  public synchronized int idleConnectionCount() {
    int total = 0;
    for (RealConnection connection : connections) {
      if (connection.transmitters.isEmpty()) total++;
    }
    return total;
  }

  public synchronized int connectionCount() {
    return connections.size();
  }
  ...
}
```

**存：**

```java
  private final Runnable cleanupRunnable = () -> {
    //循环清理
    while (true) {
      //移除连接，executor运行cleanupRunnable，调用该方法
      long waitNanos = cleanup(System.nanoTime());
      if (waitNanos == -1) return;
      if (waitNanos > 0) {
        long waitMillis = waitNanos / 1000000L;
        waitNanos -= (waitMillis * 1000000L);
        synchronized (RealConnectionPool.this) {
          try {
            //下一次清理之前的等待
            RealConnectionPool.this.wait(waitMillis, (int) waitNanos);
          } catch (InterruptedException ignored) {
          }
        }
      }
    }
  };

  //存连接
  void put(RealConnection connection) {
    assert (Thread.holdsLock(this));
    //未正在清理，加之前先清理一下
    if (!cleanupRunning) {
      cleanupRunning = true;
      executor.execute(cleanupRunnable);
    }
    connections.add(connection);
  }

```

connections是用于存连接的队列Deque。看到在add之前，使用线程池executor执行了cleanupRunnable，意思是清理连接。并且注意到清理是一个循环，并且下一次清理前要等待waitNanos时间。我们看下cleanup方法：

```java
long cleanup(long now) {
  int inUseConnectionCount = 0;//正在使用的连接数
  int idleConnectionCount = 0;//空闲连接数
  RealConnection longestIdleConnection = null;//空闲时间最长的连接
  long longestIdleDurationNs = Long.MIN_VALUE;//最长的空闲时间

  // 遍历连接：找到待清理的连接, 找到下一次要清理的时间（还未到最大空闲时间）
  // Find either a connection to evict, or the time that the next eviction is due.
  synchronized (this) {
    //遍历连接
    for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
      RealConnection connection = i.next();

      //若连接正在使用，continue，正在使用连接数+1
      // If the connection is in use, keep searching.
      if (pruneAndGetAllocationCount(connection, now) > 0) {
        inUseConnectionCount++;
        continue;
      }

      //空闲连接数+1
      idleConnectionCount++;

      // 赋值最长的空闲时间和对应连接
      // If the connection is ready to be evicted, we're done.
      long idleDurationNs = now - connection.idleAtNanos;
      if (idleDurationNs > longestIdleDurationNs) {
        longestIdleDurationNs = idleDurationNs;
        longestIdleConnection = connection;
      }
    }//...for

    //若最长的空闲时间大于5分钟 或 空闲数 大于5，就移除并关闭这个连接
    if (longestIdleDurationNs >= this.keepAliveDurationNs
        || idleConnectionCount > this.maxIdleConnections) {
      // We've found a connection to evict. Remove it from the list, then close it below (outside
      // of the synchronized block).
      // 移除连接
      connections.remove(longestIdleConnection);
    } else if (idleConnectionCount > 0) {
      // else 空闲连接数 > 0，就返回 还剩多久到达5分钟，然后wait这个时间再来清理
      // A connection will be ready to evict soon.
      return keepAliveDurationNs - longestIdleDurationNs;
    } else if (inUseConnectionCount > 0) {
      // 连接正在使用没有空闲，就5分钟后再尝试清理.
      // All connections are in use. It'll be at least the keep alive duration 'til we run again.
      return keepAliveDurationNs;
    } else {
      // 没有连接，不清理
      // No connections, idle or in use.
      cleanupRunning = false;
      return -1;
    }
  }

  //关闭移除的连接
  closeQuietly(longestIdleConnection.socket());

  // 关闭移除后 立刻 进行下一次的 尝试清理
  // Cleanup again immediately.
  return 0;
}
```

- 有空闲连接的话，如果最长的空闲时间大于5分钟 或 空闲数 大于5，就移除关闭这个最长空闲连接；如果 空闲数 不大于5 且 最长的空闲时间不大于5分钟，就返回到5分钟的剩余时间，然后等待这个时间再来清理。
- 没有空闲连接就等5分钟后再尝试清理。
- 没有连接不清理。

其中判断连接正在使用的方法pruneAndGetAllocationCount我们来看下：

```java
 private int pruneAndGetAllocationCount(RealConnection connection, long now) {
    //连接上的数据流，弱引用列表
    List<Reference<Transmitter>> references = connection.transmitters;
    for (int i = 0; i < references.size(); ) {
      Reference<Transmitter> reference = references.get(i);

      if (reference.get() != null) {
        i++;
        continue;
      }

      // 到这里，transmitter是泄漏的，要移除
      // 且此连接不能再承载新的数据流（泄漏的原因就是用户忘记close response body）
      // We've discovered a leaked transmitter. This is an application bug.
      TransmitterReference transmitterRef = (TransmitterReference) reference;
      String message = "A connection to " + connection.route().address().url()
          + " was leaked. Did you forget to close a response body?";
      Platform.get().logCloseableLeak(message, transmitterRef.callStackTrace);

      references.remove(i);
      connection.noNewExchanges = true;

      //连接因为泄漏没有数据流了，那么可以立即移除了。所以设置 开始空闲时间 是5分钟前
      // If this was the last allocation, the connection is eligible for immediate eviction.
      if (references.isEmpty()) {
        connection.idleAtNanos = now - keepAliveDurationNs;
        return 0;
      }
    }
    //返回连接上的数据流数量，大于0说明正在使用。
    return references.size();
  }

```

其中connection.transmitters，表示在此连接上的数据流，transmitters size大于1即表示多个请求复用此连接。

另外，在findConnection中，使用connectionPool.put(result)存连接后，又调用transmitter.acquireConnectionNoEvents方法：

```java
  void acquireConnectionNoEvents(RealConnection connection) {
    assert (Thread.holdsLock(connectionPool));

    if (this.connection != null) throw new IllegalStateException();
    this.connection = connection;
    //先把连接赋给transmitter,表示数据流transmitter依附在这个connection上
    //add 这个transmitter的弱引用
    connection.transmitters.add(new TransmitterReference(this, callStackTrace));
  }
```

先把连接赋给transmitter，表示数据流transmitter依附在这个connection上；然后connection.transmitters add 这个transmitter的弱引用，connection.transmitters表示这个连接承载的所有数据流，即承载的所有请求。

主要就是把连接存入队列，同时开始循环尝试清理过期连接。

**取：**

```java
  //获取连接
  boolean transmitterAcquirePooledConnection(Address address, Transmitter transmitter,
      @Nullable List<Route> routes, boolean requireMultiplexed) {
    assert (Thread.holdsLock(this));
    for (RealConnection connection : connections) {
      //要求多路复用，跳过不支持多路复用的连接
      if (requireMultiplexed && !connection.isMultiplexed()) continue;
      //不合条件，跳过
      if (!connection.isEligible(address, routes)) continue;
      //给Transmitter分配一个连接
      transmitter.acquireConnectionNoEvents(connection);
      return true;
    }
    return false;
  }

```

存的方法名是put，取的方法名却不是get，transmitterAcquirePooledConnection意思是 为transmitter 从连接池 获取连接，实际上transmitter就代表一个数据流，也就是一个http请求。注意到，在遍历中 经过判断后也是transmitter的acquireConnectionNoEvents方法，即把匹配到的connection赋给transmitter。

继续看是如何匹配的：如果requireMultiplexed为false，即不是多路复用（不是http/2），那么就要看Connection的isEligible方法了，isEligible方法返回true，就代表匹配成功：

```java
  //用于判断 连接 是否 可以承载指向address的数据流
  boolean isEligible(Address address, @Nullable List<Route> routes) {
    // If this connection is not accepting new exchanges, we're done.
    // 连接不再接受新的数据流，false
    if (transmitters.size() >= allocationLimit || noNewExchanges) return false;

    // If the non-host fields of the address don't overlap, we're done.
    // 匹配address中非host的部分
    if (!Internal.instance.equalsNonHost(this.route.address(), address)) return false;

    // If the host exactly matches, we're done: this connection can carry the address.
    // 匹配address的host，到这里也匹配的话，就return true
    if (address.url().host().equals(this.route().address().url().host())) {
      return true; // This connection is a perfect match.
    }

    // At this point we don't have a hostname match. But we still be able to carry the request if
    // our connection coalescing requirements are met. See also:
    // https://hpbn.co/optimizing-application-delivery/#eliminate-domain-sharding
    // https://daniel.haxx.se/blog/2016/08/18/http2-connection-coalescing/
    //到这里hostname是没匹配的，但是还是有机会返回true：连接合并
    // 1. 连接须是 HTTP/2.
    // 1. This connection must be HTTP/2.
    if (http2Connection == null) return false;
    // 2. IP 地址匹配
    // 2. The routes must share an IP address.
    if (routes == null || !routeMatchesAny(routes)) return false;
    // 3. 证书匹配
    // 3. This connection's server certificate's must cover the new host.
    if (address.hostnameVerifier() != OkHostnameVerifier.INSTANCE) return false;
    if (!supportsUrl(address.url())) return false;
    // 4. 证书 pinning 匹配.
    // 4. Certificate pinning must match the host.
    try {
      address.certificatePinner().check(address.url().host(), handshake().peerCertificates());
    } catch (SSLPeerUnverifiedException e) {
      return false;
    }

    return true; // The caller's address can be carried by this connection.
  }
```

取的过程就是遍历连接池，进行地址等一系列匹配。

**删：**

```java
 //移除关闭空闲连接
  public void evictAll() {
    List<RealConnection> evictedConnections = new ArrayList<>();
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();
        //如果连接上的transmitters是空，那么就从连接池移除并且关闭。
        if (connection.transmitters.isEmpty()) {
          connection.noNewExchanges = true;
          evictedConnections.add(connection);
          i.remove();
        }
      }
    }

    for (RealConnection connection : evictedConnections) {
      closeQuietly(connection.socket());
    }
  }
```

遍历连接池，如果连接上的数据流是空，那么就从连接池移除并且关闭。

我们回过头看下Transmitter的releaseConnectionNoEvents方法，如果连接不再接受新的数据流，就会调用这个方法：

```java
  //从连接上移除transmitter
  @Nullable Socket releaseConnectionNoEvents() {
    assert (Thread.holdsLock(connectionPool));

    int index = -1;
    //遍历 此数据流依附的连接 上的所有数据流，找到index
    for (int i = 0, size = this.connection.transmitters.size(); i < size; i++) {
      Reference<Transmitter> reference = this.connection.transmitters.get(i);
      if (reference.get() == this) {
        index = i;
        break;
      }
    }

    if (index == -1) throw new IllegalStateException();
    //transmitters移除此数据流
    RealConnection released = this.connection;
    released.transmitters.remove(index);
    this.connection = null;
    //如果连接上没有有数据流了，就置为空闲（等待清理），并返回待关闭的socket
    if (released.transmitters.isEmpty()) {
      released.idleAtNanos = System.nanoTime();
      if (connectionPool.connectionBecameIdle(released)) {
        return released.socket();
      }
    }

    return null;
  }

```

主要就是尝试释放连接，连接上没有数据流就关闭socket等待被清理。

好了，到这里连接池的管理就分析完了。从连接的查找到连接池的管理，就是ConnectInterceptor的内容了。



---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)