

# Android基础

| 时间       | 版本  | 说明             |
| ---------- | ----- | ---------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入   |
| 2021.01.12 | 0.0.2 | 初版，大部分完成 |

## 一、Activity

#### 0. 说下Activity生命周期 ？

在正常情况下，Activity的常用生命周期就只有如下7个：

- **onCreate()**：表示Activity**正在被创建**，常用来**初始化工作**，比如调用setContentView加载界面布局资源，初始化Activity所需数据等；
- **onRestart()**：表示Activity**正在重新启动**，一般情况下，当前Acitivty从不可见重新变为可见时，OnRestart就会被调用；
- **onStart()**：表示Activity**正在被启动**，此时Activity**可见但不在前台**，还处于后台，无法与用户交互；
- **onResume()**：表示Activity**获得焦点**，此时Activity**可见且在前台**并开始活动，这是与onStart的区别所在；
- **onPause()**：表示Activity**正在停止**，此时可做一些**存储数据、停止动画**等工作，但是不能太耗时，因为这会影响到新Activity的显示，onPause必须先执行完，新Activity的onResume才会执行；
- **onStop()**：表示Activity**即将停止**，可以做一些稍微重量级的回收工作，比如注销广播接收器、关闭网络连接等，同样不能太耗时；
- **onDestroy()**：表示Activity**即将被销毁**，这是Activity生命周期中的最后一个回调，常做**回收工作、资源释放**；

延伸：从**整个生命周期**来看，onCreate和onDestroy是配对的，分别标识着Activity的创建和销毁，并且只可能有**一次调用**； 从Activity**是否可见**来说，onStart和onStop是配对的，这两个方法可能被**调用多次**； 从Activity**是否在前台**来说，onResume和onPause是配对的，这两个方法可能被**调用多次**；

#### 1. Activity A 跳转Activity B，Activity B再按back键回退，两个过程各自的生命周期

**(1) ActivityA跳转ActivityB的过程中,各自生命周期的执行顺序?**

执行顺序如下： A.onPause －> B.onCreate －> B.onStart－> B.onResume－> A.onStop

**(2) ActivityB按back键呢?**

按下back键后： B.onPause－>A.onRestart－>A.onStart－>A.onResume－>B.onStop－>B.onDestory

**(3) ActivityB是个窗口Activity的情况下，(1)、(2)的结论呢？**

ActivityA跳转到ActivityB时，ActivityA失去焦点部分可见，故不会调用onStop，此时生命周期顺序： A.onPause －> B.onCreate －> B.onStart－> B.onResume

按下Back键后：B.onPause－>A.onResume－>B.onStop－>B.onDestory

**(4) 切换横竖屏时，onCreate会调用吗？几次？**

程序在运行时，一些设备的配置可能会改变，如：横竖屏的切换、键盘的可用性或语言的切换等，此时Activity会重新启动。

其中的过程是：在销毁之前会先调用onSaveInstancestate()去保存应用中的一些数据，然后调用 onDestory()，最后才会去调用onCreate()或者onRestoreInstanceState方法重新启动Activiy。

如果自己没有配置android:ConfigChanges，在切换屏幕时候会重新调用各个生命周期，切横屏时会执行一次onCreate，切竖屏时在2.3前会执行两次onCreate，在2.3版本及以后都执行一次。

如果设置` <android:configChanges="orientation|keyboardHidden|screenSize">`，此时Activity的生命周期不会重走一遍，Activity不会重建，只会回调onConfigurationChanged方法。

#### 2. Activity的四大启动模式，以及应用场景？

`Activity`的四大启动模式：

- `standard`：标准模式，每次都会在活动栈中生成一个新的`Activity`实例。通常我们使用的活动都是标准模式。
- `singleTop`：栈顶复用，如果`Activity`实例已经存在栈顶，那么就不会在活动栈中创建新的实例，并回调`onNewIntent`方法。比较常见的场景就是给通知跳转的`Activity`设置，因为你肯定不想前台`Activity`已经是该`Activity`的情况下，点击通知，又给你再创建一个同样的`Activity`。
- `singleTask`：栈内复用，如果`Activity`实例在当前栈中已经存在，就会将当前`Activity`实例上面的其他`Activity`实例都移除栈，并回调`onNewIntent`方法。常见于跳转到主界面。
- `singleInstance`：单实例模式，创建一个新的任务栈，这个活动实例独自处在这个活动栈中，同样被重复调用的时候会调用并回调onNewIntent方法。

#### 3. Activity中onStart和onResume的区别？onPause和onStop的区别？

首先，Activity有三类：

**前台Activity**：活跃的Activity，正在和用户交互的Activity。

**可见非前台Activity**：常见于栈顶的Activity背景透明，处在其下面的Activity就是可见但是不可和用户交互。

**后台Activity**：已经被暂停的Activity，比如已经执行了onStop方法。

所以，onStart和onStop是从activity是否可见的角度来回调的。onResume和onPause是从activity是否定位于前台这个角度来回调的。

#### 4. 多进程怎么实现？如果启动一个多进程APP，会有几个进程运行？

`android:process`属性。给android的组件设置`android:process`属性来使其运行在指定的进程中。

- AndroidMantifest.xml中的activity、service、receiver和provider均支持`android:process`属性
- 设置该属性可以使每个组件均在各自的进程中运行，或者使一些组件共享一个进程
- AndroidMantifest.xml中的application元素也支持`android:process`属性，可以修改应用程序的默认进程名（默认值为包名）

如果`android:process`的值以冒号开头的话，那么该进程就是**私有进程**；以小写字母开头，那么就是**公有进程**，`android:process`值一定要有个点号。其他应用通过设置相同的ShareUID可以和它跑在同一个进程。

Android中，默认一个APK包就对应一个进程。Android平台对每个进程有内存限制，如果一个app有多个进程，那么总的内存就是所有进程的内存的总和，使用多进程，可以提高我们APP占用的最高内存。进程个数和`android:process`指定的进程名相关。

#### 6. Activity与AppCompactActivity区别？

AppCompactActivity最终继承了Activity，但做了很多对低版本的兼容措施。

- 主界面带有标题栏
- AppCompactActivity兼容低版本

去掉AppcompaActivity的标题栏方法：

```java
//方式一：这句代码必须写在setContentView()方法的后面
getSupportActionBar().hide();

//方式二：这句代码必须写在setContentView()方法的前面
supportRequestWindowFeature(Window.FEATURE_NO_TITLE);

//清单文件（manifest.xml）里面实现
android:theme="@style/Theme.AppCompat.NoActionBar"
```

Activity去标题栏：

```java
//这句代码必须写在setContentView()前面
requestWindowFeature(Window.FEATURE_NO_TITLE);

//清单文件（manifest.xml）里面实现
<android:theme="@android:style/Theme.NoTitleBar"> 
```

#### 7. Activity 按 back 键退出，与强杀进程退出有啥区别？

（1）应用被强杀

- 整个App进程都被杀掉了，所有变量全都被清空，包括Application实例，更别提静态变量；
- 虽然变量被清空了，但 Activity 栈没有被清空，也就是说 A -> B -> C 这个栈还保存了，只是ABC 这几个 Activity 实例没有了。所以回到 App 时，显示的还是 C 页面。另外当 Activity 被强杀时，系统会调用 onSaveInstance 去让你保存一些变量；
- 当应用回到前台时，如果C页面中有静态变量或有些Application的全局变量，就NullPointer了；
- C页面不会正常走完生命周期onStop & onDestory

（2）按 Back 键回退，有序的退栈操作。

- 应用进程不会被杀掉；Activity 栈由 A -> B -> C 变成 A -> B；
- C页面会正常走完生命周期onStop & onDestory

#### 8. Activity跳转声明周期？

Activity依次A→B→C→B，其中B启动模式为singleTask，AC都为standard，生命周期分别怎么调用？如果B启动模式为singleInstance又会怎么调用？B启动模式为singleInstance不变，A→B→C的时候点击两次返回，生命周期如何调用?

**旧的Activity先onPause，新的再启动。**

1)A→B→C→B,B启动模式为`singleTask`

- 启动A的过程，生命周期调用是`(A)onCreate→(A)onStart→(A)onResume`
- 再启动B的过程，生命周期调用是 `(A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop`
- B→C的过程同上，`(B)onPause→(C)onCreate→(C)onStart→(C)onResume→(B)onStop`
- C→B的过程，由于B启动模式为singleTask，所以B会调用onNewIntent，并且将B之上的实例移除，也就是C会被移出栈。所以生命周期调用是 `(C)onPause→(B)onNewIntent→(B)onRestart→(B)onStart→(B)onResume→(C)onStop→(C)onDestory`

2)A→B→C→B,B启动模式为`singleInstance`

- 如果B为singleInstance，那么C→B的过程，C就不会被移除，因为B和C不在一个任务栈里面。所以生命周期调用是 `(C)onPause→(B)onNewIntent→(B)onRestart→(B)onStart→(B)onResume→(C)onStop`

3)A→B→C,B启动模式为`singleInstance`,点击两次返回键

- 如果B为singleInstance，A→B→C的过程，生命周期还是同前面一样正常调用。但是点击返回的时候，由于AC同任务栈，所以C点击返回，会回到A，再点击返回才回到B。所以生命周期是：(C)onPause→(A)onRestart→(A)onStart→(A)onResume→(C)onStop→(C)onDestory。
- 再次点击返回，就会回到B，所以生命周期是：(A)onPause→(B)onRestart→(B)onStart→(B)onResume→(A)onStop→(A)onDestory。

#### 9. 屏幕旋转时Activity的生命周期，如何防止Activity重建。

**onSaveInstanceState在onstop前，个onPause没有既定的关系**

- 切换屏幕的生命周期是：onConfigurationChanged->onPause->onSaveInstanceState->onStop->onDestroy->onCreate->onStart->onRestoreInstanceState->onResume
- 如果需要防止旋转时候（切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法），`Activity`重新创建的话需要做如下配置：
   在`targetSdkVersion`的值小于或等于12时，配置 android:configChanges="orientation"，
   在`targetSdkVersion`的值大于12时，配置 android:configChanges="orientation|screenSize"

#### 10. onSaveInstanceState() 与 onRestoreIntanceState()

当应用遇到意外情况（如：内存不足、用户直接按Home键）由系统销毁一个Activity时，onSaveInstanceState() 会被调用，此方法调用在onStop之前，与onPause没有既定的时序关系。

但是当用户主动去销毁一个Activity时，例如在应用中按返回键，onSaveInstanceState()就不会被调用。因为在这种情况下，用户的行为决定了不需要保存Activity的状态。通常onSaveInstanceState()只适合用于保存一些临时性的状态，而onPause()适合用于数据的持久化保存。

在activity被杀掉之前调用保存每个实例的状态，以保证该状态可以在onCreate(Bundle)或者onRestoreInstanceState(Bundle) 中恢复。

![异常情况下Activity的重建过程](https://user-gold-cdn.xitu.io/2019/3/8/1695c1aaea26f833?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 11.android中进程的优先级？

1. 前台进程：即与用户正在交互的Activity或者Activity用到的Service等，如果系统内存不足时前台进程是最晚被杀死的

2. 可见进程：可以是处于暂停状态(onPause)的Activity或者绑定在其上的Service，即被用户可见，但由于失了焦点而不能与用户交互

3. 服务进程：其中运行着使用startService方法启动的Service，虽然不被用户可见，但是却是用户关心的，例如用户正在非音乐界面听的音乐或者正在非下载页面下载的文件等；当系统要空间运行，前两者进程才会被终止

4. 后台进程：其中运行着执行onStop方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这时的进程系统一旦没了有内存就首先被杀死

5. 空进程：不包含任何应用程序的进程，这样的进程系统是一般不会让他存在的

#### 12. activity的startActivity和context的startActivity区别？

(1)从Activity中启动新的Activity时可以直接mContext.startActivity(intent)就好

(2)如果从其他Context中启动Activity则必须给intent设置Flag:

```java
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK) ; 
mContext.startActivity(intent);
```

#### 13. 子线程是否可以 context.startActivity() ？例如ApplicationContext, 会不会有什么问题？

是可以的。

创建Dialog时不可以用Application的context，只能用Activity的context。

![](http://upload-images.jianshu.io/upload_images/1187237-fb32b0f992da4781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 14. 怎样在两个 Activity 之间传递一张图片?

把 Bitmap 放到 Intent 里会导致巨大的内存损耗，所以在传递图片时应该是传递 URI 地址，新界面根据 URI 生成新图片。同时还可以到图片缓存里使用 URI 查找已有图片，节约内存。

#### 15. 如何退出 Activity？如何安全退出已调用多个 Activity 的 Application？

- 通常情况用户退出一个 Activity 只需按返回键，写代码直接调用 finish()方法就行。
- 记录打开的 Activity： 每打开一个 Activity，就记录下来。在需要退出时，关闭每一个 Activity 即可。
- 发送特定广播： 在需要结束应用时，发送一个特定的广播，每个 Activity 收到广播后，关闭即可。
- 递归退出 在打开新的 Activity 时使用 startActivityForResult，然后自己加标志，在 onActivityResult 中处理，递归关闭。
- 其实 也可以通过 intent 的 flag 来实现 intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)激活一个新的 activity。 此时如果该任务栈中已经有该 Activity，那么系统会把这个 Activity 上面的所有 Activity 干掉。其实相当于给 Activity 配置的启动模式为 SingleTop。

#### 16. 隐式启动Activity，如何判断是否有Activity匹配

- 采用`PackageManage#resolveActivity`或者`Intent#resolveActivity`方法，匹配不到就会返回null
- 采用`PackageManage#queryIntentActivities`方法，返回匹配的Activity信息

#### 17. Android怎么加速启动Activity？

- `onCreate()` 中不执行耗时操作

> 把页面显示的 View 细分一下，放在 AsyncTask 里逐步显示，用 Handler 更好。这样用户的看到的就是有层次有步骤的一个个的 View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。

- 利用多线程的目的就是尽可能的减少 `onCreate()` 和 `onReume()` 的时间，使得用户能尽快看到页面，操作页面。
- 减少主线程阻塞时间。
- 提高 Adapter 和 AdapterView 的效率。
- 优化布局文件。

#### 18. 了解哪些Activity常用的标记位Flags？

- **FLAG_ACTIVITY_NEW_TASK** : 对应singleTask启动模式，其效果和在XML中指定该启动模式相同；
- **FLAG_ACTIVITY_SINGLE_TOP** : 对应singleTop启动模式，其效果和在XML中指定该启动模式相同；
- **FLAG_ACTIVITY_CLEAR_TOP** : 具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。这个标记位一般会和singleTask模式一起出现，在这种情况下，被启动Activity的实例如果已经存在，那么系统就会回调onNewIntent。如果被启动的Activity采用standard模式启动，那么它以及连同它之上的Activity都要出栈，系统会创建新的Activity实例并放入栈中；
- **FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS** : 具有这个标记的 Activity 不会出现在历史 Activity 列表中；

#### 19. 说下 Activity跟window，view之间的关系？

- Activity在创建时会调用 `attach()` 方法初始化一个`PhoneWindow`(继承于Window)，**每一个Activity都包含了唯一一个PhoneWindow**

- Activity通过`setContentView`实际上是调用的 `getWindow().setContentView()`将View设置到PhoneWindow上，而PhoneWindow内部是通过 WindowManager 的`addView`、`removeView`、`updateViewLayout`这三个方法来管理View，**WindowManager本质是接口，最终由WindowManagerImpl实现**

- `WindowManager`为每个`Window`创建`Surface`对象，然后应用就可以通过这个`Surface`来绘制任何它想要绘制的东西。而对于`WindowManager`来说，这只不过是一块矩形区域而已

- `Surface`其实就是一个持有像素点矩阵的对象，这个像素点矩阵是组成显示在屏幕的图像的一部分。我们看到显示的每个`Window`（包括对话框、全屏的Activity、状态栏等）都有他自己绘制的`Surface`。而最终的显示可能存在`Window`之间遮挡的问题，此时就是通过`Surface Flinger`对象渲染最终的显示，使他们以正确的`Z-order`显示出来。一般`Surface`拥有一个或多个缓存（一般2个），通过双缓存来刷新，这样就可以一边绘制一边加新缓存。

- View是Window里面用于交互的UI元素。Window只attach一个View Tree（组合模式），当Window需要重绘（如，当View调用`invalidate`）时，最终转为Window的Surface，Surface被锁住（locked）并返回Canvas对象，此时View拿到Canvas对象来绘制自己。当所有View绘制完成后，Surface解锁（unlock），并且`post`到绘制缓存用于绘制，通过`Surface Flinger`来组织各个Window，显示最终的整个屏幕

#### 20. 如何启动其他应用的Activity？

在保证有权限访问的情况下，通过隐式Intent进行目标Activity的IntentFilter匹配，原则是：

- 一个intent只有同时匹配某个Activity的intent-filter中的action、category、data才算完全匹配，才能启动该Activity；
- 一个Activity可以有多个 intent-filter，一个 intent只要成功匹配任意一组 intent-filter，就可以启动该Activity；



## 二、Service

#### 0. 谈一谈Service的生命周期？

- **onCreate()**：如果service没被创建过，调用`startService()`后会执行`onCreate()`回调；如果service已处于运行中，调用`startService()`不会执行`onCreate()`方法。也就是说，`onCreate()`只会在第一次创建service时候调用，多次执行`startService()`不会重复调用`onCreate()`，此方法适合完成一些初始化工作；

- **onStartComand()**：服务启动时调用，此方法适合完成一些数据加载工作，比如会在此处创建一个线程用于下载数据或播放音乐；

- **onBind()**：服务被绑定时调用；

- **onUnBind()**：服务被解绑时调用；

- **onDestroy()**：服务停止时调用；

#### 1. Activity怎么启动Service，Activity与Service交互，Service与Thread的区别?

2种方式：

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

bindService(mIntent,conn,BIND_AUTO_CREATE);

startService(mIntent)
```

**startService()**：通过这种方式调用startService，onCreate()只会被调用一次，多次调用startSercie会多次执行`onStartCommand()`方法。如果外部没有调用`stopService()`或`stopSelf()`方法，service会一直运行。

**bindService()**：如果该服务之前**还没创建**，系统回调顺序为`onCreate()→onBind()`。如果调用`bindService()`方法前服务**已经被绑定**，多次调用`bindService()`方法**不会多次创建服务及绑定**。如果调用者希望与正在绑定的服务解除绑定，可以调用`unbindService()`方法，回调顺序为`onUnbind()→onDestroy()`；

![img](https://user-gold-cdn.xitu.io/2019/3/8/1695c1aaec35fdba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**Activity与Service交互：**Intent，Binder，跨进程通信方式都可

**Service与Thread区别：**

Service：在后台用来操作时间跨度较长的工作应用组件；Service的生命周期方法在主线程中执行，如果想执行一个长时间的工作，需要开启一个分线程（Thread）；**远程服务**在应用退出，Service不会停止，再次启动应用时，还可以以正在运行的Service通信。

Thread：用来开启一个分线程的类，做一个长时间的工作；Thread类的`run()`在分线程中执行；应用退出，Thread也不会停止；再次启动应用，不能再控制之前的Thread对象。

#### 2. Service 一定没界面吗，Activity 一定有界面吗？

- Activity 不是一定有界面。比如一个跳转逻辑控制类（机票的支付中间类）、透明页
- Service 也不是一定没界面。Service 并不依赖于用户可视的 UI 界面，但这也不是绝对的，如前台 Service 就是与 Notification 界面结合使用的；Service 中也可以弹 Toast；
- Service中执行 LayoutInflate 是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。所以从理论上看也是可以有界面的

#### 3. 怎么在Service中创建Dialog对话框？

1.在我们取得Dialog对象后，需给它设置类型，以系统对话框的形式弹出，即：

```java
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT)
```

2.在Manifest中加上权限:

```xml
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINOW" />
```

#### 4. 为什么bindService可以跟Activity生命周期联动？

- bindService 方法执行时，LoadApk 会记录 ServiceConnection 信息。

- Activity 执行 finish 方法时，会通过 LoadApk 检查 Activity 是否存在未注销/解绑的 BroadcastReceiver 和 ServiceConnection，如果有，那么会通知 AMS 注销/解绑对应的 BroadcastReceiver 和 Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作。

```java
    //ContentImpl
    @Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {
        warnIfCallingFromSystemProcess();
        return bindServiceCommon(service, conn, flags, mMainThread.getHandler(),
                Process.myUserHandle());
    }

    private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags, Handler handler, UserHandle user) {
        // Keep this in sync with DevicePolicyManager.bindDeviceAdminServiceAsUser.
        IServiceConnection sd;
		...
        if (mPackageInfo != null) {
            sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), handler, flags);
        }
        ...
    }

    //LoadedApk
    private final ArrayMap<Context, ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>> mServices  = new ArrayMap<>();

    public final IServiceConnection getServiceDispatcher(ServiceConnection c,
            Context context, Handler handler, int flags) {
        synchronized (mServices) {
            LoadedApk.ServiceDispatcher sd = null;
            ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher> map = mServices.get(context);
            if (map != null) {
                if (DEBUG) Slog.d(TAG, "Returning existing dispatcher " + sd + " for conn " + c);
                sd = map.get(c);
            }
            if (sd == null) {
                sd = new ServiceDispatcher(c, context, handler, flags);
                if (DEBUG) Slog.d(TAG, "Creating new dispatcher " + sd + " for conn " + c);
                if (map == null) {
                    map = new ArrayMap<>();
                    mServices.put(context, map);
                }
                map.put(c, sd);
            } else {
                sd.validate(context, handler);
            }
            return sd.getIServiceConnection();
        }
    }

