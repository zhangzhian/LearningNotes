# 开发者营地|Glide: 源码详解 

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

稍微进阶一点的用法，参数设置

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

### 4. 源码详解综述

本文着重分析Glide的核心流程源码：

第二章分析Glide的核心流程：包括`with()`、`load()`和`into()`

Glide的其他部分源码后续在其他同系列文章中进行分析。

## 二、核心流程

Glide 默认是配置了内存与磁盘缓存的，所以这里我们先假设禁用内存和磁盘缓存，来分析核心流程。缓存相关的放到后面分析。

代码表示如下：

```java
Glide.with(this)
        .load(url)
        .skipMemoryCache(true) // 禁用内存缓存
        .diskCacheStrategy(DiskCacheStrategy.NONE) // 禁用磁盘缓存
        .into(imageView);
```

### 1. with()

with()有6个重载方法，均返回RequestManager，如下

```java
  @NonNull
  public static RequestManager with(@NonNull Context context) {
    return getRetriever(context).get(context);
  }

  @NonNull
  public static RequestManager with(@NonNull Activity activity) {
    return getRetriever(activity).get(activity);
  }

  @NonNull
  public static RequestManager with(@NonNull FragmentActivity activity) {
    return getRetriever(activity).get(activity);
  }

  @NonNull
  public static RequestManager with(@NonNull Fragment fragment) {
    return getRetriever(fragment.getContext()).get(fragment);
  }

  @SuppressWarnings("deprecation")
  @Deprecated
  @NonNull
  public static RequestManager with(@NonNull android.app.Fragment fragment) {
    return getRetriever(fragment.getActivity()).get(fragment);
  }

  @NonNull
  public static RequestManager with(@NonNull View view) {
    return getRetriever(view.getContext()).get(view);
  }
```

这些重载方法可以分成两种情况，即 Application（Context）类型与非 Application，具体有Activity、Fragment、View 或 Fragment类型，这些参数的作用是和图片加载的生命周期相关的。

最终都会调用 `getRetriever().get()`方法，我们分为两部分看。

#### 1.1 Gilde#getRetriever()

getRetriever() 定义如下：

```java
  @NonNull
  private static RequestManagerRetriever getRetriever(@Nullable Context context) {
    Preconditions.checkNotNull(context, "...");
    return Glide.get(context).getRequestManagerRetriever();
  }
```

做一个非NULL的判断，然后调用`Glide.get(context).getRequestManagerRetriever()`返回一个`RequestManagerRetriever`。这里我们同样分为2部分来看。

##### 1.1.1 Glide#get(context)

先看下 `Glide#get(context)`：

```java
  @GuardedBy("Glide.class")
  private static volatile Glide glide;

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

这里使用了标准的DCL单例模式来获取 Glide 的实例。

先看`getAnnotationGeneratedGlideModules`方法。

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

该方法用来实例化我们用 `@GlideModule` 注解标识的自定义模块，自定义模块的用法请参考官方 [中文文档](https://muyangmin.github.io/glide-docs-cn/)。

我们在来看`checkAndInitializeGlide`方法，从名称上看是用来检查和初始化的，如下：

```java
  private static volatile boolean isInitializing;

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

这里使用了一个volatile类型的`isInitializing`变量，来防止重复初始化。

接下来`initializeGlide`方法，从名字上可以看出是进行初始化的方法，如下：

```java
  @GuardedBy("Glide.class")
  private static void initializeGlide(
      @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
    //注意new了一个GlideBuilder对象传进去。
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

`initializeGlide`方法调用了重载方法，增加了一个`GlideBuilder`对象，使用到了建造者模式，主要是完成自定义模块的相关功能和真正进行初始化。在最后通过`Glide.glide = glide;`将创建好的 Glide 赋值给 Glide 的静态全家变量。

我们需要重点关注的是` Glide glide = builder.build(applicationContext);`这里是真正的初始化。

实现在`GlideBuilder#build`如下：

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

build() 方法主要是创建一些线程池、Bitmap 池、缓存策略、Engine 等，然后利用这些来创建具体的 Glide。

继续看下 Glide 的构造函数：

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

我们看一下 GlideContext 的构造方法

```java
  private final GlideContext glideContext;
  public GlideContext(
      @NonNull Context context,
      @NonNull ArrayPool arrayPool,
      @NonNull Registry registry,
      @NonNull ImageViewTargetFactory imageViewTargetFactory,
      @NonNull RequestOptionsFactory defaultRequestOptionsFactory,
      @NonNull Map<Class<?>, TransitionOptions<?, ?>> defaultTransitionOptions,
      @NonNull List<RequestListener<Object>> defaultRequestListeners,
      @NonNull Engine engine,
      @NonNull GlideExperiments experiments,
      int logLevel) {
    super(context.getApplicationContext());
    this.arrayPool = arrayPool;
    this.registry = registry;
    this.imageViewTargetFactory = imageViewTargetFactory;
    this.defaultRequestOptionsFactory = defaultRequestOptionsFactory;
    this.defaultRequestListeners = defaultRequestListeners;
    this.defaultTransitionOptions = defaultTransitionOptions;
    this.engine = engine;
    this.experiments = experiments;
    this.logLevel = logLevel;
  }
```

存储了一些上下文数据的索引。

至此，Glide 真正创建成功。

##### 1.1.2 Glide#getRequestManagerRetriever()

接下来我们看一下`getRequestManagerRetriever()`方法

```java
  private final RequestManagerRetriever requestManagerRetriever;

  public RequestManagerRetriever getRequestManagerRetriever() {
    return requestManagerRetriever;
  }
```

返回一个`RequestManagerRetriever`对象。从名字上理解为请求管理者检索器，我们往回翻一下，看一下其相关实现。

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
      ...
	  this.requestManagerRetriever = requestManagerRetriever;
      ...
  }         
