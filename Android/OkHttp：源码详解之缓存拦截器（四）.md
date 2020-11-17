# OkHttp：源码详解之缓存拦截器（四）

> 源码基于okhttp3 java版本：3.14.x

CacheInterceptor，**缓存拦截器**，提供网络请求缓存的存取。合理使用本地缓存，有效地减少网络开销、减少响应延迟。

在解析CacheInterceptor源码前，先了解下http的缓存机制： 

**第一次请求：**

![第一次请求（图片来源于网络)](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvNjMyMTMwLzIwMTcwMi82MzIxMzAtMjAxNzAyMTAxNDIxMzQyOTEtMTk3NjkyMzA3OS5wbmc?x-oss-process=image/format,png#pic_center)

**第二次请求：**

![第二次请求（图片来源于网络)](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWFnZXMyMDE1LmNuYmxvZ3MuY29tL2Jsb2cvNjMyMTMwLzIwMTcwMi82MzIxMzAtMjAxNzAyMTAxNDE0NTMzMzgtMTI2MzI3NjIyOC5wbmc?x-oss-process=image/format,png#pic_center)

上面两张图很好的解释了http的缓存机制：根据 **缓存是否过期**、**过期后是否有修改** 来决定 请求是否使用缓存。详细说明可点击了解 [彻底弄懂HTTP缓存机制及原理](https://www.cnblogs.com/chenqf/p/6386163.html)；

CacheInterceptor添加的代码如下：

```java
interceptors.add(new CacheInterceptor(client.internalCache()));//缓存处理相关的拦截器
```

我们先看CacheInterceptor的工作流程，再看`client.internalCache()`，从名字我们能看出这是一个内部缓存

### CacheInterceptor

```java
public final class CacheInterceptor implements Interceptor {
  final @Nullable InternalCache cache;

  public CacheInterceptor(@Nullable InternalCache cache) {
    this.cache = cache;
  }

  @Override public Response intercept(Chain chain) throws IOException {
    // 先获取候选缓存，前提是有配置缓存，也就是cache不为空；
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();
    // 执行获取缓存策略的逻辑
    // 缓存策略决定是否使用缓存：
    // strategy.networkRequest为null，不使用网络
    // strategy.cacheResponse为null，不使用缓存。
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    // 网络请求
    Request networkRequest = strategy.networkRequest;
    // 本地的缓存保存的请求
    Response cacheResponse = strategy.cacheResponse;

    //根据缓存策略更新统计指标：请求次数、网络请求次数、使用缓存次数
    if (cache != null) {
      cache.trackResponse(strategy);
    }

    //有缓存 但不能用，关掉
    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }
    // networkRequest == null 不能用网络
    // 如果不使用网络数据且缓存数据为空，那么返回一个504的Response，并且body为空
    // If we're forbidden from using the network and the cache is insufficient, fail.
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // 如果不需要使用网络数据，那么就直接返回缓存的数据
    // If we don't need the network, we're done.
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    /*
     * 到这里，networkRequest != null （cacheResponse可能null，可能!null）
     * 没有命中强缓存的情况下，进行网络请求，获取response
     * 先判断是否是协商缓存（304）命中，命中则更新缓存返回response
     * 未命中使用网络请求的response返回并添加缓存
     */
    Response networkResponse = null;
    try {
      // 执行后续的拦截器逻辑
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    if (cacheResponse != null) {
      // 如果缓存数据不为空并且code为304，表示数据没有变化，继续使用缓存数据；
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        cache.trackConditionalCacheHit();
        // 更新缓存数据
        cache.update(cacheResponse, response);
        return response;
      } else {
        //如果是非304，说明服务端资源有更新，就关闭缓存body
        closeQuietly(cacheResponse.body());
      }
    }

    // 协商缓存也未命中，获取网络返回的response
    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      //网络响应可缓存（请求和响应的 头 Cache-Control都不是'no-store'）
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
        // 将网络数据保存到缓存中
        // InternalCache接口，实现在Cache类中
        // Offer this request to the cache.
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      //OkHttp默认只会对get请求进行缓存
      //不是get请求就移除缓存
      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          // 缓存失效，那么就移除缓存
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }
    return response;
  }
  ...
}
```

整体思路：**使用缓存策略CacheStrategy来决定是否使用缓存及如何使用。**

总的来说需要知道：**strategy.networkRequest为null，不使用网络；strategy.cacheResponse为null，不使用缓存**。

缓存的判断逻辑都是基于CacheStrategy，CacheStrategy生产代码如下：

```java
CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
```

把当前时间戳，请求request、候选缓存cacheCandidate传入 工厂类Factory，然后调用get方法，代码如下：

```java
    public Factory(long nowMillis, Request request, Response cacheResponse) {
      this.nowMillis = nowMillis;
      this.request = request;
      this.cacheResponse = cacheResponse;

      //解析cacheResponse，把参数赋值给自己的成员变量
      if (cacheResponse != null) {
        //获取候选缓存的请求时间、响应时间，从header中获取 过期时间、修改时间、资源标记等（如果有）
        this.sentRequestMillis = cacheResponse.sentRequestAtMillis();
        this.receivedResponseMillis = cacheResponse.receivedResponseAtMillis();
        Headers headers = cacheResponse.headers();
        for (int i = 0, size = headers.size(); i < size; i++) {
          String fieldName = headers.name(i);
          String value = headers.value(i);
          if ("Date".equalsIgnoreCase(fieldName)) {
            servedDate = HttpDate.parse(value);
            servedDateString = value;
          } else if ("Expires".equalsIgnoreCase(fieldName)) {
            expires = HttpDate.parse(value);
          } else if ("Last-Modified".equalsIgnoreCase(fieldName)) {
            lastModified = HttpDate.parse(value);
            lastModifiedString = value;
          } else if ("ETag".equalsIgnoreCase(fieldName)) {
            etag = value;
          } else if ("Age".equalsIgnoreCase(fieldName)) {
            ageSeconds = HttpHeaders.parseSeconds(value, -1);
          }
        }
      }
    }

    /**
     * Returns a strategy to satisfy {@code request} using the a cached response {@code response}.
     */
    public CacheStrategy get() {
      CacheStrategy candidate = getCandidate();
      //返回策略，交给拦截器
      //使用网络请求 但是 请求配置了只能使用缓存
      //此时即使有缓存，也是过期的缓存，所以又new了实例，两个值都为null。
      if (candidate.networkRequest != null && request.cacheControl().onlyIfCached()) {
        // We're forbidden from using the network and the cache is insufficient.
        return new CacheStrategy(null, null);
      }

      return candidate;
    }
```

我们可以看到get()方法调用了getCandidate()方法：

```java
   private CacheStrategy getCandidate() {
      // 如果不使用缓存，那么就返回一个空的Response的CacheStrategy
      if (cacheResponse == null) {
        return new CacheStrategy(request, null);
      }

      // https，但没有握手，进行网络请求
      if (request.isHttps() && cacheResponse.handshake() == null) {
        return new CacheStrategy(request, null);
      }

      // 不可缓存（请求或响应的 头 Cache-Control 是'no-store'）进行网络请求
      if (!isCacheable(cacheResponse, request)) {
        return new CacheStrategy(request, null);
      }

      //请求头的Cache-Control是no-cache 或者 请求头有"If-Modified-Since"或"If-None-Match"
      CacheControl requestCaching = request.cacheControl();
      if (requestCaching.noCache() || hasConditions(request)) {
        return new CacheStrategy(request, null);
      }

      CacheControl responseCaching = cacheResponse.cacheControl();
      /* 强缓存 */
      //缓存的年龄
      long ageMillis = cacheResponseAge();
      //缓存的有效期
      long freshMillis = computeFreshnessLifetime();
      //判断强缓存是否有效，是的话就返回缓存数据
      //比较请求头里有效期，取较小值
      if (requestCaching.maxAgeSeconds() != -1) {
        freshMillis = Math.min(freshMillis, SECONDS.toMillis(requestCaching.maxAgeSeconds()));
      }
      //可接受的最小 剩余有效时间（min-fresh标示了客户端不愿意接受 剩余有效期<=min-fresh 的缓存。）
      long minFreshMillis = 0;
      if (requestCaching.minFreshSeconds() != -1) {
        minFreshMillis = SECONDS.toMillis(requestCaching.minFreshSeconds());
      }
      //可接受的最大过期时间（max-stale指令标示了客户端愿意接收一个已经过期了的缓存，例如 过期了 1小时 还可以用）
      long maxStaleMillis = 0;
      // 第一个判断：是否要求必须去服务器验证资源状态
      // 第二个判断：获取max-stale值，如果不等于-1，说明缓存过期后还能使用指定的时长
      if (!responseCaching.mustRevalidate() && requestCaching.maxStaleSeconds() != -1) {
        maxStaleMillis = SECONDS.toMillis(requestCaching.maxStaleSeconds());
      }
      //如果响应头没有要求忽略本地缓存 且 整合后的缓存年龄 小于 整合后的过期时间，那么缓存就可以用
      if (!responseCaching.noCache() && ageMillis + minFreshMillis < freshMillis + maxStaleMillis) {
        Response.Builder builder = cacheResponse.newBuilder();
        //没有满足“可接受的最小 剩余有效时间”，加个110警告
        if (ageMillis + minFreshMillis >= freshMillis) {
          builder.addHeader("Warning", "110 HttpURLConnection \"Response is stale\"");
        }
        //isFreshnessLifetimeHeuristic表示没有过期时间，那么大于一天，就加个113警告
        long oneDayMillis = 24 * 60 * 60 * 1000L;
        if (ageMillis > oneDayMillis && isFreshnessLifetimeHeuristic()) {
          builder.addHeader("Warning", "113 HttpURLConnection \"Heuristic expiration\"");
        }
        return new CacheStrategy(null, builder.build());
      }

      /* 协商缓存 修改了Request */
      // 缓存是过期的，找缓存里的Etag、lastModified、servedDate
      // Find a condition to add to the request. If the condition is satisfied, the response body
      // will not be transmitted.
      String conditionName;
      String conditionValue;
      if (etag != null) {
        // etag协商缓存
        conditionName = "If-None-Match";
        conditionValue = etag;
      } else if (lastModified != null) {
        // Last-Modified协商缓存
        conditionName = "If-Modified-Since";
        // 最后修改时间
        conditionValue = lastModifiedString;
      } else if (servedDate != null) {
        // Last-Modified协商缓存
        conditionName = "If-Modified-Since";
        // 服务器最后修改时间
        conditionValue = servedDateString;
      } else {
        // 没有协商缓存，返回一个空的Response的CacheStrategy
        return new CacheStrategy(request, null); // No condition! Make a regular request.
      }

      // 设置header
      Headers.Builder conditionalRequestHeaders = request.headers().newBuilder();
      Internal.instance.addLenient(conditionalRequestHeaders, conditionName, conditionValue);

      Request conditionalRequest = request.newBuilder()
          .headers(conditionalRequestHeaders.build())
          .build();

      //conditionalRequest表示 有条件的网络请求：
      //有缓存但过期了，去请求网络 询问服务端，还能不能用。能用侧返回304，不能则正常执行网路请求。
      return new CacheStrategy(conditionalRequest, cacheResponse);
    }
```

内容比较多，代码都加了注释，同样时进行一些判断，获取缓存策略。

1. 没有缓存、https但没有握手、不可缓存、忽略缓存或手动配置缓存过期，都是直接进行网络请求。
2. 以上都不满足时，如果缓存没过期，那么就是用缓存（可能要添加警告）。
3. 如果缓存过期了，但响应头有Etag，Last-Modified，Date，就添加这些header 进行**协商网络请求**。
4. 如果缓存过期了，且响应头**没有**设置Etag，Last-Modified，Date，就进行网络请求。

### Cache

上面的代码设计到了缓存的增删改查操作，我们接下里就简单查看缓存是如何工作的。

`client.internalCache()`方法实现如下：

```java
@Nullable InternalCache internalCache() {
  return cache != null ? cache.internalCache : internalCache;
}
```

缓存不为空时。即我们设置了cache，返回cache.internalCache，否则返回internalCache。

先回过头来看缓存是如何传入的。

查看构造方法我们发现缓存是没有默认实现的，我们通常调用cache方法传入的。

```java
        OkHttpClient client = new OkHttpClient.Builder()
                .cache(new Cache(getExternalCacheDir(),500 * 1024 * 1024))
                .build();
```

cache方法如下：

```java
    public Builder cache(@Nullable Cache cache) {
      this.cache = cache;
      this.internalCache = null;
      return this;
    }
```

实例化一个cache对象：

```java
public final class Cache implements Closeable, Flushable{
  private static final int VERSION = 201105;
  private static final int ENTRY_METADATA = 0;
  private static final int ENTRY_BODY = 1;
  private static final int ENTRY_COUNT = 2;

  //内部类实现InternalCache
  final InternalCache internalCache = new InternalCache() {
    @Override public @Nullable Response get(Request request) throws IOException {
      // 读取
      return Cache.this.get(request);
    }

    @Override public @Nullable CacheRequest put(Response response) throws IOException {
      // 写入
      return Cache.this.put(response);
    }

    @Override public void remove(Request request) throws IOException {
      Cache.this.remove(request);
    }

    @Override public void update(Response cached, Response network) {
      Cache.this.update(cached, network);
    }

    @Override public void trackConditionalCacheHit() {
      Cache.this.trackConditionalCacheHit();
    }

    @Override public void trackResponse(CacheStrategy cacheStrategy) {
      Cache.this.trackResponse(cacheStrategy);
    }
  };
  ...
    public Cache(File directory, long maxSize) {
    this(directory, maxSize, FileSystem.SYSTEM);
  }

  Cache(File directory, long maxSize, FileSystem fileSystem) {
    this.cache = DiskLruCache.create(fileSystem, directory, VERSION, ENTRY_COUNT, maxSize);
  }
  
   @Nullable Response get(Request request) {
    String key = key(request.url());//键
    DiskLruCache.Snapshot snapshot; //缓存快照
    Entry entry;
    try {
      snapshot = cache.get(key);    //cache是okhttp的DiskLruCache
      if (snapshot == null) {
        return null;                //没缓存，直接返回
      }
    } catch (IOException e) {
      // Give up because the cache cannot be read.
      return null;
    }

    try {
      //快照得到输入流，用于创建缓存条目
      entry = new Entry(snapshot.getSource(ENTRY_METADATA));
    } catch (IOException e) {
      Util.closeQuietly(snapshot);
      return null;
    }

    //得到响应
    Response response = entry.response(snapshot);

    if (!entry.matches(request, response)) {
      Util.closeQuietly(response.body());
      return null;
    }

    return response;
  }

  @Nullable CacheRequest put(Response response) {
    String requestMethod = response.request().method();

    if (HttpMethod.invalidatesCache(response.request().method())) {
      try {
        remove(response.request());
      } catch (IOException ignored) {
        // The cache cannot be written.
      }
      return null;
    }
    // 不是get请求，不缓存
    if (!requestMethod.equals("GET")) {
      // Don't cache non-GET responses. We're technically allowed to cache
      // HEAD requests and some POST requests, but the complexity of doing
      // so is high and the benefit is low.
      return null;
    }

    if (HttpHeaders.hasVaryAll(response)) {
      return null;
    }
    //封装成日志条目
    Entry entry = new Entry(response);
    DiskLruCache.Editor editor = null;
    try {
      editor = cache.edit(key(response.request().url()));
      if (editor == null) {
        return null;
      }
      //写入缓存
      entry.writeTo(editor);
      return new CacheRequestImpl(editor);
    } catch (IOException e) {
      abortQuietly(editor);
      return null;
    }
  }

  void remove(Request request) throws IOException {
    cache.remove(key(request.url()));
  }

  void update(Response cached, Response network) {
    Entry entry = new Entry(network);
    DiskLruCache.Snapshot snapshot = ((CacheResponseBody) cached.body()).snapshot;
    DiskLruCache.Editor editor = null;
    try {
      editor = snapshot.edit(); // Returns null if snapshot is not current.
      if (editor != null) {
        entry.writeTo(editor);
        editor.commit();
      }
    } catch (IOException e) {
      abortQuietly(editor);
    }
  }
  
  
}
```

在Cache类的内部实现了InternalCache接口和增删改查对应的方法。

缓存的增删改查是通过Okhttp内部的DiskLruCache实现的，原理和jakewharton的DiskLruCache是一致的，这里就简单叙述，详细了解可查看DiskLruCache的实现。



---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)