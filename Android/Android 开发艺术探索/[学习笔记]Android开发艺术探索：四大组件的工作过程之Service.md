# [学习笔记]Android开发艺术探索：四大组件的工作过程之Service

Service有两种工作状态：

1. 启动状态：执行后台计算
2. 绑定状态：用于其他组件与Service交互

## Service的启动过程

Service的启动从 ContextWrapper 的 startService 开始

在ContextWrapper中，大部分操作通过一个 ContextImpl 对象mBase实现

/frameworks/base/core/java/android/content/ContextWrapper.java：

```java
 @Override
    public ComponentName startService(Intent service) {
        return mBase.startService(service);
    }
```

在ContextImpl中， mBase.startService() 会调用 startServiceCommon 方法，而 startServiceCommon方法又会通过 ActivityManagerNative.getDefault() （实际上就是 AMS）这个对象来启动一个服务。

/frameworks/base/core/java/android/app/ContextImpl.java:

```java
 		@Override
    public ComponentName startService(Intent service) {
        warnIfCallingFromSystemProcess();
        return startServiceCommon(service, mUser);
    }

 		private ComponentName startServiceCommon(Intent service, UserHandle user) {
        try {
            validateServiceIntent(service);
            service.prepareToLeaveProcess();
            ComponentName cn = ActivityManagerNative.getDefault().startService(
                mMainThread.getApplicationThread(), service,
                service.resolveTypeIfNeeded(getContentResolver()), user.getIdentifier());
            if (cn != null) {
                if (cn.getPackageName().equals("!")) {
                    throw new SecurityException(
                            "Not allowed to start service " + service
                            + " without permission " + cn.getClassName());
                } else if (cn.getPackageName().equals("!!")) {
                    throw new SecurityException(
                            "Unable to start service " + service
                            + ": " + cn.getClassName());
                }
            }
            return cn;
        } catch (RemoteException e) {
            return null;
        }
    }
```

AMS会通过一个 ActiveService 对象（辅助AMS进行Service管理的类）mServices来完 成启动Service： mServices.startServiceLocked() 。

/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java:

```java
  public ComponentName startService(IApplicationThread caller, Intent service,
            String resolvedType, int userId) {
        enforceNotIsolatedCaller("startService");
        // Refuse possible leaked file descriptors
        if (service != null && service.hasFileDescriptors() == true) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        if (DEBUG_SERVICE)
            Slog.v(TAG, "startService: " + service + " type=" + resolvedType);
        synchronized(this) {
            final int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            ComponentName res = mServices.startServiceLocked(caller, service,
                    resolvedType, callingPid, callingUid, userId);
            Binder.restoreCallingIdentity(origId);
            return res;
        }
    }
```

在mServices.startServiceLocked()最后会调用 startServiceInnerLocked() 方法：将 Service的信息包装成一个 ServiceRecord 对象，通过 bringUpServiceLocked() 方法来处理，bringUpServiceLocked()又调用了 realStartServiceLocked() 方法，这才真正地去启动一个Service了。

 /frameworks/base/services/java/com/android/server/am/ActiveServices.java:

```java
    ComponentName startServiceLocked(IApplicationThread caller,
   			
        ...

        return startServiceInnerLocked(smap, service, r, callerFg, addToStarting);
    }

    ComponentName startServiceInnerLocked(ServiceMap smap, Intent service,
            ServiceRecord r, boolean callerFg, boolean addToStarting) {
        
      	...
        
        String error = bringUpServiceLocked(r, service.getFlags(), callerFg, false);
        
      	...

        return r.name;
    }
                                     
     private final String bringUpServiceLocked(ServiceRecord r,
            int intentFlags, boolean execInFg, boolean whileRestarting) {
        
       	...

        if (!isolated) {
            app = mAm.getProcessRecordLocked(procName, r.appInfo.uid, false);
            if (DEBUG_MU) Slog.v(TAG_MU, "bringUpServiceLocked: appInfo.uid=" + r.appInfo.uid + " app=" + app);
            if (app != null && app.thread != null) {
                try {
                    app.addPackage(r.appInfo.packageName, mAm.mProcessStats);
                    realStartServiceLocked(r, app, execInFg);
                    return null;
                } catch (RemoteException e) {
                    Slog.w(TAG, "Exception when starting service " + r.shortName, e);
                }

                // If a dead object exception was thrown -- fall through to
                // restart the application.
            }
        } else {
            // If this service runs in an isolated process, then each time
            // we call startProcessLocked() we will get a new isolated
            // process, starting another process if we are currently waiting
            // for a previous process to come up.  To deal with this, we store
            // in the service any current isolated process it is running in or
            // waiting to have come up.
            app = r.isolatedProc;
        }

       ...
    }
                                     
  	private final void realStartServiceLocked(ServiceRecord r,
            ProcessRecord app, boolean execInFg) throws RemoteException {
        
      	...

        boolean created = false;
        try {
            String nameTerm;
            int lastPeriod = r.shortName.lastIndexOf('.');
            nameTerm = lastPeriod >= 0 ? r.shortName.substring(lastPeriod) : r.shortName;
            EventLogTags.writeAmCreateService(
                    r.userId, System.identityHashCode(r), nameTerm, r.app.uid, r.app.pid);
            synchronized (r.stats.getBatteryStats()) {
                r.stats.startLaunchedLocked();
            }
            mAm.ensurePackageDexOpt(r.serviceInfo.packageName);
            app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
            app.thread.scheduleCreateService(r, r.serviceInfo,
                    mAm.compatibilityInfoForPackageLocked(r.serviceInfo.applicationInfo),
                    app.repProcState);
            r.postNotification();
            created = true;
        } finally {
            if (!created) {
                app.services.remove(r);
                r.app = null;
                scheduleServiceRestartLocked(r, false);
            }
        }

        requestServiceBindingsLocked(r, execInFg);

        // If the service is in the started state, and there are no
        // pending arguments, then fake up one so its onStartCommand() will
        // be called.
        if (r.startRequested && r.callStart && r.pendingStarts.size() == 0) {
            r.pendingStarts.add(new ServiceRecord.StartItem(r, false, r.makeNextStartId(),
                    null, null));
        }

        sendServiceArgsLocked(r, execInFg, true);

        ...
          
    }
```

realStartServiceLocked()方法的工作如下：

i. app.thread.scheduleCreateService() 来创建Service并调用其onCreate()生命周期方法 

ii. sendServiceArgsLocked() 调用其他生命周期方法，如onStartCommand() 

iii. app.thread对象是 IApplicationThread 类型，实际上就是一个Binder，具体实现是 ApplicationThread继承ApplictionThreadNative

/frameworks/base/core/java/android/app/ActivityThread.java:

```java
   private class ApplicationThread extends ApplicationThreadNative { 
     		...
				public final void scheduleCreateService(IBinder token,
                ServiceInfo info, CompatibilityInfo compatInfo, int processState) {
            updateProcessState(processState, false);
            CreateServiceData s = new CreateServiceData();
            s.token = token;
            s.info = info;
            s.compatInfo = compatInfo;

            sendMessage(H.CREATE_SERVICE, s);
        }
     		...
   }


```

具体看app.thread.scheduleCreateService()：通过 sendMessage(H.CREATE_SERVICE , s) ， 这个过程和Activity启动过程类似，同时通过发送消息给Handler H来完成的。

 H会接受这个CREATE_SERVICE消息并通过ActivityThread的 handleCreateService() 来 完成Service的最终启动。

/frameworks/base/core/java/android/app/ActivityThread.java:

```java
 private void handleCreateService(CreateServiceData data) {
        // If we are getting ready to gc after going to the background, well
        // we are back active so skip it.
        unscheduleGcIdler();

        LoadedApk packageInfo = getPackageInfoNoCheck(
                data.info.applicationInfo, data.compatInfo);
   			//通过ClassLoader创建Service对象 
        Service service = null;
        try {
            java.lang.ClassLoader cl = packageInfo.getClassLoader();
            service = (Service) cl.loadClass(data.info.name).newInstance();
        } catch (Exception e) {
            if (!mInstrumentation.onException(service, e)) {
                throw new RuntimeException(
                    "Unable to instantiate service " + data.info.name
                    + ": " + e.toString(), e);
            }
        }

        try {
            if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);
						//创建Service内部的Context对象 
            ContextImpl context = ContextImpl.createAppContext(this, packageInfo);
            context.setOuterContext(service);
						//创建Application，并调用其onCreate()（只会有一次）
            Application app = packageInfo.makeApplication(false, mInstrumentation);
          	//通过 service.attach() 方法建立Service与context的联系（与Activity类似） 
            service.attach(context, this, data.info.name, data.token, app,
                    ActivityManagerNative.getDefault());
          	//调用service的 onCreate() 生命周期方法，至此，Service已经启动了
            service.onCreate();
          	//将Service对象存储到ActivityThread的一个ArrayMap中
            mServices.put(data.token, service);
            try {
                ActivityManagerNative.getDefault().serviceDoneExecuting(
                        data.token, 0, 0, 0);
            } catch (RemoteException e) {
                // nothing to do.
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(service, e)) {
                throw new RuntimeException(
                    "Unable to create service " + data.info.name
                    + ": " + e.toString(), e);
            }
        }
    }
```

  除此之外，ActivityThread中还会通过handleServiceArgs方法调用Service的onStartCommand方法

```java 
private void handleServiceArgs(ServiceArgsData data) {
        Service s = mServices.get(data.token);
        if (s != null) {
            try {
                if (data.args != null) {
                    data.args.setExtrasClassLoader(s.getClassLoader());
                }
                int res;
                if (!data.taskRemoved) {
                  	//调用Service的onStartCommand方法
                    res = s.onStartCommand(data.args, data.flags, data.startId);
                } else {
                    s.onTaskRemoved(data.args);
                    res = Service.START_TASK_REMOVED_COMPLETE;
                }

                QueuedWork.waitToFinish();

                try {
                    ActivityManagerNative.getDefault().serviceDoneExecuting(
                            data.token, 1, data.startId, res);
                } catch (RemoteException e) {
                    // nothing to do.
                }
                ensureJitEnabled();
            } catch (Exception e) {
                if (!mInstrumentation.onException(s, e)) {
                    throw new RuntimeException(
                            "Unable to start service " + s
                            + " with " + data.args + ": " + e.toString(), e);
                }
            }
        }
    }
```



## Service的绑定过程

Service的绑定是从 ContextWrapper 的 bindService 开始

/frameworks/base/core/java/android/content/ContextWrapper.java：

```java
	public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {
        return mBase.bindService(service, conn, flags);
    }
```

在ContextWrapper中，交给 ContextImpl 对象 mBase.bindService()

/frameworks/base/core/java/android/app/ContextImpl.java:

```java
   @Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {
        warnIfCallingFromSystemProcess();
        return bindServiceCommon(service, conn, flags, Process.myUserHandle());
    }

	private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags,
            UserHandle user) {
        IServiceConnection sd;
        if (conn == null) {
            throw new IllegalArgumentException("connection is null");
        }
        if (mPackageInfo != null) {
            sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(),
                    mMainThread.getHandler(), flags);
        } else {
            throw new RuntimeException("Not supported in system context");
        }
        validateServiceIntent(service);
        try {
            IBinder token = getActivityToken();
            if (token == null && (flags&BIND_AUTO_CREATE) == 0 && mPackageInfo != null
                    && mPackageInfo.getApplicationInfo().targetSdkVersion
                    < android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
                flags |= BIND_WAIVE_PRIORITY;
            }
            service.prepareToLeaveProcess();
            int res = ActivityManagerNative.getDefault().bindService(
                mMainThread.getApplicationThread(), getActivityToken(),
                service, service.resolveTypeIfNeeded(getContentResolver()),
                sd, flags, user.getIdentifier());
            if (res < 0) {
                throw new SecurityException(
                        "Not allowed to bind to service " + service);
            }
            return res != 0;
        } catch (RemoteException e) {
            return false;
        }
    }
	
```

