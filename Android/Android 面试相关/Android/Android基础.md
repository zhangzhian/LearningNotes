# Android基础

## 一、Activity全方位解析

### 1. Activity的生命周期

[![img](https://camo.githubusercontent.com/60c22238203b2a404df90a8803d6902f31651e2d/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d613263616130386433636263613030332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/60c22238203b2a404df90a8803d6902f31651e2d/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d613263616130386433636263613030332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)



在正常情况下，一个Activity从启动到结束会以如下顺序经历整个生命周期：
onCreate()->onStart()->onResume()->onPause()->onStop()->onDestory()。包含了六个部分，还有一个onRestart()没有调用，下面我们一一介绍这七部分内容。

(1) onCreate()：当 Activity 第一次创建时会被调用。这是生命周期的第一个方法。在这个方法中，可以做一些初始化工作，比如调用setContentView去加载界面布局资源，初始化Activity所需的数据。当然也可借助onCreate()方法中的Bundle对象来回复异常情况下Activity结束时的状态（后面会介绍）。

(2) onRestart()：表示Activity正在重新启动。一般情况下，当当前Activity从不可见重新变为可见状态时，onRestart就会被调用。这种情形一般是用户行为导致的，比如用户按Home键切换到桌面或打开了另一个新的Activity，接着用户又回到了这个Actvity。（关于这部分生命周期的历经过程，后面会介绍。）

(3) onStart(): 表示Activity正在被启动，即将开始，这时Activity已经**出现**了，但是还没有出现在前台，无法与用户交互。这个时候可以理解为**Activity已经显示出来，但是我们还看不到。**

(4) onResume():表示Activity**已经可见了，并且出现在前台并开始活动**。需要和onStart()对比，onStart的时候Activity还在后台，onResume的时候Activity才显示到前台。

(5) onPause():表示 Activity正在停止，仍可见，正常情况下，紧接着onStop就会被调用。**onPause中不能进行耗时操作，会影响到新Activity的显示。因为onPause必须执行完，新的Activity的onResume才会执行。**

(6) onStop():表示Activity即将停止，不可见，位于后台。可以做稍微重量级的回收工作，同样不能太耗时。

(7) onDestory():表示Activity即将销毁，这是Activity生命周期的最后一个回调，可以做一些回收工作和最终的资源回收。

在平常的开发中，我们经常用到的就是 onCreate()和onDestory()，做一些初始化和回收操作。

**生命周期的几种普通情况**

①针对一个特定的Activity，第一次启动，回调如下：onCreate()->onStart()->onResume()

②用户打开新的Activiy的时候，上述Activity的回调如下：onPause()->onStop()

③再次回到原Activity时，回调如下：onRestart()->onStart()->onResume()

④按back键回退时，回调如下：onPause()->onStop()->onDestory()

⑤按Home键切换到桌面后又回到该Actitivy，回调如下：onPause()->onStop()->onRestart()->onStart()->onResume()

⑥调用finish()方法后，回调如下：onDestory()**(以在onCreate()方法中调用为例，不同方法中回调不同，通常都是在onCreate()方法中调用)**

**特殊情况下的生命周期**

在一些特殊情况下，Activity的生命周期的经历有些异常，下面就是两种特殊情况。

**①横竖屏切换**

在横竖屏切换的过程中，会发生Activity被销毁并重建的过程。

在了解这种情况下的生命周期时，首先应该了解这两个回调：**onSaveInstanceState和onRestoreInstanceState**。

在Activity由于异常情况下终止时，系统会调用onSaveInstanceState来保存当前Activity的状态。这个方法的调用是在onStop之前，它和onPause没有既定的时序关系，该方法只在Activity被异常终止的情况下调用。当异常终止的Activity被重建以后，系统会调用onRestoreInstanceState，并且把Activity销毁时onSaveInstanceState方法所保存的Bundle对象参数同时传递给onRestoreInstanceState和onCreate方法。因此，可以通过onRestoreInstanceState方法来恢复Activity的状态，该方法的调用时机是在onStart之后。**其中onCreate和onRestoreInstanceState方法来恢复Activity的状态的区别：** onRestoreInstanceState回调则表明其中Bundle对象非空，不用加非空判断，onRestoreInstanceState在onStart后调用。onCreate需要非空判断。

[![img](https://camo.githubusercontent.com/720be4bf6af0852a0a1d19bb2b7efe1d772e1040/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d323364393034373166613766313264322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/720be4bf6af0852a0a1d19bb2b7efe1d772e1040/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d323364393034373166613766313264322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

横竖屏切换的生命周期：`onPause()->onSaveInstanceState()-> onStop()->onDestroy()->onCreate()->onStart()->onRestoreInstanceState->onResume()`

可以通过在AndroidManifest文件的Activity中指定如下属性：

```xml
android:configChanges = "orientation| screenSize"
```

来避免横竖屏切换时，Activity的销毁和重建，而是回调了下面的方法：

```java
	@Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
    }
```

**②资源内存不足导致优先级低的Activity被杀死**
Activity优先级的划分和下面的Activity的三种运行状态是对应的。

(1) 前台Activity——正在和用户交互的Activity，优先级最高。

(2) 可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户交互。

(3) 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。

当系统内存不足时，会按照上述优先级从低到高去杀死目标Activity所在的进程。我们在平常使用手机时，能经常感受到这一现象。这种情况下数组存储和恢复过程和上述情况一致，生命周期情况也一样。

**Activity的三种运行状态**

①Resumed（活动状态）

又叫Running状态，这个Activity正在屏幕上显示，并且有用户焦点。这个很好理解，就是用户正在操作的那个界面。

②Paused（暂停状态）

这是一个比较不常见的状态。这个Activity在屏幕上是可见的，但是并不是在屏幕最前端的那个Activity。比如有另一个非全屏或者透明的Activity是Resumed状态，没有完全遮盖这个Activity。

③Stopped（停止状态）

当Activity完全不可见时，此时Activity还在后台运行，仍然在内存中保留Activity的状态，并不是完全销毁。这个也很好理解，当跳转的另外一个界面，之前的界面还在后台，按回退按钮还会恢复原来的状态，大部分软件在打开的时候，直接按Home键，并不会关闭它，此时的Activity就是Stopped状态。

### 2. Activity的启动模式

Android提供了四种Activity启动方式：

- 标准模式（standard）
- 栈顶复用模式（singleTop）
- 栈内复用模式（singleTask）
- 单例模式（singleInstance）

**启动模式的结构——栈**

Activity的管理是采用任务栈的形式，任务栈采用“后进先出”的栈结构。

**(1)标准模式（standard）**

每启动一次Activity，就会创建一个新的Activity实例并置于栈顶。谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。

例如：Activity A启动了Activity B，则就会在A所在的栈顶压入一个新的Activity。

[![img](https://camo.githubusercontent.com/12d2c89c4a47e509953079c8dba507772775358a/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d353965306333336161646435356636322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/12d2c89c4a47e509953079c8dba507772775358a/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d353965306333336161646435356636322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

**特殊情况，如果在Service或Application中启动一个Activity，其并没有所谓的任务栈，可以使用标记位Flag来解决。解决办法：为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，创建一个新栈。**

**应用场景：** 绝大多数Activity。如果以这种方式启动的Activity被跨进程调用，在5.0之前新启动的Activity实例会放入发送Intent的Task的栈的顶部，尽管它们属于不同的程序，这似乎有点费解看起来也不是那么合理，所以在5.0之后，上述情景会创建一个新的Task，新启动的Activity就会放入刚创建的Task中，这样就合理的多了。

**(2)栈顶复用模式（singleTop）**

如果需要新建的Activity位于任务栈栈顶，那么此Activity的实例就不会重建，而是重用栈顶的实例。并回调如下方法：

```java
    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
    }
```

由于不会重建一个Activity实例，则不会回调其他生命周期方法。
如果栈顶不是新建的Activity,就会创建该Activity新的实例，并放入栈顶。

[![img](https://camo.githubusercontent.com/eb1055f68e39e6abfc73729167c828365a2cf02b/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d363337386461653765613234393239342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/eb1055f68e39e6abfc73729167c828365a2cf02b/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d363337386461653765613234393239342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

**应用场景：** 在通知栏点击收到的通知，然后需要启动一个Activity，这个Activity就可以用singleTop，否则每次点击都会新建一个Activity。当然实际的开发过程中：某个场景下连续快速点击，启动了两个Activity。如果这个时候待启动的Activity使用 singleTop模式也是可以避免这个Bug的。同standard模式，如果是外部程序启动singleTop的Activity，在Android 5.0之前新创建的Activity会位于调用者的Task中，5.0及以后会放入新的Task中。

**(3)栈内复用模式（singleTask）**

该模式一个栈内只有一个该Activity实例。该模式，可以通过在AndroidManifest文件的Activity中指定该Activity需要加载到那个栈中，即singleTask的Activity可以指定想要加载的目标栈。singleTask和taskAffinity配合使用，指定开启的Activity加入到哪个栈中。

```xml
<activity android:name=".Activity1"
	android:launchMode="singleTask"
	android:taskAffinity="com.lvr.task"
	android:label="@string/app_name">
</activity>
```

**关于taskAffinity的值：** 每个Activity都有taskAffinity属性，这个属性指出了它希望进入的Task。如果一个Activity没有显式的指明该Activity的taskAffinity，那么它的这个属性就等于Application指明的taskAffinity，如果Application也没有指明，那么该taskAffinity的值就等于包名。

**执行逻辑：**

在这种模式下，如果Activity指定的栈不存在，则创建一个栈，并把创建的Activity压入栈内。如果Activity指定的栈存在，如果其中没有该Activity实例，则会创建Activity并压入栈顶，如果其中有该Activity实例，则把该Activity实例之上的Activity杀死清除出栈，重用并让该Activity实例处在栈顶，然后调用onNewIntent()方法。

对应如下三种情况：

[![img](https://camo.githubusercontent.com/6bfed74fe06c752ee1130062872cc9a4dfb303a4/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d323330396137323232333934636438302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/6bfed74fe06c752ee1130062872cc9a4dfb303a4/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d323330396137323232333934636438302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

[![img](https://camo.githubusercontent.com/61e80bb6418365392b2ac04589d6c55c36ea0192/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d623465336462633239636462376461342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/61e80bb6418365392b2ac04589d6c55c36ea0192/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d623465336462633239636462376461342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

[![img](https://camo.githubusercontent.com/59a0583de114614db66ebce9ec1bb4fee6df1740/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d346331353938616466386435336538382e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/59a0583de114614db66ebce9ec1bb4fee6df1740/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d346331353938616466386435336538382e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

**应用场景：** 大多数App的主页。对于大部分应用，当我们在主界面点击回退按钮的时候都是退出应用，那么当我们第一次进入主界面之后，主界面位于栈底，以后不管我们打开了多少个Activity，只要我们再次回到主界面，都应该使用将主界面Activity上所有的Activity移除的方式来让主界面Activity处于栈顶，而不是往栈顶新加一个主界面Activity的实例，通过这种方式能够保证退出应用时所有的Activity都能报销毁。在跨应用Intent传递时，如果系统中不存在singleTask Activity的实例，那么将创建一个新的Task，然后创建SingleTask Activity的实例，将其放入新的Task中。

**(4)单例模式（singleInstance）**

作为栈内复用模式（singleTask）的加强版,打开该Activity时，直接创建一个新的任务栈，并创建该Activity实例放入新栈中。一旦该模式的Activity实例已经存在于某个栈中，任何应用再激活该Activity时都会重用该栈中的实例。

[![img](https://camo.githubusercontent.com/e1351e26c3295029432458a0093eca44db010e01/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d313964376434366166336537616366372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/e1351e26c3295029432458a0093eca44db010e01/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d313964376434366166336537616366372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

**应用场景：** 呼叫来电界面。这种模式的使用情况比较罕见，在Launcher中可能使用。或者你确定你需要使Activity只有一个实例。建议谨慎使用。

**前台栈和后台栈的交互**

假如目前有两个任务栈。前台任务栈为AB，后台任务栈为CD，这里假设CD的启动模式均为singleTask,现在请求启动D，那么这个后台的任务栈都会被切换到前台，这个时候整个后退列表就变成了ABCD。当用户按back返回时，列表中的activity会一一出栈，如下图。

[![img](https://camo.githubusercontent.com/7dbeee883a4c7bdf0ec07cf60dfcc40774a79867/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d346165623139343762626132376534342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/7dbeee883a4c7bdf0ec07cf60dfcc40774a79867/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d346165623139343762626132376534342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

如果不是请求启动D而是启动C，那么情况又不一样，如下图。

[![img](https://camo.githubusercontent.com/1cbfcd7e79be6a7148fba9d21a6d9fca7ef88860/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d663265616631303035636466316231642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/1cbfcd7e79be6a7148fba9d21a6d9fca7ef88860/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d663265616631303035636466316231642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

**调用SingleTask模式的后台任务栈中的Activity，会把整个栈的Actvity压入当前栈的栈顶。singleTask会具有clearTop特性，把之上的栈内Activity清除。**

**Activity的Flags**

Activity的Flags很多，这里介绍集中常用的，用于设定Activity的启动模式。可以在启动Activity时，通过Intent的addFlags()方法设置。

**(1)FLAG_ACTIVITY_NEW_TASK** 其效果与指定Activity为singleTask模式一致。

**(2)FLAG_ACTIVITY_SINGLE_TOP** 其效果与指定Activity为singleTop模式一致。

**(3)FLAG_ACTIVITY_CLEAR_TOP** 具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。如果和singleTask模式一起出现，若被启动的Activity已经存在栈中，则清除其之上的Activity，并调用该Activity的onNewIntent方法。如果被启动的Activity采用standard模式，那么该Activity连同之上的所有Activity出栈，然后创建新的Activity实例并压入栈中。



## 二、Service全方位解析

Service是Android程序中四大基础组件之一，它和Activity一样都是Context的子类，只不过它没有UI界面，是在后台运行的组件。

Service是Android中实现程序后台运行的解决方案，它非常适用于去执行那些不需要和用户交互而且还要求长期运行的任务。Service默认并不会运行在子线程中，它也不运行在一个独立的进程中，它同样执行在UI线程中，因此，不要在Service中执行耗时的操作，除非你在Service中创建了子线程来完成耗时操作。

### 1. Service种类

**按运行地点分类：**  


![](http://upload-images.jianshu.io/upload_images/3985563-af63266f00ae1fbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 **按运行类型分类：**  


![](http://upload-images.jianshu.io/upload_images/3985563-87972eef7c1b435a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  **按使用方式分类：**  


![](http://upload-images.jianshu.io/upload_images/3985563-2c6f9875b470540a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  


### 2. Service生命周期

![](http://upload-images.jianshu.io/upload_images/3985563-85614addbbec7a0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  

**OnCreate()**  
系统在service第一次创建时执行此方法，来执行**只运行一次的**初始化工作。如果service已经运行，这个方法不会被调用。

**onStartCommand()**  
每次客户端调用startService\(\)方法启动该Service都会回调该方法（**多次调用**）。一旦这个方法执行，service就启动并且在后台长期运行。通过调用stopSelf\(\)或stopService\(\)来停止服务。

**OnBind()**  
当组件调用bindService\(\)想要绑定到service时\(比如想要执行进程间通讯\)系统调用此方法（**一次调用**，一旦绑定后，下次再调用bindService\(\)不会回调该方法）。在你的实现中，你必须提供一个返回一个IBinder来以使客户端能够使用它与service通讯，你必须总是实现这个方法，但是如果你不允许绑定，那么你应返回null。

**OnUnbind()**  
当前组件调用unbindService\(\)，想要解除与service的绑定时系统调用此方法（**一次调用**，一旦解除绑定后，下次再调用unbindService\(\)会抛出异常）。

**OnDestory()**  
系统在service不再被使用并要销毁时调用此方法（**一次调用**）。service应在此方法中释放资源，比如线程，已注册的侦听器，接收器等等．这是service收到的最后一个调用。

下面介绍三种不同情况下Service的生命周期情况。

**1.startService / stopService**

生命周期顺序：onCreate-&gt;onStartCommand-&gt;onDestroy

如果一个Service被某个Activity 调用 Context.startService方法启动，那么不管是否有Activity使用bindService绑定或unbindService解除绑定到该Service，该Service都在后台运行，直到被调用stopService，或自身的stopSelf方法。当然如果系统资源不足，android系统也可能结束服务，还有一种方法可以关闭服务，在设置中，通过应用-&gt;找到自己应用-&gt;停止。

**注意点：**

①第一次 startService 会触发 onCreate 和 onStartCommand，以后在服务运行过程中，每次 startService 都只会触发 onStartCommand

②不论 startService 多少次，stopService 一次就会停止服务

**2.bindService / unbindService**

生命周期顺序：onCreate-&gt;onBind-&gt;onUnBind-&gt;onDestroy

如果一个Service在某个Activity中被调用bindService方法启动，不论bindService被调用几次，Service的onCreate方法只会执行一次，同时onStartCommand方法始终不会调用。

当建立连接后，Service会一直运行，除非调用unbindService来接触绑定、断开连接或调用该Service的Context不存在了（如Activity被Finish——**即通过bindService启动的Service的生命周期依附于启动它的Context**），系统在这时会自动停止该Service。

**注意点：**第一次 bindService 会触发 onCreate 和 onBind，以后在服务运行过程中，每次 bindService 都不会触发任何回调

**3.混合型（上面两种方式的交互）**

当一个Service在被启动\(startService\)的同时又被绑定\(bindService\)，该Service将会一直在后台运行，并且不管调用几次，onCreate方法始终只会调用一次，onStartCommand的调用次数与startService调用的次数一致（使用bindService方法不会调用onStartCommand）。同时，**调用unBindService将不会停止Service，必须调用stopService或Service自身的stopSelf来停止服务。**

**在什么情况下使用 startService 或 bindService 或 同时使用startService 和 bindService？**

①如果你只是想要启动一个后台服务长期进行某项任务那么使用 startService 便可以了。

②如果你想要与正在运行的 Service 取得联系，那么有两种方法，一种是使用 broadcast ，另外是使用 bindService ，前者的缺点是如果交流较为频繁，容易造成性能上的问题，并且 BroadcastReceiver 本身执行代码的时间是很短的（也许执行到一半，后面的代码便不会执行），而后者则没有这些问题，因此我们肯定选择使用 bindService（这个时候你便同时在使用 startService 和 bindService 了，这在 Activity 中更新 Service 的某些运行状态是相当有用的）。

③如果你的服务只是公开一个远程接口，供连接上的客服端（android 的 Service 是C/S架构）远程调用执行方法。这个时候你可以不让服务一开始就运行，而只用 bindService ，这样在第一次 bindService 的时候才会创建服务的实例运行它，这会节约很多系统资源，特别是如果你的服务是Remote Service，那么该效果会越明显（当然在 Service 创建的时候会花去一定时间，你应当注意到这点）。

### 3. Service的几种典型使用实例

**1) 不可交互的后台服务**

不可交互的后台服务即是普通的Service，通过startService\(\)方式开启。Service的生命周期很简单，分别为onCreate、onStartCommand、onDestroy这三个。

**创建服务类：**

```java
public class BackService extends Service {
    private Thread mThread;

    @Override
    public void onCreate() {
        super.onCreate();

    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        System.out.println("onBind");
        return null;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        //执行耗时操作

        mThread = new Thread() {
            @Override
            public void run() {
                try {
                    while (true) {
                        //等待停止线程
                        if (this.isInterrupted()) {
                            throw new InterruptedException();
                        }
                        //耗时操作。
                        System.out.println("执行耗时操作");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        mThread.start();
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        //停止线程
        mThread.interrupt();
    }
}
```

**配置服务：**

```java
<service android:name=".BackService">
</service>
```

如果想配置成远程服务，加如下代码：

```java
android:process="remote"
```

配置好Service类，只需要在前台，调用startService\(\)方法，就会启动耗时操作。

**注意：**

①不运行在一个独立的进程中，它同样执行在UI线程中，因此，在Service中创建了子线程来完成耗时操作。

②当Service关闭后，如果在onDestory\(\)方法中不关闭线程，你会发现我们的子线程进行的耗时操作是一直存在的，此时关闭该子线程的方法需要直接关闭该应用程序。因此，**在onDestory\(\)方法中要进行必要的清理工作。**

**2) 可交互的后台服务**

可交互的后台服务是指前台页面可以调用后台服务的方法，通过bindService\(\)方式开启。Service的生命周期很简单，分别为onCreate、onBind、onUnBind、onDestroy这四个。  
可交互的后台服务实现步骤是和不可交互的后台服务实现步骤是一样的，区别在于启动的方式和获得Service的代理对象。

**创建服务类**  
和普通Service不同在于这里返回一个代理对象，返回给前台进行获取，即前台可以获取该代理对象执行后台服务的方法

```java
public class BackService extends Service {

    @Override
    public void onCreate() {
        super.onCreate();

    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        //返回MyBinder对象
        return new MyBinder();
    }
    //需要返回给前台的Binder类
    class MyBinder extends Binder {
        public void showTip(){
            System.out.println("我是来此服务的提示");
        }
    }
    @Override
    public void onDestroy() {
        super.onDestroy();

    }
}
```

**前台调用**  
通过以下方式绑定服务：

```java
bindService(mIntent,con,BIND_AUTO_CREATE);
```

其中第二个参数：

```java
private ServiceConnection con = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            BackService.MyBinder myBinder = (BackService.MyBinder) service;
            myBinder.showTip();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
```

当建立绑定后，onServiceConnected中的service便是Service类中onBind的返回值。如此便可以调用后台服务类的方法，实现交互。

当调用unbindService\(\)停止服务，同时要在onDestory\(\)方法中做好清理工作。

**注意：通过bindService启动的Service的生命周期依附于启动它的Context。因此当前台调用bindService的Context销毁后，那么服务会自动停止。**

**3) 混合型后台服务**

将上面两种启动方式结合起来就是混合性交互的后台服务了，即可以单独运行后台服务，也可以运行后台服务中提供的方法，其完整的生命周期是：onCreate-&gt;onStartCommand-&gt;onBind-&gt;onUnBind-&gt;onDestroy

**4) 前台服务**

所谓前台服务只不是通过一定的方式将服务所在的进程级别提升了。前台服务会一直有一个正在运行的图标在系统的状态栏显示，非常类似于通知的效果。

由于后台服务优先级相对比较低，当系统出现内存不足的情况下，它就有可能会被回收掉，所以前台服务就是来弥补这个缺点的，它可以一直保持运行状态而不被系统回收。

**创建服务类**

前台服务创建很简单，其实就在Service的基础上创建一个Notification，然后使用Service的startForeground\(\)方法即可启动为前台服务。

```java
public class ForeService extends Service{
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        beginForeService();
    }

    private void beginForeService() {
        //创建通知
        Notification.Builder mBuilder = new Notification.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentText("2017-2-27")
                .setContentText("您有一条未读短信...");
        //创建点跳转的Intent(这个跳转是跳转到通知详情页)
        Intent intent = new Intent(this,NotificationShow.class);
        //创建通知详情页的栈
        TaskStackBuilder stackBulider = TaskStackBuilder.create(this);
        //为其添加父栈 当从通知详情页回退时，将退到添加的父栈中
        stackBulider.addParentStack(NotificationShow.class);
        stackBuilder.addNextIntent(intent);
        PendingIntent pendingIntent = stackBulider.getPendingIntent(0,PendingIntent.FLAG_UPDATE_CURRENT);
        //设置跳转Intent到通知中
        mBuilder.setContentIntent(pendingIntent);
        //获取通知服务
        NotificationManager nm = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        //构建通知
        Notification notification = mBuilder.build();
        //显示通知
        nm.notify(0,notification);
        //启动前台服务
        startForeground(0,notification);
    }
}
```

**启动前台服务**

```java
startService(new Intent(this, ForeService.class));
```

关于TaskStackBuilder 这一段，可能不是看的很明白，下面详细介绍。

**TaskStackBuilder在Notification通知栏中的使用**

首先是用一般的PendingIntent来进行跳转

```java
mBuilder = new NotificationCompat.Builder(this).setContent(view)
        .setSmallIcon(R.drawable.icon).setTicker("新资讯")
        .setWhen(System.currentTimeMillis())
        .setOngoing(false)
        .setAutoCancel(true);
Intent intent = new Intent(this, NotificationShow.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
mBuilder.setContentIntent(pendingIntent);
```

这里是直接用PendingIntent来跳转到NotificationShow。

在运行效果上来看，首先发送了一条Notification到通知栏上，然后这时，退出程序，即MainActivity已经不存在了，回到home主菜单，看到Notification仍然存在，当然，我们还没有点击或者cancel它，现在去点击Notification，跳转到NotificationShow界面，然后我们按下Back键，发现直接回到home菜单了。现在大多数android应用都是在通知栏中如果有Notification通知的话，点击它，然后会直接跳转到对应的应用程序的某个界面，这时如果回退，即按下Back键，会返回到该应用程序的主界面，而不是系统的home菜单。所以用上面这种PendingIntent的做法达不到目的。这里我们使用TaskStackBuilder来做。

```java
mBuilder = new NotificationCompat.Builder(this)
        .setContent(view)
        .setSmallIcon(R.drawable.icon).setTicker("新资讯")
        .setWhen(System.currentTimeMillis())
        .setOngoing(false)
        .setAutoCancel(true);
Intent intent = new Intent(this, NotificationShow.class);
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
stackBuilder.addParentStack(NotificationShow.class);
stackBuilder.addNextIntent(intent);
PendingIntent pendingIntent = stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
//PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
mBuilder.setContentIntent(pendingIntent);
```

显示用TaskStackBuilder.create\(this\)创建一个stackBuilder实例，接下来addParentStack\(\);

关于这个方法，我们查一下官方API文档：

> Add the activity parent chain as specified by the parentActivityName attribute of the activity \(or activity-alias\) element in the application's manifest to the task stack builder

这句话意思是：为跳转后的activity添加一个父activity，在activity中的manifest中添加parentActivityName即可。

那么我们就在manifest文件中添加这个属性

```java
<activity
    android:name="com.lvr.service.NotificationShow"  
    android:parentActivityName=".MainActivity" >
</activity>
```

这里我让它的parentActivity为MainActivity，也就是说在NotificationShow这个界面点击回退时，会跳转到MainActivity这个界面，而不是像上面一样直接回到了home菜单。

**注意：通过 stopForeground\(\)方法可以取消通知，即将前台服务降为后台服务。此时服务依然没有停止。通过stopService\(\)可以把前台服务停止。**



## 三、BroadcastReceiver全方位解析

- `BroadcastReceiver`（广播接收器），属于Android四大组件之一
- 在Android开发中，BroadcastReceiver的应用场景非常多广播，是一个全局的监听器，属于`Android`四大组件之一
- Android 广播分为两个角色：广播发送者、广播接收者

### 1. 作用

- 用于监听 / 接收 应用发出的广播消息，并做出响应

- 应用场景
  a. 不同组件之间通信（包括应用内 / 不同应用之间）

  b. 与 `Android` 系统在特定情况下的通信

  > 如当电话呼入时、网络可用时

  c. 多线程通信

### 2. 实现原理

- `Android`中的广播使用了设计模式中的**观察者模式**：基于消息的发布/订阅事件模型。

  > 因此，Android将广播的**发送者 和 接收者 解耦**，使得系统方便集成，更易扩展

- 模型中有3个角色：

  1. 消息订阅者（广播接收者）
  2. 消息发布者（广播发布者）
  3. 消息中心（`AMS`，即`Activity Manager Service`）

![](http://upload-images.jianshu.io/upload_images/944365-0896ba8d9155140e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 原理描述：

  1. 广播接收者 通过 `Binder`机制在 `AMS` 注册

  2. 广播发送者 通过 `Binder` 机制向 `AMS` 发送广播

  3. `AMS` 根据 广播发送者 要求，在已注册列表中，寻找合适的广播接收者

     > 寻找依据：`IntentFilter / Permission`

  4. `AMS`将广播发送到合适的广播接收者相应的消息循环队列中；

  5. 广播接收者通过 消息循环 拿到此广播，并回调 `onReceive()`

**特别注意**：广播发送者 和 广播接收者的执行 是 **异步**的，发出去的广播不会关心有无接收者接收，也不确定接收者到底是何时才能接收到；

### 3. 具体使用

具体使用流程如下：

![](http://upload-images.jianshu.io/upload_images/944365-7c9ff656ebd1b981.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来将介绍上图中的 **开发者手动完成部分**

#### 3.1 自定义广播接收者BroadcastReceiver

- 继承自BroadcastReceivre基类

- 必须复写抽象方法onReceive()方法

  > 1. 广播接收器接收到相应广播后，会自动回调onReceive()方法
  > 2. 一般情况下，onReceive方法会涉及与其他组件之间的交互，如发送Notification、启动service等
  > 3. 默认情况下，广播接收器运行在UI线程，因此，onReceive方法不能执行耗时操作，否则将导致ANR。

- 代码范例

```java
public class MyBroadcastReceiver extends BroadcastReceiver {

  //接收到广播后自动调用该方法
  @Override
  public void onReceive(Context context, Intent intent) {
    //写入接收广播后的操作
    }
}
```

#### 4.2 广播接收器注册

注册的方式分为两种：静态注册、动态注册

**静态注册**

- 在AndroidManifest.xml里通过 **<receive\>** 标签声明
- 属性说明：

```xml
<receiver
  android:enabled=["true" | "false"]
  //此broadcastReceiver能否接收其他App的发出的广播
  //默认值是由receiver中有无intent-filter决定的：如果有intent-filter，默认值为true，否则为false
  android:exported=["true" | "false"]
  android:icon="drawable resource"
  android:label="string resource"
  //继承BroadcastReceiver子类的类名
  android:name=".mBroadcastReceiver"
  //具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
  android:permission="string"
  //BroadcastReceiver运行所处的进程
  //默认为app的进程，可以指定独立的进程
  //注：Android四大基本组件都可以通过此属性指定自己的独立进程
  android:process="string" >

  //用于指定此广播接收器将接收的广播类型
  //本示例中给出的是用于接收网络状态改变时发出的广播
  <intent-filter>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>
```

- 注册示例

``` xml
<receiver
  //此广播接收者类是MyBroadcastReceiver
  android:name=".MyBroadcastReceiver" >
  //用于接收网络状态改变时发出的广播
  <intent-filter>
      <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>
```

当此App首次启动时，系统会自动实例化mBroadcastReceiver类，并注册到系统中。

**动态注册**

在代码中通过调用Context的registerReceiver()方法进行动态注册BroadcastReceiver，具体代码如下：

```Java
private MyBroadcastReceiver mBroadcastReceiver;

@Override
protected void onResume() {
    super.onResume();

    //实例化BroadcastReceiver子类 &  IntentFilter
    mBroadcastReceiver = new MyBroadcastReceiver();
    IntentFilter intentFilter = new IntentFilter();
    //设置接收广播的类型
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
    //调用Context的registerReceiver（）方法进行动态注册
    registerReceiver(mBroadcastReceiver, intentFilter);
}

//注册广播后，要在相应位置记得销毁广播
//即在onPause() 中unregisterReceiver(mBroadcastReceiver)
//当此Activity实例化时，会动态将MyBroadcastReceiver注册到系统中
//当此Activity销毁时，动态注册的MyBroadcastReceiver将不再接收到相应的广播。
@Override
protected void onPause() {
    super.onPause();
    //销毁在onResume()方法中的广播
    unregisterReceiver(mBroadcastReceiver);
}
```

**特别注意**

- 动态广播最好在Activity的onResume()注册、onPause()注销。

- 原因：对于动态广播，有注册就必然得有注销，否则会导致**内存泄露**

  > 重复注册、重复注销也不允许

**两种注册方式的区别**

![](http://upload-images.jianshu.io/upload_images/944365-0ae738c6d50c0adf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.3 广播发送者向AMS发送广播

**广播的发送**

- 广播是用”意图（Intent）“标识
- 定义广播的本质：定义广播所具备的“意图（Intent）”
- 广播发送：广播发送者将此广播的”意图“通过**sendBroadcast()方法**发送出去

**广播的类型**

广播的类型主要分为5类：

- 普通广播（Normal Broadcast）
- 系统广播（System Broadcast）
- 有序广播（Ordered Broadcast）
- 粘性广播（Sticky Broadcast）
- App应用内广播（Local Broadcast）

具体说明如下：

**① 普通广播（Normal Broadcast）**

即开发者自身定义intent的广播（最常用）。发送广播使用如下：

```Java
Intent intent = new Intent();
//对应BroadcastReceiver中intentFilter的action
intent.setAction(BROADCAST_ACTION);
//发送广播
sendBroadcast(intent);
```

- 若被注册了的广播接收者中注册时intentFilter的action与上述匹配，则会接收此广播（即进行回调onReceive()）。如下mBroadcastReceiver则会接收上述广播

```Java
<receiver 
    //此广播接收者类是mBroadcastReceiver
    android:name=".mBroadcastReceiver" >
    //用于接收网络状态改变时发出的广播
    <intent-filter>
        <action android:name="BROADCAST_ACTION" />
    </intent-filter>
</receiver>
```

- 若发送广播有相应权限，那么广播接收者也需要相应权限

**② 系统广播（System Broadcast）**

- Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播
- 每个广播都有特定的Intent - Filter（包括具体的action），Android系统广播action如下：

| 系统操作                                                     | action                               |
| ------------------------------------------------------------ | ------------------------------------ |
| 监听网络变化                                                 | android.net.conn.CONNECTIVITY_CHANGE |
| 关闭或打开飞行模式                                           | Intent.ACTION_AIRPLANE_MODE_CHANGED  |
| 充电时或电量发生变化                                         | Intent.ACTION_BATTERY_CHANGED        |
| 电池电量低                                                   | Intent.ACTION_BATTERY_LOW            |
| 电池电量充足（即从电量低变化到饱满时会发出广播               | Intent.ACTION_BATTERY_OKAY           |
| 系统启动完成后(仅广播一次)                                   | Intent.ACTION_BOOT_COMPLETED         |
| 按下照相时的拍照按键(硬件按键)时                             | Intent.ACTION_CAMERA_BUTTON          |
| 屏幕锁屏                                                     | Intent.ACTION_CLOSE_SYSTEM_DIALOGS   |
| 设备当前设置被改变时(界面语言、设备方向等)                   | Intent.ACTION_CONFIGURATION_CHANGED  |
| 插入耳机时                                                   | Intent.ACTION_HEADSET_PLUG           |
| 未正确移除SD卡但已取出来时(正确移除方法:设置--SD卡和设备内存--卸载SD卡) | Intent.ACTION_MEDIA_BAD_REMOVAL      |
| 插入外部储存装置（如SD卡）                                   | Intent.ACTION_MEDIA_CHECKING         |
| 成功安装APK                                                  | Intent.ACTION_PACKAGE_ADDED          |
| 成功删除APK                                                  | Intent.ACTION_PACKAGE_REMOVED        |
| 重启设备                                                     | Intent.ACTION_REBOOT                 |
| 屏幕被关闭                                                   | Intent.ACTION_SCREEN_OFF             |
| 屏幕被打开                                                   | Intent.ACTION_SCREEN_ON              |
| 关闭系统时                                                   | Intent.ACTION_SHUTDOWN               |
| 重启设备                                                     | Intent.ACTION_REBOOT                 |

注：当使用系统广播时，只需要在注册广播接收者时定义相关的action即可，并不需要手动发送广播，当系统有相关操作时会自动进行系统广播

**③ 有序广播（Ordered Broadcast）**

- 定义：发送出去的广播被广播接收者按照先后顺序接收

  > 有序是针对广播接收者而言的

- 广播接受者接收广播的顺序规则（同时面向静态和动态注册的广播接受者）

  1. 按照Priority属性值从大-小排序；
  2. Priority属性相同者，动态注册的广播优先；

- 特点

  1. 接收广播按顺序接收
  2. 先接收的广播接收者可以对广播进行截断，即后接收的广播接收者不再接收到此广播；
  3. 先接收的广播接收者可以对广播进行修改，那么后接收的广播接收者将接收到被修改后的广播

- 具体使用
  有序广播的使用过程与普通广播非常类似，差异仅在于广播的发送方式：

  ```Java
  sendOrderedBroadcast(intent);
  ```

**④ App应用内广播（Local Broadcast）**

- 背景：Android中的广播可以跨App直接通信（exported对于有intent-filter情况下默认值为true）

- 可能出现的问题：

  - 其他App针对性发出与当前App intent-filter相匹配的广播，由此导致当前App不断接收广播并处理；
  - 其他App注册与当前App一致的intent-filter用于接收广播，获取广播具体信息；
    即会出现安全性 & 效率性的问题。

- 解决方案
  使用App应用内广播（Local Broadcast）

  > 1. App应用内广播可理解为一种局部广播，广播的发送者和接收者都同属于一个App。
  > 2. 相比于全局广播（普通广播），App应用内广播优势体现在：安全性高 & 效率高

- 具体使用1 - 将全局广播设置成局部广播

  1. 注册广播时将exported属性设置为*false*，使得非本App内部发出的此广播不被接收；

  2. 在广播发送和接收时，增设相应权限permission，用于权限验证；

  3. 发送广播时指定该广播接收器所在的包名，此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

     > 通过 **intent.setPackage(packageName)** 指定报名

- 具体使用2 - 使用封装好的LocalBroadcastManager类
  使用方式上与全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将参数的context变成了LocalBroadcastManager的单一实例

  > 注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册，不能静态注册

```Java
//注册应用内广播接收器
//步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
mBroadcastReceiver = new MyBroadcastReceiver(); 
IntentFilter intentFilter = new IntentFilter(); 

//步骤2：实例化LocalBroadcastManager的实例
localBroadcastManager = LocalBroadcastManager.getInstance(this);

//步骤3：设置接收广播的类型 
intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

//步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);

//取消注册应用内广播接收器
localBroadcastManager.unregisterReceiver(mBroadcastReceiver);

//发送应用内广播
Intent intent = new Intent();
intent.setAction(BROADCAST_ACTION);
localBroadcastManager.sendBroadcast(intent);
```

**⑤ 粘性广播（Sticky Broadcast）**

粘性消息在发送后就一直存在于系统的消息容器里面，等待对应的处理器去处理，如果暂时没有处理器处理这个消息则一直在消息容器里面处于等待状态，粘性广播的Receiver如果被销毁，那么下次重建时会自动接收到消息数据。

```xml
<uses-permission android:name="android.permission.BROADCAST_STICKY" />
```

需要添加权限，然后使用`sendStickyBroadcast`发送

由于在Android5.0 & API 21中已经失效，所以不建议使用，在这里也不作过多的总结。



## 四、ContentProvider全方位解析

ContentProvider，即内容提供者，属于Android的四大组件之一。

### 1. 作用

进程间进行**数据交互 & 共享**，即跨进程通信

![](http://upload-images.jianshu.io/upload_images/944365-3c4339c5f1d4a0fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 原理

`ContentProvider`的底层是采用 `Android`中的`Binder`机制

### 3. 具体使用

关于`ContentProvider`的使用主要介绍以下内容：

![](http://upload-images.jianshu.io/upload_images/944365-5c9b0e2ebed36c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.1 统一资源标识符（URI）

- 定义：`Uniform Resource Identifier`，即统一资源标识符

- 作用：唯一标识 ContentProvider & 其中的数据

  > 外界进程通过 URI 找到对应的ContentProvider & 其中的数据，再进行数据操作

- 具体使用

  URI分为 系统预置 & 自定义，分别对应系统内置的数据（如通讯录、日程表等等）和自定义数据库

  > 1. 关于 系统预置`URI` 此处不作过多讲解，需要的同学可自行查看
  > 2. 此处主要讲解 自定义`URI`

![](http://upload-images.jianshu.io/upload_images/944365-96019a2054eb27cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```Java
// 设置URI
Uri uri = Uri.parse("content://com.carson.provider/User/1") 
// 上述URI指向的资源是：名为 `com.carson.provider`的`ContentProvider` 中表名 为`User` 中的 `id`为1的数据

// 特别注意：URI模式存在匹配通配符* & ＃

// *：匹配任意长度的任何有效字符的字符串
// 以下的URI 表示 匹配provider的任何内容
content://com.example.app.provider/* 
// ＃：匹配任意长度的数字字符的字符串
// 以下的URI 表示 匹配provider中的table表的所有行
content://com.example.app.provider/table/#
```

#### 3.2  MIME数据类型

- 解释：MIME：全称Multipurpose Internet Mail Extensions，多功能Internet 邮件扩充服务。它是一种多用途网际邮件扩充协议，在1992年最早应用于电子邮件系统，但后来也应用到浏览器。MIME类型就是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。
- 作用：指定某个扩展名的文件用某种应用程序来打开
  如指定`.html`文件采用`text`应用程序打开、指定`.pdf`文件采用`flash`应用程序打开
- 具体使用：

**ContentProvider根据 URI 返回MIME类型**

```java
ContentProvider.geType(uri) ；
```

**MIME类型组成**
每种`MIME`类型 由2部分组成 = 类型 + 子类型

> MIME类型是 一个 包含2部分的字符串

```java
text / html
// 类型 = text、子类型 = html

text/css
text/xml
application/pdf
```

**MIME类型形式**
`MIME`类型有2种形式：

```java
// 形式1：单条记录  
vnd.android.cursor.item/自定义
// 形式2：多条记录（集合）
vnd.android.cursor.dir/自定义 

// 注：
// 1. vnd：表示父类型和子类型具有非标准的、特定的形式。
// 2. 父类型已固定好（即不能更改），只能区别是单条还是多条记录
// 3. 子类型可自定义
```

**实例说明**

```java
<-- 单条记录 -->
  // 单个记录的MIME类型
  vnd.android.cursor.item/vnd.yourcompanyname.contenttype 

  // 若一个Uri如下
  content://com.example.transportationprovider/trains/122   
  // 则ContentProvider会通过ContentProvider.geType(url)返回以下MIME类型
  vnd.android.cursor.item/vnd.example.rail


<-- 多条记录 -->
  // 多个记录的MIME类型
  vnd.android.cursor.dir/vnd.yourcompanyname.contenttype 
  // 若一个Uri如下
  content://com.example.transportationprovider/trains 
  // 则ContentProvider会通过ContentProvider.geType(url)返回以下MIME类型
  vnd.android.cursor.dir/vnd.example.rail
```

#### 3.3 ContentProvider类

**组织数据方式**

- ContentProvider主要以表格的形式组织数据。同时也支持文件数据，只是表格形式用得比较多

- 每个表格中包含多张表，每张表包含行 & 列，分别对应记录 & 字段，同数据库


**主要方法**

- **进程**间共享数据的本质是：添加、删除、获取 & 修改（更新）数据
- 所以`ContentProvider`的核心方法也主要是上述4个作用

```java
<-- 4个核心方法 -->
  public Uri insert(Uri uri, ContentValues values) 
  // 外部进程向 ContentProvider 中添加数据

  public int delete(Uri uri, String selection, String[] selectionArgs) 
  // 外部进程 删除 ContentProvider 中的数据

  public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
  // 外部进程更新 ContentProvider 中的数据

  public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,  String sortOrder)　 
  // 外部应用 获取 ContentProvider 中的数据

	// 注：
  	// 1. 上述4个方法由外部进程回调，并运行在ContentProvider进程的Binder线程池中（不是主线程）
 	// 2. 存在多线程并发访问，需要实现线程同步
   	// a. 若ContentProvider的数据存储方式是使用SQLite & 一个，则不需要，因为SQLite内部实现好了线程同步，若是多个SQLite则需要，因为SQL对象之间无法进行线程同步
  	// b. 若ContentProvider的数据存储方式是内存，则需要自己实现线程同步

<-- 2个其他方法 -->
public boolean onCreate() 
// ContentProvider创建后 或 打开系统后其它进程第一次访问该ContentProvider时 由系统进行调用
// 注：运行在ContentProvider进程的主线程，故不能做耗时操作

public String getType(Uri uri)
// 得到数据类型，即返回当前 Url 所代表数据的MIME类型
```

- `Android`为常见的数据（如通讯录、日程表等）提供了内置了默认的`ContentProvider`

- 但也可根据需求自定义ContentProvider，但上述6个方法必须重写

  > 本文主要讲解自定义`ContentProvider`

- `ContentProvider`类并不会直接与外部进程交互，而是通过`ContentResolver` 类

#### 3.4 ContentResolver类

统一管理不同 `ContentProvider`间的操作

> 1. 通过 `URI` 即可操作 不同的`ContentProvider` 中的数据
> 2. 外部进程通过 `ContentResolver`类 从而与`ContentProvider`类进行交互

**为什么要使用通过`ContentResolver`类从而与`ContentProvider`类进行交互，而不直接访问`ContentProvider`类？**

- 一般来说，一款应用要使用多个`ContentProvider`，若需要了解每个`ContentProvider`的不同实现从而再完成数据交互，**操作成本高  & 难度大**
- 所以再`ContentProvider`类上加多了一个 `ContentResolver`类对所有的`ContentProvider`进行统一管理。

**具体使用**

`ContentResolver` 类提供了与`ContentProvider`类相同名字 & 作用的4个方法

```java
// 外部进程向 ContentProvider 中添加数据
public Uri insert(Uri uri, ContentValues values)　 

// 外部进程 删除 ContentProvider 中的数据
public int delete(Uri uri, String selection, String[] selectionArgs)

// 外部进程更新 ContentProvider 中的数据
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)　 

// 外部应用 获取 ContentProvider 中的数据
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
```

- 实例说明

```java
// 使用ContentResolver前，需要先获取ContentResolver
// 可通过在所有继承Context的类中 通过调用getContentResolver()来获得ContentResolver
ContentResolver resolver =  getContentResolver(); 

// 设置ContentProvider的URI
Uri uri = Uri.parse("content://cn.scu.myprovider/user"); 

// 根据URI 操作 ContentProvider中的数据
// 此处是获取ContentProvider中 user表的所有记录 
Cursor cursor = resolver.query(uri, null, null, null, "userid desc");
```

`Android` 提供了3个用于辅助`ContentProvide`的工具类：

- `ContentUris`
- `UriMatcher`
- `ContentObserver`

#### 3.5 ContentUris类

- 作用：操作 `URI`
- 具体使用
  核心方法有两个：`withAppendedId（）` &`parseId（）`

```java
// withAppendedId（）作用：向URI追加一个id
Uri uri = Uri.parse("content://cn.scu.myprovider/user") 
Uri resultUri = ContentUris.withAppendedId(uri, 7);  
// 最终生成后的Uri为：content://cn.scu.myprovider/user/7

// parseId（）作用：从URL中获取ID
Uri uri = Uri.parse("content://cn.scu.myprovider/user/7") 
long personid = ContentUris.parseId(uri); 
//获取的结果为:7
```

#### 3.6 UriMatcher类

在`ContentProvider` 中注册`URI`，根据 `URI` 匹配 `ContentProvider` 中对应的数据表。

- 具体使用

```Java
// 步骤1：初始化UriMatcher对象
UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
//常量UriMatcher.NO_MATCH  = 不匹配任何路径的返回码
// 即初始化时不匹配任何东西

// 步骤2：在ContentProvider 中注册URI（addURI（））
int URI_CODE_a = 1；
int URI_CODE_b = 2；
matcher.addURI("cn.scu.myprovider", "user1", URI_CODE_a);
matcher.addURI("cn.scu.myprovider", "user2", URI_CODE_b);
// 若URI资源路径 = content://cn.scu.myprovider/user1 ，则返回注册码URI_CODE_a
// 若URI资源路径 = content://cn.scu.myprovider/user2 ，则返回注册码URI_CODE_b

// 步骤3：根据URI 匹配 URI_CODE，从而匹配ContentProvider中相应的资源（match（））

@Override
public String getType (Uri uri){
    Uri uri = Uri.parse(" content://cn.scu.myprovider/user1");

    switch (matcher.match(uri)) {
        // 根据URI匹配的返回码是URI_CODE_a
        // 即matcher.match(uri) == URI_CODE_a
        case URI_CODE_a:
            return tableNameUser1;
        // 如果根据URI匹配的返回码是URI_CODE_a，则返回ContentProvider中的名为tableNameUser1的表
        case URI_CODE_b:
            return tableNameUser2;
        // 如果根据URI匹配的返回码是URI_CODE_b，则返回ContentProvider中的名为tableNameUser2的表
    }
}
```

#### 3.7 ContentObserver类

- 定义：内容观察者

- 作用：观察 Uri引起ContentProvider 中的数据变化 & 通知外界（即访问该数据访问者）

  > 当`ContentProvider` 中的数据发生变化（增、删 & 改）时，就会触发该 `ContentObserver`类

- 具体使用

```Java
// 步骤1：注册内容观察者ContentObserver
getContentResolver().registerContentObserver（uri）；
// 通过ContentResolver类进行注册，并指定需要观察的URI

// 步骤2：当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）
public class UserContentProvider extends ContentProvider {
    public Uri insert(Uri uri, ContentValues values) {
        db.insert("user", "userid", values);
        getContext().getContentResolver().notifyChange(uri, null);
        // 通知访问者
    }
}

// 步骤3：解除观察者
getContentResolver().unregisterContentObserver（uri）；
// 同样需要通过ContentResolver类进行解除
```

至此，关于`ContentProvider`的使用已经讲解完毕

### 4. 实例说明

由于`ContentProvider`不仅常用于进程间通信，同时也适用于进程内通信，所以本实例会采用ContentProvider讲解：进程内通信和进程间通信。实例说明：采用的数据源是`Android`中的`SQLite`数据库。

#### 4.1 进程内通信

- 步骤说明：
  1. 创建数据库类
  2. 自定义 `ContentProvider` 类
  3. 注册 创建的 `ContentProvider`类
  4. 进程内访问 `ContentProvider`的数据
- 具体使用

**步骤1：创建数据库类**
**DBHelper.java**

```java
public class DBHelper extends SQLiteOpenHelper {

    // 数据库名
    private static final String DATABASE_NAME = "finch.db";

    // 表名
    public static final String USER_TABLE_NAME = "user";
    public static final String JOB_TABLE_NAME = "job";

    private static final int DATABASE_VERSION = 1;
    //数据库版本号

    public DBHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

        // 创建两个表格:用户表 和职业表
        db.execSQL("CREATE TABLE IF NOT EXISTS " + USER_TABLE_NAME + "(_id INTEGER PRIMARY KEY AUTOINCREMENT," + " name TEXT)");
        db.execSQL("CREATE TABLE IF NOT EXISTS " + JOB_TABLE_NAME + "(_id INTEGER PRIMARY KEY AUTOINCREMENT," + " job TEXT)");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)   {

    }
}
```

**步骤2：自定义 ContentProvider 类**

```java
public class MyProvider extends ContentProvider {

    private Context mContext;
    DBHelper mDbHelper = null;
    SQLiteDatabase db = null;

    public static final String AUTOHORITY = "cn.scu.myprovider";
    // 设置ContentProvider的唯一标识

    public static final int User_Code = 1;
    public static final int Job_Code = 2;

    // UriMatcher类使用:在ContentProvider 中注册URI
    private static final UriMatcher mMatcher;

    static {
        mMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        // 初始化
        mMatcher.addURI(AUTOHORITY, "user", User_Code);
        mMatcher.addURI(AUTOHORITY, "job", Job_Code);
        // 若URI资源路径 = content://cn.scu.myprovider/user ，则返回注册码User_Code
        // 若URI资源路径 = content://cn.scu.myprovider/job ，则返回注册码Job_Code
    }

    // 以下是ContentProvider的6个方法

    /**
     * 初始化ContentProvider
     */
    @Override
    public boolean onCreate() {

        mContext = getContext();
        // 在ContentProvider创建时对数据库进行初始化
        // 运行在主线程，故不能做耗时操作,此处仅作展示
        mDbHelper = new DBHelper(getContext());
        db = mDbHelper.getWritableDatabase();

        // 初始化两个表的数据(先清空两个表,再各加入一个记录)
        db.execSQL("delete from user");
        db.execSQL("insert into user values(1,'Carson');");
        db.execSQL("insert into user values(2,'Kobe');");

        db.execSQL("delete from job");
        db.execSQL("insert into job values(1,'Android');");
        db.execSQL("insert into job values(2,'iOS');");

        return true;
    }

    /**
     * 添加数据
     */
    @Override
    public Uri insert(Uri uri, ContentValues values) {

        // 根据URI匹配 URI_CODE，从而匹配ContentProvider中相应的表名
        // 该方法在最下面
        String table = getTableName(uri);

        // 向该表添加数据
        db.insert(table, null, values);

        // 当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）
        mContext.getContentResolver().notifyChange(uri, null);

//        // 通过ContentUris类从URL中获取ID
//        long personid = ContentUris.parseId(uri);
//        System.out.println(personid);

        return uri;
    }

    /**
     * 查询数据
     */
    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        // 根据URI匹配 URI_CODE，从而匹配ContentProvider中相应的表名
        // 该方法在最下面
        String table = getTableName(uri);

//        // 通过ContentUris类从URL中获取ID
//        long personid = ContentUris.parseId(uri);
//        System.out.println(personid);

        // 查询数据
        return db.query(table, projection, selection, selectionArgs, null, null, sortOrder, null);
    }

    /**
     * 更新数据
     */
    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        // 由于不展示,此处不作展开
        return 0;
    }

    /**
     * 删除数据
     */
    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // 由于不展示,此处不作展开
        return 0;
    }

    @Override
    public String getType(Uri uri) {
        // 由于不展示,此处不作展开
        return null;
    }

    /**
     * 根据URI匹配 URI_CODE，从而匹配ContentProvider中相应的表名
     */
    private String getTableName(Uri uri) {
        String tableName = null;
        switch (mMatcher.match(uri)) {
            case User_Code:
                tableName = DBHelper.USER_TABLE_NAME;
                break;
            case Job_Code:
                tableName = DBHelper.JOB_TABLE_NAME;
                break;
        }
        return tableName;
    }
}
```

**步骤3：注册 创建的 ContentProvider类**
**AndroidManifest.xml**

``` xml
<provider android:name="MyProvider"
  android:authorities="cn.scu.myprovider"/>
```

**步骤4：进程内访问 ContentProvider中的数据**

**MainActivity.java**

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /**
         * 对user表进行操作
         */

        // 设置URI
        Uri uri_user = Uri.parse("content://cn.scu.myprovider/user");

        // 插入表中数据
        ContentValues values = new ContentValues();
        values.put("_id", 3);
        values.put("name", "Iverson");


        // 获取ContentResolver
        ContentResolver resolver =  getContentResolver();
        // 通过ContentResolver 根据URI 向ContentProvider中插入数据
        resolver.insert(uri_user,values);

        // 通过ContentResolver 向ContentProvider中查询数据
        Cursor cursor = resolver.query(uri_user, new String[]{"_id","name"}, null, null, null);
        while (cursor.moveToNext()){
            System.out.println("query book:" + cursor.getInt(0) +" "+ cursor.getString(1));
            // 将表中数据全部输出
        }
        cursor.close();
        // 关闭游标

        /**
         * 对job表进行操作
         */
        // 和上述类似,只是URI需要更改,从而匹配不同的URI CODE,从而找到不同的数据资源
        Uri uri_job = Uri.parse("content://cn.scu.myprovider/job");

        // 插入表中数据
        ContentValues values2 = new ContentValues();
        values2.put("_id", 3);
        values2.put("job", "NBA Player");

        // 获取ContentResolver
        ContentResolver resolver2 =  getContentResolver();
        // 通过ContentResolver 根据URI 向ContentProvider中插入数据
        resolver2.insert(uri_job,values2);

        // 通过ContentResolver 向ContentProvider中查询数据
        Cursor cursor2 = resolver2.query(uri_job, new String[]{"_id","job"}, null, null, null);
        while (cursor2.moveToNext()){
            System.out.println("query job:" + cursor2.getInt(0) +" "+ cursor2.getString(1));
            // 将表中数据全部输出
        }
        cursor2.close();
        // 关闭游标
}
}
```

**结果**

![](http://upload-images.jianshu.io/upload_images/944365-3c735e5a027df3d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.2 进程间进行数据共享

- 实例说明：本文需要创建2个进程，即创建两个工程，作用如下

![](http://upload-images.jianshu.io/upload_images/944365-c3553f24d393bd48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**进程1**

使用步骤如下：

1. 创建数据库类
2. 自定义 `ContentProvider` 类
3. 注册 创建的 `ContentProvider` 类

前2个步骤同上例相同，此处不作过多描述，此处主要讲解步骤3.

**步骤3：注册 创建的 ContentProvider类**
*AndroidManifest.xml*

```xml
<provider 
               android:name="MyProvider"
               android:authorities="scut.carson_ho.myprovider"

               // 声明外界进程可访问该Provider的权限（读 & 写）
               android:permission="scut.carson_ho.PROVIDER"             

               // 权限可细分为读 & 写的权限
               // 外界需要声明同样的读 & 写的权限才可进行相应操作，否则会报错
               // android:readPermisson = "scut.carson_ho.Read"
               // android:writePermisson = "scut.carson_ho.Write"

               // 设置此provider是否可以被其他进程使用
               android:exported="true"

  />

	// 声明本应用 可允许通信的权限
    <permission android:name="scut.carson_ho.Read" android:protectionLevel="normal"/>
    // 细分读 & 写权限如下，但本Demo直接采用全权限
    // <permission android:name="scut.carson_ho.Write" android:protectionLevel="normal"/>
    // <permission android:name="scut.carson_ho.PROVIDER" android:protectionLevel="normal"/>
```

至此，进程1创建完毕，即创建`ContentProvider` & 数据 准备好了。

**进程2**

**步骤1：声明可访问的权限**

**AndroidManifest.xml**

```Xml
    // 声明本应用可允许通信的权限（全权限）
    <uses-permission android:name="scut.carson_ho.PROVIDER"/>

    // 细分读 & 写权限如下，但本Demo直接采用全权限
    // <uses-permission android:name="scut.carson_ho.Read"/>
    //  <uses-permission android:name="scut.carson_ho.Write"/>

	// 注：声明的权限必须与进程1中设置的权限对应
```

**步骤2：访问 ContentProvider的类**

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        /**
         * 对user表进行操作
         */

        // 设置URI
        Uri uri_user = Uri.parse("content://scut.carson_ho.myprovider/user");

        // 插入表中数据
        ContentValues values = new ContentValues();
        values.put("_id", 4);
        values.put("name", "Jordan");


        // 获取ContentResolver
        ContentResolver resolver =  getContentResolver();
        // 通过ContentResolver 根据URI 向ContentProvider中插入数据
        resolver.insert(uri_user,values);

        // 通过ContentResolver 向ContentProvider中查询数据
        Cursor cursor = resolver.query(uri_user, new String[]{"_id","name"}, null, null, null);
        while (cursor.moveToNext()){
            System.out.println("query book:" + cursor.getInt(0) +" "+ cursor.getString(1));
            // 将表中数据全部输出
        }
        cursor.close();
        // 关闭游标

        /**
         * 对job表进行操作
         */
        // 和上述类似,只是URI需要更改,从而匹配不同的URI CODE,从而找到不同的数据资源
        Uri uri_job = Uri.parse("content://scut.carson_ho.myprovider/job");

        // 插入表中数据
        ContentValues values2 = new ContentValues();
        values2.put("_id", 4);
        values2.put("job", "NBA Player");

        // 获取ContentResolver
        ContentResolver resolver2 =  getContentResolver();
        // 通过ContentResolver 根据URI 向ContentProvider中插入数据
        resolver2.insert(uri_job,values2);

        // 通过ContentResolver 向ContentProvider中查询数据
        Cursor cursor2 = resolver2.query(uri_job, new String[]{"_id","job"}, null, null, null);
        while (cursor2.moveToNext()){
            System.out.println("query job:" + cursor2.getInt(0) +" "+ cursor2.getString(1));
            // 将表中数据全部输出
        }
        cursor2.close();
        // 关闭游标
    }
}
```

**结果展示**

**在进程展示时，需要先运行准备数据的进程1，再运行需要访问数据的进程2**

1. 运行准备数据的进程1
   在进程1中，我们准备好了一系列数据

   ![](http://upload-images.jianshu.io/upload_images/944365-3c79a2f1e3d0a2ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 运行需要访问数据的进程2
   在进程2中，我们先向`ContentProvider`中插入数据，再查询数据

## ![](http://upload-images.jianshu.io/upload_images/944365-16b20971852ee5c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 优点

- 安全：`ContentProvider`为应用间的数据交互提供了一个安全的环境：允许把自己的应用数据根据需求开放给 其他应用 进行 **增、删、改、查**，而不用担心因为直接开放数据库权限而带来的安全问题

- 访问简单且高效，对比于其他对外共享数据的方式，数据访问方式会因数据存储的方式而不同：
  - 采用 文件方式 对外共享数据，需要进行文件操作读写数据； 
  - 采用 `Sharedpreferences` 共享数据，需要使用sharedpreferences API读写数据

  这使得访问数据变得复杂，难度大。

  而采用ContentProvider方式，其解耦了底层数据的存储方式，使得无论底层数据存储采用何种方式，外界对数据的访问方式都是统一的，这使得访问简单，高效

  > 如一开始数据存储方式 采用 `SQLite` 数据库，后来把数据库换成 `MongoDB`，也不会对上层数据`ContentProvider`使用代码产生影响

![](http://upload-images.jianshu.io/upload_images/944365-a0e46788a2151e4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 五、Fragment详解

### 1. Fragment的生命周期

因为Fragment是依附于Activity存在的，因此它的生命周期受到Activity的生命周期影响

![](http://upload-images.jianshu.io/upload_images/1780352-f8584bc70f3c149c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Fragment比Activity多了几个生命周期的回调方法

- onAttach(Activity) 当Fragment与Activity发生关联的时候调用
- onCreateView(LayoutInflater, ViewGroup, Bundle) 创建该Fragment的视图
- onActivityCreated(Bundle) 当Activity的onCreated方法返回时调用
- onDestroyView() 与onCreateView方法相对应，当该Fragment的视图被移除时调用
- onDetach() 与onAttach方法相对应，当Fragment与Activity取消关联时调用

> 注意：除了onCreateView，其他的所有方法如果重写了，必须调用父类对于该方法的实现

### 2. Fragment的使用方式

**静态使用Fragment**

① 创建一个类继承Fragment，重写onCreateView方法，来确定Fragment要显示的布局

② 在Activity中声明该类，与普通的View对象一样

**代码演示**

MyFragment对应的布局文件item_fragment.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="@color/colorAccent"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:src="@mipmap/ic_launcher" />

</RelativeLayout>
```

继承Frgmanet的类MyFragment【请注意导包的时候导v4的Fragment的包】

```java
public class MyFragment extends Fragment {
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        /*
        * 参数1：布局文件的id
        * 参数2：容器
        * 参数3：是否将这个生成的View添加到这个容器中去
        * 作用是将布局文件封装在一个View对象中，并填充到此Fragment中
        * */
        View v = inflater.inflate(R.layout.item_fragment, container, false);
        return v;
    }
}
```

Activity对应的布局文件

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.usher.fragment.MainActivity">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:gravity="center"
        android:text="Good Boy" />

    <fragment
        android:id="@+id/myfragment"
        android:name="com.usher.fragment.MyFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

运行效果图

![](http://upload-images.jianshu.io/upload_images/1780352-bd36369dc712c754.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**动态使用Fragment**

实现点击不同的按钮，在Activity中显示不同的Fragment

**代码演示**

Fragment对应的布局文件item_fragment.xml

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="@color/colorAccent" //背景红色
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:src="@mipmap/ic_launcher" />

</RelativeLayout>
```

继承Frgmanet的类MyFragment

```java
public class MyFragment extends Fragment {
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.item_fragment, container, false);
        return v;
    }
}
```

Fragment2对应的布局文件item_fragment2.xml

```Xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="@color/colorPrimary" //背景蓝色
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:src="@mipmap/ic_launcher" />

</RelativeLayout>
```

继承Fragment2的类

```java
public class MyFragment extends Fragment {
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.item_fragment2, container, false);
        return v;
    }
}
```

MainActivity对应的布局文件

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.usher.fragment.MainActivity">

    <Button
        android:id="@+id/bt_red"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Red" />

    <Button
        android:id="@+id/bt_blue"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Blue" />

    <FrameLayout
        android:id="@+id/myframelayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

MainActivity类

```java
public class MainActivity extends AppCompatActivity {

    private Button bt_red;
    private Button bt_blue;
    private FragmentManager manager;
    private MyFragment fragment1;
    private MyFragment2 fragment2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();

        fragment1 = new MyFragment();
        fragment2 = new MyFragment2();

        //初始化FragmentManager对象
        manager = getSupportFragmentManager();

        //使用FragmentManager对象用来开启一个Fragment事务
        FragmentTransaction transaction = manager.beginTransaction();

        //默认显示fragment1
        transaction.add(R.id.myframelayout, fragment1).commit();

        //对bt_red设置监听
        bt_red.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                FragmentTransaction transaction = manager.beginTransaction();
                transaction.replace(R.id.myframelayout, fragment1).commit();
            }
        });

        //对bt_blue设置监听
        bt_blue.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                FragmentTransaction transaction = manager.beginTransaction();
                transaction.replace(R.id.myframelayout, fragment2).commit();
            }
        });
    }

    private void initView() {
        bt_red = (Button) findViewById(R.id.bt_red);
        bt_blue = (Button) findViewById(R.id.bt_blue);
    }

}
```

**显示效果**

默认显示

![](http://upload-images.jianshu.io/upload_images/1780352-03b001ae2419fc28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击BLUE按钮时

![](http://upload-images.jianshu.io/upload_images/1780352-f8889a725e75253f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击RED按钮时

![](http://upload-images.jianshu.io/upload_images/1780352-5a447a4ea3bdbb5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上代码写的比较臃肿但是比较容易看明白：

① 在Acitivity对应的布局中写上一个FramLayout控件，此空间的作用是当作Fragment的容器，Fragment通过FrameLayout显示在Acitivity里，这两个单词容易混淆，请注意

② 准备好你的Fragment，然后再Activity中实例化，v4包的Fragment是通过getSupportFragmentManager()方法新建Fragment管理器对象，此处不讨论app包下的Fragment

③ 然后通过Fragment管理器对象调用beginTransaction()方法，实例化FragmentTransaction对象，有人称之为事务

④ FragmentTransaction对象【以下直接用transaction代替】，transaction的方法主要有以下几种：

- transaction.add() 向Activity中添加一个Fragment
- transaction.remove() 从Activity中移除一个Fragment，如果被移除的Fragment没有添加到回退栈（回退栈后面会详细说），这个Fragment实例将会被销毁
- transaction.replace() 使用另一个Fragment替换当前的，实际上就是remove()然后add()的合体
- transaction.hide() 隐藏当前的Fragment，仅仅是设为不可见，并不会销毁
- transaction.show() 显示之前隐藏的Fragment
- detach() 会将view从UI中移除,和remove()不同,此时fragment的状态依然由FragmentManager维护
- attach() 重建view视图，附加到UI上并显示
- ransatcion.commit() 提交事务

注意：在add/replace/hide/show以后都要commit其效果才会在屏幕上显示出来

### 3. Fragment的回退栈

Fragment的回退栈是用来保存每一次Fragment事务发生的变化

**FragmentTransaction.addToBackStack(String)**。把当前事务的变化情况添加到回退栈。

![](http://upload-images.jianshu.io/upload_images/1780352-611dd683eec287c2.gif?imageMogr2/auto-orient/strip)

代码如下：

MainActivity的布局文件

```Xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  

    <FrameLayout  
        android:id="@+id/id_content"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent" >  
    </FrameLayout>  

</RelativeLayout>
```

MainActivity.java文件【这里添加的是app包下的Fragment，推荐v4包下的】

```java
public class MainActivity extends Activity {  

    protected void onCreate(Bundle savedInstanceState){  
        super.onCreate(savedInstanceState);  
        requestWindowFeature(Window.FEATURE_NO_TITLE);  
        setContentView(R.layout.activity_main);  

        FragmentManager fm = getFragmentManager();  
        FragmentTransaction tx = fm.beginTransaction();  
        tx.add(R.id.id_content, new FragmentOne(),"ONE");  
        tx.commit();  
    }  

}
```

FragmentOne文件

```java
public class FragmentOne extends Fragment implements OnClickListener {  

    private Button mBtn;  

    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_one, container, false);  
        mBtn = (Button) view.findViewById(R.id.id_fragment_one_btn);  
        mBtn.setOnClickListener(this);  
        return view;  
    }  

    @Override  
    public void onClick(View v) {  
        FragmentTwo fTwo = new FragmentTwo();  
        FragmentManager fm = getFragmentManager();  
        FragmentTransaction tx = fm.beginTransaction();  
        tx.replace(R.id.id_content, fTwo, "TWO");  
        tx.addToBackStack(null);  
        tx.commit();  
    }  

}
```

Fragment的点击事件里写的是replace方法，相当于remove和add的合体，并且如果不添加事务到回退栈，前一个Fragment实例会被销毁。这里很明显，我们调用tx.addToBackStack(null)将当前的事务添加到了回退栈，所以FragmentOne实例不会被销毁，但是视图层次依然会被销毁，即会调用onDestoryView和onCreateView。**所以，当之后我们从FragmentTwo返回到前一个页面的时候，视图层仍旧是重新按照代码绘制，这里仅仅是是实例没有销毁**，因此显示的页面中没有change几个字。

FragmentTwo文件

```java
public class FragmentTwo extends Fragment implements OnClickListener {  

    private Button mBtn ;  

    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_two, container, false);  
        mBtn = (Button) view.findViewById(R.id.id_fragment_two_btn);  
        mBtn.setOnClickListener(this);  
        return view ;   
    }  

    @Override  
    public void onClick(View v)  {  
        FragmentThree fThree = new FragmentThree();  
        FragmentManager fm = getFragmentManager();  
        FragmentTransaction tx = fm.beginTransaction();  
        tx.hide(this);  
        tx.add(R.id.id_content , fThree, "THREE");  
        //tx.replace(R.id.id_content, fThree, "THREE");  
        tx.addToBackStack(null);  
        tx.commit();  
    }  

}
```

这里点击时，我们没有使用replace，而是先隐藏了当前的Fragment，然后添加了FragmentThree的实例，最后将事务添加到回退栈。这样做的目的是为了给大家提供一种方案：如果不希望视图重绘该怎么做，请再次仔细看效果图，我们在FragmentTwo的EditText填写的内容，用户点击返回键回来时，内容还在。

FragmentThree文件

```java
public class FragmentThree extends Fragment implements OnClickListener {  

    private Button mBtn;  

    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_three, container, false);  
        mBtn = (Button) view.findViewById(R.id.id_fragment_three_btn);  
        mBtn.setOnClickListener(this);  
        return view;  
    }  

    @Override  
    public void onClick(View v) {  
        Toast.makeText(getActivity(), " i am a btn in Fragment three",  
                Toast.LENGTH_SHORT).show();  
    }  

}
```

如果你还是不明白请仔细将上面的代码反复敲几遍

### 4. Fragment与Activity之间的通信

Fragment依附于Activity存在，因此与Activity之间的通信可以归纳为以下几点：

- [ ] 如果你Activity中包含自己管理的Fragment的引用，可以通过引用直接访问所有的Fragment的public方法
- [ ] 如果Activity中未保存任何Fragment的引用，那么没关系，每个Fragment都有一个唯一的TAG或者ID,可以通过getFragmentManager.findFragmentByTag()或者findFragmentById()获得任何Fragment实例，然后进行操作
- [ ] Fragment中可以通过getActivity()得到当前绑定的Activity的实例，然后进行操作。

### 5. Fragment与Activity通信的优化

因为要考虑Fragment的重复使用，所以必须降低Fragment与Activity的耦合，而且Fragment更不应该直接操作别的Fragment，毕竟Fragment操作应该由它的管理者Activity来决定。

实现与上一个代码案例一模一样的功能与效果

FragmentOne文件

```java
public class FragmentOne extends Fragment implements OnClickListener {  

	private Button mBtn;  
    //设置按钮点击的回调
    public interface FOneBtnClickListener {  
        void onFOneBtnClick();  
    }  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_one, container, false);  
        mBtn = (Button) view.findViewById(R.id.id_fragment_one_btn);  
        mBtn.setOnClickListener(this);  
        return view;  
    }  
   //交给宿主Activity处理，如果它希望处理   
    @Override  
    public void onClick(View v) {  
        if (getActivity() instanceof FOneBtnClickListener) {  
            ((FOneBtnClickListener) getActivity()).onFOneBtnClick();  
        }  
    }  

}
```

可以看到，现在的FragmentOne不和任何Activity耦合，任何Activity都可以使用，并且我们声明了一个接口，来回调其点击事件，想要重写其点击事件的Activity实现此接口即可，可以看到我们在onClick中首先判断了当前绑定的Activity是否实现了该接口，如果实现了则调用。

FragmentTwo文件

```java
public class FragmentTwo extends Fragment implements OnClickListener {

    private Button mBtn ;  
    private FTwoBtnClickListener fTwoBtnClickListener ;  

    public interface FTwoBtnClickListener {  
        void onFTwoBtnClick();  
    }  

    //设置回调接口  
    public void setfTwoBtnClickListener(FTwoBtnClickListener fTwoBtnClickListener) {  
        this.fTwoBtnClickListener = fTwoBtnClickListener;  
    }  

    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        View view = inflater.inflate(R.layout.fragment_two, container, false);  
        mBtn = (Button) view.findViewById(R.id.id_fragment_two_btn);  
        mBtn.setOnClickListener(this);  
        return view ;   
    }  

    @Override  
    public void onClick(View v) {  
        if(fTwoBtnClickListener != null)  {  
            fTwoBtnClickListener.onFTwoBtnClick();  
        }  
    }  

}
```

与FragmentOne极其类似，但是我们提供了setListener这样的方法，意味着Activity不仅需要实现该接口，还必须显示调用mFTwo.setfTwoBtnClickListener(this)。

MainActivity.class文件

```java
public class MainActivity extends Activity implements FOneBtnClickListener,  
        FTwoBtnClickListener {  

    private FragmentOne mFOne;  
    private FragmentTwo mFTwo;  
    private FragmentThree mFThree;  //FragmentThree代码参考上一个例子中的代码

    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        requestWindowFeature(Window.FEATURE_NO_TITLE);  
        setContentView(R.layout.activity_main);  

        mFOne = new FragmentOne();  
        FragmentManager fm = getFragmentManager();  
        FragmentTransaction tx = fm.beginTransaction();  
        tx.add(R.id.id_content, mFOne, "ONE");  
        tx.commit();  
    }  

    //FragmentOne 按钮点击时的回调  
    @Override  
    public void onFOneBtnClick() { 
        if (mFTwo == null) {  
            mFTwo = new FragmentTwo();  
            mFTwo.setfTwoBtnClickListener(this);  
        }  
        FragmentManager fm = getFragmentManager();  
        FragmentTransaction tx = fm.beginTransaction();  
        tx.replace(R.id.id_content, mFTwo, "TWO");  
        tx.addToBackStack(null);  
        tx.commit();  
    }  

    //FragmentTwo按钮点击时的回调  
    @Override  
    public void onFTwoBtnClick() {  
        if (mFThree == null) {  
            mFThree = new FragmentThree();  
        }  
        FragmentManager fm = getFragmentManager();  
        FragmentTransaction tx = fm.beginTransaction();  
        tx.hide(mFTwo);  
        tx.add(R.id.id_content, mFThree, "THREE");  
        //tx.replace(R.id.id_content, fThree, "THREE");  
        tx.addToBackStack(null);  
        tx.commit();  
    }  

}
```

代码重构结束，与开始的效果一模一样。上面两种通信方式都是值得推荐的，随便选择一种自己喜欢的。这里再提一下：虽然Fragment和Activity可以通过getActivity与findFragmentByTag或者findFragmentById，进行任何操作，甚至在Fragment里面操作另外的Fragment，但是没有特殊理由是绝对不提倡的。Activity担任的是Fragment间类似总线一样的角色，应当由它决定Fragment如何操作。另外虽然Fragment不能响应Intent打开，但是Activity可以，Activity可以接收Intent，然后根据参数判断显示哪个Fragment。

### 6. 如何处理运行时配置发生变化

在Activity的学习中我们都知道，当屏幕旋转时，是对屏幕上的视图进行了重新绘制。因为当屏幕发生旋转，Activity发生重新启动，默认的Activity中的Fragment也会跟着Activity重新创建。所以，不断的旋转就不断绘制，这是一种很耗费内存资源的操作，那么如何来进行优化？

Fragment的文件

```java
public class FragmentOne extends Fragment {