//LoadedApk
    public void removeContextRegistrations(Context context,
            String who, String what) {
        final boolean reportRegistrationLeaks = StrictMode.vmRegistrationLeaksEnabled();
        synchronized (mReceivers) {
            ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher> rmap =
                    mReceivers.remove(context);
            if (rmap != null) {
                for (int i = 0; i < rmap.size(); i++) {
                    LoadedApk.ReceiverDispatcher rd = rmap.valueAt(i);
                    IntentReceiverLeaked leak = new IntentReceiverLeaked(
                            what + " " + who + " has leaked IntentReceiver "
                            + rd.getIntentReceiver() + " that was " +
                            "originally registered here. Are you missing a " +
                            "call to unregisterReceiver()?");
                    leak.setStackTrace(rd.getLocation().getStackTrace());
                    Slog.e(ActivityThread.TAG, leak.getMessage(), leak);
                    if (reportRegistrationLeaks) {
                        StrictMode.onIntentReceiverLeaked(leak);
                    }
                    try {
                        ActivityManager.getService().unregisterReceiver(
                                rd.getIIntentReceiver());
                    } catch (RemoteException e) {
                        throw e.rethrowFromSystemServer();
                    }
                }
            }
            mUnregisteredReceivers.remove(context);
        }

        synchronized (mServices) {
            //Slog.i(TAG, "Receiver registrations: " + mReceivers);
            ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher> smap =
                    mServices.remove(context);
            if (smap != null) {
                for (int i = 0; i < smap.size(); i++) {
                    LoadedApk.ServiceDispatcher sd = smap.valueAt(i);
                    ServiceConnectionLeaked leak = new ServiceConnectionLeaked(
                            what + " " + who + " has leaked ServiceConnection "
                            + sd.getServiceConnection() + " that was originally bound here");
                    leak.setStackTrace(sd.getLocation().getStackTrace());
                    Slog.e(ActivityThread.TAG, leak.getMessage(), leak);
                    if (reportRegistrationLeaks) {
                        StrictMode.onServiceConnectionLeaked(leak);
                    }
                    try {
                        ActivityManager.getService().unbindService(
                                sd.getIServiceConnection());
                    } catch (RemoteException e) {
                        throw e.rethrowFromSystemServer();
                    }
                    sd.doForget();
                }
            }
            mUnboundServices.remove(context);
            //Slog.i(TAG, "Service registrations: " + mServices);
        }
    }
```

#### 5. 如何保证Service不被杀死？

Android 进程不死从3个层面入手：

**1) 提供进程优先级，降低进程被杀死的概率**

方法一：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。

方法二：启动前台service。

> 进程优先级由高到低：前台进程 一 可视进程 一 服务进程 一 后台进程 一 空进程 可以使用startForeground将service放到前台状态，这样低内存时，被杀死的概率会低一些；

方法三：提升service优先级。

>  在AndroidManifest.xml文件中对于intent-filter可以通过`android:priority="1000"`这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

方法四：将APK安装到/system/app，变身为系统级应用

**2) 在进程被杀死后，进行拉活**

方法一：注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等

方法二：双进程相互唤起。

方法三：依靠系统唤起。

方法四：onDestroy方法里重启service：service + broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

**3) 依靠第三方**

根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。

> **注意**：以上机制都不能百分百保证Service不被杀死，除非做到系统白名单，与系统同生共死

#### 6. 什么是 IntentService？有何优点？

IntentService 是 Service 的子类。本质是采用Handler & HandlerThread。

**IntentService与Service的区别**

- 从属性 & 作用上来说
  Service：依赖于应用程序的主线程（不是独立的进程 or 线程）

  > 不建议在Service中编写耗时的逻辑和操作，否则会引起ANR；

  IntentService：创建一个工作线程来处理多线程任务 　　

- Service需要主动调用stopSelft()来结束服务，而IntentService不需要（在所有intent被处理完后，系统会自动关闭服务）

**IntentService与其他线程的区别**

- IntentService内部采用了HandlerThread实现，作用类似于后台线程；

- 与后台线程相比，IntentService是一种后台服务，优势是：优先级高（不容易被系统杀死），从而保证任务的执行。

  > 对于后台线程，若进程中没有活动的四大组件，则该线程的优先级非常低，容易被系统杀死，无法保证任务的执行

#### 7. Service 的 onStartCommand 方法有几种返回值？各代表什么意思？

有四种返回值，不同值代表的意思如下： 

- `START_STICKY`：如果 service 进程被 kill 掉，保留 service 的状态为开始状态，但不保留递送的 intent 对象。随后系统会尝试重新创建 service ， 由于服务状态为开始状态，所以创建服务后一定会调用 `onStartCommand(Intent,int,int)`方法。如果在此期间没有任何启动命令被传递到 service，那么参数 Intent 将为 null。 

- `START_NOT_STICKY`：“非粘性的”。使用这个返回值时，如果在执行完 `onStartCommand` 后，服务被异常 kill 掉，系统不会自动重启该服务。 

- `START_REDELIVER_INTENT`：重传 Intent。使用这个返回值时，如果在执行完 `onStartCommand` 后，服务被异 kill 掉，系统会自动重启该服务，并将 Intent 的值传入。 

- `START_STICKY_COMPATIBILITY`：`START_STICKY` 的兼容版本，但不保证服务被 kill 后一定能重启。

#### 8. Service 的 onRebind（Intent）方法在什么情况下会执行？

如果在 `onUnbind()` 方法返回 true 的情况下，且没有被销毁，再次绑定服务会被调用，否则不执行。

#### 9. Activity 调用 Service 中的方法都有哪些方式？

Activity 调用 Service 中的方法主要是通过绑定服务的模式实现的，绑定服务又分为三种方式。如下所示：

1) Extending the Binder class 

通过 Binder 接口的形式实现，当 Activity 绑定 Service 成功的时候 Activity 会在 ServiceConnection 的类 的 `onServiceConnected()` 回调方法中获取到 Service 的 `onBind()` 方法 return 过来的 Binder 的子类。 

2) Using a Messenger

这是官方给出的另外一种沟通方式。

3) Using AIDL

aidl 比较适合当客户端和服务端不在同一个应用下的场景。

#### 10. Service 如何给 Activity 发送 Message？

可以通过Messenger。

#### 11. 子线程不能代替 service 吗？

a) 不能替代 

b) 服务作为四大组件之一，是运行在主线程的，可以直接显示吐司，修改 View 等。如果要运行耗时操作，服务需要自己开启子线程。 

c) 只作为后台来理解的话，相比于线程，服务具备完善的生命周期，更方便随时释放资源。 

d) 服务自己就有上下文(Context)对象，可以确定上下文是正常可用的。线程需要从外部获取上下文对象，在运行时无法保证该对象没有被系统销毁。

#### 12. 用过哪些系统Service ？

WindowManager：管理打开的窗口程序

ActivityManager：管理应用程序的系统状态

PowerManager：电源服务

AlarmManager：闹钟服务

NotificationManager：状态栏服务

KeyguardManager：键盘锁服务



## 三、BroadcaseReceiver

#### 1. 程序A能否接收到程序B的广播？

看情况，使用全局的BroadCastRecevier能进行跨进程通信，但是注意它只能被动接收广播。此外，LocalBroadCastRecevier只限于本进程的广播间通信。

#### 2. 广播传输的数据是否有限制，是多少，为什么要限制？

Intent在传递数据时是有大小限制的，大约限制在1MB之内，你用Intent传递数据，实际上走的是跨进程通信（IPC），跨进程通信需要把数据从内核copy到进程中，每一个进程有一个接收内核数据的缓冲区，默认是1M；如果一次传递的数据超过限制，就会出现异常。

不同厂商表现不一样有可能是厂商修改了此限制的大小，也可能同样的对象在不同的机器上大小不一样。

传递大数据，不应该用Intent；考虑使用ContentProvider或者直接匿名共享内存。简单情况下可以考虑分段传输。

#### 3. 广播注册一般有几种，各有什么优缺点？

**按注册方式**：

第一种是常驻型(静态注册)：当应用程序关闭后如果有信息广播来，程序也会被系统调用，自己运行。

第二种不常驻(动态注册)：广播会跟随程序的生命周期。

**动态注册**

优点： 在android的广播机制中，动态注册优先级高于静态注册优先级，因此在必要情况下，是需要动态注册广播接收者的。

缺点： 当用来注册的 Activity 关掉后，广播也就失效了。

**静态注册** 

优点： 无需担忧广播接收器是否被关闭，只要设备是开启状态，广播接收器就是打开着的。

**按类型**：

普通广播：开发者自身定义 intent的广播（最常用），所有的广播接收器几乎会在同一时刻接受到此广播信息，**接受的先后顺序随机**；

有序广播：发送出去的广播被广播接收者**按照先后顺序接收**，同一时刻只会有一个广播接收器能够收到这条广播消息，当这个广播接收器中的逻辑执行完毕后，广播才会继续传递，且优先级（priority）高的广播接收器会先收到广播消息。有序广播可以被接收器截断使得后面的接收器无法收到它；

本地广播：仅在自己的应用内发送接收广播，也就是只有自己的应用能收到，数据更加安全，效率更高，但只能采用**动态注册**的方式；

粘性广播：这种广播会**一直滞留**，当有匹配该广播的接收器被注册后，该接收器就会收到此条广播；

#### 4. BroadCastReceiver 的生命周期 ?

a. 广播接收者的生命周期非常短暂的，在接收到广播的时候创建，`onReceive()`方法结束之后销毁； 

b. 广播接收者中不要做一些耗时的工作，否则会弹出 `Application No Response` 错误对话框； 

c. 最好也不要在广播接收者中创建子线程做耗时的工作，因为广播接收者被销毁后进程就成为了空进程，很容易被系统杀掉； 

d. 耗时的较长的工作最好放在服务中完成；

#### 5. 如何让自己的广播只让指定的 app 接收

自己的应用（假设名称为应用 A）在发送广播的时候给自己发送的广播添加自定义权限

其他应用（假设名称诶应用 B）如果想接收该广播，那么就必须知道应用 A 广播使用的权限。然后在应用的清单文件中配置。

#### 6. 什么是最终广播接收者？ 

最终广播是我们自己应用发送有序广播时通过 `ContextWrapper.sendOrderedBroadcast()`方法指定的当前应用 下的广播，该广播可能会被执行两次，第一次是作为普通广播按照优先级接收广播，第二次是作为 final receiver 必须接收一次。

#### 7. 广播的优先级对无序广播生效吗？ 

生效的。广播的优先级推荐的范围是：`[-1000,+1000]`，但是如果设置的优先级值超过这个范围也是可以的。

#### 8. 动态注册的广播优先级谁高？ 

优先级大的高，优先级一致的话谁先注册谁优先级高。

#### 9. 如何判断当前接收到的是有序还是无序广播？

在 BroadcastReceiver 类中 `onReceive()`方法中，可以调用 `boolean b = isOrderedBroadcast();`该方法是 BroadcastReceiver 类中提供的方法，用于告诉我们当前的接收到的广播是否为有序广播。

#### 10. Android 引入广播机制的用意

a.从 MVC 的角度考虑(应用程序内)，android 的四大组件本质上就是为了实现移动或者说嵌入式设备上的 MVC 架构，它们之间有时候是一种相互依存的关系，有时候又是一种补充关系，引入广播机制可以方便几大组件的信息和数据交互。 

b.程序间互通消息(例如在自己的应用程序内监听系统来电)

c.效率上(参考 UDP 的广播协议在局域网的方便性) 

d.设计模式上(反转控制的一种应用，类似监听者模式)

#### 11. 网络状态改变是无序广播还是有序广播，安装了，没启动过，会接受这个广播么？

无序广播。不会。

android 在 3.0 之后，对广播增加了一个标记：`Intent.FLAG_EXCLUDE_STOPPED_PACKAGES`， 这个是为了加强了对“停止”状态 APP 的管理（比如 app 安装后未启动或者被用户强制停止）。广播加上这个 Flag 之后，处于“停止”状态的 APP 是无法收到广播的，系统发出的广播基本都有这个 Flag。因此该类广播我们在使用的时候主要是采用动态注册的方式。



## 四、ContentProvider

#### 1. ContentProvider使用方法?如何实现数据共享的？

进行跨进程通信，实现进程间的数据交互和共享。通过Context 中 `getContentResolver()` 获得实例，通过 Uri匹配进行数据的增删改查。ContentProvider 使用表的形式来组织数据，无论数据的来源是什么，ConentProvider 都会认为是一种表，然后把数据组织成表格。

#### 2. ContentProvider的权限管理?

读写分离，权限控制-精确到表级，URI控制。

对于ContentProvider暴露出来的数据，应该是存储在自己应用内存中的数据，对于一些存储在外部存储器上的数据，并不能限制访问权限，使用ContentProvider就没有意义了。对于ContentProvider而言，有很多权限控制，可以在`AndroidManifest.xml`文件中对`<provider>`节点的属性进行配置，一般使用如下一些属性设置：

- `android:grantUriPermssions`:临时许可标志。
- `android:permission`:Provider读写权限。
- `android:readPermission`:Provider的读权限。
- `android:writePermission`:Provider的写权限。
- `android:enabled`:标记允许系统启动Provider。
- `android:exported`:标记允许其他应用程序使用这个Provider。
- `android:multiProcess`:标记允许系统启动Provider相同的进程中调用客户端。

#### 3. 说说ContentProvider、ContentResolver、ContentObserver 之间的关系？

ContentProvider：管理数据，提供数据的增删改查操作，数据源可以是数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做进程间数据共享。

ContentResolver：ContentResolver可以为不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互。

ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界。

#### 4. 为什么要用 ContentProvider？它和 sql 的实现上有什么差别？ 

ContentProvider 屏蔽了数据存储的细节，内部实现对用户完全透明，用户只需要关心操作数据的 uri 就可以了， ContentProvider 可以实现不同 app 之间共享。 

Sql 也有增删改查的方法，但是 sql 只能查询本应用下的数据库。而 ContentProvider 还可以去增删改查本地文件.xml 文件的读取等。



## 五、Fragment

#### 1. Fragment hide show replace生命周期

1）生命周期：

- `onAttach()`：Fragment和Activity相关联时调用。可以通过该方法获取Activity引用，还可以通过getArguments()获取参数。
- `onCreate()`：Fragment被创建时调用。
- `onCreateView()`：创建Fragment的布局。
- `onActivityCreated()`：当Activity完成onCreate()时调用。
- `onStart()`：当Fragment可见时调用。
- `onResume()`：当Fragment可见且可交互时调用。
- `onPause()`：当Fragment不可交互但可见时调用。
- `onStop()`：当Fragment不可见时调用。
- `onDestroyView()`：当Fragment的UI从视图结构中移除时调用。
- `onDestroy()`：销毁Fragment时调用。
- `onDetach()`：当Fragment和Activity解除关联时调用。

每个调用方法对应的生命周期变化：

- `add()`: onAttach()->…->onResume()。
- `remove()`: onPause()->…->onDetach()。
- `replace()`: 相当于旧Fragment调用`remove()`，新Fragment调用`add()`。`remove()+add()`的生命周期加起来
- `show()`: 不调用任何生命周期方法，调用该方法的前提是要显示的 Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为true。
- `hide()`: 不调用任何生命周期方法，调用该方法的前提是要显示的Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为false。

#### 2. ViewPager切换Fragment遇到过什么问题吗? 什么最耗时？

- 滑动的时候，调用setCurrentItem方法，要注意第二个参数`smoothScroll`。传false，就是直接跳到fragment，传true，就是平滑过去。一般主页切换页面都是用false。

```java
    /**
     * Set the currently selected page.
     * @param item Item index to select
     * @param smoothScroll True to smoothly scroll to the new item, false to transition immediately
     */
    public void setCurrentItem(int item, boolean smoothScroll) {
        mPopulatePending = false;
        setCurrentItemInternal(item, smoothScroll, false);
    }
```

- 禁止预加载的话，调用`setOffscreenPageLimit(0)`是无效的，因为方法里面会判断是否小于1。需要重写`setUserVisibleHint`方法，判断fragment是否可见。

```java
   //ViewPager
	public void setOffscreenPageLimit(int limit) {
        if (limit < DEFAULT_OFFSCREEN_PAGES) {//1
            Log.w(TAG, "Requested offscreen page limit " + limit + " too small; defaulting to " + DEFAULT_OFFSCREEN_PAGES);
            limit = DEFAULT_OFFSCREEN_PAGES;
        }
        if (limit != mOffscreenPageLimit) {
            mOffscreenPageLimit = limit;
            populate();
        }
    }
```

- 不要使用`getActivity()`获取activity实例，容易造成空指针，因为如果fragment已经`onDetach()`了，那么就会报空指针。所以要在`onAttach`方法里面，就去获取activity的上下文。
- `FragmentStatePagerAdapter`对`limit`外的Fragment销毁，生命周期为onPause->onStop->onDestoryView->onDestory->onDetach, onAttach->onCreate->onCreateView->onStart->onResume。也就是说切换fragment的时候有可能会多次`onCreateView`，所以需要注意处理数据。
- 由于可能多次`onCreateView`，所以我们可以把view保存起来，如果为空再去初始化数据。见代码：

```java
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        if (null == mFragmentView) {
			mFragmentView = inflater.inflate(getContentViewLayoutID(), null);
			initViewsAndEvents();
        }
        return mFragmentView;
    }
```

#### 3. Activity 与 Fragment，Fragment 与 Fragment之间怎么交互通信。

**Activity 与 Fragment通信**

Activity有Fragment的实例，所以可以执行Fragment的方法，或者传入一个接口。同样，Fragment可以通过`getActivity()`获取Activity的实例，也是可以执行方法。

**Fragment 与 Fragment之间通信**

1）直接获取另一个Fragmetn的实例

```java
getActivity().getSupportFragmentManager().findFragmentByTag("mainFragment");
```

2）接口回调

一个Fragment里面去实现接口，另一个Fragment把接口实例传进去。

3）Eventbus框架。

4）广播

#### 4. Fragment状态保存

Fragment状态保存入口:

- Activity的状态保存, 在Activity的`onSaveInstanceState()`里, 调用了FragmentController的`saveAllState()`方法, 其中会对mActive中各个Fragment的实例状态和View状态分别进行保存
- FragmentManager还提供了public方法: `saveFragmentInstanceState()`, 可以对单个Fragment进行状态保存, 这是提供给我们用的。

```java
    @Override
    public Fragment.SavedState saveFragmentInstanceState(Fragment fragment) {
        if (fragment.mIndex < 0) {
            throwException(new IllegalStateException("Fragment " + fragment
                    + " is not currently in the FragmentManager"));
        }
        if (fragment.mState > Fragment.INITIALIZING) {
            Bundle result = saveFragmentBasicState(fragment);
            return result != null ? new Fragment.SavedState(result) : null;
        }
        return null;
    }

