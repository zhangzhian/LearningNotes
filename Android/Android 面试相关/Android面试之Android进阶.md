# Android进阶

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 一、Android FrameWork
### 1. Binder

#### (1) Binder的介绍？与其他IPC方式的优缺点？

Binder是Android中特有的IPC方式，引用《Android开发艺术探索》中的话(略有改动)：

> 从IPC角度来说，Binder是Android中的一种跨进程通信方式；Binder还可以理解为虚拟的物理设备，它的设备驱动是/dev/binder；从`Android Framework`来讲，Binder是`Service Manager`连接各种`Manager`和对应的`ManagerService`的桥梁。从面向对象和CS模型来讲，`Client`通过Binder和远程的`Server`进行通讯。

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



### 4. Window

#### (1) Activity启动过程跟Window的关系？

> [《简析Window、Activity、DecorView以及ViewRoot之间的错综关系》](https://www.jianshu.com/p/8766babc40e0)

#### (2) Activity、Window、ViewRoot和DecorView之间的关系？

> [《总结UI原理和高级的UI优化方式》](https://juejin.im/post/6844903974294781965)

### 5. PMS

#### (1) Apk的安装过程？

*建议阅读：*

> [《Android Apk安装过程分析》](https://www.jianshu.com/p/953475cea991)

### 6. Contex

#### (1) 关于Context的理解？

> [《Android Context 上下文 你必须知道的一切》](https://blog.csdn.net/lmj623565791/article/details/40481055)

## 二、Android权限处理

## 三、多线程断点续传
#### (1) 多线程断点续传？

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

### 2. OkHttp

#### (1) OkHttp责任链模式

#### (2) interceptors和networkInterceptors的区别？

建议看一遍源码，过程并不复杂。

### 3. Retrofit

#### (1) 设计模式和封层解耦的理念

#### (2) 动态代理

建议看一遍源码，过程并不复杂。

### 4. Glide

#### (1) Glide和其他图片加载框架的比较？

#### (2) 如何设计一个图片加载框架？

#### (3) Glide缓存实现机制？

#### (4) Glide如何处理生命周期？

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

## 五、性能优化
[《Android 性能优化最佳实践》](https://juejin.im/post/6844903641032163336)

### 1. 布局优化

### 2. 绘制优化
### 3. 内容泄漏
### 4. 响应速度优化
### 5. 启动优化
### 6. Bitmap优化
### 7. 线程优化
### 8. RecycleView优化

## 六、插件化
## 七、组件化

## 八、架构

#### 1. MVC、MVP和MVVM是什么？

图片已有，不再给出

- MVC：Model-View-Controller，是一种分层解偶的框架，Model层提供本地数据和网络请求，View层处理视图，Controller处理逻辑，存在问题是Controller层和View层的划分不明显，Model层和View层的存在耦合。
- MVP：Model-View-Presenter，是对MVC的升级，Model层和View层与MVC的意思一致，但Model层和View层不再存在耦合，而是通过Presenter层这个桥梁进行交流。
- MVVM：Model-View-ViewModel，不同于上面的两个框架，ViewModel持有数据状态，当数据状态改变的时候，会自动通知View层进行更新。

#### 2. MVC和MVP的区别是什么？

MVP是MVC的进一步解耦，简单来讲，在MVC中，View层既可以和Controller层交互，又可以和Model层交互；而在MVP中，View层只能和Presenter层交互，Model层也只能和Presenter层交互，减少了View层和Model层的耦合，更容易定位错误的来源。

#### 3. MVVM和MVP的最大区别在哪？

MVP中的每个方法都需要你去主动调用，它其实是被动的，而MVVM中有数据驱动这个概念，当你的持有的数据状态发生变更的时候，你的View你可以监听到这个变化，从而主动去更新，这其实是主动的。

#### 4. ViewModel如何知道View层的生命周期？

事实上，如果你仅仅使用ViewModel，它是感知不了生命周期，它需要结合LiveData去感知生命周期，如果仅仅使用DataBinding去实现MVVM，它对数据源使用了弱引用，所以一定程度上可以避免内存泄漏的发生。









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

