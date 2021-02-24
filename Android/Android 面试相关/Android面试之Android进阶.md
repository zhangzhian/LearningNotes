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

Binder IPC 机制中涉及到的内存映射通过 `mmap()` 来实现，`mmap()` 是操作系统中一种内存映射的方法。内存映射简单的讲就是将用户空间的一块内存区域映射到内核空间。映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间；反之内核空间对这段区域的修改也能直接反应到用户空间。

内存映射能减少数据拷贝次数，实现用户空间和内核空间的高效互动。两个空间各自的修改能直接反映在映射的内存区域，从而被对方空间及时感知。也正因为如此，内存映射能够提供对进程间通信的支持。

Binder IPC 正是基于内存映射来实现的，但是 `mmap()` 通常是用在有物理介质的文件系统上的。而 Binder 并不存在物理介质，因此 Binder 驱动使用 `mmap()` 并不是为了在物理介质和用户空间之间建立映射，而是用来在内核空间创建数据接收的缓存空间。

一次完整的 Binder IPC 通信过程通常是这样：

![img](https://pic4.zhimg.com/80/v2-cbd7d2befbed12d4c8896f236df96dbf_720w.jpg)

1. 首先 Binder 驱动在内核空间创建一个数据接收缓存区；
2. 接着在内核空间开辟一块内核缓存区，建立**内核缓存区**和**内核中数据接收缓存区**之间的映射关系，以及**内核中数据接收缓存区**和**接收进程用户空间地址**的映射关系；
3. 发送方进程通过系统调用 `copy_from_user()` 将数据 copy 到内核中的**内核缓存区**，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

Binder通信的实质是利用**内存映射**，将用户进程的内存地址和内核的内存地址映射为同一块物理地址，也就是说他们使用的同一块物理空间，每次创建Binder的时候大概分配128的空间。数据进行传输的时候，从这个内存空间分配一点，用完了再释放即可。

介绍完 Binder IPC 的底层通信原理，接下来我们看看实现层面是如何设计的。

Binder通信的四个角色：

- `Client`：服务的请求方

- `Server`：服务的提供方

- `Service Manager`：为`Server`提供`Binder`的注册服务，为`Client`提供`Binder`的查询服务，`Server`、`Client`和`Service Manager`的通讯都是通过`Binder`。

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



#### (3) Binder 怎么验证Pid? Binder驱动了解吗？

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

UID和PID是IPCThreadState的成员变量， 都是32位的int型数据，通过移位操作，将UID和PID的信息保存到`token`，其中高32位保存UID，低32位保存PID。然后调用`clearCaller()`方法将当前本地进程pid和uid分别赋值给PID和UID，最后返回`token`。

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

#### (5) 为什么选择Binder？

为什么选用Binder，在讨论这个问题之前，我们知道Android也是基于Linux内核，Linux现有的进程间通信手段有以下几种：

- 管道：在创建时分配一个page大小的内存，缓存区大小比较有限；

- 消息队列：信息复制两次，额外的CPU消耗；不合适频繁或信息量大的通信；

- 共享内存：无须复制，共享缓冲区直接附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决；

- 套接字：作为更通用的接口，传输效率低，主要用于不同机器或跨网络的通信；

- 信号量：常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。 不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等；

既然有现有的IPC方式，为什么重新设计一套Binder机制呢。主要是出于以下三个方面的考量：

效率：传输效率主要影响因素是内存拷贝的次数，拷贝次数越少，传输速率越高。

从Android进程架构角度分析：对于消息队列、Socket和管道来说，数据先从发送方的缓存区拷贝到内核开辟的缓存区中，再从内核缓存区拷贝到接收方的缓存区，一共两次拷贝，如图：

![img](https://api2.mubu.com/v3/document_image/42784aab-0633-44d0-9b28-432f46d582b5-2297223.jpg)

而对于Binder来说，数据从发送方的缓存区拷贝到内核的缓存区，而接收方的缓存区与内核的缓存区是映射到同一块物理地址的，节省了一次数据拷贝的过程，如图：

![img](https://api2.mubu.com/v3/document_image/1cd8d44f-1c0f-413f-8373-facffdb21798-2297223.jpg)

共享内存不需要拷贝，Binder的性能仅次于共享内存。

稳定性：上面说到共享内存的性能优于Binder，那为什么不采用共享内存呢，因为共享内存需要处理并发同步问题，容易出现死锁和资源竞争，稳定性较差。Socket虽然是基于C/S架构的，但是它主要是用于网络间的通信且传输效率较低。Binder基于C/S架构 ，Server端与Client端相对独立，稳定性较好。

安全性：传统Linux IPC的接收方无法获得对方进程可靠的UID/PID，从而无法鉴别对方身份；而Binder机制为每个进程分配了UID/PID，且在Binder通信时会根据UID/PID进行有效性检测。

#### (6) Binder机制的作用和原理？

Linux系统将一个进程分为用户空间和内核空间。对于进程之间来说，用户空间的数据不可共享，内核空间的数据可以共享，为了保证安全性和独立性，一个进程不能直接操作或者访问另一个进程，即Android的进程是相互独立、隔离的，这就需要跨进程之间的数据通信方式。普通的跨进程通信方式一般需要2次内存拷贝，如下图所示：

![img](https://api2.mubu.com/v3/document_image/9629827e-9eea-44a7-8e8b-c99b6fff5898-2297223.jpg)

一次完整的 Binder IPC 通信过程通常是这样：

- 首先 Binder 驱动在内核空间创建一个数据接收缓存区。

- 接着在内核空间开辟一块内核缓存区，建立内核缓存区和内核中数据接收缓存区之间的映射关系，以及内核中数据接收缓存区和接收进程用户空间地址的映射关系。

- 发送方进程通过系统调用 `copy_from_user()` 将数据 copy 到内核中的内核缓存区，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

![img](https://api2.mubu.com/v3/document_image/dc38f08c-463e-43ae-b7bc-f2518e98f8f8-2297223.jpg)

#### (7) Binder框架中ServiceManager的作用？

Binder框架 是基于 C/S 架构的。由一系列的组件组成，包括 Client、Server、ServiceManager、Binder驱动，其中 Client、Server、Service Manager 运行在用户空间，Binder 驱动运行在内核空间。如下图所示：

![img](https://api2.mubu.com/v3/document_image/d7473277-1e81-4f01-951e-22f468889f02-2297223.jpg)

Server&Client：服务器&客户端。在Binder驱动和Service Manager提供的基础设施上，进行Client-Server之间的通信。

ServiceManager（如同DNS域名服务器）服务的管理者，将Binder名字转换为Client中对该Binder的引用，使得Client可以通过Binder名字获得Server端中Binder实体的引用。

Binder驱动（如同路由器）：负责进程之间binder通信的建立，计数管理以及数据的传递交互等底层支持。

最后，结合总结图来综合理解一下：

![img](https://api2.mubu.com/v3/document_image/7e61e800-dcad-4f98-aa5b-035064b23847-2297223.jpg)

#### (8) 跨进程传递大内存数据如何做？

binder 肯定是不行的，因为映射的最大内存只有 1M，可以采用 binder + 匿名共享内存的形式，像跨进程传递大的 bitmap 需要打开系统底层的 ashmem 机制。

请按顺序仔细阅读下列文章提升对Binder机制的理解程度：

[写给 Android 应用工程师的 Binder 原理剖析](https://juejin.im/post/6844903589635162126)

[Binder学习指南](http://weishu.me/2016/01/12/binder-index-for-newer/)

[Binder设计与实现](https://blog.csdn.net/universus/article/details/6211589)

[老罗Binder机制分析系列或Android系统源代码情景分析Binder章节](https://blog.csdn.net/luoshengyang/article/details/6618363)

#### (9) Binder线程池默认最大数量?

首先要管理线程池就要知道池子有多大，应用程序通过 `BINDER_SET_MAX_THREADS` 告诉驱动，最多可以创建几个线程。以后每个线程在创建，进入主循环，退出主循环时，都要分别使用 `BC_REGISTER_LOOP`，`BC_ENTER_LOOP`，`BC_EXIT_LOOP` 告知驱动，以便驱动标记当前线程池中各个线程的状态。每当驱动接收完数据包，并且把数据包返回给读 Binder 线程的用户空间时，都要检查一下，线程池中是不是已经没有闲置线程了。如果是，并且线程总数还没有达到线程池设定的最大线程数，就会在当前读出的数据包后面再追加一条 `BR_SPAWN_LOOPER` 命令，告诉 Server 端，线程即将不够用了，请再启动一个新线程，否则下一个请求可能不能及时响应。新线程一启动，又会通过 `BC_xxx_LOOP` 等一系列命令告知驱动更新线程的状态。这样确保了只要线程池的线程数量没有耗尽，总是会有空闲的线程在等待队列中随时待命，及时处理请求，这个就是 Binder 机制线程池管理的基本流程。

可以通过 `BINDER_SET_MAX_THREADS` 命令来告知 Binder 驱动每个进程可以创建的最大的 Binder 线程的个数，一般来说这个值默认值为15，当然我们可以自己设定。但是要注意的是，这个不是说 Binder 线程池中最大的线程数目就是15个了，这个值仅仅是对Binder 驱动来说的，它只统计使用 `BC_REGISTER_LOOPER` 命令创建的线程个数，如果达到就不在创建了。

举个我们讨论的 MediaPlayerService 的例子：

```cpp
int main(int argc __unused, char **argv __unused)
{
    signal(SIGPIPE, SIG_IGN);
    sp<ProcessState> proc(ProcessState::self());
    sp<IServiceManager> sm(defaultServiceManager());
    ALOGI("ServiceManager: %p", sm.get());
    InitializeIcuOrDie();
    MediaPlayerService::instantiate();//注册多媒体服务
    ResourceManagerService::instantiate();
    registerExtensions();
    ProcessState::self()->startThreadPool();//启动Binder线程池
    IPCThreadState::self()->joinThreadPool();//当前线程加入到线程池
}
```

这个进程的线程池中最多可以有几个 Binder 线程呢？我们来计算下，因为在 ProcessState 初始化的过程中会调用 open_driver 函数，在这个函数中设置了最大线程数为15，所以这15个线程是保底的，另外 main 函数的最后二行代码的 startThreadPool，我们知道是新启动了一个线程，并设置为主线程，是通过 `BC_ENTER_LOOP` 来创建的，不计入15个之列，所以又可以创建一个线程；同时看最后一行代码 `IPCThreadState::self()->joinThreadPool()` 是把当前的线程添加到线程池中，joinThreadPool 默认参数为 true，所以也是主线程，也是通过 `BC_ENTER_LOOP` 命令来创建的，所以又可以创建一个线程了，所以这个Media服务进程最多可以有17个线程在工作。

从以上对 Binder 线程池数量的分析我们已经知道了 Binder 线程可以有三类，总结如下：

- Binder主线程：在进程的 main 函数中通过调用 `startThreadPool()` 函数创建的线程
- Binder普通线程：是由 Binder 驱动通过发送 `BR_SPAWN_LOOPER` 命令，然后应用进程调用 `spawnPooledThread` 函数创建的线程
- Binder其它线程：是调用 `IPC.joinThreadPool()`，将当前线程直接加入 Binder 线程队列的线程，例如 media 的主线程

####  (10) Binder线程池中如果满了，对待新来的任务，会如何处理？此时client端会是什么效果？

下一个请求可能不能及时响应。


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

#### (3) Android中进程和线程的关系？区别？

- 线程是CPU调度的最小单元，同时线程是一种有限的系统资源；而进程一般指一个执行单元，在PC和移动设备上指一个程序或者一个应用。

- 一般来说，一个App程序至少有一个进程，一个进程至少有一个线程，通俗来讲就是，在App这个工厂里面有一个进程，线程就是里面的生产线，但主线程（即主生产线）只有一条，而子线程（即副生产线）可以有多个。

- 进程有自己独立的地址空间，而进程中的线程共享此地址空间，都可以并发执行。

#### (4) 如何开启多进程？应用是否可以开启N个进程？

在AndroidManifest中给四大组件指定属性android:process开启多进程模式，在内存允许的条件下可以开启N个进程。

#### (5) 为何需要IPC？多进程通信可能会出现的问题？

- 所有运行在不同进程的四大组件（Activity、Service、Receiver、ContentProvider）共享数据都会失败，这是由于Android为每个应用分配了独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这会导致在不同的虚拟机中访问同一个类的对象会产生多份副本。比如常用例子（通过开启多进程获取更大内存空间、两个或者多个应用之间共享数据、微信全家桶）。

- 一般来说，使用多进程通信会造成如下几方面的问题:

  - 静态成员和单例模式完全失效：独立的虚拟机造成。

  - 线程同步机制完全失效：独立的虚拟机造成。

  - SharedPreferences的可靠性下降：这是因为Sp不支持两个进程并发进行读写，有一定几率导致数据丢失。

  - Application会多次创建：Android系统在创建新的进程时会分配独立的虚拟机，所以这个过程其实就是启动一个应用的过程，自然也会创建新的Application。

#### (6) Android中IPC方式、各种方式优缺点？

![img](https://api2.mubu.com/v3/document_image/40d5e692-a83d-41cc-9283-c3f2481ca830-2297223.jpg)

#### (7) 简述Android系统启动流程

如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_png/jfNJsrA21FJvM8ibETbuD2XeblOibqJyK2GgwnKYRfzhibLLhHkDnjWJYjAmYC2tovOjQW9ZAibXuyB1jichecgvYicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### (8) 第一个用户级进程是哪个？

**init进程**是Android系统中用户空间的第一个进程，是所有用户进程的鼻祖。

启动入口在system/core/init/init.cpp文件中，init进程中主要做了这些事：

- **孵化出用户守护进程**。守护进程就是运行在后台的特殊进程，它不存在控制终端，会周期性处理一些任务。比如logd进程，就是用来进行日志的读写操作。

- **启动了一些重要服务**。比如开机动画。

- **孵化了Zygote进程**。Zygote进程大家都或多或少了解一些了，我们所有的应用程序都是由它孵化出来的。

- **孵化了Media Server进程，用来启动和管理整个C++ framework**，比如相机服务（camera Service）。

#### (9) Zygote进程做了些什么工作？

- **创建服务端Socket**，为后续创建进程通信做准备。
- **加载虚拟机**。没错，在Zygote进程中，会去加载下层的虚拟机。
- **fork了System Server进程**。SystemServer进程是Zygote fork的第一个进程，负责启动和管理Java Framework层，包括ActivityManagerService，PackageManagerService，WindowManagerService、Binder线程池等等。
- **fork了第一个应用进程——Launcher**，以及后续的一些系统应用进程，这就到了最上面一层——应用层了。

#### (10) Activity启动流程中，大部分都是用Binder通讯，为啥跟Zygote通信的时候要用socket呢?

ServiceManager不能保证在zygote起来的时候已经初始化好，所以无法使用Binder。

Socket 的所有者是 root，只有系统权限用户才能读写，多了安全保障。

Binder工作依赖于多线程，但是fork的时候是不允许存在多线程的，多线程情况下进程fork容易造成死锁，所以就不用Binder了。

#### (11) 怎么理解ServiceManager

ServiceManager其实是为了管理系统服务而设置的一种机制，每个服务注册在ServiceManager中，由ServiceManager统一管理，我们可以通过服务名在ServiceManager中查询对应的服务代理，从而完成调用系统服务的功能。所以ServiceManager有点类似于DNS，可以把服务名称和具体的服务记录在案，供客户端来查找。

在我们这个AIDL的案例中，能直接获取到服务端的Service，也就直接能获取到服务端的代理类IMsgManager，所以就无需通过ServiceManager这一层来寻找服务了。

而且ServiceManager本身也运行在一个单独的线程，所以它本身也是一个服务端，客户端其实是先通过跨进程获取到ServiceManager的代理对象，然后通过ServiceManager代理对象再去找到对应的服务。

而ServiceManager就像我们刚才AIDL中的Service一样，是可以直接找到的，他的句柄永远是0，是一个“众所周知”的句柄，所以每个APP程序都可以通过binder机制在自己的进程空间中创建一个ServiceManager代理对象。

所以通过ServiceManager查找系统服务并调用方法的过程是进行了两次跨进程通信。

APP进程——>ServiceManager进程——>系统服务进程（比如ActivityManagerService）

![图片](https://mmbiz.qpic.cn/mmbiz_png/jfNJsrA21FJvM8ibETbuD2XeblOibqJyK2x8RnWpXnb7N0IvBmk0fbYVQ084T2Te5Jdvp13Az9iaKlhKBNlz5NDjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

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

①先从Launcher的`startActivity()`方法，通过Binder通信，调用`ActivityManagerService#startActivity()`方法。

②一系列折腾，最后调用`startProcessLocked()`方法来创建新的进程。

③该方法会通过socket通道传递参数给Zygote进程。Zygote `fork()`自身。调用`ZygoteInit.main()`方法来实例化ActivityThread对象并最终返回新进程的pid。

④调用`ActivityThread.main()`方法，ActivityThread随后依次调用`Looper.prepareLoop()`和`Looper.loop()`来开启消息循环。

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

AMS交互角度：同上

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

`thread.attach(false)`，App进程，通过Binder IPC向`sytem_server`进程发起`attachApplication`请求；

`system_server`进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送`scheduleLaunchActivity`请求；

App进程的binder线程（`ApplicationThread`）在收到请求后，通过`Handler`向主线程发送`LAUNCH_ACTIVITY`消息；

主线程在收到Message后，通过反射机制创建目标Activity，并回调`Activity.onCreate()`等方法。

到此，第一个Activity启动。

#### (6) Launcher启动图标APP，涉及到有几个进程？

4个。Launcher，system_server，Zygote，APP。

#### (7) 广播发送和接收的原理了解吗 ？

![img](https://user-gold-cdn.xitu.io/2019/3/8/1695c1aaeac26f8d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### (8) Android系统启动流程是什么？

（提示：init进程 -> Zygote进程 –> SystemServer进程 –> 各种系统服务 –> 应用进程）

Android系统启动的核心流程如下：

- **启动电源以及系统启动**：当电源按下时引导芯片从预定义的地方（固化在ROM）开始执行，加载引导程序BootLoader到RAM，然后执行。
- **引导程序BootLoader**：BootLoader是在Android系统开始运行前的一个小程序，主要用于把系统OS拉起来并运行。
- **Linux内核启动**：当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。当其完成系统设置时，会先在系统文件中寻找init.rc文件，并启动init进程。
- **init进程启动**：初始化和启动属性服务，并且启动Zygote进程。
- **Zygote进程启动**：创建JVM并为其注册JNI方法，创建服务器端Socket，启动SystemServer进程。
- **SystemServer进程启动**：启动Binder线程池和SystemServiceManager，并且启动各种系统服务。
- **Launcher启动**：被SystemServer进程启动的AMS会启动Launcher，Launcher启动后会将已安装应用的快捷图标显示到系统桌面上。

需要更详细的分析请查看以下系列文章：

[Android系统启动流程之init进程启动](https://jsonchao.github.io/2019/02/18/Android系统启动流程之init进程启动/)

[Android系统启动流程之Zygote进程启动](https://jsonchao.github.io/2019/02/24/Android系统启动流程之Zygote进程启动/)

[Android系统启动流程之SystemServer进程启动](https://jsonchao.github.io/2019/03/03/Android系统启动流之SystemServer进程启动/)

[Android系统启动流程之Launcher进程启动](https://jsonchao.github.io/2019/03/09/Android系统启动流程之Launcher进程启动/)

#### (9) 系统是怎么帮我们启动找到桌面应用的？

通过意图，PMS 会解析所有 apk 的 `AndroidManifest.xml` ，如果解析过会存到 `package.xml` 中不会反复解析，PMS 有了它就能找到了。

#### (10) 启动一个程序，可以主界面点击图标进入，也可以从一个程序中跳转过去，二者有什么区别？

是因为启动程序（主界面也是一个app），发现了在这个程序中存在一个设置为的activity, 所以这个launcher会把icon提出来，放在主界面上。当用户点击icon的时候，发出一个Intent：

```java
Intent intent = mActivity.getPackageManager().getLaunchIntentForPackage(packageName);
mActivity.startActivity(intent);   
```

从一个程序中跳过去可以跳到任意允许的页面。

唯一的一点不同的是从icon的点击启动的intent的action是相对单一的，从程序中跳转或者启动可能样式更多一些。本质是相同的。

#### (11) AMS家族重要术语解释。

1. **ActivityManagerServices**，简称AMS，服务端对象，负责系统中所有Activity的生命周期。

2. **ActivityThread**，App的真正入口。当开启App之后，调用main()开始运行，开启消息循环队列，这就是传说的UI线程或者叫主线程。与ActivityManagerService一起完成Activity的管理工作。

3. **ApplicationThread**，用来实现ActivityManagerServie与ActivityThread之间的交互。在ActivityManagerSevice需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通信。

4. **ApplicationThreadProxy**，是ApplicationThread在服务器端的代理，负责和客户端的ApplicationThread通信。AMS就是通过该代理与ActivityThread进行通信的。

5. **Instrumentation**，每一个应用程序只有一个Instrumetation对象，每个Activity内都有一个对该对象的引用，Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。

6. **ActivityStack**，Activity在AMS的栈管理，用来记录经启动的Activity的先后关系，状态信息等。通过ActivtyStack决定是否需要启动新的进程。

7. **ActivityRecord**，ActivityStack的管理对象，每个Acivity在AMS对应一个ActivityRecord，来记录Activity状态以及其他的管理信息。其实就是服务器端的Activit对象的映像。

8. **TaskRecord**，AMS抽象出来的一个“任务”的概念，是记录ActivityRecord的栈，一个“Task”包含若干个ActivityRecord。AMS用TaskRecord确保Activity启动和退出的顺序。

#### (12) App启动流程（Activity的冷启动流程）-- 同（2）

点击应用图标后会去启动应用的Launcher Activity，如果Launcer Activity所在的进程没有创建，还会创建新进程，整体的流程就是一个Activity的启动流程。

Activity的启动流程图（放大可查看）如下所示：

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ebde239d45d4c969ffa5673885a03ad~tplv-k3u1fbpfcp-zoom-1.image)

整个流程涉及的主要角色有：

- Instrumentation: 监控应用与系统相关的交互行为。
- AMS：组件管理调度中心，什么都不干，但是什么都管。
- ActivityStarter：Activity启动的控制器，处理Intent与Flag对Activity启动的影响，具体说来有：1 寻找符合启动条件的Activity，如果有多个，让用户选择；2 校验启动参数的合法性；3 返回int参数，代表Activity是否启动成功。
- ActivityStackSupervisior：这个类的作用你从它的名字就可以看出来，它用来管理任务栈。
- ActivityStack：用来管理任务栈里的Activity。
- ActivityThread：最终干活的人，Activity、Service、BroadcastReceiver的启动、切换、调度等各种操作都在这个类里完成。

> 这里单独提一下ActivityStackSupervisior，这是高版本才有的类，它用来管理多个ActivityStack，早期的版本只有一个ActivityStack对应着手机屏幕，后来高版本支持多屏以后，就有了多个ActivityStack，于是就引入了ActivityStackSupervisior用来管理多个ActivityStack。

整个流程主要涉及四个进程：

- 调用者进程，如果是在桌面启动应用就是Launcher应用进程。
- ActivityManagerService等待所在的System Server进程，该进程主要运行着系统服务组件。
- Zygote进程，该进程主要用来fork新进程。
- 新启动的应用进程，该进程就是用来承载应用运行的进程了，它也是应用的主线程（新创建的进程就是主线程），处理组件生命周期、界面绘制等相关事情。

有了以上的理解，整个流程可以概括如下：

- 点击桌面应用图标，Launcher进程将启动Activity（MainActivity）的请求以Binder的方式发送给了AMS。
- AMS接收到启动请求后，交付ActivityStarter处理Intent和Flag等信息，然后再交给ActivityStackSupervisior/ActivityStack 处理Activity进栈相关流程。同时以Socket方式请求Zygote进程fork新进程。
- Zygote接收到新进程创建请求后fork出新进程。
- 在新进程里创建ActivityThread对象，新创建的进程就是应用的主线程，在主线程里开启Looper消息循环，开始处理创建Activity。
- ActivityThread利用ClassLoader去加载Activity、创建Activity实例，并回调Activity的onCreate()方法，这样便完成了Activity的启动。

#### (13) ActivityThread工作原理?

#### (14) AMS是如何管理Activity的？

#### (15) Activity从创建到我们看到界面，发生了哪些事

首先是通过`setContentView`加载布局，这其中创建了一个`DecorView`，然后根据然后根据activity设置的主题（theme）或者特征（Feature）加载不同的根布局文件，最后再通过inflate方法加载`layoutResID`资源文件，其实就是解析了xml文件，根据节点生成了View对象。流程图：

![img](https:////upload-images.jianshu.io/upload_images/8690467-b6f7a0a37aba3ad0.image?imageMogr2/auto-orient/strip|imageView2/2/w/1161/format/webp)

其次就是进行view绘制到界面上，这个过程发生在`handleResumeActivity`方法中，也就是触发`onResume`的方法。在这里会创建一个`ViewRootImpl`对象，作为`DecorView`的`parent`然后对`DecorView`进行测量布局和绘制三大流程。流程图：

![img](https:////upload-images.jianshu.io/upload_images/8690467-33ae193228b4436a.image?imageMogr2/auto-orient/strip|imageView2/2/w/1188/format/webp)

#### (16) AMS是如何确认Application启动完成的？关键条件是什么？

Zygote返给AMS的pid。

应用的ActivityThread#main方法中会向AMS上报Application的binder对象。

#### (17) Application#constructor、Application#onCreate、Application#attach他们的执行顺序?

1,3,2

#### (18) ContentProvider具体实现?



#### (19) 判断Activity前台进程？



#### (20) 启动未注册的Activity？



### 4. Window

#### (1) Activity启动过程跟Window的关系？除了Activity还有别的方式显示Window出来么？

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

#### (3) 理解Window和WindowManager。

- Window用于显示View和接收各种事件，Window有三种型：应用Window(每个Activity对应一个Window)、子Widow(不能单独存在，附属于特定Window)、系统window(toast和状态栏)
- Window分层级，应用Window在1-99、子Window在1000-1999、系统Window在2000-2999，WindowManager提供了增改View的三个功能。
- Window是个抽象概念：每一个Window对应着一个ViewRootImpl，Window通过ViewRootImpl来和View建立联系，View是Window存在的实体，只能通过WindowManager来访问Window。
- WindowManager的实现是WindowManagerImpl，其再委托WindowManagerGlobal来对Window进行操作，其中有四种List分别储存对应的View、ViewRootImpl、WindowManger.LayoutParams和正在被删除的View。
- Window的实体是存在于远端的WindowMangerService，所以增删改Window在本端是修改上面的几个List然后通过ViewRootImpl重绘View，通过WindowSession(每Window个对应一个)在远端修改Window。
- Activity创建Window：Activity会在`attach()`中创建Window并设置其回调(`onAttachedToWindow()`、`dispatchTouchEvent())`，Activity的Window是由Policy类创建PhoneWindow实现的。然后通过`Activity#setContentView()`调用PhoneWindow的setContentView。

#### (4) WMS是如何管理Window的？

#### (5) Surface的作用是什么？它是何时初始化的？View绘制的数据是如何显示到屏幕上的？

#### (6) Activity#setContentView中的Xml文件是如何转化成View并显示到Activity中的?

LayoutInflater是如何把xml布局文件转换成View对象的（反射）？View树如何生成的？怎么优化？

#### (7) PhoneWindow是在哪里初始化的？

启动过程会执行到，收到`H.LAUCH_ACTIVITY`消息，执行ActivityThread的handleLaunchActivity方法，这里初始化了WindowManagerGlobal，也就是WindowManager实际操作Window的类，待会会看到：

```java
public Activity handleLaunchActivity(ActivityClientRecord r,
                                     PendingTransactionActions pendingActions, 
                                     Intent customIntent) {
    ...
    WindowManagerGlobal.initialize();
    ...
    final Activity a = performLaunchActivity(r, customIntent);
    ...
    return a;
}
```

`performLaunchActivity`方法中调用了`Activity#attach()`，在该方法中：

```Java
final void attach() {
    //初始化PhoneWindow
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
    mWindow.setWindowControllerCallback(mWindowControllerCallback);
    mWindow.setCallback(this);

    //和WindowManager关联
    mWindow.setWindowManager(
            (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
            mToken, mComponent.flattenToString(),
            (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);

    mWindowManager = mWindow.getWindowManager();
}
```

可以看到，在Activity的attach方法中，创建了PhoneWindow，并且设置了callback，windowManager。

这里的callback跟事件分发有关系，可以说是当前Activity和PhoneWindow建立的联系。

#### (8) 说下 Activity跟Window，View之间的关系？

- Activity在创建时会调用 `attach()` 方法初始化一个`PhoneWindow`(继承于Window)，**每一个Activity都包含了唯一一个PhoneWindow**

- Activity通过`setContentView`实际上是调用的 `getWindow().setContentView()`将View设置到PhoneWindow上，而PhoneWindow内部是通过 WindowManager 的`addView`、`removeView`、`updateViewLayout`这三个方法来管理View，**WindowManager本质是接口，最终由WindowManagerImpl实现**

- `WindowManager`为每个`Window`创建`Surface`对象，然后应用就可以通过这个`Surface`来绘制任何它想要绘制的东西。而对于`WindowManager`来说，这只不过是一块矩形区域而已

- `Surface`其实就是一个持有像素点矩阵的对象，这个像素点矩阵是组成显示在屏幕的图像的一部分。我们看到显示的每个`Window`（包括对话框、全屏的Activity、状态栏等）都有他自己绘制的`Surface`。而最终的显示可能存在`Window`之间遮挡的问题，此时就是通过`Surface Flinger`对象渲染最终的显示，使他们以正确的`Z-order`显示出来。一般`Surface`拥有一个或多个缓存（一般2个），通过双缓存来刷新，这样就可以一边绘制一边加新缓存。

- View是Window里面用于交互的UI元素。Window只attach一个View Tree（组合模式），当Window需要重绘（如，当View调用`invalidate`）时，最终转为Window的Surface，Surface被锁住（locked）并返回Canvas对象，此时View拿到Canvas对象来绘制自己。当所有View绘制完成后，Surface解锁（unlock），并且`post`到绘制缓存用于绘制，通过`Surface Flinger`来组织各个Window，显示最终的整个屏幕

#### (9) **Window是什么**

窗口。可以理解为手机上的整个画面，所有的视图都是通过Window呈现的，比如Activity、dialog都是附加在Window上的。Window类的唯一实现是PhoneWindow，这个名字就更加好记了吧，手机窗口呗。

那Window到底在哪里呢？我们看到的View是Window吗？是也不是。

如果说的只是Window概念的话，那可以说是的，View就是Window的存在形式，Window管理着View。

如果说是Window类的话，那确实不是View，唯一实现类PhoneWindow管理着当前界面上的View，包括根布局——DecorView，和其他子view的添加删除等等。

总结：Window是个概念性的东西，你看不到他，如果你能感知它的存在，那么就是通过View，所以View是Window的存在形式，有了View，你才感知到View外层有一个Window。

#### (10) WindowManager是什么？和WMS的关系？

WindowManager就是用来管理Window的，实现类为WindowManagerImpl，实际工作会委托给WindowManagerGlobal类中完成。

而具体的Window操作，WM会通过Binder告诉WMS，WMS做最后的真正操作Window的工作，会为这个Window分配Surface，并绘制到屏幕上。

#### (11) 怎么添加一个Window？

```java
WindowManager.LayoutParams windowParams = WindowManager.LayoutParams()
windowParams.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
windowParams.type = WindowManager.LayoutParams.TYPE_SYSTEM_DIALOG
Button btn = Button(this)
windowManager.addView(btn, windowParams)
```

简单贴了下代码，加了一个Button。

有的朋友可能会疑惑了，这明明是个Button，是个View啊，咋成了Window？

刚才说过了，View是Window的表现形式，在实际实现中，添加window其实就是添加了一个你看不到的window，并且里面有View才能让你感觉得到这个是一个Window。

所以通过windowManager添加的View其实就是添加Window的过程。

这其中还有两个比较重要的属性：`flags`和`type`。下面会依次说到。

#### (12) Window怎样可以显示到锁屏界面?

Window的flag可以控制Window的显示特性，也就是该怎么显示、touch事件处理、与设备的关系、等等。所以这里问的锁屏界面显示也是其中的一种Flag。

```java
// Window不需要获取焦点，也不接受各种输入事件。
public static final int FLAG_NOT_FOCUSABLE = 0x00000008;

// @deprecated Use {@link android.R.attr#showWhenLocked} or
// {@link android.app.Activity#setShowWhenLocked(boolean)} instead to prevent an
// unintentional double life-cycle event.


// 窗口可以在锁屏的 Window 之上显示
public static final int FLAG_SHOW_WHEN_LOCKED = 0x00080000;
```

#### (13) Window三种类型都存在的情况下，显示层级是怎样?

Type表示Window的类型，一共三种：

- 应用Window。对应着一个Activity，Window层级为1~99，在视图最下层。
- 子Window。不能单独存在，需要附属在特定的父Window之中(如Dialog就是子Window)，Window层级为1000~1999。
- 系统Window。需要声明权限才能创建的Window，比如Toast和系统状态栏，Window层级为2000-2999，处在视图最上层。

可以看到，区别就是有个Window层级（z-ordered），层级高的能覆盖住层级低的，离用户更近。

#### (14) Window就是指PhoneWindow吗？

如果指的Window类，那么PhoneWindow作为唯一实现类，一般指的就是PhoneWindow。

如果指的Window这个概念，那肯定不是指PhoneWindow，而是存在于界面上真实的View。当然也不是所有的View都是Window，而是通过WindowManager添加到屏幕的view才是Window，所以PopupWindow是Window，上述问题中添加的单个View也是Window。

#### (15) 要实现可以拖动的View该怎么做？

还是接着刚才的btn例子，如果要修改btn的位置，使用updateViewLayout即可，然后在onTouch方法中传入移动的坐标即可。

```java
btn.setOnTouchListener { v, event ->
    val index = event.findPointerIndex(0)
    when (event.action) {
        ACTION_MOVE -> {
            windowParams.x = event.getRawX(index).toInt()
            windowParams.y = event.getRawY(index).toInt()
            windowManager.updateViewLayout(btn, windowParams)
        }
        else -> {
        }
    }
    false
}
```

#### (16) Window的添加、删除和更新过程？

Window的操作都是通过WindowManager来完成的，而WindowManager是一个接口，他的实现类是WindowManagerImpl，并且全部交给WindowManagerGlobal来处理。下面具体说下addView，updateViewLayout，和removeView。

**addView**

```java
//WindowManagerGlobal.java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {

        if (parentWindow != null) {
            parentWindow.adjustLayoutParamsForSubWindow(wparams);
        }

            ViewRootImpl root;
            View panelParentView = null;

            root = new ViewRootImpl(view.getContext(), display);
            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);

            try {
                root.setView(view, wparams, panelParentView);
            } 
        }
    }

```

- 这里可以看到，创建了一个ViewRootImpl实例，这样就说明了每个Window都对应着一个ViewRootImpl。
- 然后通过add方法修改了WindowManagerGlobal中的一些参数，比如mViews—存储了所有Window所对应的View，mRoots——所有Window所对应的ViewRootImpl，mParams—所有Window对应的布局参数。
- 最后调用了ViewRootImpl的setView方法,继续看看。

```java

final IWindowSession mWindowSession;

mWindowSession = WindowManagerGlobal.getWindowSession();

public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
    //
    requestLayout();

    res = mWindowSession.addToDisplay(mWindow,);
}

```

setView方法主要完成了两件事，一是通过requestLayout方法完成异步刷新界面的请求，进行完整的view绘制流程。其次，会通过IWindowSession进行一次IPC调用，交给到WMS来实现Window的添加。

其中mWindowSession是一个Binder对象，相当于在客户端的代理类，对应的服务端的实现为Session，而Session就是运行在SystemServer进程中，具体就是处于WMS服务中，最终就会调用到这个Session的addToDisplay方法，从方法名就可以猜到这个方法就是具体添加Window到屏幕的逻辑，具体就不分析了，下次说到屏幕绘制的时候再细谈。

**updateViewLayout**

```java
public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
//...
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;

    view.setLayoutParams(wparams);

    synchronized (mLock) {
        int index = findViewLocked(view, true);
        ViewRootImpl root = mRoots.get(index);
        mParams.remove(index);
        mParams.add(index, wparams);
        root.setLayoutParams(wparams, false);
    }
}
```

这里更新了WindowManager.LayoutParams和ViewRootImpl.LayoutParams，然后在ViewRootImpl内部同样会重新对View进行绘制，最后通过IPC通信，调用到WMS的relayoutWindow完成更新。

**removeView**

```java
public void removeView(View view, boolean immediate) {
    if (view == null) {
        throw new IllegalArgumentException("view must not be null");
    }

    synchronized (mLock) {
        int index = findViewLocked(view, true);
        View curView = mRoots.get(index).getView();
        removeViewLocked(index, immediate);
        if (curView == view) {
            return;
        }

        throw new IllegalStateException("Calling with view " + view
                + " but the ViewAncestor is attached to " + curView);
    }
}


private void removeViewLocked(int index, boolean immediate) {
    ViewRootImpl root = mRoots.get(index);
    View view = root.getView();

    if (view != null) {
        InputMethodManager imm = view.getContext().getSystemService(InputMethodManager.class);
        if (imm != null) {
            imm.windowDismissed(mViews.get(index).getWindowToken());
        }
    }
    boolean deferred = root.die(immediate);
    if (view != null) {
        view.assignParent(null);
        if (deferred) {
            mDyingViews.add(view);
        }
    }
} 

```

该方法中，通过view找到mRoots中的对应索引，然后同样走到ViewRootImpl中进行View删除工作，通过die方法，最终走到dispatchDetachedFromWindow()方法中，主要做了以下几件事：

- 回调onDetachedFromeWindow。
- 垃圾回收相关操作。
- 通过Session的remove()在WMS中删除Window。
- 通过Choreographer移除监听器。

#### (17) Activity、PhoneWindow、DecorView、ViewRootImpl 的关系？

- PhoneWindow 其实是 Window 的唯一子类，是 Activity 和 View 交互系统的中间层，用来管理View的，并且在Window创建（添加）的时候就新建了ViewRootImpl实例。
- DecorView 是整个 View 层级的最顶层，ViewRootImpl是DecorView 的parent，但是他并不是一个真正的 View，只是继承了ViewParent接口，用来掌管View的各种事件，包括requestLayout、invalidate、dispatchInputEvent 等等。

#### (18) Window中的token是什么，有什么用？

比如application的上下文去创建dialog的时候，就会报错：

```
unable to add window --token null
```

所以这个token跟window操作是有关系的，翻到刚才的addview方法中，还有个细节我们没说到，就是adjustLayoutParamsForSubWindow方法。

```java
//Window.java
void adjustLayoutParamsForSubWindow(WindowManager.LayoutParams wp) {
    if (wp.type >= WindowManager.LayoutParams.FIRST_SUB_WINDOW &&
            wp.type <= WindowManager.LayoutParams.LAST_SUB_WINDOW) {
        //子Window
        if (wp.token == null) {
            View decor = peekDecorView();
            if (decor != null) {
                wp.token = decor.getWindowToken();
            }
        }
    } else if (wp.type >= WindowManager.LayoutParams.FIRST_SYSTEM_WINDOW &&
            wp.type <= WindowManager.LayoutParams.LAST_SYSTEM_WINDOW) {
        //系统Window
    } else {
        //应用Window
        if (wp.token == null) {
            wp.token = mContainer == null ? mAppToken : mContainer.mAppToken;
        }

    }
}
```

上述代码分别代表了三个Window的类型：

- 子Window。需要从DecorView中拿到token。
- 系统Window。不需要token。
- 应用Window。直接拿mAppToken，mAppToken是在setWindowManager方法中传进来的，也就是新建Window的时候就带进来了token。

然后在WMS中的addWindow方法会验证这个token。

所以这个token就是用来验证是否能够添加Window，可以理解为权限验证，其实也就是为了防止开发者乱用context创建window。

拥有token的context（比如Activity）就可以操作Window。没有token的上下文（比如Application）就不允许直接添加Window到屏幕（除了系统Window）。

#### (19) Application中可以直接弹出Dialog吗？

这个问题其实跟上述问题相关：

- 如果直接使用Application的上下文是不能创建Window的，而Dialog的Window等级属于子Window，必须依附与其他的父Window，所以必须传入Activity这种有window的上下文。
- 那有没有其他办法可以在Application中弹出dialog呢？有，改成系统级Window：

```java
//检查权限
if (!Settings.canDrawOverlays(this)) {
    val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION)
    intent.data = Uri.parse("package:$packageName")
    startActivityForResult(intent, 0)
}

dialog.window.setType(WindowManager.LayoutParams.TYPE_SYSTEM_DIALOG)

<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```

- 另外还有一种办法，在Application类中，可以通过registerActivityLifecycleCallbacks监听Activity生命周期，不过这种办法也是传入了Activity的context，只不过在Application类中完成这个工作。

#### (20) 关于事件分发，事件到底是先到DecorView还是先到Window的？

这里的window可以理解为PhoneWindow，其实这道题就是问事件分发在Activity、DecorView、PhoneWindow中的顺序。

当屏幕被触摸，首先会通过硬件产生触摸事件传入内核，然后走到FrameWork层，最后经过一系列事件处理到达ViewRootImpl的processPointerEvent方法，接下来就是我们要分析的内容了：

```java
//ViewRootImpl.java
 private int processPointerEvent(QueuedInputEvent q) {
            final MotionEvent event = (MotionEvent)q.mEvent;
            ...
            //mView分发Touch事件，mView就是DecorView
            boolean handled = mView.dispatchPointerEvent(event);
            ...
        }

//DecorView.java
    public final boolean dispatchPointerEvent(MotionEvent event) {
        if (event.isTouchEvent()) {
            //分发Touch事件
            return dispatchTouchEvent(event);
        } else {
            return dispatchGenericMotionEvent(event);
        }
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        //cb其实就是对应的Activity
        final Window.Callback cb = mWindow.getCallback();
        return cb != null && !mWindow.isDestroyed() && mFeatureId < 0
                ? cb.dispatchTouchEvent(ev) : super.dispatchTouchEvent(ev);
    }


//Activity.java
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }

//PhoneWindow.java
    @Override
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return mDecor.superDispatchTouchEvent(event);
    }

//DecorView.java
    public boolean superDispatchTouchEvent(MotionEvent event) {
        return super.dispatchTouchEvent(event);
    }    

```

事件的分发流程就比较清楚了：

ViewRootImpl—>DecorView—>Activity—>PhoneWindow—>DecorView—>ViewGroup

（这其中就用到了getCallback参数，也就是之前addView中传入的callback，也就是Activity本身）

但是这个流程确实有些奇怪，为什么绕来绕去的呢，光DecorView就走了两遍？主要原因就是解耦。

- ViewRootImpl并不知道有Activity，它只是持有了DecorView。所以先传给了DecorView，而DecorView知道有Activity，所以传给了Activity。
- Activity也不知道有DecorView，它只是持有PhoneWindow，所以这么一段调用链就形成了。



### 5. PMS

#### (1) Apk的安装过程？

建议阅读：[《Android Apk安装过程分析》](https://www.jianshu.com/p/953475cea991)

APK的安装流程如下所示：

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91d5374ba2064521bb17f6df9f7b628b~tplv-k3u1fbpfcp-zoom-1.image)

- 复制APK到/data/app目录下，解压并扫描安装包。

- 资源管理器解析APK里的资源文件。

- 解析AndroidManifest文件，并在/data/data/目录下创建对应的应用数据目录。

- 然后对dex文件进行优化，并保存在dalvik-cache目录下。

- 将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中。

- 安装完成后，发送广播。

#### (2) APK 的打包过程是什么？

- 通过AAPT工具进行资源文件（包括AndroidManifest.xml、布局文件、各种xml资源等）的打包，生成R.java文件。

- 通过AIDL工具处理AIDL文件，生成相应的Java文件。

- 通过Java Compiler编译R.java、Java接口文件、Java源文件，生成.class文件。

- 通过dex命令，将.class文件和第三方库中的.class文件处理生成classes.dex，该过程主要完成Java字节码转换成Dalvik字节码，压缩常量池以及清除冗余信息等工作。

- 通过ApkBuilder工具将资源文件、DEX文件打包生成APK文件。

- 通过Jarsigner工具，利用KeyStore对生成的APK文件进行签名。

如果是正式版的APK，还会利用ZipAlign工具进行对齐处理，对齐的过程就是将APK文件中所有的资源文件距离文件的起始距位置都偏移4字节的整数倍，这样通过内存映射访问APK文件的速度会更快，并且会减少其在设备上运行时的内存占用。

#### apk组成

- dex：最终生成的Dalvik字节码。
- res：存放资源文件的目录。
- asserts：额外建立的资源文件夹。
- lib：如果存在的话，存放的是ndk编出来的so库。
- META-INF：存放签名信息

  - MANIFEST.MF（清单文件）：其中每一个资源文件都有一个SHA-256-Digest签名，MANIFEST.MF文件的SHA256（SHA1）并base64编码的结果即为CERT.SF中的SHA256-Digest-Manifest值。
  - CERT.SF（待签名文件）：除了开头处定义的SHA256（SHA1）-Digest-Manifest值，后面几项的值是对MANIFEST.MF文件中的每项再次SHA256并base64编码后的值。
  - CERT.RSA（签名结果文件）：其中包含了公钥、加密算法等信息。首先对前一步生成的MANIFEST.MF使用了SHA256（SHA1）-RSA算法，用开发者私钥签名，然后在安装时使用公钥解密。最后，将其与未加密的摘要信息（MANIFEST.MF文件）进行对比，如果相符，则表明内容没有被修改。

- AndroidManifes.xml：程序的全局清单配置文件。
- resources.arsc：编译后的二进制资源文件。

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

#### (5) v3签名key和v2还有v1有什么区别

在**v1版本**的签名中，签名以文件的形式存在于apk包中，这个版本的apk包就是一个标准的zip包，**v2**和**v1**的差别是**v2**是对整个zip包进行签名，而且在zip包中增加了一个**apk signature block**，里面保存签名信息。

![img](https://user-gold-cdn.xitu.io/2019/4/1/169d7c3ea2437de3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**v2版本**签名块（APK Signing Block）本身又主要分成三部分:

- **SignerData**（签名者数据）：主要包括签名者的证书，整个APK完整性校验hash，以及一些必要信息
- **Signature**（签名）：开发者对SignerData部分数据的签名数据
- **PublicKey**（公钥）：用于验签的公钥数据

**v3版本**签名块也分成同样的三部分，与v2不同的是在SignerData部分，v3新增了attr块，其中是由更小的level块组成。每个level块中可以存储一个证书信息。前一个level块证书验证下一个level证书，以此类推。最后一个level块的证书，要符合SignerData中本身的证书，即用来签名整个APK的公钥所属于的证书

![img](https://user-gold-cdn.xitu.io/2019/4/1/169d7c5908d3c159?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



推荐文章：

- [APK 签名方案 v3](https://source.android.google.cn/security/apksigning/v3)
- [Android P v3签名新特性](https://xuanxuanblingbling.github.io/ctf/android/2018/12/30/signature/)

#### (6) 为什么会有R文件这个映射表？直接使用资源的路径不好么？

打开R文件查看R文件源码，通过R.java类中的注释可以看出，R.java是由啊aapt工具根据应用中的资源文件夹自动生成的，因此可以把R.java理解成android应用的资源字典。

aapt生成的R.java文件的规则主要是如下两条:

1.每类资源对应的R类的一个内部类。比如所有界面布局资源对应于layout内部类；所有字符串对应与string内部类；所有表示符资源对应于id内部类。

2.每个具体的资源项对应内部类的一个public static final int类型的Field。例如前面在界面布局文件中用到了ok、show两个标识符，因此R.id类里就包含了这两个Field;由于drawable-xxxx文件里包含了icon.png图片，因此R.drawable类里包含了icon-Field。

随着我们不断的想Android项目中添加资源，R.java文件的内容也会越来越多。

主要是为了资源检索快速方法，使用资源变的简单。

### 6. Context

#### (1) 关于Context的理解？四大组件里面的Context都来源于哪里?

> [《Android Context 上下文 你必须知道的一切》](https://blog.csdn.net/lmj623565791/article/details/40481055)

#### (2) Android应用中哪些是 Context，一个应用有多少个 Context？

> [Android Context 熟悉还是陌生](https://www.jianshu.com/p/cc0bb2a71ee8)

Context的数量等于Activity的个数 + Service的个数 +1，这个1为Application。

#### (3) Context 的使用上如何避免内存泄漏？

- 不要让生命周期长于Activity的对象持有到Activity的引用
- 尽量使用Application的Context而不是Activity的Context
- 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用。如果使用静态内部类，将外部实例引用作为弱引用持有。

#### (4) 如何跨进程拿 Context？Activity 还没启动的时候如何拿 Context？

**getApplication 与 getApplicationContext 区别**：`getApplication()` 用用来获取 Application实例的，但这个方法只有在 Activity 和 Service 中才能调用。如果在一些其它的场景，比如BroadcastReceiver 中也想获得 Application 的实例，这时需要借助 `getApplicationContext()` 方法。也就是说，`getApplicationContext()` 方法的作用域会更广一些，任何一个 Context 的实例，只要调用 `getApplicationContext()` 方法都可以拿到我们的Application对象；

**Application 中方法的执行顺序为**：Application 构造方法 → `attachBaseContext()` → `onCreate()`。如果在 `attachBaseContext()` 中或执行前，调用 `getApplicationContext()` 得到的值为null

#### (5) ApplicationContext和ActivityContext的区别

这是两种不同的context，也是最常见的两种。第一种中context的生命周期与Application的生命周期相关的，context随着Application的销毁而销毁，伴随application的一生，与activity的生命周期无关。第二种中的context跟Activity的生命周期是相关的，但是对一个Application来说，Activity可以销毁几次，那么属于Activity的context就会销毁多次。至于用哪种context，得看应用场景。还有就是，在使用context的时候，小心内存泄露，防止内存泄露，注意一下几个方面：

- 不要让生命周期长的对象引用activity context，即保证引用activity的对象要与activity本身生命周期是一样的。
- 对于生命周期长的对象，可以使用application context。
- 避免非静态的内部类，尽量使用静态类，避免生命周期问题，注意内部类对外部对象引用导致的生命周期变化。

### 7. AIDL

#### (1) AIDL in out inout oneway代表什么意思？

in、out、inout表示跨进程通信中数据的流向（基本数据类型默认是in，非基本数据类型可以使用其它数据流向out、inout）。

- in参数使得实参顺利传到服务方，但服务方对实参的任何改变，不会反应回调用方。
- out参数使得实参不会真正传到服务方，只是传一个实参的初始值过去（这里实参只是作为返回值来使用的，这样除了return那里的返回值，还可以返回另外的东西），但服务方对实参的任何改变，在调用结束后会反应回调用方。
- inout参数则是上面二者的结合，实参会顺利传到服务方，且服务方对实参的任何改变，在调用结束后会反应回调用方。

> 其实inout，都是相对于服务方。in参数使得实参传到了服务方，所以是in进入了服务方；out参数使得实参在调用结束后从服务方传回给调用方，所以是out从服务方出来。

- oneway可以用来修饰在interface之前，这样会造成interface内所有的方法都隐式地带上oneway；oneway也可以修饰在interface里的各个方法之前。被oneway修饰了的方法不可以有返回值，也不可以有带out或inout的参数。

#### (2) 讲讲AIDL？如何优化多模块都使用AIDL的情况？

AIDL(Android Interface Definition Language，Android接口定义语言)：如果在一个进程中要调用另一个进程中对象的方法，可使用AIDL生成可序列化的参数，AIDL会生成一个服务端对象的代理类，通过它客户端可以实现间接调用服务端对象的方法。

AIDL的本质是系统提供了一套可快速实现Binder的工具。关键类和方法：

- AIDL接口：继承IInterface。

- Stub类：Binder的实现类，服务端通过这个类来提供服务。

- Proxy类：服务端的本地代理，客户端通过这个类调用服务端的方法。

- asInterface()：客户端调用，将服务端返回的Binder对象，转换成客户端所需要的AIDL接口类型的对象。如果客户端和服务端位于同一进程，则直接返回Stub对象本身，否则返回系统封装后的Stub.proxy对象。

- asBinder()：根据当前调用情况返回代理Proxy的Binder对象。

- onTransact()：运行在服务端的Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。

- transact()：运行在客户端，当客户端发起远程请求的同时将当前线程挂起。之后调用服务端的onTransact()直到远程请求返回，当前线程才继续执行。

当有多个业务模块都需要AIDL来进行IPC，此时需要为每个模块创建特定的aidl文件，那么相应的Service就会很多。必然会出现系统资源耗费严重、应用过度重量级的问题。解决办法是建立Binder连接池，即将每个业务模块的Binder请求统一转发到一个远程Service中去执行，从而避免重复创建Service。

工作原理：每个业务模块创建自己的AIDL接口并实现此接口，然后向服务端提供自己的唯一标识和其对应的Binder对象。服务端只需要一个Service并提供一个queryBinder接口，它会根据业务模块的特征来返回相应的Binder对象，不同的业务模块拿到所需的Binder对象后就可以进行远程方法的调用了。

#### (3) 手写实现简化版AMS（AIDL实现）

与Binder相关的几个类的职责:

- IBinder：跨进程通信的Base接口，它声明了跨进程通信需要实现的一系列抽象方法，实现了这个接口就说明可以进行跨进程通信，Client和Server都要实现此接口。
- IInterface：这也是一个Base接口，用来表示Server提供了哪些能力，是Client和Server通信的协议。
- Binder：提供Binder服务的本地对象的基类，它实现了IBinder接口，所有本地对象都要继承这个类。
- BinderProxy：在Binder.java这个文件中还定义了一个BinderProxy类，这个类表示Binder代理对象它同样实现了IBinder接口，不过它的很多实现都交由native层处理。Client中拿到的实际上是这个代理对象。
- Stub：这个类在编译aidl文件后自动生成，它继承自Binder，表示它是一个Binder本地对象；它是一个抽象类，实现了IInterface接口，表明它的子类需要实现Server将要提供的具体能力（即aidl文件中声明的方法）。
- Proxy：它实现了IInterface接口，说明它是Binder通信过程的一部分；它实现了aidl中声明的方法，但最终还是交由其中的mRemote成员来处理，说明它是一个代理对象，mRemote成员实际上就是BinderProxy。

aidl文件只是用来定义C/S交互的接口，Android在编译时会自动生成相应的Java类，生成的类中包含了Stub和Proxy静态内部类，用来封装数据转换的过程，实际使用时只关心具体的Java接口类即可。为什么Stub和Proxy是静态内部类呢？这其实只是为了将三个类放在一个文件中，提高代码的聚合性。通过上面的分析，我们其实完全可以不通过aidl，手动编码来实现Binder的通信，下面我们通过编码来实现ActivityManagerService：

1、首先定义IActivityManager接口：

```java
public interface IActivityManager extends IInterface {
    //binder描述符
    String DESCRIPTOR = "android.app.IActivityManager";
    //方法编号
    int TRANSACTION_startActivity = IBinder.FIRST_CALL_TRANSACTION + 0;
    //声明一个启动activity的方法，为了简化，这里只传入intent参数
    int startActivity(Intent intent) throws RemoteException;
}
```

2、然后，实现ActivityManagerService侧的本地Binder对象基类：

```java
// 名称随意，不一定叫Stub
public abstract class ActivityManagerNative extends Binder implements IActivityManager {

    public static IActivityManager asInterface(IBinder obj) {
        if (obj == null) {
            return null;
        }
        IActivityManager in = (IActivityManager) obj.queryLocalInterface(IActivityManager.DESCRIPTOR);
        if (in != null) {
            return in;
        }
        //代理对象，见下面的代码
        return new ActivityManagerProxy(obj);
    }

    @Override
    public IBinder asBinder() {
        return this;
    }

    @Override
    protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
        switch (code) {
            // 获取binder描述符
            case INTERFACE_TRANSACTION:
                reply.writeString(IActivityManager.DESCRIPTOR);
                return true;
            // 启动activity，从data中反序列化出intent参数后，直接调用子类startActivity方法启动activity。
            case IActivityManager.TRANSACTION_startActivity:
                data.enforceInterface(IActivityManager.DESCRIPTOR);
                Intent intent = Intent.CREATOR.createFromParcel(data);
                int result = this.startActivity(intent);
                reply.writeNoException();
                reply.writeInt(result);
                return true;
        }
        return super.onTransact(code, data, reply, flags);
    }
}
```

3、接着，实现Client侧的代理对象：

```java
public class ActivityManagerProxy implements IActivityManager {
    private IBinder mRemote;

    public ActivityManagerProxy(IBinder remote) {
        mRemote = remote;
    }

    @Override
    public IBinder asBinder() {
        return mRemote;
    }

    @Override
    public int startActivity(Intent intent) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        int result;
        try {
            // 将intent参数序列化，写入data中
            intent.writeToParcel(data, 0);
            // 调用BinderProxy对象的transact方法，交由Binder驱动处理。
            mRemote.transact(IActivityManager.TRANSACTION_startActivity, data, reply, 0);
            reply.readException();
            // 等待server执行结束后，读取执行结果
            result = reply.readInt();
        } finally {
            data.recycle();
            reply.recycle();
        }
        return result;
    }
}
```

4、最后，实现Binder本地对象（IActivityManager接口）：

```java
public class ActivityManagerService extends ActivityManagerNative {
    @Override
    public int startActivity(Intent intent) throws RemoteException {
        // 启动activity
        return 0;
    }
}
```

简化版的ActivityManagerService到这里就已经实现了，剩下就是Client只需要获取到AMS的代理对象IActivityManager就可以通信了。




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

1) 新权限思想 Android 的权限系统一直是首要的安全概念，因为这些权限只在安装的时候被询问一次。一旦安装了，app 可以 在用户毫不知晓的情况下访问权限内的所有东西，而且一般用户安装的时候很少会去仔细看权限列表，更不会去深入了解这些权限可能带来的相关危害。 但是在 Android 6.0 版本之后，系统不会在软件安装的时候就赋予该 app 所有其申请的权限，对于一些危险级别的权限，app 需要在运行时一个一个询问用户授予权限。

2) 对旧版本 App 的兼容 

只有那些 targetSdkVersion 设置为 23 及以上的应用才会出现异常，在使用危险权限的时候系统必须要获得用户的同意才能使用，要不然应用就会崩溃， 出现类似下面的错误。 `java.lang.SecurityException: Permission Denial... `所以 targetSdkVersion 如果没有设置为 23 版本或者以上，系统还是会使用旧规则：在安装的时候赋予该 app 所 申请的所有权限。所以 app 当然可以和以前一样正常使用了，但是还有一点需要注意的是 6.0 的系统里面，用户可以手动将该 app 的权限关闭，在 App info 里面 Permissions 下边，可以关闭某个权限。如果以前的老应用申请的权限 被用户手动关闭了，不会抛出异常，不会崩溃，只不过调用那些被用户禁止权限的 api 接口返回值都为 null 或者 0， 所以我们只需要做一下判空操作就可以了，这是需要注意的。

3) 普通权限和危险权限列表

只有那些危险级别的权限才需要。如果有需要申请危险级别中的一个权限，就需要进行特殊操作。还有一个比较 人 性 的 地 方 就 是 如 果 同 一 组 的 任 何 一 个 权 限 被 授 权 了 ， 其 他 权 限 也 自 动 被 授 权 。 例 如 ， 一 旦 `WRITE_EXTERNAL_STORAGE` 被授权了，app 也有 `READ_EXTERNAL_STORAGE` 权限了。

4) 支持6.0新版本权限机制 在 Android M 的 api 中，可以通过 `checkSelfPermission` 检测软件是否有某一项权限，以及使用 `requestPermissions` 去请求一组权限。

向用户发起请求之后，请求完成，会有相对应的回调方法，通知软件用户是否授予了权限。 通过在 Activity 或者 Fragment 中重写 `onRequestPermissionsResult` 方法。



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

#### (1) RXJava线程种类，怎么切换线程

`Schedulers.computation()`: 适合运行在密集计算的操作，大多数异步操作符使用该调度器

`Schedulers.io()`:适合运行I/0和阻塞操作

`Schedulers.single()`:适合需要单一线程的操作

`Schedulers.trampoline()`: 适合需要顺序运行的操作

`AndroidSchedulers.mainThread()`：Android的主线程

通过 功能性操作符`subscribeOn()` 和 `observeOn()`实现。

若`Observable.subscribeOn()`多次指定被观察者生产事件的线程，则只有第一次指定有效，其余的指定线程无效

若`Observable.observeOn()`多次指定观察者接收和响应事件的线程，则每次指定均有效，即每指定一次，就会进行一次线程的切换。

#### (2) Rxjava自定义操作符



#### (3) RxJava 变换操作符 map flatMap concatMap buffer？

map：【数据类型转换】将被观察者发送的事件转换为另一种类型的事件。

flatMap：【化解循环嵌套和接口嵌套】将被观察者发送的事件序列进行拆分和转换后合并成一个新的事件序列，最后再进行发送。

concatMap：【有序】与 flatMap 的 区别在于，拆分 & 重新合并生成的事件序列 的顺序与被观察者旧序列生产的顺序一致。

buffer：定期从被观察者发送的事件中获取一定数量的事件并放到缓存区中，然后把这些数据集合打包发射。

[RxJava中map和flatmap操作符的区别及底层实现](https://www.jianshu.com/p/af13a8278a05)

#### (4) 手写rxjava遍历数组

```java
    // 1. 设置需要传入的数组
    Integer[] items = { 0, 1, 2, 3, 4 };

    // 2. 创建被观察者对象（Observable）时传入数组
    // 在创建后就会将该数组转换成Observable & 发送该对象中的所有数据
    Observable.fromArray(items)
            .subscribe(new Observer<Integer>() {
                @Override
                public void onSubscribe(Disposable d) {
                    Log.d(TAG, "数组遍历");
                }

                @Override
                public void onNext(Integer value) {
                    Log.d(TAG, "数组中的元素 = "+ value  );
                }

                @Override
                public void onError(Throwable e) {
                    Log.d(TAG, "对Error事件作出响应");
                }

                @Override
                public void onComplete() {
                    Log.d(TAG, "遍历结束");
                }

            });
```

#### (5) zip和merge操作符区别?

merge：可作用所有数据源类型，用于合并多个数据源到一个数据源。merge在合并数据源时，如果一个合并发生异常后会立即调用观察者的onError方法，并停止合并。可通过mergeDelayError操作符，将发生的异常留到最后处理。

可作用于Flowable、Observable、Maybe、Single。将多个数据源的数据一个一个的合并在一起。当其中一个数据源发射完事件之后，若其他数据源还有数据未发射完毕，也会停止。

区别是：

- 方法的参数不一样，zip有一个合并函数，merge没有，所以zip发射数据是合并函数的返回值，merge则是交错排列多个源Observable发射的数据。
- merge的终止不会受任何一个Observable的发射完成而终止，zip则只要有一个Observable的发射完成而终止发射（merge和zip中只要有一个错误通知终止，就都终止） 

#### (6) flatmap为何要生成多个Observable？



#### (7) 常见基类及区别？

io.reactivex.Flowable：发送0个N个的数据，支持Reactive-Streams和背压

io.reactivex.Observable：发送0个N个的数据，不支持背压，

io.reactivex.Single：只能发送单个数据或者一个错误

io.reactivex.Completable：没有发送任何数据，但只处理 onComplete 和 onError 事件。

io.reactivex.Maybe：能够发射0或者1个数据，要么成功，要么失败。



### 2. OkHttp

#### (1) OkHttp责任链模式

可以说是okhttp的精髓所在了，主要体现就是拦截器的使用，具体代码可以看下面的拦截器介绍。

#### (2) interceptors和networkInterceptors的区别？

`interceptors.addAll(client.interceptors());`，这是由开发者设置的，会按照开发者的要求，在所有的拦截器处理之前进行最早的拦截处理，比如一些公共参数，Header都可以在这里添加。

`interceptors.addAll(client.networkInterceptors());`，这里也是开发者自己设置的，所以本质上和第一个拦截器差不多，但是由于位置不同，所以用处也不同。这个位置添加的拦截器可以看到请求和响应的数据了，所以可以做一些网络调试。

#### (3) OkHttp怎么实现连接池?里面怎么处理SSL？

- 为什么需要连接池？

频繁的进行建立`Sokcet`连接和断开`Socket`是非常消耗网络资源和浪费时间的，所以HTTP中的`keepalive`连接对于降低延迟和提升速度有非常重要的作用。`keepalive`机制是什么呢？也就是可以在一次TCP连接中可以持续发送多份数据而不会断开连接。所以连接的多次使用，也就是复用就变得格外重要了，而复用连接就需要对连接进行管理，于是就有了连接池的概念。

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

也就是当如果空闲连接`maxIdleConnections`超过5个或者`keepalive`时间大于5分钟，则将该连接清理掉。

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

| 拦截器                      | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| 应用拦截器                  | 处理原始请求和最终的响应：可以添加自定义header、通用参数、参数加密、网关接入等等。 |
| RetryAndFollowUpInterceptor | 处理错误重试和重定向                                         |
| BridgeInterceptor           | 应用层和网络层的桥接拦截器，主要工作是为请求添加cookie、添加固定的header，比如Host、Content-Length、Content-Type、User-Agent等等，然后保存响应结果的cookie，如果响应使用gzip压缩过，则还需要进行解压。这里会为用户构建一个能够进行网络访问的请求，同时后续工作将网络请求回来的响应Response转化为用户可用的Response。 |
| CacheInterceptor            | 缓存拦截器，获取缓存、更新缓存。如果命中缓存则不会发起网络请求。 |
| ConnectInterceptor          | 连接拦截器，内部会维护一个连接池，负责连接复用、创建连接（三次握手等等）、释放连接以及创建连接上的socket流。 |
| 网络拦截器                  | 用户自定义拦截器，通常用于监控网络层的数据传输。这个位置添加的拦截器可以看到请求和响应的数据了，所以可以做一些网络调试。 |
| CallServerInterceptor       | 请求拦截器，在前置准备工作完成后，真正发起网络请求，进行IO读写。 |

#### (6) OkHttp里面用到了什么设计模式

- 责任链模式

这个可以说是okhttp的精髓所在了，主要体现就是拦截器的使用，具体代码可以看看上述的拦截器介绍。

- 建造者模式

在Okhttp中，建造者模式也是用的挺多的，主要用处是将对象的创建与表示相分离，用Builder组装各项配置。比如Request：

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

调用 response.toString() 连接会断开，后面的取值会出问题。

#### (8) 为什么要在项目中使用这个库？

- OkHttp 提供了对最新的 HTTP 协议版本 HTTP/2 和 SPDY 的支持，这使得对同一个主机发出的所有请求都可以共享相同的套接字连接。
- 如果 HTTP/2 和 SPDY 不可用，OkHttp 会使用连接池来复用连接以提高效率。
- OkHttp 提供了对 GZIP 的默认支持来降低传输内容的大小。
- OkHttp 也提供了对 HTTP 响应的缓存机制，可以避免不必要的网络请求。
- 当网络出现问题时，OkHttp 会自动重试一个主机的多个 IP 地址。

#### (9) 这个库的优缺点是什么，跟同类型库的比较？

- 优点：在上面(8) 
- 缺点：使用的时候仍然需要自己再做一层封装。

#### (10) 这个库的核心实现原理是什么？如果让你实现这个库的某些核心功能，你会考虑怎么去实现？

OkHttp内部的请求流程：使用OkHttp会在请求的时候初始化一个Call的实例，然后执行它的`execute()`方法或`enqueue()`方法，内部最后都会执行到`getResponseWithInterceptorChain()`方法，这个方法里面通过拦截器组成的责任链，依次经过用户自定义普通拦截器、重试拦截器、桥接拦截器、缓存拦截器、连接拦截器和用户自定义网络拦截器以及访问服务器拦截器等拦截处理过程，来获取到一个响应并交给用户。其中，除了OKHttp的内部请求流程这点之外，还用到了缓存和连接池。

#### (11) OkHttp如何处理网络缓存的？缓存逻辑(CacheInterceptor)实现?

整体思路：使用缓存策略CacheStrategy来决定是否使用缓存及如何使用。

总的来说需要知道：strategy.networkRequest为null，不使用网络；strategy.cacheResponse为null，不使用缓存。

获取缓存策略：

- 没有缓存、https但没有握手、不可缓存、忽略缓存或手动配置缓存过期，都是直接进行网络请求。

- 以上都不满足时，如果缓存没过期，那么就是用缓存（可能要添加警告）。

- 如果缓存过期了，但响应头有Etag，Last-Modified，Date，就添加这些header 进行**协商网络请求**。

- 如果缓存过期了，且响应头**没有**设置Etag，Last-Modified，Date，就进行网络请求。

#### (12) http怎么知道文件过大是否传输完毕的响应？

- http协议有正文大小说明的：content-length

- 或者分块传输chunked的话 读到0\r\n\r\n 就是读完了

http响应内容比较大的话，会分成多个tcp  segment 发送，不是最后一个segment的话， tcp的payload不会有http header字段，

如果是最后一个tcp segment 的话，就会有http header 字段，同时， 数据的最后会有 "0\r\n\r\n" 这个东西，这个东西就表示数据都发送完了。

### 3. Retrofit

#### (1) 设计模式和封层解耦的理念



#### (2) 动态代理



#### (3) 编译时注解与运行时注解，为什么retrofit要使用运行时注解？什么时候用运行时注解？



#### (4) retrofit怎么做post请求



#### (5) 这个库是做什么用的？

Retrofit 是一个 RESTful 的 HTTP 网络请求框架的封装。Retrofit 2.0 开始内置 OkHttp，前者专注于接口的封装，后者专注于网络请求的高效。

#### (6) 为什么要在项目中使用这个库？

功能强大：

- 支持同步、异步
- 支持多种数据的解析 & 序列化格式
- 支持RxJava

简洁易用：

- 通过注解配置网络请求参数
- 采用大量设计模式简化使用

可扩展性好：

- 功能模块高度封装
- 解耦彻底，如自定义Converters

#### (7) 这个库都有哪些用法？对应什么样的使用场景？

后台API遵循Restful API设计风格且项目中使用到RxJava。

#### (8) 这个库的优缺点是什么，跟同类型库的比较？

优点：在上面(6) 

缺点：扩展性差，高度封装所带来的必然后果，如果服务器不能给出统一的API形式，会很难处理。

#### (9) 这个库的核心实现原理是什么？如果让你实现这个库的某些核心功能，你会考虑怎么去实现？

Retrofit主要是在create方法中采用动态代理模式（通过访问代理对象的方式来间接访问目标对象）实现接口方法，这个过程构建了一个ServiceMethod对象，根据方法注解获取请求方式，参数类型和参数注解拼接请求的链接，当一切都准备好之后会把数据添加到Retrofit的RequestBuilder中。然后当我们主动发起网络请求的时候会调用okhttp发起网络请求，okhttp的配置包括请求方式，URL等在Retrofit的RequestBuilder的`build()`方法中实现，并发起真正的网络请求。

#### (10) 你从这个库中学到什么有价值的或者说可借鉴的设计思想？

内部使用了优秀的架构设计和大量的设计模式，在我分析过Retrofit最新版的源码和大量优秀的Retrofit源码分析文章后，我发现，要想真正理解Retrofit内部的核心源码流程和设计思想，首先，需要对它使用到的九大设计模式有一定的了解，下面我简单说一说：

**创建Retrofit实例**：

- 使用建造者模式通过内部Builder类建立了一个Retroift实例。
- 网络请求工厂使用了工厂方法模式。

**创建网络请求接口的实例**：

- 首先，使用外观模式统一调用创建网络请求接口实例和网络请求参数配置的方法。
- 然后，使用动态代理动态地去创建网络请求接口实例。
- 接着，使用了建造者模式 & 单例模式创建了serviceMethod对象。
- 再者，使用了策略模式对serviceMethod对象进行网络请求参数配置，即通过解析网络请求接口方法的参数、返回值和注解类型，从Retrofit对象中获取对应的网络的url地址、网络请求执行器、网络请求适配器和数据转换器。
- 最后，使用了装饰者模式ExecuteCallBack为serviceMethod对象加入线程切换的操作，便于接受数据后通过Handler从子线程切换到主线程从而对返回数据结果进行处理。

**发送网络请求**：

- 在异步请求时，通过静态delegate代理对网络请求接口的方法中的每个参数使用对应的ParameterHanlder进行解析。

**解析数据**

**切换线程**：

- 使用了适配器模式通过检测不同的Platform使用不同的回调执行器，然后使用回调执行器切换线程，这里同样是使用了装饰模式。

**处理结果**



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

- 异步加载：线程池。由于网络会阻塞，所以读内存和硬盘可以放在一个线程池，网络需要另外一个线程池，网络也可以采用Okhttp内置的线程池。

- 切换线程：Handler。无论是RxJava、EventBus，还是Glide，只要是想从子线程切换到Android主线程，都离不开Handler。

- 缓存：LruCache、DiskLruCache

  内存缓存：LruCache(最新数据始终在LinkedHashMap最后一个)

  LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。

  当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队尾。当调用put()方法时，就会在结合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队首元素，即近期最少访问的元素。

  磁盘缓存 DiskLruCache(DiskLruCache会自动生成journal文件，这个文件是日志文件，主要记录的是缓存的操作)
  DiskLruCache和LruCache内部都是使用了LinkedHashMap去实现缓存算法的，只不过前者针对的是将缓存存在本地，而后者是直接将缓存存在内存。

- 防止OOM：软引用、LruCache、图片压缩、Bitmap像素存储位置

  软引用：LruCache里存的是软引用对象，那么当内存不足的时候，Bitmap会被回收，也就是说通过SoftReference修饰的Bitmap就不会导致OOM。

  当然，Bitmap被回收的时候，LruCache剩余的大小应该重新计算，可以写个方法，当Bitmap取出来是空的时候，LruCache清理一下，重新计算剩余内存；

  还有另一个问题，就是内存不足时软引用中的Bitmap被回收的时候，这个LruCache就形同虚设，相当于内存缓存失效了，必然出现效率问题。

  onLowMemory：当内存不足的时候，Activity、Fragment会调用onLowMemory方法，可以在这个方法里去清除缓存，Glide使用的就是这一种方式来防止OOM。

  Bitmap 像素存储位置考虑：Android 3.0到8.0 之间Bitmap像素数据存在Java堆，而8.0之后像素数据存到native堆中

- 内存泄露：注意ImageView的正确引用，生命周期管理

  通过给Activity添加自定义Fragment方式，监听生命周期，在Activity/fragment 销毁的时候，取消图片加载任务

- 列表滑动加载的问题：加载错乱、队满任务过多问题

#### (3) Glide缓存实现机制？缓存前压缩，缓存命中？

常规三级缓存的流程：强引用->软引用->硬盘缓存

当我们的APP中想要加载某张图片时，先去LruCache中寻找图片，如果LruCache中有，则直接取出来使用，如果LruCache中没有，则去SoftReference中寻找（软引用适合当cache，当内存吃紧的时候才会被回收。而weakReference在每次system.gc（）就会被回收）（当LruCache存储紧张时，会把最近最少使用的数据放到SoftReference中），如果SoftReference中有，则从SoftReference中取出图片使用，同时将图片重新放回到LruCache中，如果SoftReference中也没有图片，则去硬盘缓存中中寻找，如果有则取出来使用，同时将图片添加到LruCache中，如果没有，则连接网络从网上下载图片。图片下载完成后，将图片保存到硬盘缓存中，然后放到LruCache中。

Glide缓存机制大致分为三层：内存缓存、弱引用缓存、磁盘缓存

取的顺序是：内存、弱引用、磁盘。

存的顺序是：弱引用、内存、磁盘。

内存缓存一般都是用`LruCache`：Glide 默认内存缓存用的也是LruCache，只不过并没有用Android SDK中的LruCache，不过内部同样是基于LinkHashMap，所以原理是一样的。

磁盘缓存 DiskLruCache：DiskLruCache 跟 LruCache 实现思路是差不多的，一样是设置一个总大小，每次往硬盘写文件，总大小超过阈值，就会将旧的文件删除。DiskLruCache 同样是利用LinkHashMap的特点，只不过数组里面存的 Entry 有点变化，Editor 用于操作文件。

三层存储的机制在Engine中实现的。先说下Engine是什么？Engine这一层负责加载时做管理内存缓存的逻辑。持有MemoryCache、Map<Key, WeakReference<EngineResource<?>>>。通过load()来加载图片，加载前后会做内存存储的逻辑。如果内存缓存中没有，那么才会使用EngineJob这一层来进行异步获取硬盘资源或网络资源。EngineJob类似一个异步线程或observable。Engine是一个全局唯一的，通过Glide.getEngine()来获取。

需要一个图片资源，如果Lrucache中有相应的资源图片，那么就返回，同时从Lrucache中清除，放到activeResources中。activeResources map是盛放正在使用的资源，以弱引用的形式存在。同时资源内部有被引用的记录。如果资源没有引用记录了，那么再放回Lrucache中，同时从activeResources中清除。如果Lrucache中没有，就从activeResources中找，找到后相应资源引用加1。如果Lrucache和activeResources中没有，那么进行资源异步请求（网络/diskLrucache），请求成功后，资源放到diskLrucache和activeResources中。

使用一个弱引用map activeResources来盛放项目中正在使用的资源。Lrucache中不含有正在使用的资源。资源内部有个计数器来显示自己是不是还有被引用的情况，把正在使用的资源和没有被使用的资源分开有什么好处呢？因为当Lrucache需要移除一个缓存时，会调用resource.recycle()方法。注意到该方法上面注释写着只有没有任何consumer引用该资源的时候才可以调用这个方法。那么为什么调用resource.recycle()方法需要保证该资源没有任何consumer引用呢？glide中resource定义的recycle()要做的事情是把这个不用的资源（假设是bitmap或drawable）放到bitmapPool中。bitmapPool是一个bitmap回收再利用的库，在做transform的时候会从这个bitmapPool中拿一个bitmap进行再利用。这样就避免了重新创建bitmap，减少了内存的开支。而既然bitmapPool中的bitmap会被重复利用，那么肯定要保证回收该资源的时候（即调用资源的recycle()时），要保证该资源真的没有外界引用了。这也是为什么glide花费那么多逻辑来保证Lrucache中的资源没有外界引用的原因。



#### (4) Glide如何处理生命周期？

- 创建一个无UI的Fragment，具体来说就是SupportRequestManagerFragment/RequestManagerFragment,并绑定到当前Activity，这样Fragment就可以感知Activity的生命周期；
- 在创建Fragment时，初始化Lifecycle，LifecycleListener，并且在生命周期的`onStart()`,`onStop()`,`onDestory()`中调用相关方法;
- 在创建RequestManger的时候传入Lifecycle对象，并且LifecycleListener实现了LifecycleListener接口；
- 这样当生命周期变化的时候，就能通过接口回调去通知RequestManager处理请求.

#### (5) 有用过Glide的什么深入的API，自定义model是在Glide的什么阶段？



#### (6) 这个库都有哪些用法？对应什么样的使用场景？

图片加载：Glide.with(this).load(imageUrl).override(800, 800).placeholder().error().animate().into()。

多样式媒体加载：asBitamp、asGif。

生命周期集成。

可以配置磁盘缓存策略ALL、NONE、SOURCE、RESULT。

#### (7) 这个库的优缺点是什么，跟同类型库的比较？

库比较大，源码实现复杂。

#### (8) 这个库的核心实现原理是什么？如果让你实现这个库的某些核心功能，你会考虑怎么去实现

**Glide&with**：

1、初始化各式各样的配置信息（包括缓存，请求线程池，大小，图片格式等等）以及glide对象。

2、将glide请求和application/SupportFragment/Fragment的生命周期绑定在一块。

**Glide&load**：

设置请求url，并记录url已设置的状态。

**Glide&into**：

1、首先根据转码类transcodeClass类型返回不同的ImageViewTarget：BitmapImageViewTarget、DrawableImageViewTarget。

2、递归建立缩略图请求，没有缩略图请求，则直接进行正常请求。

3、如果没指定宽高，会根据ImageView的宽高计算出图片宽高，最终执行到`onSizeReay()`方法中的`engine.load()`方法。

4、engine是一个负责加载和管理缓存资源的类



#### (9) Glide如何确定图片加载完毕？



#### (10) Glide内存缓存如何控制大小？



### 5. LeakCanary

#### (1) LeakCanary 的收集内存泄露是在 Activity 的什么时机，大致原理

结起来主要分三个部分：检测泄露；分析泄露；通知泄露。

分析泄露用到了HAHA这个工具，通知泄露，利用安卓自身消息机制，Service, Notification, Activity这些。

检测内存泄露检测这块是LeakCanary原理的核心。

LeakCanary检测内存泄露分两个部分：

**1.监听Activity生命周期，在Activity销毁的时候通知LeakCanary。**

```java
public class LearnLeakCanaryApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        ActivityRefWatcher.install(this, new RefWatcher());
    }
}
```

注册主要做两件事：一是Application绑定Activity生命周期,Activity销毁的时候都能监听到。二是new了个RefWatcher对象，RefWatcher就做了一件事，检测泄露，如果泄露就捕捉给HAHA处理。这样看，这一句代码就完成了两件重要的工作。

ActivityRefWatcher注册时最后的调用watchActivities，在这个函数里完成Application和生命周期的监听注册。

```java
public static void install(Application application, RefWatcher refWatcher) {
        new ActivityRefWatcher(application, refWatcher).watchActivities();
}
```

```java
public void watchActivities() {
        // Make sure you don't get installed twice.
        stopWatchingActivities();
        application.registerActivityLifecycleCallbacks(lifecycleCallbacks);
}
```

lifecycleCallbacks就是生命周期回调，Activity销毁时，会被Application监听，然后走onActivityDestroyed回调。在onActivityDestroyed里，把自己交给RefWatcher，让RefWatcher去检测自己是否真的被回收。

```java
private final Application.ActivityLifecycleCallbacks lifecycleCallbacks =
            new Application.ActivityLifecycleCallbacks() {
                @Override public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                }

                @Override public void onActivityStarted(Activity activity) {
                }

                @Override public void onActivityResumed(Activity activity) {
                }

                @Override public void onActivityPaused(Activity activity) {
                }

                @Override public void onActivityStopped(Activity activity) {
                }

                @Override public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
                }

                @Override public void onActivityDestroyed(Activity activity) {
                    Log.d(TAG, activity.getLocalClassName() + " onActivityDestroyed");
                    ActivityRefWatcher.this.onActivityDestroyed(activity);
                }
            };
```

**2.内存泄露检测**

内存泄露判断主要是用到了WeakReference和ReferenceQueue，他们俩的关系很简单，弱引用对象回收了，弱引用对象就会在ReferenceQueue里，如果ReferenceQueue里没有，就说明可能泄露。

可能泄露是因为GC不一定及时，所以LeakCanary会再调一次GC，然后再检测ReferenceQueue是否存在回收的对象。如果这次还没有就是泄露了，后面的逻辑就是给HAHA分析，Notification通知。

#### (2) 为什么要在项目中使用这个库？

- 针对Android Activity组件完全自动化的内存泄漏检查，在最新的版本中，还加入了android.app.fragment的组件自动化的内存泄漏检测。
- 易用集成，使用成本低。
- 友好的界面展示和通知。

#### (3) 这个库的优缺点是什么，跟同类型库的比较？

检测结果并不是特别的准确，因为内存的释放和对象的生命周期有关也和GC的调度有关。只能检测Activity和Fragment。

#### (4) 这个库的核心实现原理是什么？如果让你实现这个库的某些核心功能，你会考虑怎么去实现？

主要分为如下7个步骤：

- RefWatcher.watch()创建了一个KeyedWeakReference用于去观察对象。
- 然后，在后台线程中，它会检测引用是否被清除了，并且是否没有触发GC。
- 如果引用仍然没有被清除，那么它将会把堆栈信息保存在文件系统中的.hprof文件里。
- HeapAnalyzerService被开启在一个独立的进程中，并且HeapAnalyzer使用了HAHA开源库解析了指定时刻的堆栈快照文件heap dump。
- 从heap dump中，HeapAnalyzer根据一个独特的引用key找到了KeyedWeakReference，并且定位了泄露的引用。
- HeapAnalyzer为了确定是否有泄露，计算了到GC Roots的最短强引用路径，然后建立了导致泄露的链式引用。
- 这个结果被传回到app进程中的DisplayLeakService，然后一个泄露通知便展现出来了。

简单来说就是：

在一个Activity执行完`onDestroy()`之后，将它放入WeakReference中，然后将这个WeakReference类型的Activity对象与ReferenceQueque关联。这时再从ReferenceQueque中查看是否有该对象，如果没有，执行gc，再次查看，还是没有的话则判断发生内存泄露了。最后用HAHA这个开源库去分析dump之后的heap内存（主要就是创建一个HprofParser解析器去解析出对应的引用内存快照文件snapshot）。

流程图：

![image](https://user-gold-cdn.xitu.io/2020/2/24/17074f730b83c0c3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

源码分析中一些核心分析点：

AndroidExcludedRefs：它是一个enum类，它声明了Android SDK和厂商定制的SDK中存在的内存泄露的case，根据AndroidExcludedRefs这个类的类名就可看出这些case都会被Leakcanary的监测过滤掉。

`buildAndInstall()`（即install方法）这个方法应该仅仅只调用一次。

debuggerControl : 判断是否处于调试模式，调试模式中不会进行内存泄漏检测。为什么呢？因为在调试过程中可能会保留上一个引用从而导致错误信息上报。

watchExecutor : 线程控制器，在 `onDestroy()` 之后并且主线程空闲时执行内存泄漏检测。

gcTrigger : 用于 GC，watchExecutor 首次检测到可能的内存泄漏，会主动进行 GC，GC 之后会再检测一次，仍然泄漏的判定为内存泄漏，最后根据heapDump信息生成相应的泄漏引用链。

gcTrigger的`runGc()`方法：这里并没有使用`System.gc()`方法进行回收，因为`System.gc()`并不会每次都执行。而是从AOSP中拷贝一段GC回收的代码，从而相比`System.gc()`更能够保证进行垃圾回收的工作。

```
Runtime.getRuntime().gc();
```

子线程延时1000ms；

`System.runFinalization();`

install方法内部最终还是调用了application的`registerActivityLifecycleCallbacks()`方法，这样就能够监听activity对应的生命周期事件了。

在`RefWatcher#watch()`中使用随机的UUID保证了每个检测对象对应的key 的唯一性。

在KeyedWeakReference内部，使用了key和name标识了一个被检测的WeakReference对象。在其构造方法中将弱引用和引用队列 ReferenceQueue 关联起来，如果弱引用reference持有的对象被GC回收，JVM就会把这个弱引用加入到与之关联的引用队列referenceQueue中。即 KeyedWeakReference 持有的 Activity 对象如果被GC回收，该对象就会加入到引用队列 referenceQueue 中。

使用Android SDK的API `Debug.dumpHprofData()` 来生成 hprof 文件。

在HeapAnalyzerService（类型为IntentService的ForegroundService）的`runAnalysis()`方法中，为了避免减慢app进程或占用内存，这里将HeapAnalyzerService设置在了一个独立的进程中。 

#### (5) 为什么不用虚引用？引用队列里面存的是什么？内存数据是如何dump出来的？

弱引用对象回收了，弱引用对象就会在ReferenceQueue里，如果ReferenceQueue里没有，就说明可能泄露。

使用Android SDK的API `Debug.dumpHprofData()` 来生成 hprof 文件。

### 6. Android Jepack(非必需)

我主要阅读了Android Jetpack中以下库的源码：

- `Lifecycle`：观察者模式，组件生命周期中发送事件。
- `DataBinding`：核心就是利用LiveData或者Observablexxx实现的观察者模-式，对16进制的状态位更新，之后根据这个状态位去更新对应的内容。
- `LiveData`：观察者模式，事件的生产消费模型。
- `ViewModel`：借用Activty异常销毁时存储隐藏Fragment的机制存储ViewModel，保证数据的生命周期尽可能的延长。
- `Paging`：设计思想。

以后有时间再给大家做源码分析。

*建议阅读：*

> [《Android Jetpack源码分析系列》](https://blog.csdn.net/mq2553299/column/info/24151)

#### 1. LifeCycle的原理是怎样的？如果在onStart里面订阅，会回调onCreate吗？

#### 2. ViewModel为什么在旋转屏幕后不会丢失状态?

#### 3. 有没有看一下Google官方的ViewModel demo?

#### 4. ViewModel在Activity初始化与在Fragment中初始化，有什么区别？

#### 5.viewModel是怎么实现双向数据绑定的？

#### 6.viewModel怎么实现自动处理生命周期？

#### 7. ViewModel的使用中有什么坑？

#### 8. DataBinding的原理了解吗？

#### 9. ViewModel如何知道View层的生命周期？

事实上，如果你仅仅使用ViewModel，它是感知不了生命周期，它需要结合LiveData去感知生命周期，如果仅仅使用DataBinding去实现MVVM，它对数据源使用了弱引用，所以一定程度上可以避免内存泄漏的发生。





## 五、性能优化

[《Android 性能优化最佳实践》](https://juejin.im/post/6844903641032163336)

### 0. 稳定优化

需要更全面更深入的理解请查看[深入探索Android稳定性优化](https://juejin.im/post/6844903972587716621)

#### 1. 你们做了哪些稳定性方面的优化？

随着项目的逐渐成熟，用户基数逐渐增多，DAU持续升高，我们遇到了很多稳定性方面的问题，对于我们技术同学遇到了很多的挑战，用户经常使用我们的App卡顿或者是功能不可用，因此我们就针对稳定性开启了专项的优化，我们主要优化了三项：

- Crash专项优化（=>2)
- 性能稳定性优化（=>2)
- 业务稳定性优化（=>3)

通过这三方面的优化我们搭建了移动端的高可用平台。同时，也做了很多的措施来让App真正地实现了高可用。

#### 2. 性能稳定性是怎么做的？

- 全面的性能优化：启动速度、内存优化、绘制优化
- 线下发现问题、优化为主
- 线上监控为主
- Crash专项优化

我们针对启动速度，内存、布局加载、卡顿、瘦身、流量、电量等多个方面做了多维的优化。

我们的优化主要分为了两个层次，即线上和线下，针对于线下呢，我们侧重于发现问题，直接解决，将问题尽可能在上线之前解决为目的。而真正到了线上呢，我们最主要的目的就是为了监控，对于各个性能纬度的监控呢，可以让我们尽可能早地获取到异常情况的报警。

同时呢，对于线上最严重的性能问题性问题：Crash，我们做了专项的优化，不仅优化了Crash的具体指标，而且也尽可能地获取了Crash发生时的详细信息，结合后端的聚合、报警等功能，便于我们快速地定位问题。

#### 3. 业务稳定性如何保障？

- 数据采集 + 报警
- 需要对项目的主流程与核心路径进行埋点监控，
- 同时还需知道每一步发生了多少异常，这样，我们就知道了所有业务流程的转换率以及相应界面的转换率
- 结合大盘，如果转换率低于某个值，进行报警
- 异常监控 + 单点追查
- 兜底策略

移动端业务高可用它侧重于用户功能完整可用，主要是为了解决一些线上一些异常情况导致用户他虽然没有崩溃，也没有性能问题，但是呢，只是单纯的功能不可用的情况，我们需要对项目的主流程、核心路径进行埋点监控，来计算每一步它真实的转换率是多少，同时呢，还需要知道在每一步到底发生了多少异常。这样我们就知道了所有业务流程的转换率以及相应界面的转换率，有了大盘的数据呢，我们就知道了，如果转换率或者是某些监控的成功率低于某个值，那很有可能就是出现了线上异常，结合了相应的报警功能，我们就不需要等用户来反馈了，这个就是业务稳定性保障的基础。

同时呢，对于一些特殊情况，比如说，开发过程当中或代码中出现了一些catch代码块，捕获住了异常，让程序不崩溃，这其实是不合理的，程序虽然没有崩溃，当时程序的功能已经变得不可用，所以呢，这些被catch的异常我们也需要上报上来，这样我们才能知道用户到底出现了什么问题而导致的异常。此外，线上还有一些单点问题，比如说用户点击登录一直进不去，这种就属于单点问题，其实我们是无法找出其和其它问题的共性之处的，所以呢，我们就必须要找到它对应的详细信息。

最后，如果发生了异常情况，我们还采取了一系列措施进行快速止损。（=>4）

#### 4. 如果发生了异常情况，怎么快速止损？

- 功能开关
- 统跳中心
- 动态修复：热修复、资源包更新
- 自主修复：安全模式

首先，需要让App具备一些高级的能力，我们对于任何要上线的新功能，要加上一个功能的开关，通过配置中心下发的开关呢，来决定是否要显示新功能的入口。如果有异常情况，可以紧急关闭新功能的入口，那就可以让这个App处于可控的状态了。

然后，我们需要给App设立路由跳转，所有的界面跳转都需要通过路由来分发，如果我们匹配到需要跳转到有bug的这样一个新功能时，那我们就不跳转了，或者是跳转到统一的异常正处理中的界面。如果这两种方式都不可以，那就可以考虑通过热修复的方式来动态修复，目前热修复的方案其实已经比较成熟了，我们完全可以低成本地在我们的项目中添加热修复的能力，当然，如果有些功能是由RN或WeeX来实现就更好了，那就可以通过更新资源包的方式来实现动态更新。而这些如果都不可以的话呢，那就可以考虑自己去给应用加上一个自主修复的能力，如果App启动多次的话，那就可以考虑清空所有的缓存数据，将App重置到安装的状态，到了最严重的等级呢，可以阻塞主线程，此时一定要等App热修复成功之后才允许用户进入。

### 1. 布局优化

需要更全面更深入的理解请查看[Android性能优化之绘制优化](https://juejin.im/post/6844904080989487118)、[深入探索Android布局优化（上）](https://juejin.im/post/6844904047355363341)、[深入探索Android布局优化（下）](https://juejin.im/post/6844904048068395015)、[深入探索Android卡顿优化（上）](https://juejin.im/post/6844904062610046990)、[深入探索Android卡顿优化（下）](https://juejin.im/post/6844904066259091469)

#### 1. 你在做布局优化的过程中用到了哪些工具？

我在做布局优化的过程中，用到了很多的工具，但是每一个工具都有它不同的使用场景，不同的场景应该使用不同的工具。下面我从线上和线下两个角度来进行分析。

比如说，我要统计线上的FPS，我使用的就是Choreographer这个类，它具有以下特性：

- 能够获取整体的帧率。
- 能够带到线上使用。
- 它获取的帧率几乎是实时的，能够满足我们的需求。

同时，在线下，如果要去优化布局加载带来的时间消耗，那就需要检测每一个布局的耗时，对此我使用的是AOP的方式，它没有侵入性，同时也不需要别的开发同学进行接入，就可以方便地获取每一个布局加载的耗时。如果还要更细粒度地去检测每一个控件的加载耗时，那么就需要使用`LayoutInflaterCompat.setFactory2`这个方法去进行Hook。

此外，我还使用了`LayoutInspector`和`Systrace`这两个工具，`Systrace`可以很方便地看到每帧的具体耗时以及这一帧在布局当中它真正做了什么。而`LayoutInspector`可以很方便地看到每一个界面的布局层级，帮助我们对层级进行优化。

#### 2. 布局为什么会导致卡顿，你又是如何优化的？

分析完布局的加载流程之后，我们发现有如下四点可能会导致布局卡顿：

- 首先，系统会将我们的Xml文件通过**IO**的方式映射的方式加载到我们的内存当中，而IO的过程可能会导致卡顿。
- 其次，布局加载的过程是一个反射的过程，而反射的过程也会可能会导致卡顿。
- 同时，这个布局的层级如果比较深，那么进行布局遍历的过程就会比较耗时。
- 最后，不合理的嵌套RelativeLayout布局也会导致重绘的次数过多。

对此，我们的优化方式有如下几种：

- 针对布局加载Xml文件的优化，我们使用了异步Inflate的方式，即AsyncLayoutInflater。它的核心原理是在子线程中对我们的Layout进行加载，而加载完成之后会将View通过Handler发送到主线程来使用。所以不会阻塞我们的主线程，加载的时间全部是在异步线程中进行消耗的。而这仅仅是一个从侧面缓解的思路。
- 后面，我们发现了一个从根源解决上述痛点的方式，即使用X2C框架。它的一个核心原理就是在开发过程我们还是使用的XML进行编写布局，但是在编译的时候它会使用APT的方式将XML布局转换为Java的方式进行布局，通过这样的方式去写布局，它有以下优点：1、它省去了使用IO的方式去加载XML布局的耗时过程。2、它是采用Java代码直接new的方式去创建控件对象，所以它也没有反射带来的性能损耗。这样就从根本上解决了布局加载过程中带来的问题。
- 然后，我们可以使用ConstraintLayout去减少我们界面布局的嵌套层级，如果原始布局层级越深，它能减少的层级就越多。而使用它也能避免嵌套RelativeLayout布局导致的重绘次数过多。
- 最后，我们可以使用`AspectJ框架（即AOP）`和`LayoutInflaterCompat.setFactory2`的方式分别去建立线下全局的布局加载速度和控件加载速度的监控体系。

#### 3. 做完布局优化有哪些成果产出？

- 首先，我们建立了一个体系化的监控手段，这里的体系还指的是线上加线下的一个综合方案，针对线下，我们使用AOP或者ARTHook，可以很方便地获取到每一个布局的加载耗时以及每一个控件的加载耗时。针对线上，我们通过Choreographer.getInstance().postFrameCallback的方式收集到了FPS，这样我们可以知道用户在哪些界面出现了丢帧的情况。
- 然后，对于布局监控方面，我们设立了FPS、布局加载时间、布局层级等一系列指标。
- 最后，在每一个版本上线之前，我们都会对我们的核心路径进行一次Review，确保我们的FPS、布局加载时间、布局层级等达到一个合理的状态。

#### 4. 你是怎么做卡顿优化的？

从项目的初期到壮大期，最后再到成熟期，每一个阶段都针对卡顿优化做了不同的处理。各个阶段所做的事情如下所示：

- 系统工具定位、解决
- 自动化卡顿方案及优化
- 线上监控及线下监测工具的建设

我做卡顿优化也是经历了一些阶段，最初我们的项目当中的一些模块出现了卡顿之后，我是通过系统工具进行了定位，我使用了Systrace，然后看了卡顿周期内的CPU状况，同时结合代码，对这个模块进行了重构，将部分代码进行了异步和延迟，在项目初期就是这样解决了问题。但是呢，随着我们项目的扩大，线下卡顿的问题也越来越多，同时，在线上，也有卡顿的反馈，但是线上的反馈卡顿，我们在线下难以复现，于是我们开始寻找自动化的卡顿监测方案，其思路是来自于Android的消息处理机制，主线程执行任何代码都会回到Looper.loop方法当中，而这个方法中有一个mLogging对象，它会在每个message的执行前后都会被调用，我们就是利用这个前后处理的时机来做到的自动化监测方案的。同时，在这个阶段，我们也完善了线上ANR的上报，我们采取的方式就是监控ANR的信息，同时结合了ANR-WatchDog，作为高版本没有文件权限的一个补充方案。在做完这个卡顿检测方案之后呢，我们还做了线上监控及线下检测工具的建设，最终实现了一整套完善，多维度的解决方案。

#### 5. 你是怎么样自动化的获取卡顿信息？

我们的思路是来自于Android的消息处理机制，主线程执行任何代码它都会走到Looper.loop方法当中，而这个函数当中有一个**mLogging**对象，它会在每个message处理前后都会被调用，而主线程发生了卡顿，那就一定会在dispatchMessage方法中执行了耗时的代码，那我们在这个message执行之前呢，我们可以在子线程当中去postDelayed一个任务，这个Delayed的时间就是我们设定的阈值，如果主线程的messaege在这个阈值之内完成了，那就取消掉这个子线程当中的任务，如果主线程的message在阈值之内没有被完成，那子线程当中的任务就会被执行，它会获取到当前主线程执行的一个堆栈，那我们就可以知道哪里发生了卡顿。

经过实践，我们发现这种方案获取的堆栈信息它不一定是准确的，因为获取到的堆栈信息它很可能是主线程最终执行的一个位置，而真正耗时的地方其实已经执行完成了，于是呢，我们就对这个方案做了一些优化，我们采取了**高频采集**的方案，也就是在一个周期内我们会多次采集主线程的堆栈信息，如果发生了卡顿，那我们就将这些卡顿信息压缩之后上报给APM后台，然后找出重复的堆栈信息，这些重复发生的堆栈大概率就是卡顿发生的一个位置，这样就提高了获取卡顿信息的一个准确性。

#### 6. 卡顿的一整套解决方案是怎么做的？

首先，针对卡顿，我们采用了**线上、线下工具相结合**的方式，线下工具我们册中医药尽可能早地去暴露问题，而针对于线上工具呢，我们侧重于监控的全面性、自动化以及异常感知的灵敏度。

同时呢，卡顿问题还有很多的难题。比如说**有的代码呢，它不到你卡顿的一个阈值，但是执行过多，或者它错误地执行了很多次，它也会导致用户感官上的一个卡顿**，所以我们在线下通过AOP的方式对常见的耗时代码进行了Hook，然后对一段时间内获取到的数据进行分析，我们就可以知道这些耗时的代码发生的时机和次数以及耗时情况。然后，看它是不是满足我们的一个预期，不满足预期的话，我们就可以直接到线下进行修改。同时，卡顿监控它还有很多容易被忽略的一个**盲区**，比如说生命周期的一个间隔，那对于这种特定的问题呢，我们就采用了编译时注解的方式修改了项目当中所有Handler的父类，对于其中的两个方法进行了监控，我们就可以知道主线程message的执行时间以及它们的调用堆栈。

对于**线上卡顿**，我们除了计算App的卡顿率、ANR率等常规指标之外呢，我们还计算了页面的秒开率、生命周期的执行时间等等。而且，在卡顿发生的时刻，我们也尽可能多地保存下来了当前的一个场景信息，这为我们之后解决或者复现这个卡顿留下了依据。

#### 7. TextView setText耗时的原因，对TextView绘制层源码的理解？

#### 8. 开放问题：优化一个列表页面的打开速度和流畅性。

#### 9. 怎么优化xml inflate的时间(涉及IO与反射)？

- 减少布局的嵌套层级

- 异步加载

  AsyncLayoutInflater，为`ViewGroup`动态添加子`View`时，我们往往使用一个layout的XML来inflate一个view，然后将其add到父容器。
  inflate包含对XML文件的读取和解析(IO操作)，并通过反射创建`View`树。当XML文件过大或页面层级过深，布局的加载就会较为耗时。
  由于这一步并非UI操作，可以转移到非主线程执行，为此，官方在扩展包提供了`AsyncLayoutInflater`。

  ```java
  AsyncLayoutInflater asyncLayoutInflater = new AsyncLayoutInflater(this);
  AsyncLayoutInflater.OnInflateFinishedListener onInflateFinishedListener = new AsyncLayoutInflater.OnInflateFinishedListener() {
      @Override
      public void onInflateFinished(@NonNull View view, int resid, @Nullable ViewGroup parent) {
          if (parent != null) {
              parent.addView(view);
          }
      }
  };
  
  for (int i = 0; i < 10; i++) {
      asyncLayoutInflater.inflate(R.layout.view_jank, container, onInflateFinishedListener);
  }
  ```

  使用`AsyncLayoutInflater`异步inflate后，主线程就不再有inflate的耗时了。

  适用场景：动态加载layout较复杂的view

- 懒加载ViewStub

  ```xml
  <ViewStub
      android:id="@+id/stub"
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:layout="@layout/real_view" />
  ```

  ```java
  ViewStub stub = findViewById(R.id.stub);
  if (stub != null) {
      stub.inflate(); // inflate一次以后，view树中就不再包含这个ViewStub了
  }
  ```

  适用场景：只在部分情况下才显示的View

  例如：网络请求失败的提示；列表为空的提示；新内容、新功能的引导，因为引导基本上只显示一次

- 延迟加载IdleHandler

  ```java
  Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
      @Override
      public boolean queueIdle() {
          // 当该Looper线程没有message要处理时才执行
          return false;
      }
  });
  ```

  在主线程注册回调，当主线程"空闲"时才执行回调中的逻辑。

  适用场景：非必需的页面元素的加载和渲染，比如未读信息的红点、新手引导等。

### 2. 绘制优化

#### 1. 有没有做过UI方面的优化，做过哪些?

[Android性能优化（二）之布局优化面面观](https://www.jianshu.com/p/4f44a178c547)

- 调试GPU过度绘制，将Overdraw降低到合理范围内；
- 减少嵌套层次及控件个数，保持view的树形结构尽量扁平（使用Hierarchy Viewer可以方便的查看），同时移除所有不需要渲染的view；
- 使用GPU配置渲染工具，定位出问题发生在具体哪个步骤，使用TraceView精准定位代码；
- 使用标签，merge减少嵌套层次、viewStub延迟初始化、include布局重用 (与merge配合使用)

#### 2. 如何优化自定义View

为了加速你的view，对于频繁调用的方法，需要尽量减少不必要的代码。先从onDraw开始，需要特别注意不应该在这里做内存分配的事情，因为它会导致GC，从而导致卡顿。在初始化或者动画间隙期间做分配内存的动作。不要在动画正在执行的时候做内存分配的事情。

你还需要尽可能的减少onDraw被调用的次数，大多数时候导致onDraw都是因为调用了`invalidate()`。因此请尽量减少调用`invaildate()`的次数。如果可能的话，尽量调用含有4个参数的`invalidate()`方法而不是没有参数的`invalidate()`。没有参数的invalidate会强制重绘整个view。

另外一个非常耗时的操作是请求layout。任何时候执行`requestLayout()`，会使得Android UI系统去遍历整个View的层级来计算出每一个view的大小。如果找到有冲突的值，它会需要重新计算好几次。另外需要尽量保持View的层级是扁平化的，这样对提高效率很有帮助。

如果你有一个复杂的UI，你应该考虑写一个自定义的ViewGroup来执行他的layout操作。与内置的view不同，自定义的view可以使得程序仅仅测量这一部分，这避免了遍历整个view的层级结构来计算大小。



### 3. 内存优化

需要更全面更深入的理解请查看[Android性能优化之内存优化](https://juejin.im/post/6844904096541966350)、[深入探索Android内存优化](https://jsonchao.github.io/2019/12/29/深入探索Android内存优化/)

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
   然后在销毁的生命周期中判断对象是否被回收。弱引用在定义的时候可以指定引用对象和一个 `ReferenceQueue`，通过该弱引用是否被加入`ReferenceQueue`就可以判断该对象是否被回收。
- 分析
   最后通过haha库来分析`hprof`文件，从而找出类之前的引用关系。

#### 3. 内存泄漏有什么方式检测？用过哪些工具，其中的原理是什么？

[Java内存问题 及 LeakCanary 原理分析](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5ab8d3d46fb9a028ca52f813)：

基本原理：用ActivityLifecycleCallbacks接口来检测Activity生命周期，主要是在**onDestroy()**方法中，手动调用 GC，然后利用ReferenceQueue+WeakReference 监听对象回收情况 ，来判断是否有释放不掉的引用，再结合dump memory的hpof文件, 用[HaHa](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsquare%2Fhaha)分析出泄漏地方；

LeakCanary会单独开一进程，用来执行分析任务，和监听任务分开处理。Application中可通过processName判断是否是任务执行进程；

利用主线程空闲的时候执行检测任务，在MessageQueue中加入了一个IdleHandler来得到主线程空闲回调；

LeakCanary检测只针对Activiy里的相关对象。其他类无法使用，还得用MAT原始方法

#### 4. 你们内存优化项目的过程是怎么做的？

**分析现状、确认问题**

我们发现我们的APP在内存方面可能存在很大的问题，第一方面的原因是我们的线上的OOM率比较高。第二点呢，我们经常会看到在我们的Android Studio的Profiler工具中内存的抖动比较频繁。这是我一个初步的现状，然后在我们知道了这个初步的现状之后，进行了问题的确认，我们经过一系列的调研以及深入研究，我们最终发现我们的项目中存在以下几点大问题，比如说：内存抖动、内存溢出、内存泄漏，还有我们的Bitmap使用非常粗犷。

**针对性优化**

比如内存抖动的解决 -> Memory Profiler工具的使用（呈现了锯齿张图形） -> 分析到具体代码存在的问题（频繁被调用的方法中出现了日志字符串的拼接），也可以说说内存泄漏或内存溢出的解决。

**效率提升**

为了不增加业务同学的工作量，我们使用了一些工具类或ARTHook这样的大图检测方案,没有任何的侵入性,同时,我们将这些技术教给了大家,然后让大家一起进行工作效率上的提升。

我们对内存优化工具Memory Profiler、MAT的使用比较熟悉，因此针对一系列不同问题的情况，我们写了一系列解决方案的文档，分享给大家。这样，我们整个团队成员的内存优化意识就变强了。

#### 5. 你做了内存优化最大的感受是什么？

**磨刀不误砍柴工**

我们一开始并没有直接去分析项目中代码哪些地方存在内存问题，而是先去学习了Google官方的一些文档，比如说学习了Memory Profiler工具的使用、学习了MAT工具的使用，在我们将这些工具学习熟练之后，当在我们的项目中遇到内存问题时，我们就能够很快地进行排查定位问题进行解决。

**技术优化必须结合业务代码**

一开始，我们做了整体APP运行阶段的一个内存上报，然后，我们在一些重点的内存消耗模块进行了一些监控，但是后面发现这些监控并没有紧密地结合我们的业务代码，比如说在梳理完项目之后，发现我们项目中存在使用多个图片库的情况，多个图片库的内存缓存肯定是不公用的，所以导致我们整个项目的内存使用量非常高。所以进行技术优化时必须结合我们的业务代码。

**系统化完善解决方案**

我们在做内存优化的过程中，不仅做了Android端的优化工作，还将我们Android端一些数据的采集上报到了我们的服务器，然后传到我们的后台，这样，方便我们的无论是Bug跟踪人员或者是Crash跟踪人员进行一系列问题的解决。

**如何检测所有不合理的地方？**

比如说大图片的检测，我们最初的一个方案是通过继承ImageView，重写它的onDraw方法来实现。但是，我们在推广它的过程中，发现很多开发人员并不接受，因为很多ImageView之前已经写过了，你现在让他去替换，工作成本是比较高的。所以说，后来我们就想，有没有一种方案可以免替换，最终我们就找到了ARTHook这样一个Hook的方案。

**如何避免内存抖动？**（代码注意事项）

内存抖动是由于短时间内有大量对象进出新生区导致的，它伴随着频繁的GC，gc会大量占用ui线程和cpu资源，会导致app整体卡顿。

避免发生内存抖动的几点建议：

- 尽量避免在循环体内创建对象，应该把对象创建移到循环体外。
- 注意自定义View的onDraw()方法会被频繁调用，所以在这里面不应该频繁的创建对象。
- 当需要大量使用Bitmap的时候，试着把它们缓存在数组或容器中实现复用。
- 对于能够复用的对象，同理可以使用对象池将它们缓存起来。






### 4. 响应速度优化

#### 1. 有什么实际解决UI卡顿优化的经历

#### 2. App上线后用户使用时卡顿怎么查看是什么原因？



### 5. 启动优化

需要更全面更深入的理解请查看[深入探索Android启动速度优化](https://juejin.im/post/6844904093786308622)

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

而这个install方法就是耗时大a户，会解压apk，遍历dex文件，压缩dex、将dex文件通过反射转换成DexFile对象、反射替换数组。

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

#### 3. 性能优化，怎么保证应用启动不卡顿? 黑白屏怎么处理?

**应用启动速度**，取决于你在application里面时候做了什么事情，比如你集成了很多sdk，并且sdk的init操作都需要在主线程里实现所以会有卡顿的感觉。在非必要的情况下可以把加载延后或则开启子线程处理

另外，影响**界面卡顿**的两大因素，分别是**界面绘制和数据处理。**

- 布局优化(使用include，merge标签，复杂布局推荐使用ConstraintLayout等)
- onCreate() 中不执行耗时操作 把页面显示的 View 细分一下，放在 AsyncTask 里逐步显示，用 Handler 更好。这样用户的看到的就是有层次有步骤的一个个的 View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。
- 利用多线程的目的就是尽可能的减少 onCreate() 和 onReume() 的时间，使得用户能尽快看到页面，操作页面。
- 减少主线程阻塞时间。
- 提高 Adapter 和 AdapterView 的效率。

推荐文章：[Android 性能优化之内存检测、卡顿优化、耗电优化、APK瘦身](https://blog.csdn.net/csdn_aiyang/article/details/74989318)

**黑白屏产生原因**：当我们在启动一个应用时，系统会去检查是否已经存在这样一个进程，如果不存在，系统的服务会先检查startActivity中的intent的信息，然后在去创建进程，最后启动Acitivy，即冷启动。而启动出现白黑屏的问题，就是在这段时间内产生的。系统在绘制页面加载布局之前，首先会初始化窗口（Window），而在进行这一步操作时，系统会根据我们设置的Theme来指定它的Theme 主题颜色，我们在Style中的设置就决定了显示的是白屏还是黑屏。

- windowIsTranslucent和windowNoTitle，将这两个属性都设置成true (会有明显的卡顿体验，不推荐)
- 如果启动页只是是一张图片，那么为启动页专一设置一个新的主题，设置主题的android:windowBackground属性为启动页背景图即可
- 使用layer-list制作一张图片launcher_layer.xml，将其设置为启动页专一主题的背景，并将其设置为启动页布局的背景。

推荐文章：[Android启动页解决攻略](https://blog.csdn.net/zivensonice/article/details/51691136)

#### 4. 启动优化是怎么做的？

- 分析现状、确认问题
- 针对性优化（先概括，引导其深入）
- 长期保持优化效果

在某一个版本之后呢，我们会发现这个启动速度变得特别慢，同时用户给我们的反馈也越来越多，所以，我们开始考虑对应用的启动速度来进行优化。然后，我们就对启动的代码进行了代码层面的梳理，我们发现应用的启动流程已经非常复杂，接着，我们通过一系列的工具来确认是否在主线程中执行了太多的耗时操作。

我们经过了细查代码之后，发现应用主线程中的任务太多，我们就想了一个方案去针对性地解决，也就是进行异步初始化。（引导=>第5题） 然后，我们还发现了另外一个问题，也可以进行针对性的优化，就是在我们的初始化代码当中有些的优先级并不是那么高，它可以不放在Application的onCreate中执行，而完全可以放在之后延迟执行的，因为我们对这些代码进行了延迟初始化，最后，我们还结合了idealHandler做了一个更优的延迟初始化的方案，利用它可以在主线程的空闲时间进行初始化，以减少启动耗时导致的卡顿现象。做完这些之后，我们的启动速度就变得很快了。

最后，我简单说下我们是怎么长期来保持启动优化的效果的。首先，我们做了我们的启动器，并且结合了我们的CI，在线上加上了很多方面的监控。（引导=> 第7题）

#### 5. 是怎么异步的，异步遇到问题没有？

- 体现演进过程
- 详细介绍启动器

我们最初是采用的普通的一个异步的方案，即new Thread + 设置线程优先级为后台线程的方式在Application的onCreate方法中进行异步初始化，后来，我们使用了线程池、IntentService的方式，但是，在我们应用的演进过程当中，发现代码会变得不够优雅，并且有些场景非常不好处理，比如说多个初始化任务直接的依赖关系，比如说某一个初始化任务需要在某一个特定的生命周期中初始化完成，这些都是使用线程池、IntentService无法实现的。所以说，我们就开始思考一个新的解决方案，它能够完美地解决我们刚刚所遇到的这些问题。

这个方案就是我们目前所使用的启动器，在启动器的概念中，我们将每一个初始化代码抽象成了一个Task，然后，对它们进行了一个排序，根据它们之间的依赖关系排了一个有向无环图，接着，使用一个异步队列进行执行，并且这个异步队列它和CPU的核心数是强烈相关的，它能够最大程度地保证我们的主线程和别的线程都能够执行我们的任务，也就是大家几乎都可以同时完成。

#### 6. 启动优化有哪些容易忽略的注意点？

- cpu time与wall time
- 注意延迟初始化的优化
- 介绍下黑科技

首先，在CPU Profiler和Systrace中有两个很重要的指标，即cpu time与wall time，我们必须清楚cpu time与wall time之间的区别，wall time指的是代码执行的时间，而cpu time指的是代码消耗CPU的时间，锁冲突会造成两者时间差距过大。我们需要以cpu time来作为我们优化的一个方向。

其次，我们不仅只追求启动速度上的一个提升，也需要注意延迟初始化的一个优化，对于延迟初始化，通常的做法是在界面显示之后才去进行加载，但是如果此时界面需要进行滑动等与用户交互的一系列操作，就会有很严重的卡顿现象，因此我们使用了idealHandler来实现cpu空闲时间来执行耗时任务，这极大地提升了用户的体验，避免了因启动耗时任务而导致的页面卡顿现象。

最后，对于启动优化，还有一些黑科技，首先，就是我们采用了类预先加载的方式，我们在MultiDex.install方法之后起了一个线程，然后用Class.forName的方式来预先触发类的加载，然后当我们这个类真正被使用的时候，就不用再进行类加载的过程了。同时，我们再看Systrace图的时候，有一部分手机其实并没有给我们应用去跑满cpu，比如说它有8核，但是却只给了我们4核等这些情况，然后，有些应用对此做了一些黑科技，它会将cpu的核心数以及cpu的频率在启动的时候去进行一个暴力的提升。

#### 7. 版本迭代导致的启动变慢有好的解决方式吗？

- 启动器
- 结合CI
- 监控完善

这种问题其实我们之前也遇到过，这的确非常难以解决。但是，我们后面对此进行了反复的思考与尝试，终于找到了一个比较好的解决方式。

首先，我们使用了启动器去管理每一个初始化任务，并且启动器中每一个任务的执行都是被其自动进行分配的，也就是说这些自动分配的task我们会尽量保证它会平均分配在我们每一个线程当中的，这和我们普通的异步是不一样的，它可以很好地缓解我们应用的启动变慢。

其次，我们还结合了CI，比如说，我们现在限制了一些类，如Application，如果有人修改了它，我们不会让这部分代码合并到主干分支或者是修改之后会有一些内部的工具如邮件的形式发送到我，然后，我就会和他确认他加的这些代码到底是耗时多少，能否异步初始化，不能异步的话就考虑延迟初始化，如果初始化时间太长，则可以考虑是否能进行懒加载，等用到的时候再去使用等等。

然后，我们会将问题尽可能地暴露在上线之前。同时，我们真正已经到了线上的一个环境下时，我们进行了监控的一个完善，我们不仅是监控了App的整个的启动时间，同时呢，我们也将每一个生命周期都进行了一个监控。比如说Application的onCreate与onAttachBaseContext方法的耗时，以及这两个生命周期之间间隔的时间，我们都进行了一个监控，如果说下一次我们发现了这个启动速度变慢了，我们就可以去查找到底是哪一个环节变慢了，我们会和以前的版本进行对比，对比完成之后呢，我们就可以来找这一段新加的代码。

#### 8. [开放问题：如果提高启动速度，设计一个延迟加载框架或者sdk的方法和注意的问题](https://blog.csdn.net/dd864140130/article/details/53558011)

#### 9. Android怎么加速启动Activity？

- onCreate() 中不执行耗时操作：把页面显示的 View 细分一下，放在 AsyncTask 里逐步显示，用 Handler 更好。这样用户的看到的就是有层次有步骤的一个个的 View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。
- 利用多线程的目的就是尽可能的减少 onCreate() 和 onReume() 的时间，使得用户能尽快看到页面，操作页面。
- 减少主线程阻塞时间。
- 提高 Adapter 和 AdapterView 的效率。
- 优化布局文件。

### 6. Bitmap优化

#### 1. 有做过什么Bitmap优化的实际经验

#### 2. 图片的三级缓存中,图片加载到内存中,如果内存快爆了,会发生什么？怎么处理？

首先我们要清楚图片的三级缓存是如何的

![img](https://user-gold-cdn.xitu.io/2019/3/19/169956a4db82e9f3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如果内存足够时不回收。内存不够时就回收软引用对象。

### 7. 线程优化



### 8. RecycleView优化



### 9. 网络优化

#### 1. 移动端获取网络数据优化的几个点

- 连接复用：节省连接建立时间，如开启 keep-alive。于Android来说默认情况下HttpURLConnection和HttpClient都开启了keep-alive。只是2.2之前HttpURLConnection存在影响连接池的Bug。
- 请求合并：即将多个请求合并为一个进行请求，比较常见的就是网页中的CSS Image Sprites。如果某个页面内请求过多，也可以考虑做一定的请求合并。
- 减少请求数据的大小：对于post请求，body可以做gzip压缩的，header也可以做数据压缩(不过只支持http 2.0)。

返回数据的body也可以做gzip压缩，body数据体积可以缩小到原来的30%左右（也可以考虑压缩返回的json数据的key数据的体积，尤其是针对返回数据格式变化不大的情况，支付宝聊天返回的数据用到了）。

- 根据用户的当前的网络质量来判断下载什么质量的图片（电商用的比较多）。
- 使用HttpDNS优化DNS：DNS存在解析慢和DNS劫持等问题，DNS 不仅支持 UDP，它还支持 TCP，但是大部分标准的 DNS 都是基于 UDP 与 DNS 服务器的 53 端口进行交互。HTTPDNS 则不同，顾名思义它是利用 HTTP 协议与 DNS 服务器的 80 端口进行交互。不走传统的 DNS 解析，从而绕过运营商的 LocalDNS 服务器，有效的防止了域名劫持，提高域名解析的效率。

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095ea619dfe002?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

[参考文章](https://www.jianshu.com/p/940be2e758ee)

#### 2. [客户端网络安全实现](http://mrpeak.cn/blog/encrypt/)

#### 3. 设计一个网络优化方案，针对移动端弱网环境?

#### 4. 怎么优化DNS解析

- 安全优化

解决方案就是用`HTTPDNS`。

`HTTPDNS`是一个新概念，他会绕过传统的运营商DNS服务器，不走传统的DNS解析。而是换成HTTP协议，直接通过HTTP协议进行请求某个DNS服务器集群，获取地址。

- 由于绕过了`运营商`，所以可以避免域名被劫持。
- 它是基于访问的`来源ip`，所以能获得更准确的解析结果
- 会有`预解析`，`解析缓存`等功能，所以解析延迟也很小

所以首先的优化，针对安全方面，就是要替换成`HTTPDNS`解析方式，就要借用阿里云和腾讯云等服务，但是这些服务可不是免费的，免费的有七牛云的 `happy-dns`。

添加依赖库，然后去实现Okhttp的DNS接口即可，简单写个例子：

```java
//导入库
    implementation 'com.qiniu:happy-dns:0.2.13'
    implementation 'com.qiniu.pili:pili-android-qos:0.8'


//实现DNS接口
public class HttpDns implements Dns {

    private DnsManager dnsManager;

    public HttpDns() {
        IResolver[] resolvers = new IResolver[1];
        try {
            resolvers[0] = new Resolver(InetAddress.getByName("119.29.29.29"));
            dnsManager = new DnsManager(NetworkInfo.normal, resolvers);
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
    }

    @Override
    public List<InetAddress> lookup(String hostname) throws UnknownHostException {
        if (dnsManager == null)  //当构造失败时使用默认解析方式
            return Dns.SYSTEM.lookup(hostname);

        try {
            String[] ips = dnsManager.query(hostname);  //获取HttpDNS解析结果
            if (ips == null || ips.length == 0) {
                return Dns.SYSTEM.lookup(hostname);
            }

            List<InetAddress> result = new ArrayList<>();
            for (String ip : ips) {  //将ip地址数组转换成所需要的对象列表
                result.addAll(Arrays.asList(InetAddress.getAllByName(ip)));
            }
            //在返回result之前，我们可以添加一些其他自己知道的IP
            return result;
        } catch (IOException e) {
            e.printStackTrace();
        }
        //当有异常发生时，使用默认解析
        return Dns.SYSTEM.lookup(hostname);
    }
}


//替换okhttp的dns解析
OkHttpClient okHttpClient = new OkHttpClient.Builder().dns(new HttpDns()).build();
```

- 速度优化

如果在测试环境，其实我们可以直接配置ip白名单，然后跳过DNS解析流程，直接获取ip地址。比如：

```java
    private static class TestDNS implements Dns{
        @Override
        public List<InetAddress> lookup(@NotNull String hostname) throws UnknownHostException {
            if ("www.test.com".equalsIgnoreCase(hostname)){
                InetAddress byAddress=InetAddress.getByAddress(hostname,new byte[]{(byte)192,(byte)168,1,1});
                return Collections.singletonList(byAddress);
            }else {
                return Dns.SYSTEM.lookup(hostname);
            }
        }
    }
```

#### 5. DNS解析超时怎么办

当我们在用OkHttp做网络请求时，如果网络设备切换路由，访问网络出现长时间无响应，很久之后会抛出 `UnknownHostException`。虽然我们在OkHttp中设置了`connectTimeout`超时时间，但是它其实对DNS的解析是不起作用的。

这种情况我们就需要在自定义的Dns类中做超时判断：

```java
public class TimeDns implements Dns {
    private long timeout;

    public TimeDns(long timeout) {
        this.timeout = timeout;
    }

    @Override
    public List<InetAddress> lookup(final String hostname) throws UnknownHostException {
        if (hostname == null) {
            throw new UnknownHostException("hostname == null");
        } else {
            try {
                FutureTask<List<InetAddress>> task = new FutureTask<>(
                        new Callable<List<InetAddress>>() {
                            @Override
                            public List<InetAddress> call() throws Exception {
                                return Arrays.asList(InetAddress.getAllByName(hostname));
                            }
                        });
                new Thread(task).start();
                return task.get(timeout, TimeUnit.MILLISECONDS);
            } catch (Exception var4) {
                UnknownHostException unknownHostException =
                        new UnknownHostException("Broken system behaviour for dns lookup of " + hostname);
                unknownHostException.initCause(var4);
                throw unknownHostException;
            }
        }
    }
}

//替换okhttp的dns解析
OkHttpClient okHttpClient = new OkHttpClient.Builder().dns(new TimeDns(5000)).build();
```



#### 6. 看视频的时候网络请求很慢怎么优化？



#### 7.当网络带宽足够大，信号足够好，下载大文件，怎么快？

开多个链接，wifi+4G同时，分片下。

协议层 ，udp去下，本地做完整性校验。

#### 8. 设计一个如何处理 app接收到服务器脏数据的方案？



### 10. App电量优化

![image](https://user-gold-cdn.xitu.io/2020/3/1/170960058d314ee7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 11. 安卓的安全优化

#### 1. 提高app安全性的方法？

#### 2. 安卓的app加固如何做？

#### 3. 安卓的混淆原理是什么？

#### 4. 谈谈你对安卓签名的理解。

#### 5. apk安全措施，当apk已经被破解了，怎么处理？

借助v1签名思想，本地做对文件md5的校验。或者借助v3的思想，连续签名。



### 12. 其他

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

#### 5. Apk的大小如何压缩 ？

一个完整APK包含以下目录（将APK文件拖到Android Studio）：

- **META-INF/**：包含**CERT.SF**和**CERT.RSA**签名文件以及**MANIFEST.MF** 清单文件。
- **assets/**：包含应用可以使用**AssetManager**对象检索的应用资源。
- **res/**：包含未编译到的资源 resources.arsc。
- **lib/**：包含特定于处理器软件层的编译代码。该目录包含了每种平台的子目录，像**armeabi，armeabi-v7a， arm64-v8a，x86，x86_64，和mips**。
- **resources.arsc**：包含已编译的资源。该文件包含**res/values/** 文件夹所有配置中的XML内容。打包工具提取此XML内容，将其编译为二进制格式，并将内容归档。此内容包括语言字符串和样式，以及直接包含在`resources.arsc`文件中的内容路径 ，例如布局文件和图像。
- **classes.dex**：包含以**Dalvik / ART**虚拟机可理解的**DEX**文件格式编译的类。
- **AndroidManifest.xml**：包含核心Android清单文件。该文件列出应用程序的名称，版本，访问权限和引用的库文件。该文件使用Android的二进制XML格式。

![img](https://user-gold-cdn.xitu.io/2019/3/27/169bd48e53be9928?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

lib、class.dex和res占用了超过90%的空间，所以这三块是优化Apk大小的重点（实际情况不唯一）

**减少res，压缩图文文件**

图片文件压缩是针对jpg和png格式的图片。我们通常会放置多套不同分辨率的图片以适配不同的屏幕，这里可以进行适当的删减。在实际使用中，只保留一到两套就足够了（保留一套的话建议保留xxhdpi，两套的话就加上hdpi），然后再对剩余的图片进行压缩(jpg采用优图压缩，png尝试采用pngquant压缩)

**减少dex文件大小**

添加资源混淆。`shrinkResources`为true表示移除未引用资源，和代码压缩协同工作。`minifyEnabled`为true表示通过`ProGuard`启用代码压缩，配合`proguardFiles`的配置对代码进行混淆并移除未使用的代码。代码混淆在压缩apk的同时，也提升了安全性。

![img](https://user-gold-cdn.xitu.io/2019/3/27/169be5f6fe340e53?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

推荐文章：[Android混淆最佳实践](https://www.jianshu.com/p/cba8ca7fc36d)

**减少lib文件大小**

- 由于引用了很多第三方库，lib文件夹占用的空间通常都很大，特别是有so库的情况下。很多so库会同时引入armeabi、armeabi-v7a和x86这几种类型，这里可以只保留armeabi或armeabi-v7a的其中一个就可以了，实际上微信等主流app都是这么做的。
- 只需在build.gradle直接配置即可，NDK配置同理

![img](https://user-gold-cdn.xitu.io/2019/3/27/169be64df0148df3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

推荐文章：[APK瘦身](https://www.jianshu.com/p/5921e9561f5f)

![image](https://user-gold-cdn.xitu.io/2020/3/2/17098b45835e575c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 六、插件化

#### 1. 为什么需要插件化？插件化的主要优点和缺点是什么？

我觉得最主要的原因是可以动态扩展功能。

把一些不常用的功能或者模块做成`插件`，就能减少原本的安装包大小，让一些功能以插件的形式在被需要的时候被加载，也就是实现了`动态加载`。

比如`动态换肤、节日促销、见不得人`的一些功能，就可以在需要的时候去下载相应模式的apk，然后再动态加载功能。所以一般这个功能适用于一些平台类的项目，比如大众点评美团这种，功能很多，用户很大概率只会用其中的一些功能，而且这些模块单独拿出来都可以作为一个app运行。但是现在用的却很少了。

**优点**

- 低耦合
- 应用间的接入和维护更便捷，每个应用团队只需要负责自己的那一部分。
- 应用及主dex的体积也会相应变小，间接地避免了65536限制。
- 第一次加载到内存的只有淘宝客户端，当使用到其它插件时才会加载相应插件到内存，以减少内存占用。

**缺点**

**对比**：

- 最早的插件化框架：2012年大众点评的屠毅敏就推出了AndroidDynamicLoader框架。
- 目前主流的插件化方案有滴滴任玉刚的VirtualApk、360的DroidPlugin、RePlugin、Wequick的Small框架。
- 如果加载的插件不需要和宿主有任何耦合，也无须和宿主进行通信，比如加载第三方App，那么推荐使用RePlugin，其他情况推荐使用VirtualApk。由于VirtualApk在加载耦合插件方面是插件化框架的首选，具有普遍的适用性，因此有必要对它的源码进行了解。

#### 2. 插件化的原理是怎样的？

**插件化**是指将 APK 分为**宿主**和**插件**的部分。把需要实现的模块或功能当做一个独立的提取出来，在 APP 运行时，我们可以动态的**载入**或者**替换插件**部分，减少**宿主**的规模

- 宿主： 就是当前运行的APP。
- 插件： 相对于插件化技术来说，就是要加载运行的apk类文件。

![img](https://user-gold-cdn.xitu.io/2019/4/7/169f6c21ae14fee0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> **类加载机制**
>
> Android中常用的两种类加载器，**DexClassLoader**和**PathClassLoader**，它们都继承于**BaseDexClassLoader**，两者**区别**在于**PathClassLoader**只能加载**内部存储目录**的dex/jar/apk文件。**DexClassLoader**支持加载**指定目录**(不限于内部)的dex/jar/apk文件
>
> **插件通信**：通过给插件apk生成相应的DexClassLoader便可以访问其中的类，可分为单DexClassLoader和多DexClassLoader两种结构。
>
> - 若使用**多ClassLoader机制**，主工程引用插件中类需要先通过插件的ClassLoader加载该类再通过**反射**调用其方法。插件化框架一般会通过统一的入口去管理对各个插件中类的访问，并且做一定的限制。
> - 若使用**单ClassLoader机制**，主工程则可以**直接通过**类名去访问插件中的类。该方式有个弊端，若两个不同的插件工程引用了一个库的不同版本，则程序可能会出错。
>
> **资源加载**
>
> 原理在于通过反射将插件apk的路径加入AssetManager中并创建Resource对象加载资源，有两种处理方式：
>
> - 合并式：addAssetPath时加入所有插件和主工程的路径；由于AssetManager中加入了所有插件和主工程的路径，因此生成的Resource可以同时访问插件和主工程的资源。但是由于主工程和各个插件都是独立编译的，生成的资源id会存在相同的情况，在访问时会产生资源冲突。
> - 独立式：各个插件只添加自己apk路径，各个插件的资源是互相隔离的，不过如果想要实现资源的共享，必须拿到对应的Resource对象。
>
> 推荐文章：
>
> - [Android动态加载技术 简单易懂的介绍方式](https://segmentfault.com/a/1190000004062866#articleHeader1)
> - [深入理解Android插件化技术](https://yq.aliyun.com/articles/361233?utm_content=m_40296)
> - [为什么要做热更新](https://www.cnblogs.com/baiqiantao/p/9160806.html)

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

#### 8. Hook以及插桩技术

**Hook**是一种用于**改变API执行结果**的技术，能够将系统的API函数执行**重定向**（应用的**触发事件**和**后台逻辑处理**是根据事件流程一步步地向下执行。而**Hook**的意思，就是在事件传送到终点前截获并监控事件的传输，像个钩子钩上事件一样，并且能够在钩上事件时，处理一些自己特定的事件，例如逆向破解App）

![img](https://user-gold-cdn.xitu.io/2019/4/17/16a2a2f8cf3e4448?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Android 中的 Hook 机制，大致有两个方式：

- 要 root 权限，直接 Hook 系统，可以干掉所有的 App。
- 无 root 权限，但是只能 Hook 自身app，对系统其它 App 无能为力。

**插桩**是以静态的方式修改第三方的代码，也就是从编译阶段，对源代码（中间代码）进行编译，而后重新打包，是**静态的篡改**； 而**Hook**则不需要再编译阶段修改第三方的源码或中间代码，是在运行时通过反射的方式修改调用，是一种**动态的篡改**

推荐文章：

- [Android插件化原理解析——Hook机制之动态代理](http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/)
- [android 插桩基本概念](https://blog.csdn.net/fei20121106/article/details/51879047)
- [Android逆向之旅](http://www.520monkey.com/)




## 七、组件化

#### 1. 组件化有详细了解过吗？

**引入组件化的原因**：项目随着需求的增加规模变得越来越大，规模的增大导致了各种业务错中复杂的交织在一起, 每个业务模块之间，代码没有约束，带来了代码边界的模糊，代码冲突时有发生, 更改一个小问题可能引起一些新的问题, 牵一发而动全身，增加一个新需求，需要熟悉相关的代码逻辑，增加开发时间

- **避免重复造轮子，可以节省开发和维护的成本。**
- **可以通过组件和模块为业务基准合理地安排人力，提高开发效率。**
- **不同的项目可以共用一个组件或模块，确保整体技术方案的统一性。**
- **为未来插件化共用同一套底层模型做准备。**

**组件化开发流程**就是把一个功能完整的App或模块拆分成**多个子模块（Module）**，每个子模块可以**独立编译运行**，也可以任意组合成另一个新的 App或模块，每个模块即不相互依赖但又可以相互交互，但是最终发布的时候是将这些组件合并统一成一个apk，遇到某些特殊情况甚至可以**升级**或者**降级**

举个简单的模型例子：

![APP架构图](https://user-gold-cdn.xitu.io/2019/4/7/169f692c69f60c06?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) ![APP代码结构图](https://user-gold-cdn.xitu.io/2019/4/7/169f690af8eea279?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) App是主application，ModuleA和ModuleB是两个业务模块（**相对独立，互不影响**），Library是基础模块，包含所有模块需要的依赖库，以及一些工具类：如网络访问、时间工具等

> 提供给各业务模块的基础组件，需要根据具体情况拆分成 aar 或者 library，像登录，基础网络层这样较为稳定的组件，一般直接打包成 aar，减少编译耗时。而像自定义 View 组件，由于随着版本迭代会有较多变化，就直接以源码形式抽离成 Library

推荐文章：[干货 | 从智行 Android 项目看组件化架构实践](https://mp.weixin.qq.com/s?__biz=MjM5MDI3MjA5MQ==&mid=2697268363&idx=1&sn=3db2dce36a912936961c671dd1f71c78&scene=21#wechat_redirect)




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

#### 3. ARouter怎么实现接口调用

#### 4. ARouter怎么实现页面拦截

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

#### 6.如果不用ARouter，你会怎么去解藕。接口？设计接口有什么需要注意的？

#### 7. 跨组件通信

组件通信场景：

- 第一种是组件之间的页面跳转 (Activity 到 Activity, Fragment 到 Fragment, Activity 到 Fragment, Fragment 到 Activity) 以及跳转时的数据传递 (基础数据类型和可序列化的自定义类类型)。
- 第二种是组件之间的自定义类和自定义方法的调用(组件向外提供服务)。

跨组件通信方案分析：

- 第一种**组件之间的页面跳转**实现简单，跳转时想传递不同类型的数据提供有相应的 API即可。

- 第二种组件之间的自定义类和

  自定义方法的调用

  要稍微复杂点，需要 ARouter 配合架构中的 公共服务(CommonService) 实现：

  - 提供服务的业务模块：
  - 在公共服务(CommonService) 中声明 Service 接口 (含有需要被调用的自定义方法), 然后在自己的模块中实现这个 Service 接口, 再通过 ARouter API 暴露实现类。
  - 使用服务的业务模块：
    - 通过 ARouter 的 API 拿到这个 Service 接口(多态持有, 实际持有实现类), 即可调用 Service 接口中声明的自定义方法, 这样就可以达到模块之间的交互。
  - 此外，可以使用 AndroidEventBus 其独有的 Tag, 可以在开发时更容易定位发送事件和接受事件的代码, 如果以组件名来作为 Tag 的前缀进行分组, 也可以更好的统一管理和查看每个组件的事件, 当然也不建议大家过多使用 EventBus。

如何管理过多的路由表？

- RouterHub 存在于基础库, 可以被看作是所有组件都需要遵守的通讯协议, 里面不仅可以放路由地址常量, 还可以放跨组件传递数据时命名的各种 Key 值, 再配以适当注释, 任何组件开发人员不需要事先沟通只要依赖了这个协议, 就知道了各自该怎样协同工作, 既提高了效率又降低了出错风险, 约定的东西自然要比口头上说强。
- Tips: 如果您觉得把每个路由地址都写在基础库的 RouterHub 中, 太麻烦了, 也可以在每个组件内部建立一个私有 RouterHub, 将不需要跨组件的路由地址放入私有 RouterHub 中管理, 只将需要跨组件的路由地址放入基础库的公有 RouterHub 中管理, 如果您不需要集中管理所有路由地址的话, 这也是比较推荐的一种方式。

ARouter路由原理：

- ARouter维护了一个路由表Warehouse，其中保存着全部的模块跳转关系，ARouter路由跳转实际上还是调用了startActivity的跳转，使用了原生的Framework机制，只是通过apt注解的形式制造出跳转规则，并人为地拦截跳转和设置跳转条件。

常见的组件化方案如下：


![img](https://user-gold-cdn.xitu.io/2019/4/7/169f6a3d472ef431?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


#### 8. 组件化中路由、埋点的实现

因为在组件化中，各个业务模块之间是各自**独立**的, 并不会存在相互依赖的关系, 所以一个业务模块是访问不了其他业务模块的代码的, 如果想从 A 业务模块的 A 页面跳转到 B 业务模块的 B 页面, 光靠模块自身是不能实现的，这就需要一种跨组件通信方案—— **路由（Router）**

路由

主要有以下两种场景:

- 第一种是**组件之间的页面跳转** (Activity 到 Activity, Fragment 到 Fragment, Activity 到 Fragment, Fragment 到 Activity) 以及跳转时的数据传递 (基础数据类型和可序列化的自定义类类型)
- 第二种是**组件之间的自定义类**和**自定义方法的调用**(组件向外提供服务)

其**原理**在于将分布在不同组件module中的某些类按照一定规则生成映射表（数据结构通常是Map，Key为一个字符串，Value为类或对象），然后在需要用到的时候从映射表中根据字符串从映射表中取出类或对象，本质上是类的查找

埋点则是在应用中特定的流程收集一些信息，用来跟踪应用使用的状况

- **代码埋点**：在某个事件发生时调用SDK里面相应的接口发送埋点数据，百度统计、友盟、TalkingData、Sensors Analytics等第三方数据统计服务商大都采用这种方案
- **全埋点**：全埋点指的是将Web页面/App内产生的所有的、满足某个条件的行为，全部上报到后台服务器
- **可视化埋点**：通过可视化工具（例如Mixpanel）配置采集节点，在Android端自动解析配置并上报埋点数据，从而实现所谓的**自动埋点**
- **无埋点**：它并不是真正的不需要埋点，而是Android端自动采集全部事件并上报埋点数据，在后端数据计算时过滤出有用数据

推荐文章：[安卓组件化开源方案实现](https://juejin.im/post/6844903565035569166)

#### 9. 如何管理过多的路由表？

## 八、热修复

#### 1. tinker的原理是什么,还用过什么热修复框架，robust的原理是什么？

#### 2. 热修复的原理，资源的热修复的原理,会不会有资源冲突的问题

#### 3. 热修复技术是怎样实现的，和插件化有什么区别？

插件化：动态加载主要解决3个技术问题：

- 1、使用ClassLoader加载类。
- 2、资源访问。
- 3、生命周期管理。

插件化是体现在功能拆分方面的，它将某个功能独立提取出来，独立开发，独立测试，再插入到主应用中。以此来减少主应用的规模。

热修复：为了修复Bug的。

**代码热修复原理**：

- 将编译好的class文件拆分打包成两个dex，绕过dex方法数量的限制以及安装时的检查，在运行时再动态加载第二个dex文件中。
- 热修复是体现在bug修复方面的，它实现的是不需要重新发版和重新安装，就可以去修复已知的bug。
- 利用PathClassLoader和DexClassLoader去加载与bug类同名的类，替换掉bug类，进而达到修复bug的目的，原理是在app打包的时候阻止类打上CLASS_ISPREVERIFIED标志，然后在热修复的时候动态改变BaseDexClassLoader对象间接引用的dexElements，替换掉旧的类。

相同点:

都使用ClassLoader来实现加载新的功能类，都可以使用PathClassLoader与DexClassLoader。

不同点：

热修复因为是为了修复Bug的，所以要将新的类替代同名的Bug类，要抢先加载新的类而不是Bug类，所以多做两件事：在原先的app打包的时候，阻止相关类去打上CLASS_ISPREVERIFIED标志，还有在热修复时动态改变BaseDexClassLoader对象间接引用的dexElements，这样才能抢先代替Bug类，完成系统不加载旧的Bug类.。  而插件化只是增加新的功能类或者是资源文件，所以不涉及抢先加载新的类这样的使命，就避过了阻止相关类去打上CLASS_ISPREVERIFIED标志和还有在热修复时动态改变BaseDexClassLoader对象间接引用的dexElements.

所以插件化比热修复简单，热修复是在插件化的基础上在进行替换旧的Bug类。

**热修复原理**

**1.资源修复**：

很多热修复框架的资源修复参考了Instant Run的资源修复的原理。

传统编译部署流程如下：

Instant Run编译部署流程如下：

- Hot Swap：修改一个现有方法中的代码时会采用Hot Swap。
- Warm Swap：修改或删除一个现有的资源文件时会采用Warm Swap。
- Cold Swap：有很多情况，如添加、删除或修改一个字段和方法、添加一个类等。

Instant Run中的资源热修复流程：

- 1、创建新的AssetManager，通过反射调用addAssetPath方法加载外部的资源，这样新创建的AssetManager就含有了外部资源。
- 2、将AssetManager类型的mAssets字段的引用全部替换为新创建的AssetManager。

**2.代码修复**：

1、类加载方案：

65536限制：

65536的主要原因是DVM Bytecode的限制，DVM指令集的方法调用指令invoke-kind索引为16bits，最多能引用65535个方法。

LinearAlloc限制：

- DVM中的LinearAlloc是一个固定的缓存区，当方法数超过了缓存区的大小时会报错。

Dex分包方案主要做的是在打包时将应用代码分成多个Dex，将应用启动时必须用到的类和这些类的直接引用类放到Dex中，其他代码放到次Dex中。当应用启动时先加载主Dex，等到应用启动后再动态地加载次Dex，从而缓解了主Dex的65536限制和LinearAlloc限制。

加载流程：

- 根据dex文件的查找流程，我们将有Bug的类Key.class进行修改，再将Key.class打包成包含dex的补丁包Patch.jar，放在Element数组dexElements的第一个元素，这样会首先找到Patch.dex中的Key.class去替换之前存在Bug的Key.class，排在数组后面的dex文件中存在Bug的Key.class根据ClassLoader的双亲委托模式就不会被加载。

类加载方案需要重启App后让ClassLoader重新加载新的类，为什么需要重启呢？

- 这是因为类是无法被卸载的，要想重新加载新的类就需要重启App，因此采用类加载方案的热修复框架是不能即时生效的。

各个热修复框架的实现细节差异：

- QQ空间的超级补丁和Nuwa是按照上面说的将补丁包放在Element数组的第一个元素得到优先加载。
- 微信的Tinker将新旧APK做了diff，得到path.dex，再将patch.dex与手机中APK的classes.dex做合并，生成新的classes.dex，然后在运行时通过反射将classes.dex放在Elements数组的第一个元素。
- 饿了么的Amigo则是将补丁包中每个dex对应的Elements取出来，之后组成新的Element数组，在运行时通过反射用新的Elements数组替换掉现有的Elements数组。

2、底层替换方案：

当我们要反射Key的show方法，会调用Key.class.getDeclaredMethod("show").invoke(Key.class.newInstance());，最终会在native层将传入的javaMethod在ART虚拟机中对应一个ArtMethod指针，ArtMethod结构体中包含了Java方法的所有信息，包括执行入口、访问权限、所属类和代码执行地址等。

替换ArtMethod结构体中的字段或者替换整个ArtMethod结构体，这就是底层替换方案。

AndFix采用的是替换ArtMethod结构体中的字段，这样会有兼容性问题，因为厂商可能会修改ArtMethod结构体，导致方法替换失败。

Sophix采用的是替换整个ArtMethod结构体，这样不会存在兼容问题。

底层替换方案直接替换了方法，可以立即生效不需要重启。采用底层替换方案主要是阿里系为主，包括AndFix、Dexposed、阿里百川、Sophix。

3、Instant Run方案：

什么是ASM？

ASM是一个java字节码操控框架，它能够动态生成类或者增强现有类的功能。ASM可以直接产生class文件，也可以在类被加载到虚拟机之前动态改变类的行为。

Instant Run在第一次构建APK时，使用ASM在每一个方法中注入了类似的代码逻辑：当change不为null时，则调用它的accesschange不为null时，则调用它的accesschange不为null时，则调用它的accessdispatch方法，参数为具体的方法名和方法参数。当MainActivity的onCreate方法做了修改，就会生成替换类MainActivityoverride，这个类实现了IncrementalChange接口，同时也会生成一个AppPatchesLoaderImpl类，这个类的getPatchedClasses方法会返回被修改的类的列表（里面包含了MainActivity），根据列表会将MainActivity的override，这个类实现了IncrementalChange接口，同时也会生成一个AppPatchesLoaderImpl类，这个类的getPatchedClasses方法会返回被修改的类的列表（里面包含了MainActivity），根据列表会将MainActivity的override，这个类实现了IncrementalChange接口，同时也会生成一个AppPatchesLoaderImpl类，这个类的getPatchedClasses方法会返回被修改的类的列表（里面包含了MainActivity），根据列表会将MainActivity的change设置为MainActivityoverride。最后这个override。最后这个override。最后这个change就不会为null，则会执行MainActivityoverride的accessoverride的accessoverride的accessdispatch方法，最终会执行onCreate方法，从而实现了onCreate方法的修改。

借鉴Instant Run原理的热修复框架有Robust和Aceso。

**3. 动态链接库修复**：

重新加载so。

加载so主要用到了System类的load和loadLibrary方法，最终都会调用到nativeLoad方法。其会调用JavaVMExt的LoadNativeLibrary函数来加载so。

so修复主要有两个方案：

- 1、将so补丁插入到NativeLibraryElement数组的前部，让so补丁的路径先被返回和加载。
- 2、调用System的load方法来接管so的加载入口。



## 九、NDK

#### 1. 对JNI是否了解

Java的优点是**跨平台**，但也因为其跨平台的的特性导致其**本地交互的能力不够强大**，一些和操作系统相关的的特性Java无法完成，于是**Java提供JNI专门用于和本地代码交互，通过JNI，用户可以调用C、C++编写的本地代码**

NDK是Android所提供的一个工具集合，通过NDK可以在Android中更加方便地通过JNI访问本地代码，其优点在于

- 提高代码的安全性。由于so库反编译困难，因此NDK提高了Android程序的安全性
- 可以很方便地使用目前已有的C/C++开源库
- 便于平台的移植。通过C/C++实现的动态库可以很方便地在其它平台上使用
- 提高程序在某些特定情形下的执行效率，但是并不能明显提升Android程序的性能

**Java调用C++**

- 在Java中声明Native方法（即需要调用的本地方法）
- 编译上述 Java源文件javac（得到 .class文件） 3。 通过 javah 命令导出JNI的头文件（.h文件）
- 使用 Java需要交互的本地代码 实现在 Java中声明的Native方法
- 编译.so库文件
- 通过Java命令执行 Java程序，最终实现Java调用本地代码

**C++调用Java**

- 从classpath路径下搜索ClassMethod这个类，并返回该类的Class对象。
- 获取类的默认构造方法ID。
- 查找实例方法的ID。
- 创建该类的实例。
- 调用对象的实例方法。

```java
JNIEXPORT void JNICALL Java_com_study_jnilearn_AccessMethod_callJavaInstaceMethod  (JNIEnv *env, jclass cls)  
{  
  jclass clazz = NULL;  
  jobject jobj = NULL;  
  jmethodID mid_construct = NULL;  
  jmethodID mid_instance = NULL;  
  jstring str_arg = NULL;  
  // 1、从classpath路径下搜索ClassMethod这个类，并返回该类的Class对象  
  clazz = (*env)->FindClass(env, "com/study/jnilearn/ClassMethod");  
  if (clazz == NULL) {  
      printf("找不到'com.study.jnilearn.ClassMethod'这个类");  
      return;  
  }  
    
  // 2、获取类的默认构造方法ID  
  mid_construct = (*env)->GetMethodID(env,clazz, "<init>","()V");  
  if (mid_construct == NULL) {  
      printf("找不到默认的构造方法");  
      return;  
  }  

  // 3、查找实例方法的ID  
  mid_instance = (*env)->GetMethodID(env, clazz, "callInstanceMethod", "(Ljava/lang/String;I)V");  
  if (mid_instance == NULL) {  

      return;  
  }  

  // 4、创建该类的实例  
  jobj = (*env)->NewObject(env,clazz,mid_construct);  
  if (jobj == NULL) {  
      printf("在com.study.jnilearn.ClassMethod类中找不到callInstanceMethod方法");  
      return;  
  }  

  // 5、调用对象的实例方法  
  str_arg = (*env)->NewStringUTF(env,"我是实例方法");  
  (*env)->CallVoidMethod(env,jobj,mid_instance,str_arg,200);  

  // 删除局部引用  
  (*env)->DeleteLocalRef(env,clazz);  
  (*env)->DeleteLocalRef(env,jobj);  
  (*env)->DeleteLocalRef(env,str_arg);  
}  
```



#### 2. 如何加载NDK库 ？如何在JNI中注册Native函数，有几种注册方法 ？

```java
   public class JniTest{
        //加载NDK库 
        static{
            System.loadLirary("jni-test");
        }
    }
```

注册JNI函数的两种方法

- 静态方法
- 动态注册

> [注册JNI函数的两种方式](https://blog.csdn.net/wwj_748/article/details/52347341)
>
> [Android JNI 篇 - 从入门到放弃](https://www.jianshu.com/p/3dab1be3b9a4)

#### 3. 你用JNI来实现过什么功能 ？ 怎么实现的 ？（加密处理、影音方面、图形图像处理)

> 推荐文章：[Android JNI 篇 - ffmpeg 获取音视频缩略图](https://www.jianshu.com/p/411761bd5f5b)

#### 4. so 的加载流程是怎样的，生命周期是怎样的？

这个要从 java 层去看源码分析，是从 ClassLoader 的 PathList 中去找到目标路径加载的，同时 so 是通过 mmap 加载映射到虚拟空间的。生命周期加载库和卸载库时分别调用 JNI_OnLoad() 和 JNI_OnUnload() 方法。




## 十、WebView

#### 1. webView加载本地图片，如何从安全方面考虑



#### 2. 有没有做过什么WebView秒开的一些优化



#### 3. native如何对h5进行鉴权，让某些页面可以调，某些页面不能调



#### 4. jsBridge实现方式



#### 5. 项目中对WebView的功能进行了怎样的增强

#### 

#### 6. 为什么不利用同步方法来做jsBridge交互？同步可以做异步，异步不能做同步



#### 7. webView与js通信

1） Android调用JS代码

**主要有两种方法：**

- 通过WebView的loadUrl（）

```java
//载入JS代码 调用javascript的callJS()方法
mWebView.loadUrl("javascript:callJS()");
```

但是这种不常用，因为它会自动刷新页面而且没有返回值，有点影响交互。

设置与Js交互的权限：`webSettings.setJavaScriptEnabled(true)`

设置允许JS弹窗：`webSettings.setJavaScriptCanOpenWindowsAutomatically(true)`

> WebView只是载体，内容的渲染需要使用webviewChromClient类去实现，通过设置WebChromeClient对象处理JavaScript的对话框。
>
> JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用。

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

该方法比第一种方法效率更高、使用更简洁，因为该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会。

Android 4.4 后才可使用。

2） JS调用Android端代码

**主要有两种方法：**

- 通过WebView的`addJavascriptInterface（）`进行对象映射

  定义一个与JS对象映射关系的Android类：AndroidtoJs：

  - 定义JS需要调用的方法，被JS调用的方法必须加入`@JavascriptInterface`注解。
  - 通过`addJavascriptInterface()`将Java对象映射到JS对象。

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

优点：使用简单，仅将Android对象和JS对象映射即可。

缺点：要注意的是4.2以后，对于被调用的函数以`@JavascriptInterface`进行注解，否则容易出发漏洞，因为js方可以通过反射调用一些本地命令，很危险。

- 通过 WebViewClient 的`shouldOverrideUrlLoading()`方法回调拦截 url

这种方法是通过`shouldOverrideUrlLoading`回调去拦截url，然后进行解析，如果是之前约定好的协议，就调用相应的方法。根据协议的参数，判断是否是所需要的url， 一般根据scheme（协议格式） & authority（协议名）判断（前两个参数）。

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

优点：不存在方式1的漏洞；

缺点：JS获取Android方法的返回值复杂,如果JS想要得到Android方法的返回值，只能通过 WebView 的 `loadUrl()`去执行 JS 方法把返回值传递回去。

#### 8. 如何避免WebView内存泄露

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

3、通过 WebChromeClient 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt()`方法回调拦截JS对话框`alert()`、`confirm()`、`prompt()` 消息：

原理：

Android通过 WebChromeClient 的`onJsAlert()`、`onJsConfirm()`、`onJsPrompt()`方法回调分别拦截JS对话框 （警告框、确认框、输入框），得到他们的消息内容，然后解析即可。

常用的拦截是：拦截 JS的输入框（即prompt()方法），因为只有prompt()可以返回任意类型的值，操作最全面方便、更加灵活；而alert()对话框没有返回值；confirm()对话框只能返回两种状态（确定 / 取消）两个值。

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d30fadf235d549798b205c3e28c2b37b~tplv-k3u1fbpfcp-zoom-1.image)

[Android：你要的WebView与 JS 交互方式 都在这里了](https://blog.csdn.net/carson_ho/article/details/64904691)

#### 9. webView还有哪些可以优化的地

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

推荐文章：[WebView性能、体验分析与优化](https://tech.meituan.com/2017/06/09/webviewperf.html)

#### 10. WebView 与 JS 交互方式，shouldOverrideUrlLoading、onJsPrompt使用有啥区别 

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

#### 11. 如何清除 webview 的缓存
webview 的缓存包括网页数据缓存（存储打开过的页面及资源）、H5 缓存（即 AppCache），webview 会将我们浏览过的网页 url 已经网页文件(css、图片、js 等)保存到数据库表中，如下；

```
/data/data/package_name/database/webview.db
/data/data/package_name/database/webviewCache.db
```

所以，我们只需要根据数据库里的信息进行缓存的处理即可。

#### 12. webview 播放视频，5.0 以上没有全屏播放按钮

实现全屏的时候把 webview 里的视频放到一个 View 里面，然后把 webview隐藏掉；即可实现全屏播放。

#### 13. Webview 中是如何控制显示加载完成的进度条的

在 WebView 的 setWebChromClient() 中 ， 重写 WebChromClient 的 openDialog()和 closeDialog()方法；实现监听进度条的显示与关闭。

#### 14. @JavaScriptInterface为什么不通过多个方法来实现？

4.4以前谷歌的webview存在安全漏洞，网站可以通过js注入就可以随便拿到客户端的重要信息，甚至轻而易举的调用本地代码进行流氓行为，谷歌后来发现有此漏洞后，增加了防御措施，js调用本地代码，开发者必须在代码申明JavascriptInterface。

```java
@SuppressWarnings("javadoc")
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface JavascriptInterface {
}
```

#### 15. 为什么WebView加载会慢呢？

这是因为在客户端中，加载H5页面之前，需要先初始化WebView，在WebView完全初始化完成之前，后续的界面加载过程都是被阻塞的。

优化手段围绕着以下两个点进行：

- 预加载WebView。
- 加载WebView的同时，请求H5页面数据。

因此常见的方法是：

- 全局WebView。
- 客户端代理页面请求。WebView初始化完成后向客户端请求数据。
- asset存放离线包。

除此之外还有一些其他的优化手段：

- 脚本执行慢，可以让脚本最后运行，不阻塞页面解析。
- DNS链接慢，可以让客户端复用使用的域名与链接。
- React框架代码执行慢，可以将这部分代码拆分出来，提前进行解析。






## 十一、架构

#### 1. MVC、MVP和MVVM是什么？

- MVC：Model-View-Controller，是一种分层解偶的框架，Model层提供本地数据和网络请求，View层处理视图，Controller处理逻辑，存在问题是Controller层和View层的划分不明显，Model层和View层的存在耦合。
- MVP：Model-View-Presenter，是对MVC的升级，Model层和View层与MVC的意思一致，但Model层和View层不再存在耦合，而是通过Presenter层这个桥梁进行交流。
- MVVM：Model-View-ViewModel，不同于上面的两个框架，ViewModel持有数据状态，当数据状态改变的时候，会自动通知View层进行更新。

**MVC**

- 视图层(View)
  对应于xml布局文件和java代码动态view部分
- 控制层(Controller)
  MVC中Android的控制层是由Activity来承担的，Activity本来主要是作为初始化页面，展示数据的操作，但是因为XML视图功能太弱，所以Activity既要负责视图的显示又要加入控制逻辑，承担的功能过多。
- 模型层(Model)
  针对业务模型，建立数据结构和相关的类，它主要负责网络请求，数据库处理，I/O的操作。

具有一定的分层，model彻底解耦，controller和view并没有解耦。层与层之间的交互尽量使用回调或者去使用消息机制去完成，尽量避免直接持有controller和view，在android中无法做到彻底分离，但在代码逻辑层面一定要分清。业务逻辑被放置在model层，能够更好的复用和修改增加业务。

**MVP**

通过引入接口BaseView，让相应的视图组件如Activity，Fragment去实现BaseView，实现了视图层的独立，通过中间层Preseter实现了Model和View的完全解耦。MVP彻底解决了MVC中View和Controller傻傻分不清楚的问题，但是随着业务逻辑的增加，一个页面可能会非常复杂，UI的改变是非常多，会有非常多的case，这样就会造成View的接口会很庞大。

**MVVM**

MVP中我们说过随着业务逻辑的增加，UI的改变多的情况下，会有非常多的跟UI相关的case，这样就会造成View的接口会很庞大。而MVVM就解决了这个问题，通过双向绑定的机制，实现数据和UI内容，只要想改其中一方，另一方都能够及时更新的一种设计理念，这样就省去了很多在View层中写很多case的情况，只需要改变数据就行。

**MVVM与DataBinding的关系？**

MVVM是一种思想，DataBinding是谷歌推出的方便实现MVVM的工具。

看起来MVVM很好的解决了MVC和MVP的不足，但是由于数据和视图的双向绑定，导致出现问题时不太好定位来源，有可能数据问题导致，也有可能业务逻辑中对视图属性的修改导致。如果项目中打算用MVVM的话可以考虑使用官方的架构组件ViewModel、LiveData、DataBinding去实现MVVM。

**三者如何选择？**

- 如果项目简单，没什么复杂性，未来改动也不大的话，那就不要用设计模式或者架构方法，只需要将每个模块封装好，方便调用即可，不要为了使用设计模式或架构方法而使用。
- 对于偏向展示型的app，绝大多数业务逻辑都在后端，app主要功能就是展示数据，交互等，建议使用mvvm。
- 对于工具类或者需要写很多业务逻辑app，使用mvp或者mvvm都可。

推荐文章：[MVC、MVP、MVVM，我到底该怎么选？](https://juejin.im/post/6844903632446423054)

[MVC MVP MVVM原理和区别？](http://www.tianmaying.com/tutorial/AndroidMVC)

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



#### 12. 封装Presenter层之后.如果Presenter层数据过大,如何解决?

对于MVP模式来说，P层如果数据逻辑过于臃肿，建议引入**RxJava**或则**Dagger**，越是复杂的逻辑，越能体现RxJava的优越性

#### MVC的情况下怎么把Activity的C和V抽离？

#### MVP 架构中 Presenter 定义为接口有什么好处？

#### MVP如何管理Presenter的生命周期，何时取消网络请求？

#### Fragment如果在Adapter中使用应该如何解耦？

#### 项目框架里有没有Base类，BaseActivity和BaseFragment这种封装导致的问题，以及解决方法？

#### 设计一个音乐播放界面，你会如何实现，用到那些类，如何设计，如何定义接口，如何与后台交互，如何缓存与下载，如何优化(15分钟时间)？

#### 从0设计一款App整体架构，如何去做？

#### 说一款你认为当前比较火的应用并设计？

#### 实现一个库，完成日志的实时上报和延迟上报两种功能，该从哪些方面考虑？

#### 如何实现一个推送，消息推送原理？推送到达率的问题？

一：客户端不断的查询服务器，检索新内容，也就是所谓的pull 或者轮询方式。

二：客户端和服务器之间维持一个TCP/IP长连接，服务器向客户端push。

[blog.csdn.net/clh604/arti…](https://blog.csdn.net/clh604/article/details/20167263)

[www.jianshu.com/p/45202dcd5…](https://www.jianshu.com/p/45202dcd5688)




## 十二、 Android 虚拟机 

#### 1. DVM 和 JVM 的区别？

a) dvm 执行的是.dex 文件，而 jvm 执行的是.class。Android 工程编译后的所有.class 字节码会被 dex 工具抽 取到一个.dex 文件中。 

JVM：`.java -> javac -> .class -> jar -> .jar`

DVM：`.java -> javac -> .class -> dx.bat -> .dex`

b) dvm 是基于寄存器的虚拟机 而 jvm 执行是基于虚拟栈的虚拟机。寄存器存取速度比栈快的多，dvm 可以根 据硬件实现最大的优化，比较适合移动设备。

JVM架构: 堆和栈的架构.

DVM架构: 寄存器(cpu上的一块高速缓存)

c) .class 文件存在很多的冗余信息，dex 工具会去除冗余信息，并把所有的.class 文件整合到.dex 文件中。减少 了 I/O 操作，提高了类的查找速度。

#### 2. Android2个虚拟机的区别（一个5.0之前，一个5.0之后）

什么是Dalvik：Dalvik是Google公司自己设计用于Android平台的Java虚拟机。Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一，它可以支持已转换为.dex(即Dalvik Executable)格式的Java应用程序的运行，.dex格式是专为Dalvik应用设计的一种压缩格式，适合内存和处理器速度有限的系统。Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为独立的Linux进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

什么是ART:Android操作系统已经成熟，Google的Android团队开始将注意力转向一些底层组件，其中之一是负责应用程序运行的Dalvik运行时。Google开发者已经花了两年时间开发更快执行效率更高更省电的替代ART运行时。ART代表Android Runtime,其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time(JIT)编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。ART则完全改变了这套做法，在应用安装的时候就预编译字节码为机器语言，这一机制叫Ahead-Of-Time(AOT)编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

ART优点：

- 系统性能的显著提升。
- 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
- 更长的电池续航能力。
- 支持更低的硬件。

ART缺点：

- 更大的存储空间占用，可能会增加10%-20%。
- 更长的应用安装时间。

#### 3. ART和Davlik中垃圾回收的区别？

#### 4. 安卓采用自动垃圾回收机制，请说下安卓内存管理的原理？

#### 5. 开放性问题：如何设计垃圾回收算法？

#### 6. Android中App是如何沙箱化的,为何要这么做？

#### 7. [一个图片在app中调用R.id后是如何找到的](https://my.oschina.net/u/255456/blog/608229)？



## 十三、其他

#### 1. token放在本地如何保存？如何加密比较好？



#### 2. 说说你对屏幕刷新机制的了解，双重缓冲，三重缓冲，黄油模型



#### 3. 阿里编程规范不建议使用线程池，为什么？

线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

#### 4. 推送sdk底层实现



#### 5. 内存抖动（代码注意事项）

定义：内存抖动是由于短时间内有大量对象进出新生区导致的，它伴随着频繁的GC，gc会大量占用ui线程和cpu资源，会导致app整体卡顿。

避免发生内存抖动的几点建议：

- 尽量避免在循环体内创建对象，应该把对象创建移到循环体外 
- 注意自定义View的onDraw()方法会被频繁调用，所以在这里面不应该频繁的创建对象
- 当需要大量使用Bitmap的时候，试着把它们缓存在数组或容器中实现复用
- 对于能够复用的对象，同理可以使用对象池将它们缓存起来

#### 6. 如何绕过9.0限制？

- 如何限制？

  1、阻止java反射和JNI。

  2、当获取方法或Field时进行检测。

  3、怎么检测？

  - 区分出是系统调用还是开发者调用：
    - 根据堆栈，回溯Class，查看ClassLoader是否是BootStrapClassLoader。

  - 区分后，再区分是否是hidden api：
    - Method，Field都有access_flag，有一些备用字段，hidden信息存储其中。

- 如何绕过？

  1、不用反射：

  - 利用一个fakelib，例如写一个android.app.ActivityThread#currentActivityThread空实现，直接调用；

  2、伪装系统调用：

  - jni修改一个class的classloder为BootStrapClassLoader，麻烦。

  - 利用系统方法去反射：
    - 利用原反射，即：getDeclaredMethod这个方法是系统的方法，通过getDeclaredmethod反射去执行hidden api。

  3、修改Method，Field中存储hidden信息的字段：

  - 利用jni去修改。

#### 7. 如何进行单元测试，如何保证App稳定 ？

要测试Android应用程序，通常会创建以下类型自动单元测试

- **本地测试**：只在本地机器JVM上运行，以最小化执行时间，这种单元测试不依赖于Android框架，或者即使有依赖，也很方便使用模拟框架来模拟依赖，以达到隔离Android依赖的目的，模拟框架如Google推荐的Mockito；
- [**Android官网-建立本地单元测试**](https://developer.android.com/training/testing/unit-testing/local-unit-tests.html)
- **检测测试**：真机或模拟器上运行的单元测试，由于需要跑到设备上，比较慢，这些测试可以访问仪器（Android系统）信息，比如被测应用程序的上下文，一般地，依赖不太方便通过模拟框架模拟时采用这种方式；
- [**Android官网-建立仪表单元测试**](https://developer.android.com/training/testing/unit-testing/instrumented-unit-tests.html)

注意：单元测试不适合测试复杂的UI交互事件

> 推荐文章：[Android 单元测试只看这一篇就够了](https://juejin.im/post/6844903645843030030)

App的稳定主要决定于整体的系统架构设计，同时也不可忽略代码编程的细节规范，正所谓“千里之堤，溃于蚁穴”，一旦考虑不周，看似无关紧要的代码片段可能会带来整体软件系统的崩溃，所以上线之前除了自己**本地化测试**之外还需要进行**Monkey压力测试**

少部分面试官可能会延伸，如Gradle自动化测试、机型适配测试等。

**首先，Android测试主要分为三个方面**：

- 单元测试（Junit4、Mockito、PowerMockito、Robolectric）
- UI测试（Espresso、UI Automator）
- 压力测试（Monkey）

WanAndroid项目和XXX项目中使用用到了单元测试和部分自动化UI测试，其中单元测试使用的是Junit4+Mockito+PowerMockito+Robolectric。下面我分别简单介绍下这些测试框架：

1、Junit4：

使用@Test注解指定一个方法为一个测试方法，除此之外，还有如下常用注解@BeforeClass->@Before->@Test->@After->@AfterClass以及@Ignore。

Junit4的主要测试方法就是断言，即assertEquals()方法。然后，你可以通过实现TestRule接口的方式重写apply()方法去自定义Junit Rule，这样就可以在执行测试方法的前后做一些通用的初始化或释放资源等工作，接着在想要的测试类中使用@Rule注解声明使用JsonChaoRule即可。（注意被@Rule注解的变量必须是final的。最后，我们直接运行对应的单元测试方法或类，如果你想要一键运行项目中所有的单元测试类，直接点击运行Gradle Projects下的app/Tasks/verification/test即可，它会在module下的build/reports/tests/下生成对应的index.html报告。

Junit4它的优点是速度快，支持代码覆盖率如jacoco等代码质量的检测工具。缺点就是无法单独对Android UI，一些类进行操作，与原生Java有一些差异。

2、Mockito：

可以使用mock()方法模拟各种各样的对象，以替代真正的对象做出希望的响应。除此之外，它还有很多验证方法调用的方式如Mockit.when(调用方法).thenReturn(验证的返回值)、verfiy(模拟对象).验证方法等等。

这里有一点要补充下：简单的测试会使整体的代码更简洁，更可读、更可维护。如果你不能把测试写的很简单，那么请在测试时重构你的代码。

最后，对于Mockito来说，它的优点是有各种各样的方式去验证"模仿对象"的互动或验证发生的某些行为。而它的缺点就是不支持mock匿名类、final类、static方法private方法。

3、PowerMockito：

因此，为了解决Mockito的缺陷，PoweMockito出现了，它扩展了Mockito，支持mock匿名类、final类、static方法、private方法。只要使用它提供的api如PowerMockito.mockStatic()去mock含静态方法或字段的类，PowerMockito.suppress(PowerMockito.method(类.class， 方法名)即可。

4、Robolectric

前面3种我们说的都是Java相关的单元测试方法，如果想在Java单元测试里面进行Android单元测试，还得使用Robolectric，它提供了一套能运行在JVM的Android代码。它提供了一系列类似ShadowToast.getLatestToast()、ShadowApplication.getInstance()这种方式来获取Android平台对应的对象。可以看到它的优点就是支持大部分Android平台依赖类的底层引用与模拟。缺点就是在异步测试的情况下有些问题，这是可以结合Mockito来将异步转为同步即可解决。

最后，自动化UI测试项目中我使用的是Expresso，它提供了一系列类似onView().check().perform()的方式来实现点击、滑动、检测页面显示等自动化的UI测试效果，这里在我的WanAndroid项目下的BasePageTest基类里面封装了一系列通用的方法，有兴趣可以去看看。



#### 8. Android中如何查看一个对象的回收情况 ？

首先要了解Java四种引用类型的场景和使用（强引用、软引用、弱引用、虛引用）

举个场景例子：**SoftReference**对象是用来保存软引用的，但它同时也是一个Java对象，所以当软引用对象被回收之后，虽然这个**SoftReference**对象的get方法返回null，但**SoftReference**对象本身并不是null，而此时这个**SoftReference**对象已经不再具有存在的价值，需要一个适当的清除机制，避免大量**SoftReference**对象带来的**内存泄露**

因此，Java提供**ReferenceQueue**来处理引用对象的回收情况。当**SoftReference**所引用的对象被GC后，**JVM**会先将**softReference**对象添加到**ReferenceQueue**这个队列中。当我们调用**ReferenceQueue的poll()方法**，如果这个队列中不是空队列，那么将返回并移除前面添加的那个Reference对象。

![img](https://user-gold-cdn.xitu.io/2019/3/27/169bd0c28741e8e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


推荐文章：[Java中的四种引用类型：强引用、软引用、弱引用和虚引用](https://segmentfault.com/a/1190000015282652#articleHeader3)

#### 9. FC(Force Close)什么时候会出现？

Error、OOM，StackOverFlowError、Runtime，比如说空指针异常

解决的办法：

- 注意内存的使用和管理
- 使用Thread.UncaughtExceptionHandler接口

#### 10. [Java多线程引发的性能问题，怎么解决](https://blog.csdn.net/luofenghan/article/details/78596950)？

#### 11. TraceView的实现原理，分析数据误差来源。

#### 12. 是否使用过SysTrace，原理的了解？

#### 13. mmap + native 日志优化？

传统日志打印有两个性能问题，一个是反复操作文件描述符表，一个是反复进入内核态。所以需要使用mmap的方式去直接读写内存。

#### 14. 设计一个方案，apk已经发出去了，java代码是最新，但是分包下发的so文件是旧版本，如何做一个兼容方案，保证兼容可用？

#### 15. 设计一个组件，统计Activity的前台时长，Fragment的前台时长。

#### 16. 设计一个埋点库。需要哪些模块。

#### 17. 友盟bug统计，混淆后怎么定位bug？没接入热修复的APP中，上线后遇到bug怎么解决？

#### 18. T1、T2、T3三个线程，如何保证它们顺序执行？也就是异步转同步的方式。

```
①Thread#join
②等待多线程完成的CountDownLatch
③FutureTask
④Executors#newSingleThreadExecutor
⑤wait/notify
```

#### 19. 动态权限适配方案，权限组的概念

#### 20. Runtime permission，如何把一个预置的app默认给它权限？不要授权。

#### 21. 如何实现进程安全写文件？