```

在Glide的初始化中，可以看到是通过初始化赋值的。初始化又是通过build函数构建的，回到build函数。

```java
  @NonNull
  Glide build(@NonNull Context context) {
  
      ...

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

这里我们可以看到 new 了一个 RequestManagerRetriever 对象。

这里的 requestManagerFactory 是前面 `GlideBuilder#setRequestManagerFactory()` 设置进来的

```java
RequestManagerRetriever requestManagerRetriever = 
new RequestManagerRetriever(requestManagerFactory, experiments);
```

看一下其构造方法。

```java
  private final RequestManagerFactory factory;
  
  public RequestManagerRetriever(
      @Nullable RequestManagerFactory factory, GlideExperiments experiments) {
    this.factory = factory != null ? factory : DEFAULT_FACTORY;
    handler = new Handler(Looper.getMainLooper(), this /* Callback */);

    frameWaiter = buildFrameWaiter(experiments);
  }

  public interface RequestManagerFactory {
    @NonNull
    RequestManager build(
        @NonNull Glide glide,
        @NonNull Lifecycle lifecycle,
        @NonNull RequestManagerTreeNode requestManagerTreeNode,
        @NonNull Context context);
  }

  private static final RequestManagerFactory DEFAULT_FACTORY =
      new RequestManagerFactory() {
        @NonNull
        @Override
        public RequestManager build(
            @NonNull Glide glide,
            @NonNull Lifecycle lifecycle,
            @NonNull RequestManagerTreeNode requestManagerTreeNode,
            @NonNull Context context) {
          return new RequestManager(glide, lifecycle, requestManagerTreeNode, context);
        }
      };
```

初始化方法中保存了 RequestManagerFactory 的索引，如果没有的话使用默认的工厂`DEFAULT_FACTORY`。RequestManagerFactory 是一个接口，只有一个方法`build`，返回一个 RequestManager 对象。可以看到工厂主要使用了生成 RequestManager 对象的。

初始化方法中还获取并保存了主线程的Handler的索引。

接下来回到`getRetriever().get()`方法的`get()`方法。

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

分成两种情况，即 Application 与非 Application 类型。

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

##### 1.2.1 ApplicationContext

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

使用DCL单例模式来获取 RequestManager，里面重新用 Application 类型的 Context 来获取 Glide 的实例。

```java
class ApplicationLifecycle implements Lifecycle {
  @Override
  public void addListener(@NonNull LifecycleListener listener) {
    listener.onStart();
  }

  @Override
  public void removeListener(@NonNull LifecycleListener listener) {
    // Do nothing.
  }
}
```

ApplicationLifecycle 为空实现，这里没有专门做生命周期的处理， 因为 Application 对象的生命周期即为应用程序的生命周期，所以在这里图片请求的生命周期是和应用程序同步的。

##### 1.2.2 非ApplicationContext

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

如果是Activity，调用 `supportFragmentGet()` 方法来获取 RequestManager。

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

首先获取了一个隐藏的 Fragment。

之后的 `current.getGlideLifecycle()` 就是实例化的 ActivityFragmentLifecycle，这样 RequestManager 就与 ActivityFragmentLifecycle 进行了关联。

最后将 RequestManager 设置到 SupportRequestManagerFragment 中。

我们详细看一下`getSupportRequestManagerFragment(fm, parentHint)`相关的代码。

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

  @VisibleForTesting
  @SuppressLint("ValidFragment")
  public SupportRequestManagerFragment(@NonNull ActivityFragmentLifecycle lifecycle) {
    this.lifecycle = lifecycle;
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

load() 的重载方法有 9 个，均返回`RequestBuilder<Drawable>`：

```java
RequestBuilder<Drawable>#load(Bitmap bitmap);
RequestBuilder<Drawable>#load(Drawable drawable);
RequestBuilder<Drawable>#load(String string);
RequestBuilder<Drawable>#load(Uri uri);
RequestBuilder<Drawable>#load(File file);
RequestBuilder<Drawable>#load(Integer resourceId);
RequestBuilder<Drawable>#load(URL url);
RequestBuilder<Drawable>#load(byte[] model);
RequestBuilder<Drawable>#load(Object model);
```

这些重载方法最终都会调用 `asDrawable().load()`。这里只拿参数为图片链接字符串的来分析。

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

`asDrawable()`方法最终创建了 RequestBuilder 的实例并返回，查看RequestBuilder的构造函数：

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
  public T apply(@NonNull BaseRequestOptions<?> o) {
    // 配置选项有很多，包括磁盘缓存策略，加载中的占位图，加载失败的占位图等
    if (isAutoCloneEnabled) {
      return clone().apply(o);
    }
    BaseRequestOptions<?> other = o;

    if (isSet(other.fields, SIZE_MULTIPLIER)) {
      sizeMultiplier = other.sizeMultiplier;
    }
    if (isSet(other.fields, USE_UNLIMITED_SOURCE_GENERATORS_POOL)) {
      useUnlimitedSourceGeneratorsPool = other.useUnlimitedSourceGeneratorsPool;
    }
    if (isSet(other.fields, USE_ANIMATION_POOL)) {
      useAnimationPool = other.useAnimationPool;
    }
    if (isSet(other.fields, DISK_CACHE_STRATEGY)) {
      diskCacheStrategy = other.diskCacheStrategy;
    }
    if (isSet(other.fields, PRIORITY)) {
      priority = other.priority;
    }
    if (isSet(other.fields, ERROR_PLACEHOLDER)) {
      errorPlaceholder = other.errorPlaceholder;
      errorId = 0;
      fields &= ~ERROR_ID;
    }
    if (isSet(other.fields, ERROR_ID)) {
      errorId = other.errorId;
      errorPlaceholder = null;
      fields &= ~ERROR_PLACEHOLDER;
    }
    if (isSet(other.fields, PLACEHOLDER)) {
      placeholderDrawable = other.placeholderDrawable;
      placeholderId = 0;
      fields &= ~PLACEHOLDER_ID;
    }
    if (isSet(other.fields, PLACEHOLDER_ID)) {
      placeholderId = other.placeholderId;
      placeholderDrawable = null;
      fields &= ~PLACEHOLDER;
    }
    if (isSet(other.fields, IS_CACHEABLE)) {
      isCacheable = other.isCacheable;
    }
    if (isSet(other.fields, OVERRIDE)) {
      overrideWidth = other.overrideWidth;
      overrideHeight = other.overrideHeight;
    }
    if (isSet(other.fields, SIGNATURE)) {
      signature = other.signature;
    }
    if (isSet(other.fields, RESOURCE_CLASS)) {
      resourceClass = other.resourceClass;
    }
    if (isSet(other.fields, FALLBACK)) {
      fallbackDrawable = other.fallbackDrawable;
      fallbackId = 0;
      fields &= ~FALLBACK_ID;
    }
    if (isSet(other.fields, FALLBACK_ID)) {
      fallbackId = other.fallbackId;
      fallbackDrawable = null;
      fields &= ~FALLBACK;
    }
    if (isSet(other.fields, THEME)) {
      theme = other.theme;
    }
    if (isSet(other.fields, TRANSFORMATION_ALLOWED)) {
      isTransformationAllowed = other.isTransformationAllowed;
    }
    if (isSet(other.fields, TRANSFORMATION_REQUIRED)) {
      isTransformationRequired = other.isTransformationRequired;
    }
    if (isSet(other.fields, TRANSFORMATION)) {
      transformations.putAll(other.transformations);
      isScaleOnlyOrNoTransform = other.isScaleOnlyOrNoTransform;
    }
    if (isSet(other.fields, ONLY_RETRIEVE_FROM_CACHE)) {
      onlyRetrieveFromCache = other.onlyRetrieveFromCache;
    }

    // Applying options with dontTransform() is expected to clear our transformations.
    if (!isTransformationAllowed) {
      transformations.clear();
      fields &= ~TRANSFORMATION;
      isTransformationRequired = false;
      fields &= ~TRANSFORMATION_REQUIRED;
      isScaleOnlyOrNoTransform = true;
    }

    fields |= other.fields;
    options.putAll(other.options);

    return selfOrThrowIfLocked();
  }
```

配置选项有很多，包括磁盘缓存策略，加载中的占位图，加载失败的占位图等。 到这里 asDrawable() 方法完成了。接下来查看load部分。

#### 2.2 RequestBuilder#load()

```java
  @Nullable private Object model;

  public RequestBuilder<TranscodeType> load(@Nullable String string) {
    //调用了 loadGeneric() 方法
    return loadGeneric(string);
  }

  @NonNull
  private RequestBuilder<TranscodeType> loadGeneric(@Nullable Object model) {
    // 将传进来的图片资源赋值给了变量 model
    this.model = model;
    // isModelSet 标记已经调用过 load() 方法
    isModelSet = true;
    return this;
  }
```

个方法非常简单，首先调用了 loadGeneric() 方法，然后 loadGeneric() 方法中将传进来的图片资源赋值给了变量 model，最后用 isModelSet 标记已经调用过 load() 方法了。

#### 2.3 总结

load() 方法比较简单了，主要是通过前面实例化的 Glide 与 RequestManager 来创建 RequestBuilder，然后将传进来的参数赋值给 model。

### 3. into()

前两个方法都没有涉及到图片的请求、缓存、解码等逻辑，其实都在 `into()` 方法中，这个方法也是最复杂的。

首先查看`into()`函数，`RequestBuilder#into()`如下：

```java
  @NonNull
  public <Y extends Target<TranscodeType>> Y into(@NonNull Y target) {
    return into(target, /*targetListener=*/ null, Executors.mainThreadExecutor());
  }

  @NonNull
  @Synthetic
  <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      Executor callbackExecutor) {
    return into(target, targetListener, /*options=*/ this, callbackExecutor);
  }

  private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> options,
      Executor callbackExecutor) {
    Preconditions.checkNotNull(target);
    if (!isModelSet) {
      throw new IllegalArgumentException("You must call #load() before calling #into()");
    }

    //创建一个Request
    Request request = buildRequest(target, targetListener, options, callbackExecutor);

    Request previous = target.getRequest();
    if (request.isEquivalentTo(previous)
        && !isSkipMemoryCacheWithCompletePreviousRequest(options, previous)) {
      if (!Preconditions.checkNotNull(previous).isRunning()) {
        previous.begin();
      }
      return target;
    }

    requestManager.clear(target);
    target.setRequest(request);
    //请求，注意Request是一个接口
    requestManager.track(target, request);

    return target;
  }
```

可以看到，调用了重载方法。在最终的方法中进行了真正的处理逻辑。

我们需要关注的点包括包含2个：`RequestBuilder#buildRequest()`和`RequestManager#track()`。

#### 3.1 RequestBuilder#buildRequest()

通过函数名可以猜测是生成一个Request对象，该函数最终返回是一个Request对象，这里我们先看一下Request对象。

```java
public interface Request {
  void begin();
  void clear();
  void pause();
  boolean isRunning();
  boolean isComplete();
  boolean isCleared();
  boolean isAnyResourceSet();
  boolean isEquivalentTo(Request other);
}
```

Request 是一个接口，包含了8个方法。这里我们先留意一下即可。

我们查看 `buildRequest()`方法：

```java
  private Request buildRequest(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {
    return buildRequestRecursive(
        /*requestLock=*/ new Object(),
        target,
        targetListener,
        /*parentCoordinator=*/ null,
        transitionOptions,
        requestOptions.getPriority(),
        requestOptions.getOverrideWidth(),
        requestOptions.getOverrideHeight(),
        requestOptions,
        callbackExecutor);
  }
```

这里调用了`buildRequestRecursive()`方法：

```java
  private Request buildRequestRecursive(
      Object requestLock,
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {

    ErrorRequestCoordinator errorRequestCoordinator = null;
    // errorBuilder 不为空，即我们设置了在主请求失败时开始新的请求（如下设置） 才会去递归构建错误请求
    // 设置在主请求失败时开始新的请求 Glide.with(this).load(url).error(Glide.with(this).load(fallbackUrl)).into(imageView);
    if (errorBuilder != null) {
      errorRequestCoordinator = new ErrorRequestCoordinator(requestLock, parentCoordinator);
      parentCoordinator = errorRequestCoordinator;
    }

    // 递归构建缩略图请求
    Request mainRequest =
        buildThumbnailRequestRecursive(
            requestLock,
            target,
            targetListener,
            parentCoordinator,
            transitionOptions,
            priority,
            overrideWidth,
            overrideHeight,
            requestOptions,
            callbackExecutor);

    if (errorRequestCoordinator == null) {
      return mainRequest;
    }

    int errorOverrideWidth = errorBuilder.getOverrideWidth();
    int errorOverrideHeight = errorBuilder.getOverrideHeight();
    if (Util.isValidDimensions(overrideWidth, overrideHeight) && !errorBuilder.isValidOverride()) {
      errorOverrideWidth = requestOptions.getOverrideWidth();
      errorOverrideHeight = requestOptions.getOverrideHeight();
    }

    // 递归构建错误请求
    Request errorRequest =
        errorBuilder.buildRequestRecursive(
            requestLock,
            target,
            targetListener,
            errorRequestCoordinator,
            errorBuilder.transitionOptions,
            errorBuilder.getPriority(),
            errorOverrideWidth,
            errorOverrideHeight,
            errorBuilder,
            callbackExecutor);
    errorRequestCoordinator.setRequests(mainRequest, errorRequest);
    return errorRequestCoordinator;
  }
```

`buildRequestRecursive()`，从名字上来看，是递归地生成请求。

`errorBuilder` 不为空，即我们设置了在主请求失败时开始新的请求（如下设置），才会走最后去递归构建错误请求。

```java
// 设置在主请求失败时开始新的请求
Glide.with(this).load(url).error(Glide.with(this).load(fallbackUrl)).into(imageView);
```

若`errorBuilder` 为空，则调用`buildThumbnailRequestRecursive()`递归地构造一个缩略图请求。然后就返回。

接下来我们主要关注`buildThumbnailRequestRecursive()`。

```java
  private Request buildThumbnailRequestRecursive(
      Object requestLock,
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      BaseRequestOptions<?> requestOptions,
      Executor callbackExecutor) {
    // 设置了缩略图请求的时候会走这里
    // Glide.with(this).load(url).thumbnail(Glide.with(this).load(thumbnailUrl)).into(imageView);
    if (thumbnailBuilder != null) {
      // Recursive case: contains a potentially recursive thumbnail request builder.
      if (isThumbnailBuilt) {
        throw new IllegalStateException(
            "You cannot use a request as both the main request and a "
                + "thumbnail, consider using clone() on the request(s) passed to thumbnail()");
      }

      TransitionOptions<?, ? super TranscodeType> thumbTransitionOptions =
          thumbnailBuilder.transitionOptions;

      // Apply our transition by default to thumbnail requests but avoid overriding custom options
      // that may have been applied on the thumbnail request explicitly.
      if (thumbnailBuilder.isDefaultTransitionOptionsSet) {
        thumbTransitionOptions = transitionOptions;
      }

      Priority thumbPriority =
          thumbnailBuilder.isPrioritySet()
              ? thumbnailBuilder.getPriority()
              : getThumbnailPriority(priority);

      int thumbOverrideWidth = thumbnailBuilder.getOverrideWidth();
      int thumbOverrideHeight = thumbnailBuilder.getOverrideHeight();
      if (Util.isValidDimensions(overrideWidth, overrideHeight)
          && !thumbnailBuilder.isValidOverride()) {
        thumbOverrideWidth = requestOptions.getOverrideWidth();
        thumbOverrideHeight = requestOptions.getOverrideHeight();
      }
      // 创建一个协调器，用来协调这两个请求，这样可以同时进行原图与缩略图的请求。
      ThumbnailRequestCoordinator coordinator =
          new ThumbnailRequestCoordinator(requestLock, parentCoordinator);
      // 获取一个原图请求
      Request fullRequest =
          obtainRequest(
              requestLock,
              target,
              targetListener,
              requestOptions,
              coordinator,
              transitionOptions,
              priority,
              overrideWidth,
              overrideHeight,
              callbackExecutor);
      isThumbnailBuilt = true;
      // Recursively generate thumbnail requests.
      // 根据设置的 thumbnailBuilder 来生成缩略图请求
      Request thumbRequest =
          thumbnailBuilder.buildRequestRecursive(
              requestLock,
              target,
              targetListener,
              coordinator,
              thumbTransitionOptions,
              thumbPriority,
              thumbOverrideWidth,
              thumbOverrideHeight,
              thumbnailBuilder,
              callbackExecutor);
      isThumbnailBuilt = false;
      coordinator.setRequests(fullRequest, thumbRequest);
      return coordinator;
    } else if (thumbSizeMultiplier != null) {
      // 设置缩略图的缩略比例
      // Glide.with(this).load(url).thumbnail(0.5f).into(imageView);
      // Base case: thumbnail multiplier generates a thumbnail request, but cannot recurse.
      // 同样是通过一个协调器来同时进行原图与缩略图的请求，不同的是这里生成缩略图用的是缩略比例。
      ThumbnailRequestCoordinator coordinator =
          new ThumbnailRequestCoordinator(requestLock, parentCoordinator);
      //原图
      Request fullRequest =
          obtainRequest(
              requestLock,
              target,
              targetListener,
              requestOptions,
              coordinator,
              transitionOptions,
              priority,
              overrideWidth,
              overrideHeight,
              callbackExecutor);
      BaseRequestOptions<?> thumbnailOptions =
          requestOptions.clone().sizeMultiplier(thumbSizeMultiplier);
      // 缩略图
      Request thumbnailRequest =
          obtainRequest(
              requestLock,
              target,
              targetListener,
              thumbnailOptions,
              coordinator,
              transitionOptions,
              getThumbnailPriority(priority),
              overrideWidth,
              overrideHeight,
              callbackExecutor);

      coordinator.setRequests(fullRequest, thumbnailRequest);
      return coordinator;
    } else {
      // 没有缩略图相关设置，直接获取原图请求。
      // Base case: no thumbnail.
      return obtainRequest(
          requestLock,
          target,
          targetListener,
          requestOptions,
          parentCoordinator,
          transitionOptions,
          priority,
          overrideWidth,
          overrideHeight,
          callbackExecutor);
    }
  }
```

函数比较长，先看 `if...else` 语句的整体结果

`thumbnailBuilder`不为空，说明设置了缩略图（如下设置）。

```java
Glide.with(this).load(url).thumbnail(Glide.with(this).load(thumbnailUrl)).into(imageView);
```

`thumbSizeMultiplier`不为空，说明设置了缩略比例（如下设置）。

```java
Glide.with(this).load(url).thumbnail(0.5f).into(imageView);
```

否则的会，没有缩略相关设置，直接获取原图。

我们只关心核心流程，所以看最后的else分支。

```java
  private Request obtainRequest(
      Object requestLock,
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      BaseRequestOptions<?> requestOptions,
      RequestCoordinator requestCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      Executor callbackExecutor) {
    // 最终是实例化了一个 SingleRequest 的实例，
    // 构建请求这一步已经完成了
    // 需要注意一下Target<TranscodeType> target,
    return SingleRequest.obtain(
        context,
        glideContext,
        requestLock,
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        glideContext.getEngine(),
        transitionOptions.getTransitionFactory(),
        callbackExecutor);
  }
}

//SingleRequest#obtain
  public static <R> SingleRequest<R> obtain(
      Context context,
      GlideContext glideContext,
      Object requestLock,
      Object model,
      Class<R> transcodeClass,
      BaseRequestOptions<?> requestOptions,
      int overrideWidth,
      int overrideHeight,
      Priority priority,
      Target<R> target,
      RequestListener<R> targetListener,
      @Nullable List<RequestListener<R>> requestListeners,
      RequestCoordinator requestCoordinator,
      Engine engine,
      TransitionFactory<? super R> animationFactory,
      Executor callbackExecutor) {
    return new SingleRequest<>(
        context,
        glideContext,
        requestLock,
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        engine,
        animationFactory,
        callbackExecutor);
  }

  private SingleRequest(
      Context context,
      GlideContext glideContext,
      @NonNull Object requestLock,
      @Nullable Object model,
      Class<R> transcodeClass,
      BaseRequestOptions<?> requestOptions,
      int overrideWidth,
      int overrideHeight,
      Priority priority,
      Target<R> target,
      @Nullable RequestListener<R> targetListener,
      @Nullable List<RequestListener<R>> requestListeners,
      RequestCoordinator requestCoordinator,
      Engine engine,
      TransitionFactory<? super R> animationFactory,
      Executor callbackExecutor) {
    this.requestLock = requestLock;
    this.context = context;
    this.glideContext = glideContext;
    this.model = model;
    this.transcodeClass = transcodeClass;
    this.requestOptions = requestOptions;
    this.overrideWidth = overrideWidth;
    this.overrideHeight = overrideHeight;
    this.priority = priority;
    this.target = target;
    this.targetListener = targetListener;
    this.requestListeners = requestListeners;
    this.requestCoordinator = requestCoordinator;
    this.engine = engine;
    this.animationFactory = animationFactory;
    this.callbackExecutor = callbackExecutor;
    status = Status.PENDING;

    if (requestOrigin == null && glideContext.getExperiments().isEnabled(LogRequestOrigins.class)) {
      requestOrigin = new RuntimeException("Glide request origin trace");
    }
  }

```

最终是实例化了一个 SingleRequest 的实例，也就是说构建请求这一步已经完成了。

前面我们注意到返回的是Request对象，可以看到SingleRequest是Request的具体实现。

```java
public final class SingleRequest<R> implements Request, SizeReadyCallback, ResourceCallback{...}
```

接下来分析执行请求这一步。

#### 3.2 RequestManager#track()

代码如下：

```java
  @GuardedBy("this")
  private final TargetTracker targetTracker = new TargetTracker();

  @GuardedBy("this")
  private final RequestTracker requestTracker;

  synchronized void track(@NonNull Target<?> target, @NonNull Request request) {
    // 将一个 target 加入 Set 集合中
    targetTracker.track(target);
    //请求
    requestTracker.runRequest(request);
  }
```

track的输入参数是`Target<?> target`和` @NonNull Request`。

分别执行了`TargetTracker#track`和`RequestTracker#runRequest`。

`TargetTracker#track()`：

```java
public final class TargetTracker implements LifecycleListener {
  private final Set<Target<?>> targets =
      Collections.newSetFromMap(new WeakHashMap<Target<?>, Boolean>());

  public void track(@NonNull Target<?> target) {
    targets.add(target);
  }

  public void untrack(@NonNull Target<?> target) {
    targets.remove(target);
  }

  @Override
  public void onStart() {
    for (Target<?> target : Util.getSnapshot(targets)) {
      target.onStart();
    }
  }

  @Override
  public void onStop() {
    for (Target<?> target : Util.getSnapshot(targets)) {
      target.onStop();
    }
  }

  @Override
  public void onDestroy() {
    for (Target<?> target : Util.getSnapshot(targets)) {
      target.onDestroy();
    }
  }

  @NonNull
  public List<Target<?>> getAll() {
    return Util.getSnapshot(targets);
  }

  public void clear() {
    targets.clear();
  }
}
```

TargetTracker看着比较简单，`track()`方法是将一个 target 加入 Set 集合中。

`RequestTracker#runRequest()`：

```java
public class RequestTracker {
  private final Set<Request> requests =
      Collections.newSetFromMap(new WeakHashMap<Request, Boolean>());
  private final List<Request> pendingRequests = new ArrayList<>();

  private boolean isPaused;
  
  /** Starts tracking the given request. */
  public void runRequest(@NonNull Request request) {
    // 将请求加入一个 Set 集合
    requests.add(request);
    // 然后判断是否暂停状态
    if (!isPaused) {
      // 不是暂停状态则开始请求
      // request为一个接口
      // 前面我们分析到不加载缩略图Request的实现为SingleRequest
      // 这里去看SingleRequest的begin方法
      request.begin();
    } else {
      // 是则清空请求
      request.clear();
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Paused, delaying request");
      }
      // 并将该请求加入待处理的请求集合中
      pendingRequests.add(request);
    }
  }  
  
}
```

代码如上所示。

首先将请求加入一个 Set 集合；然后判断是否暂停状态，是则清空请求，并将该请求加入待处理的请求集合中；不是暂停状态，则开始请求。

重点是`request.begin()`，我们上面分析过，Request的实现为SingleRequest，所以我们去看`SingleRequest#begin()`方法。

```java
  @Override
  public void begin() {
    synchronized (requestLock) {
      assertNotCallingCallbacks();
      stateVerifier.throwIfRecycled();
      startTime = LogTime.getLogTime();
      // 如果加载的资源为空，则调用加载失败的回调
      if (model == null) {
        if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
          width = overrideWidth;
          height = overrideHeight;
        }

        int logLevel = getFallbackDrawable() == null ? Log.WARN : Log.DEBUG;
        onLoadFailed(new GlideException("Received null model"), logLevel);
        return;
      }

      if (status == Status.RUNNING) {
        throw new IllegalArgumentException("Cannot restart a running request");
      }

      if (status == Status.COMPLETE) {
        // 资源加载完成
        onResourceReady(
            resource, DataSource.MEMORY_CACHE, /* isLoadedFromAlternateCacheKey= */ false);
        return;
      }

    status = Status.WAITING_FOR_SIZE;
      // 这里做了一个判断，如果我们设置了overrideWidth 和 overrideHeight（如下设置），
      // 则直接调用 onSizeReady() 方法，否则调用 getSize() 方法（第一次加载才会调用）。
      // 设置加载图片的宽高为 100x100 px
      // Glide.with(this).load(url).override(100,100).into(imageView);
      if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        onSizeReady(overrideWidth, overrideHeight);
      } else {
        // getSize() 方法的作用是通过 ViewTreeObserver 来监听 ImageView 的宽高
        // 拿到宽高后最终也是调用 onSizeReady() 方法。
        target.getSize(this);
      }

      if ((status == Status.RUNNING || status == Status.WAITING_FOR_SIZE)
          && canNotifyStatusChanged()) {
        // 刚开始加载的回调
        target.onLoadStarted(getPlaceholderDrawable());
      }
      if (IS_VERBOSE_LOGGABLE) {
        logV("finished run method in " + LogTime.getElapsedMillis(startTime));
      }
    }
  }

```

这里代码比较多。当前我们关注的第一个关键代码如下：

```java
      if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
        onSizeReady(overrideWidth, overrideHeight);
      } else {
        target.getSize(this);
      }
```

这里做了一个判断，如果我们设置了 overrideWidth 和 overrideHeight（如下代码所示），则直接调用 `onSizeReady()` 方法，否则调用 `getSize()` 方法（第一次加载才会调用）。

```java
// 设置加载图片的宽高为 100x100 px
Glide.with(this).load(url).override(100,100).into(imageView);
```

`getSize()` 方法的作用是通过 ViewTreeObserver 来监听 ImageView 的宽高，拿到宽高后最终也是调用 `onSizeReady()` 方法。

```java
//Target
public interface Target<R> extends LifecycleListener {
  ...
  void getSize(@NonNull SizeReadyCallback cb);
  ...
}

//SizeReadyCallback
public interface SizeReadyCallback {
  void onSizeReady(int width, int height);
}
```

接下来我们看 `onSizeReady()` 方法。

```java
  @Override
  public void onSizeReady(int width, int height) {
    stateVerifier.throwIfRecycled();
    synchronized (requestLock) {
      if (IS_VERBOSE_LOGGABLE) {
        logV("Got onSizeReady in " + LogTime.getElapsedMillis(startTime));
      }
      if (status != Status.WAITING_FOR_SIZE) {
        return;
      }
      status = Status.RUNNING;
      // 根据缩略比例获取图片宽高
      float sizeMultiplier = requestOptions.getSizeMultiplier();
      this.width = maybeApplySizeMultiplier(width, sizeMultiplier);
      this.height = maybeApplySizeMultiplier(height, sizeMultiplier);

      if (IS_VERBOSE_LOGGABLE) {
        logV("finished setup for calling load in " + LogTime.getElapsedMillis(startTime));
      }
      // 开始加载
      loadStatus =
          engine.load(
              glideContext,
              model,
              requestOptions.getSignature(),
              this.width,
              this.height,
              requestOptions.getResourceClass(),
              transcodeClass,
              priority,
              requestOptions.getDiskCacheStrategy(),
              requestOptions.getTransformations(),
              requestOptions.isTransformationRequired(),
              requestOptions.isScaleOnlyOrNoTransform(),
              requestOptions.getOptions(),
              requestOptions.isMemoryCacheable(),
              requestOptions.getUseUnlimitedSourceGeneratorsPool(),
              requestOptions.getUseAnimationPool(),
              requestOptions.getOnlyRetrieveFromCache(),
              this,
              callbackExecutor);

      if (status != Status.RUNNING) {
        loadStatus = null;
      }
      if (IS_VERBOSE_LOGGABLE) {
        logV("finished onSizeReady in " + LogTime.getElapsedMillis(startTime));
      }
    }
  }
```

这里着重关注`engine.load()`方法，可以猜测为正在加载的部分。

`Engine#load()`如下：

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
      // 内存中没有，则等待当前正在执行的或开始一个新的 EngineJob
      if (memoryResource == null) {
        return waitForExistingOrStartNewJob(
            glideContext,
            model,
            signature,
            width,
            height,
            resourceClass,
            transcodeClass,
            priority,
            diskCacheStrategy,
            transformations,
            isTransformationRequired,
            isScaleOnlyOrNoTransform,
            options,
            isMemoryCacheable,
            useUnlimitedSourceExecutorPool,
            useAnimationPool,
            onlyRetrieveFromCache,
            cb,
            callbackExecutor,
            key,
            startTime);
      }
    }

    // 加载完成回调
    cb.onResourceReady(
        memoryResource, DataSource.MEMORY_CACHE, /* isLoadedFromAlternateCacheKey= */ false);
    return null;
  }
