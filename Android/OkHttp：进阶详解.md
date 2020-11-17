# OkHttp：进阶详解

在[《OkHttp：基本使用详解》](https://blog.csdn.net/baidu_32237719/article/details/100125359)中描述了OkHttp的基本使用方法。已掌握了如何使用OkHttp进行基本的网络请求，但这只是最基础的使用，想要在实际的开发项目中应用我们还需知道OkHttp的底层实现方式。

本文[《OkHttp：进阶详解》](https://blog.csdn.net/baidu_32237719/article/details/109743388)主要详细的描述OkHttp的在设计和使用时涉及到的相关概念，为后续阅读源码打下基础。

接下来通过[《OkHttp：源码详解》](https://blog.csdn.net/baidu_32237719/article/details/109743808)一系列文章分析源码。

## 一、Call

HTTP客户端的工作是接受网络请求并产生其响应。

### 请求

每个HTTP请求都包含一个URL，一个方法（如GET或POST）和 header 列表。请求可能还包含body：特定内容类型的数据流。

### 响应

响应使用code（例如200表示成功或404表示找不到），header 和其自己的可选body来回答请求。

### 重写请求

为了确保正确性和效率，OkHttp在传输请求之前会先对其进行重写。

OkHttp可以添加header，包括`Content-Length`，`Transfer-Encoding`，`User-Agent`，`Host`，`Connection`，和`Content-Type`。

如果`Accept-Encoding` header未存在，将添加用于透明响应压缩的标题。

如果使用Cookie，OkHttp将在其中添加Cookie标题。

一些请求将具有缓存的响应。当此缓存的响应不是最新的，OkHttp可以执行条件GET来下载更新的响应。这种请求会在header添加`If-Modified-Since`和`If-None-Match`。

### 重写响应

如果使用透明压缩，则OkHttp将删除相应的响应标头`Content-Encoding`和`Content-Length`，它们不适用于解压缩的响应正文。

如果有条件的GET成功，则按照规范的指示将来自网络和缓存的响应合并。

### 后续请求

当请求的URL被移动，网络服务器将返回一个响应代码，例如`302`指示文档的新URL。OkHttp将遵循重定向以检索最终响应。

如果响应发出授权验证，OkHttp将要求[`Authenticator`](http://square.github.io/okhttp/4.x/okhttp/okhttp3/-authenticator/)（如果已配置）进行验证。如果身份验证器提供了证书，则将使用该证书重试请求。

### 重试请求

有时连接失败：池化连接已断开连接，或者无法访问Web服务器本身。如果其他路由可用，OkHttp将重试该请求。

### Cell

通过重写，重定向，跟进和重试，一个简单请求可能会产生许多请求和响应。OkHttp用于`Call`对这一过程（通过中间请求和响应来满足用户真正的请求）进行建模。若URL重定向或故障转移到备用IP地址，代码将继续起作用。

调用以两种方式之一执行：

- **同步：**线程阻塞，直到响应可读为止。
- **异步：**可以将请求放入任何线程中，并在响应可读时在另一个线程上被调用。

可以从任何线程取消Cell。如果尚未完成，这将使Cell失败。正在写请求body或读取响应body的时消其调用，将抛出`IOException`。

### Dispatch

对于同步调用，需要带上自己的线程，并自己管理并发的请求数量。同时连接过多会浪费资源。太少会损害延迟。

对于异步调用，`Dispatcher`实现最大同时请求数的策略。可以设置每个Web服务器的最大值（默认为5）和整体（默认为64）。

## 二、Caching

OkHttp实现一个可选的，**默认情况下关闭Cache**。

### 基本用法

```java
         File cacheDirectory = new File(Environment.getExternalStorageDirectory() + "/cache");
        int cacheSize = 10 * 1024 * 1024; // 10 MiB
        Cache cache = new Cache(cacheDirectory, cacheSize);

        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .cache(cache)
                .build();
```

### EventListener事件

缓存事件通过EventListener API公开。典型场景如下:

#### 1）缓存命中

在理想情况下，缓存可以满足请求，而无需对网络进行任何调用。这将跳过常规事件，例如DNS，连接到网络以及下载响应正文。

根据HTTP RFC的建议，文档的最长期限默认为基于“上次修改时间”到收到文档时的间隔的10％。

- Call开始
- **缓存命中**
- Call结束

#### 2）缓存未命中 

在高速缓存未命中下，可以看到正常的请求事件，但还有一个附加事件显示了高速缓存的存在。

如果尚未从网络中读取，无法缓存或基于响应缓存header超过了其生存期，则通常会出现缓存未命中。

- Call开始
- **缓存未命中**
- ProxySelectStart
- …标准事件…
- Call结束

#### 3）有条件的缓存命中 

当缓存标志需要检查缓存结果是否仍然有效时，会收到早期的cacheConditionalHit事件，然后是缓存命中或未命中。重要的是，在缓存命中的情况下，服务器不会发送响应正文。

响应将为非null`cacheResponse`和`networkResponse`。仅当响应代码为HTTP / 1.1 304 Not Modified时，才会将cacheResponse用作顶级响应。

- Call开始
- **有条件的缓存命中**
- 已获得连接
- …标准活动…
- ResponseBodyEnd *（0字节）*
- **缓存命中**
- 连接发布
- Call结束

### 缓存目录

缓存目录必须仅由单个实例拥有。

可以在不再需要时删除高速缓存。

```java
cache.delete()
```

### 裁剪缓存

可以使用evictAll删除整个缓存以临时清除空间。

```java
cache.evictAll()
```

可以使用url迭代器删除单个缓存项目。

```java
 /**
   * Returns an iterator over the URLs in this cache. This iterator doesn't throw {@code
   * ConcurrentModificationException}, but if new responses are added while iterating, their URLs
   * will not be returned. If existing responses are evicted during iteration, they will be absent
   * (unless they were already returned).
   *
   * <p>The iterator supports {@linkplain Iterator#remove}. Removing a URL from the iterator evicts
   * the corresponding response from the cache. Use this to evict selected responses.
   */  
  public Iterator<String> urls() throws IOException {
    return new Iterator<String>() {
      final Iterator<DiskLruCache.Snapshot> delegate = cache.snapshots();

      @Nullable String nextUrl;
      boolean canRemove;

      @Override public boolean hasNext() {
        if (nextUrl != null) return true;

        canRemove = false; // Prevent delegate.remove() on the wrong item!
        while (delegate.hasNext()) {
          try (DiskLruCache.Snapshot snapshot = delegate.next()) {
            BufferedSource metadata = Okio.buffer(snapshot.getSource(ENTRY_METADATA));
            nextUrl = metadata.readUtf8LineStrict();
            return true;
          } catch (IOException ignored) {
            // We couldn't read the metadata for this snapshot; possibly because the host filesystem
            // has disappeared! Skip it.
          }
        }

        return false;
      }

      @Override public String next() {
        if (!hasNext()) throw new NoSuchElementException();
        String result = nextUrl;
        nextUrl = null;
        canRemove = true;
        return result;
      }

      @Override public void remove() {
        if (!canRemove) throw new IllegalStateException("remove() before next()");
        delegate.remove();
      }
    };
  }

```

若有效的可缓存响应未缓存，需要确保是否已完全读取响应，因为除非它们被完整读取，取消或停止，否则将不会缓存响应。

### 覆盖普通的缓存行为

```java
public final class Cache implements Closeable, Flushable {}
```

将HTTP和HTTPS响应缓存到文件系统，以便可以重复使用它们，从而节省时间和带宽。

#### 1）缓存优化

为了衡量缓存的有效性，此类跟踪三个统计信息：

- **请求计数：**自创建此缓存以来发出的HTTP请求数。
- **网络计数：**需要网络使用的那些请求的数量。
- **命中计数：**高速缓存为其响应提供服务的那些请求的数量。

有时，请求将导致条件缓存命中。如果缓存包含响应的过期副本，则客户端将发出条件`GET`。然后，服务器将发送更新后的响应（如果已更改），或者发送简短的“未修改”响应（如果客户端的副本仍然有效）。这样的响应会同时增加网络计数和点击计数。

提高缓存命中率的最佳方法是配置Web服务器以返回可缓存的响应。尽管此客户端支持所有[HTTP / 1.1（RFC 7234）](http://tools.ietf.org/html/rfc7234)缓存头，但它不缓存部分响应。

#### 2）强制网络响应

在某些情况下，例如在用户单击“刷新”按钮后，可能有必要跳过缓存并直接从服务器获取数据。要强制完全刷新，请添加`no-cache` 指令：

```java
Request request = new Request.Builder()
    .cacheControl(new CacheControl.Builder().noCache().build())
    .url("http://publicobject.com/helloworld.txt")
    .build();
```

如果仅需要强制服务器验证缓存的响应，请改用更有效的`max-age=0`指令：

```java
Request request = new Request.Builder()
    .cacheControl(new CacheControl.Builder()
        .maxAge(0, TimeUnit.SECONDS)
        .build())
    .url("http://publicobject.com/helloworld.txt")
    .build();
```

#### 3）强制执行缓存响应

若想显示资源是否立即可用，以便您的应用程序可以在等待最新数据下载时显示某些内容，可以将请求限制为本地缓存的资源，请添加`only-if-cached` （表示不进行网络请求,完全只使用缓存,若缓存不命中,则返回503错误）指令：

```java
Request request = new Request.Builder()
    .cacheControl(new CacheControl.Builder()
        .onlyIfCached()
        .build())
    .url("http://publicobject.com/helloworld.txt")
    .build();
Response forceCacheResponse = client.newCall(request).execute();
if (forceCacheResponse.code() != 504) {
  // The resource was cached! Show it.
} else {
  // The resource was not cached.
}
```

在过时的响应比没有响应更好的情况下，此技术甚至更好。要允许过时的缓存响应，请使用`max-stale`以秒为单位的最大过时的指令：

```java
Request request = new Request.Builder()
    .cacheControl(new CacheControl.Builder()
        .maxStale(365, TimeUnit.DAYS)
        .build())
    .url("http://publicobject.com/helloworld.txt")
    .build();
```

该`CacheControl`类可以配置请求高速缓存的指令和响应解析缓存指令。它甚至提供了方便的常量`CacheControl.FORCE_NETWORK`和 `CacheControl.FORCE_CACHE`，可以解决上述用例。

## 三、Connection

OkHttp使用以下三种类型计划其与Web服务器的连接：URL，地址和路由。

### URL

URL是抽象的：

- 指定调用可以是纯文本（`http`）或加密（`https`），但不应该使用哪种加密算法。没有指定如何验证对等方的证书（[HostnameVerifier](http://developer.android.com/reference/javax/net/ssl/HostnameVerifier.html)）或可以信任的证书（[SSLSocketFactory](http://developer.android.com/reference/org/apache/http/conn/ssl/SSLSocketFactory.html)）。
- 他们没有指定是否应使用特定的代理服务器或如何向该代理服务器进行身份验证。

也是具体的：每个URL标识一个特定的路径（如`/square/okhttp`）和查询（如`?q=sharks&lang=en`）。每个Web服务器托管许多URL。

### 地址

地址指定了一个Web服务器（例如`github.com`）以及连接到该服务器所需的所有**静态**配置：端口号，HTTPS设置和首选的网络协议（例如HTTP / 2或SPDY）。

共享相同地址的URL也可能共享相同的基础TCP套接字连接。共享连接具有显着的性能优势：更低的延迟，更高的吞吐量（由于[TCP启动缓慢](http://www.igvita.com/2011/10/20/faster-web-vs-tcp-slow-start/)）和节省的电池。OkHttp使用一个[ConnectionPool](http://square.github.io/okhttp/4.x/okhttp/okhttp3/-connection-pool/)，它可以自动重用HTTP / 1.x连接并多路复用HTTP / 2和SPDY连接。

在OkHttp中，地址的某些字段来自URL（方案，主机名，端口），其余部分来自OkHttpClient。

### 路由

路由提供实际连接到Web服务器所需的**动态**信息。这是要尝试的特定IP地址（由DNS查询发现），要使用的确切代理服务器（如果正在使用[ProxySelector](http://developer.android.com/reference/java/net/ProxySelector.html)）以及要协商的TLS版本（HTTPS连接）。

一个地址可能有很多路由。例如，托管在多个数据中心中的Web服务器可能会在其DNS响应中产生多个IP地址。

### 连接数

当您使用OkHttp请求URL时，它的作用是：

1. 它使用URL并配置了OkHttpClient来创建**地址**。该地址指定了我们如何连接到Web服务器。
2. 它尝试从**连接池中**检索具有该地址的**连接**。
3. 如果在池中找不到连接，则选择要尝试的**路由**。这通常意味着发出DNS请求以获取服务器的IP地址。然后根据需要选择TLS版本和代理服务器。
4. 如果是新路由，则通过建立直接套接字连接，TLS隧道（用于HTTP代理上的HTTPS）或直接TLS连接来进行连接。它根据需要执行TLS握手。
5. 它发送HTTP请求并读取响应。

如果连接有问题，OkHttp将选择其他路由，然后重试。当服务器地址的一部分无法访问时，这可以使OkHttp恢复。当池连接超时或不支持尝试的TLS版本时，也很有用。

收到响应后，连接将返回到池中，以便可以将其重新用于以后的请求。闲置一段时间后，连接将从池中退出。

## 四、Events

通过事件Event，可以捕获应用程序的HTTP调用的cell。可以使用事件Event来监视：

- 应用程序发出的HTTP cell的大小和频率。
- 这些cell在基础网络上的性能。如果网络性能不足，则需要改善网络或减少使用。

### EventListener

继承[EventListener](https://square.github.io/okhttp/3.x/okhttp/okhttp3/EventListener.html)并重写感兴趣的事件的方法。

在成功HTTP（没有重定向或重试）调用中，此流程描述了事件的顺序。

![Events Diagram](https://square.github.io/okhttp/images/events%402x.png)


这是一个示例事件侦听器，可显示带有时间戳的每个事件。

```java
class PrintingEventListener extends EventListener {
  private long callStartNanos;

  private void printEvent(String name) {
    long nowNanos = System.nanoTime();
    if (name.equals("callStart")) {
      callStartNanos = nowNanos;
    }
    long elapsedNanos = nowNanos - callStartNanos;
    System.out.printf("%.3f %s%n", elapsedNanos / 1000000000d, name);
  }

  @Override public void callStart(Call call) {
    printEvent("callStart");
  }

  @Override public void callEnd(Call call) {
    printEvent("callEnd");
  }

  @Override public void dnsStart(Call call, String domainName) {
    printEvent("dnsStart");
  }

  @Override public void dnsEnd(Call call, String domainName, List<InetAddress> inetAddressList) {
    printEvent("dnsEnd");
  }

  ...
}
```

我们make了几个call：

```java
Request request = new Request.Builder()
    .url("https://publicobject.com/helloworld.txt")
    .eventListenerFactory(new PrintingEventListener())
    .build();

System.out.println("REQUEST 1 (new connection)");
try (Response response = client.newCall(request).execute()) {
  // Consume and discard the response body.
  response.body().source().readByteString();
}

System.out.println("REQUEST 2 (pooled connection)");
try (Response response = client.newCall(request).execute()) {
  // Consume and discard the response body.
  response.body().source().readByteString();
}
```

侦听器将打印相应的事件：

```java
REQUEST 1 (new connection)
0.000 callStart
0.010 dnsStart
0.017 dnsEnd
0.025 connectStart
0.117 secureConnectStart
0.586 secureConnectEnd
0.586 connectEnd
0.587 connectionAcquired
0.588 requestHeadersStart
0.590 requestHeadersEnd
0.591 responseHeadersStart
0.675 responseHeadersEnd
0.676 responseBodyStart
0.679 responseBodyEnd
0.679 connectionReleased
0.680 callEnd
REQUEST 2 (pooled connection)
0.000 callStart
0.001 connectionAcquired
0.001 requestHeadersStart
0.001 requestHeadersEnd
0.002 responseHeadersStart
0.082 responseHeadersEnd
0.082 responseBodyStart
0.082 responseBodyEnd
0.083 connectionReleased
0.083 callEnd
```

请注意，第二个呼叫如何不触发连接事件。它重用了第一个请求后的连接，从而显着提高了性能。

### EventListener.Factory

在前面的示例中，我们使用字段`callStartNanos`跟踪每个事件的经过时间。这很方便，但是如果同时执行多个调用，它将不起作用。为了应对这种情况，使用`Factory`为每个`Call`创建一个新`EventListener`实例。这允许每个侦听器保持`call-specific`的状态。

该示例为每个`Call`创建唯一的ID，并使用该ID区分日志消息中的呼叫。

```java
class PrintingEventListener extends EventListener {
  public static final Factory FACTORY = new Factory() {
    final AtomicLong nextCallId = new AtomicLong(1L);

    @Override public EventListener create(Call call) {
      long callId = nextCallId.getAndIncrement();
      System.out.printf("%04d %s%n", callId, call.request().url());
      return new PrintingEventListener(callId, System.nanoTime());
    }
  };

  final long callId;
  final long callStartNanos;

  public PrintingEventListener(long callId, long callStartNanos) {
    this.callId = callId;
    this.callStartNanos = callStartNanos;
  }

  private void printEvent(String name) {
    long elapsedNanos = System.nanoTime() - callStartNanos;
    System.out.printf("%04d %.3f %s%n", callId, elapsedNanos / 1000000000d, name);
  }

  @Override public void callStart(Call call) {
    printEvent("callStart");
  }

  @Override public void callEnd(Call call) {
    printEvent("callEnd");
  }

  ...
}
```

我们可以使用此侦听器来监听一对并发的HTTP请求：

```java
Request washingtonPostRequest = new Request.Builder()
    .url("https://www.washingtonpost.com/")
    .eventListenerFactory(PrintingEventListener.FACTORY)
    .build();
client.newCall(washingtonPostRequest).enqueue(new Callback() {
  ...
});

Request newYorkTimesRequest = new Request.Builder()
    .url("https://www.nytimes.com/")
    .build();
client.newCall(newYorkTimesRequest).enqueue(new Callback() {
  ...
});
```

侦听器将打印相应的事件：

```java
0001 https://www.washingtonpost.com/
0001 0.000 callStart
0002 https://www.nytimes.com/
0002 0.000 callStart
0002 0.010 dnsStart
0001 0.013 dnsStart
0001 0.022 dnsEnd
0002 0.019 dnsEnd
0001 0.028 connectStart
0002 0.025 connectStart
0002 0.072 secureConnectStart
0001 0.075 secureConnectStart
0001 0.386 secureConnectEnd
0002 0.390 secureConnectEnd
0002 0.400 connectEnd
0001 0.403 connectEnd
0002 0.401 connectionAcquired
0001 0.404 connectionAcquired
0001 0.406 requestHeadersStart
0002 0.403 requestHeadersStart
0001 0.414 requestHeadersEnd
0002 0.411 requestHeadersEnd
0002 0.412 responseHeadersStart
0001 0.415 responseHeadersStart
0002 0.474 responseHeadersEnd
0002 0.475 responseBodyStart
0001 0.554 responseHeadersEnd
0001 0.555 responseBodyStart
0002 0.554 responseBodyEnd
0002 0.554 connectionReleased
0002 0.554 callEnd
0001 0.624 responseBodyEnd
0001 0.624 connectionReleased
0001 0.624 callEnd
```

这样`EventListener.Factory`还可以将指标限制为一部分呼叫。这是随机抽取10％的指标：

```java
class MetricsEventListener extends EventListener {
  private static final Factory FACTORY = new Factory() {
    @Override public EventListener create(Call call) {
      if (Math.random() < 0.10) {
        return new MetricsEventListener(call);
      } else {
        return EventListener.NONE;
      }
    }
  };

  ...
}
```

### 失败事件

当操作失败时，将调用失败方法。`connectFailed()`在建立与服务器的连接失败时调用，`callFailed()`在HTTP调用永久失败时时调用。发生故障时，`start`事件可能没有相应的`end`事件。

![事件图](https://square.github.io/okhttp/images/events_with_failures%402x.png)

### 重试和后续事件

OkHttp具有弹性，可以自动从某些连接故障中恢复。在这种情况下，该`connectFailed()`事件不是终止事件，也不是紧随其后的事件`callFailed()`。尝试重试时，事件侦听器将收到多个相同类型的事件。

单个HTTP调用可能需要发出后续请求，以处理身份验证质询，重定向和HTTP层超时。在这种情况下，可能会尝试多个连接，请求和响应。后续事件是单个调用可能触发相同类型的多个事件的另一个类型。

![事件图](https://square.github.io/okhttp/images/events_with_failures_and_retries%402x.png)

### 可用性

在OkHttp 3.11中，事件可以作为公共API使用。将来的版本可能会引入新的事件类型；需要覆盖相应的方法来处理它们。



## 五、HTTPS

OkHttp试图平衡两个相互冲突的问题：

- **连接**到尽可能多的主机。其中包括运行最新版本的[boringssl](https://boringssl.googlesource.com/boringssl/)的高级主机，以及运行旧版本的[OpenSSL](https://www.openssl.org/)的过时主机。
- 连接的**安全**性。这包括使用证书验证远程Web服务器，以及使用强密码交换的数据的私密性。

在协商与HTTPS服务器的连接时，OkHttp需要知道要提供哪些[TLS版本](http://square.github.io/okhttp/3.x/okhttp/okhttp3/TlsVersion.html)和[加密序列](http://square.github.io/okhttp/3.x/okhttp/okhttp3/CipherSuite.html)。

想要最大程度地提高连接性的客户端将包括过时的TLS版本和弱设计加密序列。

想要最大程度提高安全性的严格客户端将仅限于最新的TLS版本和最强大的加密序列。

特定的安全性与连接性决定由[ConnectionSpec](http://square.github.io/okhttp/3.x/okhttp/okhttp3/ConnectionSpec.html)实现。OkHttp包含四个内置连接规范：

- `RESTRICTED_TLS` 是一种安全配置，旨在满足更严格的合规性要求。
- `MODERN_TLS` 是连接到现代HTTPS服务器的安全配置。
- `COMPATIBLE_TLS` 是一种安全配置，可连接到安全的但不是当前HTTPS的服务器。
- `CLEARTEXT`是用于`http://`URL的不安全配置。

这些宽松地遵循了[Google Cloud Policies中](https://cloud.google.com/load-balancing/docs/ssl-policies-concepts)设置的模型。后续会[跟踪](https://github.com/square/okhttp/blob/okhttp_3.14.x/TLS_CONFIGURATION_HISTORY.md)对此政策的更改。

默认情况下，OkHttp将尝试`MODERN_TLS`连接。但是，如果配置失败，则可以通过配置客户端connectionSpecs来允许回退到`COMPATIBLE_TLS`连接。

```java
OkHttpClient client = new OkHttpClient.Builder()
    .connectionSpecs(Arrays.asList(ConnectionSpec.MODERN_TLS, ConnectionSpec.COMPATIBLE_TLS))
    .build();
```

每个规范中的TLS版本和密码套件可随每个发行版而更改。

例如，在OkHttp 2.2中，为了应对[POODLE](http://googleonlinesecurity.blogspot.ca/2014/10/this-poodle-bites-exploiting-ssl-30.html)攻击，放弃了对SSL 3.0的支持。在OkHttp 2.3中，放弃了对[RC4](http://en.wikipedia.org/wiki/RC4#Security)的支持。与桌面Web浏览器一样，保持OkHttp的最新状态是确保安全的最佳方法。

可以使用一组自定义的TLS版本和加密序列来构建自己的连接规范。例如，此配置仅限于三个备受推崇的密码套件。它的缺点是它需要Android 5.0+和类似的Web服务器。

```java
ConnectionSpec spec = new ConnectionSpec.Builder(ConnectionSpec.MODERN_TLS)
    .tlsVersions(TlsVersion.TLS_1_2)
    .cipherSuites(
          CipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
          CipherSuite.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
          CipherSuite.TLS_DHE_RSA_WITH_AES_128_GCM_SHA256)
    .build();

OkHttpClient client = new OkHttpClient.Builder()
    .connectionSpecs(Collections.singletonList(spec))
    .build();
```

### 证书固定

默认情况下，OkHttp信任主机平台的证书颁发机构。此策略可最大程度地提高连接性，但会受到诸如[2011 DigiNotar](http://www.computerworld.com/article/2510951/cybercrime-hacking/hackers-spied-on-300-000-iranians-using-fake-google-certificate.html)攻击之类的证书颁发机构的攻击。它还伪造您的HTTPS服务器的证书由证书颁发机构签名。

使用[CertificatePinner](http://square.github.io/okhttp/3.x/okhttp/okhttp3/CertificatePinner.html)限制受信任的证书和证书颁发机构。证书固定可提高安全性，但会限制您的服务器团队更新其TLS证书的能力。**没有服务器的TLS管理员的许可，请勿使用证书固定！**

```java
public final class CertificatePinning {
  private final OkHttpClient client = new OkHttpClient.Builder()
      .certificatePinner(
          new CertificatePinner.Builder()
              .add("publicobject.com", "sha256/Vjs8r4z+80wjNcr1YKepWQboSIRi63WsWXhIMN+eWys=")
              .build())
      .build();

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("https://publicobject.com/robots.txt")
        .build();

    try (Response response = client.newCall(request).execute()) {
      if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

      for (Certificate certificate : response.handshake().peerCertificates()) {
        System.out.println(CertificatePinner.pin(certificate));
      }
    }
  }

  public static void main(String... args) throws Exception {
    new CertificatePinning().run();
  }
}
```

### 自定义可信证书

[完整的代码示例](https://github.com/square/okhttp/blob/master/samples/guide/src/main/java/okhttp3/recipes/CustomTrust.java)显示了如何使用您自己的证书集替换主机平台的证书颁发机构。如上所述，**在没有服务器的TLS管理员祝福的许可下**，**请勿使用自定义证书！**

```java
  public CustomTrust() {
    // This implementation just embeds the PEM files in Java strings; most applications will
    // instead read this from a resource file that gets bundled with the application.

    HandshakeCertificates certificates = new HandshakeCertificates.Builder()
        .addTrustedCertificate(letsEncryptCertificateAuthority)
        .addTrustedCertificate(entrustRootCertificateAuthority)
        .addTrustedCertificate(comodoRsaCertificationAuthority)
        // Uncomment if standard certificates are also required.
        //.addPlatformTrustedCertificates()
        .build();

    client = new OkHttpClient.Builder()
            .sslSocketFactory(certificates.sslSocketFactory(), certificates.trustManager())
            .build();
  }

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("https://publicobject.com/helloworld.txt")
        .build();

    try (Response response = client.newCall(request).execute()) {
      if (!response.isSuccessful()) {
        Headers responseHeaders = response.headers();
        for (int i = 0; i < responseHeaders.size(); i++) {
          System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
        }

        throw new IOException("Unexpected code " + response);
      }

      System.out.println(response.body().string());
    }
  }

  public static void main(String... args) throws Exception {
    new CustomTrust().run();
  }
}
```

## 六、Interceptor

拦截器是一种强大的机制，可以监视，重写和重试cell。这是一个简单的拦截器，用于记录传出请求和传入响应。

```java
class LoggingInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Request request = chain.request();

    long t1 = System.nanoTime();
    logger.info(String.format("Sending request %s on %s%n%s",
        request.url(), chain.connection(), request.headers()));

    Response response = chain.proceed(request);

    long t2 = System.nanoTime();
    logger.info(String.format("Received response for %s in %.1fms%n%s",
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));

    return response;
  }
}
```

调用`chain.proceed(request)`是每个拦截器实现的关键部分。这种简单的方法是所有HTTP工作发生的地方，产生响应来满足请求。如果`chain.proceed(request)`被多次调用，则必须关闭先前的响应主体。

拦截器可以链接。同时具有压缩拦截器和校验拦截器：将需要确定是先压缩数据然后进行校验，还是先对数据进行校验然后进行压缩。OkHttp使用列表来跟踪拦截器，并按顺序调用拦截器。

![拦截器图](https://square.github.io/okhttp/images/interceptors%402x.png)

### 应用拦截器

拦截器被注册为 **应用拦截器** 或  **网络拦截器** 。我们将使用`LoggingInterceptor`上面的定义来显示差异。

在`OkHttpClient.Builder`上调用`addInterceptor()`注册一个应用拦截器：

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new LoggingInterceptor())
    .build();

Request request = new Request.Builder()
    .url("http://www.publicobject.com/helloworld.txt")
    .header("User-Agent", "OkHttp Example")
    .build();

Response response = client.newCall(request).execute();
response.body().close();
```

该URL`http://www.publicobject.com/helloworld.txt`重定向到`https://publicobject.com/helloworld.txt`，并且OkHttp自动遵循此重定向。

应用程序拦截器被调用**一次，**并且返回的响应`chain.proceed()`具有重定向的响应：

```
INFO: Sending request http://www.publicobject.com/helloworld.txt on null
User-Agent: OkHttp Example

INFO: Received response for https://publicobject.com/helloworld.txt in 1179.7ms
Server: nginx/1.4.6 (Ubuntu)
Content-Type: text/plain
Content-Length: 1759
Connection: keep-alive
```

可以看到被重定向了，因为`response.request().url()`和`request.url()`不同。这两个日志语句记录两个不同的URL。

### 网络拦截器

`addNetworkInterceptor()`代替`addInterceptor()`

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addNetworkInterceptor(new LoggingInterceptor())
    .build();

Request request = new Request.Builder()
    .url("http://www.publicobject.com/helloworld.txt")
    .header("User-Agent", "OkHttp Example")
    .build();

Response response = client.newCall(request).execute();
response.body().close();
```

当我们运行此代码时，拦截器将运行两次。一次用于初始请求`http://www.publicobject.com/helloworld.txt`，另一个用于重定向到`https://publicobject.com/helloworld.txt`。

```
INFO: Sending request http://www.publicobject.com/helloworld.txt on Connection{www.publicobject.com:80, proxy=DIRECT hostAddress=54.187.32.157 cipherSuite=none protocol=http/1.1}
User-Agent: OkHttp Example
Host: www.publicobject.com
Connection: Keep-Alive
Accept-Encoding: gzip

INFO: Received response for http://www.publicobject.com/helloworld.txt in 115.6ms
Server: nginx/1.4.6 (Ubuntu)
Content-Type: text/html
Content-Length: 193
Connection: keep-alive
Location: https://publicobject.com/helloworld.txt

INFO: Sending request https://publicobject.com/helloworld.txt on Connection{publicobject.com:443, proxy=DIRECT hostAddress=54.187.32.157 cipherSuite=TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA protocol=http/1.1}
User-Agent: OkHttp Example
Host: publicobject.com
Connection: Keep-Alive
Accept-Encoding: gzip

INFO: Received response for https://publicobject.com/helloworld.txt in 80.9ms
Server: nginx/1.4.6 (Ubuntu)
Content-Type: text/plain
Content-Length: 1759
Connection: keep-alive
```

网络请求还包含更多数据，例如`Accept-Encoding: gzip`这种OkHttp添加的对响应压缩的支持的标头。网络拦截器的`Chain`非空值`Connection`，可用于询问用于连接到Web服务器的IP地址和TLS配置。

### 对比

每个拦截器链都有相对的优点。

**应用拦截器**

- 无需担心中间响应，例如重定向和重试。
- 即使从缓存提供HTTP响应，也总是被调用一次。
- 遵守应用程序的原始意图。不关心OkHttp注入的标头，例如`If-None-Match`。
- 允许短路而不是call `Chain.proceed()`。
- 允许重试并多次call `Chain.proceed()`。
- 可以使用withConnectTimeout，withReadTimeout，withWriteTimeout调整呼叫超时。

**网络拦截器**

- 能够对重定向和重试之类的中间响应进行操作。
- 网络短路不会使缓存响应调用。
- 观察数据，就像通过网络传输数据一样。
- 访问`Connection`带有请求的。

### 重写请求

拦截器可以添加，删除或替换请求标头。他们还可以转换那些具有一个请求的主体。例如，如果要连接到已知支持请求主体的Web服务器，则可以使用应用程序拦截器来添加请求主体压缩。

```java
/** This interceptor compresses the HTTP request body. Many webservers can't handle this! */
final class GzipRequestInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Request originalRequest = chain.request();
    if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
      return chain.proceed(originalRequest);
    }

    Request compressedRequest = originalRequest.newBuilder()
        .header("Content-Encoding", "gzip")
        .method(originalRequest.method(), gzip(originalRequest.body()))
        .build();
    return chain.proceed(compressedRequest);
  }

  private RequestBody gzip(final RequestBody body) {
    return new RequestBody() {
      @Override public MediaType contentType() {
        return body.contentType();
      }

      @Override public long contentLength() {
        return -1; // We don't know the compressed length in advance!
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        BufferedSink gzipSink = Okio.buffer(new GzipSink(sink));
        body.writeTo(gzipSink);
        gzipSink.close();
      }
    };
  }
}
```

### 重写响应

对称地，拦截器可以重写响应头并转换响应主体。这通常比重写请求标头更危险。

例如，您可以修复服务器的错误配置的`Cache-Control`响应头，以实现更好的响应缓存：

```java
/** Dangerous interceptor that rewrites the server's cache-control header. */
private static final Interceptor REWRITE_CACHE_CONTROL_INTERCEPTOR = new Interceptor() {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Response originalResponse = chain.proceed(chain.request());
    return originalResponse.newBuilder()
        .header("Cache-Control", "max-age=60")
        .build();
  }
};
```

## 七、OkHttpClient

Cell的工厂，可用于发送HTTP请求和读取其响应。

### OkHttpClient应该共享

创建单个`OkHttpClient`实例并将其用于所有HTTP调用时，OkHttp的性能最佳。这是因为每个客户端都拥有自己的连接池和线程池。重用连接和线程可减少延迟并节省内存。相反，为每个请求创建客户端都会浪费空闲池上的资源。

用于使用`new OkHttpClient()`默认设置创建共享实例：

```java
// The singleton HTTP client.
public final OkHttpClient client = new OkHttpClient();
```

或用于使用`new OkHttpClient.Builder()`自定义设置创建共享实例：

```java
// The singleton HTTP client.
public final OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new HttpLoggingInterceptor())
    .cache(new Cache(cacheDir, cacheSize))
    .build();
```

### 使用newBuilder()自定义客户

可以使用[newBuilder](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-ok-http-client/new-builder/)自定义共享的OkHttpClient实例。这将构建共享的连接池，线程池和配置的客户端。使用构建器方法为特定目的配置派生的OkHttpClient。

此示例显示一个短的500毫秒超时的呼叫：

```java
OkHttpClient eagerClient = client.newBuilder()
    .readTimeout(500, TimeUnit.MILLISECONDS)
    .build();
Response response = eagerClient.newCall(request).execute();
```

### 关机非必须

如果保留的线程和连接保持空闲状态，它们将自动释放。但是，如果编写的应用程序需要主动释放未使用的资源，则可以这样做。

使用[shutdown()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html#shutdown())关闭调度程序的执行程序服务。这也将导致之后的cell被拒绝。

```java
client.dispatcher().executorService().shutdown();
```

使用[evictAll()](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-connection-pool/evict-all/)清除连接池。请注意，连接池的守护程序线程可能不会立即退出。

```java
client.connectionPool().evictAll();
```

如果您的客户端具有缓存，请调用[close()](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-cache/close/)。请注意，针对关闭的缓存创建cell是错误的，这样做会导致调用崩溃。

```java
client.cache().close();
```

OkHttp还使用守护程序线程进行HTTP / 2连接。如果它们保持空闲状态，它们将自动退出。

---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)