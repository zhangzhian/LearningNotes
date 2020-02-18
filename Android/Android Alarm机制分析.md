# Android Alarm机制分析

从应用层到内核层，简单分析Android Alarm的工作流程。基于Android 4.4和Kernel 2.6.39。

### 应用层

 /packages/apps/DeskClock/目录下为Android的系统闹钟APP，可参考其闹钟实现。

在/packages/apps/DeskClock/src/com/android/deskclock/AlarmClockFragment.java：

```java
        mAddAlarmButton = (ImageButton) v.findViewById(R.id.alarm_add_alarm);
        mAddAlarmButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                hideUndoBar(true, null);
                startCreatingAlarm();
            }
        });
```

给按钮绑定监听事件，调用startCreatingAlarm()函数

```java
    private void startCreatingAlarm() {
        // Set the "selected" alarm as null, and we'll create the new one when the timepicker
        // comes back.
        mSelectedAlarm = null;
        AlarmUtils.showTimeEditDialog(getChildFragmentManager(),
                null, AlarmClockFragment.this, DateFormat.is24HourFormat(getActivity()));
    }
```

/packages/apps/DeskClock/src/com/android/deskclock/AlarmUtils.java：

```Java
	public static void showTimeEditDialog(FragmentManager manager, final Alarm alarm,TimePickerDialog.OnTimeSetListener listener, boolean is24HourMode) {

        ...
        TimePickerDialog dialog = TimePickerDialog.newInstance(listener,
                hour, minutes, is24HourMode);
        dialog.setThemeDark(true);

        // Make sure the dialog isn't already added.
        manager.executePendingTransactions();
        final FragmentTransaction ft = manager.beginTransaction();
        final Fragment prev = manager.findFragmentByTag(FRAG_TAG_TIME_PICKER);
        if (prev != null) {
            ft.remove(prev);
        }
        ft.commit();

        if (dialog != null && !dialog.isAdded()) {
            dialog.show(manager, FRAG_TAG_TIME_PICKER);
        }
	}
```

打开了一个时间选择的Dialog，dialog事件监听器是由AlarmClockFragment传入的TimePickerDialog.OnTimeSetListener，回调函数为onTimeSet。

再回到AlarmClockFragment.java中：

```java
/**
 * AlarmClock application.
 */
public class AlarmClockFragment extends DeskClockFragment implements
        LoaderManager.LoaderCallbacks<Cursor>,
        TimePickerDialog.OnTimeSetListener,
        View.OnTouchListener
{
     ...
          // Callback used by TimePickerDialog
    @Override
    public void onTimeSet(RadialPickerLayout view, int hourOfDay, int minute) {
        if (mSelectedAlarm == null) {
            // If mSelectedAlarm is null then we're creating a new alarm.
            Alarm a = new Alarm();
            a.alert = RingtoneManager.getActualDefaultRingtoneUri(getActivity(),
                    RingtoneManager.TYPE_ALARM);
            if (a.alert == null) {
                a.alert = Uri.parse("content://settings/system/alarm_alert");
            }
            a.hour = hourOfDay;
            a.minutes = minute;
            a.enabled = true;
            asyncAddAlarm(a);
        } else {
            mSelectedAlarm.hour = hourOfDay;
            mSelectedAlarm.minutes = minute;
            mSelectedAlarm.enabled = true;
            mScrollToAlarmId = mSelectedAlarm.id;
            asyncUpdateAlarm(mSelectedAlarm, true);
            mSelectedAlarm = null;
        }
    }
    ...
}
```

在onTimeSet函数中 asyncAddAlarm(a);

```java
 private void asyncAddAlarm(final Alarm alarm) {
        final Context context = AlarmClockFragment.this.getActivity().getApplicationContext();
        final AsyncTask<Void, Void, AlarmInstance> updateTask =
                new AsyncTask<Void, Void, AlarmInstance>() {
            @Override
            public synchronized void onPreExecute() {
                ...
            }

            @Override
            protected AlarmInstance doInBackground(Void... parameters) {
                if (context != null && alarm != null) {
                    ContentResolver cr = context.getContentResolver();

                    // Add alarm to db
                    Alarm newAlarm = Alarm.addAlarm(cr, alarm);
                    mScrollToAlarmId = newAlarm.id;

                    // Create and add instance to db
                    if (newAlarm.enabled) {
                        return setupAlarmInstance(context, newAlarm);
                    }
                }
                return null;
            }

            @Override
            protected void onPostExecute(AlarmInstance instance) {
                ...
            }
        };
        updateTask.execute();
    }
```