    private static final String TAG = "FragmentOne";  

    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        Log.e(TAG, "onCreateView");  
        View view = inflater.inflate(R.layout.fragment_one, container, false);  
        return view;  
    }  

    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        // TODO Auto-generated method stub  
        super.onCreate(savedInstanceState);  
        Log.e(TAG, "onCreate");  
    }  

    @Override  
    public void onDestroyView() {  
        // TODO Auto-generated method stub  
        super.onDestroyView();  
        Log.e(TAG, "onDestroyView");  
    }  

    @Override  
    public void onDestroy() {  
        // TODO Auto-generated method stub  
        super.onDestroy();  
        Log.e(TAG, "onDestroy");  
    }  

}
```

然后你多次翻转屏幕都会打印如下log

```
07-20 08:18:46.651: E/FragmentOne(1633): onCreate  
07-20 08:18:46.651: E/FragmentOne(1633): onCreate  
07-20 08:18:46.651: E/FragmentOne(1633): onCreate  
07-20 08:18:46.681: E/FragmentOne(1633): onCreateView  
07-20 08:18:46.831: E/FragmentOne(1633): onCreateView  
07-20 08:18:46.891: E/FragmentOne(1633): onCreateView
```

因为当屏幕发生旋转，Activity发生重新启动，默认的Activity中的Fragment也会跟着Activity重新创建；这样造成当旋转的时候，本身存在的Fragment会重新启动，然后当执行Activity的onCreate时，又会再次实例化一个新的Fragment，这就是出现的原因。

那么如何解决呢：

通过检查onCreate的参数Bundle savedInstanceState就可以判断，当前是否发生Activity的重新创建

默认的savedInstanceState会存储一些数据，包括Fragment的实例

所以，我们简单改一下代码，判断只有在savedInstanceState==null时，才进行创建Fragment实例

MainActivity.class文件

```Java
public class MainActivity extends Activity {