```

load分为几步：

构建缓存 key，然后利用这个 key 获取缓存中的资源；如果内存中没有，则等待当前正在执行的或开始一个新的 EngineJob；内存中有则调用加载完成的回调。

我们先不关注缓存，分析主要流程，也就是在缓存中查找不到数据，所以执行`waitForExistingOrStartNewJob()`。

```java
private <R> LoadStatus waitForExistingOrStartNewJob(
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
    Executor callbackExecutor,
    EngineKey key,
    long startTime) {

  // 从 map 集合中获取 EngineJob，如果不为空表示当前有正在执行的 EngineJob，添加回调并返回加载状态
  EngineJob<?> current = jobs.get(key, onlyRetrieveFromCache);
  if (current != null) {
    current.addCallback(cb, callbackExecutor);
    if (VERBOSE_IS_LOGGABLE) {
      logWithTimeAndKey("Added to existing load", startTime, key);
    }
    return new LoadStatus(cb, current);
  }

  // 使用 EngineJob 工厂构建了一个 EngineJob，该类主要用来管理加载以及当加载完成时通知回调
  EngineJob<R> engineJob =
      engineJobFactory.build(
          key,
          isMemoryCacheable,
          useUnlimitedSourceExecutorPool,
          useAnimationPool,
          onlyRetrieveFromCache);

  // 使用 DecodeJob 工厂构建了一个 DecodeJob，该类主要负责图片的解码，实现了 Runnable 接口，属于一个任务
  DecodeJob<R> decodeJob =
      decodeJobFactory.build(
          glideContext,
          model,
          key,
          signature,
          width,
          height,
          resourceClass,
          transcodeClass,
          priority,
          diskCacheStrategy,
          transformations,
          isTransformationRequired,
          isScaleOnlyOrNoTransform,
          onlyRetrieveFromCache,
          options,
          engineJob);

  // 将 EngineJob 放到一个 map 集合中
  jobs.put(key, engineJob);

  // 添加回调
  engineJob.addCallback(cb, callbackExecutor);

  // 这里调用了 EngineJob#start()
  engineJob.start(decodeJob);

  if (VERBOSE_IS_LOGGABLE) {
    logWithTimeAndKey("Started new load", startTime, key);
  }
  // 返回加载状态
  return new LoadStatus(cb, engineJob);
}
```

这里做了以下几件事：

- 从 map 集合中获取 EngineJob，如果不为空表示当前有正在执行的 EngineJob，添加回调并返回加载状态。

- 使用 EngineJob 工厂构建了一个 EngineJob，该类主要用来管理加载以及当加载完成时通知回调。

- 使用 DecodeJob 工厂构建了一个 DecodeJob，该类主要负责图片的解码，实现了 Runnable 接口，属于一个任务。

- 这里调用了 `EngineJob#start()`

