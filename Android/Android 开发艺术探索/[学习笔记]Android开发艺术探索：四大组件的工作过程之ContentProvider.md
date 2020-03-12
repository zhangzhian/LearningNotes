# [学习笔记]Android开发艺术探索：四大组件的工作过程之ContentProvider

ContentProvider是一个内存共享型组件，他通过Binder向其他组件乃至其他应用提供数据，当ContentProvider所在的进程启动的时候，ContentProvider会同时启动并且发布到AMS中，需要注意的是，这个时候ContentProvider的onCreate要先于Application的onCreate执行，这是四大组件一个少有的现象

一个应用启动的时候，入口方法在ActivityThread中的main里，main是一个静态方法，在main中会创建ActivityThread的实例并创建主线程的消息队列，然后在ActivityThread的attach中远程调用AMS的attachApplication方法并将ActivityThread提供给AMS，ActivityThread是一个Binder对象，他的Binder接口是IApplicationThread，他主要用于ActivityThread和AMS之间的通信，会调用ApplicationThread的bindApplication方法，注意这个过程同样是跨进程完成的，bindApplication的逻辑是经过ActivityThread中的mH Hander切换到ActivityThread执行，具体是方法是handlerBindApplication,在handlerApplication方法中，ActivityThread会加载一个ContentProvider，然后再调用Application的onCreate。

ContentProvider启动后，外界就可以通过他所提供的增删改查四个接口来操作ContentProvider的数据源，insert,delete,update,query，这四个方法都是通过Binder来调用的，外界无法直接访问ContentProvider，他只能通过AMS提供的Uri来获取对应的ContentProvider的Binder接口和IContentProvider，然后通过IContentProvider来访问ContentProvider的数据源。

一般来说，ContentProvider都应该是单实例的，ContentProvider到底是不是单实例的，这是由他的android:multiprocess来决定的，当他为false的时候为多实例，默认为true，在实际的开发当中，我们并没有发现多实例的案例，官方文档的解释是避免进程间通信的开销，但是在实际开发中仍然缺少使用价值，因此我们可以简单认为ContentProvider就是单实例，接下来我们来分析下ContentProvider的启动过程

党文ContentProvider需要通过ContentResolver，ContentResolver是一个抽象类，通过Context的getContentResolver方法获取的实际上是ApplicationContentResolver对象，当ContentProvider所在的进程未启动的时候，第一次访问他时就会触发ContentProvider的创建，这也就伴随着ContentProvider所在的进程启动，通过ContentProvider的第四个方法的任何一个都能启动，这里我们通过query来讲解

ContentProvider的query方法，首先获取的IContentProvider对象，不管是通过acquireUnstableProvider方法还是直接通过acquireProvider方法，他们的本质都是一样的，最终都是通过acquireProvider的方法来获取ContentProvider，下面是ApplicationContentProvider的acquireProvider具体的实现方式：

 ```java
 		protected IContentProvider acquireProvider(Context context,String auth){
        return mMainThread.acquireProvider(context,
                ContentResolver.getAuthorityWithoutUserId(auth),
                resolveUserIdFromAuthority(auth),true);
    }
 ```

ApplicationContentResolver的acquireProvider方法并没有处理任何逻辑，他直接调用了ActivityThread的acquireProvider方法，acquireProvider的这个方法源码如下

```java
   public final IContentProvider acquireProvider(
            Context c, String auth, int userId, boolean stable) {
        final IContentProvider provider = acquireExistingProvider(c, auth, userId, stable);
        if (provider != null) {
            return provider;
        }

        // There is a possible race here.  Another thread may try to acquire
        // the same provider at the same time.  When this happens, we want to ensure
        // that the first one wins.
        // Note that we cannot hold the lock while acquiring and installing the
        // provider since it might take a long time to run and it could also potentially
        // be re-entrant in the case where the provider is in the same process.
        IActivityManager.ContentProviderHolder holder = null;
        try {
            holder = ActivityManagerNative.getDefault().getContentProvider(
                    getApplicationThread(), auth, userId, stable);
        } catch (RemoteException ex) {
        }
        if (holder == null) {
            Slog.e(TAG, "Failed to find provider info for " + auth);
            return null;
        }

        // Install provider will increment the reference count for us, and break
        // any ties in the race.
        holder = installProvider(c, holder, holder.info,
                true /*noisy*/, holder.noReleaseNeeded, stable);
        return holder.provider;
    }
```