asyncAddAlarm函数中是一个AsyncTask后台任务，主要关系doInBackground函数。

通过Content Provider在数据库添加了一个数据。

然后调用了

```java
   private static AlarmInstance setupAlarmInstance(Context context, Alarm alarm) {
        ContentResolver cr = context.getContentResolver();
        AlarmInstance newInstance = alarm.createInstanceAfter(Calendar.getInstance());
        newInstance = AlarmInstance.addInstance(cr, newInstance);
        // Register instance to state manager
        AlarmStateManager.registerInstance(context, newInstance, true);
        return newInstance;
    }
```

在/packages/apps/DeskClock/src/com/android/deskclock/provider/Alarm.java中：

```Java
    public static Alarm addAlarm(ContentResolver contentResolver, Alarm alarm) {
        ContentValues values = createContentValues(alarm);
        Uri uri = contentResolver.insert(CONTENT_URI, values);
        alarm.id = getId(uri);
        return alarm;
    }
```

在setupAlarmInstance中，在AlarmStateManager注册了一个实例：

```Java
    // Register instance to state manager
    AlarmStateManager.registerInstance(context, newInstance, true);
```
然后我们来查看:

/packages/apps/DeskClock/src/com/android/deskclock/alarms/AlarmStateManager.java

```java
    /**
     * This registers the AlarmInstance to the state manager. This will look at the instance
     * and choose the most appropriate state to put it in. This is primarily used by new
     * alarms, but it can also be called when the system time changes.
     *
     * Most state changes are handled by the states themselves, but during major time changes we
     * have to correct the alarm instance state. This means we have to handle special cases as
     * describe below:
     *
     * <ul>
     *     <li>Make sure all dismissed alarms are never re-activated</li>
     *     <li>Make sure firing alarms stayed fired unless they should be auto-silenced</li>
     *     <li>Missed instance that have parents should be re-enabled if we went back in time</li>
     *     <li>If alarm was SNOOZED, then show the notification but don't update time</li>
     *     <li>If low priority notification was hidden, then make sure it stays hidden</li>
     * </ul>
     *
     * If none of these special case are found, then we just check the time and see what is the
     * proper state for the instance.
     *
     * @param context application context
     * @param instance to register
     */
    public static void registerInstance(Context context, AlarmInstance instance,
            boolean updateNextAlarm) 
    {
        ...
        // Fix states that are time sensitive
        if (currentTime.after(missedTTL)) {
            // Alarm is so old, just dismiss it
            setDismissState(context, instance);
        } else if (currentTime.after(alarmTime)) {
            setMissedState(context, instance);
        } else if (instance.mAlarmState == AlarmInstance.SNOOZE_STATE) {
            // We only want to display snooze notification and not update the time,
            // so handle showing the notification directly
            AlarmNotifications.showSnoozeNotification(context, instance);
            scheduleInstanceStateChange(context, instance.getAlarmTime(),
                    instance, AlarmInstance.FIRED_STATE);
        }
        ...
    }
```

在scheduleInstanceStateChange函数中：

```Java
  private static void scheduleInstanceStateChange(Context context, Calendar time,
            AlarmInstance instance, int newState) {
        long timeInMillis = time.getTimeInMillis();
        Log.v("Scheduling state change " + newState + " to instance " + instance.mId +
                " at " + AlarmUtils.getFormattedTime(context, time) + " (" + timeInMillis + ")");
        Intent stateChangeIntent = createStateChangeIntent(context, ALARM_MANAGER_TAG, instance,
                newState);
        PendingIntent pendingIntent = PendingIntent.getBroadcast(context, instance.hashCode(),
                stateChangeIntent, PendingIntent.FLAG_UPDATE_CURRENT);

        AlarmManager am = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        if (Utils.isKitKatOrLater()) {
            am.setExact(AlarmManager.RTC_WAKEUP, timeInMillis, pendingIntent);
        } else {
            am.set(AlarmManager.RTC_WAKEUP, timeInMillis, pendingIntent);
        }
    }
```