```

- FragmentManager的`moveToState()`方法中, 当状态回退到`ACTIVITY_CREATED`, 会调用`saveFragmentViewState()`方法, 保存View的状态

#### 5. Fragment 的 replace 和 add 方法的区别?

- Fragment 的容器一个 FrameLayout，add 的时候是把所有的 Fragment 一层一层的叠加到了 FrameLayout 上 了，而 replace 的话首先将该容器中的其他 Fragment 去除掉然后将当前 Fragment 添加到容器中。
- 一个 Fragment 容器中只能添加一个 Fragment 种类，如果多次添加则会报异常，导致程序终止，而 replace 则无所谓，随便切换。
- 因为通过 add 的方法添加的 Fragment，每个 Fragment 只能添加一次，因此如果要想达到切换效果需要通过 Fragment 的的 hide 和 show 方法结合者使用。将要显示的 show 出来，将其他 hide 起来。这个过程 Fragment 的 生命周期没有变化。通过 replace 切换 Fragment， 每次都会执行上一个Fragment 的 onDestroyView ， 新 Fragment 的 onCreateView、onStart、onResume 方法。

```java
	//BackStackRecord
   void executeOps() {
        ...
        switch (op.cmd) {
            case OP_ADD:
                f.setNextAnim(op.enterAnim);
                mManager.addFragment(f, false);
                break;
                ...
        }
   }

   //FragmentManagerImpl
   public void addFragment(Fragment fragment, boolean moveToStateNow) {
        ...
        if (!fragment.mDetached) {
            //只能添加一次
            if (mAdded.contains(fragment)) {
                throw new IllegalStateException("Fragment already added: " + fragment);
            }
        ...
    }
```

#### 6. Fragment 如何实现类似 Activity 栈的压栈和出栈效果的？

Fragment 的事物管理器内部维持了一个`ArrayList<BackStackRecord>`，该结构可以记录我们每次 add 的 Fragment 和 replace Fragment，然后当我们点击 back 按钮的时候会自动帮我们实现退栈操作。

```java
Add this transaction to the back stack. This means that the transaction will be remembered after it is committed, and will reverse its operation when later popped off the stack. Parameters:
name An optional name for this back stack state, or null. transaction.addToBackStack("name");
//实现源码 在 BackStackRecord 中
public FragmentTransaction addToBackStack(String name) {
    if (!mAllowAddToBackStack) {
    	throw new IllegalStateException( "This FragmentTransaction is not allowed to be added to the back stack.");
    }
    mAddToBackStack = true;
    mName = name;
    return this;
}
//上面的源码仅仅做了一个标记
```

除此之外因为我们要使用 FragmentManger 用的是 FragmentActivity，因此 FragmentActivity 的 onBackPress 方法必定重新覆写了。打开看一下，发现确实如此。

```java
/**
* Take care of popping the fragment back stack or finishing the activity
* as appropriate.
*/
public void onBackPressed() {
    if (!mFragments.popBackStackImmediate()) {
    	finish();
    }
}
```

`mFragments` 的原型是 FragmentManagerImpl，看看这个方法都干嘛了

```java
	ArrayList<BackStackRecord> mBackStack;	
	@Override
	public boolean popBackStackImmediate() {
        checkStateLoss();
        executePendingTransactions();
        return popBackStackState(mActivity.mHandler, null, -1, 0);
    }
    //看看 popBackStackState 方法都干了啥，其实通过名称也能大概了解 只给几个片段，代码太多了
   private boolean popBackStackImmediate(String name, int id, int flags) {
        execPendingActions();
        ensureExecReady(true);
        ...
        boolean executePop = popBackStackState(mTmpRecords, mTmpIsPop, name, id, flags);
        ...
    }    

boolean popBackStackState(ArrayList<BackStackRecord> records, ArrayList<Boolean> isRecordPop, String name, int id, int flags) {
    ...
	while (index >= 0) {
        //从后退栈中取出当前记录对象
        BackStackRecord bss = mBackStack.get(index);
        if (name != null && name.equals(bss.getName())) {
            break;
        }
        if (id >= 0 && id == bss.mIndex) {
            break;
        }
        index--;
    }
}
```

#### 7. 谈谈Activity和Fragment的区别？

相似点：都可包含布局、可有自己的生命周期

不同点：

- Fragment相比较于Activity多出4个回调周期，在控制操作上更灵活；
- Fragment可以在XML文件中直接进行写入，也可以在Activity中动态添加；
- Fragment可以使用`show()/hide()`或者`replace()`随时对Fragment进行切换，并且切换的时候不会出现明显的效果，用户体验会好；Activity虽然也可以进行切换，但是Activity之间切换会有明显的翻页或者其他的效果，在小部分内容的切换上给用户的感觉不是很好；

#### 8. getFragmentManager、getSupportFragmentManager 、getChildFragmentManager之间的区别？

- `getFragmentManager()`所得到的是所在fragment的**父容器**的管理器， `getChildFragmentManager()`所得到的是在fragment里面**子容器**的管理器， 如果是fragment嵌套fragment，那么就需要利用`getChildFragmentManager()`；
- 因为Fragment是3.0 Android系统API版本才出现的组件，所以3.0以上系统可以直接调用`getFragmentManager()`来获取`FragmentManager()`对象，而3.0以下则需要调用`getSupportFragmentManager()` 来间接获取；

#### 9. FragmentPagerAdapter与FragmentStatePagerAdapter的区别与使用场景

- 相同点 ：二者都继承PagerAdapter
- 不同点 ：**FragmentPagerAdapter**的每个Fragment会持久的保存在FragmentManager中，只要用户可以返回到页面中，它都不会被销毁。因此适用于那些数据**相对静态**的页，Fragment**数量也比较少**的那种； **FragmentStatePagerAdapter**只保留当前页面，当页面不可见时，该Fragment就会被消除，释放其资源。因此适用于那些**数据动态性**较大、**占用内存**较多，多Fragment的情况；



## 六、屏幕适配

见《Android屏幕适配方案详解》

#### 1. 你们 Android 开发的时候，对于 UI 稿的 px 是如何适配的？

今日头条 AndroidAutoSize 和 smallestWidth 方案。

####  2. 平时如何有使用屏幕适配吗？原理是什么呢？

平时的屏幕适配一般采用的头条的屏幕适配方案。简单来说，以屏幕的一边作为适配，通常是宽。

原理：设备像素`px`和设备独立像素`dp`之间的关系是

```
px = dp * density
```

假设UI给的设计图屏幕宽度基于360dp，那么设备宽的像素点已知，即px，dp也已知，360dp，所以`density = px / dp`，之后根据这个修改系统中跟`density`相关的点即可。

## 七、Lrucache

见《Android基础》中第十一章LruCache原理解析章节

LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。当调用put()方法时，就会在集合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队头。

#### 1. LruCache使用和原理

使用：

```java
public class MyImageLoader {
    private LruCache<String, Bitmap> mLruCache;

    public MyImageLoader() {
        int maxMemory = (int) (Runtime.getRuntime().maxMemory())/1024;
        int cacheSize = maxMemory / 8;
        mLruCache = new LruCache<String, Bitmap>(cacheSize) {
            @Override
            protected int sizeOf(String key, Bitmap value) {
                return value.getRowBytes()*value.getHeight()/1024;
            }
        };

    }

    /**
     * 添加图片缓存
     */
    public void addBitmap(String key, Bitmap bitmap) {
            mLruCache.put(key, bitmap);
    }

    /**
     * 从缓存中获取图片
     *
     */
    public Bitmap getBitmap(String key) {
        return mLruCache.get(key);
    }

}
```

使用方法如上，只需要提供缓存的总容量大小并重写`sizeOf`方法计算缓存对象大小即可。这里总容量的大小也是通用方法，即进程可用内存的1/8，单位kb。然后就可以使用put方法来添加缓存对象，get方法来获取缓存对象。

它的内部存在一个 LinkedHashMap 和 maxSize，把最近使用的对象用强引用存储在 LinkedHashMap 中，给出来 put 和 get 方法，每次 put 图片时计算缓存中所有图片的总大小，跟 maxSize 进行比较，大于 maxSize，就将最久添加的图片移除，反之小于 maxSize 就添加进来。

详细来说就是LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。当调用put()方法时，就会在结合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队头元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队尾。

##### LruCache put方法核心逻辑

在添加过缓存对象后，调用trimToSize()方法，来判断缓存是否已满，如果满了就要删除近期最少使用的对象。trimToSize()方法不断地删除LinkedHashMap中队头的元素，即近期最少访问的，直到缓存大小小于最大值（maxSize）。

##### LruCache get方法核心逻辑

当调用LruCache的get()方法获取集合中的缓存对象时，就代表访问了一次该元素，将会更新队列，保持整个队列是按照访问顺序排序的。

为什么会选择LinkedHashMap呢？

这跟LinkedHashMap的特性有关，LinkedHashMap的构造函数里有个布尔参数accessOrder，当它为true时，LinkedHashMap会以访问顺序为序排列元素，否则以插入顺序为序排序元素。

##### LinkedHashMap原理

LinkedHashMap 几乎和 HashMap 一样：从技术上来说，不同的是它定义了一个 Entry<K,V> header，这个 header 不是放在 Table 里，它是额外独立出来的。LinkedHashMap 通过继承 hashMap 中的 Entry<K,V>,并添加两个属性 Entry<K,V> before,after,和 header 结合起来组成一个双向链表，来实现按插入顺序或访问顺序排序。

#### 2. DiskLruCache原理

DiskLruCache与LruCache原理相似，只是多了一个journal文件来做磁盘文件的管理，如下所示：

```
libcore.io.DiskLruCache
1
1
1

DIRTY 1517126350519
CLEAN 1517126350519 5325928
REMOVE 1517126350519
```

注：这里的缓存目录是应用的缓存目录`/data/data/pckagename/cache`，未root的手机可以通过以下命令进入到该目录中或者将该目录整体拷贝出来：

```shell
//进入/data/data/pckagename/cache目录
adb shell
run-as com.your.packagename 
cp /data/data/com.your.packagename/

//将/data/data/pckagename目录拷贝出来
adb backup -noapk com.your.packagename
```

我们来分析下这个文件的内容：

第一行：libcore.io.DiskLruCache，固定字符串。

第二行：1，DiskLruCache源码版本号。 

第三行：1，App的版本号，通过open()方法传入进去的。 

第四行：1，每个key对应几个文件，一般为1. 

第五行：空行 第六行及后续行：缓存操作记录。 第六行及后续行表示缓存操作记录，关于操作记录，我们需要了解以下三点：

DIRTY 表示一个entry正在被写入。写入分两种情况，如果成功会紧接着写入一行CLEAN的记录；如果失败，会增加一行REMOVE记录。注意单独只有DIRTY状态的记录是非法的。 当手动调用remove(key)方法的时候也会写入一条REMOVE记录。 READ就是说明有一次读取的记录。 CLEAN的后面还记录了文件的长度，注意可能会一个key对应多个文件，那么就会有多个数字。



## 八、Android消息机制

见《Android消息机制详解》

#### 1. Android消息机制介绍？

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095e29e507a873?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Android消息机制中的五大概念：

- `ThreadLocal`：当前线程存储的数据仅能从当前线程取出。
- `MessageQueue`：具有时间优先级的消息队列（单链表）。
- `Looper`：轮询消息队列，看是否有新的消息到来，保存在ThreadLocal中的。
- `Handler`：具体处理逻辑的地方。
- `Message`：需要传递的消息，可以传递数据；

过程：

1. 准备工作：创建`Handler`，如果是在子线程中创建，还需要调用`Looper#prepare()`，在`Handler`的构造函数中，会绑定其中的`Looper`和`MessageQueue`。
2. 发送消息：创建消息，使用`Handler`发送。
3. 进入`MessageQueue`：因为`Handler`中绑定着消息队列，所以`Message`很自然的被放进消息队列。
4. `Looper`轮询消息队列：`Looper`是一个死循环，一直观察有没有新的消息到来，之后从`Message`取出绑定的`Handler`，最后调用`Handler`中的处理逻辑，这一切都发生在`Looper`循环的线程，这也是`Handler`能够在指定线程处理任务的原因。

消息机制的运行流程：在子线程执行完耗时操作，当Handler发送消息时，将会调用`MessageQueue.enqueueMessage`，向消息队列中添加消息。当通过`Looper.loop`开启循环后，会不断地从线程池中读取消息，即调用`MessageQueue.next`，然后调用目标Handler（即发送该消息的Handler）的`dispatchMessage`方法传递消息，然后返回到Handler所在线程，目标Handler收到消息，调用`handleMessage`方法，接收消息，处理消息。

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095e041314ecf9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

注：虚线表示关联关系，实线表示调用关系。

#### 2. Looper在主线程中死循环为什么没有导致界面的卡死？

1. 造成**ANR**的不是主线程阻塞，而是主线程的Looper消息处理过程发生了**任务阻塞**，无法响应手势操作，不能及时刷新UI，`Looper.loop()`这个操作本身不会导致这个情况。

2. **阻塞与程序无响应**没有必然关系，虽然主线程在没有消息可处理的时候是阻塞的，但是只要保证有消息的时候能够立刻处理，程序是不会无响应的。Looper会在没有消息的时候阻塞当前线程，释放CPU资源，等到有消息到来的时候，再唤醒主线程。

3. App进程中是需要死循环的，如果循环结束的话，App进程就结束了。

#### 3. IdHandler(闲时机制）介绍

介绍： IdleHandler是在Hanlder空闲时处理空闲任务的一种机制。

IdleHandler 可以用来提升性能，主要用在我们希望能够在当前线程消息队列空闲时做些事情，最好不要做耗时操作。

```java
//getMainLooper().myQueue()或者Looper.myQueue()
Looper.myQueue().addIdleHandler(new IdleHandler() {  
    @Override  
    public boolean queueIdle() {  
        //你要处理的事情
        return false;//返回true就是单次回调后不删除，下次进入空闲时继续回调该方法，false只回调单次。    
    }  
});
```

执行场景：

- `MessageQueue`没有消息，队列为空的时候。
- `MessageQueue`属于延迟消息，当前没有消息执行的时候。

会不会发生死循环： 答案是否定的，`MessageQueue`使用计数的方法保证一次调用`MessageQueue#next`方法只会使用一次的`IdleHandler`集合。

#### 4. 同步屏障?

屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。

同步屏障是通过`MessageQueue#postSyncBarrier`方法插入到消息队列的:

- 屏障消息和普通消息的区别在于屏障没有tartget，普通消息有target是因为它需要将消息分发给对应的target，而屏障不需要被分发，它就是用来挡住普通消息来保证异步消息优先处理的。
- 屏障和普通消息一样可以根据时间来插入到消息队列中的适当位置，并且只会挡住它后面的同步消息的分发。
- postSyncBarrier返回一个int类型的数值，通过这个数值可以撤销屏障。
- postSyncBarrier方法是私有的，如果我们想调用它就得使用反射。
- 插入普通消息会唤醒消息队列，但是插入屏障不会。

屏障就遍历整个消息链表找到最近的一条异步消息，在遍历的过程中只有异步消息才会被处理执行到。可以看到通过这种方式就挡住了所有的普通消息。

发送异步消息：

- 构造器传入：

```java
     /**
      * @hide
      */
    public Handler(boolean async) {}

    /**
     * @hide
     */
    public Handler(Callback callback, boolean async) { }

    /**
     * @hide
     */
    public Handler(Looper looper, Callback callback, boolean async) {}
```

当调用`handler.sendMessage(msg)`发送消息，最终会走到：

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);//把消息设置为异步消息
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

- 公开的方法：在发送消息时通过 `message.setAsynchronous(true)`将消息设为异步的，这个方法是公开的，我们可以正常使用。

```java
	    Message message=Message.obtain();
        message.setAsynchronous(true);
        handler.sendMessage(message);
```

移除屏障：移除屏障可以通过MessageQueue的removeSyncBarrier(int token) 方法。

#### 5. postDelay()的具体实现？

```java
 	public final boolean postDelayed(Runnable r, long delayMillis)
    {
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }
```

handler.postDelay并不是先等待一定的时间再放入到MessageQueue中，而是直接进入MessageQueue，以MessageQueue的时间顺序排列和唤醒的方式结合实现的。

如果队列中只有这个消息，那么消息不会被发送，而是计算到时唤醒的时间，先将Looper阻塞，到时间就唤醒它。但如果此时要加入新消息，该消息队列的对头跟delay时间相比更长，则插入到头部，按照触发时间进行排序，队头的时间最小、队尾的时间最大。

如下

#### 6. post()与sendMessage()区别？

在子线程中通过Handler的post\(\)方式或send\(\)方式发送消息，最终都是调用了`sendMessageAtTime()`方法。

post的参数是RunRnable，通过getPostMessage封装为Message，sendMessage的参数是Message。

```java
//post方法
	public final boolean post(RunRnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
//send方法
	public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }
```

底层都是通过sendMessageAtTime()->enqueueMessage()->enqueueMessage():

```java
    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
	public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
 	private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        //调用MessageQueue的enqueueMessage方法
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

#### 7. 使用Handler需要注意什么问题，怎么解决的? 如何修复匿名内部类 Handler 造成的内存泄露？

- 有延时消息，在界面关闭后及时移除`Message/Runnable`，调用`handler.removeCallbacksAndMessages(null)`

- 非静态内部类他会持有他外部类的强引用，所以就有可能导致非静态内部类的生命周期可能比外部类更长，容易造成内存泄漏，最常见的就是`Handler`。

**怎么修改？**改成静态内部类，然后弱引用方式修饰外部类

**为何handler要定义为static?** 因为静态内部类不持有外部类的引用，所以使用静态的handler不会导致activity的泄露

**还要用WeakReference 包裹外部类的对象?** 这是因为我们需要使用外部类的成员，可以通过"activity. "获取变量方法等，如果直接使用强引用，显然会导致activity泄露。

```java
public class MainActivity extends AppCompatActivity {

    private TextView mTextView;
    private MyHandler mMyHandler;
    
    private static class MyHandler extends Handler {
        private WeakReference<MainActivity> mWeakReference;

        public MyHandler(MainActivity activity) {
            mWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            MainActivity mainActivity = mWeakReference.get();
            switch (msg.what) {
                case 1:
                    if (mainActivity != null) 
                        mainActivity.mTextView.setText(msg.obj + "");
                    break;
                default:
                    break;
            }
        }
    }
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = findViewById(R.id.tv_handler);
        mMyHandler = new MyHandler(MainActivity.this);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //模拟耗时操作
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //耗时操作完成，发送消息给UI线程
                Message msg = Message.obtain();
                msg.what = 1;
                msg.obj = "更新UI";
                mMyHandler.sendMessage(msg);
            }
        }).start();
    }
}
```

#### 8. 一个线程有几个Looper？为什么？

一个线程只能有一个Looper。

首先使用Looper必须要先调用**Looper.prepare()**：

```java
	public static void prepare() {
        prepare(true);
	}

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {	//多次调用prepare抛出异常
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

关键性的`sThreadLocal.set(new Looper(quitAllowed))`，先看sThreadLocal：

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```

ThreadLocal：代表了一个线程局部的变量，每条线程都只能看到自己的值，并不会意识到其它的线程中也存在该变量。

在这里ThreadLocal的作用是保证了每个线程都有各自的Looper。

#### 9.消息分发的优先级？

```java
    public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

- Message的回调方法：`message.callback.run()`，优先级最高；  

```java
//1.创建该Message的CallBack
Message msgCallBack = Message.obtain(handler, new Runnable() {
    @Override
    public void run() {
    }
});
//2.post系列方法

//handleCallback方法中调用的是Runnable的run方法。
private static void handleCallback(Message message) {
    message.callback.run();
}
```

- Handler中Callback的回调方法：`Handler.mCallback.handleMessage(msg)`，优先级仅次于1；  

```java
//1.构建Handler的CallBack
Handler.Callback callback = new Handler.Callback() {
    @Override
    public boolean handleMessage(@NonNull Message msg) {
      	//retrun true，就不执行下面的逻辑了，可以用于做优先级的处理
        return false;
    }
};
//2.Handler(callback)
```

- Handler的默认方法：`Handler.handleMessage(msg)`，优先级最低。

对于很多情况下，消息分发后的处理方法是第3种情况，即`Handler.handleMessage()`，一般地往往通过覆写该方法从而实现自己的业务逻辑。

#### 10.主线程的Looper何时退出

在App退出时，ActivityThread中的mH（Handler）收到消息后，执行退出。

```java
//ActivityThread.java
case EXIT_APPLICATION:
    if (mInitialApplication != null) {
        mInitialApplication.onTerminate();
    }
    Looper.myLooper().quit();
    break;
```

如果你尝试手动退出主线程Looper，便会抛出异常，**主线程不允许退出，一旦退出就意味着程序挂了**。

#### 11.Handler锁相关问题

`MessageQueue#enqueueMessage()`内部通过synchronized关键字保证线程安全。同时`messagequeue.next()`内部也会通过synchronized加锁，确保取的时候线程安全。

#### 12. 一个线程能否创建多个Handler，Handler跟Looper之间的对应关系 ？

- 一个Thread只能有一个Looper，一个MessageQueen，可以有多个Handler
- 以一个线程为基准，他们的数量级关系是： Thread(1) : Looper(1) : MessageQueue(1) : Handler(N)

#### 13. Handler 引起的内存泄露原因以及最佳解决方案

泄露原因：

Handler 允许我们发送延时消息，如果在延时期间用户关闭了 Activity，那么该 Activity 会泄露。 这个泄露是因为 Message 会持有 Handler，而又因为 Java 的特性，内部类会持有外部类，使得 Activity 会被 Handler 持有，这样最终就导致 Activity 泄露。

解决方案：

将 Handler 定义成静态的内部类，在内部持有Activity的弱引用，并在Acitivity的`onDestroy()`中调用`handler.removeCallbacksAndMessages(null)`及时移除所有消息。

#### 14. 可以在子线程直接new一个Handler吗？怎么做？

不可以，因为在**主线程**中，Activity内部包含一个Looper对象，它会自动管理Looper，处理子线程中发送过来的消息。而对于**子线程**而言，没有任何对象帮助我们维护Looper对象，所以需要我们自己手动维护。所以要在子线程开启Handler要先创建Looper，并开启Looper循环。

```java
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_three);
        new Thread(new Runnable() {
            @Override
            public void run() {
                //创建Looper，MessageQueue
                Looper.prepare();
                new Handler().post(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(HandlerActivity.this,"toast",Toast.LENGTH_LONG).show();
                    }
                });
                //开始处理消息
                Looper.loop();
            }
        }).start();
    }
```

#### 15. Message可以如何创建？哪种效果更好，为什么？

可以通过三种方法创建：

- 直接生成实例**Message m = new Message()**
- 通过**Message m = Message.obtain()**
- 通过**Message m = mHandler.obtainMessage()**

后两者效果更好，因为Android默认的消息池中消息数量是50(8.0)，而后两者是直接在消息池中取出一个Message实例，这样做就可以避免多生成Message实例。

#### 16. 当MessageQueue 没有消息的时候，在干什么，会占用CPU资源吗?

`MessageQueue` 没有消息时，便阻塞在 loop 的 `queue.next()` 方法这里。具体就是会调用到`nativePollOnce`方法里，最终调用到`epoll_wait()`进行阻塞等待。

这时，主线程会进行休眠状态，也就不会消耗CPU资源。当下个消息到达的时候，就会通过pipe管道写入数据然后唤醒主线程进行工作。

这里涉及到阻塞和唤醒的机制叫做 `epoll 机制`。

 `epoll 机制`是一种**文件描述符和I/O多路复用**：

> 在Linux操作系统中，可以将一切都看作是文件，而文件描述符简称fd，当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符，可以理解为一个索引值。

> I/O多路复用是一种机制，让单个进程可以监视多个文件描述符，一旦某个描述符就绪（一般是读就绪或写就绪），能够通知程序进行相应的读写操作

所以`I/O`多路复用其实就是一种监听读写的通知机制，而Linux提供的三种 IO 复用方式分别是：`select、poll 和 epoll` 。而这其中`epoll`是大部分情况下性能最好的多路I/O就绪通知方法。

所以，这里用到的`epoll`其实就是一种I/O多路复用方式，用来监控多个文件描述符的I/O事件。通过`epoll_wait`方法等待I/O事件，如果当前没有可用的事件则阻塞调用线程。




## 九、View事件分发机制

#### 1. 自定义View,事件分发机制讲一讲

我觉得事件分发机制流程可以分为三部分，分别是`从外传里，从里传外，消费之后`。

1）**首先，从最外面一层传到最里面一层：**

如果当前是`viewgroup`层级，就会判断 `onInterceptTouchEvent`是否为true，如果为true，则代表事件要消费在这一层级，不再往下传递。接着便执行当前 viewgroup 的onTouchEvent方法。如果`onInterceptTouchEvent`为false，则代表事件继续传递到下一层级的 `dispatchTouchEvent`方法，接着一样的代码逻辑，一直到最里面一层的view。

伪代码解释：

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    boolean isConsume = false;
    if (isViewGroup) {
        if (onInterceptTouchEvent(event)) {
            isConsume = onTouchEvent(event);
        } else {
            isConsume = child.dispatchTouchEvent(event);
        }

    } else {
        //isView
        isConsume = onTouchEvent(event);
    }
    return isConsume;
}
```

2）**到最里层的view之后，view本身还是可以选择消费或者传到外面。**

到最里面一层就会直接执行`onTouchEvent`方法，这时候，view有没有权利拒绝消费事件呢？ 按道理view作为最底层的，应该是没有发言权才对。但是呢，秉着公平公正原则，view也是可以拒绝的，可以在`onTouchEvent`方法返回false，表示他不想消费这个事件。那么它的父容器的`onTouchEvent`又会被调用，如果父容器的onTouchEvent又返回false，则又交给上一级。一直到最上层，也就是Activity的`onTouchEvent`被调用。

伪代码解释：

```java
public void handleTouchEvent(MotionEvent event) {
    if (!onTouchEvent(event)) {
        getParent.onTouchEvent(event);
    }
}
```

3）**消费之后**

当某一层viewGroup的`onInterceptTouchEvent`为true，则代表当前层级要消费事件。如果它的`onTouchListener`被设置了的话，则onTouch会被调用，如果onTouch的返回值返回true，则`onTouchEvent`不会被调用。如果返回false或者没有设置onTouchListener，则会继续调用onTouchEvent。而onClick方法则是设置了`onClickListener`则会被正常调用。

伪代码解释：

```java
public void consumeEvent(MotionEvent event) {
    if (setOnTouchListener) {
        int tag = onTouch();
        if (!tag) {
            onTouchEvent(event);
        }
    } else {
        onTouchEvent(event);
    }

    if (setOnClickListener) {
        onClick();
    }
}
```

#### 2.dispatchTouchEvent,onInterceptEvent,onTouchEvent顺序，关系

Android事件分发顺序：**Activity（Window） -> ViewGroup -> View**

![](http://upload-images.jianshu.io/upload_images/944365-aa8416fc6d2e5ecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3. 代码实现一个长按事件

```java
public class LongPressView extends View{  
    private int mLastMotionX, mLastMotionY;  
    //是否移动了  
    private boolean isMoved;  
    //长按的runnable  
    private Runnable mLongPressRunnable;  
    //移动的阈值  
    private static final int TOUCH_SLOP = 20;  
  