上面的代码首先会从ActivityThread中超找是否已经存在目标的ContentProvider，如果存在就直接返回，ActivityThread中通过mProviderMap来存储已经启动的ContentProvider

```java
 		// The lock of mProviderMap protects the following variables.
    final ArrayMap<ProviderKey, ProviderClientRecord> mProviderMap
        = new ArrayMap<ProviderKey, ProviderClientRecord>();
```

如果目前ContentProvider并没有启动，那么久发送一个进程间请求给AMS让其启动目标ContentProvider，最后通过installProvider方法来修改引用计数，那么AMS是如何启动ContentProvider的呢？我们知道，ContentProvider被启动时伴随着进程也会被启动，那么AMS重，首先会启动ContentProvider所在的进程，然后再ContentProvider，启动进程是由AMS的startProcessLocked方法来完成的，其内部主要是通过Process的start方法来完成一系列的新进程启动，新进程启动后其入口方法为ActivityThread的main方法：

```java
    public static void main(String[] args) {
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        Security.addProvider(new AndroidKeyStoreProvider());

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    } 
```

可以看到，ActivityThread的main方法是一个静态方法，他在内部先创建了一个实例并且调用了attach方法来进行一系列的初始化，接着开始消息循环了，最终会将ApplicationThread对象通过AMS的attchApplication方法跨进程传递给AMS，最终AMS会将ContentProvider的创建过程

```java
          try {
                mgr.attachApplication(mAppThread);
            } catch (RemoteException ex) {
                // Ignore
            } 
```

AMS的attachApplication方法又调用attachApplicationLocked方法，attachApplicationLocked中又调用了bindApplication，注意这个过程也是进程间调用

ActivityThread的bindApplication会发送一个BIND_APPLICATION类型的消息给mH,他是一个Handler，他收到消息后会调用ActivityThread，过程如下：

```java
    				AppBindData data = new AppBindData();
            data.processName = processName;
            data.appInfo = appInfo;
            data.providers = providers;
            data.instrumentationName = instrumentationName;
            data.instrumentationArgs = instrumentationArgs;
            data.instrumentationWatcher = instrumentationWatcher;
            data.instrumentationUiAutomationConnection = instrumentationUiConnection;
            data.debugMode = debugMode;
            data.enableOpenGlTrace = enableOpenGlTrace;
            data.restrictedBackupMode = isRestrictedBackupMode;
            data.persistent = persistent;
            data.config = config;
            data.compatInfo = compatInfo;
            data.initProfilerInfo = profilerInfo;
            sendMessage(H.BIND_APPLICATION, data);
```

ActivityThread的bindBaseApplication则完成了Application的创建以及ContentProvider的创建，可以分为如下四个步骤：

1.创建ContextImpl和instrumentation

```java
            ContextImpl instrContext = ContextImpl.createAppContext(this, pi);

            try {
                java.lang.ClassLoader cl = instrContext.getClassLoader();
                mInstrumentation = (Instrumentation)
                    cl.loadClass(data.instrumentationName.getClassName()).newInstance();
            } catch (Exception e) {
                throw new RuntimeException(
                    "Unable to instantiate instrumentation "
                    + data.instrumentationName + ": " + e.toString(), e);
            }

            mInstrumentation.init(this, instrContext, appContext,
                   new ComponentName(ii.packageName, ii.name), data.instrumentationWatcher,
                   data.instrumentationUiAutomationConnection);
```

2.创建Application对象

```java
            Application app = data.info.makeApplication(data.restrictedBackupMode, null);
            mInitialApplication = app;
```

3.启动当前进程的ContentProvider并调用onCreate方法

```java
       		if (!data.restrictedBackupMode) {
                List<ProviderInfo> providers = data.providers;
                if (providers != null) {
                    installContentProviders(app, providers);
                    // For process that contains content providers, we want to
                    // ensure that the JIT is enabled "at some point".
                    mH.sendEmptyMessageDelayed(H.ENABLE_JIT, 10*1000);
                }
            }
```

 installContentProviders完成了ContentProvider的启动工作，他的实现，首先是遍历ProviderInfo的列表并一一调用installProvider方法来启动他们，接着将以及启动的ContentProvider发送到AMS中，AMS会把他们存储在ProviderMap中，这样一来外部调用者可以直接从AMS中获取ContentProvider了

