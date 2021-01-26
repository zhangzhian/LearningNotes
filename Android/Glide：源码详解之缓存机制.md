# 开发者营地|Glide: 源码详解之缓存机制

## 一、Glide缓存概述

默认情况下，Glide 在加载图片之前会依次检查是否有以下缓存：

1. 活动资源 (Active Resources)：正在使用的图片
2. 内存缓存 (Memory cache)：内存缓存中的图片
3. 资源类型（Resource）：磁盘缓存中转换过后的图片
4. 数据来源 (Data)：磁盘缓存中的原始图片

也就是说 Glide 中实际有四级缓存，前两个属于内存缓存，后两个属于磁盘缓存。以上每步是按顺序检查的，检查到哪一步有缓存就直接返回图片，否则继续检查下一步。如果都没有缓存，则 Glide 会从原始资源（File、Uri、远程图片 url 等）中加载图片。

## 二、缓存 Key

缓存功能必然要有一个唯一的缓存 Key 用来存储和查找对应的缓存数据。

上一篇文章中分析整体流程的时候有提到，是在 Engine 类的 load() 方法中生成的：

```java
  private final EngineKeyFactory keyFactory;
  public <R> LoadStatus load(
      GlideContext glideContext,
      Object model,
      Key signature,
      int width,
      int height,
      Class<?> resourceClass,
      Class<R> transcodeClass,
      Priority priority,
      DiskCacheStrategy diskCacheStrategy,
      Map<Class<?>, Transformation<?>> transformations,
      boolean isTransformationRequired,
      boolean isScaleOnlyOrNoTransform,
      Options options,
      boolean isMemoryCacheable,
      boolean useUnlimitedSourceExecutorPool,
      boolean useAnimationPool,
      boolean onlyRetrieveFromCache,
      ResourceCallback cb,
      Executor callbackExecutor) {
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;
    // 构建缓存 key
    EngineKey key =
        keyFactory.buildKey(
            model,
            signature,
            width,
            height,
            transformations,
            resourceClass,
            transcodeClass,
            options);
    // 从内存中加载资源
    ...
  }
```

我们查看`EngineKeyFactory#buildKey()`方法的实现：

```java
class EngineKeyFactory {

  @SuppressWarnings("rawtypes")
  EngineKey buildKey(
      Object model,
      Key signature,
      int width,
      int height,
      Map<Class<?>, Transformation<?>> transformations,
      Class<?> resourceClass,
      Class<?> transcodeClass,
      Options options) {
    return new EngineKey(
        model, signature, width, height, transformations, resourceClass, transcodeClass, options);
  }
}
```

生成了一个EngineKey对象。查看代码：

```java
class EngineKey implements Key {
  private final Object model;
  private final int width;
  private final int height;
  private final Class<?> resourceClass;
  private final Class<?> transcodeClass;
  private final Key signature;
  private final Map<Class<?>, Transformation<?>> transformations;
  private final Options options;
  private int hashCode;

  EngineKey(
      Object model,
      Key signature,
      int width,
      int height,
      Map<Class<?>, Transformation<?>> transformations,
      Class<?> resourceClass,
      Class<?> transcodeClass,
      Options options) {
    this.model = Preconditions.checkNotNull(model);
    this.signature = Preconditions.checkNotNull(signature, "Signature must not be null");
    this.width = width;
    this.height = height;
    this.transformations = Preconditions.checkNotNull(transformations);
    this.resourceClass =
        Preconditions.checkNotNull(resourceClass, "Resource class must not be null");
    this.transcodeClass =
        Preconditions.checkNotNull(transcodeClass, "Transcode class must not be null");
    this.options = Preconditions.checkNotNull(options);
  }
    
  @Override
  public boolean equals(Object o) {
    if (o instanceof EngineKey) {
      EngineKey other = (EngineKey) o;
      return model.equals(other.model)
          && signature.equals(other.signature)
          && height == other.height
          && width == other.width
          && transformations.equals(other.transformations)
          && resourceClass.equals(other.resourceClass)
          && transcodeClass.equals(other.transcodeClass)
          && options.equals(other.options);
    }
    return false;
  }

  @Override
  public int hashCode() {
    if (hashCode == 0) {
      hashCode = model.hashCode();
      hashCode = 31 * hashCode + signature.hashCode();
      hashCode = 31 * hashCode + width;
      hashCode = 31 * hashCode + height;
      hashCode = 31 * hashCode + transformations.hashCode();
      hashCode = 31 * hashCode + resourceClass.hashCode();
      hashCode = 31 * hashCode + transcodeClass.hashCode();
      hashCode = 31 * hashCode + options.hashCode();
    }
    return hashCode;
  }
  
    @Override
  public String toString() {
    return "...";
  }
    
  @Override
  public void updateDiskCacheKey(@NonNull MessageDigest messageDigest) {
    throw new UnsupportedOperationException();
  }
}
```