最终会调用ContextImpl的 bindServiceCommon 方法，这个方法完成两件事：

i. 将客户端的ServiceConnection转化成 ServiceDispatcher.InnerConnection 对象。ServiceDispatcher连接ServiceConnection和InnerConnection。 

这个过程通过 LoadedApk 的 getServiceDispatcher 方法来实现，将客户端的 ServiceConnection和ServiceDispatcher的映射关系存在一个ArrayMap中。

 ii. 通过AMS来完成Service的具体绑定过 程 ActivityManagerNative.getDefault().bindService()

/frameworks/base/core/java/android/app/LoadedApk.java:

```java
 public final IServiceConnection getServiceDispatcher(ServiceConnection c,
            Context context, Handler handler, int flags) {
        synchronized (mServices) {
            LoadedApk.ServiceDispatcher sd = null;
            ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher> map = mServices.get(context);
            if (map != null) {
                sd = map.get(c);
            }
            if (sd == null) {
                sd = new ServiceDispatcher(c, context, handler, flags);
                if (map == null) {
                    map = new ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>();
                    mServices.put(context, map);
                }
                map.put(c, sd);
            } else {
                sd.validate(context, handler);
            }
            return sd.getIServiceConnection();
        }
    }
```

mServices 定义：

```java
private final ArrayMap<Context, ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>> mServices = new ArrayMap<Context, ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>>();
```



/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java：

```java
 	public int bindService(IApplicationThread caller, IBinder token,
            Intent service, String resolvedType,
            IServiceConnection connection, int flags, int userId) {
        enforceNotIsolatedCaller("bindService");
        // Refuse possible leaked file descriptors
        if (service != null && service.hasFileDescriptors() == true) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        synchronized(this) {
            return mServices.bindServiceLocked(caller, token, service, resolvedType,
                    connection, flags, userId);
        }
    }
```

 AMS中，bindService()方法再调用 bindServiceLocked() ，bindServiceLocked()再调 用 bringUpServiceLocked() ，bringUpServiceLocked()又会调用 realStartServiceLocked() 。

/frameworks/base/services/java/com/android/server/am/ActiveServices.java:

```java
 private final void realStartServiceLocked(ServiceRecord r,
            ProcessRecord app, boolean execInFg) throws RemoteException {
        ...
        boolean created = false;
        try {
            String nameTerm;
            int lastPeriod = r.shortName.lastIndexOf('.');
            nameTerm = lastPeriod >= 0 ? r.shortName.substring(lastPeriod) : r.shortName;
            EventLogTags.writeAmCreateService(
                    r.userId, System.identityHashCode(r), nameTerm, r.app.uid, r.app.pid);
            synchronized (r.stats.getBatteryStats()) {
                r.stats.startLaunchedLocked();
            }
            mAm.ensurePackageDexOpt(r.serviceInfo.packageName);
            app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
            app.thread.scheduleCreateService(r, r.serviceInfo,
                    mAm.compatibilityInfoForPackageLocked(r.serviceInfo.applicationInfo),
                    app.repProcState);
            r.postNotification();
            created = true;
        } finally {
            if (!created) {
                app.services.remove(r);
                r.app = null;
                scheduleServiceRestartLocked(r, false);
            }
        }

   			requestServiceBindingsLocked(r, execInFg);

        // If the service is in the started state, and there are no
        // pending arguments, then fake up one so its onStartCommand() will
        // be called.
        if (r.startRequested && r.callStart && r.pendingStarts.size() == 0) {
            r.pendingStarts.add(new ServiceRecord.StartItem(r, false, r.makeNextStartId(),
                    null, null));
        }

        sendServiceArgsLocked(r, execInFg, true);

        ...
    }
```