    public LongPressView(Context context) {  
        super(context);  
        mLongPressRunnable = new Runnable() {  
              
            @Override  
            public void run() {     
                //触发
                performLongClick();  
            }  
        };  
    }  
  
    public boolean dispatchTouchEvent(MotionEvent event) {  
        int x = (int) event.getX();  
        int y = (int) event.getY();  
          
        switch(event.getAction()) {  
        case MotionEvent.ACTION_DOWN:  
            mLastMotionX = x;  
            mLastMotionY = y; 
            //置false
            isMoved = false;  
            //添加一个postDelayed事件
            postDelayed(mLongPressRunnable, ViewConfiguration.getLongPressTimeout());  
            break;  
        case MotionEvent.ACTION_MOVE:  
            if(isMoved) break;  
            if(Math.abs(mLastMotionX-x) > TOUCH_SLOP   
                    || Math.abs(mLastMotionY-y) > TOUCH_SLOP) {  
                //移动超过阈值，则表示移动了  
                isMoved = true;  
                //remove
                removeCallbacks(mLongPressRunnable);  
            }  
            break;  
        case MotionEvent.ACTION_UP:  
            //释放了  
            removeCallbacks(mLongPressRunnable);  
            break;  
        }  
        return true;  
    }  
}  

```

#### 4. 手势操作ActionCancel后怎么取消

如果某一个子View处理了Down事件，那么随之而来的Move和Up事件也会交给它处理。但是交给它处理之前，父View还是可以拦截事件的，如果拦截了事件，那么子View就会收到一个Cancel事件，并且不会收到后续的Move和Up事件。

手势操作ActionCancel是ViewGroup拦截了Move事件，这个Move事件将会转化为Cancel事件传递给子View

取消2种方式：

- 修改ViewGroup不拦截Move事件
- 子View可以通过设置`requestDisallowInterceptTouchEvent(true)`来达到禁止父ViewGroup拦截事件的目的。

#### 5. setOnTouchListener,onClickeListener和onTouchEvent的关系

如果它的`onTouchListener`被设置了的话，则onTouch会被调用，如果onTouch的返回值返回true，则`onTouchEvent`不会被调用。如果返回false或者没有设置onTouchListener，则会继续调用onTouchEvent。而onClick方法则是onTouchEvent被调用且设置了`onClickListener`则会被正常调用。

伪代码解释：

```java
public void consumeEvent(MotionEvent event) {
    if (setOnTouchListener) {
        int tag = onTouch();
        if (!tag) {
            onTouchEvent(event);
        }
    } else {
        onTouchEvent(event);
    }
}

public void onTouchEvent(event){
    if (setOnClickListener) {
        onClick();
    }
}
```

#### 6. 横向 ScrollView、纵向 ListView 怎么处理滑动冲突?

解决滑动冲突的根本就是要在适当的位置进行拦截，那么就有两种解决办法：

- `外部拦截`：从父view端处理，根据情况决定事件是否分发到子view
- `内部拦截`：从子view端处理，根据情况决定是否阻止父view进行拦截，其中的关键就是`requestDisallowInterceptTouchEvent`方法。

**外部拦截法**，其实就是在`onInterceptTouchEvnet`方法里面进行判断，是否拦截，见代码：

```java
    //外部拦截法：父view.java      
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean intercepted = false;
        //父view拦截条件
        boolean parentCanIntercept;

        switch (ev.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                intercepted = false;
                break;
            case MotionEvent.ACTION_MOVE:
                if (parentCanIntercept) {
                    intercepted = true;
                } else {
                    intercepted = false;
                }
                break;
            case MotionEvent.ACTION_UP:
                intercepted = false;
                break;
        }
        return intercepted;

    }
```

还是比较简单的，直接判断拦截条件，然后返回true就代表拦截，false就不拦截，传到子view。注意的是`ACTION_DOWN`状态不要拦截，如果拦截，那么后续事件就直接交给父view处理了，也就没有拦截不拦截的问题了。

**内部拦截法**，就是通过`requestDisallowInterceptTouchEvent`方法让父view不要拦截。

```java
    //父view.java            
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (ev.getActionMasked() == MotionEvent.ACTION_DOWN) {
            return false;
        } else {
            return true;
        }
    }

    //子view.java
    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        //父view拦截条件
        boolean parentCanIntercept;

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                if (parentCanIntercept) {
                    getParent().requestDisallowInterceptTouchEvent(false);
                }
                break;
            case MotionEvent.ACTION_UP:
                break;
        }
        return super.dispatchTouchEvent(event);
    }
```

`requestDisallowInterceptTouchEvent(true)`的意思是阻止父view拦截事件，也就是传入true之后，父view就不会再调用`onInterceptTouchEvent`。反之，传入false就代表父view可以拦截，也就是会走到父view的`onInterceptTouchEvent`方法。所以需要父view拦截的时候，就传入flase，需要父view不拦截的时候就传入true。

#### 7. MotionEvent是什么？包含几种事件？什么条件下会产生？

MotionEvent是手指接触屏幕后所产生的一系列事件。典型的事件类型有如下：

- **ACTION_DOWN**：手指刚接触屏幕
- **ACTION_MOVE**：手指在屏幕上移动
- **ACTION_UP**：手指从屏幕上松开的一瞬间
- **ACTION_CANCELL**：手指保持按下操作，并从当前控件转移到外层控件时触发

正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，考虑如下几种情况：

- 点击屏幕后松开，事件序列：DOWN→UP
- 点击屏幕滑动一会再松开，事件序列为DOWN→MOVE→.....→MOVE→UP

#### 8. ACTION_CANCEL什么时候触发，触摸button然后滑动到外部抬起会触发点击事件吗，再滑动回去抬起会么？

- 一般`ACTION_CANCEL`和`ACTION_UP`都作为View一段事件处理的结束。如果在父View中拦截`ACTION_UP`或`ACTION_MOVE`，在第一次父视图拦截消息的瞬间，父视图指定子视图不接受后续消息了，同时子视图会收到`ACTION_CANCEL`事件。
- 如果触摸某个控件，但是又不是在这个控件的区域上抬起（移动到别的地方了），就会出现`ACTION_CANCEL`事件。

不会。

#### 9. onAttachToWindow什么时候调用？

View的`onAttachToWindow() `是在其`dispatchAttachedToWindow(AttachInfo info, int visibility)`里被无条件调用的；

而View的`dispatchAttachedToWindow()`有两个被调用途径:

1. ViewRootImpl 第一次 `performTraversal()`时会将整个view tree里所有有view的 `dispatchAttachedToWindow()` DFS 调用一遍.
2. ViewGroup 的 `addViewInner(View child, int index, LayoutParams params, boolean preventRequestLayout):`



## 十、View绘制

#### 1. 怎么计算一个View在屏幕可见部分的百分比？

- View.getGlobalVisibleRect(rect); 	// 以屏幕 左上角 为参考系的
- View.getLocalVisibleRect(rect);       //以目标 View 左上角 为参考系

true: View 全部或者部分可见 

false: View 全部不可见

```java

```

#### 2. 如何自定义实现一个FlexLayout

```java

```

#### 3. 自定义实现一个九宫格如何实现

```java

```

#### 4. onMeasure,onLayout,onDraw关系

![img](http://upload-images.jianshu.io/upload_images/3985563-5f3c64af676d9aee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

View的测量流程：

![img](http://upload-images.jianshu.io/upload_images/3985563-d1a57294428ff668.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

View的布局流程：

![img](http://upload-images.jianshu.io/upload_images/3985563-8aefac42b3912539.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

View的绘制过程遵循如下几步：

①绘制背景 background.draw(canvas)

②绘制自己（onDraw）

③绘制Children(dispatchDraw)

④绘制装饰（onDrawScrollBars）

View绘制流程：

![img](http://upload-images.jianshu.io/upload_images/3985563-594f6b3cde8762c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从View的测量、布局和绘制原理来看，要实现自定义View，根据自定义View的种类不同，可能分别要自定义实现不同的方法。但是这些方法不外乎：**onMeasure()方法，onLayout()方法，onDraw()方法。**

**onMeasure()方法**：单一View，一般重写此方法，针对wrap_content情况，规定View默认的大小值，避免于match_parent情况一致。ViewGroup，若不重写，就会执行和单子View中相同逻辑，不会测量子View。一般会重写onMeasure()方法，循环测量子View。

**onLayout()方法**：单一View，不需要实现该方法。ViewGroup必须实现，该方法是个抽象方法，实现该方法，来对子View进行布局。

**onDraw()方法**：无论单一View，或者ViewGroup都需要实现该方法，因其是个空方法

#### 5.onCreate,onResume,onStart里面，什么地方可以获得宽高

如果在onCreate、onStart、onResume中直接调用View的getWidth/getHeight方法，是无法得到View宽高的正确信息，因为view的measure过程与Activity的生命周期是不同步的，所以无法保证在这些生命周期里view的measure已经完成。所以很有可能获取的宽高为0。

所以主要有以下三个方法来获取view的宽高：

**view.post()方法**

在该方法里的runnable对象，能保证view已经绘制完成，也就是执行完measure、layout和draw方法了。

```java
    view.post(new Runnable() {
        @Override
        public void run() {
            int width = view.getWidth();
            int hight = view.getHeight();
        }
    });
```
**onWindowFocusChanged方法**

Activity中可以重写onWindowFocusChanged方法，该方法表示Activity的窗口得到焦点或者失去焦点的时候，所以Activitiy获取焦点时，view肯定绘制完成了，这时候获取宽高也是没问题的：

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if(hasFocus){
        int width = view.getWidth();
        int hight = view.getHeight();
    }
}
```
**ViewTreeObserver注册OnGlobalLayoutListener接口**

ViewTreeObserver是一个观察者，主要是用来观察视图树的各种变化。OnGlobalLayoutListener的作用是当View树的状态发生改变或者View树中某view的可见性发生改变时，OnGlobalLayoutListener的onGlobalLayout方法将会被回调。因此，此时获取view的宽高也是可以的。

```java
    ViewTreeObserver observer = title_name.getViewTreeObserver();
    observer.addOnGlobalLayoutListener(new OnGlobalLayoutListener() {
        @Override
        public void onGlobalLayout() {
           int width = view.getWidth();
           int hight = view.getHeight();
        }
    });
```

#### 6.为什么view.post可以获得宽高，有看过view.post的源码吗？

能获取宽高的原因肯定就是因为在此之前view 绘制已经完成，所以`View.post()` 添加的任务能够保证在所有 View 绘制流程结束之后才被执行。

看看post的源码：

```java
 public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action); 
        }
        // Assume that post will succeed later
        ViewRootImpl.getRunQueue().post(action);  
        return true;
    }

//RunQueue .class
     void post(Runnable action) {
            postDelayed(action, 0);
        }
 
        void postDelayed(Runnable action, long delayMillis) {
            HandlerAction handlerAction = new HandlerAction();
            handlerAction.action = action;
            handlerAction.delay = delayMillis;
 
            synchronized (mActions) {
                mActions.add(handlerAction);
            }
        }

        void executeActions(Handler handler) {
            synchronized (mActions) {
                final ArrayList<HandlerAction> actions = mActions;
                final int count = actions.size();
 
                for (int i = 0; i < count; i++) {
                    final HandlerAction handlerAction = actions.get(i);
                    handler.postDelayed(handlerAction.action, handlerAction.delay);
                }
 
                actions.clear();
            }
        }
```

所以在执行`View.post()`的方法时，那些Runnable并没有马上被执行，而是保存到RunQueue里面,然后通过`executeActions`方法执行，也就是通过handler，post了一个延时任务Runnable。而`executeActions`方法什么时候会执行呢？

```java
private void performTraversals() {
    getRunQueue().executeActions(attachInfo.mHandler);
    ...
    performMeasure();
    ...
    performLayout();
    ...
    performDraw();
 }
```

可以看到在`performTraversals`方法中执行了，但是在view绘制之前，这是因为在绘制之前就把需要执行的`runnable`封装成Message发送到`MessageQueue`里排队了，但是Looper不会马上去取这个消息，因为`Looper`会按顺序取消息，主线程还有什么消息没执行完呢？其实就是当前的这个`performTraversals`所在的任务，所以要等下面的·performMeasure，performLayout，performDraw·都执行完，也就是view绘制完毕了，才会去执行之前我们post的那个runnable，也就是我们能在`view.post`方法里的`runnable`能获取宽高的主要原因了。

`view.post()`的原理：

**以Handler为基础，`view.post()` 将传入任务添加到 View绘制任务所在的消息队列尾部，从而保证View.post() 任务的执行时机是在View 绘制任务完成之后的。** 其中，几个关键点：

- `view.post()`实际操作：将`view.post()`传入的任务保存到一个数组里 
- `view.post()`添加的任务添加到 View绘制任务所在的消息队列尾部的时机：View 绘制流程的开始阶段，即 `ViewRootImpl.performTraversals()`
- `view.post()`添加的任务执行时机：在View绘制任务之后

#### 7. 自定义LinearLayout，怎么测量子View宽高

```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }

	 void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
         ...
         for (int i = 0; i < count; ++i) {
            final View child = getVirtualChildAt(i);
         	final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

            final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                    mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                            + widthUsed, lp.width);
            final int childHeightMeasureSpec = getChildMeasureSpec(
                parentHeightMeasureSpec, mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin + heightUsed, lp.height);

            child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
         }
         ...
     }
```

#### 8. setFactory和setFactory2有什么区别？

```

```

#### 9. 如何求当前Activity View的深度

```java
public static int getViewHierarchyMaxDeep(Activity activity) {
    View decorView = activity.getWindow().getDecorView();
    int height = getDeep(decorView, 1);
    Log.e("TAG", activity.getClass().getSimpleName() + " height is:" + height);
    return height;
}

// curr为当前View的层级
private static int getDeep(View view, int curr) {
    if (view instanceof ViewGroup) {
        ViewGroup parent = (ViewGroup) view;
        int childCount = parent.getChildCount();
        if (childCount > 0) {
            // 层级+1
            int max = curr + 1;
            for (int i = 0; i < childCount; i++) {
                View child = parent.getChildAt(i);
                int height = getDeep(child, curr + 1);
                if (max < height) {
                    max = height;
                }
            }
            return max;
        } else {
            return curr;
        }
    } else {
        return curr;
    }
}
```

#### 10. 计算一个view的嵌套层级

```java
private int getParents(ViewParents view){
    if(view.getParents() == null) 
        return 0;
    } else {
    return (1 + getParents(view.getParents));
   }
}
```

#### 11. Activity内LinearLayout红色wrap_content,包含View绿色wrap_content,求界面颜色

```xml
<LinearLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="@color/red"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <View
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@color/green"
    />
</LinearLayout>
```

答案是绿色

![img](http://upload-images.jianshu.io/upload_images/3985563-e3f20c6662effb7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 12. 自定义view效率高于xml定义吗？说明理由。

自定义view效率高于xml定义：少了解析xml；自定义View 减少了ViewGroup与View之间的测量，包括父量子，子量自身，子在父中位置摆放，当子view变化时，父的某些属性都会跟着变化。

#### 13. scrollTo()和scollBy()的区别？

- scollBy内部调用了scrollTo，它是基于当前位置的相对滑动；而scrollTo是绝对滑动，因此如果使用相同输入参数多次调用scrollTo方法，由于View的初始位置是不变的，所以只会出现一次View滚动的效果

- 两者都只能对View内容的滑动，而非使View本身滑动。可以使用Scroller有过度滑动的效果

#### 14. Scroller是怎么实现View的弹性滑动？

在`MotionEvent.ACTION_UP`事件触发时调用`startScroll()`方法，该方法并没有进行实际的滑动操作，而是记录滑动相关量（滑动距离、滑动时间）

接着调用`invalidate/postInvalidate`方法，请求 View 重绘，导致`view.draw`方法被执行

当View重绘后会在draw方法中调用`computeScroll()`方法，而`computeScroll()`又会去向`Scroller`获取当前的`scrollX`和`scrollY`；然后通过`scrollTo`方法实现滑动；接着又调用`postInvalidate`方法来进行第二次重绘，和之前流程一样，如此反复导致View不断进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，直到整个滑动过成结束。

![img](https://user-gold-cdn.xitu.io/2019/3/8/1695c37ec2b0e11b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```java
mScroller = new Scroller(context);


@Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_UP:
                // 滚动开始时X的坐标,滚动开始时Y的坐标,横向滚动的距离,纵向滚动的距离
                mScroller.startScroll(getScrollX(), 0, dx, 0);
                invalidate();
                break;
        }
        return super.onTouchEvent(event);
    }

@Override
    public void computeScroll() {
        // 重写computeScroll()方法，并在其内部完成平滑滚动的逻辑
        if (mScroller.computeScrollOffset()) {
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            invalidate();
        }
    }
```

#### 15. invalidate()和postInvalidate()的区别 ？

- `invalidate()`与`postInvalidate()`都用于刷新View，主要区别是`invalidate()`在主线程中调用，若在子线程中使用需要配合handler

- `postInvalidate()`可在子线程中直接调用

#### 16. 自定义View如何考虑机型适配 ?

- 合理使用warp_content，match_parent

- 尽可能的是使用RelativeLayout，引入android的约束布局

- 针对不同的机型，使用不同的布局文件放在对应的目录下，android会自动匹配。

- 尽量使用点9图片。

- 使用与密度无关的像素单位dp，sp

- 切图的时候切大分辨率的图，应用到布局当中。在小分辨率的手机上也会有很好的显示效果。

#### 17. 说下Measurepec这个类

作用：通过宽测量值**widthMeasureSpec**和高测量值**heightMeasureSpec**决定View的大小

组成：一个32位int值，高2位代表**SpecMode**(测量模式)，低30位代表**SpecSize**( 某种测量模式下的规格大小)。

三种模式：

- **UNSPECIFIED**：父容器不对View有任何限制，要多大有多大。常用于系统内部。
- **EXACTLY**(精确模式)：父视图为子视图指定一个确切的尺寸SpecSize。对应LyaoutParams中的match_parent或具体数值。
- **AT_MOST**(最大模式)：父容器为子视图指定一个最大尺寸SpecSize，View的大小不能大于这个值。对应LayoutParams中的wrap_content。

决定因素：值由**子View的布局参数LayoutParams**和父容器的**MeasureSpec**值共同决定，所以就有一个父布局测量模式，子视图布局参数，以及子view本身的`MeasureSpec`关系图,具体规则见下图：

![img](https://user-gold-cdn.xitu.io/2019/4/1/169d7a649cc67de5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```java
    public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }

```

实际应用时：

对于自定义的单一view，一般可以不处理`onMeasure`方法，如果要对宽高进行自定义，就重写onMeasure方法，并将算好的宽高通过`setMeasuredDimension`方法传进去。

对于自定义的ViewGroup，一般需要重写`onMeasure`方法，并且调用`measureChildren`方法遍历所有子View并进行测量（measureChild方法是测量具体某一个view的宽高），然后可以通过`getMeasuredWidth/getMeasuredHeight`获取宽高，最后通过`setMeasuredDimension`方法存储本身的总宽高。

#### 18. requestLayout和invalidate?

`requestLayout`方法是用来触发绘制流程，他会会一层层调用 parent 的`requestLayout`，一直到最上层也就是`ViewRootImpl#requestLayout`，这里也就是判断线程的地方了，最后会执行到`performMeasure -> performLayout -> performDraw` 三个绘制流程，也就是测量——布局——绘制。

```java
    @Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();//执行绘制流程
        }
    }
```

其中`performMeasure`方法会执行到View的measure方法，用来测量大小。`performLayout`方法会执行到view的layout方法，用来计算位置。`performDraw`方法需要注意下，他会执行到view的draw方法，但是并不一定会进行绘制，只有只有 flag 被设置为 `PFLAG_DIRTY_OPAQUE` 才会进行绘制。

`invalidate`方法也是用来触发绘制流程，主要表现就是会调用`draw()`方法。虽然他也会走到`scheduleTraversals`方法，也就是会走到三大流程，但是View会通过`mPrivateFlags`来判断是否进行`onMeasure`和`onLayout`操作。而在用`invalidate`方法时，更新了`mPrivateFlags`，所以不会进行`measure`和`layout`。同时他也会设置Flag为`PFLAG_DIRTY_OPAQUE`，所以肯定会执行onDraw方法。

```java
private void invalidateRectOnScreen(Rect dirty) {
        final Rect localDirty = mDirty;
        //...
        if (!mWillDrawSoon && (intersected || mIsAnimating)) {
            scheduleTraversals();//执行绘制流程
        }
    }
```