    private static final String TAG = "FragmentOne";  
    private FragmentOne mFOne;  

    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        requestWindowFeature(Window.FEATURE_NO_TITLE);  
        setContentView(R.layout.activity_main);  

        Log.e(TAG, savedInstanceState+"");  

        if(savedInstanceState == null) {  
            mFOne = new FragmentOne();  
            FragmentManager fm = getFragmentManager();  
            FragmentTransaction tx = fm.beginTransaction();  
            tx.add(R.id.id_content, mFOne, "ONE");  
            tx.commit();  
        }  
    }  

}
```

现在无论进行多次旋转都只会有一个Fragment实例在Activity中，现在还存在一个问题，就是重新绘制时，Fragment发生重建，原本的数据如何保持？
和Activity类似，Fragment也有onSaveInstanceState的方法，在此方法中进行保存数据，然后在onCreate或者onCreateView或者onActivityCreated进行恢复都可以。

## 六、Android消息机制

### 1. 消息机制概述

#### 1.1 消息机制的简介

在Android中使用消息机制，我们首先想到的就是Handler。没错，Handler是Android消息机制的上层接口。Handler的使用过程很简单，通过它可以轻松地将一个任务切换到Handler所在的线程中去执行。通常情况下，Handler的使用场景就是更新UI。  
如下就是使用消息机制的一个简单实例：

```java
public class Activity extends android.app.Activity {
    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            System.out.println(msg.what);
        }
    };
    @Override
    public void onCreate(Bundle savedInstanceState, PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                //...............耗时操作
                Message message = Message.obtain();
                message.what = 1;
                mHandler.sendMessage(message);
            }
        }).start();
    }
}
```

在子线程中，进行耗时操作，执行完操作后，发送消息，通知主线程更新UI。这便是消息机制的典型应用场景。我们通常只会接触到Handler和Message来完成消息机制，其实内部还有两大助手来共同完成消息传递。

#### 1.2 消息机制的模型

消息机制主要包含：MessageQueue，Handler和Looper这三大部分，以及Message。

**Message：**需要传递的消息，可以传递数据；

**MessageQueue：**消息队列，但是它的内部实现并不是用的队列，实际上是通过一个单链表的数据结构来维护消息列表，因为单链表在插入和删除上比较有优势。主要功能向消息池投递消息\(MessageQueue.enqueueMessage\)和取走消息池的消息\(MessageQueue.next\)；

**Handler：**消息辅助类，主要功能向消息池发送各种消息事件\(Handler.sendMessage\)和处理相应消息事件\(Handler.handleMessage\)；

**Looper：**不断循环执行\(Looper.loop\)，从MessageQueue中读取消息，按分发机制将消息分发给目标处理者。

#### 1.3 消息机制的架构

**消息机制的运行流程：**在子线程执行完耗时操作，当Handler发送消息时，将会调用`MessageQueue.enqueueMessage`，向消息队列中添加消息。当通过`Looper.loop`开启循环后，会不断地从线程池中读取消息，即调用`MessageQueue.next`，然后调用目标Handler（即发送该消息的Handler）的`dispatchMessage`方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用`handleMessage`方法，接收消息，处理消息。

![](http://upload-images.jianshu.io/upload_images/3985563-d7da4f5ba49f6887.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

**MessageQueue，Handler和Looper三者之间的关系：**每个线程中只能存在一个Looper，Looper是保存在ThreadLocal中的。主线程（UI线程）已经创建了一个Looper，所以在主线程中不需要再创建Looper，但是在其他线程中需要创建Looper。每个线程中可以有多个Handler，即一个Looper可以处理来自多个Handler的消息。 Looper中维护一个MessageQueue，来维护消息队列，消息队列中的Message可以来自不同的Handler。

![](http://upload-images.jianshu.io/upload_images/3985563-88a27b5906166c63.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

下面是消息机制的整体架构图，接下来我们将慢慢解剖整个架构。

![](http://upload-images.jianshu.io/upload_images/3985563-6c25004471646c1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

从中我们可以看出：  
Looper有一个MessageQueue消息队列；  
MessageQueue有一组待处理的Message；  
Message中记录发送和处理消息的Handler；  
Handler中有Looper和MessageQueue。

### 2. 消息机制的源码解析

#### 2.1 Looper

要想使用消息机制，首先要创建一个Looper。  

**初始化Looper**  

无参情况下，默认调用`prepare(true);`表示的是这个Looper可以退出，而对于false的情况则表示当前Looper不可以退出。

```java
 public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