这里我们主要关注`EngineJob#start()`过程。

```java
  public synchronized void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    // 直接将 DecodeJob 任务放到了一个线程池中去执行，也就是说从这里开始切换到子线程了
    GlideExecutor executor =
        decodeJob.willDecodeFromCache() ? diskCacheExecutor : getActiveSourceExecutor();
    //关注 DecodeJob 的 run() 方法
    executor.execute(decodeJob);
  }
```

可以看到，这里是将DecodeJob放到一个线程池中执行。

```java
class DecodeJob<R>
    implements DataFetcherGenerator.FetcherReadyCallback,
        Runnable,
        Comparable<DecodeJob<?>>,
        Poolable {

  @Override
  public void run() {

    GlideTrace.beginSectionFormat("DecodeJob#run(model=%s)", model);
    DataFetcher<?> localFetcher = currentFetcher;
    try {
      // 被取消，则调用加载失败的回调
      if (isCancelled) {
        notifyFailed();
        return;
      }
      // 执行
      runWrapped();
    } catch (CallbackException e) {
      throw e;
    } catch (Throwable t) {
      if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG,"...");
      }
      if (stage != Stage.ENCODE) {
        throwables.add(t);
        notifyFailed();
      }
      if (!isCancelled) {
        throw t;
      }
      throw t;
    } finally {
      if (localFetcher != null) {
        localFetcher.cleanup();
      }
      GlideTrace.endSection();
    }
  }
            
}
```