最后看一下`scheduleTraversals`方法中三大绘制流程逻辑，`FORCE_LAYOUT`标志才会`onMeasure`和`onLayout`，`PFLAG_DIRTY_OPAQUE`标志才会`onDraw`：

```java
  public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    final boolean forceLayout = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT;
    // 只有mPrivateFlags为PFLAG_FORCE_LAYOUT的时候才会进行onMeasure方法
    if (forceLayout || needsLayout) {
      onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    // 设置 LAYOUT_REQUIRED flag
    mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
  }


  public void layout(int l, int t, int r, int b) {
    ...
    //判断标记位为PFLAG_LAYOUT_REQUIRED的时候才进行onLayout方法
    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        onLayout(changed, l, t, r, b);
        }
    }



public void draw(Canvas canvas) {
    final int privateFlags = mPrivateFlags;
    // flag 是 PFLAG_DIRTY_OPAQUE 则需要绘制
    final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
            (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
    mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;
    if (!dirtyOpaque) {
        drawBackground(canvas);
    }
    if (!dirtyOpaque) onDraw(canvas);
    // 绘制 Child
    dispatchDraw(canvas);
    // foreground 不管 dirtyOpaque 标志，每次都会绘制
    onDrawForeground(canvas);
}   
```

总结：

> 虽然两者都是用来触发绘制流程，但是在measure和layout过程中，只会对 flag 设置为 FORCE_LAYOUT 的情况进行重新测量和布局，而draw方法中只会重绘flag为 dirty 的区域。requestLayout 是用来设置FORCE_LAYOUT标志，invalidate 用来设置 dirty 标志。所以 requestLayout 只会触发 measure 和 layout，invalidate 只会触发 draw。



## 十一、Drawbale和动画

#### 1. 介绍一下android动画

- View 动画：
  - 作用对象是 View，可用 xml 定义，建议 xml 实现比较易读
  - 支持四种效果：平移、缩放、旋转、透明度
- 帧动画：
  - 通过 AnimationDrawable 实现，容易 OOM
- 属性动画：
  - 可作用于任何对象，可用 xml 定义，Android 3.0 引入，建议代码实现比较灵活
  - 包括 ObjectAnimator、ValueAnimator、AnimatorSet
  - 时间插值器：根据时间流逝的百分比计算当前属性改变的百分比，系统预置匀速、加速、减速等插值器
  - 类型估值器：根据当前属性改变的百分比计算改变后的属性值，系统预置整型、浮点、色值等类型估值器

- 使用注意事项：避免使用帧动画，容易OOM；界面销毁时停止动画，避免内存泄漏；开启硬件加速，提高动画流畅性

- 硬件加速原理：将 cpu 一部分工作分担给 gpu ，使用 gpu 完成绘制工作；从工作分摊和绘制机制两个方面优化了绘制速度

属性动画有两个步聚：

1.计算属性值

2.为目标对象的属性设置属性值，即应用和刷新动画
    