这里看出，不能重复创建Looper，只能创建一个。创建Looper,并保存在ThreadLocal。其中ThreadLocal是线程本地存储区（Thread Local Storage，简称为TLS），每个线程都有自己的私有的本地存储区域，不同线程之间彼此不能访问对方的TLS区域。  

**开启Looper**

```java
public static void loop() {
    final Looper me = myLooper();  //获取TLS存储的Looper对象 
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;  //获取Looper对象中的消息队列

    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) { //进入loop的主循环方法
        Message msg = queue.next(); //可能会阻塞,因为next()方法可能会无限循环
        if (msg == null) { //消息为空，则退出循环
            return;
        }

        Printer logging = me.mLogging;  //默认为null，可通过setMessageLogging()方法来指定输出，用于debug功能
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        msg.target.dispatchMessage(msg); //获取msg的目标Handler，然后用于分发Message 
        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
         
        }
        msg.recycleUnchecked(); 
    }
}
```

loop\(\)进入循环模式，不断重复下面的操作，直到消息为空时退出循环:  

读取MessageQueue的下一条Message（关于next\(\)，后面详细介绍）;  

把Message分发给相应的target。

**当next\(\)取出下一条消息时，队列中已经没有消息时，next\(\)会无限循环，产生阻塞。等待MessageQueue中加入消息，然后重新唤醒。**

