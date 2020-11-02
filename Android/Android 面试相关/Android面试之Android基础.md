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

#### 4. 如何求当前Activity View的深度



#### 5. 多进程怎么实现？如果启动一个多进程APP，会有几个进程运行？

#### 6. Activity与AppCompactActivity区别，Activity会打包到包里面去吗？

#### 7. Activity 按 back 键退出，与强杀进程退出有啥区别？

（1）应用被强杀

- 整个App进程都被杀掉了，所有变量全都被清空，包括Application实例，更别提静态变量；
- 虽然变量被清空了，但 Activity 栈没有被清空，也就是说 A -> B -> C 这个栈还保存了，只是ABC 这几个 Activity 实例没有了。所以回到 App 时，显示的还是 C 页面。另外当 Activity 被强杀时，系统会调用 onSaveInstance 去让你保存一些变量；
- 当应用回到前台时，如果C页面中有静态变量或有些Application的全局变量，就NullPointer了；
- C页面不会正常走完生命周期onStop & onDestory

（2）按 Back 键回退

- 应用进程不会被杀掉；Activity 栈由 A -> B -> C 变成 A -> B；
- C页面会正常走完生命周期onStop & onDestory





## 二、Service

#### 1. Activity怎么启动Service，Activity与Service交互，Service与Thread的区别

#### 2. Service 一定没界面吗，Activity 一定有界面吗？

- Activity 不是一定有界面。比如一个跳转逻辑控制类（机票的支付中间类）、透明页

- [Service 也不是一定没界面](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5dbe43cf518825244b38a6c8)。Service 并不依赖于用户可视的 UI 界面，但这也不是绝对的，如前台 Service 就是与 Notification 界面结合使用的；Service 中也可以弹 Toast；

- [Service中执行 LayoutInflate 是合法的](https://www.jianshu.com/p/94e0f9ab3f1d)，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。所以从理论上看也是可以有界面的

## 三、BroadcaseReceiver



## 四、ContentProvider



## 五、Fragment

#### 1. Fragment hide show生命周期

#### 2. Fragment replace生命周期变化

#### 3. ViewPager切换Fragment什么最耗时？



## 六、屏幕适配

参考：Android屏幕适配方案详解

#### 1. 你们 Android 开发的时候，对于 UI 稿的 px 是如何适配的？

今日头条 AndroidAutoSize和smallestWidth方案

####  2. 平时如何有使用屏幕适配吗？原理是什么呢？

平时的屏幕适配一般采用的头条的屏幕适配方案。简单来说，以屏幕的一边作为适配，通常是宽。

原理：设备像素`px`和设备独立像素`dp`之间的关系是

```
px = dp * density
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

#### 4. 同步屏障

#### 5. 如何修复匿名内部类 Handler 造成的内存泄露？

#### 6. Handler 机制中是怎么保证每个线程的 Looper 是唯一的？



#### Handler机制整体流程；

#### postDelay()的具体实现；

#### post()与sendMessage()区别；

#### 使用Handler需要注意什么问题，怎么解决的?



#### 简单描述下Handler,Handler是怎么切换线程的,Handler同步屏障



#### Handler机制了解吗？一个线程有几个Looper？为什么？



## 九、View事件分发机制

#### 1. 自定义View,事件分发机制讲一讲

#### 2.dispatchTouchEvent,onInterceptEvent,onTouchEvent顺序，关系

#### 3. 代码实现一个长按事件

#### 4. 手势操作ActionCancel后怎么取消

https://www.jianshu.com/p/3581fcf302fd

#### 5. setOnTouchListener,onClickeListener和onTouchEvent的关系

#### 6. 横向 ScrollView、纵向 ListView 怎么处理滑动手势冲突

> [Android 实践之 ScrollView 中滑动冲突处理](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fxiaohanluo%2Farticle%2Fdetails%2F52130923)

## 十、View绘制

#### 1. 怎么计算一个View在屏幕可见部分的百分比？



#### 2. 如何自定义实现一个FlexLayout



#### 3. 自定义实现一个九宫格如何实现



#### 4. onMeasure,onLayout,onDraw关系



#### 5.onCreate,onResume,onStart里面，什么地方可以获得宽高

#### 6.为什么view.post可以获得宽高，有看过view.post的源码吗？

#### 7. 自定义LinearLayout，怎么测量子View宽高

#### 8. setFactory和setFactory2有什么区别？



## 十一、Drawbale和动画

#### 1. 介绍一下android动画



#### 2. Drawable与View有什么区别,Drawable有哪些子类



#### 3. 属性动画更新时会回调onDraw吗？



#### 4. 两个getDrawable取得的对象，有什么区别？



#### 5. 补间动画与属性动画的区别，哪个效率更高？

#### 6. 动画里面用到了什么设计模式



#### 7. 动画连续调用的原理是什么？



#### 8. 自定义圆角图片







## 十二、AsyncTask

#### 1. AsyncTask内存泄露





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

## 十四、RecyclerView

#### 1. RecyclerView是怎么处理内部ViewClick冲突的



#### 2. 讲一下RecyclerView的缓存机制,滑动10个，再滑回去，会有几个执行onBindView



#### 3. 如何实现RecyclerView的局部更新，用过payload吗,notifyItemChange方法中的参数？



#### 4. RecyclerView的缓存结构是怎样的？缓存的是什么？cachedView会执行onBindView吗?



#### 5. RecyclerView嵌套RecyclerView，NestScrollView嵌套ScrollView滑动冲突



#### 6. RecyclerView 缓存结构，RecyclerView预取，RecyclerView局部刷新



## 十五、ViewPager

#### 1. ViewPager2原理

#### 2. ViewPager中嵌套ViewPager怎么处理滑动冲突

#### 3. viewpager切换掉帧有什么处理经验？

#### 4. 说说事件分发机制，怎么写一个不能滑动的ViewPager



## 十四、其他

#### 1. 子线程是否可以 context.startActivity() ？例如ApplicationContext, 会不会有什么问题？

是可以的。

#### 2. ConstraintLayout实现三等分,ConstraintLayout动画



#### 3. CoordinatorLayout自定义behavior,可以拦截什么？



#### 4. SharedParence可以跨进程通信吗？如何改造成可以跨进程通信的.commit和apply的区别.



#### 6. Bundle是什么数据结构?利用什么传递数据



#### 7. 一个wrap_content的ImageView，加载远程图片，传什么参数裁剪比较好?



#### 8. 平常抓包用什么工具？



#### 9.实现一个下载功能的接口



#### 10. attachToWindow什么时候调用？

#### 

#### 11. Activity内LinearLayout红色wrap_content,包含View绿色wrap_content,求界面颜色

#### 12.有用DSL,anko写过布局吗？

#### 13. SharedPreference原理？读取xml是在哪个线程?



#### 14. 怎么优化xml inflate的时间，涉及IO与反射。了解compose吗？