![image](https://upload-images.jianshu.io/upload_images/2893137-fe2225f697a60433.png?imageMogr2/auto-orient/)
    
计算属性分为3个过程：

过程一：

计算已完成动画分数 elapsed fraction。为了执行一个动画，你需要创建一个ValueAnimator，并且指定目标对象属性的开始、结束和持续时间。在调用 start 后的整个动画过程中，ValueAnimator 会根据已经完成的动画时间计算得到一个0 到 1 之间的分数，代表该动画的已完成动画百分比。0表示 0%，1 表示 100%。

过程二：

计算插值（动画变化率）interpolated fraction 。当 ValueAnimator计算完已完成的动画分数后，它会调用当前设置的TimeInterpolator，去计算得到一个interpolated（插值）分数，在计算过程中，已完成动画百分比会被加入到新的插值计算中。

过程三：

计算属性值当插值分数计算完成后，ValueAnimator会根据插值分数调用合适的 TypeEvaluator去计算运动中的属性值。
以上分析引入了两个概念：已完成动画分数（elapsed fraction）、插值分数( interpolated fraction )。

**原理及特点：**

1.属性动画：

插值器：作用是根据时间流逝的百分比来计算属性变化的百分比

估值器：在插值器的基础上由这个东西来计算出属性到底变化了多少数值的类

其实就是利用插值器和估值器，来计出各个时刻View的属性，然后通过改变View的属性来实现View的动画效果。

2.View动画:

只是影像变化，view的实际位置还在原来地方。

3.帧动画：

是在xml中定义好一系列图片之后，使用AnimatonDrawable来播放的动画。

**它们的区别：**

属性动画才是真正的实现了 view 的移动，补间动画对view 的移动更像是在不同地方绘制了一个影子，实际对象还是处于原来的地方。

当动画的 repeatCount 设置为无限循环时，如果在Activity退出时没有及时将动画停止，属性动画会导致Activity无法释放而导致内存泄漏，而补间动画却没问题。

xml 文件实现的补间动画，复用率极高。在 Activity切换，窗口弹出时等情景中有着很好的效果。

使用帧动画时需要注意，不要使用过多特别大的图，容导致内存不足。

#### 2.Bitmap、Drawable与View有什么区别，Drawable有哪些子类

Bitmap： 仅仅就是一个位图，你可以理解为一张图片在内存中的映射。 

View：View最大的作用是2个：一个是draw，也就是canvas的draw方法，还有一个作用就是测量大小

Drawable： 其实本身和bitmap没有关系, 你可以把他理解为是一个绘制工具，和view的第一个作用是一摸一样的，你能用view的canvas 画出来的东西 你用drawable 一样可以画出来，不一样的是drawable 仅仅能绘制，但是不能测量自己的大小，但是view可以。 换句话说 **drawable 承担了view的一半作用**。

**既然drawable 是 view 的一半 那还用drawable干啥？**

其实主要是复用，假设你要自定义 一组view，那你就可以把这一组view中共同的部分抽成一个drawable ，这样view就可以复用这个drawble了，不用写重复的 canvas.draw 方法 。 

**Drawable有哪些子类？**

BitmapDrawable：是对bitmap的一种封装，可以设置它包装的bitmap在BitmapDrawable区域内的绘制方式。

NinePatchDrawable：对 .9 格式图片进行封装，.9 格式图片可根据所需宽高进行相应缩放而不失真。

ShapeDrawable：可理解为通过颜色来构造的图形，图片效果可为纯色或渐变。

LayerDrawable：表示一种层次化的Drawable集合，通过将不同的Drawable放置在不同的层面从而达到一种叠加后的效果。

StateListDrawable：也表示Drawable的集合，每个Drawable对应View的一种状态，系统根据View的状态选择合适的Drawable。

LevelListDrawable：同样表示一个Drawable集合，集合中每个Drawable都有一个等级（Level）。

TransitionDrawable：实现两个Drawable之间淡入淡出的效果。

InsetDrawable：可以将其他的Drawable内嵌到自己当中，并可以在四周留出一定的间距。

ScaleDrawable：根据自己的等级（level）将指定的Drawable缩放到一定的比例。

ClipDrawable：根据当前自己的等级来裁剪一个Drawable。

ColorDrawable：一个专门用指定颜色来填充画布的Drawable

#### 3. 属性动画更新时会回调onDraw吗？

不像传统动画那样需要不停调用onDraw方法绘制界面，可以通过get、set方法，去真实的改变一个view的属性的。

#### 4. 两个getDrawable取得的对象，有什么区别？

1. 当你这个Drawable不受主题影响时

   ```java
   ResourcesCompat.getDrawable(getResources(), R.drawable.name, null);
   ```

2. 当你这个Drawable受当前Activity主题的影响时

   ```java
   ContextCompat.getDrawable(getActivity(), R.drawable.name);
   ```

3. 当你这个Drawable想使用另外一个主题样式时

   ```java
   ResourcesCompat.getDrawable(getResources(), R.drawable.name, anotherTheme);
   ```

#### 5. 补间动画与属性动画的区别，哪个效率更高？

补间动画和属性动画主要区别：

- 作用对象不同，补间动画只能作用在view上，属性动画可以作用在所有对象上。
- 属性变化不同，补间动画只是改变显示效果，不会改变view的属性，比如位置、宽高等，而属性动画实际改变对象的属性。
- 动画效果不同，补间动画只能实现位移、缩放、旋转和透明度四种动画操作，而属性动画还能实现补间动画所有效果及其他更多动画效果。

属性动画操作的是对象的实例属性，例如translationX，然后反射调用set和get方法，多个属性动画同时执行，会频繁反射调用类方法，降低性能。

补间动画只产生了一个动画效果，其真实的坐标并没有发生改变，是效果一直在发生变化，没有频繁反射调用方法的耗费性能操作。

#### 6. 动画里面用到了什么设计模式?

策略模式：有一系列的算法，将每个算法封装起来（每个算法可以封装到不同的类中），各个算法之间可以替换，策略模式让算法独立于使用它的客户而独立变化。

属性动画，设置不同的插值器对象，就可以得到不同的变化曲线。

#### 7. 动画连续调用的原理是什么？

动画的本质都是一帧一帧的展现给用户的，只不要当fps小于60的时候，人眼基本看不出间隔，也就成了所谓的流畅动画。

#### 8. 自定义圆角图片

```java

public class RoundRectImageView extends ImageView {

    /*圆角的半径，依次为左上角xy半径，右上角，右下角，左下角*/
    private float[] rids = {10.0f, 10.0f, 10.0f, 10.0f, 0.0f, 0.0f, 0.0f, 0.0f,};
    
    ...
    
    protected void onDraw(Canvas canvas) {
        Path path = new Path();
        int w = this.getWidth();
        int h = this.getHeight();
        /*向路径中添加圆角矩形。radii数组定义圆角矩形的四个圆角的x,y半径。radii长度必须为8*/
        path.addRoundRect(new RectF(0, 0, w, h), rids, Path.Direction.CW);
        canvas.clipPath(path);
        super.onDraw(canvas);
    }
}
```

#### 9. Canvas.save()跟Canvas.restore()的调用时机

save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。

restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。

save和restore要配对使用（restore可以比save少，但不能多），如果restore调用次数比save多，会引发Error。save和restore操作执行的时机不同，就能造成绘制的图形不同。

#### 10. 为什么属性动画移动后仍可点击？

播放补间动画的时候，我们所看到的变化，都只是临时的。而属性动画呢，它所改变的东西，却会更新到这个View所对应的矩阵中，所以当ViewGroup分派事件的时候，会正确的将当前触摸坐标，转换成矩阵变化后的坐标。



## 十二、多线程

#### 1. AsyncTask的缺陷和问题，说说他的原理。

**AsyncTask是什么？**

AsyncTask是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。

AsyncTask是一个抽象的泛型类，它提供了Params、Progress和Result这三个泛型参数，其中Params表示参数的类型，Progress表示后台任务的执行进度和类型，而Result则表示后台任务的返回结果的类型，如果AsyncTask不需要传递具体的参数，那么这三个泛型参数可以用Void来代替。

**关于线程池：**

AsyncTask对应的线程池ThreadPoolExecutor都是进程范围内共享的，且都是static的，所以是AsyncTask控制着进程范围内所有的子类实例。由于这个限制的存在，当使用默认线程池时，如果线程数超过线程池的最大容量，线程池就会爆掉(3.0后默认串行执行，不会出现个问题)。针对这种情况，可以尝试自定义线程池，配合AsyncTask使用。

**关于默认线程池：**

AsyncTask里面线程池是一个核心线程数为CPU + 1，最大线程数为CPU * 2 + 1，工作队列长度为128的线程池，线程等待队列的最大等待数为128，但是可以自定义线程池。线程池是由AsyncTask来处理的，线程池允许tasks并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉。所以希望tasks能够串行运行的话，使用SERIAL_EXECUTOR。

**AsyncTask在不同的SDK版本中的区别：**

调用AsyncTask的execute方法不能立即执行程序的原因及改善方案通过查阅官方文档发现，AsyncTask首次引入时，异步任务是在一个独立的线程中顺序的执行，也就是说一次只执行一个任务，不能并行的执行，从1.6开始，AsyncTask引入了线程池，支持同时执行5个异步任务，也就是说只能有5个线程运行，超过的线程只能等待，等待前的线程直到某个执行完了才被调度和运行。换句话说，如果进程中的AsyncTask实例个数超过5个，那么假如前5都运行很长时间的话，那么第6个只能等待机会了。这是AsyncTask的一个限制，而且对于2.3以前的版本无法解决。如果你的应用需要大量的后台线程去执行任务，那么只能放弃使用AsyncTask，自己创建线程池来管理Thread。不得不说，虽然AsyncTask较Thread使用起来方便，但是它最多只能同时运行5个线程，这也大大局限了它的作用，你必须要小心设计你的应用，错开使用AsyncTask时间，尽力做到分时，或者保证数量不会大于5个，否就会遇到上面提到的问题。可能是Google意识到了AsynTask的局限性了，从Android 3.0开始对AsyncTask的API做出了一些调整：每次只启动一个线程执行一个任务，完了之后再执行第二个任务，也就是相当于只有一个后台线程在执行所提交的任务。

**一些问题：**

1.生命周期

很多开发者会认为一个在Activity中创建的AsyncTask会随着Activity的销毁而销毁。然而事实并非如此。AsynTask会一直执行，直到doInBackground()方法执行完毕，然后，如果cancel(boolean)被调用,那么onCancelled(Result result)方法会被执行；否则，执行onPostExecute(Result result)方法。如果我们的Activity销毁之前，没有取消AsyncTask，这有可能让我们的应用崩溃(crash)。因为它想要处理的view已经不存在了。所以，我们是必须确保在销毁活动之前取消任务。总之，我们使用AsyncTask需要确保AsyncTask正确的取消。

2.内存泄漏

如果AsyncTask被声明为Activity的非静态内部类，那么AsyncTask会保留一个对Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄漏。

3.结果丢失

屏幕旋转或Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效。

4.并行还是串行

在Android1.6之前的版本，AsyncTask是串行的，在1.6之后的版本，采用线程池处理并行任务，但是从Android 3.0开始，为了避免AsyncTask所带来的并发错误，又采用一个线程来串行执行任务。可以使用`executeOnExecutor()`方法来并行地执行任务。

**AsyncTask原理**

- AsyncTask中有两个线程池（SerialExecutor和THREAD_POOL_EXECUTOR）和一个Handler（InternalHandler），其中线程池SerialExecutor用于任务的排队，而线程池THREAD_POOL_EXECUTOR用于真正地执行任务，InternalHandler用于将执行环境从线程池切换到主线程。
- sHandler是一个静态的Handler对象，为了能够将执行环境切换到主线程，这就要求sHandler这个对象必须在主线程创建。由于静态成员会在加载类的时候进行初始化，因此这就变相要求AsyncTask的类必须在主线程中加载，否则同一个进程中的AsyncTask都将无法正常工作。

#### 2. Thread、AsyncTask、IntentService的使用场景与特点。

1. Thread线程，独立运行与于 Activity 的，当Activity 被 finish 后，如果没有主动停止 Thread或者 run 方法没有执行完，其会一直执行下去。
2. AsyncTask 封装了两个线程池和一个Handler（SerialExecutor用于排队，THREAD_POOL_EXECUTOR为真正的执行任务，Handler将工作线程切换到主线程），其必须在 UI线程中创建，execute 方法必须在 UI线程中执行，一个任务实例只允许执行一次，执行多次抛出异常，用于网络请求或者简单数据处理。
3. IntentService：处理异步请求，实现多线程，在onHandleIntent中处理耗时操作，多个耗时任务会依次执行，执行完毕自动结束。

#### 3. Android中了解哪些方便线程切换的类？

- **AsyncTask**：底层封装了线程池和Handler，便于执行后台任务以及在子线程中进行UI操作。

- **HandlerThread**：一种具有消息循环的线程，其内部可使用Handler。

- **IntentService**：是一种异步、会自动停止的服务，内部采用HandlerThread。

#### 4. 直接在Activity中创建一个thread跟在service中创建一个thread之间的区别？

**在Activity中被创建**：该Thread的就是为这个Activity服务的，完成这个特定的Activity交代的任务，主动通知该Activity一些消息和事件，Activity销毁后，该Thread也没有存活的意义了。

**在Service中被创建**：这是保证最长生命周期的Thread的唯一方式，只要整个Service不退出，Thread就可以一直在后台执行，一般在Service的onCreate()中创建，在onDestroy()中销毁。所以，在Service中创建的Thread，适合长期执行一些独立于APP的后台任务，比较常见的就是：在Service中保持与服务器端的长连接。

#### 5. Handler、Thread和HandlerThread的差别？

**Handler**：在android中负责发送和处理消息，通过它可以实现其他支线线程与主线程之间的消息通讯。

**Thread**：Java进程中执行运算的最小单位，亦即执行处理机调度的基本单位。某一进程中一路单独运行的程序。

**HandlerThread**：一个继承自Thread的类HandlerThread，Android中没有对Java中的Thread进行任何封装，而是提供了一个继承自Thread的类HandlerThread类，这个类对Java的Thread做了很多便利的封装。HandlerThread继承于Thread，所以它本质就是个Thread。与普通Thread的差别就在于，它在内部直接实现了Looper的实现，这是Handler消息机制必不可少的。有了自己的looper，可以让我们在自己的线程中分发和处理消息。如果不用HandlerThread的话，需要手动去调用Looper.prepare()和Looper.loop()这些方法。

#### 6. 多线程是否一定会高效

多线程的优点：

- 方便高效的内存共享，多进程下内存共享比较不便，且会抵消掉多进程编程的好处
- 较轻的上下文切换开销，不用切换地址空间，不用更改CR3寄存器，不用清空TLB
- 线程上的任务执行完后自动销毁

多线程的缺点：

- 开启线程需要占用一定的内存空间(默认情况下,每一个线程都占512KB)
- 如果开启大量的线程,会占用大量的内存空间,降低程序的性能
- 线程越多,cpu在调用线程上的开销就越大
- 程序设计更加复杂,比如线程间的通信、多线程的数据共享

综上得出，多线程**不一定**能提高效率，在内存空间紧张的情况下反而是一种负担，因此在日常开发中，应尽量

- **不要频繁创建，销毁线程，如有需求需要使用线程池**
- **减少线程间同步和通信（最为关键）**
- **避免需要频繁共享写的数据**
- **合理安排共享数据结构，避免伪共享（false sharing）**
- **使用非阻塞数据结构/算法**
- **避免可能产生可伸缩性问题的系统调用（比如mmap）**
- **避免产生大量缺页异常，尽量使用Huge Page**
- **可以的话使用用户态轻量级线程代替内核线程**



## 十三、Bitmap

#### 1. 下载一张很大的图，如何保证不 oom？

在加载图片时候先检查一下图片的大小：用 BitmapFactory.Options 参数，参数中的 inJustDecodeBounds 属性设置为 true 就可以让解析图片方法禁止为 bitmap 分配内存，返回值也不再是一个 Bitmap 对象，而是 null。虽然 Bitmap是 null 了，但是 BitmapFactory.Options 的 outWidth、outHeight 和 outMimeType 属性都会被赋值。从而可以得到图片的宽、高、大小。得到图片的大小后，我们就可以决定是否把整张图片加载到内存中还是加载一个压缩版的图片到内存中，从而就可以解决 OOM 异常。

#### 2. Bitmap的内存计算方式？

在已知图片的长和宽的像素的情况下，影响内存大小的因素会有**资源文件位置和像素点大小**。

**像素点大小**： 常见的像素点有：

- ARGB_8888：4个字节
- ARGB_4444、ARGB_565：2个字节
- ALPHA_8   每个像素占用1字节      

**资源文件位置**： 不同dpi对应存放的文件夹

![资源文件夹](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6516590e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

比如一个一张图片的像素为`180*180px`，`dpi`(设备独立像素密度)为320，如果它仅仅存放在`drawable-hdpi`，则有：

```
横向像素点 = 180 * 320/240 + 0.5f = 240 px
纵向像素点 = 180 * 320/240 + 0.5f = 240 px
```

如果 如果它仅仅存放在`drawable-xxhdpi`，则有：

```
横向像素点 = 180 * 320/480 + 0.5f = 120 px
纵向像素点 = 180 * 320/480 + 0.5f = 120 px
```

所以，对于一张`180*180px`的图片，设备dpi为320，资源图片仅仅存在`drawable-hdpi`，像素点大小为`ARGB_4444`，最后生成的文件内存大小为：

```
横向像素点 = 180 * 320/240 + 0.5f = 240 px
纵向像素点 = 180 * 320/240 + 0.5f = 240 px
内存大小 = 240 * 240 * 2 = 115200byte 约等于 112.5kb
```

Bitamp 所占内存大小 = 宽度像素 x （inTargetDensity / inDensity） x 高度像素 x （inTargetDensity / inDensity）x 一个像素所占的内存字节大小

> 这里inDensity表示目标图片的dpi（放在哪个资源文件夹下），inTargetDensity表示目标屏幕的dpi，所以你可以发现inDensity和inTargetDensity会对Bitmap的宽高进行拉伸，进而改变Bitmap占用内存的大小。

在Bitmap里有两个获取内存占用大小的方法。

- **getByteCount()**：API12 加入，代表存储 Bitmap 的像素需要的最少内存。
- **getAllocationByteCount()**：API19 加入，代表在内存中为 Bitmap 分配的内存大小，代替了 getByteCount() 方法。
- 在**不复用 Bitmap** 时，`getByteCount()` 和 `getAllocationByteCount()` 返回的结果是一样的。在通过**复用 Bitmap** 来解码图片时，那么 `getByteCount()` 表示新解码图片占用内存的大 小，`getAllocationByteCount()` 表示被复用 Bitmap 真实占用的内存大小。



#### 3.  Bitmap的高效加载？

Bitmap的高效加载在Glide中也用到了，思路：

1. 获取需要的长和宽，一般获取控件的长和宽。
2. 设置`BitmapFactory.Options`中的`inJustDecodeBounds`为true，可以帮助我们在不加载进内存的方式获得`Bitmap`的长和宽。
3. 对需要的长和宽和Bitmap的长和宽进行对比，从而获得压缩比例，放入`BitmapFactory.Options`中的`inSampleSize`属性。
4. 设置`BitmapFactory.Options`中的`inJustDecodeBounds`为false，将图片加载进内存，进而设置到控件中。

#### 4. 谈谈你对 Bitmap 的理解 , 什么时候应该手动调用 bitmap.recycle()？

Bitmap 是 android 中经常使用的一个类，它代表了一个图片资源。Bitmap消耗内存很严重，如果不注意优化代码，经常会出现 OOM 问题，优化方式通常有这么几种： 1. 使用缓存； 2. 压缩图片； 3. 及时回收；

至于什么时候需要手动调用 recycle，这就看具体场景了，原则是当我们不再使用 Bitmap 时，需要回收之。另外，我们需要注意，2.3 之前 Bitmap 对象与像素数据是分开存放的，Bitmap 对象存在 java Heap 中而像素数据存放在 Native Memory 中，这时很有必要调用 recycle 回收内存。但是 2.3 之后，Bitmap 对
象和像素数据都是存在 Heap 中，GC 可以回收其内存。

#### 5. 内存中如果加载一张500*500的png高清图片应该是占用多少的内存?

**不考虑屏幕比的话**：占用内存=500 * 500 * 4 = 1000000B ≈ 0.95MB

**考虑屏幕比的的话**：占用内存= 宽度像素 x （inTargetDensity / inDensity） x 高度像素 x （inTargetDensity / inDensity）x 一个像素所占的内存字节大小

inDensity表示目标图片的dpi（放在哪个资源文件夹下），inTargetDensity表示目标屏幕的dpi

![img](https://user-gold-cdn.xitu.io/2019/3/19/169957db5956a922?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)




## 十四、ListView/RecyclerView

#### 1. RecyclerView是怎么处理内部ViewClick冲突的

要监听RecyclerView中Item的点击事件一般有两种实现方式，第一种是在Adapter中进行点击事件的处理，第二种是在外部实现点击事件处理。

```java
// 1.定义接口
public interface OnItemClickListener {
    void onItemClick(View view, Book book);
}

// 2.监听器的注入，可使用构造时注入，也可以使用setter注入。
private Context mContext;
private OnItemClickListener mOnItemClickListener;
public ClickRecyclerViewAdapter(Context context, OnItemClickListener onItemClickListener) {
    mContext = context;
    mOnItemClickListener = onItemClickListener;
}

public void setOnItemClickListener(OnItemClickListener onItemClickListener) {
    mOnItemClickListener = onItemClickListener;
}
// 3.数据绑定及具体控件的事件监听
public void onBindViewHolder(@NonNull MyViewHolder holder, final int position) {
    holder.titleBtn.setText("按钮：" + (position + 1));
    holder.titleBtn.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Toast.makeText(mContext, "在adapter中实现点击处理逻辑", Toast.LENGTH_SHORT).show();
        }
    });

    holder.desTv.setText(data.get(position).toString());
    holder.desTv.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            mOnItemClickListener.onItemClick(v, data.get(position));
        }
    });
}

// 4.在初始化Adapter时可以注入事件处理逻辑
ClickRecyclerViewAdapter adapter=new ClickRecyclerViewAdapter(this, new ClickRecyclerViewAdapter.OnItemClickListener() {
        @Override
        public void onItemClick(View view, Book book) {
            Snackbar.make(view,"外部注入的点击逻辑："+view.getId(),Snackbar.LENGTH_SHORT).setAction("Action",null).show();
        }
});
```
#### 2. RecyclerView滑动10个，再滑回去，会有几个执行onBindView

需要重新执行onBindView的只有一种缓存区，就是缓存池mRecyclerPool。

所以我们假设从加载RecyclView开始（页面假设可以容纳7条数据）：

- 首先，7条数据会依次调用onCreateViewHolder和onBindViewHolder。
- 往下滑一条（position=7），那么会把position=0的数据放到mCacheViews中。此时mCacheViews缓存区数量为1，mRecyclerPool数量为0。然后新出现的position=7的数据通过postion在mCacheViews中找不到对应的ViewHolder，通过itemtype也在mRecyclerPool中找不到对应的数据，所以会调用onCreateViewHolder和onBindViewHolder方法。
- 再往下滑一条数据（position=8），如上。
- 再往下滑一条数据（position=9），position=2的数据会放到mCacheViews中，但是由于mCacheViews缓存区默认容量为2，所以position=0的数据会被清空数据然后放到mRecyclerPool缓存池中。而新出现的position=9数据由于在mRecyclerPool中还是找不到相应type的ViewHolder，所以还是会走onCreateViewHolder和onBindViewHolder方法。所以此时mCacheViews缓存区数量为2，mRecyclerPool数量为1。
- 再往下滑一条数据（position=10），这时候由于可以在mRecyclerPool中找到相同viewtype的ViewHolder了。所以就直接复用了，并调用onBindViewHolder方法绑定数据。
- 后面依次类推，刚消失的两条数据会被放到mCacheViews中，再出现的时候是不会调用onBindViewHolder方法，而复用的第三条数据是从mRecyclerPool中取得，就会调用onBindViewHolder方法了。

4）所以这个问题就得出结论了（假设mCacheViews容量为默认值2）：

- 如果一开始滑动的是新数据，那么滑动10个，就会走10个bindview方法。然后滑回去，会走（10-2）个bindview方法。一共18次调用。
- 如果一开始滑动的是老数据，那么滑动（10-2）个，就会走8个bindview方法。然后滑回去，会走（10-2）个bindview方法。一共16次调用。

但是但是，实际情况又有点不一样。因为Recycleview在v25版本引入了一个新的机制，预取机制。

预取机制，就是在滑动过程中，会把将要展示的一个元素提前缓存到mCachedViews中，所以滑动10个元素的时候，第11个元素也会被创建，也就多走了一次bindview方法。但是滑回去的时候不影响，因为就算提前取了一个缓存数据，只是把bindview方法提前了，并不影响总的绑定item数量。

所以滑动的是新数据的情况下就会多一次调用bindview方法。

总结

- 滑动10个，再滑回去，bindview可以是19次调用，可以是16次调用。
- 缓存的其实就是缓存item的view，在Recycleview中就是viewholder。
- cachedView就是mCacheViews缓存区中的view，是不需要重新绑定数据的。

#### 3. 如何实现RecyclerView的局部更新，用过payload吗，notifyItemChanged方法中的参数？

关于RecycleView的数据更新，主要有以下几个方法：

- `notifyDataSetChanged()`，刷新全部可见的item。
- `notifyItemChanged(int)`，刷新指定item。
- `notifyItemRangeChanged(int,int)`，从指定位置开始刷新指定个item。
- `notifyItemInserted(int)、notifyItemMoved(int)、notifyItemRemoved(int)`。插入、移动一个并自动刷新。
- `notifyItemChanged(int, Object)`，局部刷新。

可以看到，关于view的局部刷新就是`notifyItemChanged(int, Object)`方法，下面具体说说：

`notifyItemChange`有两个构造方法：

- `notifyItemChanged(int position, @Nullable Object payload)`
- `notifyItemChanged(int position)`

其中`payload`参数可以认为是你要刷新的一个标示，比如我有时候只想刷新`itemView`中的`textview`，有时候只想刷新`imageview`，那么我就可以通过`payload`参数来标示这个特殊的需求了。

```java
public abstract static class Adapter<VH extends ViewHolder> {
    ...
	//payload Optional parameter, use null to identify a "full" update
	//如果payload参数是null，那么就会来一个“完整的”更新，也就是说会全部更新。
	public final void notifyItemChanged(int position, Object payload) {
        mObservable.notifyItemRangeChanged(position, 1, payload);
    }
}

static class AdapterDataObservable extends Observable<AdapterDataObserver> {
    ...
    public void notifyItemRangeChanged(int positionStart, int itemCount, Object payload) 	 {
        for (int i = mObservers.size() - 1; i >= 0; i--) {
            mObservers.get(i).onItemRangeChanged(positionStart, itemCount, payload);
        }
    }
}
```
payload可以用来存储一些变量值或者常数，然后在notifyItemChanged中传进去payload，指定某个item进行刷新，在这个Item的onBindViewHolder中就可以从第三个参数拿到传过来的payload。

如果payloads不为空并且viewHolder已经绑定了旧数据了，那么adapter会使用payloads参数进行布局刷新。如果payloads为空，adapter就会重新绑定数据，也就是刷新整个item。但是adapter不能保证payload通过nofityItemChanged方法会被onBindViewHolder接收，例如当view没有绑定到screen时，payloads就会失效被丢弃。

```java
@Override
public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
//为空  不使用
}

@Override
public void onBindViewHolder(final RecyclerView.ViewHolder holder, final int position, List payloads) {
    final ContactHolder contact= (ContactHolder) holder;
    if(payloads.isEmpty()){
    //payloads为空 即不是调用notifyItemChanged(position,payloads)方法执行的
    //在这里进行初始化item全部控件
       contact.userName.setText(mList.get(position).getName());
       contact.userId.setText(mList.get(position).getId());
       contact.userImg.setImageResources(mList.get(position).getImg());

    }else{
        //payloads不为空 即调用notifyItemChanged(position,payloads)方法后执行的
        //在这里可以获取payloads中的数据  进行局部刷新
        //假设是int类型
        int type= (int) payloads.get(0);// 刷新哪个部分 标志位
        switch(type){
        case 0:
        contact.userName.setText(mList.get(position).getName());//只刷新userName
        break;
        case 1:
        contact.userId.setText(mList.get(position).getId());//只刷新userId
        break;
        case 2:
        contact.userImg.setImageResources(mList.get(position).getImg());//只刷新userImg
        break;
        }
    }
}

public class ContactHolder extends RecyclerView.ViewHolder{
    public TextView userName;
    public TextView userId;
    public ImageView userImg;

    public ContactHolder(View itemView) {
        super(itemView);
        userName= (TextView) itemView.findViewById(R.id.ContactSelect_list_item_user_name);
        userId= (TextView) itemView.findViewById(R.id.ContactSelect_list_item_user_employeeId);
        userImg= (ImageView) itemView.findViewById(R.id.ContactSelect_list_item_img);
    }
}
```


#### 4. RecyclerView的缓存结构是怎样的？缓存的是什么？cachedView会执行onBindView吗?

Recycleview有四级缓存，分别是`mAttachedScrap(屏幕内)`，`mCacheViews(屏幕外)`，`mViewCacheExtension(自定义缓存)`，`mRecyclerPool(缓存池)`

- `mAttachedScrap(屏幕内)`，用于屏幕内itemview快速重用，不需要重新createView和bindView
- `mCacheViews(屏幕外)`，保存最近移出屏幕的ViewHolder，包含数据和position信息，复用时必须是相同位置的ViewHolder才能复用，应用场景在那些需要来回滑动的列表中，当往回滑动时，能直接复用ViewHolder数据，不需要重新bindView。
- `mViewCacheExtension(自定义缓存)`，不直接使用，需要用户自定义实现，默认不实现。
- `mRecyclerPool(缓存池)`，当cacheView满了后或者adapter被更换，将cacheView中移出的ViewHolder放到Pool中，放之前会把ViewHolder数据清除掉，所以复用时需要重新bindView。

四级缓存按照顺序需要依次读取。所以**完整缓存流程**是：

**保存缓存流程**：

- 插入或是删除itemView时，先把屏幕内的ViewHolder保存至AttachedScrap中
- 滑动屏幕的时候，先消失的itemview会保存到CacheView，CacheView大小默认是2，超过数量的话按照先入先出原则，移出头部的itemview保存到RecyclerPool缓存池（如果有自定义缓存就会保存到自定义缓存里），RecyclerPool缓存池会按照itemview的itemtype进行保存，每个itemTyep缓存个数为5个，超过就会被回收。

**获取缓存流程**：

- AttachedScrap中获取，通过pos匹配holder

  ——>获取失败，从CacheView中获取，也是通过pos获取holder缓存
  
  ——>获取失败，从自定义缓存中获取缓存
  
  ——>获取失败，从mRecyclerPool中获取(bindview)
  
  ——>获取失败，重新创建viewholder(createViewHolder并bindview)。

需要注意的是，如果从缓存池找到缓存，还需要重新bindview。

#### 5. RecyclerView嵌套RecyclerView，NestScrollView嵌套ScrollView滑动冲突

1）`RecyclerView`嵌套`RecyclerView`的情况下，如果两者都要上下滑动，那么就会引起滑动冲突。默认情况下外层的RecycleView可滑，内层不可滑。

解决滑动冲突的办法有两种：**内部拦截法和外部拦截法**。

```java
public class ChildPresenter extends RecyclerView {
    ...
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch(ev){
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE: 
                //父层ViewGroup不要拦截点击事件 
        		getParent().requestDisallowInterceptTouchEvent(true);
            case MotionEvent.ACTION_UP:
                //正常走父view的滑动
        		getParent().requestDisallowInterceptTouchEvent(false);
        }
        return super.dispatchTouchEvent(ev);
    }
}
```

2）关于`ScrclerView`的滑动冲突还是同样的解决办法，就是进行事件拦截。

还有一个办法就是用`Nestedscrollview`代替`ScrollView`，`Nestedscrollview`是官方为了解决滑动冲突问题而设计的新的View。它的定义就是支持嵌套滑动的ScrollView。

所以直接替换成`Nestedscrollview`就能保证两者都能正常滑动了。但是要注意设置`RecyclerView.setNestedScrollingEnabled(false)`这个方法，用来取消RecyclerView本身的滑动效果。

这是因为RecyclerView默认是`setNestedScrollingEnabled(true)`，这个方法的含义是支持嵌套滚动的。也就是说当它嵌套在`NestedScrollView`中时,默认会随着`NestedScrollView`滚动而滚动,放弃了自己的滚动。所以给我们的感觉就是滞留、卡顿。所以我们将它设置为false就解决了卡顿问题，让他正常的滑动，不受外部影响。



#### 6. RecyclerView预取

因为Recycleview在v25版本引入了一个新的机制，预取机制。

预取机制，就是在滑动过程中，会把将要展示的一个元素提前缓存到mCachedViews中。

#### 7. Recycleview和Listview区别

`Recycleview布局效果更多`，增加了纵向，表格，瀑布流等效果

`Recycleview去掉了一些api`，比如setEmptyview，onItemClickListener等等，给到用户更多的自定义可能

`Recycleview去掉了设置头部底部item的功能`，专向通过viewholder的不同type实现

`Recycleview实现了一些局部刷新`，比如notifyitemchanged

`Recycleview自带了一些布局变化的动画效果`，也可以通过自定义ItemAnimator类实现自定义动画效果

`Recycleview缓存机制更全面`，增加两级缓存，还支持自定义缓存逻辑

动画区别：

- 在**RecyclerView**中，内置有许多动画API，例如：`notifyItemChanged()`, `notifyDataInserted()`, `notifyItemMoved()`等等；如果需要自定义动画效果，可以通过实现（`RecyclerView.ItemAnimator`类）完成自定义动画效果，然后调用`RecyclerView.setItemAnimator()`；
- 但是**ListView**并没有实现动画效果，但我们可以在Adapter自己实现item的动画效果；

刷新区别：

- ListView中通常刷新数据是用全局刷新`notifyDataSetChanged()`，这样一来就会非常消耗资源；**本身无法实现局部刷新**，但是如果要在ListView实现**局部刷新**，依然是可以实现的，当一个item数据刷新时，我们可以在Adapter中，实现一个`onItemChanged()`方法，在方法里面获取到这个item的position（可以通过`getFirstVisiblePosition()`），然后调用`getView()`方法来刷新这个item的数据；
- RecyclerView中可以实现局部刷新，例如：`notifyItemChanged()`

缓存区别：

- RecyclerView比ListView多两级缓存，支持多个ItemView缓存，支持开发者自定义缓存处理逻辑，支持所有RecyclerView共用同一个RecyclerViewPool(缓存池)。
- ListView和RecyclerView缓存机制基本一致，但缓存使用不同

#### 8. 说说RecyclerView性能优化。

- `bindViewHolder`方法是在UI线程进行的，此方法不能耗时操作，不然将会影响滑动流畅性。比如进行日期的格式化。
- 对于新增或删除的时候，可以使用`diffutil`进行局部刷新，少用全局刷新
- 对于`itemVIew`进行布局优化，比如少嵌套等。
- 25.1.0 (>=21)及以上使用`Prefetch` 功能，也就是预取功能，嵌套时且使用的是LinearLayoutManager，子RecyclerView可通过`setInitialPrefatchItemCount`设置预取个数
- 加大`RecyclerView缓存`，比如cacheview大小默认为2，可以设置大点，用空间来换取时间，提高流畅度
- 如果高度固定，可以设置`setHasFixedSize(true)`来避免requestLayout浪费资源，否则每次更新数据都会重新测量高度。

```java
void onItemsInsertedOrRemoved() {
   if (hasFixedSize) layoutChildren();
   else requestLayout();
}
```

- 如果多个`RecycledView` 的 Adapter 是一样的，比如嵌套的 RecyclerView 中存在一样的 Adapter，可以通过设置 `RecyclerView.setRecycledViewPool(pool);`来共用一个 `RecycledViewPool`。这样就减少了创建VIewholder的开销。
- 在RecyclerView的元素比较高，一屏只能显示一个元素的时候，第一次滑动到第二个元素会卡顿。这种情况就可以通过设置额外的缓存空间，重写`getExtraLayoutSpace`方法即可。

```java
new LinearLayoutManager(this) {
    @Override
    protected int getExtraLayoutSpace(RecyclerView.State state) {
        return size;
    }
};
```

- 设置`RecyclerView.addOnScrollListener();`来在滑动过程中停止加载的操作。
- 减少对象的创建，比如设置监听事件，可以全局创建一个，所有view公用一个listener，并且放到`CreateView`里面去创建监听，因为CreateView调用要少于bindview。这样就减少了对象创建所造成的消耗
- 用`notifyDataSetChange`时，适配器不知道整个数据集中的那些内容以及存在，再重新匹配`ViewHolder`时会花生闪烁。设置`adapter.setHasStableIds(true)`，并重写`getItemId()`来给每个Item一个唯一的ID，也就是唯一标识，就使itemview的焦点固定，解决了闪烁问题。

#### 9. ListView 如何提高其效率？ 

① 复用 ConvertView

② 自定义静态类 ViewHolder 

③ 使用分页加载

④ 使用 WeakRefrence 引用 ImageView 对象

#### 10. ViewHolder 为什么要声明为静态类？ 

非静态内部类拥有外部类对象的强引用，因此为了避免对外部类（外部类很可能是 Activity）对象的引用，那么最 好将内部类声明为 static 的。

#### 11. ListView 使用了哪些设计模式？ 

- 适配器
- 观察者 
- 享元设计模式

#### 12. 当 ListView 数据集改变后，如何更新 ListView？ 

使用该 ListView 的 adapter 的 `notifyDataSetChanged()`方法。该方法会使 ListView 重新绘制。

#### 13. ListView 如何实现分页加载

① 设置 ListView 的滚动监听器：`setOnScrollListener(new OnScrollListener{….})`
在监听器中有两个方法： 滚动状态发生变化的方法(`onScrollStateChanged`)和 listView 被滚动时调用的方法
(`onScroll`)

② 在滚动状态发生改变的方法中，有三种状态：

- 手指按下移动的状态： `SCROLL_STATE_TOUCH_SCROLL`: // 触摸滑动
- 惯性滚动（滑翔（flging）状态）： `SCROLL_STATE_FLING`: // 滑翔
- 静止状态： `SCROLL_STATE_IDLE`: // 静止

对不同的状态进行处理：

分批加载数据，只关心静止状态：关心最后一个可见的条目，如果最后一个可见条目就是数据适配器（集合）里的最后一个，此时可加载更多的数据。在每次加载的时候，计算出滚动的数量，当滚动的数量大于等于总数量的时候，可以提示用户无更多数据了。

#### 14. ListView 可以显示多种类型的条目吗？ 

可以的，ListView 显示的每个条目都是通过 baseAdapter 的 `getView(int position, View convertView, ViewGroup parent)`来展示的，理论上我们完全可以让每个条目都是不同类型的 view，除此之外 adapter 还提供了 `getViewTypeCount()`和 `getItemViewType(int position)`两个方法。在 getView 方法中我们可以根据不同的 viewtype 加载不同的布局文件。 

#### 15. ListView 如何定位到指定位置?

可以通过 ListView 提供的 `listview.setSelection(pos);`方法。

#### 16. ListView 中图片错位的问题是如何产生的?

图片错位问题的本质源于我们的 listview 使用了缓存 convertView，假设一种场景，一个 listview 一屏显示九个 item，那么在拉出第十个 item 的时候，事实上该 item 是重复使用了第一个 item，也就是说在第一个 item 从网络中 下载图片并最终要显示的时候，其实该 item 已经不在当前显示区域内了，此时显示的后果将可能在第十个 item 上输 出图像，这就导致了图片错位的问题。所以解决之道在于可见则显示，不可见则不显示。

#### 17. 如何在 ScrollView 中如何嵌入 ListView

在 ScrollView 添加一个 ListView 会导致 listview 控件显示不全，通常只会显示一条，这是因为两个控件的滚动事件冲突导致。所以需要通过 listview 中的 item 数量去计算 listview 的显示高度，从而使其完整展示，如下提供一个方法供参考。

```java
lv = (ListView) findViewById(R.id.lv);
adapter = new MyAdapter();
lv.setAdapter(adapter);
setListViewHeightBasedOnChildren(lv);
----------------------------------------------------
public void setListViewHeightBasedOnChildren(ListView listView) {
    ListAdapter listAdapter = listView.getAdapter();
    if (listAdapter == null) {
    	return;
    }
    int totalHeight = 0;
    for (int i = 0; i < listAdapter.getCount(); i++) {
        View listItem = listAdapter.getView(i, null, listView);
        listItem.measure(0, 0);
        totalHeight += listItem.getMeasuredHeight();
    }
    ViewGroup.LayoutParams params = listView.getLayoutParams();
    params.height = totalHeight + (listView.getDividerHeight() * (listAdapter.getCount() - 1));
    params.height += 5;// if without this statement,the listview will be a
    // little short
    listView.setLayoutParams(params);
}
```

> 注意:如果直接将 ListView 放到 ScrollView 中,那么上面的代码依然是没有效果的。必须将 ListVIew 放到
> LinearLayout 等其他容器中才行。

现阶段最好的处理的方式是： 自定义 ListView，重载 `onMeasure()`方法，设置全部显示

```java
public class ScrollViewWithListView extends ListView {
    public ScrollViewWithListView(android.content.Context context, android.util.AttributeSet attrs) {
        super(context, attrs);
    }
    /**
    * Integer.MAX_VALUE >> 2,如果不设置，系统默认设置是显示两条
    */
    public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}
```

#### 18. ListView的Adapter是什么Adapter

![img](https://user-gold-cdn.xitu.io/2019/3/20/1699a79b39f0c556?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- **BaseAdapter**：抽象类，实际开发中我们会继承这个类并且重写相关方法，用得最多的一个适配器。
- **ArrayAdapter**：支持泛型操作，最简单的一个适配器，只能展现一行文字。
- **SimpleAdapter**：同样具有良好扩展性的一个适配器，可以自定义多种效果。
- **SimpleCursorAdapter**：用于显示简单文本类型的listView，一般在数据库那里会用到，不过有点过时，不推荐使用。



## 十五、ViewPager

#### 1. ViewPager2原理

ViewPager2继承ViewGroup，内部核心是RecycleView加LinearLayoutManager，其实就是对RecycleView封装了一层。

#### 2. ViewPager中嵌套ViewPager怎么处理滑动冲突

```java
public class ChildViewPager extends ViewPager {
    public ChildViewPager(@NonNull Context context) {
    	super(context);
    }

    public ChildViewPager(@NonNull Context context, @Nullable AttributeSet attrs) {
    	super(context, attrs);
    }

	private float x1;

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //告知父控件 把事件下发给子控件处理
                getParent().requestDisallowInterceptTouchEvent(true);
                x1 = ev.getX();
                break;
            case MotionEvent.ACTION_MOVE:
                //拿到当前显示页下标
                int curPosition = getCurrentItem();
                //手指移动时的X坐标
                float x2 = ev.getX();
                if (curPosition == 0) {
                    if (Math.abs(x2 - x1) > 50) {
                    	//当当前页面在下标为0的时候或为最后一页时，由父亲拦截触摸事件
                    	getParent().requestDisallowInterceptTouchEvent(false);
                    } else {
                    	getParent().requestDisallowInterceptTouchEvent(true);
                    }
                } else {
                	//其他情况，由孩子拦截触摸事件
                	getParent().requestDisallowInterceptTouchEvent(true);
                }
            break;
        }
        return super.dispatchTouchEvent(ev);
    }
}
```

#### 3. Viewpager切换掉帧有什么处理经验？

应用UI卡顿常见原因主要在以下几个方面：

1. 在UI线程中做轻微耗时操作，导致UI线程卡顿；
2. 布局Layout过于复杂，无法在16ms内完成渲染；
3. 同一时间动画执行的次数过多，导致CPU或GPU负载过重；
4. View过度绘制，导致某些像素在同一帧时间内被绘制多次，从而使CPU或GPU负载过重；
5. View频繁的触发measure、layout，导致measure、layout累计耗时过多及整个View频繁的重新渲染；
6. 内存频繁触发GC过多（同一帧中频繁创建内存），导致暂时阻塞渲染操作；
7. 冗余资源及逻辑等导致加载和执行缓慢；

首先发现问题，通过GPU柱状图判断卡顿程度。然后通过TraceView定位卡顿的方法，打log方式找到更具体的耗时细节，然后逐个优化。

#### 4. 怎么写一个不能滑动的ViewPager?

```java
public class CustomViewPager extends ViewPager {

    private boolean isCanScroll = true;

    public CustomViewPager(Context context) {
        super(context);
    }

    public CustomViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    /**
     * 设置其是否能滑动换页
     * @param isCanScroll false 不能换页， true 可以滑动换页
     */
    public void setScanScroll(boolean isCanScroll) {
        this.isCanScroll = isCanScroll;
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return isCanScroll && super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        return isCanScroll && super.onTouchEvent(ev);

    }
}
```

#### 5. ViewPager使用细节，如何设置成每次只初始化当前的Fragment，其他的不初始化（提示：Fragment懒加载）？

Viewpager 默认加载 3 个。1 个 Activity 里面可能会以 Viewpager（或其他容器）与多个 Fragment 来组合使用， 而如果每个 fragment 都需要去加载数据，或从本地加载，或从网络加载，那么在这个 Activity 刚创建的时候就变成 需要初始化大量资源。所以我们要进行懒加载。

自定义一个 LazyLoadFragment 基类，利用 `setUserVisibleHint` 和 `生命周期方法`，通过对 Fragment 状态判断，进行数据加载，并将数据加载的接口提供开放出去，供子类使用。然后在子类 Fragment 中实现 requestData 方法即可。这里添加了一个 isDataLoaded 变量，目的是避免重复加载数据。考虑到有时候需要刷新数据的问题，便提供了一个用于强制刷新的参数判断。

```java
public abstract class LazyLoadFragment extends BaseFragment {
    protected boolean isViewInitiated;
    protected boolean isDataLoaded;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        isViewInitiated = true;
        prepareRequestData();
    }
    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        prepareRequestData();
    }
    public abstract void requestData();
    public boolean prepareRequestData() {
        return prepareRequestData(false);
    }
    public boolean prepareRequestData(boolean forceUpdate) {
        if (getUserVisibleHint() && isViewInitiated && (!isDataLoaded || forceUpdate)) {
            requestData();
            isDataLoaded = true;
            return true;
        }
        return false;
    }
}
```



## 十六、数据存储

#### 0. 描述一下Android数据持久存储方式？

- **SharedPreferences存储**：一种轻型的数据存储方式，本质是基于XML文件存储的key-value键值对数据，通常用来存储一些简单的配置信息（如应用程序的各种配置信息）；

  > - 勿存储大型复杂数据，这会引起内存GC、阻塞主线程使页面卡顿产生ANR
  > - 勿在多进程模式下，操作Sp
  > - 不要多次edit和apply，尽量批量修改一次提交
  > - 建议apply，少用commit

- **SQLite数据库存储**：一种轻量级嵌入式数据库引擎，它的运算速度非常快，占用资源很少，常用来存储大量复杂的关系数据；

- **File文件存储**：写入和读取文件的方法和 Java中实现I/O的程序一样；

- **网络存储**：主要在远程的服务器中存储相关数据，用户操作的相关数据可以同步到服务器上；

#### 1. SharedPreferences是如何保证线程安全的，其内部的实现用到了哪些锁

SharedPreferences的本质是用键值对的方式保存数据到xml文件，然后对文件进行读写操作。

- 对于读操作，加一把锁就够了：

```java
public String getString(String key, @Nullable String defValue) {
    synchronized (mLock) {
        String v = (String)mMap.get(key);
        return v != null ? v : defValue;
    }
}
```

- 对于写操作，由于是两步操作，一个是`editor.put`，一个是`commit`或者`apply`所以其实是需要两把锁的：

  `mEditorLock`和`mWritingToDiskLock`。

```java
//第一把锁，操作Editor类的map对象
public final class EditorImpl implements Editor {
  @Override
  public Editor putString(String key, String value) {
      synchronized (mEditorLock) {
          mEditorMap.put(key, value);
          return this;
      }
  }
}


//第二把锁，操作文件的写入
synchronized (mWritingToDiskLock) {
    writeToFile(mcr, isFromSyncCommit);
}
```

#### 2. SharedParence可以跨进程通信吗？如何改造成可以跨进程通信？

1） SharedPreferences是进程不安全的，因为没有使用跨进程的锁。既然是进程不安全，那么久有可能在多进程操作的时候发生数据异常。

2） 我们有两个办法能保证进程安全：

- 使用跨进程组件，也就是ContentProvider，这也是官方推荐的做法。通过ContentProvider对多进程进行了处理，使得不同进程都是通过ContentProvider访问SharedPreferences。
- 加文件锁，由于SharedPreferences的本质是读写文件，所以我们对文件加锁，就能保证进程安全了。

#### 3. SharedPreferences 操作有文件备份吗？是怎么完成备份的？

- SharedPreferences 的写入操作，首先是将源文件备份：

```java
  if (!backupFileExists) {
      !mFile.renameTo(mBackupFile);
  }
```

- 再写入所有数据，只有写入成功，并且通过 `sync` 完成落盘后，才会将 Backup（.bak） 文件删除。
- 如果写入过程中进程被杀，或者关机等非正常情况发生。进程再次启动后如果发现该 SharedPreferences 存在 Backup 文件，就将 Backup 文件重名为源文件，原本未完成写入的文件就直接丢弃，这样就能保证之前数据的正确。

#### 4.SharedPreference原理？读取xml是在哪个线程?

（1）`getSharedPreferences()`在创建一个SharedPreferences，会先判断是否有对应的xml文件（SharedPreferences存储数据的保存格式），如果存在则会有一个预加载操作，这个操作将把xml文件的内容通过I/O操作和xmlUtil解析后保存在一个map对象中。如果不存在则会创建一个对应的xml。

（2）而在对数据进行读取时，是从内存中该map对象中进行读取；

（3）在使用SharedPreferences保存数据时，主要分为以下两步：把数据先写入内存，写到map集合中、将数据写到硬盘文件保持一致性；由此可以得出，数据在保存的时候，是以key-value的格式保存在xml文件中。

（4）写完数据后，要对写入的数据进行提交保存，主要有以下两种方式：

​		a. `commit()`：线程安全，性能慢，一般在当前线程完成文件操作，会有返回值；

​		b. `apply()`：线程不安全，性能高，异步处理I/O操作，一般在singleThreadExecutor中执行，没有返回值

第一次读的时候，主线程会挂起wait，等到**整个文件load完**毕，才被唤醒。

**整个文件load的实现**：开个线程，从磁盘中解析xml到内存，如果文件比较大那么这个会耗时，那么主线程就会等待比较久。

#### 5. SharedPrefrences的apply和commit有什么区别？

- apply没有返回值，而commit返回boolean，表明修改是否提交成功。

- apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。 

- apply方法不会提示任何失败的提示。 由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。

#### 6. sp 频繁操作有什么后果？sp 能存多少数据？

Sp 的底层是由 xml 来实现的，操作 sp 的过程就是 xml 的序列化和解析的过程。Xml 是存储在磁盘上的，因此考 虑到需要 I/O 速度问题，sp 不适宜频繁操作。同时序列化 xml 是就是将内存中的数据写到 xml 文件中，由于 dvm 的内存是很有限的，因此单个 sp 文件不建议太大，具体多大是没有一个具体的要求的，数据大小肯定不能超过DVM 堆内存。其实 sp 设置的目的就是为了保存用户的偏好和配置信息的，因此不要保存太多的数据。

#### 7. 了解SQLite中的事务操作吗？是如何做的?

SQLite在做CRDU操作时都默认开启了事务，然后把SQL语句翻译成对应的`SQLiteStatement`并调用其相应的CRUD方法，此时整个操作还是在`rollback journal`这个临时文件上进行，只有操作顺利完成才会更新db数据库，否则会被回滚。

#### 8. 使用SQLite做批量操作有什么好的方法吗？

使用SQLiteDatabase的`beginTransaction()`方法开启一个事务，将批量操作SQL语句转化为`SQLiteStatement`并进行批量操作，结束后`endTransaction()`。

> 在Android SQLite里，对所有的写入操作（insert、update）等，都会在底层默默创建一个Transaction来完成。如果上层已经创建Transaction了），底层则不会再次创建。

#### 9. 如何删除SQLite中表的个别字段

SQLite数据库只允许增加字段而不允许修改和删除表字段，只能创建新表保留原有字段，删除原表。

#### 10. 使用SQLite时会有哪些优化操作？

- 使用事务做批量操作
- 及时关闭Cursor，避免内存泄露
- 耗时操作异步化：数据库的操作属于本地IO耗时操作，建议放入异步线程中处理
- ContentValues的容量调整：ContentValues内部采用HashMap来存储Key-Value数据，ContentValues初始容量为8，扩容时翻倍。因此建议对ContentValues填入的内容进行估量，设置合理的初始化容量，减少不必要的内部扩容操作
- 使用索引加快检索速度：对于查询操作量级较大、业务对查询要求较高的推荐使用索引
- ContentValues复用，Clear后再赋值

#### 16. SQLite如何修改表?

数据库升级增加表和删除表都不涉及数据迁移，但是修改表涉及到对原有数据进行迁移。

升级的方法如下所示：

- 将现有表命名为临时表。
- 创建新表。
- 将临时表的数据导入新表。
- 删除临时表。

如果是跨版本数据库升级，可以有两种方式：

- 逐级升级，确定相邻版本与现在版本的差别，V1升级到V2,V2升级到V3，依次类推。
- 跨级升级，确定每个版本与现在数据库的差别，为每个case编写专门升级大代码。

## 十七、其他控件

#### 1. 请问 Gridview 能添加头布局吗？

GridView 本身没有添加头布局的方法 api 可以使用 ScrollView 与 GridView 结合，让 GridView 充满 ScrollView，不让 GridView 滑动而只让 ScrollView 滑动；具体做法是重载 GridView 的 onMeasure()方法。示例如下：

```java
@Override
public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
	super.onMeasure(widthMeasureSpec, expandSpec);
}
```

#### 2. ConstraintLayout实现三等分,ConstraintLayout动画



#### 3. CoordinatorLayout自定义behavior,可以拦截什么？



#### 4. 一个wrap_content的ImageView，加载远程图片，传什么参数裁剪比较好?

```xml
		<ImageView
            android:id="@+id/img_1"
            android:layout_width="math_parent"
            android:layout_height="wrap_content"
            android:adjustViewBounds="true"
            android:scaleType="centerInside"
            android:src="@drawable/img_bg" />
```

adjustViewBounds影响的是ImageView的比例（不是图片的比例），所以，对于adjustViewBounds的定义应该是这样：调整ImageView的边界，使得ImageView和图片有一样的长宽比例。

#### 5. SurfaceView的理解？

它是什么？他的继承方式是什么？与 View 的区别(从源码角度，如加载，绘制等)。

SurfaceView 中采用了双缓冲机制，保证了 UI 界面的流畅性，同时 SurfaceView不在主线程中绘制，而是另开辟一个线程去绘制，所以它不妨碍 UI 线程；

SurfaceView 继承于 View，他和 View 主要有以下三点区别：

- View 底层没有双缓冲机制，SurfaceView 有；
- View 主要适用于主动更新，而 SurfaceView 适用与被动的更新，如频繁的刷新
- View 会在主线程中去更新 UI，而 SurfaceView 则在子线程中刷新；

SurfaceView 的内容不在应用窗口上，所以不能使用变换（平移、缩放、旋转等）。也难以放在 ListView 或者 ScrollView 中，不能使用 UI 控件的一些特性比如View.setAlpha()。

View：显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等；必须在 UI 主线程内更新画面，速度较慢。

SurfaceView：基于 view 视图进行拓展的视图类，更适合 2D 游戏的开发；是 View 的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比 View 快，Camera 预览界面使用 SurfaceView。

GLSurfaceView：基于 SurfaceView 视图再次进行拓展的视图类，专用于 3D 游戏开发的视图；是 SurfaceView 的子类，openGL 专用。 

#### 6. LinearLayout、FrameLayout、RelativeLayout性能对比，为什么？

- RelativeLayout会让子View调用2次onMeasure，LinearLayout 在有weight时，也会调用子 View 2次onMeasure

- RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin。

- 在不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout。

#### 7. 为什么Google给开发者默认新建了个RelativeLayout，而自己却在DecorView中用了个LinearLayout？

因为DecorView的层级深度是已知而且固定的，上面一个标题栏，下面一个内容栏。采用RelativeLayout并不会降低层级深度，所以此时在根节点上用LinearLayout是效率最高的。而之所以给开发者默认新建了个RelativeLayout是希望开发者能采用尽量少的View层级来表达布局以实现性能最优，因为复杂的View嵌套对性能的影响会更大一些。

#### 8. 请例举Android中常用布局类型，并简述其用法以及排版效率

Android中常用布局分为**传统布局**和**新型布局**

传统布局（编写XML代码、代码生成）：

- **框架布局（FrameLayout）**
- **线性布局（LinearLayout）**
- **绝对布局（AbsoluteLayout）**
- **相对布局（RelativeLayout）**
- **表格布局（TableLayout）**

新型布局（可视化拖拽控件、编写XML代码、代码生成）：

- **约束布局（ConstrainLayout）**

![img](https://user-gold-cdn.xitu.io/2019/4/18/16a2f8e1327c53b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 9. Merge、ViewStub 的作用?

Merge: 减少视图层级，可以删除多余的层级。和Include标签配套使用

ViewStub: 按需加载，减少内存使用量、加快渲染速度、不支持 merge 标签。



## 十八、其他

#### 1. 描述一下 Android 的系统架构

![img](https://pic1.zhimg.com/80/v2-24518bd357cd59ec75b5cf96db1457d8_720w.jpg)

Android 是一种基于 Linux 的开放源代码软件栈，为广泛的设备和机型而创建。下图所示为 Android 平台的五大组件：

**应用程序**

Android 系统包括一套用于电子邮件、短信、日历、互联网浏览和联系人等的核心应用。与用户可以选择安装的应用一样，没有特殊状态。因此第三方应用可成为用户的默认网络浏览器、短信 Messenger 甚至默认键盘（有一些例外，例如系统的“设置”应用）。

系统应用可用作用户的应用，以及提供开发者可从其自己的应用访问的主要功能。

**Java API Framework**

可通过以 Java 语言编写的 API 使用 Android OS 的整个功能集。这些 API 形成创建 Android 应用所需的构建块，它们可简化核心模块化系统组件和服务的重复使用，包括以下组件和服务：

- 丰富、可扩展的视图系统，可用以构建应用的 UI，包括列表、网格、文本框、按钮甚至可嵌入的网络浏览器
- 资源管理器，用于访问非代码资源，例如本地化的字符串、图形和布局文件
- 通知管理器，可让所有应用在状态栏中显示自定义提醒
- Activity 管理器，用于管理应用的生命周期，提供常见的导航返回栈
- 内容提供程序，可让应用访问其他应用（例如“联系人”应用）中的数据或者共享其自己的数据

开发者可以完全访问 Android 系统应用使用的框架 API。

**系统运行库**

1) 原生 C/C++ 库

许多核心 Android 系统组件和服务（例如 ART 和 HAL）构建自原生代码，需要以 C 和 C++ 编写的原生库。Android 平台提供 Java 框架 API 以向应用显示其中部分原生库的功能。例如，您可以通过 Android 框架的 Java OpenGL API 访问 OpenGL ES，以支持在应用中绘制和操作 2D 和 3D 图形。如果开发的是需要 C 或 C++ 代码的应用，可以使用 Android NDK 直接从原生代码访问某些原生平台库。

2) Android Runtime

对于运行 Android 5.0（API 级别 21）或更高版本的设备，每个应用都在其自己的进程中运行，并且有其自己的 Android Runtime (ART) 实例。ART 编写为通过执行 DEX 文件在低内存设备上运行多个虚拟机，DEX 文件是一种专为 Android 设计的字节码格式，经过优化，使用的内存很少。编译工具链（例如 Jack）将 Java 源代码编译为 DEX 字节码，使其可在 Android 平台上运行。

ART 的部分主要功能包括：

- 预先 (AOT) 和即时 (JIT) 编译
- 优化的垃圾回收 (GC)
- 更好的调试支持，包括专用采样分析器、详细的诊断异常和崩溃报告，并且能够设置监视点以监控特定字段

在 Android 版本 5.0（API 级别 21）之前，Dalvik 是 Android Runtime。

Android 还包含一套**核心运行时库**，可提供 Java API 框架使用的 Java 编程语言大部分功能，包括一些 Java 8 语言功能。

**硬件抽象层 (HAL)**

硬件抽象层 (HAL) 提供标准界面，向更高级别的 Java API 框架显示设备硬件功能。HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个界面，例如相机或蓝牙模块。当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块。

**Linux 内核**

Android 平台的基础是 Linux 内核。例如，Android Runtime (ART) 依靠 Linux 内核来执行底层功能，例如线程和低层内存管理。使用 Linux 内核可让 Android 利用主要安全功能，并且允许设备制造商为著名的内核开发硬件驱动程序。

对于Android应用开发来说，最好能手绘下面的系统架构图：

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095ea677939c5b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 2. 谈一谈 Android 的安全机制 

- Android 是基于 Linux 内核的，因此 Linux 对文件权限的控制同样适用于 Android，在 Android 中每个应用都有自己的`/data/data/包名`文件夹，该文件夹只能该应用访问，而其他应用则无权访问。
- Android 的权限机制保护了用户的合法权益 如果我们的代码想拨打电话、发送短信、访问通信录、定位、访问 sdcard 等所有可能侵犯用于权益的行为都是必须要在 `AndroidManifest.xml` 中进行声明的，这样就给了用户一个知情权。 
- Android 的代码混淆保护了开发者的劳动成果
- Android 的签名机制

#### 3. Android 的四大组件都需要在清单文件中注册吗？

Activity 、 Service 、 ContentProvider 如 果 要 使 用 则 必 须 在 AndroidManifest.xml 中 进 行 注 册 ， 而 BroadcastReceiver 则有两种注册方式，静态注册和动态注册。其中静态注册就是指在 AndroidManifest.xml 中进行

#### 4. 平常抓包用什么工具？

Fiddler，Wireshark

#### 5. Bundle是什么数据结构? 利用什么传递数据

Bundle主要用于传递数据；它保存的数据，内部其实就是维护了一个Map<String,Object>。

我们经常使用Bundle在Activity之间传递数据，传递的数据可以是boolean、byte、int、long、float、double、string等基本类型或它们对应的数组，也可以是对象或对象数组。当Bundle传递的是对象或对象数组时，必须实现Serializable 或Parcelable接口。下面分别介绍Activity之间如何传递基本类型、传递对象。

#### 6. 如何判断是否有 SD 卡？ 

通过如下方法： 

`Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)` 如果返回 true 就是有 sdcard，如果返回 false 则没有。

#### 7. Bundle传递对象为什么需要序列化？Serialzable和Parcelable的区别？

因为bundle传递数据时只支持基本数据类型，所以在传递对象时需要序列化转换成可存储或可传输的本质状态（字节流）。序列化后的对象可以在网络、IPC（比如启动另一个进程的Activity、Service和Reciver）之间进行传输，也可以存储到本地。

Serializable（Java自带）：

Serializable 是序列化的意思，表示将一个对象转换成存储或可传输的状态。序列化后的对象可以在网络上进传输，也可以存储到本地。

Parcelable（android专用）：

除了Serializable之外，使用Parcelable也可以实现相同的效果，不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这也就实现传递对象的功能了。

**区别总结如下所示**：

**平台区别**：

Serializable是属于 Java 自带的，表示一个对象可以转换成可存储或者可传输的状态，序列化后的对象可以在网络上进行传输，也可以存储到本地。

Parcelable 是属于 Android 专用。不过不同于Serializable，Parcelable实现的原理是将一个完整的对象进行分解。而分解后的每一部分都是Intent所支持的数据类型。

**编写上的区别**：

Serializable代码量少，写起来方便

Parcelable代码多一些，略复杂

**选择的原则**：

如果是仅仅在内存中使用，比如activity、service之间进行对象的传递，强烈推荐使用Parcelable，因为Parcelable比Serializable性能高很多。因为Serializable在序列化的时候会产生大量的临时变量， 从而引起频繁的GC。

如果是持久化操作，推荐Serializable，虽然Serializable效率比较低，但是还是要选择它，因为在外界有变化的情况下，Parcelable不能很好的保存数据的持续性。

**本质的区别**：

Serializable的本质是使用了反射，序列化的过程比较慢，这种机制在序列化的时候会创建很多临时的对象，比引起频繁的GC

Parcelable方式的本质是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的类型，这样就实现了传递对象的功能了。

#### 8. Android5.0~10.0各版本新特性

Android5.0新特性

- **MaterialDesign设计风格**
- **支持64位ART虚拟机**（5.0推出的ART虚拟机，在5.0之前都是Dalvik。他们的区别是：
  Dalvik,每次运行,字节码都需要通过即时编译器转换成机器码(JIT)。
  ART,第一次安装应用的时候,字节码就会预先编译成机器码(AOT)）

- 通知详情可以用户自己设计

Android6.0新特性

- **动态权限管理**

- 支持快速充电的切换
- 支持文件夹拖拽应用
- 相机新增专业模式

Android7.0新特性

- **多窗口支持**
- **V2签名**

- 增强的Java8语言模式
- 夜间模式

Android8.0（O）新特性

- **优化通知**


    通知渠道 (Notification Channel)
    通知标志
    休眠
    通知超时
    通知设置
    通知清除   

- **画中画模式**：清单中Activity设置android:supportsPictureInPicture
- **后台限制**

- 自动填充框架
- 系统优化
- 等等优化很多

Android9.0（P）新特性

- **室内WIFI定位**
- **“刘海”屏幕支持**

- 安全增强
- 等等优化很多

Android10.0（Q）新特性

- **夜间模式**：包括手机上的所有应用都可以为其设置暗黑模式。
- **桌面模式**：提供类似于PC的体验，但是远远不能代替PC。
- **屏幕录制**：通过长按“电源”菜单中的"屏幕快照"来开启。

推荐文章：[Android Developers 官方文档](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels)

#### 9. android中有哪几种解析xml的类，官方推荐哪种？以及它们的原理和区别？

**DOM解析**

优点:

1.XML树在内存中完整存储,因此可以直接修改其数据结构.

2.可以通过该解析器随时访问XML树中的任何一个节点.

3.DOM解析器的API在使用上也相对比较简单.

缺点:

如果XML文档体积比较大时,将文档读入内存是非消耗系统资源的.

使用场景:

- DOM 是与平台和语言无关的方式表示 XML文档的官方 W3C 标准.
- DOM 是以层次结构组织的节点的集合.这个层次结构允许开人员在树中寻找特定信息.分析该结构通常需要加载整个文档和构造层次结构,然后才能进行任何工作.
- DOM 是基于对象层次结构的.

**SAX解析**

优点：SAX 对内存的要求比较低,因为它让开发人员自己来决定所要处理的标签.特别是当开发人员只需要处理文档中包含的部分数据时,SAX 这种扩展能力得到了更好的体现。

缺点：用SAX方式进行XML解析时,需要顺序执行,所以很难访问同一文档中的不同数据.此外,在基于该方式的解析编码程序也相对复杂。

工作原理：对文档进行顺序扫描，当扫描到文档(document)开始与结束、元素(element)开始与结束、文档 (document)结束等地方时通知事件处理函数，由事件处理函数做相应动作，然后继续同样的扫描，直至文档结束。

使用场景：对于含有数据量十分巨大,而又不用对文档的所有数据行遍历或者分析的时候,使用该方法十分有效.该方法不将整个文档读入内存,而只需读取到程序所需的文档标记处即可。

**Pull解析**

PULL解析器的运行方式和SAX类似，都是基于事件的模式。不同的是，在PULL解析过程中返回的是数字，且我们需要自己获取产生的事件然后做相应的操作，而不像SAX那样由处理器触发一种事件的方法，执行我们的代码。

解析过程：XML pull提供了开始元素和结束元素。当某个元素开始时，我们可以调用parser．nextText从XML文档中提取所有字符数据。当解释到一个文档结束时，自动生成EndDocument事件。

优点：PULL解析器小巧轻便，解析速度快，简单易用，非常适合在Android移动设备中使用，Android系统内部在解析各种XML时也是用PULL解析器，**Android官方推荐开发者们使用Pull解析技术**。Pull解析技术是第三方开发的开源技术，它同样可以应用于Java开发。

#### 10. Jar和Aar的区别?

Jar包里面只有代码，aar里面不光有代码还包括资源文件，比如 drawable 文件，xml资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度。

#### 11. Android为每个应用程序分配的内存大小是多少?

android程序内存一般限制在16M，也有的是24M。近几年手机发展较快，一般都会分配两百兆左右，和具体机型有关。



#### 13. Asset目录与res目录的区别？

assets：不会在 R 文件中生成相应标记，存放到这里的资源在打包时会打包到程序安装包中。（通过 AssetManager 类访问这些文件）

res：会在 R 文件中生成 id 标记，资源在打包时如果使用到则打包到安装包中，未用到不会打入安装包中。

res/anim：存放动画资源。

res/raw：和 asset 下文件一样，打包时直接打入程序安装包中（会映射到 R 文件中）。

#### 14. 通过google提供的Gson解析json时，定义JavaBean的规则是什么？

1) 实现序列化 Serializable

2) 属性私有化，并提供get，set方法

3) 提供无参构造

4) 属性名必须与json串中属性名保持一致 （因为Gson解析json串底层用到了Java的反射原理）