可以看出AlarmStateManager是系统闹钟APP用来管理闹钟状态的类。

我们可以看到AlarmManager的setExact和set函数。

```java
    if (Utils.isKitKatOrLater()) {
        am.setExact(AlarmManager.RTC_WAKEUP, timeInMillis, pendingIntent);
    } else {
        am.set(AlarmManager.RTC_WAKEUP, timeInMillis, pendingIntent);
    }
```
这里是RTC_WAKEUP, 这就保证了即使系统睡眠了，都能唤醒，闹钟工作。

综上，可以看出系统闹钟APPAlarmManager和开发应用层APP使用的是同样的AlarmManager。

接下来具体分析AlarmManager的实现即可。

### 框架层

#### Java层

在/frameworks/base/core/java/android/app/AlarmManager.java中：

```java
public class AlarmManager
{
    private static final String TAG = "AlarmManager";

    /**
     * Alarm time in {@link System#currentTimeMillis System.currentTimeMillis()}
     * (wall clock time in UTC), which will wake up the device when
     * it goes off.
     */
    public static final int RTC_WAKEUP = 0;
    /**
     * Alarm time in {@link System#currentTimeMillis System.currentTimeMillis()}
     * (wall clock time in UTC).  This alarm does not wake the
     * device up; if it goes off while the device is asleep, it will not be
     * delivered until the next time the device wakes up.
     */
    public static final int RTC = 1;
    /**
     * Alarm time in {@link android.os.SystemClock#elapsedRealtime
     * SystemClock.elapsedRealtime()} (time since boot, including sleep),
     * which will wake up the device when it goes off.
     */
    public static final int ELAPSED_REALTIME_WAKEUP = 2;
    /**
     * Alarm time in {@link android.os.SystemClock#elapsedRealtime
     * SystemClock.elapsedRealtime()} (time since boot, including sleep).
     * This alarm does not wake the device up; if it goes off while the device
     * is asleep, it will not be delivered until the next time the device
     * wakes up.
     */
    public static final int ELAPSED_REALTIME = 3;

    ...
    
    /**
     * TBW: new 'exact' alarm that must be delivered as nearly as possible
     * to the precise time specified.
     */
    public void setExact(int type, long triggerAtMillis, PendingIntent operation) {
        setImpl(type, triggerAtMillis, WINDOW_EXACT, 0, operation, null);
    }

    /** @hide */
    public void set(int type, long triggerAtMillis, long windowMillis, long intervalMillis,
            PendingIntent operation, WorkSource workSource) {
        setImpl(type, triggerAtMillis, windowMillis, intervalMillis, operation, workSource);
    }

    private void setImpl(int type, long triggerAtMillis, long windowMillis, long intervalMillis,
            PendingIntent operation, WorkSource workSource) {
        if (triggerAtMillis < 0) {
            /* NOTYET
            if (mAlwaysExact) {
                // Fatal error for KLP+ apps to use negative trigger times
                throw new IllegalArgumentException("Invalid alarm trigger time "
                        + triggerAtMillis);
            }
            */
            triggerAtMillis = 0;
        }

        try {
            mService.set(type, triggerAtMillis, windowMillis, intervalMillis, operation,
                    workSource);
        } catch (RemoteException ex) {
        }
    }


    ...
}
```

主要分为4中模式：

- **RTC_WAKEUP**：UTC时间，会在设备关闭时唤醒它

- **RTC**：UTC时间，此警报不会唤醒设备；如果它在设备处于休眠状态时关闭，则在下次设备唤醒之前不会发送。

- **ELAPSED_REALTIME_WAKEUP**：开机后的时间，包括睡眠，会在设备关闭时唤醒它

- **ELAPSED_REALTIME**：开机后的时间，包括睡眠；如果它在设备处于休眠状态时关闭，则在下次设备唤醒之前不会发送。

