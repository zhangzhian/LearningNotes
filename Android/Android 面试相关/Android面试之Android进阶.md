# Android进阶

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## Android FrameWork
### 一、Binder
### 二、进程
### 三、四大组件启动相关
### 四、Window
### 五、PMS
### 六、Contex

## Android权限处理

## 多线程断点续传
#### Http Header Range
####  RandomAccessFile

## 第三方库
### 一、RxJava2
### 二、OkHttp3
### 三、Retrofit2
### 四、Glide
### 五、Android Jepack
​		LiveData
​		Lifecycle
​		ViewModel
​		Data Binding
​		Paging

## 性能优化
### 一、布局优化
### 二、绘制优化
### 三、内容泄漏
### 四、响应速度优化
### 五、启动优化
### 六、Bitmap优化
### 七、线程优化
### 八、RecycleView优化

## 插件化
## 组件化





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