**主线程中不需要自己创建Looper，这是由于在程序启动的时候，系统已经帮我们自动调用了**`Looper.prepare()`**方法。查看ActivityThread中的**`main()`**方法，代码如下所示：**

```java
  public static void main(String[] args) {
..........................
        Looper.prepareMainLooper();
  ..........................
        Looper.loop();
  ..........................

    }
```

其中```prepareMainLooper()``方法会调用`prepare(false)`方法。

#### 2.2 Handler

**创建Handler**

```java
public Handler() {
    this(null, false);
}

public Handler(Callback callback, boolean async) {
   .................................
    //必须先执行Looper.prepare()，才能获取Looper对象，否则为null.
    mLooper = Looper.myLooper();  //从当前线程的TLS中获取Looper对象
    if (mLooper == null) {
        throw new RuntimeException("");
    }
    mQueue = mLooper.mQueue; //消息队列，来自Looper对象
    mCallback = callback;  //回调方法
    mAsynchronous = async; //设置消息是否为异步处理方式
}
```

对于Handler的无参构造方法，默认采用当前线程TLS中的Looper对象，并且callback回调方法为null，且消息为同步处理方式。只要执行的`Looper.prepare()`方法，那么便可以获取有效的Looper对象。

#### 2.3 发送消息

发送消息有几种方式，但是归根结底都是调用了`sendMessageAtTime()`方法。

在子线程中通过Handler的post\(\)方式或send\(\)方式发送消息，最终都是调用了`sendMessageAtTime()`方法。

**post方法**

```java
 public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
