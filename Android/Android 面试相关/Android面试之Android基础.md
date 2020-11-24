# Android基础

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 一、Activity

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

如果设置 android:configChanges="orientation|keyboardHidden|screenSize">，此时Activity的生命周期不会重走一遍，Activity不会重建，只会回调onConfigurationChanged方法。

#### 2. Activity的四大启动模式，以及应用场景？

`Activity`的四大启动模式：

- `standard`：标准模式，每次都会在活动栈中生成一个新的`Activity`实例。通常我们使用的活动都是标准模式。
- `singleTop`：栈顶复用，如果`Activity`实例已经存在栈顶，那么就不会在活动栈中创建新的实例，并回调onNewIntent方法。比较常见的场景就是给通知跳转的`Activity`设置，因为你肯定不想前台`Activity`已经是该`Activity`的情况下，点击通知，又给你再创建一个同样的`Activity`。
- `singleTask`：栈内复用，如果`Activity`实例在当前栈中已经存在，就会将当前`Activity`实例上面的其他`Activity`实例都移除栈，并回调onNewIntent方法。常见于跳转到主界面。
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

#### 8. Activity依次A→B→C→B，其中B启动模式为singleTask，AC都为standard，生命周期分别怎么调用？如果B启动模式为singleInstance又会怎么调用？B启动模式为singleInstance不变，A→B→C的时候点击两次返回，生命周期如何调用。

**旧的Activity先onPause，新的再启动。**

1)A→B→C→B,B启动模式为`singleTask`

- 启动A的过程，生命周期调用是 (A)onCreate→(A)onStart→(A)onResume
- 再启动B的过程，生命周期调用是 (A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop
- B→C的过程同上，(B)onPause→(C)onCreate→(C)onStart→(C)onResume→(B)onStop
- C→B的过程，由于B启动模式为singleTask，所以B会调用onNewIntent，并且将B之上的实例移除，也就是C会被移出栈。所以生命周期调用是 (C)onPause→(B)onNewIntent→(B)onRestart→(B)onStart→(B)onResume→(C)onStop→(C)onDestory

2)A→B→C→B,B启动模式为`singleInstance`

- 如果B为singleInstance，那么C→B的过程，C就不会被移除，因为B和C不在一个任务栈里面。所以生命周期调用是 (C)onPause→(B)onNewIntent→(B)onRestart→(B)onStart→(B)onResume→(C)onStop

3)A→B→C,B启动模式为`singleInstance`,点击两次返回键

- 如果B为singleInstance，A→B→C的过程，生命周期还是同前面一样正常调用。但是点击返回的时候，由于AC同任务栈，所以C点击返回，会回到A，再点击返回才回到B。所以生命周期是：(C)onPause→(A)onRestart→(A)onStart→(A)onResume→(C)onStop→(C)onDestory。
- 再次点击返回，就会回到B，所以生命周期是：(A)onPause→(B)onRestart→(B)onStart→(B)onResume→(A)onStop→(A)onDestory。

#### 9. 屏幕旋转时Activity的生命周期，如何防止Activity重建。

**onSaveInstanceState在onstop前，个onPause没有既定的关系**

- 切换屏幕的生命周期是：onConfigurationChanged->onPause->onSaveInstanceState->onStop->onDestroy->onCreate->onStart->onRestoreInstanceState->onResume
- 如果需要防止旋转时候，`Activity`重新创建的话需要做如下配置：
   在`targetSdkVersion`的值小于或等于12时，配置 android:configChanges="orientation"，
   在`targetSdkVersion`的值大于12时，配置 android:configChanges="orientation|screenSize"

#### 10. onSaveInstanceState() 与 onRestoreIntanceState()

当应用遇到意外情况（如：内存不足、用户直接按Home键）由系统销毁一个Activity时，onSaveInstanceState() 会被调用。但是当用户主动去销毁一个Activity时，例如在应用中按返回键，onSaveInstanceState()就不会被调用。因为在这种情况下，用户的行为决定了不需要保存Activity的状态。通常onSaveInstanceState()只适合用于保存一些临时性的状态，而onPause()适合用于数据的持久化保存。

在activity被杀掉之前调用保存每个实例的状态，以保证该状态可以在onCreate(Bundle)或者onRestoreInstanceState(Bundle) 中恢复。这个方法在一个activity被杀死前调用，当该activity在将来某个时刻回来时可以恢复其先前状态。