setExact和set均调用了setImpl函数，setImpl函数中核心是：

```java
 mService.set(type, triggerAtMillis, windowMillis, intervalMillis, operation,
                    workSource);
```

查看mService的定义：

```java
private final IAlarmManager mService;
```

在/frameworks/base/core/java/android/app/IAlarmManager.aidl：

```java
/**
 * System private API for talking with the alarm manager service.
 *
 * {@hide}
 */
interface IAlarmManager {
	/** windowLength == 0 means exact; windowLength < 0 means the let the OS decide */
    void set(int type, long triggerAtTime, long windowLength,
            long interval, in PendingIntent operation, in WorkSource workSource);
    void setTime(long millis);
    void setTimeZone(String zone);
    void remove(in PendingIntent operation);
}
```

可以看到是一个aidl文件，要用到进程间通信技术。

查找具体实现，在/frameworks/base/services/java/com/android/server/AlarmManagerService.java：

```Java
class AlarmManagerService extends IAlarmManager.Stub {
    
	...
       
    @Override
    public void set(int type, long triggerAtTime, long windowLength, long interval,
            PendingIntent operation, WorkSource workSource) {
        if (workSource != null) {
            mContext.enforceCallingPermission(
                    android.Manifest.permission.UPDATE_DEVICE_STATS,
                    "AlarmManager.set");
        }

        set(type, triggerAtTime, windowLength, interval, operation, false, workSource);
    }
    
     public void set(int type, long triggerAtTime, long windowLength, long interval,
            PendingIntent operation, boolean isStandalone, WorkSource workSource) {
    	
        ... 
        
        final long nowElapsed = SystemClock.elapsedRealtime();
        final long triggerElapsed = convertToElapsed(triggerAtTime, type);
        final long maxElapsed;
        if (windowLength == AlarmManager.WINDOW_EXACT) {
            maxElapsed = triggerElapsed;
        } else if (windowLength < 0) {
            maxElapsed = maxTriggerTime(nowElapsed, triggerElapsed, interval);
        } else {
            maxElapsed = triggerElapsed + windowLength;
        }

        synchronized (mLock) {
            if (DEBUG_BATCH) {
                Slog.v(TAG, "set(" + operation + ") : type=" + type
                        + " triggerAtTime=" + triggerAtTime + " win=" + windowLength
                        + " tElapsed=" + triggerElapsed + " maxElapsed=" + maxElapsed
                        + " interval=" + interval + " standalone=" + isStandalone);
            }
            setImplLocked(type, triggerAtTime, triggerElapsed, maxElapsed,
                    interval, operation, isStandalone, true, workSource);
        }
    }
   
  	 private void setImplLocked(int type, long when, long whenElapsed, long maxWhen, long interval,PendingIntent operation, boolean isStandalone, boolean doValidate,
            WorkSource workSource) {
    
         ...
         if (reschedule) {
            rescheduleKernelAlarmsLocked();
         }    
         ...
         
    }
    
    private void rescheduleKernelAlarmsLocked() {
        // Schedule the next upcoming wakeup alarm.  If there is a deliverable batch
        // prior to that which contains no wakeups, we schedule that as well.
        if (mAlarmBatches.size() > 0) {
            final Batch firstWakeup = findFirstWakeupBatchLocked();
            final Batch firstBatch = mAlarmBatches.get(0);
            if (firstWakeup != null && mNextWakeup != firstWakeup.start) {
                mNextWakeup = firstWakeup.start;
                setLocked(ELAPSED_REALTIME_WAKEUP, firstWakeup.start);
            }
            if (firstBatch != firstWakeup && mNextNonWakeup != firstBatch.start) {
                mNextNonWakeup = firstBatch.start;
                setLocked(ELAPSED_REALTIME, firstBatch.start);
            }
        }
    }    
    
	private void setLocked(int type, long when)
    {
        if (mDescriptor != -1)
        {
            // The kernel never triggers alarms with negative wakeup times
            // so we ensure they are positive.
            long alarmSeconds, alarmNanoseconds;
            if (when < 0) {
                alarmSeconds = 0;
                alarmNanoseconds = 0;
            } else {
                alarmSeconds = when / 1000;
                alarmNanoseconds = (when % 1000) * 1000 * 1000;
            }

            set(mDescriptor, type, alarmSeconds, alarmNanoseconds);
        }
        else
        {
            Message msg = Message.obtain();
            msg.what = ALARM_EVENT;

            mHandler.removeMessages(ALARM_EVENT);
            mHandler.sendMessageAtTime(msg, when);
        }
    }
    
	...
        
    private native int init();
    private native void close(int fd);
    private native void set(int fd, int type, long seconds, long nanoseconds);
    private native int waitForAlarm(int fd);
    private native int setKernelTimezone(int fd, int minuteswest);
    
    ...
            
}
```