public final boolean postAtTime(Runnable r, long uptimeMillis)
    {
        return sendMessageAtTime(getPostMessage(r), uptimeMillis);
    }
 public final boolean postAtTime(Runnable r, Object token, long uptimeMillis)
    {
        return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
    }
 public final boolean postDelayed(Runnable r, long delayMillis)
    {
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }
```

**send方法**

```java
public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
 public final boolean sendEmptyMessage(int what)
    {
        return sendEmptyMessageDelayed(what, 0);
    } 
public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageDelayed(msg, delayMillis);
    }
 public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageAtTime(msg, uptimeMillis);
    }
 public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```

就连子线程中调用Activity中的runOnUiThread\(\)中更新UI，其实也是发送消息通知主线程更新UI，最终也会调用`sendMessageAtTime()`方法。

```java
 public final void runOnUiThread(Runnable action) {
        if (Thread.currentThread() != mUiThread) {
            mHandler.post(action);
        } else {
            action.run();
        }
    }
```

如果当前的线程不等于UI线程\(主线程\)，就去调用Handler的post\(\)方法，最终会调用`sendMessageAtTime()`方法。否则就直接调用Runnable对象的run\(\)方法。

**sendMessageAtTime\(\)**

```java
 public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
       //其中mQueue是消息队列，从Looper中获取的
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        //调用enqueueMessage方法
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

```java
 private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        //调用MessageQueue的enqueueMessage方法
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

可以看到sendMessageAtTime\(\)\`方法的作用很简单，就是调用MessageQueue的enqueueMessage\(\)方法，往消息队列中添加一个消息。  

下面来看enqueueMessage\(\)方法的具体执行逻辑。  

**enqueueMessage\(\)**

```java
boolean enqueueMessage(Message msg, long when) {
    // 每一个Message必须有一个target
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }
    synchronized (this) {
        if (mQuitting) {  //正在退出时，回收msg，加入到消息池
            msg.recycle();
            return false;
        }
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            //p为null(代表MessageQueue没有消息） 或者msg的触发时间是队列中最早的， 则进入该该分支
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked; 
        } else {
            //将消息按时间顺序插入到MessageQueue。一般地，不需要唤醒事件队列，除非
            //消息队头存在barrier，并且同时Message是队列中最早的异步消息。
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p;
            prev.next = msg;
        }
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

MessageQueue是按照Message触发时间的先后顺序排列的，队头的消息是将要最早触发的消息。当有消息需要加入消息队列时，会从队列头开始遍历，直到找到消息应该插入的合适位置，以保证所有消息的时间顺序。

#### 2.4 获取消息

当发送了消息后，在MessageQueue维护了消息队列，然后在Looper中通过`loop()`方法，不断地获取消息。上面对`loop()`方法进行了介绍，其中最重要的是调用了`queue.next()`方法,通过该方法来提取下一条信息。下面我们来看一下`next()`方法的具体流程。  
**next\(\)**

```java
Message next() {
    final long ptr = mPtr;
    if (ptr == 0) { //当消息循环已经退出，则直接返回
        return null;
    }
    int pendingIdleHandlerCount = -1; // 循环迭代的首次为-1
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        //阻塞操作，当等待nextPollTimeoutMillis时长，或者消息队列被唤醒，都会返回
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                //当消息Handler为空时，查询MessageQueue中的下一条异步消息msg，为空则退出循环。
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    //当异步消息触发时间大于当前时间，则设置下一次轮询的超时时长
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 获取一条消息，并返回
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    //设置消息的使用状态，即flags |= FLAG_IN_USE
                    msg.markInUse();
                    return msg;   //成功地获取MessageQueue中的下一条即将要执行的消息
                }
            } else {
                //没有消息
                nextPollTimeoutMillis = -1;
            }
         //消息正在退出，返回null
            if (mQuitting) {
                dispose();
                return null;
            }
            ...............................
    }
}
```

nativePollOnce是阻塞操作，其中nextPollTimeoutMillis代表下一个消息到来前，还需要等待的时长；当nextPollTimeoutMillis = -1时，表示消息队列中无消息，会一直等待下去。  
可以看出`next()`方法根据消息的触发时间，获取下一条需要执行的消息,队列中消息为空时，则会进行阻塞操作。

#### 2.5 分发消息

在loop\(\)方法中，获取到下一条消息后，执行`msg.target.dispatchMessage(msg)`，来分发消息到目标Handler对象。  下面就来具体看下`dispatchMessage(msg)`方法的执行流程。  

**dispatchMessage\(\)**

```java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        //当Message存在回调方法，回调msg.callback.run()方法；
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //当Handler存在Callback成员变量时，回调方法handleMessage()；
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //Handler自身的回调方法handleMessage()
        handleMessage(msg);
    }
}
```

```java
private static void handleCallback(Message message) {
        message.callback.run();
    }
```

**分发消息流程：**  

当Message的`msg.callback`不为空时，则回调方法`msg.callback.run()`;  

当Handler的`mCallback`不为空时，则回调方法`mCallback.handleMessage(msg)`；  

最后调用Handler自身的回调方法`handleMessage()`，该方法默认为空，Handler子类通过覆写该方法来完成具体的逻辑。

**消息分发的优先级：**  

Message的回调方法：`message.callback.run()`，优先级最高；  

Handler中Callback的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；  

Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。

对于很多情况下，消息分发后的处理方法是第3种情况，即`Handler.handleMessage()`，一般地往往通过覆写该方法从而实现自己的业务逻辑。

### 3. 总结

以上便是消息机制的原理，以及从源码角度来解析消息机制的运行过程。可以简单地用下图来理解。