真的执行是在` runWrapped()`方法中。

```java
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

runReason 的默认值为 INITIALIZE，所以走的是第一个 case。

由于我们配置了禁用内存与磁盘缓存，所以得到的 stage 为 SOURCE，currentGenerator 为 SourceGenerator。

拿到资源执行器接着就是执行了，调用了`runGenerators()` 方法。

接下来我们查看`runGenerators()` 方法

```java
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

    if ((stage == Stage.FINISHED || isCancelled) && !isStarted) {
      notifyFailed();
    }
  }
```

我们分析的currentGenerator 在这里为 SourceGenerator，所以调用 `startNext()` 方法的时候实际调用的是 `SourceGenerator#startNext()`。查看该方法后发现，执行完返回的结果为 true，所以 while 循环是进不去的。我们主要就是分析`SourceGenerator#startNext()`方法。

```java
  @Override
  public boolean startNext() {
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
      // 通过 helper#getLoadData() 获取 LoadData 集合，
      // 实际集合里面也就只有一个元素，然后通过索引获取一个 LoadData
      loadData = helper.getLoadData().get(loadDataListIndex++);
      if (loadData != null
          && (helper.getDiskCacheStrategy().isDataCacheable(loadData.fetcher.getDataSource())
              || helper.hasLoadPath(loadData.fetcher.getDataClass()))) {
        started = true;
        // 这里表示开始加载了
        startNextLoad(loadData);
      }
    }
    return started;
  }
```

这里我们先看`helper.getLoadData().get()`，再看`SourceGenerator#startNextLoad(loadData)`。

`helper.getLoadData().get()`：

`helper#getLoadData()` 获取 LoadData 集合，实际集合里面也就只有一个元素，然后通过索引获取一个 LoadData。

```java
  List<LoadData<?>> getLoadData() {
    if (!isLoadDataSet) {
      isLoadDataSet = true;
      loadData.clear();
      List<ModelLoader<Object, ?>> modelLoaders = glideContext.getRegistry().getModelLoaders(model);

      for (int i = 0, size = modelLoaders.size(); i < size; i++) {
        ModelLoader<Object, ?> modelLoader = modelLoaders.get(i);
        // 通过 ModelLoader 对象的 buildLoadData() 方法获取的 LoadData
        // 从网络加载数据，这里实际调用的是
        // com.bumptech.glide.load.model.stream HttpGlideUrlLoader#buildLoadData()
        LoadData<?> current = modelLoader.buildLoadData(model, width, height, options);
        if (current != null) {
          loadData.add(current);
        }
      }
    }
    return loadData;
  }
```

可以看到，这里是通过 ModelLoader 对象的`buildLoadData()` 方法获取的 LoadData。

我们分析的是从网络加载数据，所以这里实际调用的是 `HttpGlideUrlLoader#buildLoadData()`，代码如下所示：

```java
  @Override
  public LoadData<InputStream> buildLoadData(
      @NonNull GlideUrl model, int width, int height, @NonNull Options options) {
    GlideUrl url = model;
    if (modelCache != null) {
      url = modelCache.get(model, 0, 0);
      if (url == null) {
        modelCache.put(model, 0, 0, model);
        url = model;
      }
    }
    int timeout = options.get(TIMEOUT);
    // 实例化 LoadData 的时候顺带实例化了 HttpUrlFetcher
    return new LoadData<>(url, new HttpUrlFetcher(url, timeout));
  }
```

最后一行实例化 LoadData 的时候顺带实例化了 HttpUrlFetcher。

接下来我们分析`SourceGenerator#startNextLoad(loadData)`：

```java
  private void startNextLoad(final LoadData<?> toStart) {
    // loadData.fetcher 就是刚刚实例化 LoadData 的时候传进来的 HttpUrlFetcher
    // 这里调用的是 HttpUrlFetcher#loadData()
    loadData.fetcher.loadData(
        helper.getPriority(),
        //这里的回调需要留心一下，后面会用到
        new DataCallback<Object>() {
          @Override
          public void onDataReady(@Nullable Object data) {
            if (isCurrentRequest(toStart)) {
              onDataReadyInternal(toStart, data);
            }
          }

          @Override
          public void onLoadFailed(@NonNull Exception e) {
            if (isCurrentRequest(toStart)) {
              onLoadFailedInternal(toStart, e);
            }
          }
        });
  }
```

这里的 `loadData.fetcher` 就是刚刚实例化 LoadData 的时候传进来的 HttpUrlFetcher，所以这里调用的是 `HttpUrlFetcher#loadData()`：

```java
  @Override
  public void loadData(
      @NonNull Priority priority, @NonNull DataCallback<? super InputStream> callback) {
    long startTime = LogTime.getLogTime();
    try {
      // 通过重定向加载数据
      InputStream result = loadDataWithRedirects(glideUrl.toURL(), 0, null, glideUrl.getHeaders());
      // 加载成功，回调数据
      // 我们现在关注的实现在SourceGenerator
      callback.onDataReady(result);
    } catch (IOException e) {
      if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG, "Failed to load data for url", e);
      }
      callback.onLoadFailed(e);
    } finally {
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Finished http url fetcher fetch in " + LogTime.getElapsedMillis(startTime));
      }
    }
  }
```

这里我们需要关注2个点：`loadDataWithRedirects()`和`callback.onDataReady()`

`loadDataWithRedirects()`在加载完数据后返回了 InputStream：

```java
  private InputStream loadDataWithRedirects(
      URL url, int redirects, URL lastUrl, Map<String, String> headers) throws HttpException {
    if (redirects >= MAXIMUM_REDIRECTS) {
      throw new HttpException(
          "Too many (> " + MAXIMUM_REDIRECTS + ") redirects!", INVALID_STATUS_CODE);
    } else {
      try {
        if (lastUrl != null && url.toURI().equals(lastUrl.toURI())) {
          throw new HttpException("In re-direct loop", INVALID_STATUS_CODE);
        }
      } catch (URISyntaxException e) {
      }
    }

    // 获取 HttpURLConnection 实例
    urlConnection = buildAndConfigureConnection(url, headers);

    try {
      urlConnection.connect();
      stream = urlConnection.getInputStream();
    } catch (IOException e) {
      throw new HttpException(
          "Failed to connect or obtain data", getHttpStatusCodeOrInvalid(urlConnection), e);
    }

    if (isCancelled) {
      return null;
    }

    final int statusCode = getHttpStatusCodeOrInvalid(urlConnection);
    if (isHttpOk(statusCode)) {
      // 请求成功
      // 获取 InputStream
      return getStreamForSuccessfulRequest(urlConnection);
    } else if (isHttpRedirect(statusCode)) {
      // 重定向请求
      String redirectUrlString = urlConnection.getHeaderField(REDIRECT_HEADER_FIELD);
      if (TextUtils.isEmpty(redirectUrlString)) {
        throw new HttpException("Received empty or null redirect url", statusCode);
      }
      URL redirectUrl;
      try {
        redirectUrl = new URL(url, redirectUrlString);
      } catch (MalformedURLException e) {
        throw new HttpException("Bad redirect url: " + redirectUrlString, statusCode, e);
      }
      cleanup();
      //递归调用
      return loadDataWithRedirects(redirectUrl, redirects + 1, url, headers);
    } else if (statusCode == INVALID_STATUS_CODE) {
      throw new HttpException(statusCode);
    } else {
      try {
        throw new HttpException(urlConnection.getResponseMessage(), statusCode);
      } catch (IOException e) {
        throw new HttpException("Failed to get a response message", statusCode, e);
      }
    }
  }
```

