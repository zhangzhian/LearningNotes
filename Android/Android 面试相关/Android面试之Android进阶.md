# Android进阶

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 一、Android FrameWork
### 1. Binder

#### (1) Binder的介绍？与其他IPC方式的优缺点？

Binder是Android中特有的IPC方式

> 从IPC角度来说，Binder是Android中的一种跨进程通信方式；Binder可以理解为虚拟的物理设备，它的设备驱动是/dev/binder；从`Android Framework`来讲，Binder是`Service Manager`连接各种`Manager`和对应的`ManagerService`的桥梁。从面向对象和CS模型来讲，`Client`通过Binder和远程的`Server`进行通讯。

基于Binder，Android还实现了其他的IPC方式，比如`AIDL`、`Messenger`和`ContentProvider`。

与其他IPC比较：

- 效率高：除了内存共享外，其他IPC都需要进行两次数据拷贝，而因为Binder使用内存映射的关系，仅需要一次数据拷贝。
- 安全性好：接收方可以从数据包中获取发送发的进程Id和用户Id，方便验证发送方的身份，其他IPC方式想要验证只能够主动存入，但是这有可能在发送的过程中被修改。
- 稳定性：基于C/S架构，职责明确、架构清晰，稳定性较好

#### (2) Binder的通信过程？Binder的原理？

Android 系统就可以通过动态添加一个内核模块运行在内核空间，用户进程之间通过这个内核模块作为桥梁来实现通信。

> 在 Android 系统中，这个运行在内核空间，负责各个用户进程通过 Binder 实现通信的内核模块就叫 **Binder 驱动**（Binder Dirver）。

在 Android 系统中用户进程之间是如何通过这Binder 驱动来实现通信的呢？**内存映射**。

Binder IPC 机制中涉及到的内存映射通过 mmap() 来实现，mmap() 是操作系统中一种内存映射的方法。内存映射简单的讲就是将用户空间的一块内存区域映射到内核空间。映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间；反之内核空间对这段区域的修改也能直接反应到用户空间。

内存映射能减少数据拷贝次数，实现用户空间和内核空间的高效互动。两个空间各自的修改能直接反映在映射的内存区域，从而被对方空间及时感知。也正因为如此，内存映射能够提供对进程间通信的支持。

Binder IPC 正是基于内存映射（mmap）来实现的，但是 mmap() 通常是用在有物理介质的文件系统上的。而 Binder 并不存在物理介质，因此 Binder 驱动使用 mmap() 并不是为了在物理介质和用户空间之间建立映射，而是用来在内核空间创建数据接收的缓存空间。

一次完整的 Binder IPC 通信过程通常是这样：

