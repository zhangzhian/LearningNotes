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
- `singleTop`：栈顶复用，如果`Activity`实例已经存在栈顶，那么就不会在活动栈中创建新的实例，并回调onNewIntent方法。比较常见的场景就是给通知跳转的`Activity`设置，因为你肯定不想前台`Activity`已经是该`Activity`的情况下，点击通知，又给你再创建一个同样的`Activity`。
- `singleTask`：栈内复用，如果`Activity`实例在当前栈中已经存在，就会将当前`Activity`实例上面的其他`Activity`实例都移除栈，并回调onNewIntent方法。常见于跳转到主界面。
- `singleInstance`：单实例模式，创建一个新的任务栈，这个活动实例独自处在这个活动栈中，同样被重复调用的时候会调用并回调onNewIntent方法。

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

#### 8. Activity依次A→B→C→B，其中B启动模式为singleTask，AC都为standard，生命周期分别怎么调用？如果B启动模式为singleInstance又会怎么调用？B启动模式为singleInstance不变，A→B→C的时候点击两次返回，生命周期如何调用。

**旧的Activity先onPause，新的再启动。**

1)A→B→C→B,B启动模式为`singleTask`

- 启动A的过程，生命周期调用是 (A)onCreate→(A)onStart→(A)onResume
- 再启动B的过程，生命周期调用是 (A)onPause→(B)onCreate→(B)onStart→(B)onResume→(A)onStop
- B→C的过程同上
- C→B的过程，由于B启动模式为singleTask，所以B会调用onNewIntent，并且将B之上的实例移除，也就是C会被移出栈。所以生命周期调用是 (C)onPause→(B)onNewIntent→(B)onRestart→(B)onStart→(B)onResume→(C)onStop→(C)onDestory

2)A→B→C→B,B启动模式为`singleInstance`

- 如果B为singleInstance，那么C→B的过程，C就不会被移除，因为B和C不在一个任务栈里面。所以生命周期调用是 (C)onPause→(B)onNewIntent→(B)onRestart→(B)onStart→(B)onResume→(C)onStop

3)A→B→C,B启动模式为`singleInstance`,点击两次返回键

- 如果B为singleInstance，A→B→C的过程，生命周期还是同前面一样正常调用。但是点击返回的时候，由于AC同任务栈，所以C点击返回，会回到A，再点击返回才回到B。所以生命周期是：(C)onPause→(A)onRestart→(A)onStart→(A)onResume→(C)onStop→(C)onDestory。
- 再次点击返回，就会回到B，所以生命周期是：**(A)onPause→(B)onRestart→(B)onStart→(B)onResume→(A)onStop→(A)onDestory**。

#### 9. 屏幕旋转时Activity的生命周期，如何防止Activity重建。

**onSaveInstanceState在onstop前，个onPause没有既定的关系**

- 切换屏幕的生命周期是：onConfigurationChanged->onPause->onSaveInstanceState->onStop->onDestroy->onCreate->onStart->onRestoreInstanceState->onResume
- 如果需要防止旋转时候，`Activity`重新创建的话需要做如下配置：
   在`targetSdkVersion`的值小于或等于12时，配置 android:configChanges="orientation"，
   在`targetSdkVersion`的值大于12时，配置 android:configChanges="orientation|screenSize"



## 二、Service

#### 1. Activity怎么启动Service，Activity与Service交互，Service与Thread的区别

#### 2. Service 一定没界面吗，Activity 一定有界面吗？

- Activity 不是一定有界面。比如一个跳转逻辑控制类（机票的支付中间类）、透明页

- [Service 也不是一定没界面](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F5dbe43cf518825244b38a6c8)。Service 并不依赖于用户可视的 UI 界面，但这也不是绝对的，如前台 Service 就是与 Notification 界面结合使用的；Service 中也可以弹 Toast；

- [Service中执行 LayoutInflate 是合法的](https://www.jianshu.com/p/94e0f9ab3f1d)，但是会使用系统默认的主题样式，如果你自定义了某些样式可能不会被使用。所以从理论上看也是可以有界面的

## 三、BroadcaseReceiver



## 四、ContentProvider



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

#### 2. ViewPager切换Fragment遇到过什么问题吗?什么最耗时？

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

- Activity 与 Fragment通信

Activity有Fragment的实例，所以可以执行Fragment的方法，或者传入一个接口。同样，Fragment可以通过`getActivity()`获取Activity的实例，也是可以执行方法。

- Fragment 与 Fragment之间通信

1）直接获取另一个Fragmetn的实例

```java
getActivity().getSupportFragmentManager().findFragmentByTag("mainFragment");
```

2）接口回调

一个Fragment里面去实现接口，另一个Fragment把接口实例传进去。

3）Eventbus等框架。



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

#### 3. 代码实现一个长按事件

#### 4. 手势操作ActionCancel后怎么取消

https://www.jianshu.com/p/3581fcf302fd

#### 5. setOnTouchListener,onClickeListener和onTouchEvent的关系

#### 6. 横向 ScrollView、纵向 ListView 怎么处理滑动冲突?

> [Android 实践之 ScrollView 中滑动冲突处理](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fxiaohanluo%2Farticle%2Fdetails%2F52130923)

**解决滑动冲突的办法。**

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



#### 2. 如何自定义实现一个FlexLayout



#### 3. 自定义实现一个九宫格如何实现



#### 4. onMeasure,onLayout,onDraw关系



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



## 十四、其他

#### 1. 子线程是否可以 context.startActivity() ？例如ApplicationContext, 会不会有什么问题？

是可以的。

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



#### 6. Bundle是什么数据结构?利用什么传递数据



#### 7. 一个wrap_content的ImageView，加载远程图片，传什么参数裁剪比较好?



#### 8. 平常抓包用什么工具？



#### 9.实现一个下载功能的接口



#### 10. attachToWindow什么时候调用？



#### 11. Activity内LinearLayout红色wrap_content,包含View绿色wrap_content,求界面颜色

#### 12.有用DSL,anko写过布局吗？

#### 13. SharedPreference原理？读取xml是在哪个线程?



#### 14. 怎么优化xml inflate的时间，涉及IO与反射。了解compose吗？