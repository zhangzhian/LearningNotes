# 开发者营地|LeakCanary：源码详解

## 一、简介

> 本文基于最新2.6版本

[LeakCanary](https://github.com/square/leakcanary) 是一个 Android 内存泄漏检测框架，2.0 之前与 2.0 之后有较大的改变，例如 2.0 之后使用 Kotlin 重写，分析 hprof 文件的工具由原来的 [HAHA](https://github.com/square/haha) 替换成 [Shark](https://square.github.io/leakcanary/shark/) 等。

LeakCanary自动检测以下对象的泄漏：

- 销毁的 Activity 实例

- 销毁的 Fragment 实例

- 销毁的 fragment View 实例

- 清除 ViewModel 实例

简单对比一下：

**2.0 之前**：（1）在 app 的 build.gradle 中添加依赖（2）在你的 Application 中进行初始化：

```java
public class MyApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
      return;
    }
    LeakCanary.install(this);
  }
}
```

**2.0 之后**：在 app 的 build.gradle 中添加如下依赖即可。

```groovy
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.6'
}
```

通过过滤Logcat中的LeakCanary标记，确认LeakCanary正在启动时运行：

```
D LeakCanary: LeakCanary is running and ready to detect leaks
```

## 二、源码分析

### 1. 初始化

在新版中只是添加如下依赖，并未在 Application 中进行初始化，那么是如何初始化的呢？

实际它是使用 ContentProvider 实现的，ContentProvider 的 onCreate 方法会在 Application 的 attachBaseContext 方法执行完之后执行。所以 LeakCanary 是利用了这个机制在 onCreate 方法中初始化的.

源码如下：

```kotlin
/*AppWatcherInstaller 位于 leakcanary-object-watcher-android 模块*/
internal sealed class AppWatcherInstaller : ContentProvider() {
  ...
  override fun onCreate(): Boolean {
    val application = context!!.applicationContext as Application
    // 这里初始化
    AppWatcher.manualInstall(application)
    return true
  }

  override fun query(...): Cursor? {
    return null
  }

  override fun getType(...): String? {
    return null
  }

  override fun insert(...): Uri? {
    return null
  }

  override fun delete(...): Int {
    return 0
  }

  override fun update(...): Int {
    return 0
  }
}

```

可以看到，源码中定义一个类实现 ContentProvider，仅仅只是进行了初始化，其他方法都没有做任何事。

这种方式免去了写一个 Application，然后在这里初始化的步骤，确实提高了用户体验，但是如果所有第三方库都这样做，势必会影响应用的启动速度。而 LeakCanary 只用在 debug 版本。所以，如果公司内部封装的一些只在 debug 版本使用的库，也可以参考这种做法。

### 2. InternalAppWatcher#install()

前面初始化的时候调用完 `AppWatcher#manualInstall()` 后，接着会调用 `InternalAppWatcher#install()`。看一下这个方法：

```kotlin
  /*InternalAppWatcher*/
  fun install(application: Application) {
    // 检查是否在主线程调用
    checkMainThread()
    // 是否已初始化
    if (this::application.isInitialized) {
      return
    }
    InternalAppWatcher.application = application
    if (isDebuggableBuild) {
      SharkLog.logger = DefaultCanaryLog()
    }
    // 获取配置，用在后面判断是否检测某种类型的泄漏
    val configProvider = { AppWatcher.config }
    // (1)
    ActivityDestroyWatcher.install(application, objectWatcher, configProvider)
    // (2)
    FragmentDestroyWatcher.install(application, objectWatcher, configProvider)
    // (3)
    onAppWatcherInstalled(application)
  }
```

源码中我标记了 3 个关注点，分别如下：

**（1）ActivityDestroyWatcher#install()**

```kotlin
    /*ActivityDestroyWatcher*/
    fun install(
      application: Application,
      objectWatcher: ObjectWatcher,
      configProvider: () -> Config
    ) {
      val activityDestroyWatcher =
        ActivityDestroyWatcher(objectWatcher, configProvider)
      application.registerActivityLifecycleCallbacks(activityDestroyWatcher.lifecycleCallbacks)
    }
```

首先创建 ActivityDestroyWatcher 的实例，然后调用 registerActivityLifecycleCallbacks 方法注册 Activity  生命周期的监听，当 Activity 销毁的时候就会回调 onActivityDestroyed 方法。

看下 onActivityDestroyed 方法：

```kotlin
  /*ActivityDestroyWatcher*/
  private val lifecycleCallbacks =
    object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
      override fun onActivityDestroyed(activity: Activity) {
        if (configProvider().watchActivities) {
          objectWatcher.watch(
              activity, "${activity::class.java.name} received Activity#onDestroy() callback"
          )
        }
      }
    }
```

判断是否检测 Activity，是则调用 ObjectWatcher 的 watch 方法将该 Activity 添加到观察集合中，这里先不分析 watch 方法。

**（2）FragmentDestroyWatcher#install()**

```kotlin
  /*FragmentDestroyWatcher*/
  fun install(
    application: Application,
    objectWatcher: ObjectWatcher,
    configProvider: () -> AppWatcher.Config
  ) {
    val fragmentDestroyWatchers = mutableListOf<(Activity) -> Unit>()

    // (1)
    if (SDK_INT >= O) {
      fragmentDestroyWatchers.add(
          AndroidOFragmentDestroyWatcher(objectWatcher, configProvider)
      )
    }

    // (2)
    getWatcherIfAvailable(
        ANDROIDX_FRAGMENT_CLASS_NAME,
        ANDROIDX_FRAGMENT_DESTROY_WATCHER_CLASS_NAME,
        objectWatcher,
        configProvider
    )?.let {
      fragmentDestroyWatchers.add(it)
    }

    // (3)
    getWatcherIfAvailable(
        ANDROID_SUPPORT_FRAGMENT_CLASS_NAME,
        ANDROID_SUPPORT_FRAGMENT_DESTROY_WATCHER_CLASS_NAME,
        objectWatcher,
        configProvider
    )?.let {
      fragmentDestroyWatchers.add(it)
    }

    if (fragmentDestroyWatchers.size == 0) {
      return
    }
    // (4)
    application.registerActivityLifecycleCallbacks(object : Application.ActivityLifecycleCallbacks by noOpDelegate() {
      override fun onActivityCreated(
        activity: Activity,
        savedInstanceState: Bundle?
      ) {
        for (watcher in fragmentDestroyWatchers) {
          watcher(activity)
        }
      }
    })
  }
```

源码中我标记了 4 个关注点，分别如下：

- （1）：系统版本大于等于 8.0 时，Fragment 用的是 android.app.Fragment，所以使用 AndroidOFragmentDestroyWatcher 检测，然后将它添加到集合中。
- （2）：看下 getWatcherIfAvailable 方法：

```kotlin
  /*FragmentDestroyWatcher*/
  private fun getWatcherIfAvailable(
    fragmentClassName: String,
    watcherClassName: String,
    objectWatcher: ObjectWatcher,
    configProvider: () -> AppWatcher.Config
  ): ((Activity) -> Unit)? {

    // androidx.fragment.app.Fragment 与 AndroidXFragmentDestroyWatcher 可以找到
    return if (classAvailable(fragmentClassName) &&
        classAvailable(watcherClassName)
    ) {
      // 反射获取 AndroidXFragmentDestroyWatcher 的构造方法
      val watcherConstructor = Class.forName(watcherClassName)
          .getDeclaredConstructor(ObjectWatcher::class.java, Function0::class.java)
      // 反射实例化 AndroidXFragmentDestroyWatcher 对象
      @Suppress("UNCHECKED_CAST")
      watcherConstructor.newInstance(objectWatcher, configProvider) as (Activity) -> Unit

    } else {
      null
    }
  }
```

这里判断引入的是否是 AndroidX 库，如果是则 Fragment 用的是 androidx.fragment.app.Fragment，需要通过反射实例化 AndroidXFragmentDestroyWatcher 对象。

接着使用 fragmentDestroyWatchers.add(it) 将 AndroidXFragmentDestroyWatcher 添加到集合中。

- （3）：与关注点（2）类似，判断引入的是否是 Support 库，如果是则 Fragment 用的是 android.support.v4.app.Fragment，需要通过反射实例化 AndroidSupportFragmentDestroyWatcher 对象，然后添加到集合中。
- （4）：注册 Activity  生命周期的监听，当 Activity 销毁的时候就会回调 onActivityDestroyed 方法，然后遍历 fragmentDestroyWatchers 集合拿到刚刚保存的 xxFragmentDestroyWatcher，接着注册 Fragment 生命周期的监听。

这里的 watcher(activity) 对于不熟悉 kotlin 的人来说可能不太好理解，它其实是使用了 Kotlin 中的高阶函数。我们可以发现 xxFragmentDestroyWatcher 都继承了 (Activity) -> Unit，转成 java 后其实是实现了 Function1 接口，然后重写了它的 invoke 函数。所以这里调用 watcher(activity)，实际是调用了 xxFragmentDestroyWatcher 的 invoke 方法。

AndroidOFragmentDestroyWatcher、AndroidSupportFragmentDestroyWatcher 与 AndroidXFragmentDestroyWatcher（多了个 ViewModelClearedWatcher 检测） 逻辑类似，所以这里只分析 AndroidXFragmentDestroyWatcher。看一下 AndroidXFragmentDestroyWatcher：

```kotlin
internal class AndroidXFragmentDestroyWatcher(
  private val objectWatcher: ObjectWatcher,
  private val configProvider: () -> Config
) : (Activity) -> Unit {

  private val fragmentLifecycleCallbacks = object : FragmentManager.FragmentLifecycleCallbacks() {

    override fun onFragmentCreated(
      fm: FragmentManager,
      fragment: Fragment,
      savedInstanceState: Bundle?
    ) {
      // (3)
      ViewModelClearedWatcher.install(fragment, objectWatcher, configProvider)
    }

    override fun onFragmentViewDestroyed(
      fm: FragmentManager,
      fragment: Fragment
    ) {
      val view = fragment.view
      if (view != null && configProvider().watchFragmentViews) {
        // (4)
        objectWatcher.watch(
            view, "${fragment::class.java.name} received Fragment#onDestroyView() callback " +
            "(references to its views should be cleared to prevent leaks)"
        )
      }
    }

    override fun onFragmentDestroyed(
      fm: FragmentManager,
      fragment: Fragment
    ) {
      if (configProvider().watchFragments) {
        // (5)
        objectWatcher.watch(
            fragment, "${fragment::class.java.name} received Fragment#onDestroy() callback"
        )
      }
    }
  }

  override fun invoke(activity: Activity) {
    if (activity is FragmentActivity) {
      val supportFragmentManager = activity.supportFragmentManager
      // (1)
      supportFragmentManager.registerFragmentLifecycleCallbacks(fragmentLifecycleCallbacks, true)
      // (2)
      ViewModelClearedWatcher.install(activity, objectWatcher, configProvider)
    }
  }
}
```

源码中我标记了 5 个关注点，分别如下：

- （1）：注册 Fragment 生命周期的监听。
- （2）：使用 ViewModelClearedWatcher 检测 Activity 中的 ViewModel。
- （3）：Fragment 创建的时候，使用 ViewModelClearedWatcher 检测 Fragment 中的 ViewModel。
- （4）：onFragmentViewDestroyed 生命周期回调的时候将 Fragment 中的 View 添加到观察集合中。
- （5）：onFragmentDestroyed 生命周期回调的时候将 Fragment 添加到观察集合中。

下面再具体看下 ViewModelClearedWatcher：

```kotlin
internal class ViewModelClearedWatcher(
  storeOwner: ViewModelStoreOwner,
  private val objectWatcher: ObjectWatcher,
  private val configProvider: () -> Config
) : ViewModel() {

  private val viewModelMap: Map<String, ViewModel>?

  init {
    // (3)
    viewModelMap = try {
      val mMapField = ViewModelStore::class.java.getDeclaredField("mMap")
      mMapField.isAccessible = true
      @Suppress("UNCHECKED_CAST")
      mMapField[storeOwner.viewModelStore] as Map<String, ViewModel>
    } catch (ignored: Exception) {
      null
    }
  }

  // (4)
  override fun onCleared() {
    if (viewModelMap != null && configProvider().watchViewModels) {
      viewModelMap.values.forEach { viewModel ->
        objectWatcher.watch(
            viewModel, "${viewModel::class.java.name} received ViewModel#onCleared() callback"
        )
      }
    }
  }

  companion object {
    fun install(
      storeOwner: ViewModelStoreOwner,
      objectWatcher: ObjectWatcher,
      configProvider: () -> Config
    ) {
      // (1)
      val provider = ViewModelProvider(storeOwner, object : Factory {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel?> create(modelClass: Class<T>): T =
          ViewModelClearedWatcher(storeOwner, objectWatcher, configProvider) as T
      })
      // (2)
      provider.get(ViewModelClearedWatcher::class.java)
    }
  }
}
```

源码中我标记了 4 个关注点，分别如下：

- （1）：创建 ViewModelClearedWatcher 的实例。
- （2）：将 ViewModel 保存到 ViewModelStore 中的 mMap 集合。
- （3）：反射拿到 ViewModelStore 中的 mMap 集合赋值给 viewModelMap 集合。
- （4）：当不再使用某个 ViewModel 时，会回调 ViewModel 的 onCleared 方法，这个时候检测 ViewModel。

**（3）onAppWatcherInstalled()**
 继续回去看 InternalAppWatcher#install() 中的关注点（3），这里的 onAppWatcherInstalled(application) 也是利用了 Kotlin 中的高阶函数，所以这里实际是调用了 InternalLeakCanary 的 invoke 方法。

那么这个 onAppWatcherInstalled 是哪里赋值的呢？其实是在 init 代码块中：

```kotlin
  /*InternalAppWatcher*/
  private val onAppWatcherInstalled: (Application) -> Unit

  init {
    val internalLeakCanary = try {
      val leakCanaryListener = Class.forName("leakcanary.internal.InternalLeakCanary")
      leakCanaryListener.getDeclaredField("INSTANCE")
          .get(null)
    } catch (ignored: Throwable) {
      NoLeakCanary
    }
    @kotlin.Suppress("UNCHECKED_CAST")
    onAppWatcherInstalled = internalLeakCanary as (Application) -> Unit
  }
```

可以看到，这里是通过反射获取 InternalLeakCanary 的实例，然后赋值给 onAppWatcherInstalled。

继续看下 InternalLeakCanary 的 invoke 方法：

```kotlin
  /*InternalLeakCanary */
  override fun invoke(application: Application) {
    _application = application

    checkRunningInDebuggableBuild()

    // (1)
    AppWatcher.objectWatcher.addOnObjectRetainedListener(this)

    // (2)
    val heapDumper = AndroidHeapDumper(application, createLeakDirectoryProvider(application))

    // (3)
    val gcTrigger = GcTrigger.Default

    val configProvider = { LeakCanary.config }

    val handlerThread = HandlerThread(LEAK_CANARY_THREAD_NAME)
    handlerThread.start()
    val backgroundHandler = Handler(handlerThread.looper)

    // (4)
    heapDumpTrigger = HeapDumpTrigger(
        application, backgroundHandler, AppWatcher.objectWatcher, gcTrigger, heapDumper,
        configProvider
    )
    application.registerVisibilityListener { applicationVisible ->
      this.applicationVisible = applicationVisible
      heapDumpTrigger.onApplicationVisibilityChanged(applicationVisible)
    }
    registerResumedActivityListener(application)
    // (5)
    addDynamicShortcut(application)

    Handler().post {
      SharkLog.d {
        when (val iCanHasHeap = HeapDumpControl.iCanHasHeap()) {
          is Yup -> application.getString(R.string.leak_canary_heap_dump_enabled_text)
          is Nope -> application.getString(
              R.string.leak_canary_heap_dump_disabled_text, iCanHasHeap.reason()
          )
        }
      }
    }
  }
```

源码中我标记了 5 个关注点，分别如下：

- （1）：添加未回收对象的监听，用于后续回调的时候用。
- （2）：创建 AndroidHeapDumper 的实例，用于生成 hprof 文件。
- （3）：创建 GcTrigger 的实例，用于触发 GC 操作。
- （4）：创建 HeapDumpTrigger 的实例，用于检测。
- （5）：添加进入内存泄漏界面的快捷方式。

到这里 InternalAppWatcher 的 install 方法就分析完了，总结一下。

**小结**
 install 方法主要是将所有需要检测的对象对应的观察者准备好，即 ActivityDestroyWatcher、AndroidOFragmentDestroyWatcher、AndroidXFragmentDestroyWatcher、AndroidSupportFragmentDestroyWatcher 与 ViewModelClearedWatcher。还有准备好后面检测内存泄漏需要用到的 AndroidHeapDumper、GcTrigger、HeapDumpTrigger 等。

### 3. ObjectWatcher#watch()

InternalAppWatcher 的 install 方法已经将所有需要检测的对象对应的观察者，以及后面检测内存泄漏需要用到的类都准备好了。然后每个对象销毁的时候就会调用 ObjectWatcher 的 watch 方法，我们看下这个方法：

```kotlin
  /*ObjectWatcher*/
  // 观察集合
  private val watchedObjects = mutableMapOf<String, KeyedWeakReference>()
  // 引用队列
  private val queue = ReferenceQueue<Any>()

  @Synchronized fun watch(
    watchedObject: Any,
    description: String
  ) {
    if (!isEnabled()) {
      return
    }
    // (1)
    removeWeaklyReachableObjects()
    // 获取唯一标识符
    val key = UUID.randomUUID()
        .toString()
    val watchUptimeMillis = clock.uptimeMillis()
    // (2)
    val reference =
      KeyedWeakReference(watchedObject, key, description, watchUptimeMillis, queue)
	...
    // (3)
    watchedObjects[key] = reference
    // (4)
    checkRetainedExecutor.execute {
      moveToRetained(key)
    }
  }
```

源码中我标记了 4 个关注点，分别如下：

- （1）：removeWeaklyReachableObjects 方法点进去看看：

```kotlin
  /*ObjectWatcher*/
  private fun removeWeaklyReachableObjects() {
    var ref: KeyedWeakReference?
    do {
      ref = queue.poll() as KeyedWeakReference?
      if (ref != null) {
        watchedObjects.remove(ref.key)
      }
    } while (ref != null)
  }
复制代码
```

先看下 KeyedWeakReference：

```kotlin
class KeyedWeakReference(
  referent: Any,
  val key: String,
  val description: String,
  val watchUptimeMillis: Long,
  referenceQueue: ReferenceQueue<Any>
) : WeakReference<Any>(
    referent, referenceQueue
) {}
```

可以看到，KeyedWeakReference 继承了 WeakReference，并且传入了观察对象和引用队列，所以 KeyedWeakReference 是一个弱引用类型，作用是将需要观察的对象与弱引用关联。

`每当发生 GC 时，弱引用所持有的对象就会被回收，并且 JVM 会把该对象放入关联的引用队列中。`所以上面 removeWeaklyReachableObjects 方法的意思就是从引用队列中获取 KeyedWeakReference，如果可以获取到说明该对象是已经被回收的对象，则将它从观察集合（watchedObjects）中移除。

- （2）：将需要观察的对象封装到 KeyedWeakReference 中。
- （3）：将 reference 保存到观察集合中。
- （4）：通过执行器执行一个任务，它是在 InternalAppWatcher 中初始化的：

```kotlin
  /*InternalAppWatcher*/
  private val mainHandler by lazy {
    Handler(Looper.getMainLooper())
  }

  private val checkRetainedExecutor = Executor {
    // 其中 val watchDurationMillis: Long = TimeUnit.SECONDS.toMillis(5)
    mainHandler.postDelayed(it, AppWatcher.config.watchDurationMillis)
  }

  val objectWatcher = ObjectWatcher(
      clock = clock,
      checkRetainedExecutor = checkRetainedExecutor,
      isEnabled = { true }
  )
```

可以看到，内部是使用一个 Handler 发送了一个 5 秒的延迟消息。

继续看 moveToRetained 方法：

```kotlin
  /*ObjectWatcher*/
  @Synchronized private fun moveToRetained(key: String) {
    // (1)
    removeWeaklyReachableObjects()
    // (2)
    val retainedRef = watchedObjects[key]
    if (retainedRef != null) {
      retainedRef.retainedUptimeMillis = clock.uptimeMillis()
      // (3)
      onObjectRetainedListeners.forEach { it.onObjectRetained() }
    }
  }
```

源码中我标记了 3 个关注点，分别如下：

- （1）：与上面一样，将被回收的对象从观察集合（watchedObjects）中移除。
- （2）：从观察集合中取出 key 对应的观察对象。
- （3）：onObjectRetainedListeners 是前面 InternalLeakCanary 的 invoke 方法中添加的，这里遍历回调 onObjectRetained 方法：

```kotlin
  /*InternalLeakCanary*/
  override fun onObjectRetained() = scheduleRetainedObjectCheck()

  /*InternalLeakCanary*/
  fun scheduleRetainedObjectCheck() {
    if (this::heapDumpTrigger.isInitialized) {
      heapDumpTrigger.scheduleRetainedObjectCheck()
    }
  }
```

最终走到 HeapDumpTrigger 的 scheduleRetainedObjectCheck 方法：

```kotlin
  /*HeapDumpTrigger*/
  fun scheduleRetainedObjectCheck(
    delayMillis: Long = 0L
  ) {
    val checkCurrentlyScheduledAt = checkScheduledAt
    if (checkCurrentlyScheduledAt > 0) {
      return
    }
    checkScheduledAt = SystemClock.uptimeMillis() + delayMillis
    backgroundHandler.postDelayed({
      checkScheduledAt = 0
      checkRetainedObjects()
    }, delayMillis)
  }
```

如果已经在检测则直接 return，否则使用 Handler 发送一个延迟消息检测未回收的对象。

看下 checkRetainedObjects 方法：

```kotlin
  /*HeapDumpTrigger*/
  private fun checkRetainedObjects() {
    ...
    val config = configProvider()
    ...
    // (1)
    var retainedReferenceCount = objectWatcher.retainedObjectCount

    // (2)
    if (retainedReferenceCount > 0) {
      gcTrigger.runGc()
      retainedReferenceCount = objectWatcher.retainedObjectCount
    }

    // (3)
    if (checkRetainedCount(retainedReferenceCount, config.retainedVisibleThreshold)) return

    val now = SystemClock.uptimeMillis()
    val elapsedSinceLastDumpMillis = now - lastHeapDumpUptimeMillis
    // (4)
    if (elapsedSinceLastDumpMillis < WAIT_BETWEEN_HEAP_DUMPS_MILLIS) {
      onRetainInstanceListener.onEvent(DumpHappenedRecently)
      showRetainedCountNotification(
          objectCount = retainedReferenceCount,
          contentText = application.getString(R.string.leak_canary_notification_retained_dump_wait)
      )
      scheduleRetainedObjectCheck(
          delayMillis = WAIT_BETWEEN_HEAP_DUMPS_MILLIS - elapsedSinceLastDumpMillis
      )
      return
    }

    dismissRetainedCountNotification()
    // (5)
    dumpHeap(retainedReferenceCount, retry = true)
  }
```

源码中我标记了 5 个关注点，分别如下：

- （1）：获取未回收对象的数量。
- （2）：未回收对象的数量大于 0，则手动执行 GC 操作，然后再获取一次。
- （3）：检查未回收对象的数量，如果小于触发 dump heap 的阈值（5 个），则不继续执行下面的 dump heap。
- （4）：当前时间距离上次 dump head 的时间小于 1 分钟，那么使用 Handler 发送一个延迟消息，等满足一分钟的时候再次调用 checkRetainedObjects 方法检测。
- （5）：执行 dump heap。看一下 dumpHeap 方法：

```kotlin
  /*HeapDumpTrigger*/
  private fun dumpHeap(
    retainedReferenceCount: Int,
    retry: Boolean
  ) {
    saveResourceIdNamesToMemory()
    val heapDumpUptimeMillis = SystemClock.uptimeMillis()
    KeyedWeakReference.heapDumpUptimeMillis = heapDumpUptimeMillis
    // (1)
    when (val heapDumpResult = heapDumper.dumpHeap()) {
      is NoHeapDump -> {
        if (retry) {
          SharkLog.d { "Failed to dump heap, will retry in $WAIT_AFTER_DUMP_FAILED_MILLIS ms" }
          // (2)
          scheduleRetainedObjectCheck(
              delayMillis = WAIT_AFTER_DUMP_FAILED_MILLIS
          )
        } else {
          SharkLog.d { "Failed to dump heap, will not automatically retry" }
        }
        showRetainedCountNotification(
            objectCount = retainedReferenceCount,
            contentText = application.getString(
                R.string.leak_canary_notification_retained_dump_failed
            )
        )
      }
      is HeapDump -> {
        lastDisplayedRetainedObjectCount = 0
        lastHeapDumpUptimeMillis = SystemClock.uptimeMillis()
        // (3)
        objectWatcher.clearObjectsWatchedBefore(heapDumpUptimeMillis)
        // (4)
        HeapAnalyzerService.runAnalysis(
            context = application,
            heapDumpFile = heapDumpResult.file,
            heapDumpDurationMillis = heapDumpResult.durationMillis
        )
      }
    }
  }
```

源码中我标记了 5 个关注点，分别如下：

- （1）：调用 HeapDumper#dumpHeap() 生成 hprof 文件，内部实际是调用了 AndroidHeapDumper#dumpHeap()。

看一下这个方法：

```kotlin
  /*AndroidHeapDumper*/
  override fun dumpHeap(): DumpHeapResult {
    val heapDumpFile = leakDirectoryProvider.newHeapDumpFile() ?: return NoHeapDump
	...
    return try {
      val durationMillis = measureDurationMillis {
        // 关注点
        Debug.dumpHprofData(heapDumpFile.absolutePath)
      }
      if (heapDumpFile.length() == 0L) {
        SharkLog.d { "Dumped heap file is 0 byte length" }
        NoHeapDump
      } else {
        HeapDump(file = heapDumpFile, durationMillis = durationMillis)
      }
    } catch (e: Exception) {
      ...
    } finally {
      ...
    }
  }
```

可以看到，这里是调用了系统的 Debug#dumpHprofData() 生成 hprof 文件。

- （2）：没有生成 hprof 文件则发送延迟消息重新再检测。
- （3）：有生成 hprof 文件则清除 watchedObjects 中保存的 KeyedWeakReference。
- （4）：最后调用 HeapAnalyzerService#runAnalysis() 进行 hprof 文件的分析。

**小结**
 ObjectWatcher 的 watch 方法主要是检测发生内存泄漏的对象，核心原理是`基于  WeakReference 和 ReferenceQueue 实现的。也就是每当发生 GC 时，弱引用所持有的对象就会被回收，并且 JVM 会把该对象放入关联的引用队列中。`

具体步骤为：

1. 将需要观察的对象封装到 KeyedWeakReference 中，这样观察对象和 WeakReference（弱引用）、ReferenceQueue（引用队列） 就进行了关联。
2. 接着将封装好的 KeyedWeakReference 保存到 watchedObjects（观察集合）。
3. 每当发生 GC 时，KeyedWeakReference 所持有的对象就会被回收，并加入到引用队列。
4. 然后遍历引用队列中保存的观察对象，从观察集合中删除这些对象。因为保存在引用队列中的都是已经回收的对象，所以最后观察集合中剩下的就是发生内存泄漏的对象了。
5. 最后如果未回收的对象大于等于 5 个，就进行 dump head 操作生成 hprof 文件。

### 4. HeapAnalyzerService#runAnalysis()

上面已经找到内存泄漏的对象并且生成了 hprof 文件，那么 runAnalysis 方法就是用来分析该文件的。该方法就不详细分析了，只分析重点流程。先看一下这个方法：

```kotlin
    /*HeapAnalyzerService*/
    fun runAnalysis(
      context: Context,
      heapDumpFile: File,
      heapDumpDurationMillis: Long? = null
    ) {
      val intent = Intent(context, HeapAnalyzerService::class.java)
      intent.putExtra(HEAPDUMP_FILE_EXTRA, heapDumpFile)
      heapDumpDurationMillis?.let {
        intent.putExtra(HEAPDUMP_DURATION_MILLIS, heapDumpDurationMillis)
      }
      // 调用下面的 startForegroundService 方法
      startForegroundService(context, intent)
    }

    /*HeapAnalyzerService*/
    private fun startForegroundService(
      context: Context,
      intent: Intent
    ) {
      if (SDK_INT >= 26) {
        context.startForegroundService(intent)
      } else {
        context.startService(intent)
      }
    }
```

可以看到，这里开启了一个服务来分析 hprof 文件。HeapAnalyzerService 继承了 ForegroundService，ForegroundService 又继承了 IntentService，所以 HeapAnalyzerService 是一个任务执行完成后会自动停止的服务。启动服务后最终会回调到 HeapAnalyzerService 的 onHandleIntentInForeground 方法：

```kotlin
  /*HeapAnalyzerService*/
  override fun onHandleIntentInForeground(intent: Intent?) {
    ...
    val heapDumpFile = intent.getSerializableExtra(HEAPDUMP_FILE_EXTRA) as File
    val heapDumpDurationMillis = intent.getLongExtra(HEAPDUMP_DURATION_MILLIS, -1)

    val config = LeakCanary.config
    val heapAnalysis = if (heapDumpFile.exists()) {
      // 关注点
      analyzeHeap(heapDumpFile, config)
    } else {
      missingFileFailure(heapDumpFile)
    }
    val fullHeapAnalysis = when (heapAnalysis) {
      is HeapAnalysisSuccess -> heapAnalysis.copy(dumpDurationMillis = heapDumpDurationMillis)
      is HeapAnalysisFailure -> heapAnalysis.copy(dumpDurationMillis = heapDumpDurationMillis)
    }
    onAnalysisProgress(REPORTING_HEAP_ANALYSIS)
    config.onHeapAnalyzedListener.onHeapAnalyzed(fullHeapAnalysis)
  }
```

这里拿到 hprof 文件，然后调用 analyzeHeap 进行分析。点进去看看：

```kotlin
  /*HeapAnalyzerService*/
  private fun analyzeHeap(
    heapDumpFile: File,
    config: Config
  ): HeapAnalysis {
    val heapAnalyzer = HeapAnalyzer(this)
	...
    return heapAnalyzer.analyze(
        heapDumpFile = heapDumpFile,
        leakingObjectFinder = config.leakingObjectFinder,
        referenceMatchers = config.referenceMatchers,
        computeRetainedHeapSize = config.computeRetainedHeapSize,
        objectInspectors = config.objectInspectors,
        metadataExtractor = config.metadataExtractor,
        proguardMapping = proguardMappingReader?.readProguardMapping()
    )
  }
```

到这里就开始使用 Shark 库进行 hprof 文件的分析了。看一下 HeapAnalyzer 的 analyze 方法：

```kotlin
  /*HeapAnalyzer*/
  fun analyze(
    heapDumpFile: File,
    leakingObjectFinder: LeakingObjectFinder,
    referenceMatchers: List<ReferenceMatcher> = emptyList(),
    computeRetainedHeapSize: Boolean = false,
    objectInspectors: List<ObjectInspector> = emptyList(),
    metadataExtractor: MetadataExtractor = MetadataExtractor.NO_OP,
    proguardMapping: ProguardMapping? = null
  ): HeapAnalysis {
    ...
    return try {
      listener.onAnalysisProgress(PARSING_HEAP_DUMP)
      val sourceProvider = ConstantMemoryMetricsDualSourceProvider(FileSourceProvider(heapDumpFile))
      // (1)
	  sourceProvider.openHeapGraph(proguardMapping).use { graph ->
        // (2)
        val helpers =
          FindLeakInput(graph, referenceMatchers, computeRetainedHeapSize, objectInspectors)
        // (3)
        val result = helpers.analyzeGraph(
            metadataExtractor, leakingObjectFinder, heapDumpFile, analysisStartNanoTime
        )
        ...
      }
    } catch (exception: Throwable) {
      ...
    }
  }
```

可以看到，这里首先生成 heap graph，也就是 heap 的对象关系图。然后将它代入构造一个 FindLeakInput 对象，最后调用 analyzeGraph 方法分析 graph。

看一下 analyzeGraph 方法：

```kotlin
  /*HeapAnalyzer--FindLeakInput*/
  private fun FindLeakInput.analyzeGraph(
    metadataExtractor: MetadataExtractor,
    leakingObjectFinder: LeakingObjectFinder,
    heapDumpFile: File,
    analysisStartNanoTime: Long
  ): HeapAnalysisSuccess {
    ...
    // (1)
    val leakingObjectIds = leakingObjectFinder.findLeakingObjectIds(graph)
    // (2)
    val (applicationLeaks, libraryLeaks) = findLeaks(leakingObjectIds)
    // (3)
    return HeapAnalysisSuccess(
        heapDumpFile = heapDumpFile,
        createdAtTimeMillis = System.currentTimeMillis(),
        analysisDurationMillis = since(analysisStartNanoTime),
        metadata = metadata,
        applicationLeaks = applicationLeaks,
        libraryLeaks = libraryLeaks
    )
  }
```

首先根据 heap graph 找到泄露对象的一组 id，然后根据这些泄漏对象的 id 找到漏对象到 GC roots 的路径，因为一个对象有可能被多个对象引用，为了方便分析这里只保留每个泄漏对象到 GC roots 的最短路劲，最后将 hprof 分析结果返回。

上面的 hprof 分析结果最终会返回到 HeapAnalyzerService#onHandleIntentInForeground() 中：

```kotlin
  /*HeapAnalyzerService*/
  override fun onHandleIntentInForeground(intent: Intent?) {
    ...
    config.onHeapAnalyzedListener.onHeapAnalyzed(fullHeapAnalysis)
  }
```

接着回调到 DefaultOnHeapAnalyzedListener#onHeapAnalyzed()。如下：

```kotlin
  /*DefaultOnHeapAnalyzedListener*/
  override fun onHeapAnalyzed(heapAnalysis: HeapAnalysis) {

    val id = LeaksDbHelper(application).writableDatabase.use { db ->
      HeapAnalysisTable.insert(db, heapAnalysis)
    }

    val (contentTitle, screenToShow) = when (heapAnalysis) {
      is HeapAnalysisFailure -> application.getString(
          R.string.leak_canary_analysis_failed
      ) to HeapAnalysisFailureScreen(id)
      is HeapAnalysisSuccess -> {
        val retainedObjectCount = heapAnalysis.allLeaks.sumBy { it.leakTraces.size }
        val leakTypeCount = heapAnalysis.applicationLeaks.size + heapAnalysis.libraryLeaks.size
        application.getString(
            R.string.leak_canary_analysis_success_notification, retainedObjectCount, leakTypeCount
        ) to HeapDumpScreen(id)
      }
    }

    if (InternalLeakCanary.formFactor == TV) {
      showToast(heapAnalysis)
      printIntentInfo()
    } else {
      // 调用下面的 showNotification 方法
      showNotification(screenToShow, contentTitle)
    }
  }

  /*DefaultOnHeapAnalyzedListener*/
  private fun showNotification(
    screenToShow: Screen,
    contentTitle: String
  ) {
    // (1)
    val pendingIntent = LeakActivity.createPendingIntent(
        application, arrayListOf(HeapDumpsScreen(), screenToShow)
    )

    val contentText = application.getString(R.string.leak_canary_notification_message)

    // (2)
    Notifications.showNotification(
        application, contentTitle, contentText, pendingIntent,
        R.id.leak_canary_notification_analysis_result,
        LEAKCANARY_MAX
    )
  }
```

可以看到这里构建了一个 PendingIntent，并显示通知栏消息，当点击通知栏消息后就会跳转到 LeakActivity 界面，也就是展示内存泄漏的界面。

到这里，LeakCanary 的源码就分析完了，先总结一下 runAnalysis 方法。

**小结**
 runAnalysis 方法中开启一个 IntentService 服务来分析 hprof 文件，分析库使用的是 Shark。具体步骤是首选生成 heap graph（对象关系图），接着根据 heap graph 找到泄漏对象到 GC roots 的最短路径，最后将 hprof 分析结果返回并显示通知栏消息，当点击通知栏消息后就会跳转到展示内存泄漏的界面。

## 三、总结

总结一下整个流程：

1. 首先注册 Activity/Fragment（ViewModel 不需要注册） 生命周期的监听。
2. 然后每个对象销毁的时候将它们添加到观察集合中（watchedObjects），并且将该对象与弱引用（WeakReference）和引用队列（ReferenceQueque）进行关联。这样每当发生 GC 时，弱引用所持有的对象就会被回收，并加入到引用队列。
3. 然后遍历引用队列中保存的观察对象，从观察集合中删除这些对象，最后观察集合中剩下的就是发生内存泄漏的对象了。
4. 最后生成 hprof 文件，并用 Shark 开源库去分析该文件。