可以看到，这里传入了 model（File、Uri、远程图片 url 等）、签名、宽高（这里的宽高是指显示图片的 View 的宽高，不是图片的宽高）等参数，然后通过 EngineKeyFactory 构建了一个 EngineKey 对象（即缓存 Key），然后 EngineKey 通过重写 equals() 与 hashCode() 方法来保证缓存 Key 的唯一性。

虽然决定缓存 Key 的参数很多，但是加载图片的代码写好后这些参数都是不会变的。很多人遇到的 “服务器返回的图片变了，但是前端显示的还是以前的图片” 的问题就是这个原因，因为虽然服务器返回的图片变了，但是图片 url 还是以前那个，其他决定缓存 Key 的参数也不会变，Glide 就认为有该缓存，就会直接从缓存中获取，而不是重新下载，所以显示的还是以前的图片。

对于这个问题，有几种方法可以解决，分别如下：

（1）图片 url 不要固定 也就是说如果某个图片改变了，那么该图片的 url 也要跟着改变。

（2）使用 signature() 更改缓存 Key 我们刚刚知道了决定缓存 Key 的参数包括 signature，刚好 Glide 提供了 signature() 方法来更改该参数。具体如下：

```java
Glide.with(this).load(url).signature(new ObjectKey(timeModified)).into(imageView);
```

其中 timeModified 可以是任意数据，这里用图片的更改时间。例如图片改变了，那么服务器应该改变该字段的值，然后随图片 url 一起返回给前端，这样前端加载的时候就知道图片改变了，需要重新下载。

（3）禁用缓存 前端加载图片的时候设置禁用内存与磁盘缓存，这样每次加载都会重新下载最新的。

```java
Glide.with(this)
        .load(url)
        .skipMemoryCache(true) // 禁用内存缓存
        .diskCacheStrategy(DiskCacheStrategy.NONE) // 禁用磁盘缓存
        .into(imageView);
```

以上 3 种方法都可以解决问题，但是推荐使用第一种，这样设计是比较规范的，后台人员就应该这么设计。第二种方法也可以，但是这样无疑是给后端、前端人员都增加了麻烦。第三种是最不推荐的，相当于舍弃了缓存功能，每次都要从服务器重新下载图片，不仅浪费用户流量，而且每次加载需要等待也影响用户体验。

## 三、缓存策略

在讲 Glide 中的内存缓存与磁盘缓存之前，我们先了解下缓存策略。例如加载一张图片显示到设备上的缓存策略应该这样设计：

> 当程序第一次从网络上加载图片后，就将它缓存到设备磁盘中，下次使用这张图片的时候就不用再从网络上加载了。为了提升用户体验，往往还会在内存中缓存一份，因为从内存中加载图片比从磁盘中加载要快。程序下次加载这张图片的时候首先从内存中查找，如果没有就去磁盘中查找，都没有才从网络上加载。

这里的缓存策略涉及到缓存的添加、获取和删除操作，什么时候进行这些操作等逻辑就构成了一种缓存算法。

目前常用的一种缓存算法是 LRU（Least Recently Used），即最近最少使用算法。它的核心思想是当缓存满时，会优先淘汰那些最近最少使用的缓存对象。采用 LRU 算法的缓存有两种：LruCache 和 DiskLruCache，LruCache 用于实现内存缓存，DiskLruCache 则用于实现磁盘缓存，两者结合使用就可以实现上面的缓存策略。

LruCache 和 DiskLruCache 的内部算法原理是采用一个 LinkedHashMap 以强引用的方式存储外界的缓存对象，其提供了 get() 和 put() 方法来完成缓存的获取和添加的操作。当缓存满时，会移除较早使用的缓存对象，然后再添加新的缓存对象。可以用如下流程图表示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ebf1666274fb4cb8886d1930e786dd0f~tplv-k3u1fbpfcp-zoom-1.image)