![](http://upload-images.jianshu.io/upload_images/3985563-b3295b67a2b0477f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)



## 七、Android事件分发机制★

### 1. 基础认知

**事件分发的对象是谁？**

**答：事件**

- 当用户触摸屏幕时（View或ViewGroup派生的控件），将产生点击事件（Touch事件）。

  > Touch事件相关细节（发生触摸的位置、时间、历史记录、手势动作等）被封装成MotionEvent对象

- 主要发生的Touch事件有如下四种：

  - MotionEvent.ACTION_DOWN：按下View（所有事件的开始）
  - MotionEvent.ACTION_MOVE：滑动View
  - MotionEvent.ACTION_CANCEL：非人为原因结束本次事件
  - MotionEvent.ACTION_UP：抬起View（与DOWN对应）

- 事件列：从手指接触屏幕至手指离开屏幕，这个过程产生的一系列事件
  任何事件列都是以DOWN事件开始，UP事件结束，中间有无数的MOVE事件，如下图：

![](http://upload-images.jianshu.io/upload_images/944365-79b1e86793514e99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

即当一个MotionEvent 产生后，系统需要把这个事件传递给一个具体的 View 去处理, 

**事件分发的本质**

**答：将点击事件（MotionEvent）向某个View进行传递并最终得到处理**

> 即当一个点击事件发生后，系统需要将这个事件传递给一个具体的View去处理。**这个事件传递的过程就是分发过程。**

**事件在哪些对象之间进行传递？**

**答：Activity、ViewGroup、View**

> 一个点击事件产生后，传递顺序是：Activity（Window） -> ViewGroup -> View

- Android的UI界面是由Activity、ViewGroup、View及其派生类组合而成的

  ![](http://upload-images.jianshu.io/upload_images/944365-ece40d4524784ffa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- View是所有UI组件的基类

  > 一般Button、ImageView、TextView等控件都是继承父类View

- ViewGroup是容纳UI组件的容器，即一组View的集合（包含很多子View和子VewGroup），

  > 1. 其本身也是从View派生的，即ViewGroup是View的子类
  > 2. 是Android所有布局的父类或间接父类：项目用到的布局（LinearLayout、RelativeLayout等），都继承自ViewGroup，即属于ViewGroup子类。
  > 3. 与普通View的区别：ViewGroup实际上也是一个View，只不过比起View，它多了可以包含子View和定义布局参数的功能。

**事件分发过程由哪些方法协作完成？**

**答：dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()**

![](http://upload-images.jianshu.io/upload_images/944365-a5eeeae6ee27682a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 下文会对这3个方法进行详细介绍

**总结**

- Android事件分发机制的本质是要解决：

  点击事件由哪个对象发出，经过哪些对象，最终达到哪个对象并最终得到处理。

  > 这里的对象是指Activity、ViewGroup、View

- Android中事件分发顺序：**Activity（Window） -> ViewGroup -> View**

- 事件分发过程由dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()三个方法协助完成

经过上述3个问题，相信大家已经对Android的事件分发有了感性的认知，接下来，我将详细介绍Android事件分发机制。

### 2. 事件分发机制方法和流程介绍

- 事件分发过程由dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()三个方法p完成，如下图1：

![](http://upload-images.jianshu.io/upload_images/944365-74bdb5c375a37100.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方法详细介绍

- Android事件分发流程如下：（必须熟记）

  > Android事件分发顺序：**Activity（Window） -> ViewGroup -> View**

![](http://upload-images.jianshu.io/upload_images/944365-aa8416fc6d2e5ecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中：

- super：调用父类方法
- true：消费事件，即事件不继续往下传递
- false：不消费事件，事件也不继续往下传递 / 交由给父控件onTouchEvent（）处理

**接下来，我将详细介绍这3个方法及相关流程。**

**dispatchTouchEvent()**

| 属性     | 介绍                                             |
| -------- | ------------------------------------------------ |
| 使用对象 | Activity、ViewGroup、View                        |
| 作用     | 分发点击事件                                     |
| 调用时刻 | 当点击事件能够传递给当前View时，该方法就会被调用 |
| 返回结果 | 是否消费当前事件，详细情况如下：                 |

**1. 默认情况：根据当前对象的不同而返回方法不同**

| 对象      | 返回方法                   | 备注                                      |
| --------- | -------------------------- | ----------------------------------------- |
| Activity  | super.dispatchTouchEvent() | 即调用父类ViewGroup的dispatchTouchEvent() |
| ViewGroup | onIntercepTouchEvent()     | 即调用自身的onIntercepTouchEvent()        |
| View      | onTouchEvent（）           | 即调用自身的onTouchEvent（）              |

流程解析

**2. 返回true**

- 消费事件
- 事件不会往下传递
- 后续事件（Move、Up）会继续分发到该View
- 流程图如下：

![](http://upload-images.jianshu.io/upload_images/944365-919f10d7d671cdea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3. 返回false**

- 不消费事件

- 事件不会往下传递

- 将事件回传给父控件的onTouchEvent()处理

  > Activity例外：返回false=消费事件

- 后续事件（Move、Up）会继续分发到该View(与onTouchEvent()区别）

- 流程图如下：

  ![](http://upload-images.jianshu.io/upload_images/944365-8b45e9551e833955.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**onTouchEvent()**

| 属性     | 介绍                                     |
| -------- | ---------------------------------------- |
| 使用对象 | Activity、ViewGroup、View                |
| 作用     | 处理点击事件                             |
| 调用时刻 | 在dispatchTouchEvent()内部调用           |
| 返回结果 | 是否消费（处理）当前事件，详细情况如下： |

> 与dispatchTouchEvent()类似

**1. 返回true**

- 自己处理（消费）该事情

- 事件停止传递

- 该事件序列的后续事件（Move、Up）让其处理；

- 流程图如下：

  ![](http://upload-images.jianshu.io/upload_images/944365-cd63e2c3b7f89b47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**2. 返回false（同默认实现：调用父类onTouchEvent()）**

- 不处理（消费）该事件

- 事件往上传递给父控件的onTouchEvent()处理

- 当前View不再接受此事件列的其他事件（Move、Up）；

- 流程图如下：

  ![](http://upload-images.jianshu.io/upload_images/944365-9ce60722ac0a9a36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**onInterceptTouchEvent()**

| 属性     | 介绍                                      |
| -------- | ----------------------------------------- |
| 使用对象 | ViewGroup（注：Activity、View都没该方法） |
| 作用     | 拦截事件，即自己处理该事件                |
| 调用时刻 | 在ViewGroup的dispatchTouchEvent()内部调用 |
| 返回结果 | 是否拦截当前事件，详细情况如下：          |

返回结果

![](http://upload-images.jianshu.io/upload_images/944365-6b34c1017b14104d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 流程图如下：

![](http://upload-images.jianshu.io/upload_images/944365-37be4474ef7a1741.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**三者关系**

下面将用一段伪代码来阐述上述三个方法的关系和点击事件传递规则

```java
// 点击事件产生后，会直接调用dispatchTouchEvent（）方法
public boolean dispatchTouchEvent(MotionEvent ev) {

    //代表是否消耗事件
    boolean consume = false;


    if (onInterceptTouchEvent(ev)) {
    //如果onInterceptTouchEvent()返回true则代表当前View拦截了点击事件
    //则该点击事件则会交给当前View进行处理
    //即调用onTouchEvent (）方法去处理点击事件
      consume = onTouchEvent (ev) ;

    } else {
      //如果onInterceptTouchEvent()返回false则代表当前View不拦截点击事件
      //则该点击事件则会继续传递给它的子元素
      //子元素的dispatchTouchEvent（）就会被调用，重复上述过程
      //直到点击事件被最终处理为止
      consume = child.dispatchTouchEvent (ev) ;
    }

    return consume;
   }
```

### 3. 事件分发场景介绍

下面我将利用例子来说明常见的点击事件传递情况

#### 3.1 背景描述

我们将要讨论的布局层次如下：

![](http://upload-images.jianshu.io/upload_images/944365-ecac6247816a3db1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 最外层：Activiy A，包含两个子View：ViewGroup B、View C
- 中间层：ViewGroup B，包含一个子View：View C
- 最内层：View C

假设用户首先触摸到屏幕上View C上的某个点（如图中黄色区域），那么Action_DOWN事件就在该点产生，然后用户移动手指并最后离开屏幕。

#### 3.2 一般的事件传递情况

一般的事件传递场景有：

- 默认情况
- 处理事件
- 拦截DOWN事件
- 拦截后续事件（MOVE、UP）

**3.2.1 默认情况**

- 即不对控件里的方法(dispatchTouchEvent()、onTouchEvent()、onInterceptTouchEvent())进行重写或更改返回值
- 那么调用的是这3个方法的默认实现：调用父类的方法
- 事件传递情况：（如图下所示）
  - 从Activity A---->ViewGroup B--->View C，从上往下调用dispatchTouchEvent()
  - 再由View C--->ViewGroup B --->Activity A，从下往上调用onTouchEvent()

![](http://upload-images.jianshu.io/upload_images/944365-69229fd4a804c0f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注：虽然ViewGroup B的onInterceptTouchEvent方法对DOWN事件返回了false，后续的事件（MOVE、UP）依然会传递给它的onInterceptTouchEvent()

> 这一点与onTouchEvent的行为是不一样的。

**3.2.2 处理事件**

假设View C希望处理这个点击事件，即C被设置成可点击的（Clickable）或者覆写了C的onTouchEvent方法返回true。

> 最常见的：设置Button按钮来响应点击事件

事件传递情况：（如下图）

- DOWN事件被传递给C的onTouchEvent方法，该方法返回true，表示处理这个事件
- 因为C正在处理这个事件，那么DOWN事件将不再往上传递给B和A的onTouchEvent()；
- 该事件列的其他事件（Move、Up）也将传递给C的onTouchEvent()

![](http://upload-images.jianshu.io/upload_images/944365-26290b0d33370853.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3.2.3 拦截DOWN事件**

假设ViewGroup B希望处理这个点击事件，即B覆写了onInterceptTouchEvent()返回true、onTouchEvent()返回true。
事件传递情况：（如下图）

- DOWN事件被传递给B的onInterceptTouchEvent()方法，该方法返回true，表示拦截这个事件，即自己处理这个事件（不再往下传递）

- 调用onTouchEvent()处理事件（DOWN事件将不再往上传递给A的onTouchEvent()）

- 该事件列的其他事件（Move、Up）将直接传递给B的onTouchEvent()

  > 该事件列的其他事件（Move、Up）将不会再传递给B的onInterceptTouchEvent方法，该方法一旦返回一次true，就再也不会被调用了。

![](http://upload-images.jianshu.io/upload_images/944365-d8acd35d3a50e091.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3.2.4 拦截DOWN的后续事件**

假设ViewGroup B没有拦截DOWN事件（还是View C来处理DOWN事件），但它拦截了接下来的MOVE事件。

- DOWN事件传递到C的onTouchEvent方法，返回了true。

- 在后续到来的MOVE事件，B的onInterceptTouchEvent方法返回true拦截该MOVE事件，但该事件并没有传递给B；这个MOVE事件将会被系统变成一个CANCEL事件传递给C的onTouchEvent方法

- 后续又来了一个MOVE事件，该MOVE事件才会直接传递给B的onTouchEvent()

  > 1. 后续事件将直接传递给B的onTouchEvent()处理
  > 2. 后续事件将不会再传递给B的onInterceptTouchEvent方法，该方法一旦返回一次true，就再也不会被调用了。

- C再也不会收到该事件列产生的后续事件。

  ![](http://upload-images.jianshu.io/upload_images/944365-edb138a07cbb990c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

特别注意：

- 如果ViewGroup A 拦截了一个半路的事件（如MOVE），这个事件将会被系统变成一个CANCEL事件并传递给之前处理该事件的子View；
- 该事件不会再传递给ViewGroup A的onTouchEvent()
- 只有再到来的事件才会传递到ViewGroup A的onTouchEvent()

### 4. Android事件分发机制源码分析

- Android中事件分发顺序：**Activity（Window） -> ViewGroup -> View**，再次贴出下图：

![](http://upload-images.jianshu.io/upload_images/944365-aa8416fc6d2e5ecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中：

- super：调用父类方法
- true：消费事件，即事件不继续往下传递
- false：不消费事件，事件继续往下传递 / 交由给父控件onTouchEvent（）处理

**所以，要想充分理解Android分发机制，本质上是要理解：**

- Activity对点击事件的分发机制
- ViewGroup对点击事件的分发机制
- View对点击事件的分发机制

接下来，我将通过源码分析详细介绍Activity、View和ViewGroup的事件分发机制

#### 4.1 Activity的事件分发机制

**源码分析**

- 当一个点击事件发生时，事件最先传到Activity的dispatchTouchEvent()进行事件分发

  > 具体是由Activity的Window来完成

- 我们来看下Activity的dispatchTouchEvent()的源码

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
        //关注点1
        //一般事件列开始都是DOWN，所以这里基本是true
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            //关注点2
            onUserInteraction();
        }
        //关注点3
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
```

**关注点1**
一般事件列开始都是DOWN（按下按钮），所以这里返回true，执行onUserInteraction()

**关注点2**
先来看下onUserInteraction()源码

```java
  /**
     * Called whenever a key, touch, or trackball event is dispatched to the
     * activity.  Implement this method if you wish to know that the user has
     * interacted with the device in some way while your activity is running.
     * This callback and {@link #onUserLeaveHint} are intended to help
     * activities manage status bar notifications intelligently; specifically,
     * for helping activities determine the proper time to cancel a notfication.
     *
     * <p>All calls to your activity's {@link #onUserLeaveHint} callback will
     * be accompanied by calls to {@link #onUserInteraction}.  This
     * ensures that your activity will be told of relevant user activity such
     * as pulling down the notification pane and touching an item there.
     *
     * <p>Note that this callback will be invoked for the touch down action
     * that begins a touch gesture, but may not be invoked for the touch-moved
     * and touch-up actions that follow.
     *
     * @see #onUserLeaveHint()
     */
public void onUserInteraction() { 
}
```

从源码可以看出：

- 该方法为空方法
- 从注释得知：当此activity在栈顶时，触屏点击按home，back，menu键等都会触发此方法
- 所以onUserInteraction()主要用于屏保

**关注点3**

- Window类是抽象类，且PhoneWindow是Window类的唯一实现类

- superDispatchTouchEvent(ev)是抽象方法

- 通过PhoneWindow类中看一下superDispatchTouchEvent()的作用

  ```Java
  @Override
  public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
  //mDecor是DecorView的实例
  //DecorView是视图的顶层view，继承自FrameLayout，是所有界面的父类
  }
  ```

- 接下来我们看mDecor.superDispatchTouchEvent(event)：

```java
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
//DecorView继承自FrameLayout
//那么它的父类就是ViewGroup
而super.dispatchTouchEvent(event)方法，其实就应该是ViewGroup的dispatchTouchEvent()

}
```

所以：

- **执行getWindow().superDispatchTouchEvent(ev)实际上是执行了ViewGroup.dispatchTouchEvent(event)**

- 再回到最初的代码：

  ```java
  public boolean dispatchTouchEvent(MotionEvent ev) {
  
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            //关注点2
            onUserInteraction();
        }
        //关注点3
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
  ```

  由于一般事件列开始都是DOWN，所以这里返回true，基本上都会进入`getWindow().superDispatchTouchEvent(ev)`的判断

- 所以，执行**Activity.dispatchTouchEvent(ev)实际上是执行了ViewGroup.dispatchTouchEvent(event)**

- 这样事件就从 Activity 传递到了 ViewGroup 

**汇总**

当一个点击事件发生时，调用顺序如下

1. 事件最先传到Activity的dispatchTouchEvent()进行事件分发
2. 调用Window类实现类PhoneWindow的superDispatchTouchEvent()
3. 调用DecorView的superDispatchTouchEvent()
4. 最终调用DecorView父类的dispatchTouchEvent()，**即ViewGroup的dispatchTouchEvent()**

**结论**

- 当一个点击事件发生时，事件最先传到Activity的dispatchTouchEvent()进行事件分发，最终是调用了ViewGroup的dispatchTouchEvent()方法
- 这样事件就从 Activity 传递到了 ViewGroup 

#### 4.2 ViewGroup事件的分发机制

在讲解ViewGroup事件的分发机制之前我们先来看个Demo

**Demo讲解**

**布局如下：**

![](http://upload-images.jianshu.io/upload_images/944365-b0bf3dd7ad41b335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**结果测试：**
只点击Button

![](http://upload-images.jianshu.io/upload_images/944365-e3fd145ff5e454a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再点击空白处

![](http://upload-images.jianshu.io/upload_images/944365-252e4ef9f7243705.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上面的测试结果发现：

- 当点击Button时，执行Button的onClick()，但ViewGroupLayout注册的onTouch（）不会执行
- 只有点击空白区域时才会执行ViewGroupLayout的onTouch（）;
- 结论：Button的onClick()将事件消费掉了，因此事件不会再继续向下传递。

接下来，我们开始进行ViewGroup事件分发的源码分析

**源码分析**

ViewGroup的dispatchTouchEvent()源码分析,该方法比较复杂，篇幅有限，就截取几个重要的逻辑片段进行介绍，来解析整个分发流程。

```java
// 发生ACTION_DOWN事件或者已经发生过ACTION_DOWN,并且将mFirstTouchTarget赋值，才进入此区域，主要功能是拦截器
        final boolean intercepted;
        if (actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null) {
            //disallowIntercept：是否禁用事件拦截的功能(默认是false),即不禁用
            //可以在子View通过调用requestDisallowInterceptTouchEvent方法对这个值进行修改，不让该View拦截事件
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            //默认情况下会进入该方法
            if (!disallowIntercept) {
                //调用拦截方法
                intercepted = onInterceptTouchEvent(ev); 
                ev.setAction(action);
            } else {
                intercepted = false;
            }
        } else {
            // 当没有触摸targets，且不是down事件时，开始持续拦截触摸。
            intercepted = true;
        }
```

这一段的内容主要是为判断是否拦截。如果当前事件的MotionEvent.ACTION_DOWN，则进入判断，调用ViewGroup onInterceptTouchEvent()方法的值，判断是否拦截。如果mFirstTouchTarget != null，即已经发生过MotionEvent.ACTION_DOWN，并且该事件已经有ViewGroup的子View进行处理了，那么也进入判断，调用ViewGroup onInterceptTouchEvent()方法的值，判断是否拦截。如果不是以上两种情况，即已经是MOVE或UP事件了，并且之前的事件没有对象进行处理，则设置成true，开始拦截接下来的所有事件。**这也就解释了如果子View的onTouchEvent()方法返回false，那么接下来的一些列事件都不会交给他处理。如果VieGroup的onInterceptTouchEvent()第一次执行为true，则mFirstTouchTarget = null，则也会使得接下来不会调用onInterceptTouchEvent()，直接将拦截设置为true。**

当ViewGroup不拦截事件的时候，事件会向下分发交由它的子View或ViewGroup进行处理。

```java
  /* 从最底层的父视图开始遍历，
   ** 找寻newTouchTarget，即上面的mFirstTouchTarget
   ** 如果已经存在找寻newTouchTarget，说明正在接收触摸事件，则跳出循环。
    */
for (int i = childrenCount - 1; i >= 0; i--) {
  final int childIndex = customOrder
    ? getChildDrawingOrder(childrenCount, i) : i;
  final View child = (preorderedList == null)
    ? children[childIndex] : preorderedList.get(childIndex);

  // 如果当前视图无法获取用户焦点，则跳过本次循环
  if (childWithAccessibilityFocus != null) {
     if (childWithAccessibilityFocus != child) {
        continue;
     }
     childWithAccessibilityFocus = null;
     i = childrenCount - 1;
  }
  //如果view不可见，或者触摸的坐标点不在view的范围内，则跳过本次循环
  if (!canViewReceivePointerEvents(child) 
      || !isTransformedTouchPointInView(x, y, child, null)) {
    ev.setTargetAccessibilityFocus(false);
    continue;
    }

   newTouchTarget = getTouchTarget(child);
   // 已经开始接收触摸事件,并退出整个循环。
   if (newTouchTarget != null) {
       newTouchTarget.pointerIdBits |= idBitsToAssign;
       break;
    }

    //重置取消或抬起标志位
    //如果触摸位置在child的区域内，则把事件分发给子View或ViewGroup
    if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
        // 获取TouchDown的时间点
        mLastTouchDownTime = ev.getDownTime();
        // 获取TouchDown的Index
        if (preorderedList != null) {
           for (int j = 0; j < childrenCount; j++) {
               if (children[childIndex] == mChildren[j]) {
                    mLastTouchDownIndex = j;
                    break;
                }
           }
         } else {
                 mLastTouchDownIndex = childIndex;
                }

      //获取TouchDown的x,y坐标
      mLastTouchDownX = ev.getX();
      mLastTouchDownY = ev.getY();
      //添加TouchTarget,则mFirstTouchTarget != null。
      newTouchTarget = addTouchTarget(child, idBitsToAssign);
      //表示以及分发给NewTouchTarget
      alreadyDispatchedToNewTouchTarget = true;
      break;
}
```

`dispatchTransformedTouchEvent()`方法实际就是调用子元素的`dispatchTouchEvent()`方法。
其中`dispatchTransformedTouchEvent()`方法的重要逻辑如下：

```java
 if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
```

由于其中传递的child不为空，所以就会调用子元素的dispatchTouchEvent()。
如果子元素的dispatchTouchEvent()方法返回true，那么mFirstTouchTarget就会被赋值，同时跳出for循环。

```Java
//添加TouchTarget,则mFirstTouchTarget != null。
newTouchTarget = addTouchTarget(child, idBitsToAssign);
 //表示以及分发给NewTouchTarget
 alreadyDispatchedToNewTouchTarget = true;
```

其中在`addTouchTarget(child, idBitsToAssign);`内部完成mFirstTouchTarget被赋值。
如果mFirstTouchTarget为空，将会让ViewGroup默认拦截所有操作。
如果遍历所有子View或ViewGroup，都没有消费事件。ViewGroup会自己处理事件。

**结论**

- Android事件分发是先传递到ViewGroup，再由ViewGroup传递到View

- 在ViewGroup中通过onInterceptTouchEvent()对事件传递进行拦截

  > 1. onInterceptTouchEvent方法返回true代表拦截事件，即不允许事件继续向子View传递；
  > 2. 返回false代表不拦截事件，即允许事件继续向子View传递；（默认返回false）
  > 3. 子View中如果将传递的事件消费掉，ViewGroup中将无法接收到任何事件。

#### 4.3 View事件的分发机制

View中dispatchTouchEvent()的源码分析

```java
public boolean dispatchTouchEvent(MotionEvent event) {  
    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
            mOnTouchListener.onTouch(this, event)) {  
        return true;  
    }  
    return onTouchEvent(event);  
}
```

从上面可以看出：

- 只有以下三个条件都为真，dispatchTouchEvent()才返回true；否则执行onTouchEvent(event)方法

  ```java
  第一个条件：mOnTouchListener != null；
  第二个条件：(mViewFlags & ENABLED_MASK) == ENABLED；
  第三个条件：mOnTouchListener.onTouch(this, event)；
  ```

- 下面，我们来看看下这三个判断条件：

**第一个条件：mOnTouchListener!= null**

```java
//mOnTouchListener是在View类下setOnTouchListener方法里赋值的
public void setOnTouchListener(OnTouchListener l) { 

//即只要我们给控件注册了Touch事件，mOnTouchListener就一定被赋值（不为空）
    mOnTouchListener = l;  
}
```

**第二个条件：(mViewFlags & ENABLED_MASK) == ENABLED**

- 该条件是判断当前点击的控件是否enable
- 由于很多View默认是enable的，因此该条件恒定为true

**第三个条件：mOnTouchListener.onTouch(this, event)**

- 回调控件注册Touch事件时的onTouch方法

  ```java
  //手动调用设置
  button.setOnTouchListener(new OnTouchListener() {  
  
    @Override  
    public boolean onTouch(View v, MotionEvent event) {  
  
        return false;  
    }  
  });
  ```

- 如果在onTouch方法返回true，就会让上述三个条件全部成立，从而整个方法直接返回true。

- 如果在onTouch方法里返回false，就会去执行onTouchEvent(event)方法。

接下来，我们继续看：**onTouchEvent(event)**的源码分析

```java
public boolean onTouchEvent(MotionEvent event) {  
    final int viewFlags = mViewFlags;  
    if ((viewFlags & ENABLED_MASK) == DISABLED) {  
        // A disabled view that is clickable still consumes the touch  
        // events, it just doesn't respond to them.  
        return (((viewFlags & CLICKABLE) == CLICKABLE ||  
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));  
    }  
    if (mTouchDelegate != null) {  
        if (mTouchDelegate.onTouchEvent(event)) {  
            return true;  
        }  
    }  
     //如果该控件是可以点击的就会进入到下两行的switch判断中去；

    if (((viewFlags & CLICKABLE) == CLICKABLE ||  
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {  
    //如果当前的事件是抬起手指，则会进入到MotionEvent.ACTION_UP这个case当中。

        switch (event.getAction()) {  
            case MotionEvent.ACTION_UP:  
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  
               // 在经过种种判断之后，会执行到关注点1的performClick()方法。
               //请往下看关注点1
                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {  
                    // take focus if we don't have it already and we should in  
                    // touch mode.  
                    boolean focusTaken = false;  
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {  
                        focusTaken = requestFocus();  
                    }  
                    if (!mHasPerformedLongPress) {  
                        // This is a tap, so remove the longpress check  
                        removeLongPressCallback();  
                        // Only perform take click actions if we were in the pressed state  
                        if (!focusTaken) {  
                            // Use a Runnable and post this rather than calling  
                            // performClick directly. This lets other visual state  
                            // of the view update before click actions start.  
                            if (mPerformClick == null) {  
                                mPerformClick = new PerformClick();  
                            }  
                            if (!post(mPerformClick)) {  
            //关注点1
            //请往下看performClick()的源码分析
                                performClick();  
                            }  
                        }  
                    }  
                    if (mUnsetPressedState == null) {  
                        mUnsetPressedState = new UnsetPressedState();  
                    }  
                    if (prepressed) {  
                        mPrivateFlags |= PRESSED;  
                        refreshDrawableState();  
                        postDelayed(mUnsetPressedState,  
                                ViewConfiguration.getPressedStateDuration());  
                    } else if (!post(mUnsetPressedState)) {  
                        // If the post failed, unpress right now  
                        mUnsetPressedState.run();  
                    }  
                    removeTapCallback();  
                }  
                break;  
            case MotionEvent.ACTION_DOWN:  
                if (mPendingCheckForTap == null) {  
                    mPendingCheckForTap = new CheckForTap();  
                }  
                mPrivateFlags |= PREPRESSED;  
                mHasPerformedLongPress = false;  
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                break;  
            case MotionEvent.ACTION_CANCEL:  
                mPrivateFlags &= ~PRESSED;  
                refreshDrawableState();  
                removeTapCallback();  
                break;  
            case MotionEvent.ACTION_MOVE:  
                final int x = (int) event.getX();  
                final int y = (int) event.getY();  
                // Be lenient about moving outside of buttons  
                int slop = mTouchSlop;  
                if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
                        (y < 0 - slop) || (y >= getHeight() + slop)) {  
                    // Outside button  
                    removeTapCallback();  
                    if ((mPrivateFlags & PRESSED) != 0) {  
                        // Remove any future long press/tap checks  
                        removeLongPressCallback();  
                        // Need to switch from pressed to not pressed  
                        mPrivateFlags &= ~PRESSED;  
                        refreshDrawableState();  
                    }  
                }  
                break;  
        }  
//如果该控件是可以点击的，就一定会返回true
        return true;  
    }  
//如果该控件是不可以点击的，就一定会返回false
    return false;  
}
```

**关注点1：**
performClick()的源码分析

```java
public boolean performClick() {  
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  

    if (mOnClickListener != null) {  
        playSoundEffect(SoundEffectConstants.CLICK);  
        mOnClickListener.onClick(this);  
        return true;  
    }  
    return false;  
}
```

- 只要mOnClickListener不为null，就会去调用onClick方法；
- 那么，mOnClickListener又是在哪里赋值的呢？请继续看：

```java
public void setOnClickListener(OnClickListener l) {  
    if (!isClickable()) {  
        setClickable(true);  
    }  
    mOnClickListener = l;  
}
```

- 当我们通过调用setOnClickListener方法来给控件注册一个点击事件时，就会给mOnClickListener赋值（不为空），即会回调onClick（）。

**结论**

1. onTouch（）的执行高于onClick（）

2. 每当控件被点击时：

   - 如果在回调onTouch()里返回false，就会让dispatchTouchEvent方法返回false，那么就会执行onTouchEvent()；如果回调了setOnClickListener()来给控件注册点击事件的话，最后会在performClick()方法里回调onClick()。

     > onTouch()返回false（该事件没被onTouch()消费掉） = 执行onTouchEvent() = 执行OnClick()

   - 如果在回调onTouch()里返回true，就会让dispatchTouchEvent方法返回true，那么将不会执行onTouchEvent()，即onClick()也不会执行；

     > onTouch()返回true（该事件被onTouch()消费掉） = dispatchTouchEvent()返回true（不会再继续向下传递） = 不会执行onTouchEvent() = 不会执行OnClick()

**下面我将用Demo验证上述的结论**

**Demo论证**

**Demo1：在回调onTouch()里返回true**

![](http://upload-images.jianshu.io/upload_images/944365-6849145f3ce37ec5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
//设置OnTouchListener()
   button.setOnTouchListener(new View.OnTouchListener() {

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("执行了onTouch(), 动作是:" + event.getAction());

                return true;
            }
        });

//设置OnClickListener
    button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                System.out.println("执行了onClick()");
            }
        });
```

点击Button，测试结果如下：

![](http://upload-images.jianshu.io/upload_images/944365-c3e0ae42abb533a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Demo2：在回调onTouch()里返回false**

```java
//设置OnTouchListener()
   button.setOnTouchListener(new View.OnTouchListener() {

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                System.out.println("执行了onTouch(), 动作是:" + event.getAction());

                return false;
            }
        });

//设置OnClickListener
    button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                System.out.println("执行了onClick()");
            }
        });
```

点击Button，测试结果如下：

![](http://upload-images.jianshu.io/upload_images/944365-dc8fb10e301b05b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**总结：onTouch()返回true就认为该事件被onTouch()消费掉，因而不会再继续向下传递，即不会执行OnClick()。**

### 5. 思考点

#### 5.1 onTouch()和onTouchEvent()的区别

- 这两个方法都是在View的dispatchTouchEvent中调用，但onTouch优先于onTouchEvent执行。

- 如果在onTouch方法中返回true将事件消费掉，onTouchEvent()将不会再执行。

- 特别注意：请看下面代码

  ```java
  //&&为短路与，即如果前面条件为false，将不再往下执行
  //所以，onTouch能够得到执行需要两个前提条件：
  //1. mOnTouchListener的值不能为空
  //2. 当前点击的控件必须是enable的。
  mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
            mOnTouchListener.onTouch(this, event)
  ```

- 因此如果你有一个控件是非enable的，那么给它注册onTouch事件将永远得不到执行。对于这一类控件，如果我们想要监听它的touch事件，就必须通过在该控件中重写onTouchEvent方法来实现。

#### 5.2 Touch事件的后续事件（MOVE、UP）层级传递

- 如果给控件注册了Touch事件，每次点击都会触发一系列action事件（ACTION_DOWN，ACTION_MOVE，ACTION_UP等）

- 当dispatchTouchEvent在进行事件分发的时候，只有前一个事件（如ACTION_DOWN）返回true，才会收到后一个事件（ACTION_MOVE和ACTION_UP）

  > 即如果在执行ACTION_DOWN时返回false，后面一系列的ACTION_MOVE和ACTION_UP事件都不会执行

从上面对事件分发机制分析知：

- dispatchTouchEvent()和 onTouchEvent()消费事件、终结事件传递（返回true） 

- 而onInterceptTouchEvent 并不能消费事件，它相当于是一个分叉口起到分流导流的作用，对后续的ACTION_MOVE和ACTION_UP事件接收起到非常大的作用

  > 请记住：接收了ACTION_DOWN事件的函数不一定能收到后续事件（ACTION_MOVE、ACTION_UP）

**这里给出ACTION_MOVE和ACTION_UP事件的传递结论**：

- 如果在某个对象（Activity、ViewGroup、View）的dispatchTouchEvent()消费事件（返回true），那么收到ACTION_DOWN的函数也能收到ACTION_MOVE和ACTION_UP

  > 黑线：ACTION_DOWN事件传递方向
  > 红线：ACTION_MOVE和ACTION_UP事件传递方向

![](http://upload-images.jianshu.io/upload_images/944365-93d0b1496e9e6ca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 如果在某个对象（Activity、ViewGroup、View）的onTouchEvent()消费事件（返回true），那么ACTION_MOVE和ACTION_UP的事件从上往下传到这个View后就不再往下传递了，而直接传给自己的onTouchEvent()并结束本次事件传递过程。

  > 黑线：ACTION_DOWN事件传递方向
  > 红线：ACTION_MOVE和ACTION_UP事件传递方向

![img](http://upload-images.jianshu.io/upload_images/944365-9d639a0b9ebf7b4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





## 八、AsyncTask详解



## 九、HandlerThread详解



## 十、IntentService详解



## 十一、LruCache原理解析



## 十二、Window、Activity、DecorView以及ViewRoot之间的关系



## 十三、View测量、布局及绘制原理



## 十四、Android虚拟机及编译过程



## 十五、Android进程间通信方式



## 十六、Android Bitmap压缩策略



## 十七、Android动画总结



## 十八、Android进程优先级



## 十九、Android Context详解