改部分主要是进行网络请求，可以看到 Glide 底层是采用 HttpURLConnection 来进行网络请求的，请求成功后返回了 InputStream。

然后我们看`callback.onDataReady`，将 InputStream 回调返回，回顾一下`startNextLoad()`方法，传入的回调使用了匿名内部类，在`onDataReady()`方法中调用了`onDataReadyInternal(toStart, data)`方法。

```java
  private void startNextLoad(final LoadData<?> toStart) {
    // loadData.fetcher 就是刚刚实例化 LoadData 的时候传进来的 HttpUrlFetcher
    // 这里调用的是 HttpUrlFetcher#loadData()
    loadData.fetcher.loadData(
        helper.getPriority(),
        //这里的回调需要留心一下，后面会用到
        new DataCallback<Object>() {
          @Override
          public void onDataReady(@Nullable Object data) {
            if (isCurrentRequest(toStart)) {
              onDataReadyInternal(toStart, data);
            }
          }

          @Override
          public void onLoadFailed(@NonNull Exception e) {
            if (isCurrentRequest(toStart)) {
              onLoadFailedInternal(toStart, e);
            }
          }
        });
  }

```

接下来我们关注`onDataReadyInternal(toStart, data)`：

```java
private final FetcherReadyCallback cb;

@Synthetic
  void onDataReadyInternal(LoadData<?> loadData, Object data) {
    DiskCacheStrategy diskCacheStrategy = helper.getDiskCacheStrategy();
    if (data != null && diskCacheStrategy.isDataCacheable(loadData.fetcher.getDataSource())) {
      dataToCache = data;
      cb.reschedule();
    } else {
      // 执行这里
      // 实现在DecodeJob
      cb.onDataFetcherReady(
          loadData.sourceKey,
          data,
          loadData.fetcher,
          loadData.fetcher.getDataSource(),
          originalKey);
    }
  }

//DecodeJob
  @Override
  public void onDataFetcherReady(
      Key sourceKey, Object data, DataFetcher<?> fetcher, DataSource dataSource, Key attemptedKey) {
    this.currentSourceKey = sourceKey;
    this.currentData = data;
    this.currentFetcher = fetcher;
    this.currentDataSource = dataSource;
    this.currentAttemptingKey = attemptedKey;
    this.isLoadingFromAlternateCacheKey = sourceKey != decodeHelper.getCacheKeys().get(0);

    if (Thread.currentThread() != currentThread) {
      runReason = RunReason.DECODE_DATA;
      callback.reschedule(this);
    } else {
      GlideTrace.beginSection("DecodeJob.decodeFromRetrievedData");
      try {
        // 解码
        decodeFromRetrievedData();
      } finally {
        GlideTrace.endSection();
      }
    }
  }
```

如上所示，经过一系列调用，最终调用`decodeFromRetrievedData()`进行解码，实现如下：

```java
  private void decodeFromRetrievedData() {
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      logWithTimeAndKey(
          "Retrieved data",
          startFetchTime,
          "data: "
              + currentData
              + ", cache key: "
              + currentSourceKey
              + ", fetcher: "
              + currentFetcher);
    }
    Resource<R> resource = null;
    try {
      //解码
      resource = decodeFromData(currentFetcher, currentData, currentDataSource);
    } catch (GlideException e) {
      e.setLoggingDetails(currentAttemptingKey, currentDataSource);
      throwables.add(e);
    }
    if (resource != null) {
      //解码完成，通知下去
      notifyEncodeAndRelease(resource, currentDataSource, isLoadingFromAlternateCacheKey);
    } else {
      runGenerators();
    }
  }
```

这里有2个主要关注的部分：`decodeFromData()` 和`notifyEncodeAndRelease()`

`decodeFromData()` 

```java
 private <Data> Resource<R> decodeFromData(
      DataFetcher<?> fetcher, Data data, DataSource dataSource) throws GlideException {
    try {
      if (data == null) {
        return null;
      }
      long startTime = LogTime.getLogTime();
      Resource<R> result = decodeFromFetcher(data, dataSource);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Decoded result " + result, startTime);
      }
      return result;
    } finally {
      fetcher.cleanup();
    }
  }
  
    private <Data> Resource<R> decodeFromFetcher(Data data, DataSource dataSource)
      throws GlideException {
    // 获取解码器，解码器里面封装了 DecodePath，它就是用来解码转码的
    LoadPath<Data, ?, R> path = decodeHelper.getLoadPath((Class<Data>) data.getClass());
    // 通过解码器解析数据
    return runLoadPath(data, dataSource, path);
  }


  private <Data, ResourceType> Resource<R> runLoadPath(
      Data data, DataSource dataSource, LoadPath<Data, ResourceType, R> path)
      throws GlideException {
    Options options = getOptionsWithHardwareConfig(dataSource);
    DataRewinder<Data> rewinder = glideContext.getRegistry().getRewinder(data);
    try {
      // ResourceType in DecodeCallback below is required for compilation to work with gradle.
      // 将解码任务传递给 LoadPath 完成
      return path.load(
          rewinder, options, width, height, new DecodeCallback<ResourceType>(dataSource));
    } finally {
      rewinder.cleanup();
    }
  }

//LoadPath
  public Resource<Transcode> load(
      DataRewinder<Data> rewinder,
      @NonNull Options options,
      int width,
      int height,
      DecodePath.DecodeCallback<ResourceType> decodeCallback)
      throws GlideException {
    List<Throwable> throwables = Preconditions.checkNotNull(listPool.acquire());
    try {
      return loadWithExceptionList(rewinder, options, width, height, decodeCallback, throwables);
    } finally {
      listPool.release(throwables);
    }
  }

  private Resource<Transcode> loadWithExceptionList(
      DataRewinder<Data> rewinder,
      @NonNull Options options,
      int width,
      int height,
      DecodePath.DecodeCallback<ResourceType> decodeCallback,
      List<Throwable> exceptions)
      throws GlideException {
    Resource<Transcode> result = null;
    //noinspection ForLoopReplaceableByForEach to improve perf
    for (int i = 0, size = decodePaths.size(); i < size; i++) {
      DecodePath<Data, ResourceType, Transcode> path = decodePaths.get(i);
      try {
        // 开始解析数据
        result = path.decode(rewinder, width, height, options, decodeCallback);
      } catch (GlideException e) {
        exceptions.add(e);
      }
      if (result != null) {
        break;
      }
    }

    if (result == null) {
      throw new GlideException(failureMessage, new ArrayList<>(exceptions));
    }

    return result;
  }

```

可以看到，上面主要是获取解码器（LoadPath），然后解码器里面是封装了 DecodePath，经过一系列调用，最后其实是交给 DecodePath 的 `decode()` 方法来真正开始解析数据的。

`decode()` 方法如下：

```java
  public Resource<Transcode> decode(
      DataRewinder<DataType> rewinder,
      int width,
      int height,
      @NonNull Options options,
      DecodeCallback<ResourceType> callback)
      throws GlideException {
    // 这里是一个解码过程，主要是将原始数据解码成原始图片
    Resource<ResourceType> decoded = decodeResource(rewinder, width, height, options);
    // 这里会调用 callback#onResourceDecoded() 进行回调，这里是回调到 DecodeJob#onResourceDecoded()
    Resource<ResourceType> transformed = callback.onResourceDecoded(decoded);
    // transcoder 为 BitmapDrawableTranscoder，所以这里调用的是 BitmapDrawableTranscoder#transcode()
    return transcoder.transcode(transformed, options);
  }

```

这里分为三个部分：

- `decodeResource()`：

```java
@NonNull
private Resource<ResourceType> decodeResource(
    DataRewinder<DataType> rewinder, int width, int height, @NonNull Options options)
    throws GlideException {
  List<Throwable> exceptions = Preconditions.checkNotNull(listPool.acquire());
  try {
    return decodeResourceWithList(rewinder, width, height, options, exceptions);
  } finally {
    listPool.release(exceptions);
  }
}
  @NonNull
  private Resource<ResourceType> decodeResourceWithList(
      DataRewinder<DataType> rewinder,
      int width,
      int height,
      @NonNull Options options,
      List<Throwable> exceptions)
      throws GlideException {
    Resource<ResourceType> result = null;
    //noinspection ForLoopReplaceableByForEach to improve perf
    for (int i = 0, size = decoders.size(); i < size; i++) {
      ResourceDecoder<DataType, ResourceType> decoder = decoders.get(i);
      try {
        DataType data = rewinder.rewindAndGet();
        if (decoder.handles(data, options)) {
          data = rewinder.rewindAndGet();
          //调用的是 StreamBitmapDecoder#decode()
          result = decoder.decode(data, width, height, options);
        }
      } catch (IOException | RuntimeException | OutOfMemoryError e) {
        if (Log.isLoggable(TAG, Log.VERBOSE)) {
          Log.v(TAG, "Failed to decode data for " + decoder, e);
        }
        exceptions.add(e);
      }

      if (result != null) {
        break;
      }
    }

    if (result == null) {
      throw new GlideException(failureMessage, new ArrayList<>(exceptions));
    }
    return result;
  }
```

可以看到，这里遍历拿到可以解码当前数据的资源解码器，然后调用 decode() 方法进行解码。