AMS的realStartServiceLocked()会调 用 ActiveServices 的 requrestServiceBindingLocked() 方法。

```java
 private final void requestServiceBindingsLocked(ServiceRecord r, boolean execInFg) {
        for (int i=r.bindings.size()-1; i>=0; i--) {
            IntentBindRecord ibr = r.bindings.valueAt(i);
            if (!requestServiceBindingLocked(r, ibr, execInFg, false)) {
                break;
            }
        }
    }

 	private final boolean requestServiceBindingLocked(ServiceRecord r,
            IntentBindRecord i, boolean execInFg, boolean rebind) {
        if (r.app == null || r.app.thread == null) {
            // If service is not currently running, can't yet bind.
            return false;
        }
        if ((!i.requested || rebind) && i.apps.size() > 0) {
            try {
                bumpServiceExecutingLocked(r, execInFg, "bind");
                r.app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
                r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                        r.app.repProcState);
                if (!rebind) {
                    i.requested = true;
                }
                i.hasBound = true;
                i.doRebind = false;
            } catch (RemoteException e) {
                if (DEBUG_SERVICE) Slog.v(TAG, "Crashed while binding " + r);
                return false;
            }
        }
        return true;
    }

```

requrestServiceBindingLocked() 方法，最终是调用了 ServiceRecord对象r的app.thread.scheduleBindService() 方法。

/frameworks/base/core/java/android/app/ActivityThread.java:

```java
   public final void scheduleBindService(IBinder token, Intent intent,
                boolean rebind, int processState) {
            updateProcessState(processState, false);
            BindServiceData s = new BindServiceData();
            s.token = token;
            s.intent = intent;
            s.rebind = rebind;

            if (DEBUG_SERVICE)
                Slog.v(TAG, "scheduleBindService token=" + token + " intent=" + intent + " uid="
                        + Binder.getCallingUid() + " pid=" + Binder.getCallingPid());
            sendMessage(H.BIND_SERVICE, s);
        }
```

ApplicationThread的一系列以schedule开头的方法，内部都通过Handler H来中转： scheduleBindService()内部也是通过 sendMessage(H.BIND_SERVICE , s)。

```java
 private void handleBindService(BindServiceData data) {
        Service s = mServices.get(data.token);
        if (DEBUG_SERVICE)
            Slog.v(TAG, "handleBindService s=" + s + " rebind=" + data.rebind);
        if (s != null) {
            try {
                data.intent.setExtrasClassLoader(s.getClassLoader());
                try {
                    if (!data.rebind) {
                        IBinder binder = s.onBind(data.intent);
                        ActivityManagerNative.getDefault().publishService(
                                data.token, data.intent, binder);
                    } else {
                        s.onRebind(data.intent);
                        ActivityManagerNative.getDefault().serviceDoneExecuting(
                                data.token, 0, 0, 0);
                    }
                    ensureJitEnabled();
                } catch (RemoteException ex) {
                }
            } catch (Exception e) {
                if (!mInstrumentation.onException(s, e)) {
                    throw new RuntimeException(
                            "Unable to bind to service " + s
                            + " with " + data.intent + ": " + e.toString(), e);
                }
            }
        }
    }
```

在H内部接收到BIND_SERVICE这类消息时就交给 ActivityThread 的 handleBindService() 方法处理： 

i. 根据Servcie的token取出Service对象 

ii. 调用Service的 onBind() 方法，至此，Service就处于绑定状态了。 

iii. 这时客户端还不知道已经成功连接Service，需要调用客户端的binder对象来调用客户端的ServiceConnection中的 onServiceConnected() 方法，这个通过 ActivityManagerNative.getDefault().publishService() 进行。