#### 15. json解析方式的区别？

1) SDK提供JSONArray，JSONObject

2) google提供的 Gson

通过fromJson()实现对象的反序列化（即将json串转换为对象类型）

通过toJson()实现对象的序列化 （即将对象类型转换为json串）

3) 阿里的fastjson

#### 16. 编译期注解和运行时注解

运行期注解(RunTime)利用反射去获取信息还是比较损耗性能的，对应`@Retention（RetentionPolicy.RUNTIME）`。

编译期(Compile time)注解，以及处理编译期注解的手段APT和Javapoet，对应`@Retention(RetentionPolicy.CLASS)`。

其中apt+javaPoet目前也是应用比较广泛，在一些大的开源库，如EventBus3.0+,页面路由 ARout、Dagger、Retrofit等均有使用的身影，注解不仅仅是通过反射一种方式来使用，也可以使用APT在编译期处理

#### 17. 强引用置为null，会不会被回收？

不会立即释放对象占用的内存。 如果对象的引用被置为null，只是断开了当前线程栈帧中对该对象的引用关系，而 垃圾收集器是运行在后台的线程，只有当用户线程运行到安全点(safe point)或者安全区域才会扫描对象引用关系，扫描到对象没有被引用则会标记对象，这时候仍然不会立即释放该对象内存，因为有些对象是可恢复的（在 finalize方法中恢复引用 ）。只有确定了对象无法恢复引用的时候才会清除对象内存。