[深入理解](https://www.jianshu.com/p/89e0a7533dbe)

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



## 二、Service

#### 1. Activity怎么启动Service，Activity与Service交互，Service与Thread的区别

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

**Activity与Service交互：**Intent，Binder，跨进程通信方式都可

**Service与Thread区别：**

Service：在后台用来操作时间跨度较长的工作应用组件；Service的生命周期方法在主线程中执行，如果想执行一个长时间的工作，需要开启一个分线程（Thread）；远程服务在应用退出，Service不会停止，再次启动应用时，还可以以正在运行的Service通信。

Thread：用来开启一个分线程的类，做一个长时间的工作；Thread类的run( )在分线程中执行；应用退出，Thread也不会停止；再次启动应用，不能再控制之前的Thread对象。

#### 2. Service 一定没界面吗，Activity 一定有界面吗？

- Activity 不是一定有界面。比如一个跳转逻辑控制类（机票的支付中间类）、透明页
- Service 也不是一定没界面。Service 并不依赖于用户可视的 UI 界面，但这也不是绝对的，如前台 Service 就是与 Notification 界面结合使用的；Service 中也可以弹 Toast；
- Service中执行 LayoutInflate 是合法的，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。所以从理论上看也是可以有界面的

#### 3. 怎么在Service中创建Dialog对话框？

1.在我们取得Dialog对象后，需给它设置类型，即：

```java
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT)
```

2.在Manifest中加上权限:

```xml
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINOW" />
```

#### 4. 为什么bindService可以跟Activity生命周期联动？

1、bindService 方法执行时，LoadedApk 会记录 ServiceConnection 信息。

2、Activity 执行 finish 方法时，会通过 LoadedApk 检查 Activity 是否存在未注销/解绑的 BroadcastReceiver 和 ServiceConnection，如果有，那么会通知 AMS 注销/解绑对应的 BroadcastReceiver 和 Service，并打印异常信息，告诉用户应该主动执行注销/解绑的操作。

#### 5. 如何保证Service不被杀死？

Android 进程不死从3个层面入手：

A.提供进程优先级，降低进程被杀死的概率

方法一：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。

方法二：启动前台service。

方法三：提升service优先级：

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

B. 在进程被杀死后，进行拉活

方法一：注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等

方法二：双进程相互唤起。

方法三：依靠系统唤起。

方法四：onDestroy方法里重启service：service + broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

C. 依靠第三方

根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。

## 三、BroadcaseReceiver

#### 1. 程序A能否接收到程序B的广播？

看情况，使用全局的BroadCastRecevier能进行跨进程通信，但是注意它只能被动接收广播。此外，LocalBroadCastRecevier只限于本进程的广播间通信。

#### 2. 广播传输的数据是否有限制，是多少，为什么要限制？

Intent在传递数据时是有大小限制的，大约限制在1MB之内，你用Intent传递数据，实际上走的是跨进程通信（IPC），跨进程通信需要把数据从内核copy到进程中，每一个进程有一个接收内核数据的缓冲区，默认是1M；如果一次传递的数据超过限制，就会出现异常。

不同厂商表现不一样有可能是厂商修改了此限制的大小，也可能同样的对象在不同的机器上大小不一样。

传递大数据，不应该用Intent；考虑使用ContentProvider或者直接匿名共享内存。简单情况下可以考虑分段传输。

#### 3. 广播注册一般有几种，各有什么优缺点？

第一种是常驻型(静态注册)：当应用程序关闭后如果有信息广播来，程序也会被系统调用，自己运行。

第二种不常驻(动态注册)：广播会跟随程序的生命周期。

动态注册 

优点： 在android的广播机制中，动态注册优先级高于静态注册优先级，因此在必要情况下，是需要动态注册广播接收者的。

缺点： 当用来注册的 Activity 关掉后，广播也就失效了。

静态注册 

优点： 无需担忧广播接收器是否被关闭，只要设备是开启状态，广播接收器就是打开着的。

## 四、ContentProvider

#### 1. ContentProvider使用方法。

进行跨进程通信，实现进程间的数据交互和共享。通过Context 中 getContentResolver() 获得实例，通过 Uri匹配进行数据的增删改查。ContentProvider使用表的形式来组织数据，无论数据的来源是什么，ConentProvider 都会认为是一种表，然后把数据组织成表格。

#### 2. ContentProvider的权限管理(读写分离，权限控制-精确到表级，URL控制)。

　对于ContentProvider暴露出来的数据，应该是存储在自己应用内存中的数据，对于一些存储在外部存储器上的数据，并不能限制访问权限，使用ContentProvider就没有意义了。对于ContentProvider而言，有很多权限控制，可以在AndroidManifest.xml文件中对<provider>节点的属性进行配置，一般使用如下一些属性设置：

- android:grantUriPermssions:临时许可标志。
- android:permission:Provider读写权限。
- android:readPermission:Provider的读权限。
- android:writePermission:Provider的写权限。
- android:enabled:标记允许系统启动Provider。
- android:exported:标记允许其他应用程序使用这个Provider。
- android:multiProcess:标记允许系统启动Provider相同的进程中调用客户端。

#### 3. 说说ContentProvider、ContentResolver、ContentObserver 之间的关系？

ContentProvider：管理数据，提供数据的增删改查操作，数据源可以是数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做进程间数据共享。

ContentResolver：ContentResolver可以为不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互。

ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界。

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
- `replace()`: 相当于旧Fragment调用remove()，新Fragment调用add()。remove()+add()的生命周期加起来
- `show()`: 不调用任何生命周期方法，调用该方法的前提是要显示的 Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为true。
- `hide()`: 不调用任何生命周期方法，调用该方法的前提是要显示的Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为false。

#### 2. ViewPager切换Fragment遇到过什么问题吗? 什么最耗时？

- 滑动的时候，调用setCurrentItem方法，要注意第二个参数`smoothScroll`。传false，就是直接跳到fragment，传true，就是平滑过去。一般主页切换页面都是用false。
- 禁止预加载的话，调用`setOffscreenPageLimit(0)`是无效的，因为方法里面会判断是否小于1。需要重写`setUserVisibleHint`方法，判断fragment是否可见。
- 不要使用`getActivity()`获取activity实例，容易造成空指针，因为如果fragment已经onDetach()了，那么就会报空指针。所以要在`onAttach`方法里面，就去获取activity的上下文。
- `FragmentStatePagerAdapter`对limit外的Fragment销毁，生命周期为onPause->onStop->onDestoryView->onDestory->onDetach, onAttach->onCreate->onCreateView->onStart->onResume。也就是说切换fragment的时候有可能会多次`onCreateView`，所以需要注意处理数据。
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

3）Eventbus、RxBus、Otto等框架。

4）广播

#### 4. Fragment状态保存

Fragment状态保存入口:

- Activity的状态保存, 在Activity的onSaveInstanceState()里, 调用了FragmentManger的saveAllState()方法, 其中会对mActive中各个Fragment的实例状态和View状态分别进行保存.

- FragmentManager还提供了public方法: saveFragmentInstanceState(), 可以对单个Fragment进行状态保存, 这是提供给我们用的。

- FragmentManager的moveToState()方法中, 当状态回退到ACTIVITY_CREATED, 会调用saveFragmentViewState()方法, 保存View的状态.



## 六、屏幕适配

见《Android屏幕适配方案详解》

#### 1. 你们 Android 开发的时候，对于 UI 稿的 px 是如何适配的？

今日头条 AndroidAutoSize和smallestWidth方案。

####  2. 平时如何有使用屏幕适配吗？原理是什么呢？

平时的屏幕适配一般采用的头条的屏幕适配方案。简单来说，以屏幕的一边作为适配，通常是宽。

原理：设备像素`px`和设备独立像素`dp`之间的关系是

```
px = dp * density
```

假设UI给的设计图屏幕宽度基于360dp，那么设备宽的像素点已知，即px，dp也已知，360dp，所以`density = px / dp`，之后根据这个修改系统中跟`density`相关的点即可。

## 七、Lrucache

见《Android基础》中LruCache原理解析章节

## 八、Android消息机制

#### 1. Android消息机制介绍？

Android消息机制中的四大概念：

- `ThreadLocal`：当前线程存储的数据仅能从当前线程取出。
- `MessageQueue`：具有时间优先级的消息队列（单链表）。
- `Looper`：轮询消息队列，看是否有新的消息到来。
- `Handler`：具体处理逻辑的地方。
- `Message`：需要传递的消息，可以传递数据；

过程：

1. 准备工作：创建`Handler`，如果是在子线程中创建，还需要调用`Looper#prepare()`，在`Handler`的构造函数中，会绑定其中的`Looper`和`MessageQueue`。
2. 发送消息：创建消息，使用`Handler`发送。
3. 进入`MessageQueue`：因为`Handler`中绑定着消息队列，所以`Message`很自然的被放进消息队列。
4. `Looper`轮询消息队列：`Looper`是一个死循环，一直观察有没有新的消息到来，之后从`Message`取出绑定的`Handler`，最后调用`Handler`中的处理逻辑，这一切都发生在`Looper`循环的线程，这也是`Handler`能够在指定线程处理任务的原因。

#### 2. Looper在主线程中死循环为什么没有导致界面的卡死？

1. 导致卡死的是在Ui线程中执行耗时操作导致界面出现掉帧，甚至`ANR`，`Looper.loop()`这个操作本身不会导致这个情况。

2. 有人可能会说，我在点击事件中设置死循环会导致界面卡死，同样都是死循环，不都一样的吗？Looper会在没有消息的时候阻塞当前线程，释放CPU资源，等到有消息到来的时候，再唤醒主线程。

3. App进程中是需要死循环的，如果循环结束的话，App进程就结束了。

[《Android中为什么主线程不会因为Looper.loop()里的死循环卡死？》](https://www.zhihu.com/question/34652589)

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

同步屏障是通过MessageQueue的postSyncBarrier方法插入到消息队列的:

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

当调用handler.sendMessage(msg)发送消息，最终会走到：

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);//把消息设置为异步消息
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

- 公开的方法：在发送消息时通过 message.setAsynchronous(true)将消息设为异步的，这个方法是公开的，我们可以正常使用。

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

- 有延时消息，在界面关闭后及时移除Message/Runnable，调用`handler.removeCallbacksAndMessages(null)`

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

MessageQueue#enqueueMessage()内部通过synchronized关键字保证线程安全。同时messagequeue.next()内部也会通过synchronized加锁，确保取的时候线程安全。





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
public class LongPressView2 extends View{  
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
            isMoved = false;  
            postDelayed(mLongPressRunnable, ViewConfiguration.getLongPressTimeout());  
            break;  
        case MotionEvent.ACTION_MOVE:  
            if(isMoved) break;  
            if(Math.abs(mLastMotionX-x) > TOUCH_SLOP   
                    || Math.abs(mLastMotionY-y) > TOUCH_SLOP) {  
                //移动超过阈值，则表示移动了  
                isMoved = true;  
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

https://www.jianshu.com/p/3581fcf302fd

如果某一个子View处理了Down事件，那么随之而来的Move和Up事件也会交给它处理。但是交给它处理之前，父View还是可以拦截事件的，如果拦截了事件，那么子View就会收到一个Cancel事件，并且不会收到后续的Move和Up事件。

手势操作ActionCancel是ViewGroup拦截了Move事件，这个Move事件将会转化为Cancel事件传递给子View

取消2种方式：

- 修改ViewGroup不拦截Move事件
- 子View可以通过设置`requestDisallowInterceptTouchEvent(true)`来达到禁止父ViewGroup拦截事件的目的。

[Android事件分发之ACTION_CANCEL机制及作用](https://blog.csdn.net/cufelsd/article/details/89471402)

#### 5. setOnTouchListener,onClickeListener和onTouchEvent的关系

如果它的`onTouchListener`被设置了的话，则onTouch会被调用，如果onTouch的返回值返回true，则`onTouchEvent`不会被调用。如果返回false或者没有设置onTouchListener，则会继续调用onTouchEvent。而onClick方法则是设置了`onClickListener`则会被正常调用。

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

## 十、View绘制

#### 1. 怎么计算一个View在屏幕可见部分的百分比？

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

#### 5.onCreate,onResume,onStart里面，什么地方可以获得宽高

如果在onCreate、onStart、onResume中直接调用View的getWidth/getHeight方法，是无法得到View宽高的正确信息，因为view的measure过程与Activity的生命周期是不同步的，所以无法保证在这些生命周期里view
的measure已经完成。所以很有可能获取的宽高为0。

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

View.post()的原理：**以Handler为基础，View.post() 将传入任务添加到 View绘制任务所在的消息队列尾部，从而保证View.post() 任务的执行时机是在View 绘制任务完成之后的。** 其中，几个关键点：

- View.post()实际操作：将view.post()传入的任务保存到一个数组里 
- View.post()添加的任务 添加到 View绘制任务所在的消息队列尾部的时机：View 绘制流程的开始阶段，即 ViewRootImpl.performTraversals()
- View.post()添加的任务执行时机：在View绘制任务之后

#### 7. 自定义LinearLayout，怎么测量子View宽高



#### 8. setFactory和setFactory2有什么区别？



#### 9. 如何求当前Activity View的深度

```java

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





## 十一、Drawbale和动画

#### 1. 介绍一下android动画

- View 动画：
  - 作用对象是 View，可用 xml 定义，建议 xml 实现比较易读
  - 支持四种效果：平移、缩放、旋转、透明度
- 帧动画：
  - 通过 AnimationDrawable 实现，容易 OOM
- 属性动画：
  - 可作用于任何对象，可用 xml 定义，Android 3 引入，建议代码实现比较灵活
  - 包括 ObjectAnimator、ValuetAnimator、AnimatorSet
  - 时间插值器：根据时间流逝的百分比计算当前属性改变的百分比，系统预置匀速、加速、减速等插值器
  - 类型估值器：根据当前属性改变的百分比计算改变后的属性值，系统预置整型、浮点、色值等类型估值器
  - 使用注意事项：避免使用帧动画，容易OOM；界面销毁时停止动画，避免内存泄漏；开启硬件加速，提高动画流畅性
  - 硬件加速原理：将 cpu 一部分工作分担给 gpu ，使用 gpu 完成绘制工作；从工作分摊和绘制机制两个方面优化了绘制速度


- tween 补间动画。通过指定View的初末状态和变化方式，对View的内容完成一系列的图形变换来实现动画效果。 Alpha, Scale ,Translate, Rotate。
- frame 帧动画。AnimationDrawable控制animation-list.xml布局
- PropertyAnimation 属性动画3.0引入，属性动画核心思想是对值的变化。

Property Animation 动画有两个步聚：

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

估值器：在1的基础上由这个东西来计算出属性到底变化了多少数值的类

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

**为什么属性动画移动后仍可点击？**

播放补间动画的时候，我们所看到的变化，都只是临时的。而属性动画呢，它所改变的东西，却会更新到这个View所对应的矩阵中，所以当ViewGroup分派事件的时候，会正确的将当前触摸坐标，转换成矩阵变化后的坐标，这就是为什么播放补间动画不会改变触摸区域的原因了。

#### 2. Drawable与View有什么区别,Drawable有哪些子类



#### 3. 属性动画更新时会回调onDraw吗？



#### 4. 两个getDrawable取得的对象，有什么区别？



#### 5. 补间动画与属性动画的区别，哪个效率更高？



#### 6. 动画里面用到了什么设计模式



#### 7. 动画连续调用的原理是什么？



#### 8. 自定义圆角图片



#### 9. Canvas.save()跟Canvas.restore()的调用时机

save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。

restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。

save和restore要配对使用（restore可以比save少，但不能多），如果restore调用次数比save多，会引发Error。save和restore操作执行的时机不同，就能造成绘制的图形不同。



## 十二、AsyncTask

#### 1. AsyncTask的缺陷和问题，说说他的原理。

**AsyncTask是什么？**

AsyncTask是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。

AsyncTask是一个抽象的泛型类，它提供了Params、Progress和Result这三个泛型参数，其中Params表示参数的类型，Progress表示后台任务的执行进度和类型，而Result则表示后台任务的返回结果的类型，如果AsyncTask不需要传递具体的参数，那么这三个泛型参数可以用Void来代替。

**关于线程池：**

AsyncTask对应的线程池ThreadPoolExecutor都是进程范围内共享的，且都是static的，所以是Asynctask控制着进程范围内所有的子类实例。由于这个限制的存在，当使用默认线程池时，如果线程数超过线程池的最大容量，线程池就会爆掉(3.0后默认串行执行，不会出现个问题)。针对这种情况，可以尝试自定义线程池，配合Asynctask使用。

**关于默认线程池：**

AsyncTask里面线程池是一个核心线程数为CPU + 1，最大线程数为CPU * 2 + 1，工作队列长度为128的线程池，线程等待队列的最大等待数为28，但是可以自定义线程池。线程池是由AsyncTask来处理的，线程池允许tasks并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉。所以希望tasks能够串行运行的话，使用SERIAL_EXECUTOR。

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

在Android1.6之前的版本，AsyncTask是串行的，在1.6之后的版本，采用线程池处理并行任务，但是从Android 3.0开始，为了避免AsyncTask所带来的并发错误，又采用一个线程来串行执行任务。可以使用executeOnExecutor()方法来并行地执行任务。

**AsyncTask原理**

- AsyncTask中有两个线程池（SerialExecutor和THREAD_POOL_EXECUTOR）和一个Handler（InternalHandler），其中线程池SerialExecutor用于任务的排队，而线程池THREAD_POOL_EXECUTOR用于真正地执行任务，InternalHandler用于将执行环境从线程池切换到主线程。
- sHandler是一个静态的Handler对象，为了能够将执行环境切换到主线程，这就要求sHandler这个对象必须在主线程创建。由于静态成员会在加载类的时候进行初始化，因此这就变相要求AsyncTask的类必须在主线程中加载，否则同一个进程中的AsyncTask都将无法正常工作。

#### 2. Thread、AsyncTask、IntentService的使用场景与特点。

1. Thread线程，独立运行与于 Activity 的，当Activity 被 finish 后，如果没有主动停止 Thread或者 run 方法没有执行完，其会一直执行下去。
2. AsyncTask 封装了两个线程池和一个Handler（SerialExecutor用于排队，THREAD_POOL_EXECUTOR为真正的执行任务，Handler将工作线程切换到主线程），其必须在 UI线程中创建，execute 方法必须在 UI线程中执行，一个任务实例只允许执行一次，执行多次抛出异常，用于网络请求或者简单数据处理。
3. IntentService：处理异步请求，实现多线程，在onHandleIntent中处理耗时操作，多个耗时任务会依次执行，执行完毕自动结束。





## 十三、Bitmap

#### 1. 下载一张很大的图，如何保证不 oom？



#### 2. Bitmap的内存计算方式？

在已知图片的长和宽的像素的情况下，影响内存大小的因素会有**资源文件位置和像素点大小**。

**像素点大小**： 常见的像素点有：

- ARGB_8888：4个字节
- ARGB_4444、ARGB_565：2个字节
- ALPHA_8   每个像素占用1byte内存       

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

> [《Android Bitmap的内存大小是如何计算的？》](https://ivonhoe.github.io/2017/03/22/Bitmap&Memory/)

#### 3.  Bitmap的高效加载？

Bitmap的高效加载在Glide中也用到了，思路：

1. 获取需要的长和宽，一般获取控件的长和宽。
2. 设置`BitmapFactory.Options`中的`inJustDecodeBounds`为true，可以帮助我们在不加载进内存的方式获得`Bitmap`的长和宽。
3. 对需要的长和宽和Bitmap的长和宽进行对比，从而获得压缩比例，放入`BitmapFactory.Options`中的`inSampleSize`属性。
4. 设置`BitmapFactory.Options`中的`inJustDecodeBounds`为false，将图片加载进内存，进而设置到控件中。

## 十四、RecyclerView

#### 1. RecyclerView是怎么处理内部ViewClick冲突的



#### 2. 讲一下RecyclerView的缓存机制,滑动10个，再滑回去，会有几个执行onBindView



#### 3. 如何实现RecyclerView的局部更新，用过payload吗,notifyItemChange方法中的参数？



#### 4. RecyclerView的缓存结构是怎样的？缓存的是什么？cachedView会执行onBindView吗?

Recycleview有四级缓存，分别是`mAttachedScrap(屏幕内)`，`mCacheViews(屏幕外)`，`mViewCacheExtension(自定义缓存)`，`mRecyclerPool(缓存池)`

- `mAttachedScrap(屏幕内)`，用于屏幕内itemview快速重用，不需要重新createView和bindView
- `mCacheViews(屏幕外)`，保存最近移出屏幕的ViewHolder，包含数据和position信息，复用时必须是相同位置的ViewHolder才能复用，应用场景在那些需要来回滑动的列表中，当往回滑动时，能直接复用ViewHolder数据，不需要重新bindView。
- `mViewCacheExtension(自定义缓存)`，不直接使用，需要用户自定义实现，默认不实现。
- `mRecyclerPool(缓存池)`，当cacheView满了后或者adapter被更换，将cacheView中移出的ViewHolder放到Pool中，放之前会把ViewHolder数据清除掉，所以复用时需要重新bindView。

四级缓存按照顺序需要依次读取。所以**完整缓存流程**是：

保存缓存流程：

- 插入或是删除itemView时，先把屏幕内的ViewHolder保存至AttachedScrap中
- 滑动屏幕的时候，先消失的itemview会保存到CacheView，CacheView大小默认是2，超过数量的话按照先入先出原则，移出头部的itemview保存到RecyclerPool缓存池（如果有自定义缓存就会保存到自定义缓存里），RecyclerPool缓存池会按照itemview的itemtype进行保存，每个itemTyep缓存个数为5个，超过就会被回收。

获取缓存流程：

- AttachedScrap中获取，通过pos匹配holder

  ——>获取失败，从CacheView中获取，也是通过pos获取holder缓存
  ——>获取失败，从自定义缓存中获取缓存——>获取失败，从mRecyclerPool中获取
  ——>获取失败，重新创建viewholder——createViewHolder并bindview。

需要注意的是，如果从缓存池找到缓存，还需要重新bindview。

#### 5. RecyclerView嵌套RecyclerView，NestScrollView嵌套ScrollView滑动冲突



#### 6. RecyclerView 缓存结构，RecyclerView预取，RecyclerView局部刷新



#### 7. Recycleview和listview区别

`Recycleview布局效果更多`，增加了纵向，表格，瀑布流等效果

`Recycleview去掉了一些api`，比如setEmptyview，onItemClickListener等等，给到用户更多的自定义可能

`Recycleview去掉了设置头部底部item的功能`，专向通过viewholder的不同type实现

`Recycleview实现了一些局部刷新`，比如notifyitemchanged

`Recycleview自带了一些布局变化的动画效果`，也可以通过自定义ItemAnimator类实现自定义动画效果

`Recycleview缓存机制更全面`，增加两级缓存，还支持自定义缓存逻辑

#### 8. 说说RecyclerView性能优化。

- `bindViewHolder`方法是在UI线程进行的，此方法不能耗时操作，不然将会影响滑动流畅性。比如进行日期的格式化。
- 对于新增或删除的时候，可以使用`diffutil`进行局部刷新，少用全局刷新
- 对于`itemVIew`进行布局优化，比如少嵌套等。
- 25.1.0 (>=21)及以上使用`Prefetch` 功能，也就是预取功能，嵌套时且使用的是LinearLayoutManager，子RecyclerView可通过setInitialPrefatchItemCount设置预取个数
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
- 用`notifyDataSetChange`时，适配器不知道整个数据集中的那些内容以及存在，再重新匹配`ViewHolder`时会花生闪烁。设置adapter.setHasStableIds(true)，并重写`getItemId()`来给每个Item一个唯一的ID，也就是唯一标识，就使itemview的焦点固定，解决了闪烁问题。



## 十五、ViewPager

#### 1. ViewPager2原理

#### 2. ViewPager中嵌套ViewPager怎么处理滑动冲突

#### 3. viewpager切换掉帧有什么处理经验？

#### 4. 说说事件分发机制，怎么写一个不能滑动的ViewPager

#### 5. ViewPager使用细节，如何设置成每次只初始化当前的Fragment，其他的不初始化（提示：Fragment懒加载）？

自定义一个 LazyLoadFragment 基类，利用 setUserVisibleHint 和 生命周期方法，通过对 Fragment 状态判断，进行数据加载，并将数据加载的接口提供开放出去，供子类使用。然后在子类 Fragment 中实现 requestData 方法即可。这里添加了一个 isDataLoaded 变量，目的是避免重复加载数据。考虑到有时候需要刷新数据的问题，便提供了一个用于强制刷新的参数判断。

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


#### 

## 十四、其他

#### 1. 子线程是否可以 context.startActivity() ？例如ApplicationContext, 会不会有什么问题？

是可以的。

创建对话框时不可以用Application的context，只能用Activity的context。

#### 2. ConstraintLayout实现三等分,ConstraintLayout动画



#### 3. CoordinatorLayout自定义behavior,可以拦截什么？



#### 4. SharedPreferences是如何保证线程安全的，其内部的实现用到了哪些锁

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

- 对于写操作，由于是两步操作，一个是editor.put，一个是commit或者apply所以其实是需要两把锁的：

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

#### 5. SharedParence可以跨进程通信吗？如何改造成可以跨进程通信的commit和apply的区别？

1） SharedPreferences是进程不安全的，因为没有使用跨进程的锁。既然是进程不安全，那么久有可能在多进程操作的时候发生数据异常。

2） 我们有两个办法能保证进程安全：

- 使用跨进程组件，也就是ContentProvider，这也是官方推荐的做法。通过ContentProvider对多进程进行了处理，使得不同进程都是通过ContentProvider访问SharedPreferences。
- 加文件锁，由于SharedPreferences的本质是读写文件，所以我们对文件加锁，就能保证进程安全了。

#### 6. SharedPreferences 操作有文件备份吗？是怎么完成备份的？

- SharedPreferences 的写入操作，首先是将源文件备份：

```java
  if (!backupFileExists) {
      !mFile.renameTo(mBackupFile);
  }
```

- 再写入所有数据，只有写入成功，并且通过 sync 完成落盘后，才会将 Backup（.bak） 文件删除。
- 如果写入过程中进程被杀，或者关机等非正常情况发生。进程再次启动后如果发现该 SharedPreferences 存在 Backup 文件，就将 Backup 文件重名为源文件，原本未完成写入的文件就直接丢弃，这样就能保证之前数据的正确。



#### 7. 一个wrap_content的ImageView，加载远程图片，传什么参数裁剪比较好?



#### 8. 平常抓包用什么工具？

Fiddler

#### 9.实现一个下载功能的接口



#### 10. attachToWindow什么时候调用？



#### 11. Activity内LinearLayout红色wrap_content,包含View绿色wrap_content,求界面颜色



#### 12. Bundle是什么数据结构?利用什么传递数据



#### 13. SharedPreference原理？读取xml是在哪个线程?

#### 14. Bunder传递对象为什么需要序列化？Serialzable和Parcelable的区别？

因为bundle传递数据时只支持基本数据类型，所以在传递对象时需要序列化转换成可存储或可传输的本质状态（字节流）。序列化后的对象可以在网络、IPC（比如启动另一个进程的Activity、Service和Reciver）之间进行传输，也可以存储到本地。

Serializable（Java自带）：

Serializable 是序列化的意思，表示将一个对象转换成存储或可传输的状态。序列化后的对象可以在网络上进传输，也可以存储到本地。

Parcelable（android专用）：

除了Serializable之外，使用Parcelable也可以实现相同的效果，不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这也就实现传递对象的功能了。

区别总结如下图所示：

![image](https://user-gold-cdn.xitu.io/2019/3/8/1695c349f019c41f?imageslim)

#### 14. 怎么优化xml inflate的时间，涉及IO与反射。了解compose吗？



#### 15. Android各版本新特性

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

Android10.0（Q）目前曝光的新特性

- **夜间模式**：包括手机上的所有应用都可以为其设置暗黑模式。
- **桌面模式**：提供类似于PC的体验，但是远远不能代替PC。
- **屏幕录制**：通过长按“电源”菜单中的"屏幕快照"来开启。

#### 16. android中有哪几种解析xml的类,官方推荐哪种？以及它们的原理和区别？

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

优点:SAX 对内存的要求比较低,因为它让开发人员自己来决定所要处理的标签.特别是当开发人员只需要处理文档中包含的部分数据时,SAX 这种扩展能力得到了更好的体现.

缺点:用SAX方式进行XML解析时,需要顺序执行,所以很难访问同一文档中的不同数据.此外,在基于该方式的解析编码程序也相对复杂.

使用场景:对于含有数据量十分巨大,而又不用对文档的所有数据行遍历或者分析的时候,使用该方法十分有效.该方法不将整个文档读入内存,而只需读取到程序所需的文档标记处即可.

**Xmlpull解析**

android SDK提供了xmlpullapi,xmlpull和sax类似,是基于流（stream）操作文件,后者根据节点事件回调开发者编写的处理程序.因为是基于流的处理,因此xmlpull和sax都比较节约内存资源,不会像dom那样要把所有节点以对象树的形式展现在内存中.xmpull比sax更简明,而且不需要扫描完整个流.

#### 17. Jar和Aar的区别

Jar包里面只有代码，aar里面不光有代码还包括资源文件，比如 drawable 文件，xml资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度。

#### 18. Android为每个应用程序分配的内存大小是多少

android程序内存一般限制在16M，也有的是24M。近几年手机发展较快，一般都会分配两百兆左右，和具体机型有关。

#### 19. Jar和Aar的区别

Jar包里面只有代码，aar里面不光有代码还包括资源文件，比如 drawable 文件，xml资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度。

#### 20. Android为每个应用程序分配的内存大小是多少

android程序内存一般限制在16M，也有的是24M。近几年手机发展较快，一般都会分配两百兆左右，和具体机型有关。

#### 21. Merge、ViewStub 的作用。

Merge: 减少视图层级，可以删除多余的层级。和Include标签配套使用

ViewStub: 按需加载，减少内存使用量、加快渲染速度、不支持 merge 标签。

#### 22. Asset目录与res目录的区别？

assets：不会在 R 文件中生成相应标记，存放到这里的资源在打包时会打包到程序安装包中。（通过 AssetManager 类访问这些文件）

res：会在 R 文件中生成 id 标记，资源在打包时如果使用到则打包到安装包中，未用到不会打入安装包中。

res/anim：存放动画资源。

res/raw：和 asset 下文件一样，打包时直接打入程序安装包中（会映射到 R 文件中）。

#### 23. 通过google提供的Gson解析json时，定义JavaBean的规则是什么？

1) 实现序列化 Serializable

2) 属性私有化，并提供get，set方法

3) 提供无参构造

4) 属性名必须与json串中属性名保持一致 （因为Gson解析json串底层用到了Java的反射原理）

#### 24. json解析方式的两种区别？

1) SDK提供JSONArray，JSONObject

2) google提供的 Gson

通过fromJson()实现对象的反序列化（即将json串转换为对象类型）

通过toJson()实现对象的序列化 （即将对象类型转换为json串）

3) 阿里的fastjson

#### 25. 数据库升级增加表和删除表都不涉及数据迁移，但是修改表涉及到对原有数据进行迁移。升级的方法如下所示：

- 将现有表命名为临时表。
- 创建新表。
- 将临时表的数据导入新表。
- 删除临时表。

如果是跨版本数据库升级，可以有两种方式：

- 逐级升级，确定相邻版本与现在版本的差别，V1升级到V2,V2升级到V3，依次类推。
- 跨级升级，确定每个版本与现在数据库的差别，为每个case编写专门升级大代码。

#### 26. 编译期注解跟运行时注解

运行期注解(RunTime)利用反射去获取信息还是比较损耗性能的，对应@Retention（RetentionPolicy.RUNTIME）。

编译期(Compile time)注解，以及处理编译期注解的手段APT和Javapoet，对应@Retention(RetentionPolicy.CLASS)。

其中apt+javaPoet目前也是应用比较广泛，在一些大的开源库，如EventBus3.0+,页面路由 ARout、Dagger、Retrofit等均有使用的身影，注解不仅仅是通过反射一种方式来使用，也可以使用APT在编译期处理

#### 27. 强引用置为null，会不会被回收？

不会立即释放对象占用的内存。 如果对象的引用被置为null，只是断开了当前线程栈帧中对该对象的引用关系，而 垃圾收集器是运行在后台的线程，只有当用户线程运行到安全点(safe point)或者安全区域才会扫描对象引用关系，扫描到对象没有被引用则会标记对象，这时候仍然不会立即释放该对象内存，因为有些对象是可恢复的（在 finalize方法中恢复引用 ）。只有确定了对象无法恢复引用的时候才会清除对象内存。

#### 28. 是否了解硬件加速？

硬件加速就是运用GPU优秀的运算能力来加快渲染的速度，而通常的基于软件的绘制渲染模式是完全利用CPU来完成渲染。

1. 硬件加速是从API 11引入，API 14之后才默认开启。对于标准的绘制操作和控件都是支持的，但是对于自定义View的时候或者一些特殊的绘制函数就需要考虑是否需要关闭硬件加速。

2. 我们面对不支持硬件加速的情况，就需要限制硬件加速，这个兼容性的问题是因为硬件加速是把View的绘制函数转化为使用OpenGL的函数来进完成实际的绘制的，那么必然会存在OpenGL中不支持原始回执函数的情况，对于这些绘制函数，就会失效。

3. 硬件加速的消耗问题，因为是使用OpenGL，需要把系统中OpenGL加载到内存中，OpenGL API调用就会占用8MB，而实际上会占用更多内存，并且使用了硬件必然增加耗电量了。

4. 硬件加速的优势还有display list的设计，使用这个我们不需要每次重绘都执行大量的代码，基于软件的绘制模式会重绘脏区域内的所有控件，而display只会更新列表，然后绘制列表内的控件。
5. CPU更擅长复杂逻辑控制，而GPU得益于大量ALU和并行结构设计，更擅长数学运算。

#### 29. 对于应用更新这块是如何做的？(灰度，强制更新，分区域更新)

1、通过接口获取线上版本号，versionCode
2、比较线上的versionCode 和本地的versionCode，弹出更新窗口
3、下载APK文件（文件下载）
4、安装APK

灰度：
(1) 找单一渠道投放特别版本。
(2) 做升级平台的改造，允许针对部分用户推送升级通知甚至版本强制升级。
(3) 开放单独的下载入口。
(4) 是两个版本的代码都打到app包里，然后在app端植入测试框架，用来控制显示哪个版本。测试框架负责与服务器端api通信，由服务器端控制app上A/B版本的分布，可以实现指定的一组用户看到A版本，其它用户看到B版本。服务端会有相应的报表来显示A/B版本的数量和效果对比。最后可以由服务端的后台来控制，全部用户在线切换到A或者B版本~

无论哪种方法都需要做好版本管理工作，分配特别的版本号以示区别。
当然，既然是做灰度，数据监控（常规数据、新特性数据、主要业务数据）还是要做到位，该打的数据桩要打。
还有，灰度版最好有收回的能力，一般就是强制升级下一个正式版。

强制更新:一般的处理就是进入应用就弹窗通知用户有版本更新，弹窗可以没有取消按钮并不能取消。这样用户就只能选择更新或者关闭应用了，当然也可以添加取消按钮，但是如果用户选择取消则直接退出应用。

增量更新：bsdiff：二进制差分工具bsdiff是相应的补丁合成工具,根据两个不同版本的二进制文件，生成补丁文件.patch文件。通过bspatch使旧的apk文件与不定文件合成新的apk。 注意通过apk文件的md5值进行区分版本。

#### 30. 请解释安卓为啥要加签名机制。

1、开发者的身份认证

由于开发商可能通过使用相同的 Package Name 来混淆替换已经安装的程序，以此保证签名不同的包不被替换。

2、保证信息传输的完整性

签名对于包中的每个文件进行处理，以此确保包中内容不被替换。

3、防止交易中的抵赖发生， Market 对软件的要求。

#### 31. 如何通过Gradle配置多渠道包？

用于生成不同渠道的包

    android {  
        productFlavors {
            xiaomi {}
            baidu {}
            wandoujia {}
            _360 {}        // 或“"360"{}”，数字需下划线开头或加上双引号
        }
    }

执行./gradlew assembleRelease ，将会打出所有渠道的release包；

执行./gradlew assembleWandoujia，将会打出豌豆荚渠道的release和debug版的包；

执行./gradlew assembleWandoujiaRelease将生成豌豆荚的release包。

因此，可以结合buildType和productFlavor生成不同的Build Variants，即类型与渠道不同的组合。

#### 32. ddms 和 traceView 的区别？

ddms 原意是：davik debug monitor service。简单的说 ddms 是一个程序执行查看器，在里面可以看见线程和堆栈等信息，traceView 是程序性能分析器。traceview 是 ddms 中的一部分内容。

Traceview 是 Android 平台特有的数据采集和分析工具，它主要用于分析 Android 中应用程序的 hotspot（瓶颈）。Traceview 本身只是一个数据分析工具，而数据的采集则需要使用 Android SDK 中的 Debug 类或者利用DDMS 工具。二者的用法如下：开发者在一些关键代码段开始前调用 Android SDK 中 Debug 类的 startMethodTracing 函数，并在关键代码段结束前调用 stopMethodTracing 函数。这两个函数运行过程中将采集运行时间内该应用所有线程（注意，只能是 Java线程） 的函数执行情况， 并将采集数据保存到/mnt/sdcard/下的一个文件中。 开发者然后需要利用 SDK 中的 Traceview工具来分析这些数据。

#### 33. SharedPrefrences的apply和commit有什么区别？

这两个方法的区别在于：

1. apply没有返回值而commit返回boolean表明修改是否提交成功。

2. apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。 

3. apply方法不会提示任何失败的提示。 
   由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。