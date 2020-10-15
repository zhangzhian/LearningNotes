# Android基础

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 一、Activity

### 页面跳转

#### 1. Activity A 跳转Activity B，Activity B再按back键回退，两个过程各自的生命周期

(1) ActivityA跳转ActivityB的过程中,各自生命周期的执行顺序?

执行顺序如下： A.onPause －> B.onCreate －> B.onStart－> B.onResume－> A.onStop

(2) ActivityB 按back键呢?

按下back键后： B.onPause－>A.onRestart－>A.onStart－>A.onResume－>B.onStop－>B.onDestory

(3) ActivityB是个窗口Activity的情况下，(1)、(2)的结论呢？

ActivityA跳转到ActivityB时，ActivityA失去焦点部分可见，故不会调用onStop，此时生命周期顺序： A.onPause －> B.onCreate －> B.onStart－> B.onResume
按下Back键后：B.onPause－>A.onResume－>B.onStop－>B.onDestory

(4) 切换横竖屏时，onCreate会调用吗？几次？

程序在运行时，一些设备的配置可能会改变，如：横竖屏的切换、键盘的可用性或语言的切换等，此时Activity会重新启动。

其中的过程是：在销毁之前会先调用onSaveInstancestate()去保存应用中的一些数据，然后调用 onDestory()，最后才会去调用onCreate()或者onRestoreInstanceState方法重新启动Activiy。

如果自己没有配置android:ConfigChanges，在切换屏幕时候会重新调用各个生命周期，切横屏时会执行一次onCreate，切竖屏时在2.3前会执行两次onCreate，在2.3版本及以后都执行一次。

如果设置 android:configChanges="orientation|keyboardHidden|screenSize">，此时Activity的生命周期不会重走一遍，Activity不会重建，只会回调onConfigurationChanged方法。

## 二、Service



## 三、BroadcaseReceiver



## 四、ContentProvider



## 五、Fragment



## 六、屏幕适配

参考：Android屏幕适配方案详解

#### 1. 你们 Android 开发的时候，对于 UI 稿的 px 是如何适配的？

今日头条 AndroidAutoSize和smallestWidth方案

## 七、Lrucache



## 八、Android消息机制

### Handler机制

Handler机制整体流程；

Looper.loop()为什么不会阻塞主线程；

IdHandler(闲时机制）；

postDelay()的具体实现；

post()与sendMessage()区别；

使用Handler需要注意什么问题，怎么解决的?

## 九、View事件分发机制



## 十、View绘制

#### 1. 怎么计算一个View在屏幕可见部分的百分比？



## 十一、动画



## 十二、AsyncTask



## 十三、Bitmap

#### 1. 下载一张很大的图，如何保证不 oom？

 [Android性能优化（五）之细说Bitmap](https://www.jianshu.com/p/e49ec7d053b3)

## 十四、其他

#### 1. 子线程是否可以 context.startActivity() ？例如ApplicationContext, 会不会有什么问题？

是可以的。