/frameworks/base/services/java/com/android/server/am/ActivityManagerService.java:

```java
  public void publishService(IBinder token, Intent intent, IBinder service) {
        // Refuse possible leaked file descriptors
        if (intent != null && intent.hasFileDescriptors() == true) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        synchronized(this) {
            if (!(token instanceof ServiceRecord)) {
                throw new IllegalArgumentException("Invalid service token");
            }
            mServices.publishServiceLocked((ServiceRecord)token, intent, service);
        }
    }
```

AMS的publishService()交给ActivityService对象 mServices 的 publishServiceLocked() 来 处理

/frameworks/base/services/java/com/android/server/am/ActiveServices.java:

```java
void publishServiceLocked(ServiceRecord r, Intent intent, IBinder service) {
        final long origId = Binder.clearCallingIdentity();
       	...
                            try {
                                c.conn.connected(r.name, service);
                            } catch (Exception e) {
                                Slog.w(TAG, "Failure sending service " + r.name +
                                      " to connection " + c.conn.asBinder() +
                                      " (in " + c.binding.client.processName + ")", e);
                            }
        ...
    }
```

核心代码就一句话 c.conn.connected(r.name,service) 。对象c的类型 是 ConnectionRecord ，c.conn就是ServiceDispatcher.InnerConnection对象，service就是 Service的onBind方法返回的Binder对象。

/frameworks/base/core/java/android/app/LoadedApk.java

```java
  private static class InnerConnection extends IServiceConnection.Stub {
            final WeakReference<LoadedApk.ServiceDispatcher> mDispatcher;

            InnerConnection(LoadedApk.ServiceDispatcher sd) {
                mDispatcher = new WeakReference<LoadedApk.ServiceDispatcher>(sd);
            }

            public void connected(ComponentName name, IBinder service) throws RemoteException {
                LoadedApk.ServiceDispatcher sd = mDispatcher.get();
                if (sd != null) {
                    sd.connected(name, service);
                }
            }
        }
```



```java
public void connected(ComponentName name, IBinder service) {
            if (mActivityThread != null) {
                mActivityThread.post(new RunConnection(name, service, 0));
            } else {
                doConnected(name, service);
            }
        }
```

c.conn.connected(r.name,service)内部实现是交给了 mActivityThread.post(new RunnConnection(name ,service,0)); 实现。

ServiceDispatcher的mActivityThread是一个 Handler，其实就是ActivityThread中的H。这样一来RunConnection就经由H的post方法从而运行在主线程中，因此客户端ServiceConnection中的方法是在主线程中被回调 的。

```java
		private final class RunConnection implements Runnable {
            RunConnection(ComponentName name, IBinder service, int command) {
                mName = name;
                mService = service;
                mCommand = command;
            }

            public void run() {
                if (mCommand == 0) {
                    doConnected(mName, mService);
                } else if (mCommand == 1) {
                    doDeath(mName, mService);
                }
            }

            final ComponentName mName;
            final IBinder mService;
            final int mCommand;
		}
```

```java
   public void doConnected(ComponentName name, IBinder service) {
   
   			...	
   				
        // If there is a new service, it is now connected.
        if (service != null) {
            mConnection.onServiceConnected(name, service);
        }
    }
```
RunConnection的定义如下：

i. 继承Runnable接口， run() 方法的实现也是简单调用了ServiceDispatcher 的 doConnected 方法。 

ii. 由于ServiceDispatcher内部保存了客户端的ServiceConntion对象，可以很方便地调 用ServiceConntion对象的 onServiceConnected 方法。 

iii. 客户端的onServiceConnected方法执行后，Service的绑定过程也就完成了。 

iv. service绑定后通过ServiceDispatcher通知客户端的过程可以说明 ServiceDispatcher起着连接ServiceConnection和InnerConnection的作用。 至于Service的停止和解除绑定的过程，系统流程都是类似的。