下面要讲的 Glide 中的内存缓存与磁盘缓存也是用的 LruCache 和 DiskLruCache，只不过 LruCache 用的不是 SDK 中的，而是自己写的，但是看了原理其实是一样的。而 DiskLruCache 用的是 JakeWharton 封装的 [DiskLruCache](https://github.com/JakeWharton/DiskLruCache)。

## 四、内存缓存

Glide 默认是配置了内存缓存的，当然 Glide 也提供了 API 给我们开启和禁用，如下：

```java
// 开启内存缓存
Glide.with(this).load(url).skipMemoryCache(false).into(imageView);
// 禁用内存缓存
Glide.with(this).load(url).skipMemoryCache(true).into(imageView);
```

文章开头说了，Glide 在加载图片之前会依次检查四级缓存。现在缓存 Key 也拿到了，那么我们先看看前两级中的内存缓存是怎么获取的（下面分析的时候需要用默认加载语句或者手动开启内存缓存）。从上一篇文章知道，获取内存缓存的代码也是在 Engine 类的 load() 方法中，我们进去看看：

```java
 public <R> LoadStatus load(
      GlideContext glideContext,
      Object model,
      Key signature,
      int width,
      int height,
      Class<?> resourceClass,
      Class<R> transcodeClass,
      Priority priority,
      DiskCacheStrategy diskCacheStrategy,
      Map<Class<?>, Transformation<?>> transformations,
      boolean isTransformationRequired,
      boolean isScaleOnlyOrNoTransform,
      Options options,
      boolean isMemoryCacheable,
      boolean useUnlimitedSourceExecutorPool,
      boolean useAnimationPool,
      boolean onlyRetrieveFromCache,
      ResourceCallback cb,
      Executor callbackExecutor) {
    long startTime = VERBOSE_IS_LOGGABLE ? LogTime.getLogTime() : 0;
    // 构建缓存 key
    EngineKey key =
        keyFactory.buildKey(
            model,
            signature,
            width,
            height,
            transformations,
            resourceClass,
            transcodeClass,
            options);
    // 从内存中加载资源
    EngineResource<?> memoryResource;
    synchronized (this) {
      memoryResource = loadFromMemory(key, isMemoryCacheable, startTime);
	  ...
    }
    // 加载完成回调
    cb.onResourceReady(
        memoryResource, DataSource.MEMORY_CACHE, /* isLoadedFromAlternateCacheKey= */ false);
    return null;
  }
```

继续点击 `loadFromMemory()` 方法进去看看：

```java
  @Nullable
  private EngineResource<?> loadFromMemory(
      EngineKey key, boolean isMemoryCacheable, long startTime) {
    if (!isMemoryCacheable) {
      return null;
    }
    // 表示从 ActiveResources 中加载缓存数据。
    EngineResource<?> active = loadFromActiveResources(key);
    if (active != null) {
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from active resources", startTime, key);
      }
      return active;
    }

    // 表示从内存缓存中加载缓存数据
    EngineResource<?> cached = loadFromCache(key);
    if (cached != null) {
      if (VERBOSE_IS_LOGGABLE) {
        logWithTimeAndKey("Loaded resource from cache", startTime, key);
      }
      return cached;
    }

    return null;
  }
```

就是 Glide 四级缓存中的前两级。ActiveResources 里面主要包含了一个 HashMap 的相关操作，然后 HashMap 中保存的值又是弱引用来引用的，也就是说这里是采用一个**弱引用的 HashMap** 来缓存活动资源。下面我们分析下这两个关注点：

### loadFromActiveResources()

```java
  private final ActiveResources activeResources;
  @Nullable
  private EngineResource<?> loadFromActiveResources(Key key) {
    EngineResource<?> active = activeResources.get(key);
    if (active != null) {
      active.acquire();
    }

    return active;
  }
```

继续查看`ActiveResources#get()`方法：

```java
  final Map<Key, ResourceWeakReference> activeEngineResources = new HashMap<>();  
  @Nullable
  synchronized EngineResource<?> get(Key key) {
    // 从 HashMap 中获取 ResourceWeakReference
    ResourceWeakReference activeRef = activeEngineResources.get(key);
    if (activeRef == null) {
      return null;
    }
    // 从弱引用中获取活动资源
    EngineResource<?> active = activeRef.get();
    if (active == null) {
      cleanupActiveReference(activeRef);
    }
    return active;
  }
```

可以看到，这里首先从 HashMap 中获取 ResourceWeakReference（继承了弱引用），然后从弱引用中获取了活动资源`（获取活动资源）`，即正在使用的图片。也就是说正在使用的图片实际是通过弱引用维护，然后保存在 HashMap 中的。

查看`EngineResource#acquire()`方法：

```java
  synchronized void acquire() {
    if (isRecycled) {
      throw new IllegalStateException("Cannot acquire a recycled resource");
    }
    ++acquired;
  }
```

发现这里是将 acquired 变量 +1，这个变量用来记录图片被引用的次数。该变量除了 acquire() 方法中做了 +1 操作，还在 release() 方法中做了 -1 的操作，如下：

```java
  /*EngineResource*/
  void release() {
    boolean release = false;
    synchronized (this) {
      if (acquired <= 0) {
        throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
      }
      if (--acquired == 0) {
        release = true;
      }
    }
    if (release) {
      listener.onResourceReleased(key, this);
    }
  }
  
  /*Engine*/
  @Override
  public void onResourceReleased(Key cacheKey, EngineResource<?> resource) {
    activeResources.deactivate(cacheKey);
    if (resource.isMemoryCacheable()) {
      cache.put(cacheKey, resource);
    } else {
      resourceRecycler.recycle(resource, /*forceNextFrame=*/ false);
    }
  }
  
  /*ActiveResources*/
  synchronized void deactivate(Key key) {
    ResourceWeakReference removed = activeEngineResources.remove(key);
    if (removed != null) {
      removed.reset();
    }
  }
```

可以看到，当 acquired 减到 0 的时候，又回调了 Engine#onResourceReleased()。在 onResourceReleased() 方法中首先将活动资源从弱引用的 HashMap 中移除`（清理活动资源）`，然后将它缓存到内存缓存中`（存储内存缓存）`。

也就是说，release() 方法主要是释放资源。当我们从一屏滑动到下一屏的时候，上一屏的图片就会看不到，这个时候就会调用该方法。还有我们关闭当前显示图片的页面时会调用 onDestroy() 方法，最终也会调用该方法。这两种情况很明显是不需要用到该图片了，那么理所当然的会调用 release() 方法来释放弱引用的 HashMap 中缓存的活动资源。

这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用 LruCache 来进行缓存的功能。

### loadFromCache()

```java
  private EngineResource<?> loadFromCache(Key key) {
    //这里是获取内存缓存
    EngineResource<?> cached = getEngineResourceFromCache(key);
    if (cached != null) {
      //将 acquired 变量 +1，然后用来记录图片被引用的次数。
      cached.acquire();
      //将内存中获取的缓存数据缓存到弱引用的 HashMap 中。
      activeResources.activate(key, cached);
    }
    return cached;
  }
```

先查看`getEngineResourceFromCache()`方法:

```java
  private EngineResource<?> getEngineResourceFromCache(Key key) {
    Resource<?> cached = cache.remove(key);

    final EngineResource<?> result;
    if (cached == null) {
      result = null;
    } else if (cached instanceof EngineResource) {
      // Save an object allocation if we've cached an EngineResource (the typical case).
      result = (EngineResource<?>) cached;
    } else {
      result =
          new EngineResource<>(
              cached, /*isMemoryCacheable=*/ true, /*isRecyclable=*/ true, key, /*listener=*/ this);
    }
    return result;
  }
```

可以看到，这里的 cache 就是 LruResourceCache，remove() 操作就是移除缓存的同时获取该缓存`（获取内存缓存）`。LruResourceCache 继承了 LruCache，虽然不是 SDK 中的 LruCache，但是看了原理其实是一样的，也就是说内存缓存使用的是 LRU 算法实现的。

`getEngineResourceFromCache()`方法之后主要做了获取活动资源、清理活动资源、获取内存缓存、存储内存缓存。其中清理内存缓存的操作 LRU 算法已经自动帮我们实现了，那是不是发现少了存储活动资源的步骤？

活动资源是哪里来的呢？其实就是我们从网络请求中返回的数据。从上一篇文章（可以回去搜索 onEngineJobComplete() 回忆下上下文）可以知道网络请求回来后先进行解码，然后在 Engine#onEngineJobComplete() 方法中进行了活动资源的存储，我再贴下代码：

```java
  /*Engine*/
  @Override
  public synchronized void onEngineJobComplete(
      EngineJob<?> engineJob, Key key, EngineResource<?> resource) {
    // A null resource indicates that the load failed, usually due to an exception.
    if (resource != null && resource.isMemoryCacheable()) {
      activeResources.activate(key, resource);
    }

    jobs.removeIfCurrent(key, engineJob);
  }

  /*ActiveResources*/
  synchronized void activate(Key key, EngineResource<?> resource) {
    ResourceWeakReference toPut =
        new ResourceWeakReference(
            key, resource, resourceReferenceQueue, isActiveResourceRetentionAllowed);

	// 存储活动资源
    ResourceWeakReference removed = activeEngineResources.put(key, toPut);
    if (removed != null) {
      removed.reset();
    }
  }
复制代码
```

以上就是 Glide 内存缓存的原理，但是我们发现除了利用 LruCache 实现的内存缓存，还有一个是利用弱引用的 HashMap 实现的。一般如果让我们设计，可能就只会想到用 LruCache 实现内存缓存。

使用activeResources来缓存正在使用中的图片，可以保护这些图片不会被LruCache算法回收掉。

## 五、磁盘缓存

### 5.1 磁盘缓存策略

前面说了禁用缓存只需要如下设置即可：

```java
Glide.with(this).load(url).diskCacheStrategy(DiskCacheStrategy.NONE).into(imageView);
```

上面的 DiskCacheStrategy 封装的是磁盘缓存策略，一共有如下几种策略：

1. ALL：既缓存原始图片，也缓存转换过后的图片。
2. NONE：不缓存任何内容。
3. DATA：只缓存原始图片。
4. RESOURCE：只缓存转换过后的图片。
5. AUTOMATIC：默认策略，它会尝试对本地和远程图片使用最佳的策略。如果是远程图片，则只缓存原始图片；如果是本地图片，那么只缓存转换过后的图片。

其实 5 种策略总结起来对应的就是文章开头说的后两级缓存，即资源类型（Resource）与 数据来源（Data），下面通过源码来分析下它们是在哪里获取、存储和清理缓存的。

### 5.2 资源类型（Resource）

该级缓存只缓存转换过后的图片，那么我们需要先配置如下策略：

```java
Glide.with(this).load(url).diskCacheStrategy(DiskCacheStrategy.RESOURCE).into(imageView);
```

通过上一篇文章可知，当我们从主线程切换到子线程去执行请求的时候用到了磁盘缓存策略，那么我们这里直接从 `DecodeJob#run()` 方法开始分析：

```java
  /*DecodeJob*/
  @Override
  public void run() {

    ...
    try {
	  // 执行
      runWrapped();
    } catch (CallbackException e) {
      throw e;
    }
	...

  }
```

继续 `runWrapped()` 方法：

```java
  /*DecodeJob*/
  private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        // runReason 的默认值为 INITIALIZE，走的是第一个 case。
        // 若配置了禁用内存与磁盘缓存，getNextStage得到的stage为 SOURCE
        // 获取资源状态
        stage = getNextStage(Stage.INITIALIZE);
        // 根据资源状态获取资源执行器
        currentGenerator = getNextGenerator();
        // 执行
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized run reason: " + runReason);
    }
  }

  /*DecodeJob*/
  private Stage getNextStage(Stage current) {
    switch (current) {
      case INITIALIZE:
        return diskCacheStrategy.decodeCachedResource()
            ? Stage.RESOURCE_CACHE
            : getNextStage(Stage.RESOURCE_CACHE);
      case RESOURCE_CACHE:
        return diskCacheStrategy.decodeCachedData()
            ? Stage.DATA_CACHE
            : getNextStage(Stage.DATA_CACHE);
      case DATA_CACHE:
        // Skip loading from source if the user opted to only retrieve the resource from cache.
        return onlyRetrieveFromCache ? Stage.FINISHED : Stage.SOURCE;
      case SOURCE:
      case FINISHED:
        return Stage.FINISHED;
      default:
        throw new IllegalArgumentException("Unrecognized stage: " + current);
    }
  }

  /*DecodeJob*/
  private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized stage: " + stage);
    }
  }
```

这里会根据缓存策略获取到资源状态，然后再根据资源状态获取资源执行器，最后调用 `runGenerators()` 方法，我们查看该方法：

```java
  /*DecodeJob*/
  private void runGenerators() {
    currentThread = Thread.currentThread();
    startFetchTime = LogTime.getLogTime();
    boolean isStarted = false;
    // 若 currentGenerator 在这里为 SourceGenerator
    // 调用 startNext() 方法的时候实际调用的是 SourceGenerator#startNext()
    // 该方法执行完返回的结果为 true，所以 while 循环是进不去的
    while (!isCancelled
        && currentGenerator != null
        && !(isStarted = currentGenerator.startNext())) {
      stage = getNextStage(stage);
      currentGenerator = getNextGenerator();

      if (stage == Stage.SOURCE) {
        reschedule();
        return;
      }
    }
  }
```

可以看到，该方法中会调用当前执行器的 `startNext()` 方法，因为我们配置的缓存策略是 RESOURCE，所以这里直接看 `ResourceCacheGenerator#startNext()`方法：

```java
  /*ResourceCacheGenerator*/
  @Override
  public boolean startNext() {

    ...

    while (modelLoaders == null || !hasNextModelLoader()) {

      ...
		
	  // 首先构建缓存 Key
      currentKey =
          new ResourceCacheKey(
              helper.getArrayPool(),
              sourceId,
              helper.getSignature(),
              helper.getWidth(),
              helper.getHeight(),
              transformation,
              resourceClass,
              helper.getOptions());
	  // 根据缓存 Key 去获取缓存文件
      cacheFile = helper.getDiskCache().get(currentKey);
      if (cacheFile != null) {
        sourceKey = sourceId;
        modelLoaders = helper.getModelLoaders(cacheFile);
        modelLoaderIndex = 0;
      }
    }

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      ModelLoader<File, ?> modelLoader = modelLoaders.get(modelLoaderIndex++);
      loadData =
          modelLoader.buildLoadData(
              cacheFile, helper.getWidth(), helper.getHeight(), helper.getOptions());
      if (loadData != null && helper.hasLoadPath(loadData.fetcher.getDataClass())) {
        started = true;
		//最后将缓存文件加载成需要的数据
        loadData.fetcher.loadData(helper.getPriority(), this);
      }
    }

    return started;
  }
```

可以看到，根据我标记的关注点这里首先构建缓存 Key，然后根据缓存 Key 去获取缓存文件`（获取转换后的图片）`，最后将缓存文件加载成需要的数据。

其中 `helper.getDiskCache()` 为 DiskLruCacheWrapper，内部是通过 DiskLruCache 操作的，也就是说这一级的磁盘缓存使用的是 LRU 算法实现的。

因为获取的是缓存文件，所以这里的 `loadData.fetcher` 实际为 ByteBufferFileLoader，继续看 `ByteBufferFileLoader#loadData()`:

```java
    /*ByteBufferFileLoader*/
    @Override
    public void loadData(
        @NonNull Priority priority, @NonNull DataCallback<? super ByteBuffer> callback) {
      ByteBuffer result;
      try {
        //将缓存文件转换成 ByteBuffer
        result = ByteBufferUtil.fromFile(file);
        //回调返回结果
        callback.onDataReady(result);
      } catch (IOException e) {
        if (Log.isLoggable(TAG, Log.DEBUG)) {
          Log.d(TAG, "Failed to obtain ByteBuffer for file", e);
        }
        callback.onLoadFailed(e);
        return;
      }

      callback.onDataReady(result);
    }
```

这里主要是将缓存文件转换成 ByteBuffer，然后通过 `onDataReady()` 方法回调出去，最终回调到 DecodeJob 的 `onDataFetcherReady()` 方法中，后面的流程就跟上一篇文章差不多了。



关于存储缓存的流程，刚刚获取缓存 Key 的时候用的是 ResourceCacheKey，那么存储缓存与获取缓存肯定都是用的 ResourceCacheKey，经过查找发现除了 ResourceCacheGenerator，只有在 DecodeJob 的 onResourceDecoded() 方法中使用到：

```java
  /*DecodeJob*/
  <Z> Resource<Z> onResourceDecoded(DataSource dataSource, @NonNull Resource<Z> decoded) {
    
	...
	// 缓存相关
    boolean isFromAlternateCacheKey = !decodeHelper.isSourceKey(currentSourceKey);
    if (diskCacheStrategy.isResourceCacheable(
        isFromAlternateCacheKey, dataSource, encodeStrategy)) {
      if (encoder == null) {
        throw new Registry.NoResultEncoderAvailableException(transformed.get().getClass());
      }
      final Key key;
	  // 首先根据缓存策略构建不同的缓存 Key
      switch (encodeStrategy) {
        case SOURCE:
          key = new DataCacheKey(currentSourceKey, signature);
          break;
        case TRANSFORMED:
          key =
              new ResourceCacheKey(
                  decodeHelper.getArrayPool(),
                  currentSourceKey,
                  signature,
                  width,
                  height,
                  appliedTransformation,
                  resourceSubClass,
                  options);
          break;
        default:
          throw new IllegalArgumentException("Unknown strategy: " + encodeStrategy);
      }

      LockedResource<Z> lockedResult = LockedResource.obtain(transformed);
	  // 初始化，给变量 key 赋值
      deferredEncodeManager.init(key, encoder, lockedResult);
      result = lockedResult;
    }
    return result;
  }
```

内部又调用了 init() 方法：

```java
  private static class DeferredEncodeManager<Z> {
    private Key key;
    private ResourceEncoder<Z> encoder;
    private LockedResource<Z> toEncode;

    <X> void init(Key key, ResourceEncoder<X> encoder, LockedResource<X> toEncode) {
      this.key = key;
      this.encoder = (ResourceEncoder<Z>) encoder;
      this.toEncode = (LockedResource<Z>) toEncode;
    }

    void encode(DiskCacheProvider diskCacheProvider, Options options) {
      GlideTrace.beginSection("DecodeJob.encode");
      try {
		//存储缓存的操作
        diskCacheProvider
            .getDiskCache()
            .put(key, new DataCacheWriter<>(encoder, toEncode, options));
      } finally {
        toEncode.unlock();
        GlideTrace.endSection();
      }
    }
}
```

可以看到，根据我标记的关注点这里首先根据缓存策略构建不同的缓存 Key，然后调用 DeferredEncodeManager 的 init() 方法给变量 key 赋值，然后 key 又在 encode() 方法中使用了，该方法中就做了存储缓存的操作`（存储转换后的图片）`。

查看 `encode()` 方法，发现在 `DecodeJob#notifyEncodeAndRelease()` 方法中被调用了：

```java
  /*DecodeJob */
  private void notifyEncodeAndRelease(Resource<R> resource, DataSource dataSource) {
    if (resource instanceof Initializable) {
      ((Initializable) resource).initialize();
    }

    Resource<R> result = resource;
    LockedResource<R> lockedResource = null;
    if (deferredEncodeManager.hasResourceToEncode()) {
      lockedResource = LockedResource.obtain(resource);
      result = lockedResource;
    }

    notifyComplete(result, dataSource);

    stage = Stage.ENCODE;
    try {
      if (deferredEncodeManager.hasResourceToEncode()) {
		// 将资源缓存到磁盘
        deferredEncodeManager.encode(diskCacheProvider, options);
      }
    } finally {
      if (lockedResource != null) {
        lockedResource.unlock();
      }
    }
     // 完成，释放各种资源  
    onEncodeComplete();
  }
```

`notifyEncodeAndRelease()` 方法是我们上一篇文章中讲的解码完成了通知下去的步骤，也就是说第一次加载的时候在 `SourceGenerator#startNext()` 中请求到数据，然后解码完成，最后再存储缓存。

上面已经实现了转换后的图片的获取、存储，剩下的清理操作 LRU 算法已经自动帮我们实现了。

接下来继续看下原始图片是怎么获取、存储与清理的。

### 5.3 数据来源（Data）

该级缓存只缓存原始图片，那么我们需要先配置如下策略：

```java
Glide.with(this).load(url).diskCacheStrategy(DiskCacheStrategy.DATA).into(imageView);
```

与资源类型一样，只不过这里缓存策略换成了 DATA，所以前面就不讲了，我们直接看 `DataCacheGenerator#startNext()` 方法：

```java
  @Override
  public boolean startNext() {
    while (modelLoaders == null || !hasNextModelLoader()) {
      sourceIdIndex++;
      if (sourceIdIndex >= cacheKeys.size()) {
        return false;
      }

      Key sourceId = cacheKeys.get(sourceIdIndex);

      // 首先构建缓存 Key
      Key originalKey = new DataCacheKey(sourceId, helper.getSignature());
      // 根据缓存 Key 去获取缓存文件
      cacheFile = helper.getDiskCache().get(originalKey);
      if (cacheFile != null) {
        this.sourceKey = sourceId;
        modelLoaders = helper.getModelLoaders(cacheFile);
        modelLoaderIndex = 0;
      }
    }

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      ModelLoader<File, ?> modelLoader = modelLoaders.get(modelLoaderIndex++);
      loadData =
          modelLoader.buildLoadData(
              cacheFile, helper.getWidth(), helper.getHeight(), helper.getOptions());
      if (loadData != null && helper.hasLoadPath(loadData.fetcher.getDataClass())) {
        started = true;
        //将缓存文件加载成需要的数据
        loadData.fetcher.loadData(helper.getPriority(), this);
      }
    }
    return started;
  }
```

可以看到，这里首先构建缓存 Key，然后根据缓存 Key 去获取缓存文件`（获取原始图片）`，最后将缓存文件加载成需要的数据。

与资源类型一样，这里的 `helper.getDiskCache()` 也为 DiskLruCacheWrapper，所以这一级的磁盘缓存使用的也是 LRU 算法实现的。

这里获取的同样是缓存文件，所以这里的 `loadData.fetcher` 也为 ByteBufferFileLoader，最终也是回调到 DecodeJob 的 `onDataFetcherReady()` 方法中。

关于存储缓存，用反推的方法，但是发现除了 DataCacheGenerator 还有两个地方用到了。
 第一个与资源类型一样是在 DecodeJob#onResourceDecoded()：

```java
  /*DecodeJob*/
  <Z> Resource<Z> onResourceDecoded(DataSource dataSource, @NonNull Resource<Z> decoded) {
    
	...

    boolean isFromAlternateCacheKey = !decodeHelper.isSourceKey(currentSourceKey);
	// 缓存策略的判断
    if (diskCacheStrategy.isResourceCacheable(
        isFromAlternateCacheKey, dataSource, encodeStrategy)) {
      if (encoder == null) {
        throw new Registry.NoResultEncoderAvailableException(transformed.get().getClass());
      }
      final Key key;
      switch (encodeStrategy) {
        case SOURCE:
          key = new DataCacheKey(currentSourceKey, signature);
          break;
        case TRANSFORMED:
          key =
              new ResourceCacheKey(
                  decodeHelper.getArrayPool(),
                  currentSourceKey,
                  signature,
                  width,
                  height,
                  appliedTransformation,
                  resourceSubClass,
                  options);
          break;
        default:
          throw new IllegalArgumentException("Unknown strategy: " + encodeStrategy);
      }

      LockedResource<Z> lockedResult = LockedResource.obtain(transformed);
      deferredEncodeManager.init(key, encoder, lockedResult);
      result = lockedResult;
    }
    return result;
  }
```

这里的关注点（1）做了一个缓存策略的判断，因为前面配置的缓存策略是 DATA，所以这里调用的是 DATA 中的 `isResourceCacheable()` 方法：

```java
  /*DiskCacheStrategy*/
  public static final DiskCacheStrategy DATA =
      new DiskCacheStrategy() {
        @Override
        public boolean isDataCacheable(DataSource dataSource) {
          return dataSource != DataSource.DATA_DISK_CACHE && dataSource != DataSource.MEMORY_CACHE;
        }

        // 调用的是该方法
        @Override
        public boolean isResourceCacheable(
            boolean isFromAlternateCacheKey, DataSource dataSource, EncodeStrategy encodeStrategy) {
          return false;
        }

        @Override
        public boolean decodeCachedResource() {
          return false;
        }

        @Override
        public boolean decodeCachedData() {
          return true;
        }
      };
```

可以看到，`isResourceCacheable()` 方法始终返回 false，所以上面是进不去的，可以排除。

那我们继续看下另一个地方用到的：

```java
  /*SourceGenerator*/
  private void cacheData(Object dataToCache) {
    long startTime = LogTime.getLogTime();
    try {
      Encoder<Object> encoder = helper.getSourceEncoder(dataToCache);
      DataCacheWriter<Object> writer =
          new DataCacheWriter<>(encoder, dataToCache, helper.getOptions());
	  // 首先构建缓存 Key
      originalKey = new DataCacheKey(loadData.sourceKey, helper.getSignature());
	  // 存储缓存
      helper.getDiskCache().put(originalKey, writer);

	  ...

    } finally {
      loadData.fetcher.cleanup();
    }

    sourceCacheGenerator =
        new DataCacheGenerator(Collections.singletonList(loadData.sourceKey), helper, this);
  }
```

这里首先构建缓存 Key，然后存储缓存`（存储原始图片）`。而该方法是在 `SourceGenerator#startNext()` 中调用的：

```java
  /*SourceGenerator*/
  @Override
  public boolean startNext() {
	// 首先判断缓存不为空才进行缓存数据的操作
    if (dataToCache != null) {
      Object data = dataToCache;
      dataToCache = null;
      cacheData(data);
    }

    if (sourceCacheGenerator != null && sourceCacheGenerator.startNext()) {
      return true;
    }
    sourceCacheGenerator = null;

    loadData = null;
    boolean started = false;
    while (!started && hasNextModelLoader()) {
      loadData = helper.getLoadData().get(loadDataListIndex++);
      if (loadData != null
          && (helper.getDiskCacheStrategy().isDataCacheable(loadData.fetcher.getDataSource())
              || helper.hasLoadPath(loadData.fetcher.getDataClass()))) {
        started = true;
        startNextLoad(loadData);
      }
    }
    return started;
  }
```

可以看到，首先判断缓存不为空才进行缓存数据的操作，那我们看下 dataToCache 是哪里被赋值了呗，查找发现只有在 `SourceGenerator#onDataReadyInternal()` 中赋值过：

```java
  /*SourceGenerator*/
  void onDataReadyInternal(LoadData<?> loadData, Object data) {
    DiskCacheStrategy diskCacheStrategy = helper.getDiskCacheStrategy();
    if (data != null && diskCacheStrategy.isDataCacheable(loadData.fetcher.getDataSource())) {
	  // 赋值
      dataToCache = data;
      // We might be being called back on someone else's thread. Before doing anything, we should
      // reschedule to get back onto Glide's thread.
	  // 回调
      cb.reschedule();
    } else {
      cb.onDataFetcherReady(
          loadData.sourceKey,
          data,
          loadData.fetcher,
          loadData.fetcher.getDataSource(),
          originalKey);
    }
  }
```

可以看到，`onDataReadyInternal()` 方法又是我们熟悉的，也就是上一篇文章中加载完数据后调用的。

上一篇文章是因为禁用了缓存，所以走的是 else。这里配置的缓存策略是 DATA，所以自然走的是 if。

那么赋值完成，下一步肯定要用到，我们继续跟这里的回调方法，发现调用的是 `EngineJob#reschedule()` 方法：

```java
  /*EngineJob*/
  @Override
  public void reschedule(DecodeJob<?> job) {
    getActiveSourceExecutor().execute(job);
  }
```

这里又用线程池执行了 DecodeJob，所以最后又回到了 `SourceGenerator#startNext()` 方法，这时候 dataToCache 就不是空了，所以就将数据缓存起来了。其实 cacheData() 方法中存储缓存的时候还构建了一个 DataCacheGenerator，然后存储完成又执行了 `DataCacheGenerator#startNext()`，这里再从磁盘获取缓存后才将图片显示到控件上，也就是说网络请求拿到数据后是先缓存数据，然后再从磁盘获取缓存才显示到控件上。

同理，原始图片的清理操作也是 LRU 算法自动帮我们实现了。