通过上述代码，AlarmManagerService的set函数最终调用了native的set函数进行设置。

同时，注意到AlarmManagerService中含有一个AlarmThread内部类：

```java
  private class AlarmThread extends Thread
    {
        public AlarmThread()
        {
            super("AlarmManager");
        }

        public void run()
        {
            ArrayList<Alarm> triggerList = new ArrayList<Alarm>();

            while (true)
            {
                int result = waitForAlarm(mDescriptor);

                triggerList.clear();

                if ((result & TIME_CHANGED_MASK) != 0) {
                    if (DEBUG_BATCH) {
                        Slog.v(TAG, "Time changed notification from kernel; rebatching");
                    }
                    remove(mTimeTickSender);
                    rebatchAllAlarms();
                    mClockReceiver.scheduleTimeTickEvent();
                    Intent intent = new Intent(Intent.ACTION_TIME_CHANGED);
                    intent.addFlags(Intent.FLAG_RECEIVER_REPLACE_PENDING
                            | Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT);
                    mContext.sendBroadcastAsUser(intent, UserHandle.ALL);
                }

                synchronized (mLock) {
                    final long nowRTC = System.currentTimeMillis();
                    final long nowELAPSED = SystemClock.elapsedRealtime();
                    if (localLOGV) Slog.v(
                        TAG, "Checking for alarms... rtc=" + nowRTC
                        + ", elapsed=" + nowELAPSED);

                    if (WAKEUP_STATS) {
                        if ((result & IS_WAKEUP_MASK) != 0) {
                            long newEarliest = nowRTC - RECENT_WAKEUP_PERIOD;
                            int n = 0;
                            for (WakeupEvent event : mRecentWakeups) {
                                if (event.when > newEarliest) break;
                                n++; // number of now-stale entries at the list head
                            }
                            for (int i = 0; i < n; i++) {
                                mRecentWakeups.remove();
                            }

                            recordWakeupAlarms(mAlarmBatches, nowELAPSED, nowRTC);
                        }
                    }

                    triggerAlarmsLocked(triggerList, nowELAPSED, nowRTC);
                    rescheduleKernelAlarmsLocked();

                    // now deliver the alarm intents
                    for (int i=0; i<triggerList.size(); i++) {
                        Alarm alarm = triggerList.get(i);
                        try {
                            if (localLOGV) Slog.v(TAG, "sending alarm " + alarm);
                            alarm.operation.send(mContext, 0,
                                    mBackgroundIntent.putExtra(
                                            Intent.EXTRA_ALARM_COUNT, alarm.count),
                                    mResultReceiver, mHandler);

                            ...
                            
                        } catch (PendingIntent.CanceledException e) {
                            if (alarm.repeatInterval > 0) {
                                // This IntentSender is no longer valid, but this
                                // is a repeating alarm, so toss the hoser.
                                remove(alarm.operation);
                            }
                        } catch (RuntimeException e) {
                            Slog.w(TAG, "Failure sending alarm.", e);
                        }
                    }
                }
            }
        }
    }
```

AlarmManagerService  中 AlarmThread  会一直调用waitForAlarm轮询，如果打开失败就直接返回，成功就会做一些动作，重新计算时间，发送广播等。

#### Native层

/frameworks/base/services/jni/com_android_server_AlarmManagerService.cpp

