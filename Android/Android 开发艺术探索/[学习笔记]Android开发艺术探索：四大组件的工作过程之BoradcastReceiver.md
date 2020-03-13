# [学习笔记]Android开发艺术探索：四大组件的工作过程之BoradcastReceiver

## 广播的注册过程

静态注册：在应用的安装时由系统自动完成注册，具体来说是PMS（PackageManagerServer）来完成整个注册过程。其他三大组件也是。

动态注册：从ContentWrapper的registerReceiver方法开始， 调用了自己的registerReceiverInternal方法。

```java
		private Intent registerReceiverInternal(BroadcastReceiver receiver, int userId,
            IntentFilter filter, String broadcastPermission,
            Handler scheduler, Context context) {
        IIntentReceiver rd = null;
        if (receiver != null) {
            if (mPackageInfo != null && context != null) {
                if (scheduler == null) {
                    scheduler = mMainThread.getHandler();
                }
                rd = mPackageInfo.getReceiverDispatcher(
                    receiver, context, scheduler,
                    mMainThread.getInstrumentation(), true);
            } else {
                if (scheduler == null) {
                    scheduler = mMainThread.getHandler();
                }
                rd = new LoadedApk.ReceiverDispatcher(
                        receiver, context, scheduler, null, true).getIIntentReceiver();
            }
        }
        try {
            return ActivityManagerNative.getDefault().registerReceiver(
                    mMainThread.getApplicationThread(), mBasePackageName,
                    rd, filter, broadcastPermission, userId);
        } catch (RemoteException e) {
            return null;
        }
    } 
```

系统首先从mPackageInfo获取IIntentReceiver对象，然后采用跨进程的方式向AMS发送广播注册的请求。所以采用IIntentReceiver而不是直接采用BroadCastReceiver是因为这个过程是一个进程间通信的过程，他BroadCastReceiver是一个Android的组件是不能跨进程的，所以需要IIntentReceiver来中转一下，毫无疑问，IIntentReceiver必然是一个Binder接口，他的具体实现是LoadedApk.ReceiverDispatcher.IIntentReceiver,ReceiverDispatcher内部同时保存了BoradcastReceiver和IIntentReceiver。

ReceiverDispatcher的getIIntentReceiver的实现：

```java
		public IIntentReceiver getReceiverDispatcher(BroadcastReceiver r,
            Context context, Handler handler,
            Instrumentation instrumentation, boolean registered) {
        synchronized (mReceivers) {
            LoadedApk.ReceiverDispatcher rd = null;
            ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher> map = null;
            if (registered) {
                map = mReceivers.get(context);
                if (map != null) {
                    rd = map.get(r);
                }
            }
            if (rd == null) {
                rd = new ReceiverDispatcher(r, context, handler,
                        instrumentation, registered);
                if (registered) {
                    if (map == null) {
                        map = new ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher>();
                        mReceivers.put(context, map);
                    }
                    map.put(r, rd);
                }
            } else {
                rd.validate(context, handler);
            }
            rd.mForgotten = false;
            return rd.getIIntentReceiver();
        }
    } 
```

getReceiverDispatcher方法重新创建了一个ReceiverDispatcher对象并将球保持的InnerReceiver对象返回，其中InnerReceiver对象和BroadcastReceiver都是在ReceiverDispatcher的构造方法中被保存起来的。

注册广播的真正实现是在AMS，AMS的registerReceiver方法：

```java
		public Intent registerReceiver(IApplicationThread caller, String callerPackage,
            IIntentReceiver receiver, IntentFilter filter, String permission, int userId) {
          	
  					...					
  
            BroadcastFilter bf = new BroadcastFilter(filter, rl, callerPackage,
                    permission, callingUid, userId);
            rl.add(bf);
            if (!bf.debugCheck()) {
                Slog.w(TAG, "==> For Dynamic broadast");
            }
            mReceiverResolver.addFilter(bf);

            ...
        }
    }
```



## 广播的发送和接收过程

当send发送之后，AMS会查找出匹配的广播接收器并且将广播交给他去处理，关闭的发送有几种：普通广播，有序广播 和 粘性广播，他们的实现方式有一些不一样，但是过程都是一样的。

广播的发送和接收，可以说是两个阶段。这里从发射的广播说起，广播的发送仍然开始于ContextWrapper的sendBroadcast，之所以不是Context,那是因为Context的sendBroadcast是一个抽象的方法，和广播的注册过程一样，他也什么都没有做，也是ContextImpl去处理的。

 ```java
  	@Override
    public void sendBroadcast(Intent intent) {
        warnIfCallingFromSystemProcess();
        String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());
        try {
            intent.prepareToLeaveProcess();
            ActivityManagerNative.getDefault().broadcastIntent(
                mMainThread.getApplicationThread(), intent, resolvedType, null,
                Activity.RESULT_OK, null, null, null, AppOpsManager.OP_NONE, false, false,
                getUserId());
        } catch (RemoteException e) {
        }
    }
 ```

