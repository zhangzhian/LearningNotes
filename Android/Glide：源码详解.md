# Glide: 源码详解 | 通过源码深入理解Glide

> 本文聚焦于Glide的源码，基于Glide4.11.0

## 一、简介

[Glide的GitHub](https://muyangmin.github.io/glide-docs-cn/)

Glide是一个快速高效的Android图片加载库，注重于平滑的滚动。Glide提供了易用的API，高性能、可扩展的图片解码管道（decode pipeline），以及自动的资源池技术。

### 1. 简单使用

1、添加依赖：

```groovy
repositories {
  google()
  jcenter()
}

dependencies {
  implementation 'com.github.bumptech.glide:glide:4.11.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.11.0'
}
```

2、添加网络权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

3、一句代码加载图片到ImageView

```java
Glide.with(fragment).load(url).into(imageView);
```

进阶一点的用法，参数设置

```java
RequestOptions options = new RequestOptions()
            .placeholder(R.drawable.ic_launcher_background)
            .error(R.mipmap.ic_launcher)
            .diskCacheStrategy(DiskCacheStrategy.NONE)
    		.override(200, 100);
    
Glide.with(this).load(url).apply(options).into(imageView);
```

### 2. 对比

**Glide：**

- 多种图片格式的缓存，适用于更多的内容表现形式（如Gif、WebP、缩略图、Video）
- 生命周期集成（根据Activity或者Fragment的生命周期管理图片加载请求）
- 高效处理Bitmap（bitmap的复用和主动回收，减少系统回收压力）
- 高效的缓存策略，灵活（Picasso只会缓存原始尺寸的图片，Glide缓存的是多种规格），加载速度快且内存开销小（默认Bitmap格式的不同，使得内存开销是Picasso的一半）

**Fresco：**

- 最大的优势在于5.0以下(最低2.3)的bitmap加载。在5.0以下系统，Fresco将图片放到一个特别的内存区域(Ashmem区)
- 大大减少OOM（在更底层的Native层对OOM进行处理，图片将不再占用App的内存）
- 适用于需要高性能加载大量图片的场景

### 3. 进阶使用

关于Glide的详细使用请查看这个官方 [中文文档](https://muyangmin.github.io/glide-docs-cn/)，这里就不赘述了。

## 二、核心流程

Glide 默认是配置了内存与磁盘缓存的，所以这里我们先禁用内存和磁盘缓存，来分析核心流程。代码如下：

```java
Glide.with(this)
        .load(url)
        .skipMemoryCache(true) // 禁用内存缓存
        .diskCacheStrategy(DiskCacheStrategy.NONE) // 禁用磁盘缓存
        .into(imageView);
```

缓存机制下一节分析。

### 1. with()

with()有6个重载方法，如下

```java
Glide#with(Context context)
Glide#with(Activity activity)
Glide#with(FragmentActivity activity)
Glide#with(Fragment fragment)
Glide#with(android.app.Fragment fragment)
Glide#with(View view)
```

这些重载方法可以分成两种情况，即 Application（Context）类型与非 Application（Activity、Fragment、View#getContext() 或 Fragment）类型，这些参数的作用是确定图片加载的生命周期。

最终都会调用 `getRetriever().get()`方法，我们分为两部分看。

#### 1.1 Gilde#getRetriever()

getRetriever() 定义如下：

```java
  @NonNull
  private static RequestManagerRetriever getRetriever(@Nullable Context context) {
    ...
    return Glide.get(context).getRequestManagerRetriever();
  }
```

继续看下 Glide#get(context)：

```java
  public static Glide get(@NonNull Context context) {
    //使用了双重校验锁的单例模式来获取 Glide 的实例
    if (glide == null) {
      //该方法用来实例化我们用 @GlideModule 注解标识的自定义模块
      GeneratedAppGlideModule annotationGeneratedModule =
          getAnnotationGeneratedGlideModules(context.getApplicationContext());
      synchronized (Glide.class) {
        if (glide == null) {
          //检查和初始化
          checkAndInitializeGlide(context, annotationGeneratedModule);
        }
      }
    }

    return glide;
  }
```

这里使用了DCL单例模式来获取 Glide 的实例，先看getAnnotationGeneratedGlideModules方法。

```java
  private static GeneratedAppGlideModule getAnnotationGeneratedGlideModules(Context context) {
    GeneratedAppGlideModule result = null;
    try {
      Class<GeneratedAppGlideModule> clazz =
          (Class<GeneratedAppGlideModule>)
              Class.forName("com.bumptech.glide.GeneratedAppGlideModuleImpl");
      result =
          clazz.getDeclaredConstructor(Context.class).newInstance(context.getApplicationContext());
    } catch (ClassNotFoundException e) {
      ...
      // These exceptions can't be squashed across all versions of Android.
    } catch (InstantiationException e) {
      throwIncorrectGlideModule(e);
    } catch (IllegalAccessException e) {
      throwIncorrectGlideModule(e);
    } catch (NoSuchMethodException e) {
      throwIncorrectGlideModule(e);
    } catch (InvocationTargetException e) {
      throwIncorrectGlideModule(e);
    }
    return result;
  }
```

该方法用来实例化我们用 @GlideModule 注解标识的自定义模块，自定义模块的用法请参考官方 [中文文档](https://muyangmin.github.io/glide-docs-cn/)。

我们在来看checkAndInitializeGlide方法，从名称上看是用来检查和初始化的，如下：

```java
  @GuardedBy("Glide.class")
  private static void checkAndInitializeGlide(
      @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
    // 不能重复初始化
    if (isInitializing) {
      throw new IllegalStateException(
          "You cannot call Glide.get() in registerComponents(),"
              + " use the provided Glide instance instead");
    }
    isInitializing = true;
    //初始化
    initializeGlide(context, generatedAppGlideModule);
    isInitializing = false;
  }
```

关注initializeGlide方法，如下：

```java
  @GuardedBy("Glide.class")
  private static void initializeGlide(
      @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
    initializeGlide(context, new GlideBuilder(), generatedAppGlideModule);
  }

  @GuardedBy("Glide.class")
  @SuppressWarnings("deprecation")
  private static void initializeGlide(
      @NonNull Context context,
      @NonNull GlideBuilder builder,
      @Nullable GeneratedAppGlideModule annotationGeneratedModule) {
    Context applicationContext = context.getApplicationContext();
    List<com.bumptech.glide.module.GlideModule> manifestModules = Collections.emptyList();
    if (annotationGeneratedModule == null || annotationGeneratedModule.isManifestParsingEnabled()) {
      //将 AndroidManifest.xml 中所有值为 GlideModule 的 meta-data 配置读取出来，并将相应的自定义模块实例化
      manifestModules = new ManifestParser(applicationContext).parse();
    }

    ...

    // 从注解生成的 GeneratedAppGlideModule 中获取 RequestManagerFactory
    RequestManagerRetriever.RequestManagerFactory factory =
        annotationGeneratedModule != null
            ? annotationGeneratedModule.getRequestManagerFactory()
            : null;
    // 将 RequestManagerFactory 设置到 GlideBuilder
    builder.setRequestManagerFactory(factory);
    // 3.x版本的自定义模块中更改 Glide 的配置
    for (com.bumptech.glide.module.GlideModule module : manifestModules) {
      module.applyOptions(applicationContext, builder);
    }
    // 4.x版本的自定义模块中更改 Glide 的配置
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.applyOptions(applicationContext, builder);
    }
    //真正初始化的地方，使用建造者模式创建 Glide
    Glide glide = builder.build(applicationContext);
    for (com.bumptech.glide.module.GlideModule module : manifestModules) {
      try {
        //3.x自定义模块中替换 Glide 组件
        module.registerComponents(applicationContext, glide, glide.registry);
      } catch (AbstractMethodError e) {
        throw new IllegalStateException(
            "Attempting to register a Glide v3 module. If you see this, you or one of your"
                + " dependencies may be including Glide v3 even though you're using Glide v4."
                + " You'll need to find and remove (or update) the offending dependency."
                + " The v3 module name is: "
                + module.getClass().getName(),
            e);
      }
    }
    //4.x自定义模块中替换 Glide 组件
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.registerComponents(applicationContext, glide, glide.registry);
    }
    applicationContext.registerComponentCallbacks(glide);
    //将创建的 Glide 赋值给 Glide 的静态变量
    Glide.glide = glide;
  }

```

initializeGlide方法调用了重载方法，主要是完成自定义模块的相关功能和真正进行初始化。

我们需要重点关注的是` Glide glide = builder.build(applicationContext);`

实现在GlideBuilder#build如下：

```java
 @NonNull
  Glide build(@NonNull Context context) {
    if (sourceExecutor == null) {
      // 创建网络请求线程池
      sourceExecutor = GlideExecutor.newSourceExecutor();
    }

    if (diskCacheExecutor == null) {
      // 创建磁盘缓存线程池
      diskCacheExecutor = GlideExecutor.newDiskCacheExecutor();
    }

    if (animationExecutor == null) {
      // 创建动画线程池
      animationExecutor = GlideExecutor.newAnimationExecutor();
    }

    if (memorySizeCalculator == null) {
      // 创建内存大小计算器
      memorySizeCalculator = new MemorySizeCalculator.Builder(context).build();
    }

    if (connectivityMonitorFactory == null) {
      // 创建默认网络连接监视器工厂
      connectivityMonitorFactory = new DefaultConnectivityMonitorFactory();
    }

    // 创建 Bitmap 池
    if (bitmapPool == null) {
      int size = memorySizeCalculator.getBitmapPoolSize();
      if (size > 0) {
        bitmapPool = new LruBitmapPool(size);
      } else {
        bitmapPool = new BitmapPoolAdapter();
      }
    }

    if (arrayPool == null) {
      // 创建固定大小的数组池（4MB），使用 LRU 策略来保持数组池在最大字节数以下
      arrayPool = new LruArrayPool(memorySizeCalculator.getArrayPoolSizeInBytes());
    }

    if (memoryCache == null) {
      // 创建内存缓存
      memoryCache = new LruResourceCache(memorySizeCalculator.getMemoryCacheSize());
    }

    if (diskCacheFactory == null) {
      // 创建磁盘缓存工厂
      diskCacheFactory = new InternalCacheDiskCacheFactory(context);
    }

    // 创建加载以及管理活动资源和缓存资源的引擎
    if (engine == null) {
      engine =
          new Engine(
              memoryCache,
              diskCacheFactory,
              diskCacheExecutor,
              sourceExecutor,
              GlideExecutor.newUnlimitedSourceExecutor(),
              animationExecutor,
              isActiveResourceRetentionAllowed);
    }

    if (defaultRequestListeners == null) {
      defaultRequestListeners = Collections.emptyList();
    } else {
      defaultRequestListeners = Collections.unmodifiableList(defaultRequestListeners);
    }

    // 创建请求管理类，这里的 requestManagerFactory 就是前面 GlideBuilder#setRequestManagerFactory() 设置进来的
    // 也就是 @GlideModule 注解中获取的
    GlideExperiments experiments = glideExperimentsBuilder.build();
    RequestManagerRetriever requestManagerRetriever =
        new RequestManagerRetriever(requestManagerFactory, experiments);

    // 创建 Glide
    return new Glide(
        context,
        engine,
        memoryCache,
        bitmapPool,
        arrayPool,
        requestManagerRetriever,
        connectivityMonitorFactory,
        logLevel,
        defaultRequestOptionsFactory,
        defaultTransitionOptions,
        defaultRequestListeners,
        experiments);
  }
```

build() 方法主要是创建一些线程池、Bitmap 池、缓存策略、Engine 等，然后利用这些来创建具体的 Glide。继续看下 Glide 的构造函数：

```java
  Glide(
      @NonNull Context context,
      @NonNull Engine engine,
      @NonNull MemoryCache memoryCache,
      @NonNull BitmapPool bitmapPool,
      @NonNull ArrayPool arrayPool,
      @NonNull RequestManagerRetriever requestManagerRetriever,
      @NonNull ConnectivityMonitorFactory connectivityMonitorFactory,
      int logLevel,
      @NonNull RequestOptionsFactory defaultRequestOptionsFactory,
      @NonNull Map<Class<?>, TransitionOptions<?, ?>> defaultTransitionOptions,
      @NonNull List<RequestListener<Object>> defaultRequestListeners,
      GlideExperiments experiments) {
    // 将传进来的参数赋值给 Glide 类中的一些常量，方便后续使用
    this.engine = engine;
    this.bitmapPool = bitmapPool;
    this.arrayPool = arrayPool;
    this.memoryCache = memoryCache;
    this.requestManagerRetriever = requestManagerRetriever;
    this.connectivityMonitorFactory = connectivityMonitorFactory;
    this.defaultRequestOptionsFactory = defaultRequestOptionsFactory;

    final Resources resources = context.getResources();

    // 创建 Registry，Registry 的作用是管理组件注册，用来扩展或替换 Glide 的默认加载、解码和编码逻辑
    registry = new Registry();
    registry.register(new DefaultImageHeaderParser());
    // Right now we're only using this parser for HEIF images, which are only supported on OMR1+.
    // If we need this for other file types, we should consider removing this restriction.
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O_MR1) {
      registry.register(new ExifInterfaceImageHeaderParser());
    }
    //后续的流程是创建一些处理图片的解析器、解码器、转码器等，然后将他们添加到 Registry 中
	...
    // 创建 ImageViewTargetFactory，用来给 View 获取正确类型的 ViewTarget（BitmapImageViewTarget 或 DrawableImageViewTarget）
    ImageViewTargetFactory imageViewTargetFactory = new ImageViewTargetFactory();
    // 构建一个 Glide 专属的上下文
    glideContext =
        new GlideContext(
            context,
            arrayPool,
            registry,
            imageViewTargetFactory,
            defaultRequestOptionsFactory,
            defaultTransitionOptions,
            defaultRequestListeners,
            engine,
            experiments,
            logLevel);
  }
```

至此，Glide 真正创建成功。接下来回到`getRetriever().get()`方法的get方法

#### 1.2 RequestManagerRetriever#get()

RequestManagerRetriever#get() 的重载方法同样有 6 个：

```java
RequestManagerRetriever#get(Context context)
RequestManagerRetriever#get(Activity activity)
RequestManagerRetriever#get(FragmentActivity activity)
RequestManagerRetriever#get(Fragment fragment)
RequestManagerRetriever#get(android.app.Fragment fragment)
RequestManagerRetriever#get(View view)
```

同样可以分成两种情况，即 Application 与非 Application 类型。

先看下 Application 类型的情况：

```java
  @NonNull
  public RequestManager get(@NonNull Context context) {
    if (context == null) {
      throw new IllegalArgumentException("You cannot start a load on a null Context");
      //判断如果当前线程是在主线程
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
      //传入非Application类型的参数
      if (context instanceof FragmentActivity) {
        return get((FragmentActivity) context);
      } else if (context instanceof Activity) {
        return get((Activity) context);
      } else if (context instanceof ContextWrapper
          && ((ContextWrapper) context).getBaseContext().getApplicationContext() != null) {
        return get(((ContextWrapper) context).getBaseContext());
      }
    }
    //即传入Application类型的参数
    return getApplicationManager(context);
  }
```

如果当前线程是在主线程，并且 context 不属于 Application 类型，那么会走对应重载方法。属于 Application 类型，就会调用 getApplicationManager(context) 方法。

实现如下：

```java
 @NonNull
  private RequestManager getApplicationManager(@NonNull Context context) {
    // 使用了DCL单例模式来获取 RequestManager
    if (applicationManager == null) {
      synchronized (this) {
        if (applicationManager == null) {
          // 用 Application 类型的 Context 来获取 Glide 的实例
          // 这里并没有专门做生命周期的处理
          // 因为 Application 对象的生命周期即为应用程序的生命周期
          // 所以在这里图片请求的生命周期是和应用程序同步的
          Glide glide = Glide.get(context.getApplicationContext());
          applicationManager =
              factory.build(
                  glide,
                  new ApplicationLifecycle(),
                  new EmptyRequestManagerTreeNode(),
                  context.getApplicationContext());
        }
      }
    }

    return applicationManager;
  }
```

使用DCL单例模式来获取 RequestManager，里面重新用 Application 类型的 Context 来获取 Glide 的实例。这里没有专门做生命周期的处理， 因为 Application 对象的生命周期即为应用程序的生命周期，所以在这里图片请求的生命周期是和应用程序同步的。

接下来看非 Application 类型的情况，以FragmentActivity 为例：

```java
  public RequestManager get(@NonNull FragmentActivity activity) {
    // 是否在非UI线程
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else {
      // 检查 Activity 是否销毁
      assertNotDestroyed(activity);
      frameWaiter.registerSelf(activity);
      // 获取当前 Activity 的 FragmentManager
      FragmentManager fm = activity.getSupportFragmentManager();
      return supportFragmentGet(activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }
```

如果是Activity，调用 supportFragmentGet() 方法来获取 RequestManager。

```java
  private RequestManager supportFragmentGet(
      @NonNull Context context,
      @NonNull FragmentManager fm,
      @Nullable Fragment parentHint,
      boolean isParentVisible) {
    // 获取了一个隐藏的 Fragment
    SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm, parentHint);
    RequestManager requestManager = current.getRequestManager();
    // 实际是将 SupportRequestManagerFragment、RequestManager、ActivityFragmentLifecycle 都关联在一起了
    // 又因为 Fragment 的生命周期和 Activity 是同步的，
    // 所以 Activity 生命周期发生变化的时候，隐藏的 Fragment 的生命周期是同步变化的，
    // 这样 Glide 就可以根据这个 Fragment 的生命周期进行请求管理了。
    if (requestManager == null) {
      Glide glide = Glide.get(context);
      // 这里的 current.getGlideLifecycle() 就是
      // getSupportRequestManagerFragment 中实例化的 ActivityFragmentLifecycle
      // 这样 RequestManager 就与 ActivityFragmentLifecycle 进行了关联。
      requestManager =
          factory.build(
              glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
      if (isParentVisible) {
        requestManager.onStart();
      }
      // 将 RequestManager 设置到 SupportRequestManagerFragment 中
      current.setRequestManager(requestManager);
    }
    return requestManager;
  }
```

首先获取了一个隐藏的 Fragment。之后的 current.getGlideLifecycle() 就是实例化的 ActivityFragmentLifecycle，这样 RequestManager 就与 ActivityFragmentLifecycle 进行了关联。

最后将 RequestManager 设置到 SupportRequestManagerFragment 中。

```java
  //RequestManagerRetriever#getSupportRequestManagerFragment()
  @NonNull
  private SupportRequestManagerFragment getSupportRequestManagerFragment(
      @NonNull final FragmentManager fm, @Nullable Fragment parentHint) {
    SupportRequestManagerFragment current =
        (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
      current = pendingSupportRequestManagerFragments.get(fm);
      if (current == null) {
        //实例化了 SupportRequestManagerFragment
        current = new SupportRequestManagerFragment();
        current.setParentFragmentHint(parentHint);
        pendingSupportRequestManagerFragments.put(fm, current);
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
        handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }

 //SupportRequestManagerFragment#SupportRequestManagerFragment()
 public SupportRequestManagerFragment() {
    // 实例化了 ActivityFragmentLifecycle
    // ActivityFragmentLifecycle 实现了 Lifecycle
    // 主要用来监听 Activity 与 Fragment 的生命周期。
    // ActivityFragmentLifecycle 与 SupportRequestManagerFragment 的生命周期关联起来了
    this(new ActivityFragmentLifecycle());
  }

//ActivityFragmentLifecycle
class ActivityFragmentLifecycle implements Lifecycle {
  private final Set<LifecycleListener> lifecycleListeners =
      Collections.newSetFromMap(new WeakHashMap<LifecycleListener, Boolean>());
  private boolean isStarted;
  private boolean isDestroyed;

  @Override
  public void addListener(@NonNull LifecycleListener listener) {
    lifecycleListeners.add(listener);

    if (isDestroyed) {
      listener.onDestroy();
    } else if (isStarted) {
      listener.onStart();
    } else {
      listener.onStop();
    }
  }

  @Override
  public void removeListener(@NonNull LifecycleListener listener) {
    lifecycleListeners.remove(listener);
  }

  void onStart() {
    isStarted = true;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onStart();
    }
  }

  void onStop() {
    isStarted = false;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onStop();
    }
  }

  void onDestroy() {
    isDestroyed = true;
    for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
      lifecycleListener.onDestroy();
    }
  }
}

```

在实例化 SupportRequestManagerFragment 的时候又实例化了 ActivityFragmentLifecycle，ActivityFragmentLifecycle 实现了 Lifecycle，主要用来监听 Activity 与 Fragment 的生命周期。

实际是将 SupportRequestManagerFragment、RequestManager、ActivityFragmentLifecycle 都关联在一起了，又因为 Fragment 的生命周期和 Activity 是同步的， 所以 Activity 生命周期发生变化的时候，隐藏的 Fragment 的生命周期是同步变化的，这样 Glide 就可以根据这个 Fragment 的生命周期进行请求管理了。

#### 1.3 总结

with() 方法主要功能如下：

- 获取 AndroidManifest.xml 文件中配置的自定义模块与 @GlideModule 注解标识的自定义模块，然后进行 Glide 配置的更改与组件的替换。
- 初始化构建 Glide 实例需要的各种配置信息，例如线程池、Bitmap 池、缓存策略、Engine 等，然后利用这些来创建具体的 Glide。
- 将 Glide 的请求与 Application 或者隐藏的 Fragment 的生命周期进行绑定。

### 2. load()

load() 的重载方法有 9 个：

```java
RequestManager#load(Bitmap bitmap);
RequestManager#load(Drawable drawable);
RequestManager#load(String string);
RequestManager#load(Uri uri);
RequestManager#load(File file);
RequestManager#load(Integer resourceId);
RequestManager#load(URL url);
RequestManager#load(byte[] model);
RequestManager#load(Object model);
```

这些重载方法最终都会调用 asDrawable().load()。这里只拿参数为图片链接字符串的来分析。

```java
  public RequestBuilder<Drawable> load(@Nullable String string) {
    return asDrawable().load(string);
  }
```

分为`asDrawable()`和`load(string)`2个部分。我们分别来分析

#### 2.1 RequestManager#asDrawable()

```java
  public RequestBuilder<Drawable> asDrawable() {
    return as(Drawable.class);
  }

  public <ResourceType> RequestBuilder<ResourceType> as(
      @NonNull Class<ResourceType> resourceClass) {
    // 创建一个 RequestBuilder 的实例
    return new RequestBuilder<>(glide, this, resourceClass, context);
  }

```

asDrawable方法最终创建了 RequestBuilder 的实例并返回，查看RequestBuilder的构造函数：

```java
  protected RequestBuilder(
      @NonNull Glide glide,
      RequestManager requestManager,
      Class<TranscodeType> transcodeClass,
      Context context) {
    // 给 RequestBuilder 中的一些常量进行赋值
    this.glide = glide;
    this.requestManager = requestManager;
    this.transcodeClass = transcodeClass;
    this.context = context;
    this.transitionOptions = requestManager.getDefaultTransitionOptions(transcodeClass);
    this.glideContext = glide.getGlideContext();
    // 初始化请求监听
    initRequestListeners(requestManager.getDefaultRequestListeners());
    // 将默认选项应用于请求
    apply(requestManager.getDefaultRequestOptions());
  }
```

apply是将默认选项应用于请求，实现如下：

```java
  public RequestBuilder<TranscodeType> apply(@NonNull BaseRequestOptions<?> requestOptions) {
    Preconditions.checkNotNull(requestOptions);
    // 调用了父类的 apply() 方法
    return super.apply(requestOptions);
  }
```

调用了父类BaseRequestOptions的 apply 方法：

```java

```









#### 2.2 RequestBuilder#load()



### 3. into()



## 三、 缓存机制



---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)