```c
namespace android {

static jint android_server_AlarmManagerService_setKernelTimezone(JNIEnv* env, jobject obj, jint fd, jint minswest)
{
    struct timezone tz;

    tz.tz_minuteswest = minswest;
    tz.tz_dsttime = 0;

    int result = settimeofday(NULL, &tz);
    if (result < 0) {
        ALOGE("Unable to set kernel timezone to %d: %s\n", minswest, strerror(errno));
        return -1;
    } else {
        ALOGD("Kernel timezone updated to %d minutes west of GMT\n", minswest);
    }

    return 0;
}

static jint android_server_AlarmManagerService_init(JNIEnv* env, jobject obj)
{
    return open("/dev/alarm", O_RDWR);
}

static void android_server_AlarmManagerService_close(JNIEnv* env, jobject obj, jint fd)
{
	close(fd);
}

static void android_server_AlarmManagerService_set(JNIEnv* env, jobject obj, jint fd, jint type, jlong seconds, jlong nanoseconds)
{
    struct timespec ts;
    ts.tv_sec = seconds;
    ts.tv_nsec = nanoseconds;

	int result = ioctl(fd, ANDROID_ALARM_SET(type), &ts);
	if (result < 0)
	{
        ALOGE("Unable to set alarm to %lld.%09lld: %s\n", seconds, nanoseconds, strerror(errno));
    }
}

static jint android_server_AlarmManagerService_waitForAlarm(JNIEnv* env, jobject obj, jint fd)
{
	int result = 0;

	do
	{
		result = ioctl(fd, ANDROID_ALARM_WAIT);
	} while (result < 0 && errno == EINTR);

	if (result < 0)
	{
        ALOGE("Unable to wait on alarm: %s\n", strerror(errno));
        return 0;
    }

    return result;
}

static JNINativeMethod sMethods[] = {
     /* name, signature, funcPtr */
	{"init", "()I", (void*)android_server_AlarmManagerService_init},
	{"close", "(I)V", (void*)android_server_AlarmManagerService_close},
	{"set", "(IIJJ)V", (void*)android_server_AlarmManagerService_set},
    {"waitForAlarm", "(I)I", (void*)android_server_AlarmManagerService_waitForAlarm},
    {"setKernelTimezone", "(II)I", (void*)android_server_AlarmManagerService_setKernelTimezone},
};

int register_android_server_AlarmManagerService(JNIEnv* env)
{
    return jniRegisterNativeMethods(env, "com/android/server/AlarmManagerService",
                                    sMethods, NELEM(sMethods));
}

} /* namespace android */
```

android_server_AlarmManagerService_set中关键语句：

```c
int result = ioctl(fd, ANDROID_ALARM_SET(type), &ts);
```

在计算机中，ioctl(input/output control)是一个专用于设备输入输出操作的系统调用,该调用传入一个跟设备有关的请求码，系统调用的功能完全取决于请求码。

