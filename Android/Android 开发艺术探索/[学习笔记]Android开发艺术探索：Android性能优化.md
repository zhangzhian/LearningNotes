# [学习笔记]Android开发艺术探索：Android性能优化

Android设备作为一种移动设备，不管是内存还是CPU的性能都受到了一定的限制，也意味着Android程序不可能无限制的使用内存和CPU资源，过多的使用内存容易导致OOM，过多的使用CPU资源容易导致手机变得卡顿甚至无响应（ANR）。这也对开发人员提出了更高的要求。
 本章主要介绍一些有效的性能优化方法。主要包括布局优化、绘制优化、内存泄漏优化、响应速度优化、ListView优化、Bitmap优化、线程优化等；同时还介绍了ANR日志的分析方法。

## Android的性能优化方法

Google官方的Android性能优化典范专题短视频课程是学习Android性能优化极佳的课程，目前已更新到第五季；
 放一个Google官方维护的国内方便访问的链接地址 [youku地址](http://v.youku.com/v_show/id_XMTQ4MDU3Nzc3Mg==.html?f=26771407)

### 布局优化

布局优化的思想就是尽量减少布局文件的层级，这样绘制界面时工作量就少了，那么程序的性能自然就高了。
 删除无用的控件和层级，其次就是有选择的使用性能较低的ViewGroup，如果布局中既可以使用Linearlayout也可以使用RelativeLayout，那就是用LinearLayout，因为RelativeLayout功能比较复杂，它的布局过程需要花费更多的CPU时间。有时候通过LinearLayou无法实现产品效果，需要通过嵌套来完成，这种情况还是推荐使用RelativeLayout，因为ViewGroup的嵌套相当于增加了布局的层级，同样降低程序性能。

另一种手段是采用<include>标签、<merge>标签和ViewStub。

<include>标签
<include>标签用于布局重用，可以将一个指定的布局文件加载到当前布局文件中。<include>只支持android:layout_开头的属性，当然android:id这个属性是个特例；如果指定了android:layout_这种属性，那么要求android:layout_width和android:layout_height必须存在，否则android:layout_属性无法生效。如果<include>指定了id属性，同时被包含的布局文件的根元素也指定了id属性，会以<include>指定的这个id属性为准。

<merge>标签
<merge>标签一般和<include>标签一起使用从而减少布局的层级。如果当前布局是一个竖直方向的LinearLayout，这个时候被包含的布局文件也采用竖直的LinearLayout，那么显然被包含的布局文件中的这个LinearLayout是多余的，通过<merge>标签就可以去掉多余的那一层LinearLayout。

ViewStub
ViewStub意义在于按需加载所需的布局文件，因为实际开发中，有很多布局文件在正常情况下是不会现实的，比如网络异常的界面，这个时候就没必要在整个界面初始化的时候将其加载进来，在需要使用的时候再加载会更好。在需要加载ViewStub布局时：

```cpp
((ViewStub)findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
//或者
View importPanel = ((ViewStub)findViewById(R.id.stub_import)).inflate();
```

当ViewStub通过setVisibility或者inflate方法加载后，ViewStub就会被它内部的布局替换掉，ViewStub也就不再是整个布局结构的一部分了。

### 绘制优化

View的onDraw方法要避免执行大量的操作；

1. onDraw中不要创建大量的局部对象，因为onDraw方法会被频繁调用，这样就会在一瞬间产生大量的临时对象，不仅会占用过多内存还会导致系统频繁GC，降低程序执行效率。
2. onDraw也不要做耗时的任务，也不能执行成千上万的循环操作，尽管每次循环都很轻量级，但大量循环依然十分抢占CPU的时间片，这会造成View的绘制过程不流畅。根据Google官方给出的标准，View绘制保持在60fps是最佳的，这也就要求每帧的绘制时间不超过16ms(1000/60)；所以要尽量降低onDraw方法的复杂度。

### 内存泄露优化

内存泄露是最容易犯的错误之一，内存泄露优化主要分两个方面；
 一方面是开发过程中避免写出有内存泄露的代码，另一方面是通过一些分析工具如LeakCanary或MAT来找出潜在的内存泄露继而解决。

1. 静态变量导致的内存泄露
    比如Activity内，一静态Conext引用了当前Activity，所以当前Activity无法释放。或者一静态变量，内部持有了当前Activity，Activity在需要释放的时候依然无法释放。
2. 单例模式导致的内存泄露
    比如单例模式持有了Activity，而且也没用解注册的操作。因为单例模式的生命周期和Application保存一致，生命周期比Activity要长，这样一来就导致Activity对象无法及时被释放。
3. 属性动画导致的内存泄露
    属性动画中有一类无限循环的动画，如果在Activity播放了此类动画并且没有在onDestroy中去停止动画，那么动画会一直播放下去，并且这个时候Activity的View会被动画持有，而View又持有了Activity，最终导致Activity无法释放。解决办法是在Activity的onDrstroy中调用animator.cancel()来停止动画。

### 响应速度优化和ANR日志分析

响应速度优化的核心思想就是避免在主线程中去做耗时操作，将耗时操作放在其他线程当中去执行。Activity如果5秒无法响应屏幕触摸事件或者键盘输入事件就会触发ANR，而BroadcastReceiver如果10秒还未执行完操作也会出现ANR。
 当一个进程发生ANR以后系统会在/data/anr的目录下创建一个文件traces.txt，通过分析该文件就能定位出ANR的原因。

### ListView优化和Bitmap优化

ListView/GridView优化：采用ViewHolder避免在getView中执行耗时操作；其次通过列表的滑动状态来控制任务的执行频率，比如快速滑动时不是和开启大量异步任务；最后可以尝试开启硬件加速使得ListView的滑动更加流畅。
 Bitmap优化：主要是想是根据需要对图片进行采样显示，详细请参考12章。

### 线程优化

线程优化的思想是采用线程池，避免程序存在大量的Thread。详细参考第11章的内容。

### 一些性能优化的小建议

- 避免创建过多的对象，尤其在循环、onDraw这类方法中，谨慎创建对象；
- 不要过多的使用枚举，枚举占用的内存空间比整形大。
- 常量使用static final来修饰；
- 使用一些Android特有的数据结构，比如SparseArray和Pair等，他们都具有更好的性能；
- 适当的使用软引用和弱引用；
- 采用内存缓存和磁盘缓存；
- 尽量采用静态内部类，这样可以避免非静态内部类隐式持有外部类所导致的内存泄露问题。

## 提高程序的可维护性

1. **提高可读性**：命名规范；代码之间排版需留出合理的空白来区分不同的代码块；针对非常关键的代码添加注释。
2. **代码的层级性**：不要把一段业务逻辑放在一个方法或者一个类中全部实现，要把它分成几个子逻辑，然后每个子逻辑做自己的事情，这样即显得代码层级分明，这样利于提高程序的可扩展性。
3. 恰当的使用**设计模式**可以提高代码的可维护性和可扩展性，Android程序容易遇到性能瓶颈，要控制设计的度，不能太牵强，避免过度设计。