```java
  private void installContentProviders(
            Context context, List<ProviderInfo> providers) {
        final ArrayList<IActivityManager.ContentProviderHolder> results =
            new ArrayList<IActivityManager.ContentProviderHolder>();

        for (ProviderInfo cpi : providers) {
            if (DEBUG_PROVIDER) {
                StringBuilder buf = new StringBuilder(128);
                buf.append("Pub ");
                buf.append(cpi.authority);
                buf.append(": ");
                buf.append(cpi.name);
                Log.i(TAG, buf.toString());
            }
            IActivityManager.ContentProviderHolder cph = installProvider(context, null, cpi,
                    false /*noisy*/, true /*noReleaseNeeded*/, true /*stable*/);
            if (cph != null) {
                cph.noReleaseNeeded = true;
                results.add(cph);
            }
        }

        try {
            ActivityManagerNative.getDefault().publishContentProviders(
                getApplicationThread(), results);
        } catch (RemoteException ex) {
        }
    }
```

下面看一下ContentProvider对象的创建过程，在installProvider方法中有下面一段代码，并通过类加载器完成了ContentProvider对象的创建：

```java
					 try {
                final java.lang.ClassLoader cl = c.getClassLoader();
                localProvider = (ContentProvider)cl.
                    loadClass(info.name).newInstance();
                provider = localProvider.getIContentProvider();
                if (provider == null) {
                    Slog.e(TAG, "Failed to instantiate class " +
                          info.name + " from sourceDir " +
                          info.applicationInfo.sourceDir);
                    return null;
                }
                if (DEBUG_PROVIDER) Slog.v(
                    TAG, "Instantiating local provider " + info.name);
                // XXX Need to create the correct context for this provider.
                localProvider.attachInfo(c, info); 
```

在上述代码中，除了完成ContentProvider对象的创建，还会通过ContentProvider的attachInfo方法来调用他的onCreate方法，如下所示：

```java
    private void attachInfo(Context context, ProviderInfo info, boolean testing) {
        mNoPerms = testing;

        /*
         * Only allow it to be set once, so after the content service gives
         * this to us clients can't change it.
         */
        if (mContext == null) {
            mContext = context;
            if (context != null) {
                mTransport.mAppOpsManager = (AppOpsManager) context.getSystemService(
                        Context.APP_OPS_SERVICE);
            }
            mMyUid = Process.myUid();
            if (info != null) {
                setReadPermission(info.readPermission);
                setWritePermission(info.writePermission);
                setPathPermissions(info.pathPermissions);
                mExported = info.exported;
                mSingleUser = (info.flags & ProviderInfo.FLAG_SINGLE_USER) != 0;
                setAuthorities(info.authority);
            }
            ContentProvider.this.onCreate();
        }
    } 
```

到此为止，ContentProvider已经被创建并且其onCreate方法也被调用了，这意味着他已经启动完成了

4.调用Application的onCreate方法

```java
            try {
                mInstrumentation.callApplicationOnCreate(app);
            } catch (Exception e) {
                if (!mInstrumentation.onException(app, e)) {
                    throw new RuntimeException(
                        "Unable to create application " + app.getClass().getName()
                        + ": " + e.toString(), e);
                }
            }
```

经过上面的四个步骤，ContentProvider已经启动了，并且在其所在进程的Application也已经启动了，这意味着ContentProvider所在的进程已经完成了整个的启动过程，然后其他应用通过AMS来访问这个ContentProvider，我们来看下query的实现

```java
        @Override
        public Cursor query(String callingPkg, Uri uri, String[] projection,
                String selection, String[] selectionArgs, String sortOrder,
                ICancellationSignal cancellationSignal) {
            validateIncomingUri(uri);
            uri = getUriWithoutUserId(uri);
            if (enforceReadPermission(callingPkg, uri, null) != AppOpsManager.MODE_ALLOWED) {
                return rejectQuery(uri, projection, selection, selectionArgs, sortOrder,
                        CancellationSignal.fromTransport(cancellationSignal));
            }
            final String original = setCallingPackage(callingPkg);
            try {
                return ContentProvider.this.query(
                        uri, projection, selection, selectionArgs, sortOrder,
                        CancellationSignal.fromTransport(cancellationSignal));
            } finally {
                setCallingPackage(original);
            }
        }
```

很显然，ContentProvider.Transport的query方法调用了ContentProvider的query方法，query方法的执行结果再通过Binder返回给调用者，这样一来整个调用过程就完成了，除了query方法，其他方法也是类似的，这里就不去深究了