[参考ioctl介绍博文](https://blog.csdn.net/qq_19923217/article/details/82698787)

接下来我们需要查看内核层的处理方式，ANDROID_ALARM_SET为关键字查找。

### 内核层

在/drivers/rtc/alarm-dev.c：

```c
static long alarm_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
	...
        
	case ANDROID_ALARM_SET(0):
		if (copy_from_user(&new_alarm_time, (void __user *)arg,
		    sizeof(new_alarm_time))) {
			rv = -EFAULT;
			goto err1;
		}
from_old_alarm_set:
		spin_lock_irqsave(&alarm_slock, flags);
		pr_alarm(IO, "alarm %d set %ld.%09ld\n", alarm_type,
			new_alarm_time.tv_sec, new_alarm_time.tv_nsec);
		alarm_enabled |= alarm_type_mask;
		alarm_start_range(&alarms[alarm_type],
			timespec_to_ktime(new_alarm_time),
			timespec_to_ktime(new_alarm_time));
		spin_unlock_irqrestore(&alarm_slock, flags);
		if (ANDROID_ALARM_BASE_CMD(cmd) != ANDROID_ALARM_SET_AND_WAIT(0)
		    && cmd != ANDROID_ALARM_SET_AND_WAIT_OLD)
			break;
    
    case ANDROID_ALARM_WAIT:
		spin_lock_irqsave(&alarm_slock, flags);
		pr_alarm(IO, "alarm wait\n");
		if (!alarm_pending && wait_pending) {
			wake_unlock(&alarm_wake_lock);
			wait_pending = 0;
		}
		spin_unlock_irqrestore(&alarm_slock, flags);
		rv = wait_event_interruptible(alarm_wait_queue, alarm_pending);
		if (rv)
			goto err1;
		spin_lock_irqsave(&alarm_slock, flags);
		rv = alarm_pending;
		wait_pending = 1;
		alarm_pending = 0;
		spin_unlock_irqrestore(&alarm_slock, flags);
		break;
    
	...
        
	case ANDROID_ALARM_SET_RTC:
		if (copy_from_user(&new_rtc_time, (void __user *)arg,
		    sizeof(new_rtc_time))) {
			rv = -EFAULT;
			goto err1;
		}
		rv = alarm_set_rtc(new_rtc_time);
		spin_lock_irqsave(&alarm_slock, flags);
		alarm_pending |= ANDROID_ALARM_TIME_CHANGE_MASK;
		wake_up(&alarm_wait_queue);
		spin_unlock_irqrestore(&alarm_slock, flags);
		if (rv < 0)
			goto err1;
		break;
	
    ...

	default:
		rv = -EINVAL;
		goto err1;
	}
err1:
	return rv;
}
```

调用了alarm_start_range  设置闹钟， alarm_set_rtc设置RTC
这两个函数在 android_alarm.h 声明，在 alarm.c 里实现

/include/linux/android_alarm.h:

```c
#ifndef _LINUX_ANDROID_ALARM_H
#define _LINUX_ANDROID_ALARM_H

#include <linux/ioctl.h>
#include <linux/time.h>

enum android_alarm_type {
	/* return code bit numbers or set alarm arg */
	ANDROID_ALARM_RTC_WAKEUP,
	ANDROID_ALARM_RTC,
	ANDROID_ALARM_ELAPSED_REALTIME_WAKEUP,
	ANDROID_ALARM_ELAPSED_REALTIME,
	ANDROID_ALARM_SYSTEMTIME,

	ANDROID_ALARM_TYPE_COUNT,

	/* return code bit numbers */
	/* ANDROID_ALARM_TIME_CHANGE = 16 */
};

#ifdef __KERNEL__

#include <linux/ktime.h>
#include <linux/rbtree.h>

/*
 * The alarm interface is similar to the hrtimer interface but adds support
 * for wakeup from suspend. It also adds an elapsed realtime clock that can
 * be used for periodic timers that need to keep runing while the system is
 * suspended and not be disrupted when the wall time is set.
 */

/**
 * struct alarm - the basic alarm structure
 * @node:	red black tree node for time ordered insertion
 * @type:	alarm type. rtc/elapsed-realtime/systemtime, wakeup/non-wakeup.
 * @softexpires: the absolute earliest expiry time of the alarm.
 * @expires:	the absolute expiry time.
 * @function:	alarm expiry callback function
 *
 * The alarm structure must be initialized by alarm_init()
 *
 */

struct alarm {
	struct rb_node 		node;
	enum android_alarm_type type;
	ktime_t			softexpires;
	ktime_t			expires;
	void			(*function)(struct alarm *);
};

void alarm_init(struct alarm *alarm,
	enum android_alarm_type type, void (*function)(struct alarm *));
void alarm_start_range(struct alarm *alarm, ktime_t start, ktime_t end);
int alarm_try_to_cancel(struct alarm *alarm);
int alarm_cancel(struct alarm *alarm);
ktime_t alarm_get_elapsed_realtime(void);

/* set rtc while preserving elapsed realtime */
int alarm_set_rtc(const struct timespec ts);

#endif

enum android_alarm_return_flags {
	ANDROID_ALARM_RTC_WAKEUP_MASK = 1U << ANDROID_ALARM_RTC_WAKEUP,
	ANDROID_ALARM_RTC_MASK = 1U << ANDROID_ALARM_RTC,
	ANDROID_ALARM_ELAPSED_REALTIME_WAKEUP_MASK =
				1U << ANDROID_ALARM_ELAPSED_REALTIME_WAKEUP,
	ANDROID_ALARM_ELAPSED_REALTIME_MASK =
				1U << ANDROID_ALARM_ELAPSED_REALTIME,
	ANDROID_ALARM_SYSTEMTIME_MASK = 1U << ANDROID_ALARM_SYSTEMTIME,
	ANDROID_ALARM_TIME_CHANGE_MASK = 1U << 16
};

/* Disable alarm */
#define ANDROID_ALARM_CLEAR(type)           _IO('a', 0 | ((type) << 4))

/* Ack last alarm and wait for next */
#define ANDROID_ALARM_WAIT                  _IO('a', 1)

#define ALARM_IOW(c, type, size)            _IOW('a', (c) | ((type) << 4), size)
/* Set alarm */
#define ANDROID_ALARM_SET(type)             ALARM_IOW(2, type, struct timespec)
#define ANDROID_ALARM_SET_AND_WAIT(type)    ALARM_IOW(3, type, struct timespec)
#define ANDROID_ALARM_GET_TIME(type)        ALARM_IOW(4, type, struct timespec)
#define ANDROID_ALARM_SET_RTC               _IOW('a', 5, struct timespec)
#define ANDROID_ALARM_BASE_CMD(cmd)         (cmd & ~(_IOC(0, 0, 0xf0, 0)))
#define ANDROID_ALARM_IOCTL_TO_TYPE(cmd)    (_IOC_NR(cmd) >> 4)

#endif
```

/drivers/rtc/alarm.c:

```c
/**
 * alarm_start_range - (re)start an alarm
 * @alarm:	the alarm to be added
 * @start:	earliest expiry time
 * @end:	expiry time
 */
void alarm_start_range(struct alarm *alarm, ktime_t start, ktime_t end)
{
	unsigned long flags;

	spin_lock_irqsave(&alarm_slock, flags);
	alarm->softexpires = start;
	alarm->expires = end;
	alarm_enqueue_locked(alarm);
	spin_unlock_irqrestore(&alarm_slock, flags);
}

...

/**
 * alarm_set_rtc - set the kernel and rtc walltime
 * @new_time:	timespec value containing the new time
 */
int alarm_set_rtc(struct timespec new_time)
{
	...
	ret = rtc_set_time(alarm_rtc_dev, &rtc_new_rtc_time);
    ...
	return ret;
}

static int alarm_suspend(struct platform_device *pdev, pm_message_t state)


static int alarm_resume(struct platform_device *pdev)

```

xref: /drivers/rtc/interface.c:

```c
int rtc_set_time(struct rtc_device *rtc, struct rtc_time *tm)
{
	int err;

	err = rtc_valid_tm(tm);
	if (err != 0)
		return err;

	err = mutex_lock_interruptible(&rtc->ops_lock);
	if (err)
		return err;

	if (!rtc->ops)
		err = -ENODEV;
	else if (rtc->ops->set_time)
		err = rtc->ops->set_time(rtc->dev.parent, tm);
	else if (rtc->ops->set_mmss) {
		unsigned long secs;
		err = rtc_tm_to_time(tm, &secs);
		if (err == 0)
			err = rtc->ops->set_mmss(rtc->dev.parent, secs);
	} else
		err = -EINVAL;

	mutex_unlock(&rtc->ops_lock);
	return err;
}
EXPORT_SYMBOL_GPL(rtc_set_time);
```

在 rtc->ops->set_time(rtc->dev.parent, tm)中set_time 就看到具体的是那个RTC芯片，这里不再继续分析。

alarm.c  里面实现了 alarm_suspend  alarm_resume 函数。如果不需要唤醒系统，设置闹钟并不会往rtc 芯片的寄存器上写数据，通过上层写到设备文件/dev/alarm 里面，AlarmThread 会不停的去轮寻下一个时间有没有闹钟，直接从设备文件 /dev/alarm 里面读取。

如果需要唤醒系统，alarm 的alarm_suspend就会写到下层的rtc芯片的寄存器上去， 然后即使系统suspend之后，闹钟通过rtc 也能唤醒系统。

