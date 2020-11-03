# Android进阶

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 一、Android FrameWork
### 1. Binder

#### (1) Binder的介绍？与其他IPC方式的优缺点？

Binder是Android中特有的IPC方式，引用《Android开发艺术探索》中的话(略有改动)：

> 从IPC角度来说，Binder是Android中的一种跨进程通信方式；Binder可以理解为虚拟的物理设备，它的设备驱动是/dev/binder；从`Android Framework`来讲，Binder是`Service Manager`连接各种`Manager`和对应的`ManagerService`的桥梁。从面向对象和CS模型来讲，`Client`通过Binder和远程的`Server`进行通讯。

基于Binder，Android还实现了其他的IPC方式，比如`AIDL`、`Messenger`和`ContentProvider`。

与其他IPC比较：

- 效率高：除了内存共享外，其他IPC都需要进行两次数据拷贝，而因为Binder使用内存映射的关系，仅需要一次数据拷贝。
- 安全性好：接收方可以从数据包中获取发送发的进程Id和用户Id，方便验证发送方的身份，其他IPC想要实验只能够主动存入，但是这有可能在发送的过程中被修改。

#### (2) Binder的通信过程？Binder的原理？

图片：

![Binder通信过程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a654b4bff7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

其实这个过程也可以从AIDL生成的代码中看出。

原理：

![Binder结构](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a683f50b4e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Binder的结构： `Client`：服务的请求方。 `Server`：服务的提供方。 `Service Manager`：为`Server`提供`Binder`的注册服务，为`Client`提供`Binder`的查询服务，`Server`、`Client`和`Service Manage`r的通讯都是通过Binder。 `Binder驱动`：负责Binder通信机制的建立，提供一系列底层支持。

从上图中，Binder通信的过程是这样的：

1. Server在Service Manager中注册：Server进程在创建的时候，也会创建对应的Binder实体，如果要提供服务给Client，就必须为Binder实体注册一个名字。
2. Client通过Service Manager获取服务：Client知道服务中Binder实体的名字后，通过名字从Service Manager获取Binder实体的引用。
3. Client使用服务与Server进行通信：Client通过调用Binder实体与Server进行通信。

更详细一点？

Binder通信的实质是利用内存映射，将用户进程的内存地址和内核的内存地址映射为同一块物理地址，也就是说他们使用的同一块物理空间，每次创建Binder的时候大概分配128的空间。数据进行传输的时候，从这个内存空间分配一点，用完了再释放即可。

#### (3) binder怎么验证pid?binder驱动了解吗？



#### (4) binder进程间通信可以调用原进程方法吗？



### 2. 进程

#### (1) Android有哪些序列化方式？

为了解决Android中内存序列化速度过慢的问题，Android使用了`Parcelable`。

|  对比  |  `Serializable`  | `Parcelable` |
| :----: | :--------------: | :----------: |
| 易用性 |       简单       |  不是很简单  |
|  效率  |        低        |      高      |
|  场景  | IO、网络和数据库 |    内存中    |

#### (2) Zygote孕育进程过程？

![Zygote工作流程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a68eb05620?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### (3) 点击桌面图标的启动过程？涉及的进程和组件

> [默认Home应用程序（Launcher）的启动过程源码分析](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.kancloud.cn%2Falex_wsc%2Fandroids%2F477726)



### 3. 四大组件启动相关

#### (1) Activity的启动过程？

![Activity启动流程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a68cd14124?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> [《3分钟看懂Activity启动流程》](https://www.jianshu.com/p/9ecea420eb52)

#### (2) App的启动过程？

介绍一下App进程和System Server进程如何联系：

![App进程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6aaf5fe97?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- `ActivityThread`：依赖于Ui线程，实际处理与`AMS`中交互的工作。
- `ActivityManagerService`：负责`Activity`、`Service`等的生命周期工作。
- `ApplicationThread`：`System Server`进程中`ApplicatonThreadProxy`的服务端，帮助`System Server`进程跟App进程交流。
- `System Server`：Android核心的进程，掌管着Android系统中各种重要的服务。



![App启动流程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6bd3067d1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



具体过程：

1. 用户点击App图标，`Lanuacher`进程通过`Binder`联系到`System Server`进程发起`startActivity`。
2. `System Server`通过`Socket`联系到`Zygote`，`fork`出一个新的App进程。
3. 创建出一个新的App进程以后，`Zygote`启动App进程的`ActivityThread#main()`方法。
4. 在`ActivtiyThread`中，调用`AMS`进行`ApplicationThread`的绑定。
5. `AMS`发送创建`Application`的消息给`ApplicationThread`，进而转交给`ActivityThread`中的`H`，它是一个`Handler`，接着进行`Application`的创建工作。
6. `AMS`以同样的方式创建`Activity`，接着就是大家熟悉的创建`Activity`的工作了。

#### (3) 登陆功能，登陆成功然后跳转到一个新Activity，中间涉及什么？从事件传递，网络请求,AMS交互角度分析



#### (4) AMS交互调用生命周期是顺序的吗？



#### (5) 说说App的启动过程,在ActivityThread的main方法里面做了什么事，什么时候启动第一个Activity？

#### (6) Launcher启动图标，有几个进程？



#### (7) 启动优化做过什么工作？如果首页就要用到的初始化？



### 4. Window

#### (1) Activity启动过程跟Window的关系？

> [《简析Window、Activity、DecorView以及ViewRoot之间的错综关系》](https://www.jianshu.com/p/8766babc40e0)

#### (2) Activity、Window、ViewRoot和DecorView之间的关系？

> [《总结UI原理和高级的UI优化方式》](https://juejin.im/post/6844903974294781965)

每个 Activity 包含了一个 Window对象，这个对象是由 PhoneWindow做的实现。而 PhoneWindow 将 DecorView作为了一个应用窗口的根 View，这个 DecorView 又把屏幕划分为了两个区域：一个是 TitleView，一个是ContentView，而我们平时在 Xml 文件中写的布局正好是展示在 ContentView 中的。

### 5. PMS

#### (1) Apk的安装过程？

*建议阅读：*

> [《Android Apk安装过程分析》](https://www.jianshu.com/p/953475cea991)

### 6. Contex

#### (1) 关于Context的理解？

> [《Android Context 上下文 你必须知道的一切》](https://blog.csdn.net/lmj623565791/article/details/40481055)

#### (2) Android应用中哪些是 Context，一个应用有多少个 Context？

> [Android Context 熟悉还是陌生](https://www.jianshu.com/p/cc0bb2a71ee8)

#### (3) Context 的使用上如何避免内存泄漏？

#### (4) 如何跨进程拿 Context？如 Activity 还没启动的时候如何拿 Context？

**getApplication 与 getApplicationContext 区别**：getApplication() 用用来获取 Application实例的，但这个方法只有在 Activity 和 Service 中才能调用。如果在一些其它的场景，比如BroadcastReceiver 中也想获得 Application 的实例，这时需要借助 getApplicationContext() 方法。也就是说，getApplicationContext() 方法的作用域会更广一些，任何一个 Context 的实例，只要调用 getApplicationContext() 方法都可以拿到我们的Application对象；

**Application 中方法的执行顺序为**：Application 构造方法 → attachBaseContext() → onCreate()。如果在 attachBaseContext() 中或执行前，调用 getApplicationContext() 得到的值为null



### 7. AIDL

#### (1) AIDL in out oneWay代表什么意思？



## 二、Android权限处理

## 三、多线程断点续传
#### (1) 多线程断点续传r？

基础知识：

- Http基础：在Http请求中，可以加入请求头`Range`，下载指定区间的文件数。
- `RandomAccessFile`：支持随机访问，可以从指定位置进行数据的读写。

有了这个基础以后，思路就清晰了：

1. 通过`HttpUrlConnection`获取文件长度。
2. 自己分配好线程进行制定区间的文件数据的下载。
3. 获取到数据流以后，使用`RandomAccessFile`进行指定位置的读写。。

## 四、第三方库
### 1. RxJava

RxJava难在各种操作符，我们了解一下大致的设计思想即可。

建议寻找一些RxJava的文章。

#### (1) RXJava怎么切换线程

#### (2) Rxjava自定义操作符

#### (3) 广播与RxBus的区别，全局广播与局部广播区别





### 2. OkHttp

#### (1) OkHttp责任链模式

#### (2) interceptors和networkInterceptors的区别？

建议看一遍源码，过程并不复杂。

#### (3) OkHttp怎么实现连接池?里面怎么处理SSL？

#### (4) 如果让你来实现一个网络框架，你会考虑什么

#### (5) OkHttp网络拦截器，应用拦截器?OKHttp有哪些拦截器，分别起什么作用

#### (6) OkHttp里面用到了什么设计模式

#### (7) okhttp 在 response 返回后，调用了 response.toString()，后面再使用 response 会用什么问题？

调用 response.toString() 连接会断开，后面的取值会出问题



### 3. Retrofit

#### (1) 设计模式和封层解耦的理念

#### (2) 动态代理

建议看一遍源码，过程并不复杂。



#### (3) 编译时注解与运行时注解，为什么retrofit要使用运行时注解？什么时候用运行时注解？

#### (4) retrofit怎么做post请求



### 4. Glide

#### (1) Glide和其他图片加载框架的比较？

#### (2) 如何设计一个图片加载框架？

#### (3) Glide缓存实现机制？

#### (4) Glide如何处理生命周期？

#### (5) 有用过Glide的什么深入的API，自定义model是在Glide的什么阶段



[《Glide最全解析》](https://blog.csdn.net/sinyu890807/category_9268670.html)
[《面试官：简历上最好不要写Glide，不是问源码那么简单》](https://juejin.im/post/6844903986412126216)

### 5. Android Jepack(非必需)
我主要阅读了Android Jetpack中以下库的源码：

- `Lifecycle`：观察者模式，组件生命周期中发送事件。
- `DataBinding`：核心就是利用LiveData或者Observablexxx实现的观察者模式，对16进制的状态位更新，之后根据这个状态位去更新对应的内容。
- `LiveData`：观察者模式，事件的生产消费模型。
- `ViewModel`：借用Activty异常销毁时存储隐藏Fragment的机制存储ViewModel，保证数据的生命周期尽可能的延长。
- `Paging`：设计思想。

以后有时间再给大家做源码分析。

*建议阅读：*

> [《Android Jetpack源码分析系列》](https://blog.csdn.net/mq2553299/column/info/24151)

#### 1. LifeCycle的原理是怎样的？如果在onStart里面订阅，会回调onCreate吗？

#### 2. ViewModel为什么在旋转屏幕后不会丢失状态

#### 3. 有没有看一下Google官方的ViewModel demo

#### 4. ViewModel在Activity初始化与在Fragment中初始化，有什么区别？

#### 5.viewModel是怎么实现双向数据绑定的？

#### 6.viewModel怎么实现自动处理生命周期？

#### 7. ViewModel的使用中有什么坑？

#### 8. DataBinding的原理了解吗？

#### 9. ViewModel如何知道View层的生命周期？

事实上，如果你仅仅使用ViewModel，它是感知不了生命周期，它需要结合LiveData去感知生命周期，如果仅仅使用DataBinding去实现MVVM，它对数据源使用了弱引用，所以一定程度上可以避免内存泄漏的发生。





## 五、性能优化

[《Android 性能优化最佳实践》](https://juejin.im/post/6844903641032163336)

### 1. 布局优化

### 2. 绘制优化
### 3. 内容泄漏

#### 1. 什么是内存泄漏？有哪些原因会引起内存泄漏？

> [Android 内存泄露与分析工具](https://www.jianshu.com/p/97fb764f2669)

内存泄漏（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。
简单点说，手机给我们的应用提供了一定大小的堆内存，在不断创建对象的过程中，也在不断的GC(java的垃圾回收机制)，所以内存正常情况下会保持一个平稳的值。
但是出现内存泄漏就会导致某个实例，比如Activity的实例，应用被某个地方引用到了，不能正常释放，从而导致内存占用越来越大，这就是内存泄漏。

主要有`四类情况`：

- 集合类泄漏
- 单例/静态变量造成的内存泄漏
- 匿名内部类/非静态内部类
- 资源未关闭造成的内存泄漏

1）集合类泄漏

集合类添加元素后，仍引用着集合元素对象，导致该集合中的元素对象无法被回收，从而导致内存泄露。

```java
static List<Object> mList = new ArrayList<>();
   for (int i = 0; i < 100; i++) {
       Object obj = new Object();
      mList.add(obj);
       obj = null;
    }
```

解决办法就是把集合也释放掉。

```java
  mList.clear();
  mList = null;
```

2）单例/静态变量造成的内存泄漏

单例模式具有其`静态特性`，它的生命周期等于应用程序的生命周期，正是因为这一点，往往很容易造成内存泄漏。

```java
public class SingleInstance {

    private static SingleInstance mInstance;
    private Context mContext;

    private SingleInstance(Context context){
        this.mContext = context;
    }

    public static SingleInstance newInstance(Context context){
        if(mInstance == null){
            mInstance = new SingleInstance(context);
        }
        return sInstance;
    }
}
```

比如这个单例模式，如果我们调用`newInstance`方法时候把Activity的`context`传进去，那么就是生命周期长的持有了生命周期短的引用，造成了内存泄漏。要修改的话把context改成`context.getApplicationContext()`即可。

3）匿名内部类/非静态内部类

非静态内部类他会持有他外部类的强引用，所以就有可能导致非静态内部类的生命周期可能比外部类更长，容易造成内存泄漏，最常见的就是`Handler`。

```java
public class TestActivity extends Activity {
	private TextView mTextView;
    private Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
			switch (msg.what) {
                case 1:
                    mTextView.setText(msg.obj + "");
                    break;
                default:
                    break;
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);
        mTextView = findViewById(R.id.tv_handler);
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
                mHandler.sendMessage(msg);
            }
        }).start();
    }
```

**怎么修改呢？**

改成静态内部类，然后弱引用方式修饰外部类

**为何handler要定义为static?**

因为静态内部类不持有外部类的引用，所以使用静态的handler不会导致activity的泄露

**还要用WeakReference 包裹外部类的对象?**

这是因为我们需要使用外部类的成员，可以通过"activity. "获取变量方法等，如果直接使用强引用，显然会导致activity泄露。

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

4）资源未关闭造成的内存泄漏，比如：

- 网络、文件等流忘记关闭
- 手动注册广播时，退出时忘记`unregisterReceiver()`
- Service 执行完后忘记 `stopSelf()`
- EventBus 等观察者模式的框架忘记手动解除注册

### 该怎么发现和解决内存泄漏？

1、使用工具，比如`Memory Profiler`，可以查看app的内存实时情况，捕获堆转储，就生成了一个内存快照，`hprof`文件。通过查看文件，可以看到哪些类发生了内存泄漏。

2、使用库，比较出名的就是`LeakCanary`，导入库，然后运行后，就可以发现app内的内存泄漏情况。

这里说下`LeakCanary`的原理：

- 监听
   首先通过`ActivityLifecycleCallbacks`和`FragmentLifeCycleCallbacks`监听Activity和Fragment的生命周期。
- 判断
   然后在销毁的生命周期中判断对象是否被回收。弱引用在定义的时候可以指定引用对象和一个 `ReferenceQueue`，通过该弱引用是否被加入ReferenceQueue就可以判断该对象是否被回收。
- 分析
   最后通过haha库来分析`hprof`文件，从而找出类之前的引用关系。

#### 2. 内存泄漏有什么方式检测？用过哪些工具，其中的原理是什么？

[Java内存问题 及 LeakCanary 原理分析](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5ab8d3d46fb9a028ca52f813)：

基本原理：用ActivityLifecycleCallbacks接口来检测Activity生命周期，主要是在**onDestroy()**方法中，手动调用 GC，然后利用ReferenceQueue+WeakReference 监听对象回收情况 ，来判断是否有释放不掉的引用，再结合dump memory的hpof文件, 用[HaHa](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsquare%2Fhaha)分析出泄漏地方；

LeakCanary会单独开一进程，用来执行分析任务，和监听任务分开处理。Application中可通过processName判断是否是任务执行进程；

利用主线程空闲的时候执行检测任务，在MessageQueue中加入了一个IdleHandler来得到主线程空闲回调；

LeakCanary检测只针对Activiy里的相关对象。其他类无法使用，还得用MAT原始方法



### 4. 响应速度优化

#### 1. 有什么实际解决UI卡顿优化的经历



### 5. 启动优化

#### 1. 具体有哪些启动优化方法？

- 障眼法之闪屏页

为了消除启动时的白屏/黑屏，可以通过设置android:windowBackground，让人感觉一点击icon就启动完毕了的感觉。

```java
        <activity android:name=".ui.activity.启动activity"
            android:theme="@style/MyAppTheme"
            android:screenOrientation="portrait">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <style name="MyAppTheme" parent="Theme.AppCompat.NoActionBar">
            <item name="android:windowBackground">@drawable/logo</item>
        </style>
```

- 预创建Activity

对象第一次创建的时候，java虚拟机首先检查类对应的Class 对象是否已经加载。如果没有加载，jvm会根据类名查找.class文件，将其Class对象载入。同一个类第二次new的时候就不需要加载类对象，而是直接实例化，创建时间就缩短了。

- 第三方库懒加载

很多第三方开源库都说在Application中进行初始化，所以可以把一些不是需要启动就初始化的三方库的初始化放到后面，按需初始化，这样就能让Application变得更轻。

- WebView启动优化

webview第一次启动会非常耗时，具体优化方法可以看 webview章节

- 线程优化

线程是程序运行的基本单位，线程的频繁创建是耗性能的，所以大家应该都会用线程池。单个cpu情况下，即使是开多个线程，同时也只有一个线程可以工作，所以线程池的大小要根据cpu个数来确定。

- MultiDex 优化

由于65536方法限制，所以一般class文件要生成多个dex文件，Android5.0以下，ClassLoader加载类的时候只会从class.dex（主dex）里加载，所以要执行MultiDex.install(context)方法才能正常读取dex类。

而这个install方法就是耗时大户，会解压apk，遍历dex文件，压缩dex、将dex文件通过反射转换成DexFile对象、反射替换数组。

这里需要的方案就是今日头条方案：

- 在Application的attachBaseContext方法里，启动另一个进程的LoadDexActivity去异步执行MultiDex逻辑，显示Loading。
- 然后主进程Application进入while循环，不断检测MultiDex操作是否完成
- MultiDex执行完之后主进程Application继续走，ContentProvider初始化和Application onCreate方法，也就是执行主进程正常的逻辑。

所以重点就是单开进程去执行MultiDex逻辑，这样就不影响APP的启动了。

#### 2. 如何分析启动耗时的方法?

- Systrace + 函数插桩

也就是通过在方法的入口和出口加入统计代码，从而统计方法耗时



```java
class Trace{
    public static void i(String tag){
        android.os.Trace.beginSection(tag);
    }

    public static void o(){
        android.os.Trace.endSection();
    }
}


void test(){
    Trace.i("test");
    System.out.println("doSomething");
    Trace.o();
}
```

- BlockCanary：BlockCanary 可以监听主线程耗时的方法，就是在主线程消息循环打出日志的地入手, 当一个消息操作时间超过阀值后, 记录系统各种资源的状态, 并展示出来。所以我们将阈值设置低一点，这样的话如果一个方法执行时间超过200毫秒，获取堆栈信息。

而记录时间的方法就是通过looper()方法中循环去从MessageQueue中去取msg的时候，在dispatchMessage方法前后会有logging日志打印，所以只需要自定义一个Printer，重写println(String x)方法即可实现耗时统计了。



### 6. Bitmap优化

#### 1. 有做过什么Bitmap优化的实际经验



### 7. 线程优化
### 8. RecycleView优化

#### 

### 9.其他

#### 1. ANR了解过吗？有没有实际的ANR定位问题的经历

#### 2. 哪些原因会导致 oom？

**虚拟机堆内存不足**：内存泄漏（内存缓增）、大对象/大图片（内存突增）

**内存碎片，无足够连续内存空间**：循环中创建对象、字符串拼接...

**系统底层限制**：FD 数量超出限制、线程数量超出限制、其他系统限制

#### 3. 为什么会出现oom？

为了整个Android系统的内存控制需要，Android 系统为每一个应用程序都设置了一个硬性的Dalvik Heap Size 最大限制阈值，这个阈值在不同的设备上会因为 RAM 大小不同而各有差异。如果应用占用内存空间已经接近这个阈值，此时再尝试分配内存的话，很容易引起OutOfMemoryError 的错误。

#### debug 包有什么修改方式使不出现 oom？

Android为每个进程分配内存时，采用弹性的分配方式，即刚开始并不会给应用分配很多的内存，而是给每一个进程[分配一个“够用”的内存大小](https://www.jianshu.com/p/500ab0f48dc3)，这个值由具体的设备决定

在AndroidManifest.xml中的application标签中设置largeHeap为true，可以申请最多的内存的限制

这个内存限制的值是在 /system/build.prop文件中可以[查看与修改](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.droidviews.com%2Fedit-build-prop-file-on-android%2F)



## 六、插件化
#### 1. 为什么需要插件化？插件化的主要优点和缺点是什么？

我觉得最主要的原因是可以动态扩展功能。

把一些不常用的功能或者模块做成`插件`，就能减少原本的安装包大小，让一些功能以插件的形式在被需要的时候被加载，也就是实现了`动态加载`。

比如`动态换肤、节日促销、见不得人`的一些功能，就可以在需要的时候去下载相应模式的apk，然后再动态加载功能。所以一般这个功能适用于一些平台类的项目，比如大众点评美团这种，功能很多，用户很大概率只会用其中的一些功能，而且这些模块单独拿出来都可以作为一个app运行。但是现在用的却很少了。

**优点**



**缺点**



#### 2. 插件化的原理是怎样的？

要实现插件化，也就是实现从apk读取所有数据，要考虑三个问题：

- `读取插件代码`，完成插件中代码的加载和与主工程的互相调用
- `读取插件资源`，完成插件中资源的加载和与主工程的互相访问
- `四大组件管理`

1）读取插件代码，其实也就是进行插件中的类加载。所以用到类加载器就可以了。

Android中常用的有两种类加载器，`DexClassLoader`和`PathClassLoader`，它们都继承于`BaseDexClassLoader`。区别在于DexClassLoader多传了一个`optimizedDirectory`参数，表示缓存我们需要加载的dex文件的，并创建一个`DexFile`对象，而且这个路径必须为内部存储路径。而`PathClassLoader`这个参数为null，意思就是不会缓存到内部存储空间了，而是直接用原来的文件路径加载。所以`DexClassLoader`功能更为强大，可以加载外部的dex文件。

同时由于双亲委派机制，在构造插件的`ClassLoader`时会传入主工程的`ClassLoader`作为父加载器，所以插件是可以直接可以通过类名引用主工程的类。
 而主工程调用插件则需要通过`DexClassLoader`去加载类，然后反射调用方法。

2）读取插件资源，主要是通过`AssetManager`进行访问。具体代码如下：

```java
/**
 * 加载插件的资源：通过AssetManager添加插件的APK资源路径
 */
protected void loadPluginResources() {
    //反射加载资源
    try {
        AssetManager assetManager = AssetManager.class.newInstance();
        Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
        addAssetPath.invoke(assetManager, mDexPath);
        mAssetManager = assetManager;
    } catch (Exception e) {
        e.printStackTrace();
    }
    Resources superRes = super.getResources();
    mResources = new Resources(mAssetManager, superRes.getDisplayMetrics(), superRes.getConfiguration());
}   
```

通过addAssetPath方法把插件的路径穿进去，就可以访问到插件的资源了。

3）四大组件管理

 为什么单独说下四大组件呢？因为四大组件不仅要把他们的类加载出来，还要去管理他们的生命周期，在`AndroidManifest.xml`中注册。这也是插件化中比较重要的一部分。这里重点说下Activity。

主要实现方法是通过Hook技术，主要的方案就是先用一个在`AndroidManifest.xml`中注册的Activity来进行占坑，用来通过AMS的校验，接着在合适的时机用插件`Activity`替换占坑的`Activity`。

> Hook 技术又叫做钩子函数，在系统没有调用该函数之前，钩子程序就先捕获该消息，钩子函数先得到控制权，这时钩子函数既可以加工处理（改变）该函数的执行行为，还可以强制结束消息的传递。简单来说，就是把系统的程序拉出来变成我们自己执行代码片段。

这里的hook其实就是我们常说的下钩子，可以改变函数的内部行为。

这里加载插件Activity用到hook技术，有两个可以hook的点，分别是：

- Hook IActivityManager
   上面说了，首先会在AndroidManifest.xml中注册的Activity来进行占坑，然后合适的时机来替换我们要加载的Activity。所以我们主要需要两步操作：
   `第一步`：使用占坑的这个Activity完成AMS验证。
   也就是让AMS知道我们要启动的Activity是在xml里面注册过的哦。具体代码如下：

```java
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            if ("startActivity".contains(method.getName())) {
                //换掉
                Intent intent = null;
                int index = 0;
                for (int i = 0; i < args.length; i++) {
                    Object arg = args[i];
                    if (arg instanceof Intent) {
                        //说明找到了startActivity的Intent参数
                        intent = (Intent) args[i];
                        //这个意图是不能被启动的，因为Acitivity没有在清单文件中注册
                        index = i;
                    }
                }
               //伪造一个代理的Intent，代理Intent启动的是proxyActivity
                Intent proxyIntent = new Intent();
                ComponentName componentName = new ComponentName(context, proxyActivity);
                proxyIntent.setComponent(componentName);
                proxyIntent.putExtra("oldIntent", intent);
                args[index] = proxyIntent;
            }

            return method.invoke(iActivityManagerObject, args);
        }
```

`第二步`：替换回我们的Activity。
 上面一步是把我们实际要启动的Activity换成了我们xml里面注册的activity来躲过验证，那么后续我们就需要把Activity换回来。
 Activity启动的最后一步其实是通过H（一个handler）中重写的handleMessage方法会对`LAUNCH_ACTIVITY`类型的消息进行处理，最终会调用Activity的onCreate方法。最后会调用到Handler的`dispatchMessage`方法用于处理消息，如果Handler的Callback类型的`mCallback`不为null，就会执行mCallback的`handleMessage`方法。 所以我们能hook的点就是这个`mCallback`。



```java
 public static void hookHandler() throws Exception {
        Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
        Object currentActivityThread= FieldUtil.getField(activityThreadClass ,null,"sCurrentActivityThread");//1
        Field mHField = FieldUtil.getField(activityThread,"mH");//2
        Handler mH = (Handler) mHField.get(currentActivityThread);//3
        FieldUtil.setField(Handler.class,mH,"mCallback",new HCallback(mH));
    }

public class HCallback implements Handler.Callback{
    //...
    @Override
    public boolean handleMessage(Message msg) {
        if (msg.what == LAUNCH_ACTIVITY) {
            Object r = msg.obj;
            try {
                //得到消息中的Intent(启动SubActivity的Intent)
                Intent intent = (Intent) FieldUtil.getField(r.getClass(), r, "intent");
                //得到此前保存起来的Intent(启动TargetActivity的Intent)
                Intent target = intent.getParcelableExtra(HookHelper.TARGET_INTENT);
                //将启动SubActivity的Intent替换为启动TargetActivity的Intent
                intent.setComponent(target.getComponent());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        mHandler.handleMessage(msg);
        return true;
    }
}
```

用自定义的HCallback来替换mH中的`mCallback`即可完成Activity的替换了。

- Hook Instrumentation

这个方法是由于`startActivityForResult`方法中调用了Instrumentation的`execStartActivity`方法来激活Activity的生命周期，所以可以通过替换`Instrumentation`来完成，然后在`Instrumentation`的`execStartActivity`方法中用占坑`SubActivity`来通过AMS的验证，在`Instrumentation`的`newActivity`方法中还原TargetActivity。

```java
public class InstrumentationProxy extends Instrumentation {
    private Instrumentation mInstrumentation;
    private PackageManager mPackageManager;
    public InstrumentationProxy(Instrumentation instrumentation, PackageManager packageManager) {
        mInstrumentation = instrumentation;
        mPackageManager = packageManager;
    }
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        List<ResolveInfo> infos = mPackageManager.queryIntentActivities(intent, PackageManager.MATCH_ALL);
        if (infos == null || infos.size() == 0) {
            intent.putExtra(HookHelper.TARGET_INTENsT_NAME, intent.getComponent().getClassName());//1
            intent.setClassName(who, "com.example.liuwangshu.pluginactivity.StubActivity");//2
        }
        try {
            Method execMethod = Instrumentation.class.getDeclaredMethod("execStartActivity",
                    Context.class, IBinder.class, IBinder.class, Activity.class, Intent.class, int.class, Bundle.class);
            return (ActivityResult) execMethod.invoke(mInstrumentation, who, contextThread, token,
                    target, intent, requestCode, options);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public Activity newActivity(ClassLoader cl, String className, Intent intent) throws InstantiationException,
        IllegalAccessException, ClassNotFoundException {
        String intentName = intent.getStringExtra(HookHelper.TARGET_INTENT_NAME);
        if (!TextUtils.isEmpty(intentName)) {
            return super.newActivity(cl, intentName, intent);
        }
        return super.newActivity(cl, className, intent);
    }

}

  public static void hookInstrumentation(Context context) throws Exception {
        Class<?> contextImplClass = Class.forName("android.app.ContextImpl");
        Field mMainThreadField  =FieldUtil.getField(contextImplClass,"mMainThread");//1
        Object activityThread = mMainThreadField.get(context);//2
        Class<?> activityThreadClass = Class.forName("android.app.ActivityThread");
        Field mInstrumentationField=FieldUtil.getField(activityThreadClass,"mInstrumentation");//3
        FieldUtil.setField(activityThreadClass,activityThread,"mInstrumentation",new InstrumentationProxy((Instrumentation) mInstrumentationField.get(activityThread),
                context.getPackageManager()));
    }
```

#### 3. 有没有什么非运行时插件化的解决方案？

#### 4. 资源的插件化id重复如何解决？

#### 5.插件化换肤方案

#### 6. startActivity hook了哪个方法

#### 7. 市面上的一些插件化方案以及你的想法
前几年插件化还是很火的，比如Dynamic-Load-Apk（任玉刚），DroidPlugin，RePlugin（360），VirtualApk（滴滴），但是现在机会都没怎么在运营了，好多框架都最多只支持到Android9。

这是为什么呢？我觉得一个是维护成本太高难以兼容，每更新一次源码，就要重新维护一次。二就是确实插件化技术现在用的不多了，以前用插件化框架干嘛？主要是比如增加新的功能，让功能模块之间解耦。现在有RN可以进行插件化功能，有组件化可以进行项目解耦。所以用的人就不多咯。

虽然插件化用的不多了，但是我觉得技术还是可以了解的，而且热更新主要用的也是这些技术。方案可以被淘汰，但是技术不会。

## 七、组件化

#### 1. 组件化有详细了解过吗？

#### 2. ARouter详细原理

`ARouter`是阿里巴巴研发的一个用于解决组件间，模块间界面跳转问题的框架。

所以简单的说，就是用来跳转界面的，不同于平时用到的显式或隐式跳转，只需要在对应的界面上`添加注解`，就可以实现跳转，看个案例：

```java
@Route(path = "/test/activity")
public class YourActivity extend Activity {
    ...
}

//跳转
ARouter.getInstance().build("/test/activity").navigation();
```

使用很方便，通过一个`path`就可以进行跳转了，那么原理是什么呢？

其实仔细思考下，就可以联想到，既然关键跳转过程是通过`path`跳转到具体的`activity`，那么原理无非就是把`path`和`Activity`一一对应起来就行了。没错，其实就是通过注释，通过`apt`技术，也就是注解处理工具，把path和activity关联起来了。主要有以下几个步骤：

- 代码里加入的`@Route`注解，会在编译时期通过apt生成一些存储path和activity.class映射关系的类文件
- app进程启动的时候会加载这些类文件，把保存这些映射关系的数据读到内存里(保存在map里)
- 进行路由跳转的时候，通过`build()`方法传入要到达页面的路由地址，ARouter会通过它自己存储的路由表找到路由地址对应的Activity.class
- 然后`new Intent`方法，如果有调用`ARouter`的`withString()`方法，就会调用`intent.putExtra(String name, String value)`方法添加参数
- 最后调用`navigation()`方法，它的内部会调用startActivity(intent)进行跳转

#### 3.ARouter怎么实现接口调用

#### 4.ARouter怎么实现页面拦截

先说一个拦截器的案例，用作页面跳转时候检验是否登录，然后判断跳转到登录页面还是目标页面：

```java
   @Interceptor(name = "login", priority = 6)
    public class LoginInterceptorImpl implements IInterceptor {
        @Override
        public void process(Postcard postcard, InterceptorCallback callback) {
            String path = postcard.getPath();
            boolean isLogin = SPUtils.getInstance().getBoolean(ConfigConstants.SP_IS_LOGIN, false);
    
            if (isLogin) { 
                // 如果已经登录不拦截
                callback.onContinue(postcard);
            } else {  
                // 如果没有登录，进行拦截
                callback.onInterrupt(postcard);
            }
        }
    
        @Override
        public void init(Context context) {
            LogUtils.v("初始化成功"); 
        }
    }

    //使用
    ARouter.getInstance().build(ConfigConstants.SECOND_PATH)
                             .withString("msg", "123")
                              .navigation(this,new LoginNavigationCallbackImpl()); 
                              // 第二个参数是路由跳转的回调
    // 拦截的回调
    public class LoginNavigationCallbackImpl  implements NavigationCallback{
        @Override 
        public void onFound(Postcard postcard) {
        }
    
        @Override 
        public void onLost(Postcard postcard) {
        }
    
        @Override   
        public void onArrival(Postcard postcard) {
        }
    
        @Override
        public void onInterrupt(Postcard postcard) {
            //拦截并跳转到登录页
            String path = postcard.getPath();
            Bundle bundle = postcard.getExtras();
            ARouter.getInstance().build(ConfigConstants.LOGIN_PATH)
                    .with(bundle)
                    .withString(ConfigConstants.PATH, path)
                    .navigation();
        }
    }
```

拦截器实现`IInterceptor`接口，使用注解`@Interceptor`，这个拦截器就会自动被注册了，同样是使用APT技术自动生成映射关系类。这里还有一个优先级参数`priority`，数值越小，就会越先执行。

#### 5. 怎么应用到组件化中

首先，在公用组件的build.gradle中添加依赖：

```java
dependencies {
    api 'com.alibaba:arouter-api:1.4.0'
    annotationProcessor 'com.alibaba:arouter-compiler:1.2.1'
}
```

其次，必须在每个业务组件，也就是用到了`arouter`的组件中都声明`annotationProcessorOptions`，否则会无法通过apt生成索引文件，也就无法正常跳转了：

```java
//业务组件的build.gradle
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = [AROUTER_MODULE_NAME: project.getName()]
            }
        }
    }
}
dependencies {
    annotationProcessor 'com.alibaba:arouter-compiler:1.2.1'
    implementation '公用组件'
}
```

这个`arguments`是用来设置给编译处理器的一些参数，这里就把`[AROUTER_MODULE_NAME: project.getName()]`键值对传了过去，方便Arouter使用apt的时候进行数据处理，也是Arouter库所规定的配置。

然后就可以正常使用了。

#### 4.如果不用ARouter，你会怎么去解藕。接口？设计接口有什么需要注意的？



## 八、热修复

#### 1. tinker的原理是什么,还用过什么热修复框架，robust的原理是什么？

#### 2. 热修复的原理，资源的热修复的原理,会不会有资源冲突的问题



## 九、NDK



## 十、WebView

#### 1. h5与native通信你做过什么工作？



#### 2. webView加载本地图片，如何从安全方面考虑



#### 3. h5与native交互，webView.loadUrl与webView.evaluateUrl区别



#### 4. 有没有做过什么WebView秒开的一些优化



#### 5. native如何对h5进行鉴权，让某些页面可以调，某些页面不能调



#### 6. jsBridge实现方式

#### 7. 项目中对WebView的功能进行了怎样的增强

#### 8. 项目中的Webview与native通信

#### 9. 为什么不利用同步方法来做jsBridge交互？同步可以做异步，异步不能做同步

#### 10. webView与js通信

1） Android调用JS代码

**主要有两种方法：**

- 通过WebView的loadUrl（）

```java
// 调用javascript的callJS()方法
mWebView.loadUrl("javascript:callJS()");
```

但是这种不常用，因为它会自动刷新页面而且没有返回值，有点影响交互。

- 通过WebView的`evaluateJavascript（）`

```java
mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
```

这种就比较全面了。调用方法并且获取返回值。

2） JS调用Android端代码

**主要有两种方法：**

- 通过WebView的`addJavascriptInterface（）`进行对象映射

```java
public class AndroidtoJs extends Object {
    // 定义JS需要调用的方法
    // 被JS调用的方法必须加入@JavascriptInterface注解
    @JavascriptInterface
    public void hello(String msg) {
        System.out.println("JS调用了Android的hello方法");
    }
}

mWebView.addJavascriptInterface(new AndroidtoJs(), "test");

//js中：
function callAndroid(){
     // 由于对象映射，所以调用test对象等于调用Android映射的对象
     test.hello("js调用了android中的hello方法");
}
```

这种方法虽然很好用，但是要注意的是4.2以后，对于被调用的函数以`@JavascriptInterface`进行注解，否则容易出发漏洞，因为js方可以通过反射调用一些本地命令，很危险。

- 通过 WebViewClient 的`shouldOverrideUrlLoading ()`方法回调拦截 url

这种方法是通过`shouldOverrideUrlLoading`回调去拦截url，然后进行解析，如果是之前约定好的协议，就调用相应的方法。

```java
// 复写WebViewClient类的shouldOverrideUrlLoading方法
mWebView.setWebViewClient(new WebViewClient() {
        @Override
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            Uri uri = Uri.parse(url);                                 
            // 如果url的协议 = 预先约定的 js 协议
            if ( uri.getScheme().equals("js")) {
            // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议
                if (uri.getAuthority().equals("webview")) {
                    System.out.println("js调用了Android的方法");
                    // 可以在协议上带有参数并传递到Android上
                    HashMap<String, String> params = new HashMap<>();
                    Set<String> collection = uri.getQueryParameterNames();
                }
                return true;
            }
            return super.shouldOverrideUrlLoading(view, url);
            }
        }
    );
```

#### 11. 如何避免WebView内存泄露

WebView的内存泄露主要是因为在页面销毁后，WebView的资源无法马上释放所导致的。

现在主流的是两种方法：

1）不在xml布局中添加`webview`标签，采用在代码中new出来的方式，并在页面销毁的时候去释放`webview`资源

```java
//addview
private WeakReference<BaseWebActivity> webActivityReference = new WeakReference<BaseWebActivity>(this);
mWebView = new BridgeWebView(webActivityReference .get());
webview_container.addView(mWebView);


//销毁
ViewParent parent = mWebView.getParent();
if (parent != null) {
    ((ViewGroup) parent).removeView(mWebView);
}
mWebView.stopLoading();
mWebView.getSettings().setJavaScriptEnabled(false);
mWebView.clearHistory();
mWebView.clearView();
mWebView.removeAllViews();
mWebView.destroy()；
mWebView=null；
```

2）另起一个进程加载webview，页面销毁后干掉这个进程。但是这个方法的麻烦之处就在于`进程间通信`。

使用方法很简单，xml文件中写出进程名即可，销毁的时候调用`System.exit(0)`

```java
<activity android:name=".WebActivity"
   android:process=":remoteweb"/>

System.exit(0)   
```

#### 12. webView还有哪些可以优化的地方

- 提前初始化或者使用`全局WebView`。首次初始化WebView会比第二次初始化慢很多。初始化后，即使WebView已释放，但一些多WebView共用的全局服务/资源对想仍未释放，而第二次初始化不需要生成，因此初始化变快。
- DNS采用和客户端API相同的域名，`DNS解析`也是耗时比较多的部分，所以用客户端API相同的域名因为其DNS会被缓存，所以打开webView的时候就不会再耗时在DNS上了
- 对于JS的优化，尽量不要用`偏重的框架`，比如React。其次是高性能要求页面还是需要后端渲染。最后就是app中的网页框架要统一，这样就可以对js进行缓存和复用。

这里有美团团队的总结方案，如下：

- WebView初始化慢，可以在`初始化`同时先请求数据，让后端和网络不要闲着。
- 后端处理慢，可以让服务器`分trunk输出`，在后端计算的同时前端也加载网络静态资源。
- 脚本执行慢，就让`脚本在最后运行`，不阻塞页面解析。
- 同时，合理的`预加载、预缓存`可以让加载速度的瓶颈更小。
- WebView初始化慢，就随时`初始化`好一个WebView待用。
- DNS和链接慢，想办法复用客户端使用的`域名和链接`。
- 脚本执行慢，可以把`框架代码拆分`出来，在请求页面之前就执行好。



## 十一、架构

#### 1. MVC、MVP和MVVM是什么？

图片已有，不再给出

- MVC：Model-View-Controller，是一种分层解偶的框架，Model层提供本地数据和网络请求，View层处理视图，Controller处理逻辑，存在问题是Controller层和View层的划分不明显，Model层和View层的存在耦合。
- MVP：Model-View-Presenter，是对MVC的升级，Model层和View层与MVC的意思一致，但Model层和View层不再存在耦合，而是通过Presenter层这个桥梁进行交流。
- MVVM：Model-View-ViewModel，不同于上面的两个框架，ViewModel持有数据状态，当数据状态改变的时候，会自动通知View层进行更新。

#### 2. MVC和MVP的区别是什么？

MVP是MVC的进一步解耦，简单来讲，在MVC中，View层既可以和Controller层交互，又可以和Model层交互；而在MVP中，View层只能和Presenter层交互，Model层也只能和Presenter层交互，减少了View层和Model层的耦合，更容易定位错误的来源。

#### 3. MVVM和MVP的最大区别在哪？

MVP中的每个方法都需要你去主动调用，它其实是被动的，而MVVM中有数据驱动这个概念，当你的持有的数据状态发生变更的时候，你的View你可以监听到这个变化，从而主动去更新，这其实是主动的。



#### 5. 讲讲MVP模式中内存泄漏的问题，怎么处理内存泄漏



#### 6. 项目搭建过程中有什么经验,有用到什么gradle脚本，分包有做什么操作

#### 7. mainfest中配置LargeHeap，真的能分配到大内存吗？ 

#### 8. 如果叫你实现，你会怎样实现一个多主题的效果

#### 10. mvp与mvvm的区别，mvvm怎么更新UI

#### 11. 如何在网络框架里直接避免内存泄漏，不需要在presenter中释放订阅

## 十二、 Android 虚拟机 

#### 1. Android 虚拟机区别，编译区别，dex区别

## 十三、其他

#### 1. token放在本地如何保存？如何加密比较好？



#### 2. 说说你对屏幕刷新机制的了解，双重缓冲，三重缓冲，黄油模型



#### 3. 阿里编程规范不建议使用线程池，为什么？



#### 4. 你们项目的稳定性如何？有做过什么稳定性优化的工作？

#### 5. 推送sdk底层实现

#### 6. LeakCanary 的收集内存泄露是在 Activity 的什么时机，大致原理



#### ClassLoader 的双亲委派机制

[深入探讨 Java 类加载器](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fjava%2Fj-lo-classloader%2F) 

#### 什么情况会导致内存泄漏，如何修复？

#### 有没有做过UI方面的优化，做过哪些?

[Android性能优化（二）之布局优化面面观](https://www.jianshu.com/p/4f44a178c547)

- 调试GPU过度绘制，将Overdraw降低到合理范围内；
- 减少嵌套层次及控件个数，保持view的树形结构尽量扁平（使用Hierarchy Viewer可以方便的查看），同时移除所有不需要渲染的view；
- 使用GPU配置渲染工具，定位出问题发生在具体哪个步骤，使用TraceView精准定位代码；
- 使用标签，merge减少嵌套层次、viewStub延迟初始化、include布局重用 (与merge配合使用)

#### WebView 与 JS 交互方式，shouldOverrideUrlLoading、onJsPrompt使用有啥区别 

[最全面总结 Android WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)

#### 你们公司 Picasso 有使用过没，介绍下



#### Picasso 单引擎，在多 Bundle 的情况下怎么保证数据隔离的？



#### 介绍下 Binder 机制，与内存共享机制有什么区别？

- [为什么Android要采用Binder作为IPC机制？ - Gityuan的回答](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F39440766%2Fanswer%2F89210950)
- [Android匿名共享内存（Ashmem）原理](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F59e818bb6fb9a044fd10de38%23heading-1)
- [图文详解 Android Binder跨进程通信的原理](https://www.jianshu.com/p/4ee3fd07da14)

#### APK 的打包过程是什么？

aapt 工具打包资源文件，生成 R.java 文件

aidl 工具处理 AIDL 文件，生成对应的 .java 文件

javac 工具编译 Java 文件，生成对应的 .class 文件

把 .class 文件转化成 Davik VM 支持的 .dex 文件

apkbuilder 工具打包生成未签名的 .apk 文件

jarsigner 对未签名 .apk 文件进行签名

zipalign 工具对签名后的 .apk 文件进行对齐处理

#### APK 为什么要签名？是否了解过具体的签名机制？

Android 为了确认 apk 开发者身份和防止内容的篡改，设计了一套 apk 签名的方案保证 apk 的安全性，即在打包时由开发者进行 apk 的签名，在安装 apk 时Android 系统会有相应的开发者身份和内容正确性的验证，只有验证通过才可以安装 apk，签名过程和验证的设计就是基于非对称加密的思想。
 Android 在 7.0 以前使用的一套签名方案：在 apk 根目录下的 META-INF/ 文件夹下生成签名文件，然后在安装时在系统的 PackageManagerService 里进行签名文件的验证。
 从 7.0 开始，Android 提供了新的 V2 签名方案：利用 apk(zip) 压缩文件的格式，在几个原始内容区之外增加了一块用于存放签名信息的数据区，然后同样在安装时在系统的 PackageManagerService 里进行 V2 版本的签名验证，V2 方案会更安全、使校验更快安装更快。
 当然 V2 签名方案会向后兼容，如果没有使用 V2 签名就会默认走 V1 签名方案的验证过程。

#### 为什么要分 dex ？SDK 21 不分 dex，直接全部加载会不会有什么问题？