因为当前数据是 InputStream，所以这里遍历拿到的 ResourceDecoder 其实是 StreamBitmapDecoder，所以调用的是 `StreamBitmapDecoder#decode()`。

```java

  @Override
  public Resource<Bitmap> decode(
      @NonNull InputStream source, int width, int height, @NonNull Options options)
      throws IOException {

 
    final RecyclableBufferedInputStream bufferedStream;
    final boolean ownsBufferedStream;
    if (source instanceof RecyclableBufferedInputStream) {
      bufferedStream = (RecyclableBufferedInputStream) source;
      ownsBufferedStream = false;
    } else {
      bufferedStream = new RecyclableBufferedInputStream(source, byteArrayPool);
      ownsBufferedStream = true;
    }

    ExceptionCatchingInputStream exceptionStream =
        ExceptionCatchingInputStream.obtain(bufferedStream);

    MarkEnforcingInputStream invalidatingStream = new MarkEnforcingInputStream(exceptionStream);
    UntrustedCallbacks callbacks = new UntrustedCallbacks(bufferedStream, exceptionStream);
    try {
      //Downsampler#decode
      return downsampler.decode(invalidatingStream, width, height, options, callbacks);
    } finally {
      exceptionStream.release();
      if (ownsBufferedStream) {
        bufferedStream.release();
      }
    }
  }


  public Resource<Bitmap> decode(
      InputStream is,
      int requestedWidth,
      int requestedHeight,
      Options options,
      DecodeCallbacks callbacks)
      throws IOException {
    return decode(
        new ImageReader.InputStreamImageReader(is, parsers, byteArrayPool),
        requestedWidth,
        requestedHeight,
        options,
        callbacks);
  }

  private Resource<Bitmap> decode(
      ImageReader imageReader,
      int requestedWidth,
      int requestedHeight,
      Options options,
      DecodeCallbacks callbacks)
      throws IOException {
    ...

    try {
      //根据输入流解码得到 Bitmap
      Bitmap result =
          decodeFromWrappedStreams(
              imageReader,
              bitmapFactoryOptions,
              downsampleStrategy,
              decodeFormat,
              preferredColorSpace,
              isHardwareConfigAllowed,
              requestedWidth,
              requestedHeight,
              fixBitmapToRequestedDimensions,
              callbacks);
      //将 Bitmap 包装成 Resource<Bitmap> 返回
      return BitmapResource.obtain(result, bitmapPool);
    } finally {
      releaseOptions(bitmapFactoryOptions);
      byteArrayPool.put(bytesForOptions);
    }
  }


```

流程如上，调用了 `Downsampler#decode()`，然后将输入流解码得到 Bitmap，最后将 Bitmap 包装成 Resource 返回。

- `callback.onResourceDecoded()`：

这里会调用 `callback#onResourceDecoded()` 进行回调，这里是回调到 `DecodeJob#onResourceDecoded()`：

```java
  private final class DecodeCallback<Z> implements DecodePath.DecodeCallback<Z> {

    private final DataSource dataSource;

    @Synthetic
    DecodeCallback(DataSource dataSource) {
      this.dataSource = dataSource;
    }

    @NonNull
    @Override
    public Resource<Z> onResourceDecoded(@NonNull Resource<Z> decoded) {
      return DecodeJob.this.onResourceDecoded(dataSource, decoded);
    }
  }

```

继续：

```java
  @Synthetic
  @NonNull
  <Z> Resource<Z> onResourceDecoded(DataSource dataSource, @NonNull Resource<Z> decoded) {
    @SuppressWarnings("unchecked")
    Class<Z> resourceSubClass = (Class<Z>) decoded.get().getClass();
    Transformation<Z> appliedTransformation = null;
    Resource<Z> transformed = decoded;
    // 如果不是从磁盘缓存中获取的，则需要对资源进行转换
    if (dataSource != DataSource.RESOURCE_DISK_CACHE) {
      appliedTransformation = decodeHelper.getTransformation(resourceSubClass);
      transformed = appliedTransformation.transform(glideContext, decoded, width, height);
    }
    // TODO: Make this the responsibility of the Transformation.
    if (!decoded.equals(transformed)) {
      decoded.recycle();
    }

    final EncodeStrategy encodeStrategy;
    final ResourceEncoder<Z> encoder;
    if (decodeHelper.isResourceEncoderAvailable(transformed)) {
      encoder = decodeHelper.getResultEncoder(transformed);
      encodeStrategy = encoder.getEncodeStrategy(options);
    } else {
      encoder = null;
      encodeStrategy = EncodeStrategy.NONE;
    }
    // 缓存相关
    ....
    
    return result;
  }
```

可以看到，这里的任务是对资源进行转换，也就是我们请求的时候如果配置了 centerCrop、fitCenter 等，这里就需要转换成对应的资源。

- `transcoder.transcode()`:

`transcoder.transcode(transformed, options) `中的 transcoder 为 BitmapDrawableTranscoder，所以这里调用的是 `BitmapDrawableTranscoder#transcode()`：

```java
  public Resource<BitmapDrawable> transcode(
      @NonNull Resource<Bitmap> toTranscode, @NonNull Options options) {
    return LazyBitmapDrawableResource.obtain(resources, toTranscode);
  }

  @Nullable
  public static Resource<BitmapDrawable> obtain(
      @NonNull Resources resources, @Nullable Resource<Bitmap> bitmapResource) {
    if (bitmapResource == null) {
      return null;
    }
    return new LazyBitmapDrawableResource(resources, bitmapResource);
  }

  private LazyBitmapDrawableResource(
      @NonNull Resources resources, @NonNull Resource<Bitmap> bitmapResource) {
    this.resources = Preconditions.checkNotNull(resources);
    this.bitmapResource = Preconditions.checkNotNull(bitmapResource);
  }

```

因为 LazyBitmapDrawableResource 实现了 Resource，所以最后是返回了一个 Resource，也就是说转码这一步是将 Bitmap 转成了 Resource。

`DecodeJob#decodeFromRetrievedData() `中的解码的流程就走完了。

接下来我们回来看`notifyEncodeAndRelease()`方法：

```java
  //DecodeJob
  private void decodeFromRetrievedData() {
    ...
    Resource<R> resource = null;
    try {
      //解码
      resource = decodeFromData(currentFetcher, currentData, currentDataSource);
    } catch (GlideException e) {
      e.setLoggingDetails(currentAttemptingKey, currentDataSource);
      throwables.add(e);
    }
    if (resource != null) {
      //解码完成，通知下去
      notifyEncodeAndRelease(resource, currentDataSource, isLoadingFromAlternateCacheKey);
    } else {
      runGenerators();
    }
  }

  private void notifyEncodeAndRelease(
      Resource<R> resource, DataSource dataSource, boolean isLoadedFromAlternateCacheKey) {
    if (resource instanceof Initializable) {
      ((Initializable) resource).initialize();
    }

    Resource<R> result = resource;
    LockedResource<R> lockedResource = null;
    if (deferredEncodeManager.hasResourceToEncode()) {
      lockedResource = LockedResource.obtain(resource);
      result = lockedResource;
    }
    // 通知外面已经完成了
    notifyComplete(result, dataSource, isLoadedFromAlternateCacheKey);

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

我们需要着重关注的是`notifyComplete()`。

```java
  private void notifyComplete(
      Resource<R> resource, DataSource dataSource, boolean isLoadedFromAlternateCacheKey) {
    setNotifiedOrThrow();
    // 通知结果回调
    callback.onResourceReady(resource, dataSource, isLoadedFromAlternateCacheKey);
  }
```

这里的 callback 只有一个实现类就是 EngineJob，所以直接看 `EngineJob#onResourceReady()`：

```java
  @Override
  public void onResourceReady(
      Resource<R> resource, DataSource dataSource, boolean isLoadedFromAlternateCacheKey) {
    synchronized (this) {
      this.resource = resource;
      this.dataSource = dataSource;
      this.isLoadedFromAlternateCacheKey = isLoadedFromAlternateCacheKey;
    }
    // 通知结果回调
    notifyCallbacksOfResult();
  }

  @Synthetic
  void notifyCallbacksOfResult() {
    ResourceCallbacksAndExecutors copy;
    Key localKey;
    EngineResource<?> localResource;
    synchronized (this) {
      stateVerifier.throwIfRecycled();
      // 如果取消，则回收和释放资源
      if (isCancelled) {
        resource.recycle();
        release();
        return;
      } else if (cbs.isEmpty()) {
        throw new IllegalStateException("Received a resource without any callbacks to notify");
      } else if (hasResource) {
        throw new IllegalStateException("Already have resource");
      }
      engineResource = engineResourceFactory.build(resource, isCacheable, key, resourceListener);
        
      hasResource = true;
      copy = cbs.copy();
      incrementPendingCallbacks(copy.size() + 1);

      localKey = key;
      localResource = engineResource;
    }

    //这里表示 EngineJob 完成了，回调给 Engine
    engineJobListener.onEngineJobComplete(this, localKey, localResource);

    //遍历 copy 后拿到了线程池的执行器（entry.executor）
    for (final ResourceCallbackAndExecutor entry : copy) {
      entry.executor.execute(new CallResourceReady(entry.cb));
    }
    decrementPendingCallbacks();
  }

```

这里我们关注2个地方：`engineJobListener.onEngineJobComplete`和`entry.executor.execute`相关部分。

`engineJobListener.onEngineJobComplete`，表示 EngineJob 完成了，回调给 Engine，进入 onEngineJobComplete() 