#### 18. 是否了解硬件加速？

硬件加速就是运用GPU优秀的运算能力来加快渲染的速度，而通常的基于软件的绘制渲染模式是完全利用CPU来完成渲染。

1. 硬件加速是从API 11引入，API 14之后才默认开启。对于标准的绘制操作和控件都是支持的，但是对于自定义View的时候或者一些特殊的绘制函数就需要考虑是否需要关闭硬件加速。

2. 我们面对不支持硬件加速的情况，就需要限制硬件加速，这个兼容性的问题是因为硬件加速是把View的绘制函数转化为使用OpenGL的函数来进完成实际的绘制的，那么必然会存在OpenGL中不支持原始回执函数的情况，对于这些绘制函数，就会失效。

3. 硬件加速的消耗问题，因为是使用OpenGL，需要把系统中OpenGL加载到内存中，OpenGL API调用就会占用8MB，而实际上会占用更多内存，并且使用了硬件必然增加耗电量了。

4. 硬件加速的优势还有`display list`的设计，使用这个我们不需要每次重绘都执行大量的代码，基于软件的绘制模式会重绘脏区域内的所有控件，而display只会更新列表，然后绘制列表内的控件。
5. CPU更擅长复杂逻辑控制，而GPU得益于大量ALU和并行结构设计，更擅长数学运算。

#### 19. 对于应用更新这块是如何做的？(灰度，强制更新，增量更新)

**内部更新**：

- 通过接口获取线上版本号，versionCode
- 比较线上的versionCode 和本地的versionCode，弹出更新窗口
- 下载APK文件（文件下载）
- 安装APK

**强制更新**：

一般的处理就是进入应用就弹窗通知用户有版本更新，弹窗可以没有取消按钮并不能取消。这样用户就只能选择更新或者关闭应用了，当然也可以添加取消按钮，但是如果用户选择取消则直接退出应用。

**增量更新**：

bsdiff：二进制差分工具bsdiff是相应的补丁合成工具,根据两个不同版本的二进制文件，生成补丁文件.patch文件。通过bspatch使旧的apk文件与不定文件合成新的apk。 注意通过apk文件的md5值进行区分版本。

**灰度更新**：

(1) 找单一渠道投放特别版本。

(2) 做升级平台的改造，允许针对部分用户推送升级通知甚至版本强制升级。

(3) 开放单独的下载入口。

(4) 是两个版本的代码都打到app包里，然后在app端植入测试框架，用来控制显示哪个版本。测试框架负责与服务器端api通信，由服务器端控制app上A/B版本的分布，可以实现指定的一组用户看到A版本，其它用户看到B版本。服务端会有相应的报表来显示A/B版本的数量和效果对比。最后可以由服务端的后台来控制，全部用户在线切换到A或者B版本~

无论哪种方法都需要做好版本管理工作，分配特别的版本号以示区别。当然，既然是做灰度，数据监控（常规数据、新特性数据、主要业务数据）还是要做到位，该打的数据桩要打。还有，灰度版最好有收回的能力，一般就是强制升级下一个正式版。

#### 20. 请解释安卓为啥要加签名机制。

1. 开发者的身份认证，由于开发商可能通过使用相同的 Package Name 来混淆替换已经安装的程序，以此保证签名不同的包不被替换。
2. 保证信息传输的完整性，签名对于包中的每个文件进行处理，以此确保包中内容不被替换。
3. 防止交易中的抵赖发生， Market 对软件的要求。

#### 21. 如何通过Gradle配置多渠道包？

首先要了解设置多渠道的原因。在安装包中添加不同的标识，配合自动化埋点，应用在请求网络的时候携带渠道信息，方便后台做运营统计，比如说统计我们的应用在不同应用市场的下载量等信息。

这里以友盟统计为例

- 首先在manifest.xml文件中设置动态渠道变量：

  ![img](https://user-gold-cdn.xitu.io/2019/3/28/169c30d0cdbfb111?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 接着在app目录下的`build.gradle`中配置`productFlavors`，也就是配置打包的渠道：

  ![img](https://user-gold-cdn.xitu.io/2019/3/28/169c31116ef61848?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 最后在编辑器下方的Teminal输出命令行：

执行`./gradlew assembleRelease` ，将会打出所有渠道的release包；

执行`./gradlew assembleVIVO`，将会打出VIVO渠道的release和debug版的包；

执行`./gradlew assembleVIVORelease`将生成VIVO渠道的release包。

因此，可以结合buildType和productFlavor生成不同的Build Variants，即类型与渠道不同的组合。

推荐文章：[美团Android自动化之旅—Walle生成渠道包](https://github.com/Meituan-Dianping/walle)

#### 22. DDMS 和 TraceView 的区别？

DDMS 原意是：davik debug monitor service。简单的说 ddms 是一个程序执行查看器，在里面可以看见线程和堆栈等信息，traceView 是程序性能分析器。traceview 是 ddms 中的一部分内容。

Traceview 是 Android 平台特有的数据采集和分析工具，它主要用于分析 Android 中应用程序的 hotspot（瓶颈）。Traceview 本身只是一个数据分析工具，而数据的采集则需要使用 Android SDK 中的 Debug 类或者利用DDMS 工具。二者的用法如下：开发者在一些关键代码段开始前调用 Android SDK 中 Debug 类的 startMethodTracing 函数，并在关键代码段结束前调用 stopMethodTracing 函数。这两个函数运行过程中将采集运行时间内该应用所有线程（注意，只能是 Java线程） 的函数执行情况， 并将采集数据保存到/mnt/sdcard/下的一个文件中。 开发者然后需要利用 SDK 中的 Traceview工具来分析这些数据。

#### 23. AndroidManifest.xml 中的 targerSDK 设置有什么作用？

用于指定 Android 应用中所需要使用的 SDK 的版本，比如我们的应用必须运行于 Android 4.1 以上版本的系统 SDK 之上，那么就需要指定应用支持最小的 SDK 版本数为 16；当然，每个 SDK 版本都会有指定的整数值与之对应，比如我们最常用的 Android 2.3 的版本数是 11。当然，除了可以指定最低版本之外，标签还可以指定最高版本和目标版本，语法范例如下。

```
 < android:targetSdkVersion="integer" 
android:maxSdkVersion="integer" />
```

#### 24. 简单描述下 Android 数字签名？签名机制？

Android的签名机制包含有**消息摘要**、**数字签名**和**数字证书**

- **消息摘要**：在消息数据上，执行一个单向的 Hash 函数，生成一个固定长度的Hash值
- **数字签名**：一种以电子形式存储消息签名的方法，一个完整的数字签名方案应该由两部分组成：**签名算法和验证算法**
- **数字证书**：一个经证书授权（Certificate Authentication）中心数字签名的包含公钥拥有者信息以及公钥的文件

在 Android 系统中，所有安装到系统的应用程序都必有一个数字证书，此数字证书用于标识应用程序的作者和在 应用程序之间建立信任关系。 

Android 系统要求每一个安装进系统的应用程序都是经过数字证书签名的，数字证书的私钥则保存在程序开发者 的手中。Android 将数字证书用来标识应用程序的作者和在应用程序之间建立信任关系，不是用来决定最终用户可以 安装哪些应用程序。 

这个数字证书并不需要权威的数字证书签名机构认证(CA)，它只是用来让应用程序包自我认证的。 

同一个开发者的多个程序尽可能使用同一个数字证书，这可以带来以下好处。 

(1)有利于程序升级，当新版程序和旧版程序的数字证书相同时，Android 系统才会认为这两个程序是同一个程序 的不同版本。如果新版程序和旧版程序的数字证书不相同，则 Android 系统认为他们是不同的程序，并产生冲突，会 要求新程序更改包名。 

(2)有利于程序的模块化设计和开发。Android 系统允许拥有同一个数字签名的程序运行在一个进程中，Android 程序会将他们视为同一个程序。所以开发者可以将自己的程序分模块开发，而用户只需要在需要的时候下载适当的模 块。 

在签名时，需要考虑数字证书的有效期：

(1)数字证书的有效期要包含程序的预计生命周期，一旦数字证书失效，持有改数字证书的程序将不能正常升级。

(2)如果多个程序使用同一个数字证书，则该数字证书的有效期要包含所有程序的预计生命周期。 

(3)Android Market 强制要求所有应用程序数字证书的有效期要持续到 2033 年 10 月 22 日以后。 

Android 数字证书包含以下几个要点： 

(1)所有的应用程序都必须有数字证书，Android 系统不会安装一个没有数字证书的应用程序 

(2)Android 程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证 

(3)如果要正式发布一个 Android ，必须使用一个合适的私钥生成的数字证书来给程序签名，而不能使用 adt 插

件或者 ant 工具生成的调试证书来发布。 

(4)数字证书都是有有效期的，Android 只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装 在系统中，即使证书过期也不会影响程序的正常功能。

#### 25. 如何导入外部数据库?

把原数据库包括在项目源码的 res/raw。

android系统下数据库应该存放在 /data/data/com.（package name）/ 目录下，所以我们需要做的是把已有的数据库传入那个目录下。操作方法是用FileInputStream读取原数据库，再用FileOutputStream把读取到的东西写入到那个目录。

#### 27. 说下冷启动与热启动是什么，区别，如何优化，使用场景等。

app 冷启动： 当应用启动时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用， 这个启动方式就叫做冷启动（后台不存在该应用进程）。冷启动因为系统会重新创建一个新的进程分配给它，所以会先创建和初始化 Application 类，再创建和初始化 MainActivity 类（包括一系列的测量、布局、 绘制），最后显示在界面上。 

app 热启动： 当应用已经被打开， 但是被按下返回键、Home 键等按键时回到 桌面或者是其他程序的时候，再重新打开该 app 时， 这个方式叫做热启动（后台已经存在该应用进程）。热启动因为会从已有的进程中来启动，所以热启动就 不会走 Application 这步了，而是直接走 MainActivity（包括一系列的测量、布局、 绘制），所以热启动的过程只需要创建和初始化一个 MainActivity 就行了，而不必创建和初始化 Application

冷启动的流程：

当点击 app 的启动图标时，安卓系统会从 Zygote 进程中 fork 创建出一个新的进程分配给该应用，之后会依次创建和初始化 Application 类、创建 MainActivity 类、加载主题样式 Theme 中的 windowBackground 等属性设置给 MainActivity 以及配置 Activity 层级上的一些属性、再 inflate 布局、当 onCreate/onStart/onResume 方法都走完了后最后才进行 contentView 的 measure/layout/draw 显示在界面上

冷启动的生命周期简要流程： 

Application 构造方法 –> attachBaseContext()–>onCreate –>Activity 构造方法 –> onCreate() –> 配置主体中的背景等操作 –>onStart() –> onResume() –> 测量、布 局、绘制显示

冷启动的优化主要是视觉上的优化，解决白屏问题，提高用户体验，所以通过上面冷启动的过程。能做的优化如下：

- 减少 onCreate()方法的工作量
- 不要让 Application 参与业务的操作
- 不要在 Application 进行耗时操作
- 不要以静态变量的方式在 Application 保存数据
- 减少布局的复杂度和层级
- 减少主线程耗时

#### 28. 为什么冷启动会有白屏黑屏问题？

原因在于加载主题样式 Theme 中的 windowBackground 等属性设置给 MainActivity 发生在 inflate 布局当 onCreate/onStart/onResume 方法之前，而 windowBackground 背景被设置成了白色或者黑色，所以我们进入 app 的第一个 界面的时候会造成先白屏或黑屏一下再进入界面。

解决思路如下

- 给他设置 windowBackground 背景跟启动页的背景相同，如果你的启动页是张图片那么可以直接给 windowBackground 这个属性设置该图片那么就不会有一闪的效果了

- 设置背景是透明的，给人一种延迟启动的感觉。将背景颜色设置为透明色，这样当用户点击桌面 APP 图片的时候，并不会"立即"进入 APP，而且在桌面上停留一会，其实这时候 APP 已经是启动的了，只是我们心 机的把 Theme 里的 windowBackground 的颜色设置成透明的，强行把锅甩给了手机应用厂商
- 将 Application 中的不必要的初始化动作实现懒加载，比如，在SpashActivity 显示后再发送消息到 Application，去初始化，这样可以将初始化的动作放在后边，缩短应用启动到用户看到界面的时间

#### 29. 怎样防范 APP 被反编译 

（1）加壳保护：就是在程序的外面再包裹上另外一段代码，保护里面的代 码不被非法修改或反编译，在程序运行的时候优先取得程序的控制权做一些 我们自己想做的工作。 

（2）dex 文件格式 ：apk 生成后所有的 java 生成的 class 文件都被 dx 命令整合成了一个 classes.dex 文件，当 apk 运行时 dalvik 虚拟机加载 classes.dex 文件并且用 dexopt 命令进行进一步的优化成 odex 文件。在这 个过程中修改 dalvik 指令来达到我们的目的。

#### 30. Oom 是否可以try catch ？

只有在一种情况下，这样做是可行的：

在try语句中声明了很大的对象，导致OOM，并且可以确认OOM是由try语句中的对象声明导致的，那么在catch语句中，可以释放掉这些对象，解决OOM的问题，继续执行剩余语句。

但是这通常不是合适的做法。

Java中管理内存除了显式地catch OOM之外还有更多有效的方法：比如SoftReference, WeakReference, 硬盘缓存等。 在JVM用光内存之前，会多次触发GC，这些GC会降低程序运行的效率。 如果OOM的原因不是try语句中的对象（比如内存泄漏），那么在catch语句中会继续抛出OOM。

#### 31. 系统为什么不建议在子线程中访问UI？

这是因为 Android 的UI控件不是线程安全的，如果在多线程中并发访问可能会导致UI控件处于不可预期的状态，

那么为什么系统不对UI控件的访问加上锁机制呢？缺点有两个：

1. 首先加上锁机制会让UI访问的逻辑变得复杂
2. 锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行。

所以最简单且高效的方法就是采用单线程模型来处理UI操作。
