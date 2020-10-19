# Android基础

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 一、Activity

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

#### 2. Activity的四大启动模式，以及应用场景？

`Activity`的四大启动模式：

- `standard`：标准模式，每次都会在活动栈中生成一个新的`Activity`实例。通常我们使用的活动都是标准模式。
- `singleTop`：栈顶复用，如果`Activity`实例已经存在栈顶，那么就不会在活动栈中创建新的实例。比较常见的场景就是给通知跳转的`Activity`设置，因为你肯定不想前台`Activity`已经是该`Activity`的情况下，点击通知，又给你再创建一个同样的`Activity`。
- `singleTask`：栈内复用，如果`Activity`实例在当前栈中已经存在，就会将当前`Activity`实例上面的其他`Activity`实例都移除栈。常见于跳转到主界面。
- `singleInstance`：单实例模式，创建一个新的任务栈，这个活动实例独自处在这个活动栈中。

#### 3. Activity中onStart和onResume的区别？onPause和onStop的区别？

首先，Activity有三类：

前台Activity：活跃的Activity，正在和用户交互的Activity。
可见但非前台的Activity：常见于栈顶的Activity背景透明，处在其下面的Activity就是可见但是不可和用户交互。
后台Activity：已经被暂停的Activity，比如已经执行了onStop方法。

所以，onStart和onStop通常指的是当前活动是否位于前台这个角度，而onResume和onPause从是否可见这个角度来讲的。

#### 



## 二、Service



## 三、BroadcaseReceiver



## 四、ContentProvider



## 五、Fragment



## 六、屏幕适配

参考：Android屏幕适配方案详解

#### 1. 你们 Android 开发的时候，对于 UI 稿的 px 是如何适配的？

今日头条 AndroidAutoSize和smallestWidth方案

####  2. 平时如何有使用屏幕适配吗？原理是什么呢？

平时的屏幕适配一般采用的头条的屏幕适配方案。简单来说，以屏幕的一边作为适配，通常是宽。

原理：设备像素`px`和设备独立像素`dp`之间的关系是

```
px = dp * density
复制代码
```

假设UI给的设计图屏幕宽度基于360dp，那么设备宽的像素点已知，即px，dp也已知，360dp，所以`density = px / dp`，之后根据这个修改系统中跟`density`相关的点即可。



## 七、Lrucache



## 八、Android消息机制

#### 1. Android消息机制介绍？

Android消息机制中的四大概念：

- `ThreadLocal`：当前线程存储的数据仅能从当前线程取出。
- `MessageQueue`：具有时间优先级的消息队列。
- `Looper`：轮询消息队列，看是否有新的消息到来。
- `Handler`：具体处理逻辑的地方。

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

执行场景：

- `MessageQueue`没有消息，队列为空的时候。
- `MessageQueue`属于延迟消息，当前没有消息执行的时候。

会不会发生死循环： 答案是否定的，`MessageQueue`使用计数的方法保证一次调用`MessageQueue#next`方法只会使用一次的`IdleHandler`集合。

#### Handler机制整体流程；

#### postDelay()的具体实现；

#### post()与sendMessage()区别；

#### 使用Handler需要注意什么问题，怎么解决的?

## 九、View事件分发机制



## 十、View绘制

#### 1. 怎么计算一个View在屏幕可见部分的百分比？



## 十一、动画



## 十二、AsyncTask



## 十三、Bitmap

#### 1. 下载一张很大的图，如何保证不 oom？

 [Android性能优化（五）之细说Bitmap](https://www.jianshu.com/p/e49ec7d053b3)

#### 2. Bitmap的内存计算方式？

在已知图片的长和宽的像素的情况下，影响内存大小的因素会有**资源文件位置和像素点大小**。

**像素点大小**： 常见的像素点有：

- ARGB_8888：4个字节
- ARGB_4444、ARGB_565：2个字节

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

## 十四、其他

#### 1. 子线程是否可以 context.startActivity() ？例如ApplicationContext, 会不会有什么问题？

是可以的。