```java
 //Engine
  @Override
  public synchronized void onEngineJobComplete(
      EngineJob<?> engineJob, Key key, EngineResource<?> resource) {
    // A null resource indicates that the load failed, usually due to an exception.
    if (resource != null && resource.isMemoryCacheable()) {
      activeResources.activate(key, resource);
    }

    jobs.removeIfCurrent(key, engineJob);
  } 

  synchronized void activate(Key key, EngineResource<?> resource) {
    ResourceWeakReference toPut =
        new ResourceWeakReference(
            key, resource, resourceReferenceQueue, isActiveResourceRetentionAllowed);

    ResourceWeakReference removed = activeEngineResources.put(key, toPut);
    if (removed != null) {
      removed.reset();
    }
  }
```

这里主要判断是否配置了内存缓存，有的话就存到缓存中。

`entry.executor.execute`相关部分：

这里遍历 copy 后拿到了线程池的执行器（entry.executor），copy 是上面 `copy = cbs.copy();`

反推代码：

```java
    // 1. cbs.copy()
    /*ResourceCallbacksAndExecutors*/
    ResourceCallbacksAndExecutors copy() {
      return new ResourceCallbacksAndExecutors(new ArrayList<>(callbacksAndExecutors));
    }

    // 2. callbacksAndExecutors 集合是哪里添加元素的
    /*ResourceCallbacksAndExecutors*/
    void add(ResourceCallback cb, Executor executor) {
      callbacksAndExecutors.add(new ResourceCallbackAndExecutor(cb, executor));
    }

  // 3. 上面的 add() 方法是哪里调用的
  /*EngineJob*/
  synchronized void addCallback(final ResourceCallback cb, Executor callbackExecutor) {
    cbs.add(cb, callbackExecutor);
  }

  // 4. 上面的 addCallback() 方法是哪里调用的
  /*Engine*/
  private <R> LoadStatus waitForExistingOrStartNewJob(
	  ...
      Executor callbackExecutor,
      ...
	  ) {

    engineJob.addCallback(cb, callbackExecutor);
    return new LoadStatus(cb, engineJob);
  }

```

发现是从 `waitForExistingOrStartNewJob()` 方法那里传进来的，最后发现是从 into() 方法哪里传过来的。

```java
  /*RequestBuilder*/
  @NonNull
  public <Y extends Target<TranscodeType>> Y into(@NonNull Y target) {
    return into(target, /*targetListener=*/ null, Executors.mainThreadExecutor());
  }
```

就是这里的 `Executors.mainThreadExecutor()`：

```java
public final class Executors {
  private Executors() {
    // Utility class.
  }

  private static final Executor MAIN_THREAD_EXECUTOR =
      new Executor() {
        @Override
        public void execute(@NonNull Runnable command) {
          Util.postOnUiThread(command);
        }
      };
  private static final Executor DIRECT_EXECUTOR =
      new Executor() {
        @Override
        public void execute(@NonNull Runnable command) {
          command.run();
        }
      };

  /** Posts executions to the main thread. */
  public static Executor mainThreadExecutor() {
    return MAIN_THREAD_EXECUTOR;
  }

  /** Immediately calls {@link Runnable#run()} on the current thread. */
  public static Executor directExecutor() {
    return DIRECT_EXECUTOR;
  }

  @VisibleForTesting
  public static void shutdownAndAwaitTermination(ExecutorService pool) {
    long shutdownSeconds = 5;
    pool.shutdownNow();
    try {
      if (!pool.awaitTermination(shutdownSeconds, TimeUnit.SECONDS)) {
        pool.shutdownNow();
        if (!pool.awaitTermination(shutdownSeconds, TimeUnit.SECONDS)) {
          throw new RuntimeException("Failed to shutdown");
        }
      }
    } catch (InterruptedException ie) {
      pool.shutdownNow();
      Thread.currentThread().interrupt();
      throw new RuntimeException(ie);
    }
  }
}

```

这里是创建了一个 Executor 的实例，然后重写了 `execute()` 方法，但是里面却用的是主线程中的 Handler 来 `post()` 一个任务，也就是**切换到了主线程**去执行了。

`entry.executor.execute(new CallResourceReady(entry.cb))`实际执行的是 `handler.post(command)`，CallResourceReady 中的 run() 方法是在主线程中执行的。

看一下 `CallResourceReady#run()`：

```java
  // EngineJob
  private class CallResourceReady implements Runnable {

    private final ResourceCallback cb;

    CallResourceReady(ResourceCallback cb) {
      this.cb = cb;
    }

    @Override
    public void run() {、
      synchronized (cb.getLock()) {
        synchronized (EngineJob.this) {
          if (cbs.contains(cb)) {
            // Acquire for this particular callback.
            engineResource.acquire();
            callCallbackOnResourceReady(cb);
            removeCallback(cb);
          }
          decrementPendingCallbacks();
        }
      }
    }
  }

  
  void callCallbackOnResourceReady(ResourceCallback cb) {
    try {
	  // 资源准备好了，回调给上一层
      cb.onResourceReady(engineResource, dataSource);
    } catch (Throwable t) {
      throw new CallbackException(t);
    }
  }

```

回调到 `SingleRequest#onResourceReady()`:

```java
  @Override
  public void onResourceReady(
      Resource<?> resource, DataSource dataSource, boolean isLoadedFromAlternateCacheKey) {
    stateVerifier.throwIfRecycled();
    Resource<?> toRelease = null;
    try {
      synchronized (requestLock) {
        loadStatus = null;
        if (resource == null) {
          GlideException exception =
              new GlideException( "...");
          onLoadFailed(exception);
          return;
        }

        Object received = resource.get();
        if (received == null || !transcodeClass.isAssignableFrom(received.getClass())) {
          toRelease = resource;
          this.resource = null;
          GlideException exception =
              new GlideException( "..."));
          onLoadFailed(exception);
          return;
        }

        if (!canSetResource()) {
          toRelease = resource;
          this.resource = null;
          // We can't put the status to complete before asking canSetResource().
          status = Status.COMPLETE;
          return;
        }
		 //重载方法
        onResourceReady(
            (Resource<R>) resource, (R) received, dataSource, isLoadedFromAlternateCacheKey);
      }
    } finally {
      if (toRelease != null) {
        engine.release(toRelease);
      }
    }
  }
```

前面是一些加载失败的判断，看下最后的 `onResourceReady()`：

```java
 private void onResourceReady(
      Resource<R> resource, R result, DataSource dataSource, boolean isAlternateCacheKey) {
    // We must call isFirstReadyResource before setting status.
    boolean isFirstResource = isFirstReadyResource();
    status = Status.COMPLETE;
    this.resource = resource;

    if (glideContext.getLogLevel() <= Log.DEBUG) {
      Log.d( GLIDE_TAG,  "...“);
    }

    isCallingCallbacks = true;
    try {
      boolean anyListenerHandledUpdatingTarget = false;
      if (requestListeners != null) {
        for (RequestListener<R> listener : requestListeners) {
          anyListenerHandledUpdatingTarget |=
              listener.onResourceReady(result, model, target, dataSource, isFirstResource);
        }
      }
      anyListenerHandledUpdatingTarget |=
          targetListener != null
              && targetListener.onResourceReady(result, model, target, dataSource, isFirstResource);

      if (!anyListenerHandledUpdatingTarget) {
        Transition<? super R> animation = animationFactory.build(dataSource, isFirstResource);
        // 回调给 ImageViewTarget
        target.onResourceReady(result, animation);
      }
    } finally {
      isCallingCallbacks = false;
    }
	 // 通知加载成功
    notifyLoadSuccess();
  }
```

这里的 target 为 ImageViewTarget。

`ImageViewTarget#onResourceReady() `的代码如下：

```java
  @Override
  public void onResourceReady(@NonNull Z resource, @Nullable Transition<? super Z> transition) {
    if (transition == null || !transition.transition(resource, this)) {
      setResourceInternal(resource);
    } else {
      maybeUpdateAnimatable(resource);
    }
  }

  private void setResourceInternal(@Nullable Z resource) {
    // Order matters here. Set the resource first to make sure that the Drawable has a valid and
    // non-null Callback before starting it.
    setResource(resource);
    maybeUpdateAnimatable(resource);
  }

```

在前面 `GlideContext#buildImageViewTarget()`中创建的 ImageViewTarget 的实现类是 DrawableImageViewTarget，所以这里调用的是 `DrawableImageViewTarget#setResource()`，代码如下：

```
  @Override
  protected void setResource(@Nullable Drawable resource) {
    view.setImageDrawable(resource);
  }
```

到这里终于将图片显示到我们的 ImageView 中了。

into() 方法也结束了。

#### 3.3 总结

into() 方法很复杂。主要做了发起网络请求、缓存数据、解码并显示图片。

大概流程为： 发起网络请求前先判断是否有内存缓存，有则直接从内存缓存那里获取数据进行显示，没有则判断是否有磁盘缓存；有磁盘缓存则直接从磁盘缓存那里获取数据进行显示，没有才发起网络请求；网络请求成功后将返回的数据存储到内存与磁盘缓存（如果有配置），最后将返回的输入流解码成 Drawable 显示在 ImageView 上。

## 参考资料

1. [Android 主流开源框架（六）Glide 的执行流程源码解析](https://juejin.cn/post/6844904152636604424#heading-11)

2. [Android主流三方库源码分析（三、深入理解Glide源码）](https://juejin.cn/post/6844904049595121672#heading-13)



---

**我的[学习笔记](https://github.com/zhangzhian/LearningNotes)，欢迎star和fork**

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)