![img](https://pic4.zhimg.com/80/v2-cbd7d2befbed12d4c8896f236df96dbf_720w.jpg)

1. 首先 Binder 驱动在内核空间创建一个数据接收缓存区；
2. 接着在内核空间开辟一块内核缓存区，建立**内核缓存区**和**内核中数据接收缓存区**之间的映射关系，以及**内核中数据接收缓存区**和**接收进程用户空间地址**的映射关系；
3. 发送方进程通过系统调用 copy_from_user() 将数据 copy 到内核中的**内核缓存区**，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

Binder通信的实质是利用**内存映射**，将用户进程的内存地址和内核的内存地址映射为同一块物理地址，也就是说他们使用的同一块物理空间，每次创建Binder的时候大概分配128的空间。数据进行传输的时候，从这个内存空间分配一点，用完了再释放即可。

介绍完 Binder IPC 的底层通信原理，接下来我们看看实现层面是如何设计的。

Binder通信的四个角色：

- `Client`：服务的请求方

- `Server`：服务的提供方

- `Service Manager`：为`Server`提供`Binder`的注册服务，为`Client`提供`Binder`的查询服务，`Server`、`Client`和`Service Manage`r的通讯都是通过Binder

- `Binder驱动`：负责Binder通信机制的建立，提供一系列底层支持。

> Client、Server、ServiceManager、Binder 驱动这几个组件在通信过程中扮演的角色就如同互联网中服务器（Server）、客户端（Client）、DNS域名服务器（ServiceManager）以及路由器（Binder 驱动）之前的关系。

![img](https://pic3.zhimg.com/80/v2-729b3444cd784d882215a24067893d0e_720w.jpg)

Binder通信的过程是这样的：

1. Server在Service Manager中注册：Server进程在创建的时候，也会创建对应的Binder实体，如果要提供服务给Client，就必须为Binder实体注册一个名字。
2. Client通过Service Manager获取服务：Client知道服务中Binder实体的名字后，通过名字从Service Manager获取Binder实体的引用。
3. Client使用服务与Server进行通信：Client通过调用Binder实体与Server进行通信。

![img](https://pic4.zhimg.com/80/v2-67854cdf14d07a6a4acf9d675354e1ff_720w.jpg)

Binder 通信中的代理模式：前面我们介绍过跨进程通信的过程都有 Binder 驱动的参与，因此在数据流经 Binder 驱动的时候驱动会对数据做一层转换。当 A 进程想要获取 B 进程中的 object 时，驱动并不会真的把 object 返回给 A，而是返回了一个跟 object 看起来一模一样的代理对象 objectProxy，这个 objectProxy 具有和 object 一摸一样的方法，但是这些方法并没有 B 进程中 object 对象那些方法的能力，这些方法只需要把请求参数交给驱动即可。对于 A 进程来说和直接调用 object 中的方法是一样的。

当 Binder 驱动接收到 A 进程的消息后，发现这是个 objectProxy 就去查询自己维护的表单，一查发现这是 B 进程 object 的代理对象。于是就会去通知 B 进程调用 object 的方法，并要求 B 进程把返回结果发给自己。当驱动拿到 B 进程的返回结果后就会转发给 A 进程，一次通信就完成了。

![img](https://pic2.zhimg.com/80/v2-13361906ecda16e36a3b9cbe3d38cbc1_720w.jpg)



#### (3) Binder 怎么验证Pid? binder驱动了解吗？

```java
//作用是清空远程调用端的uid和pid，用当前本地进程的uid和pid替代；
public static final native long clearCallingIdentity();
//作用是恢复远程调用端的uid和pid信息，正好是`clearCallingIdentity`的反过程;
public static final native void restoreCallingIdentity(long token);
```

这两个方法涉及的uid和pid，每个线程都有自己独一无二的`IPCThreadState`对象，记录当前线程的pid和uid，可通过方法`Binder.getCallingPid()`和`Binder.getCallingUid()`获取相应的pid和uid。

`clearCallingIdentity()`, `restoreCallingIdentity()`这两个方法使用过程都是成对使用的，这两个方法配合使用，用于权限控制检测功能。

这两个方法是native方法，通过Binder的JNI调用，在`android_util_Binder.cpp`文件中定义了native方法所对应的jni方法。

```cpp
static jlong android_os_Binder_clearCallingIdentity(JNIEnv* env, jobject clazz)
{
    //调用IPCThreadState类的方法执行
    return IPCThreadState::self()->clearCallingIdentity();
}

int64_t IPCThreadState::clearCallingIdentity()
{
    int64_t token = ((int64_t)mCallingUid<<32) | mCallingPid;
    clearCaller();
    return token;
}

void IPCThreadState::clearCaller()
{
    mCallingPid = getpid(); //当前进程pid赋值给mCallingPid
    mCallingUid = getuid(); //当前进程uid赋值给mCallingUid
}
```

- mCallingUid(记为UID)，保存Binder IPC通信的调用方进程的Uid；
- mCallingPid(记为PID)，保存Binder IPC通信的调用方进程的Pid；

UID和PID是IPCThreadState的成员变量， 都是32位的int型数据，通过移位操作，将UID和PID的信息保存到`token`，其中高32位保存UID，低32位保存PID。然后调用clearCaller()方法将当前本地进程pid和uid分别赋值给PID和UID，最后返回`token`。

```cpp
static void android_os_Binder_restoreCallingIdentity(JNIEnv* env, jobject clazz, jlong token)
{
    //token记录着uid信息，将其右移32位得到的是uid
    int uid = (int)(token>>32);
    if (uid > 0 && uid < 999) {
        //目前Android中不存在小于999的uid，当uid<999则抛出异常。
        char buf[128];
        jniThrowException(env, "java/lang/IllegalStateException", buf);
        return;
    }
    //调用IPCThreadState类的方法执行
    IPCThreadState::self()->restoreCallingIdentity(token);
}

void IPCThreadState::restoreCallingIdentity(int64_t token)
{
    mCallingUid = (int)(token>>32);
    mCallingPid = (int)token;
}
```

从`token`中解析出PID和UID，并赋值给相应的变量。该方法正好是`clearCallingIdentity`的反过程。



线程A通过Binder远程调用线程B：则线程B的IPCThreadState中的`mCallingUid`和`mCallingPid`保存的就是线程A的UID和PID。这时在线程B中调用`Binder.getCallingPid()`和`Binder.getCallingUid()`方法便可获取线程A的UID和PID，然后利用UID和PID进行权限比对，判断线程A是否有权限调用线程B的某个方法。

线程B通过Binder调用当前线程的某个组件：此时线程B是线程B某个组件的调用端，则`mCallingUid`和`mCallingPid`应该保存当前线程B的PID和UID，故需要调用`clearCallingIdentity()`方法完成这个功能。当线程B调用完某个组件，由于线程B仍然处于线程A的被调用端，因此`mCallingUid`和`mCallingPid`需要恢复成线程A的UID和PID，这是调用`restoreCallingIdentity()`即可完成。

![binder_clearCallingIdentity](http://gityuan.com/images/binder/binder_clearCallingIdentity.jpg)

一句话：图中过程2（调用组件2开始之前）执行`clearCallingIdentity()`，过程3（调用组件2结束之后）执行`restoreCallingIdentity()`。

#### (4) Binder 进程间通信可以调用原进程方法吗？

可以的。调用原进程方法不走跨进程通信，直接调用。

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



### 3. 四大组件启动相关

#### (1) Activity的启动过程？

见下面APP启动过程。

#### (2) App的启动过程？

介绍一下App进程和System Server进程如何联系：

![App进程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6aaf5fe97?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- `ActivityThread`：依赖于Ui线程，实际处理与`AMS`中交互的工作。
- `ActivityManagerService`：负责`Activity`、`Service`等的生命周期工作。
- `ApplicationThread`：`System Server`进程中`ApplicatonThreadProxy`的服务端，帮助`System Server`进程跟App进程交流。
- `System Server`：Android核心的进程，掌管着Android系统中各种重要的服务。



![App启动流程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6bd3067d1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

具体过程：

① 点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；

② system_server进程接收到请求后，向zygote进程发送创建进程的请求；

③ Zygote进程fork出新的子进程，即App进程；

④ App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；

⑤ system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；

⑥ App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；

⑦ 主线程在收到Message后，通过反射机制创建目标Activity，并回调Activity.onCreate()等方法。

⑧ 到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。

**启动流程分析**：

**1.创建进程**

①先从Launcher的startActivity()方法，通过Binder通信，调用`ActivityManagerService#startActivity()`方法。

②一系列折腾，最后调用`startProcessLocked()`方法来创建新的进程。

③该方法会通过socket通道传递参数给Zygote进程。Zygote fork()自身。调用`ZygoteInit.main()`方法来实例化ActivityThread对象并最终返回新进程的pid。

④调用ActivityThread.main()方法，ActivityThread随后依次调用Looper.prepareLoop()和Looper.loop()来开启消息循环。

**方法调用流程图如下:**

![img](http://upload-images.jianshu.io/upload_images/3985563-25c23ee6ccb48048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**更直白的流程解释：**

![img](http://upload-images.jianshu.io/upload_images/3985563-ed91fd7c240e6bd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

①App发起进程：当从桌面启动应用，则发起进程便是Launcher所在进程；当从某App内启动远程进程 ，则发送进程便是该App所在进程。发起进程先通过binder发送消息给system_server进程；

②system_server进程：调用`Process.start()`方法，通过socket向zygote进程发送创建新进程的请求；

③zygote进程：在执行`ZygoteInit.main()`后便进入`runSelectLoop()`循环体内，当有客户端连接时便会执行`ZygoteConnection.runOnce()`方法，再经过层层调用后fork出新的应用进程；

④新进程：执行`handleChildProc()`方法，最后调用`ActivityThread.main()`方法。

**2.绑定Application**

上面创建进程后，执行`ActivityThread#main()`方法，随后调用`ActivityThread#attach()`方法。

将进程和指定的Application绑定起来。这个是通过ActivityThread对象中调用`bindApplication()`方法完成的。该方法发送一个`BIND_APPLICATION`的消息到消息队列中, 最终通过`handleBindApplication()`方法处理该消息. 然后调用`makeApplication()`方法来加载App的classes到内存中。

**方法调用流程图如下：**

![img](http://upload-images.jianshu.io/upload_images/3985563-0eb6b9d2b091de3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**更直白的流程解释：**

![img](http://upload-images.jianshu.io/upload_images/3985563-d8def9358f4646e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3.显示Activity界面**

经过前两个步骤之后, 系统已经拥有了该application的进程。 后面的调用顺序就是普通的从一个已经存在的进程中启动一个新进程的activity了。

实际调用方法是`realStartActivity()`, 它会调用application线程对象中的`scheduleLaunchActivity()`发送一个`LAUNCH_ACTIVITY`消息到消息队列中, 通过 `handleLaunchActivity()`来处理该消息。在 `handleLaunchActivity()`通过`performLaunchActiivty()`方法回调Activity的onCreate()方法和onStart()方法，然后通过`handleResumeActivity()`方法，回调`Activity的onResume()`方法，最终显示Activity界面。

![img](http://upload-images.jianshu.io/upload_images/3985563-5222775558226c7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**更直白的流程解释：**

![img](http://upload-images.jianshu.io/upload_images/3985563-5f711b4bca6bf21b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### (3) 登陆功能，登陆成功然后跳转到一个新Activity，中间涉及什么？从事件传递，网络请求, AMS交互角度分析



#### (4) AMS交互调用生命周期是顺序调用的吗？

不是。通过给H发Message。

#### (5) 说说App的启动过程,在ActivityThread的main方法里面做了什么事，什么时候启动第一个Activity？

```java
public static void main(String[] args){
    ...
    Looper.prepareMainLooper(); 
    //初始化Looper
    ...
    ActivityThread thread = new ActivityThread();
    //实例化一个ActivityThread
    thread.attach(false);
    //这个方法最后就是为了发送出创建Application的消息
    ... 
    Looper.loop();
    //主线程进入无限循环状态，等待接收消息
}
```

`thread.attach(false)`，App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；

system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；

App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；

主线程在收到Message后，通过反射机制创建目标Activity，并回调Activity.onCreate()等方法。

到此，第一个Activity启动。

#### (6) Launcher启动图标，有几个进程？

4个。Launcher，system_server，Zygote，APP。

### 4. Window

#### (1) Activity启动过程跟Window的关系？

> [《简析Window、Activity、DecorView以及ViewRoot之间的错综关系》](https://www.jianshu.com/p/8766babc40e0)

#### (2) Activity、Window、ViewRoot和DecorView之间的关系？

> [《总结UI原理和高级的UI优化方式》](https://juejin.im/post/6844903974294781965)

每个 Activity 包含了一个 Window对象，这个对象是由 PhoneWindow做的实现。而 PhoneWindow 将 DecorView作为了一个应用窗口的根 View，这个 DecorView 又把屏幕划分为了两个区域：一个是 TitleView，一个是ContentView，而我们平时在 Xml 文件中写的布局正好是展示在 ContentView 中的。

**Activity**

Activity并不负责视图控制，它只是控制生命周期和处理事件。真正控制视图的是Window。一个Activity包含了一个Window，Window才是真正代表一个窗口。**Activity就像一个控制器，统筹视图的添加与显示，以及通过其他回调方法，来与Window、以及View进行交互。**

**Window**

Window是视图的承载器，内部持有一个 DecorView，而这个DecorView才是 view 的根布局。Window是一个抽象类，实际在Activity中持有的是其子类PhoneWindow。PhoneWindow中有个内部类DecorView，通过创建DecorView来加载Activity中设置的布局`R.layout.activity_main`。Window 通过WindowManager将DecorView加载其中，并将DecorView交给ViewRoot，进行视图绘制以及其他交互。

**DecorView**

DecorView是FrameLayout的子类，它可以被认为是Android视图树的根节点视图。DecorView作为顶级View，一般情况下它内部包含一个竖直方向的LinearLayout，**在这个LinearLayout里面有上下三个部分，上面是个ViewStub，延迟加载的视图（应该是设置ActionBar，根据Theme设置），中间的是标题栏(根据Theme设置，有的布局没有)，下面的是内容栏。** 

**ViewRoot**

ViewRoot可能比较陌生，但是其作用非常重大。所有View的绘制以及事件分发等交互都是通过它来执行或传递的。

ViewRoot对应**ViewRootImpl类，它是连接WindowManagerService和DecorView的纽带**，View的三大流程（测量（measure），布局（layout），绘制（draw））均通过ViewRoot来完成。

ViewRoot并不属于View树的一份子。从源码实现上来看，它既非View的子类，也非View的父类，但是，它实现了ViewParent接口，这让它可以作为View的**名义上的父视图**。RootView继承了Handler类，可以接收事件并分发，Android的所有触屏事件、按键事件、界面刷新等事件都是通过ViewRoot进行分发的。

下面结构图可以清晰的揭示四者之间的关系：

![img](http://upload-images.jianshu.io/upload_images/3985563-e773ab2cb83ad214.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5. PMS

#### (1) Apk的安装过程？

*建议阅读：*

> [《Android Apk安装过程分析》](https://www.jianshu.com/p/953475cea991)

#### (2) APK 的打包过程是什么？

aapt 工具打包资源文件，生成 R.java 文件

aidl 工具处理 AIDL 文件，生成对应的 .java 文件

javac 工具编译 Java 文件，生成对应的 .class 文件

把 .class 文件转化成 Davik VM 支持的 .dex 文件

apkbuilder 工具打包生成未签名的 .apk 文件

jarsigner 对未签名 .apk 文件进行签名

zipalign 工具对签名后的 .apk 文件进行对齐处理

#### (3) APK 为什么要签名？是否了解过具体的签名机制？

Android 为了确认 apk 开发者身份和防止内容的篡改，设计了一套 apk 签名的方案保证 apk 的安全性，即在打包时由开发者进行 apk 的签名，在安装 apk 时Android 系统会有相应的开发者身份和内容正确性的验证，只有验证通过才可以安装 apk，签名过程和验证的设计就是基于非对称加密的思想。
 Android 在 7.0 以前使用的一套签名方案：在 apk 根目录下的 META-INF/ 文件夹下生成签名文件，然后在安装时在系统的 PackageManagerService 里进行签名文件的验证。
 从 7.0 开始，Android 提供了新的 V2 签名方案：利用 apk(zip) 压缩文件的格式，在几个原始内容区之外增加了一块用于存放签名信息的数据区，然后同样在安装时在系统的 PackageManagerService 里进行 V2 版本的签名验证，V2 方案会更安全、使校验更快安装更快。
 当然 V2 签名方案会向后兼容，如果没有使用 V2 签名就会默认走 V1 签名方案的验证过程。

#### (4) 为什么要分 dex ？SDK 21 不分 dex，直接全部加载会不会有什么问题？

为了避开单个dex方法数65536的限制。

当应用及其引用的库包含的方法数超过 65536 时，会遇到一个构建错误，指明应用已达到 Android 构建架构规定的引用限制：

```
trouble writing output:
Too many field references: 131000; max is 65536.
You may try using --multi-dex option.
```

较低版本的构建系统会报告一个不同的错误，但指示的是同一问题：

```
Conversion to Dalvik format failed:
Unable to execute dex: method ID not in [0, 0xffff]: 65536
```

这两种错误情况会显示一个共同的数字：65536。此数字是单个 Dalvik Executable (DEX) 字节码文件内的代码可调用的引用总数。

Android 5.0 之前版本的 MultiDex 支持：向模块级 `build.gradle` 文件中添加 MultiDex 库：

```
dependencies {
    def multidex_version = "2.0.1"
    implementation "androidx.multidex:multidex:$multidex_version"
}
```

如果您使用的不是 AndroidX，请改为添加以下已弃用的支持库依赖项：

```groovy
dependencies {
  implementation 'com.android.support:multidex:1.0.3'
}
```

Android 5.0（API 级别 21）及更高版本使用名为 ART 的运行时，它本身支持从 APK 文件加载多个 DEX 文件。ART 在应用安装时执行预编译，这会扫描查找 `classesN.dex` 文件，并将它们编译成单个 `.oat` 文件，以供 Android 设备执行。因此，如果 `minSdkVersion` 为 21 或更高版本，系统会默认启用 MultiDex，并且您不需要 MultiDex 库。

### 6. Context

#### (1) 关于Context的理解？

> [《Android Context 上下文 你必须知道的一切》](https://blog.csdn.net/lmj623565791/article/details/40481055)

#### (2) Android应用中哪些是 Context，一个应用有多少个 Context？

> [Android Context 熟悉还是陌生](https://www.jianshu.com/p/cc0bb2a71ee8)

Context的数量等于Activity的个数 + Service的个数 +1，这个1为Application。

#### (3) Context 的使用上如何避免内存泄漏？

- 不要让生命周期长于Activity的对象持有到Activity的引用
- 尽量使用Application的Context而不是Activity的Context
- 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用。如果使用静态内部类，将外部实例引用作为弱引用持有。

#### (4) 如何跨进程拿 Context？Activity 还没启动的时候如何拿 Context？

**getApplication 与 getApplicationContext 区别**：getApplication() 用用来获取 Application实例的，但这个方法只有在 Activity 和 Service 中才能调用。如果在一些其它的场景，比如BroadcastReceiver 中也想获得 Application 的实例，这时需要借助 getApplicationContext() 方法。也就是说，getApplicationContext() 方法的作用域会更广一些，任何一个 Context 的实例，只要调用 getApplicationContext() 方法都可以拿到我们的Application对象；

**Application 中方法的执行顺序为**：Application 构造方法 → attachBaseContext() → onCreate()。如果在 attachBaseContext() 中或执行前，调用 getApplicationContext() 得到的值为null

#### (5) ApplicationContext和ActivityContext的区别

这是两种不同的context，也是最常见的两种。第一种中context的生命周期与Application的生命周期相关的，context随着Application的销毁而销毁，伴随application的一生，与activity的生命周期无关.第二种中的context跟Activity的生命周期是相关的，但是对一个Application来说，Activity可以销毁几次，那么属于Activity的context就会销毁多次。至于用哪种context，得看应用场景。还有就是，在使用context的时候，小心内存泄露，防止内存泄露，注意一下几个方面：

- 不要让生命周期长的对象引用activity context，即保证引用activity的对象要与activity本身生命周期是一样的。
- 对于生命周期长的对象，可以使用application context。
- 避免非静态的内部类，尽量使用静态类，避免生命周期问题，注意内部类对外部对象引用导致的生命周期变化。

### 7. AIDL

#### (1) AIDL in out oneway代表什么意思？

in、out、inout表示跨进程通信中数据的流向（基本数据类型默认是in，非基本数据类型可以使用其它数据流向out、inout）。

- in参数使得实参顺利传到服务方，但服务方对实参的任何改变，不会反应回调用方。
- out参数使得实参不会真正传到服务方，只是传一个实参的初始值过去（这里实参只是作为返回值来使用的，这样除了return那里的返回值，还可以返回另外的东西），但服务方对实参的任何改变，在调用结束后会反应回调用方。
- inout参数则是上面二者的结合，实参会顺利传到服务方，且服务方对实参的任何改变，在调用结束后会反应回调用方。
- 其实inout，都是相对于服务方。in参数使得实参传到了服务方，所以是in进入了服务方；out参数使得实参在调用结束后从服务方传回给调用方，所以是out从服务方出来。

oneway可以用来修饰在interface之前，这样会造成interface内所有的方法都隐式地带上oneway；oneway也可以修饰在interface里的各个方法之前。

被oneway修饰了的方法不可以有返回值，也不可以有带out或inout的参数。

## 二、Android权限处理

#### 1. 解释一下 Android 程序运行时权限与文件系统权限的区别？

apk 程序是运行在虚拟机上的,对应的是 Android 独特的权限机制，只有体现到文件系统上时才使用 linux 的权限设置。

linux 文件系统上的权限如下表示 `-rwxr-x--x system system 4156 2010-04-30 16:13 test.apk`

Android 的权限规则:

a. Android 中的 apk 必须签名 

b. 基于 UserID 的进程级别的安全机制 

c .默认 apk 生成的数据对外是不可见的

d. AndroidManifest.xml 中的显式权限声明

从 Android6.0 之后，Android 升级了权限机制，提出了动态权限的概念。

#### 2. Android6.0 的权限机制?

1) 新权限思想 Android 的权限系统一直是首要的安全概念，因为这些权限只在安装的时候被询问一次。一旦安装了，app 可以 在用户毫不知晓的情况下访问权限内的所有东西，而且一般用户安装的时候很少会去仔细看权限列表，更不会去深入 了解这些权限可能带来的相关危害。 但是在 Android 6.0 版本之后，系统不会在软件安装的时候就赋予该 app 所有其申请的权限，对于一些危险级别的权限，app 需要在运行时一个一个询问用户授予权限。

2) 对旧版本 App 的兼容 

只有那些 targetSdkVersion 设置为 23 及以上的应用才会出现异常，在使用危险权限的时候系统必须要获得用户的同意才能使用，要不然应用就会崩溃， 出现类似下面的错误。 `java.lang.SecurityException: Permission Denial... `所以 targetSdkVersion 如果没有设置为 23 版本或者以上，系统还是会使用旧规则：在安装的时候赋予该 app 所 申请的所有权限。所以 app 当然可以和以前一样正常使用了，但是还有一点需要注意的是 6.0 的系统里面，用户可以手动将该 app 的权限关闭，在 App info 里面 Permissions 下边，可以关闭某个权限。如果以前的老应用申请的权限 被用户手动关闭了，不会抛出异常，不会崩溃，只不过调用那些被用户禁止权限的 api 接口返回值都为 null 或者 0， 所以我们只需要做一下判空操作就可以了，这是需要注意的。

3) 普通权限和危险权限列表

只有那些危险级别的权限才需要。如果有需要申请危险级别中的一个权限，就需要进行特殊操作。还有一个比较 人 性 的 地 方 就 是 如 果 同 一 组 的 任 何 一 个 权 限 被 授 权 了 ， 其 他 权 限 也 自 动 被 授 权 。 例 如 ， 一 旦 `WRITE_EXTERNAL_STORAGE` 被授权了，app 也有 `READ_EXTERNAL_STORAGE` 权限了。

4) 支持6.0新版本权限机制 在 Android M 的 api 中，可以通过 checkSelfPermission 检测软件是否有某一项权限，以及使用 requestPermissions 去请求一组权限。

向用户发起请求之后，请求完成，会有相对应的回调方法，通知软件用户是否授予了权限。 通过在 Activity 或者 Fragment 中重写 onRequestPermissionsResult 方法。



## 三、多线程断点续传
#### (1) 多线程断点续传？

基础知识：

- Http基础：在Http请求中，可以加入请求头`Range`，下载指定区间的文件数。
- `RandomAccessFile`：支持随机访问，可以从指定位置进行数据的读写。

有了这个基础以后，思路就清晰了：

1. 通过`HttpUrlConnection`获取文件长度。
2. 自己分配好线程进行制定区间的文件数据的下载。
3. 获取到数据流以后，使用`RandomAccessFile`进行指定位置的读写。

## 四、第三方库
### 1. RxJava

#### (1) RXJava怎么切换线程



#### (2) Rxjava自定义操作符



### 2. OkHttp

#### (1) OkHttp责任链模式

可以说是okhttp的精髓所在了，主要体现就是拦截器的使用，具体代码可以看下面的拦截器介绍。

#### (2) interceptors和networkInterceptors的区别？

`addInterceptor(Interceptor)`，这是由开发者设置的，会按照开发者的要求，在所有的拦截器处理之前进行最早的拦截处理，比如一些公共参数，Header都可以在这里添加。

`networkInterceptors`，这里也是开发者自己设置的，所以本质上和第一个拦截器差不多，但是由于位置不同，所以用处也不同。这个位置添加的拦截器可以看到请求和响应的数据了，所以可以做一些网络调试。

#### (3) OkHttp怎么实现连接池?里面怎么处理SSL？

- 为什么需要连接池？

频繁的进行建立`Sokcet`连接和断开`Socket`是非常消耗网络资源和浪费时间的，所以HTTP中的`keepalive`连接对于降低延迟和提升速度有非常重要的作用。`keepalive机制`是什么呢？也就是可以在一次TCP连接中可以持续发送多份数据而不会断开连接。所以连接的多次使用，也就是复用就变得格外重要了，而复用连接就需要对连接进行管理，于是就有了连接池的概念。

OkHttp中使用`ConectionPool`实现连接池，默认支持5个并发`KeepAlive`，默认链路生命为5分钟。

- 怎么实现的？

1）首先，`ConectionPool`中维护了一个双端队列`Deque`，也就是两端都可以进出的队列，用来存储连接。
2）然后在`ConnectInterceptor`，也就是负责建立连接的拦截器中，首先会找可用连接，也就是从连接池中去获取连接，具体的就是会调用到`ConectionPool`的get方法。

```java
RealConnection get(Address address, StreamAllocation streamAllocation, Route route) {
    assert (Thread.holdsLock(this));
    for (RealConnection connection : connections) {
      if (connection.isEligible(address, route)) {
        streamAllocation.acquire(connection, true);
        return connection;
      }
    }
    return null;
  }
```

也就是遍历了双端队列，如果连接有效，就会调用acquire方法计数并返回这个连接。

3）如果没找到可用连接，就会创建新连接，并会把这个建立的连接加入到双端队列中，同时开始运行线程池中的线程，其实就是调用了`ConectionPool`的put方法。

```java
public final class ConnectionPool {
    void put(RealConnection connection) {
        if (!cleanupRunning) {
            //没有连接的时候调用
            cleanupRunning = true;
            executor.execute(cleanupRunnable);
        }
        connections.add(connection);
    }
}
```

3）其实这个线程池中只有一个线程，是用来清理连接的，也就是上述的`cleanupRunnable`

```java
private final Runnable cleanupRunnable = new Runnable() {
        @Override
        public void run() {
            while (true) {
                //执行清理，并返回下次需要清理的时间。
                long waitNanos = cleanup(System.nanoTime());
                if (waitNanos == -1) return;
                if (waitNanos > 0) {
                    long waitMillis = waitNanos / 1000000L;
                    waitNanos -= (waitMillis * 1000000L);
                    synchronized (ConnectionPool.this) {
                        //在timeout时间内释放锁
                        try {
                            ConnectionPool.this.wait(waitMillis, (int) waitNanos);
                        } catch (InterruptedException ignored) {
                        }
                    }
                }
            }
        }
    };
```

这个`runnable`会不停的调用cleanup方法清理线程池，并返回下一次清理的时间间隔，然后进入wait等待。

怎么清理的呢？看看源码：

```java
long cleanup(long now) {
    synchronized (this) {
      //遍历连接
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();

        //检查连接是否是空闲状态，
        //不是，则inUseConnectionCount + 1
        //是 ，则idleConnectionCount + 1
        if (pruneAndGetAllocationCount(connection, now) > 0) {
          inUseConnectionCount++;
          continue;
        }

        idleConnectionCount++;

        // If the connection is ready to be evicted, we're done.
        long idleDurationNs = now - connection.idleAtNanos;
        if (idleDurationNs > longestIdleDurationNs) {
          longestIdleDurationNs = idleDurationNs;
          longestIdleConnection = connection;
        }
      }

      //如果超过keepAliveDurationNs或maxIdleConnections，
      //从双端队列connections中移除
      if (longestIdleDurationNs >= this.keepAliveDurationNs
          || idleConnectionCount > this.maxIdleConnections) {      
        connections.remove(longestIdleConnection);
      } else if (idleConnectionCount > 0) {      //如果空闲连接次数>0,返回将要到期的时间
        // A connection will be ready to evict soon.
        return keepAliveDurationNs - longestIdleDurationNs;
      } else if (inUseConnectionCount > 0) {
        // 连接依然在使用中，返回保持连接的周期5分钟
        return keepAliveDurationNs;
      } else {
        // No connections, idle or in use.
        cleanupRunning = false;
        return -1;
      }
    }

    closeQuietly(longestIdleConnection.socket());

    // Cleanup again immediately.
    return 0;
  }
```

也就是当如果空闲连接`maxIdleConnections`超过5个或者keepalive时间大于5分钟，则将该连接清理掉。

4）**这里有个问题，怎样属于空闲连接？**

其实就是有关刚才说到的一个方法`acquire`计数方法：

```java
  public void acquire(RealConnection connection, boolean reportedAcquired) {
    assert (Thread.holdsLock(connectionPool));
    if (this.connection != null) throw new IllegalStateException();

    this.connection = connection;
    this.reportedAcquired = reportedAcquired;
    connection.allocations.add(new StreamAllocationReference(this, callStackTrace));
  }
```

在`RealConnection`中，有一个`StreamAllocation`虚引用列表`allocations`。每创建一个连接，就会把连接对应的`StreamAllocationReference`添加进该列表中，如果连接关闭以后就将该对象移除。

5）连接池的工作就这么多，并不复杂，主要就是管理双端队列`Deque<RealConnection>`，可以用的连接就直接用，然后定期清理连接，同时通过对`StreamAllocation`的引用计数实现自动回收

#### (4) 如果让你来实现一个网络框架，你会考虑什么



#### (5) OKHttp有哪些拦截器，分别起什么作用？

`OKHTTP`的拦截器是把所有的拦截器放到一个list里，然后每次依次执行拦截器，并且在每个拦截器分成三部分：

- 预处理拦截器内容
- 通过`proceed`方法把请求交给下一个拦截器
- 下一个拦截器处理完成并返回，后续处理工作。

这样依次下去就形成了一个链式调用，看看源码，具体有哪些拦截器：

```java
  Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }
```

根据源码可知，一共七个拦截器：

- `addInterceptor(Interceptor)`，这是由开发者设置的，会按照开发者的要求，在所有的拦截器处理之前进行最早的拦截处理，比如一些公共参数，Header都可以在这里添加。
- `RetryAndFollowUpInterceptor`，这里会对连接做一些初始化工作，以及请求失败的充实工作，重定向的后续请求工作。跟他的名字一样，就是做重试工作还有一些连接跟踪工作。
- `BridgeInterceptor`，这里会为用户构建一个能够进行网络访问的请求，同时后续工作将网络请求回来的响应Response转化为用户可用的Response，比如添加文件类型，content-length计算添加，gzip解包。
- `CacheInterceptor`，这里主要是处理cache相关处理，会根据OkHttpClient对象的配置以及缓存策略对请求值进行缓存，而且如果本地有了可⽤的Cache，就可以在没有网络交互的情况下就返回缓存结果。
- `ConnectInterceptor`，这里主要就是负责建立连接了，会建立TCP连接或者TLS连接，以及负责编码解码的HttpCodec
- `networkInterceptors`，这里也是开发者自己设置的，所以本质上和第一个拦截器差不多，但是由于位置不同，所以用处也不同。这个位置添加的拦截器可以看到请求和响应的数据了，所以可以做一些网络调试。
- `CallServerInterceptor`，这里就是进行网络数据的请求和响应了，也就是实际的网络I/O操作，通过socket读写数据。

#### (6) OkHttp里面用到了什么设计模式

- 责任链模式

这个不要太明显，可以说是okhttp的精髓所在了，主要体现就是拦截器的使用，具体代码可以看看上述的拦截器介绍。

- 建造者模式

在Okhttp中，建造者模式也是用的挺多的，主要用处是将对象的创建与表示相分离，用Builder组装各项配置。
 比如Request：

```java
public class Request {
  public static class Builder {
    @Nullable HttpUrl url;
    String method;
    Headers.Builder headers;
    @Nullable RequestBody body;
    public Request build() {
      return new Request(this);
    }
  }
}
```

- 工厂模式

工厂模式和建造者模式类似，区别就在于工厂模式侧重点在于对象的生成过程，而建造者模式主要是侧重对象的各个参数配置。
 例子有CacheInterceptor拦截器中又个CacheStrategy对象：



```java
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();

    public Factory(long nowMillis, Request request, Response cacheResponse) {
      this.nowMillis = nowMillis;
      this.request = request;
      this.cacheResponse = cacheResponse;

      if (cacheResponse != null) {
        this.sentRequestMillis = cacheResponse.sentRequestAtMillis();
        this.receivedResponseMillis = cacheResponse.receivedResponseAtMillis();
        Headers headers = cacheResponse.headers();
        for (int i = 0, size = headers.size(); i < size; i++) {
          String fieldName = headers.name(i);
          String value = headers.value(i);
          if ("Date".equalsIgnoreCase(fieldName)) {
            servedDate = HttpDate.parse(value);
            servedDateString = value;
          } else if ("Expires".equalsIgnoreCase(fieldName)) {
            expires = HttpDate.parse(value);
          } else if ("Last-Modified".equalsIgnoreCase(fieldName)) {
            lastModified = HttpDate.parse(value);
            lastModifiedString = value;
          } else if ("ETag".equalsIgnoreCase(fieldName)) {
            etag = value;
          } else if ("Age".equalsIgnoreCase(fieldName)) {
            ageSeconds = HttpHeaders.parseSeconds(value, -1);
          }
        }
      }
    }
```

- 观察者模式

Okhttp中websocket的使用，由于webSocket属于长连接，所以需要进行监听，这里是用到了观察者模式：

```java
  final WebSocketListener listener;
  @Override public void onReadMessage(String text) throws IOException {
    listener.onMessage(this, text);
  }
```

- 单例模式

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

**Glide：**

- 多种图片格式的缓存，适用于更多的内容表现形式（如Gif、WebP、缩略图、Video）
- 生命周期集成（根据Activity或者Fragment的生命周期管理图片加载请求）
- 高效处理Bitmap（bitmap的复用和主动回收，减少系统回收压力）
- 高效的缓存策略，灵活（Picasso只会缓存原始尺寸的图片，Glide缓存的是多种规格），加载速度快且内存开销小（默认Bitmap格式的不同，使得内存开销是Picasso的一半）

**Fresco：**

- 最大的优势在于5.0以下(最低2.3)的bitmap加载。在5.0以下系统，Fresco将图片放到一个特别的内存区域(Ashmem区)
- 大大减少OOM（在更底层的Native层对OOM进行处理，图片将不再占用App的内存）
- 适用于需要高性能加载大量图片的场景

#### (2) 如何设计一个图片加载框架？

#### (3) Glide缓存实现机制？

#### (4) Glide如何处理生命周期？

#### (5) 有用过Glide的什么深入的API，自定义model是在Glide的什么阶段



[《Glide最全解析》](https://blog.csdn.net/sinyu890807/category_9268670.html)
[《面试官：简历上最好不要写Glide，不是问源码那么简单》](https://juejin.im/post/6844903986412126216)

### 5. LeakCanary

#### (1) LeakCanary 的收集内存泄露是在 Activity 的什么时机，大致原理



### 6. Android Jepack(非必需)
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

#### 2. 该怎么发现和解决内存泄漏？

1、使用工具，比如`Memory Profiler`，可以查看app的内存实时情况，捕获堆转储，就生成了一个内存快照，`hprof`文件。通过查看文件，可以看到哪些类发生了内存泄漏。

2、使用库，比较出名的就是`LeakCanary`，导入库，然后运行后，就可以发现app内的内存泄漏情况。

这里说下`LeakCanary`的原理：

- 监听
   首先通过`ActivityLifecycleCallbacks`和`FragmentLifeCycleCallbacks`监听Activity和Fragment的生命周期。
- 判断
   然后在销毁的生命周期中判断对象是否被回收。弱引用在定义的时候可以指定引用对象和一个 `ReferenceQueue`，通过该弱引用是否被加入ReferenceQueue就可以判断该对象是否被回收。
- 分析
   最后通过haha库来分析`hprof`文件，从而找出类之前的引用关系。

#### 3. 内存泄漏有什么方式检测？用过哪些工具，其中的原理是什么？

[Java内存问题 及 LeakCanary 原理分析](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5ab8d3d46fb9a028ca52f813)：

基本原理：用ActivityLifecycleCallbacks接口来检测Activity生命周期，主要是在**onDestroy()**方法中，手动调用 GC，然后利用ReferenceQueue+WeakReference 监听对象回收情况 ，来判断是否有释放不掉的引用，再结合dump memory的hpof文件, 用[HaHa](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsquare%2Fhaha)分析出泄漏地方；

LeakCanary会单独开一进程，用来执行分析任务，和监听任务分开处理。Application中可通过processName判断是否是任务执行进程；

利用主线程空闲的时候执行检测任务，在MessageQueue中加入了一个IdleHandler来得到主线程空闲回调；

LeakCanary检测只针对Activiy里的相关对象。其他类无法使用，还得用MAT原始方法

#### 

### 4. 响应速度优化

#### 1. 有什么实际解决UI卡顿优化的经历



### 5. 启动优化

#### 1. 具体有哪些启动优化方法？如果首页就要用到的初始化？



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



### 9.其他

#### 1. 什么是ANR 如何避免它？有没有实际的ANR定位问题的经历

答：在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应（ANR：Application NotResponding）对话框。

用户可以选择让程序继续运行，但是，他们在使用你的应用程序时，并不希望每次都要处理这个对话框。因此，在程序里对响应性能的设计很重要这样，这样系统就不会显示ANR给用户。

不同的组件发生ANR的时间不一样，Activity是5秒，BroadCastReceiver是10秒，Service是20秒（均为前台）。

如果开发机器上出现问题，我们可以通过查看/data/anr/traces.txt即可，最新的ANR信息在最开始部分。

- 主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
- 主线程中存在耗时的计算
- 主线程中错误的操作，比如Thread.wait或者Thread.sleep等 Android系统会监控程序的响应状况，一旦出现下面两种情况，则弹出ANR对话框
- 应用在5秒内未响应用户的输入事件（如按键或者触摸）
- BroadcastReceiver未在10秒内完成相关的处理
- Service在特定的时间内无法处理完成 20秒

修正：

1、使用AsyncTask处理耗时IO操作。

2、使用Thread或者HandlerThread时，调用Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，因为默认Thread的优先级和主线程相同。

3、使用Handler处理工作线程结果，而不是使用Thread.wait()或者Thread.sleep()来阻塞主线程。

4、Activity的onCreate和onResume回调中尽量避免耗时的代码。
BroadcastReceiver中onReceive代码也要尽量减少耗时，建议使用IntentService处理。

解决方案：

将所有耗时操作，比如访问网络，Socket通信，查询大量SQL 语句，复杂逻辑计算等都放在子线程中去，然后通过handler.sendMessage、runonUIThread、AsyncTask、RxJava等方式更新UI。无论如何都要确保用户界面的流畅度。如果耗时操作需要让用户等待，那么可以在界面上显示度条。

[深入回答](http://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649493643&idx=1&sn=34b51d1f61bd2ecaa8fd0a2d39c4d1d1&chksm=8eec9b74b99b126246acc4547597dfe55c836b8f689b2d1a65bdf1ee2054ced2fc070bfa2678&mpshare=1&scene=24&srcid=0116vzNfMMv2dLizhAT8mEYq#rd)

#### 2. 哪些原因会导致 oom？

**虚拟机堆内存不足**：内存泄漏（内存缓增）、大对象/大图片（内存突增）

**内存碎片，无足够连续内存空间**：循环中创建对象、字符串拼接...

**系统底层限制**：FD 数量超出限制、线程数量超出限制、其他系统限制

#### 3. 为什么会出现oom？

为了整个Android系统的内存控制需要，Android 系统为每一个应用程序都设置了一个硬性的Dalvik Heap Size 最大限制阈值，这个阈值在不同的设备上会因为 RAM 大小不同而各有差异。如果应用占用内存空间已经接近这个阈值，此时再尝试分配内存的话，很容易引起OutOfMemoryError 的错误。

#### 4. debug 包有什么修改方式使不出现 oom？

Android为每个进程分配内存时，采用弹性的分配方式，即刚开始并不会给应用分配很多的内存，而是给每一个进程[分配一个“够用”的内存大小](https://www.jianshu.com/p/500ab0f48dc3)，这个值由具体的设备决定

在AndroidManifest.xml中的application标签中设置largeHeap为true，可以申请最多的内存的限制

这个内存限制的值是在 /system/build.prop文件中可以[查看与修改](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.droidviews.com%2Fedit-build-prop-file-on-android%2F)

#### 5. Android怎么加速启动Activity？

- onCreate() 中不执行耗时操作：把页面显示的 View 细分一下，放在 AsyncTask 里逐步显示，用 Handler 更好。这样用户的看到的就是有层次有步骤的一个个的 View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。
- 利用多线程的目的就是尽可能的减少 onCreate() 和 onReume() 的时间，使得用户能尽快看到页面，操作页面。
- 减少主线程阻塞时间。
- 提高 Adapter 和 AdapterView 的效率。
- 优化布局文件。

#### 6. 有没有做过UI方面的优化，做过哪些?

[Android性能优化（二）之布局优化面面观](https://www.jianshu.com/p/4f44a178c547)

- 调试GPU过度绘制，将Overdraw降低到合理范围内；
- 减少嵌套层次及控件个数，保持view的树形结构尽量扁平（使用Hierarchy Viewer可以方便的查看），同时移除所有不需要渲染的view；
- 使用GPU配置渲染工具，定位出问题发生在具体哪个步骤，使用TraceView精准定位代码；
- 使用标签，merge减少嵌套层次、viewStub延迟初始化、include布局重用 (与merge配合使用)



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

#### 13. WebView 与 JS 交互方式，shouldOverrideUrlLoading、onJsPrompt使用有啥区别 

[最全面总结 Android WebView与 JS 的交互方式](https://www.jianshu.com/p/345f4d8a5cfa)

（1）android 中利用 webview 调用网页上的 js 代码。 首先将 webview 控件的支持 js 的属性设置为 true，，然后通过 loadUrl 就可以 直接进行调用，如下所示：

```java
 mWebView.getSettings().setJavaScriptEnabled(true);
 mWebView.loadUrl("javascript:test()"); 
```

（2）网页上调用 android 中 java 代码的方法 在网页中调用 java 代码，需要在 webview 控件中添加 javascriptInterface。如 下所示： 

```java
mWebView.addJavascriptInterface(new Object() {
    public void clickOnAndroid() {
        mHandler.post(new Runnable() {
            public void run() {
                Toast.makeText(Test.this, “测试调用 java”，
                Toast.LENGTH_LONG).show();
            }
        });
    }
}, "demo");
```

在网页中，只需要像调用 js 方法一样，进行调用就可以

```xml
<div id='b'>
<aonclick="window.demo.clickOnAndroid()">b.c</a>
</div>
```

（3）Java 代码调用 js 并传参

首先需要带参数的 js 函数，如 function test(str)，然后只需在调用 js 时传入参 数即可，如下所示： `mWebView.loadUrl("javascript:test('aa')");`

（4）Js 中调用 java 函数并传参 首先一样需要带参数的函数形式，但需注意此处的参数需要 final 类型，即得到 以后不可修改，如果需要修改其中的值，可以先设置中间变量，然后进行修改。 如下所示：

```java
mWebView.addJavascriptInterface(new Object() {
    public void clickOnAndroid(final int i) {
        mHandler.post(new Runnable() {
        public void run() {
			int j = i;
            j++;
            Toast.makeText(Test.this, "测试调用 java" + String.valueOf(j), Toast.LENGTH_LONG).show();
		}
     });
    }
}, "demo");

```

然后在 html 页面中，利用如下代码
```xml
<div id='b'>
<aonclick="window.demo.clickOnAndroid(2)">b.c</a>
</div>
```

#### 14. 如何清除 webview 的缓存
webview 的缓存包括网页数据缓存（存储打开过的页面及资源）、H5 缓存（即 AppCache），webview 会将我们浏览过的网页 url 已经网页文件(css、图片、js 等)保存到数据库表中，如下；

```
/data/data/package_name/database/webview.db
/data/data/package_name/database/webviewCache.db
```

所以，我们只需要根据数据库里的信息进行缓存的处理即可。

#### 15. webview 播放视频，5.0 以上没有全屏播放按钮

实现全屏的时候把 webview 里的视频放到一个 View 里面，然后把 webview隐藏掉；即可实现全屏播放。

#### 16. Webview 中 是 如 何 控 制 显 示 加 载 完 成 的 进 度 条 的

在 WebView 的 setWebChromClient() 中 ， 重 写 WebChromClient 的 openDialog()和 closeDialog()方法；实现监听进度条的显示与关闭。

#### 17. @JavaScriptInterface为什么不通过多个方法来实现？

4.4以前谷歌的webview存在安全漏洞，网站可以通过js注入就可以随便拿到客户端的重要信息，甚至轻而易举的调用本地代码进行流氓行为，谷歌后来发现有此漏洞后，增加了防御措施，js调用本地代码，开发者必须在代码申明JavascriptInterface。

```java
@SuppressWarnings("javadoc")
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface JavascriptInterface {
}
```





## 十一、架构

#### 1. MVC、MVP和MVVM是什么？

- MVC：Model-View-Controller，是一种分层解偶的框架，Model层提供本地数据和网络请求，View层处理视图，Controller处理逻辑，存在问题是Controller层和View层的划分不明显，Model层和View层的存在耦合。
- MVP：Model-View-Presenter，是对MVC的升级，Model层和View层与MVC的意思一致，但Model层和View层不再存在耦合，而是通过Presenter层这个桥梁进行交流。
- MVVM：Model-View-ViewModel，不同于上面的两个框架，ViewModel持有数据状态，当数据状态改变的时候，会自动通知View层进行更新。

MVC:

- 视图层(View)
  对应于xml布局文件和java代码动态view部分
- 控制层(Controller)
  MVC中Android的控制层是由Activity来承担的，Activity本来主要是作为初始化页面，展示数据的操作，但是因为XML视图功能太弱，所以Activity既要负责视图的显示又要加入控制逻辑，承担的功能过多。
- 模型层(Model)
  针对业务模型，建立数据结构和相关的类，它主要负责网络请求，数据库处理，I/O的操作。

具有一定的分层，model彻底解耦，controller和view并没有解耦
层与层之间的交互尽量使用回调或者去使用消息机制去完成，尽量避免直接持有
controller和view在android中无法做到彻底分离，但在代码逻辑层面一定要分清
业务逻辑被放置在model层，能够更好的复用和修改增加业务。

MVP：

通过引入接口BaseView，让相应的视图组件如Activity，Fragment去实现BaseView，实现了视图层的独立，通过中间层Preseter实现了Model和View的完全解耦。MVP彻底解决了MVC中View和Controller傻傻分不清楚的问题，但是随着业务逻辑的增加，一个页面可能会非常复杂，UI的改变是非常多，会有非常多的case，这样就会造成View的接口会很庞大。

MVVM：

MVP中我们说过随着业务逻辑的增加，UI的改变多的情况下，会有非常多的跟UI相关的case，这样就会造成View的接口会很庞大。而MVVM就解决了这个问题，通过双向绑定的机制，实现数据和UI内容，只要想改其中一方，另一方都能够及时更新的一种设计理念，这样就省去了很多在View层中写很多case的情况，只需要改变数据就行。

MVVM与DataBinding的关系？

MVVM是一种思想，DataBinding是谷歌推出的方便实现MVVM的工具。

看起来MVVM很好的解决了MVC和MVP的不足，但是由于数据和视图的双向绑定，导致出现问题时不太好定位来源，有可能数据问题导致，也有可能业务逻辑中对视图属性的修改导致。如果项目中打算用MVVM的话可以考虑使用官方的架构组件ViewModel、LiveData、DataBinding去实现MVVM。

三者如何选择？

- 如果项目简单，没什么复杂性，未来改动也不大的话，那就不要用设计模式或者架构方法，只需要将每个模块封装好，方便调用即可，不要为了使用设计模式或架构方法而使用。
- 对于偏向展示型的app，绝大多数业务逻辑都在后端，app主要功能就是展示数据，交互等，建议使用mvvm。
- 对于工具类或者需要写很多业务逻辑app，使用mvp或者mvvm都可。

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

#### 1. DVM 和 JVM 的区别？

a) dvm 执行的是.dex 文件，而 jvm 执行的是.class。Android 工程编译后的所有.class 字节码会被 dex 工具抽 取到一个.dex 文件中。 

b) dvm 是基于寄存器的虚拟机 而 jvm 执行是基于虚拟栈的虚拟机。寄存器存取速度比栈快的多，dvm 可以根 据硬件实现最大的优化，比较适合移动设备。

c) .class 文件存在很多的冗余信息，dex 工具会去除冗余信息，并把所有的.class 文件整合到.dex 文件中。减少 了 I/O 操作，提高了类的查找速度。



## 十三、其他

#### 1. token放在本地如何保存？如何加密比较好？



#### 2. 说说你对屏幕刷新机制的了解，双重缓冲，三重缓冲，黄油模型



#### 3. 阿里编程规范不建议使用线程池，为什么？



#### 4. 你们项目的稳定性如何？有做过什么稳定性优化的工作？



#### 5. 推送sdk底层实现