这段代码里，直接异步向AMS发送一个broadcastIntent

```java
		public final int broadcastIntent(IApplicationThread caller,
            Intent intent, String resolvedType, IIntentReceiver resultTo,
            int resultCode, String resultData, Bundle map,
            String requiredPermission, int appOp, boolean serialized, boolean sticky, int userId) {
        enforceNotIsolatedCaller("broadcastIntent");
        synchronized(this) {
            intent = verifyBroadcastLocked(intent);

            final ProcessRecord callerApp = getRecordForAppLocked(caller);
            final int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            int res = broadcastIntentLocked(callerApp,
                    callerApp != null ? callerApp.info.packageName : null,
                    intent, resolvedType, resultTo,
                    resultCode, resultData, map, requiredPermission, appOp, serialized, sticky,
                    callingPid, callingUid, userId);
            Binder.restoreCallingIdentity(origId);
            return res;
        }
    }
```

这里随即又调用了一个broadcastIntentLocked，这个方法就有点复杂了。在这个方法里最开始有这么一行

```java
 		// By default broadcasts do not go to stopped apps.
   intent.addFlags(Intent.FLAG_EXCLUDE_STOPPED_PACKAGES);
```

在Android5.0中，默认情况下广播不会发送给已经停止的应用，其实在3.1就已经有这种特性，在broadcastIntentLocked内部，会根据intent-filter来匹配并且进行广播过滤，最终将满足的广播添加到BrocastQueue中，接着就发送给对应的广播接收者了。源码如下：

```java
 				int NR = registeredReceivers != null ? registeredReceivers.size() : 0;
        if (!ordered && NR > 0) {
            // If we are not serializing this broadcast, then send the
            // registered receivers separately so they don't wait for the
            // components to be launched.
            final BroadcastQueue queue = broadcastQueueForIntent(intent);
            BroadcastRecord r = new BroadcastRecord(queue, intent, callerApp,
                    callerPackage, callingPid, callingUid, resolvedType, requiredPermission,
                    appOp, registeredReceivers, resultTo, resultCode, resultData, map,
                    ordered, sticky, false, userId);
            if (DEBUG_BROADCAST) Slog.v(
                    TAG, "Enqueueing parallel broadcast " + r);
            final boolean replaced = replacePending && queue.replaceParallelBroadcastLocked(r);
            if (!replaced) {
                queue.enqueueParallelBroadcastLocked(r);
                queue.scheduleBroadcastsLocked();
            }
            registeredReceivers = null;
            NR = 0;
        }
```

下面我们看一下BroadcastQueue中广播的发送过程

```java
 public void scheduleBroadcastsLocked() {
        if (DEBUG_BROADCAST) Slog.v(TAG, "Schedule broadcasts ["
                + mQueueName + "]: current="
                + mBroadcastsScheduled);

        if (mBroadcastsScheduled) {
            return;
        }
        mHandler.sendMessage(mHandler.obtainMessage(BROADCAST_INTENT_MSG, this));
        mBroadcastsScheduled = true;
    }
```

就是发了一个Handler

```java
  					// First, deliver any non-serialized broadcasts right away.
            while (mParallelBroadcasts.size() > 0) {
                r = mParallelBroadcasts.remove(0);
                r.dispatchTime = SystemClock.uptimeMillis();
                r.dispatchClockTime = System.currentTimeMillis();
                final int N = r.receivers.size();
                if (DEBUG_BROADCAST_LIGHT) Slog.v(TAG, "Processing parallel broadcast ["
                        + mQueueName + "] " + r);
                for (int i=0; i<N; i++) {
                    Object target = r.receivers.get(i);
                    if (DEBUG_BROADCAST)  Slog.v(TAG,
                            "Delivering non-ordered on [" + mQueueName + "] to registered "
                            + target + ": " + r);
                    deliverToRegisteredReceiverLocked(r, (BroadcastFilter)target, false);
                }
                addBroadcastToHistoryLocked(r);
                if (DEBUG_BROADCAST_LIGHT) Slog.v(TAG, "Done with parallel broadcast ["
                        + mQueueName + "] " + r);
            }
```

在这段代码中，会创建一个Args对象并且通过post方式执行Args的逻辑，而Args实现了Runnable的接口，在这里面有这么一段代码

```java
		final BroadcastReceiver receiver = mReceiver;
    receiver.setPendingResult(this);
    receiver.onReceiver(mContext,intent);
```

这个时候onReceiver被执行了，应用也就收到了广播了，这就是整个广播的工作过程。









