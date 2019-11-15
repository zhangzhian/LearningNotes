#[学习笔记] Android群英传：Android性能优化

---
主要内容

- 布局优化
- 内存优化
- 使用各种工具进行分析，优化


##一.布局优化

###1.Android UI渲染机制
人眼所看到的流畅画面，需要画面的帧数达到40帧每秒到60帧每秒，最佳的ftp在60左右，这也是评价一个显卡性能的一个高低标准之一，在Android中，系统通过VSYNC信号出发对UI的渲染、重绘，其间隔时间是16ms。这个16ms其实就是1000ms中显示60帧画面的单位时间。系统每次渲染都保持在16ms之内，则UI将十分的流畅，但这也是需要将所有的逻辑都保证在16ms里，如果16ms不能完成绘制，那么就会造成丢帧的现象，即当前该重绘的帧被未完成的逻辑阻塞

Android系统提供了检测UI渲染时间的工具，打开“开发者选项”，选择“Profile GPU Rendering”（我的手机是“GPU呈现模式分析”），选中“On screen as bars”（我的为“在屏幕上显示为条形图”）。每一条柱状线都包括三部分，蓝色代表测量绘制Display List的时间，红色代表OpenGL渲染Display List所需要的时间，黄色代表CPU等待GPU处理的时间，中间绿色横线代表VSYNC时间16ms，需要尽量将所有条形图都控制在这条绿线之下。

![这里写图片描述](http://img.blog.csdn.net/20160430134922086)


###2.避免Overdraw
Overdraw过渡绘制会浪费很多CPU、GPU资源，例如系统默认会绘制Activity的背景，而如果再给布局绘制了重叠的背景，那么默认Activity的背景就属于无效的过渡绘制。Android系统在开发者选项中提供了这样一个检测工具--“Enable GPU Overdraw”。借助它可以判断Overdraw的次数。尽量增大蓝色的区域，减少红色的区域

![这里写图片描述](http://img.blog.csdn.net/20160430134934006)

通过这个工具可以查看当前区域中绘制的次数，从而尽量优化绘图层次，尽量增大蓝色的区域，减少红色的区域

###3.优化布局层级
在Android中系统对View的测量、布局和绘制都是通过遍历View树来进行的，如果View树太高，就会影响其速度，因此优化布局的第一个方法就是减低view树的高度，Google也在其api文档中建议view树的结构不宜超过十层

在早期的Android版本中，Google使用线性布局作为默认布局，而在现在的Android使用的是相对布局（默认），原因就是通过扁平的相对布局来降低通过线性布局所产生的树的高度，从而提高布局的效率

###4.避免嵌套过多无用的布局

嵌套的布局会让view树的高度越来越高，所以在布局中，需要根据自身布局的特点，来选择不同的layout组件，从而避免通过某一种layout组件来实现功能时的局限性，从而造成嵌套过多的形象

####1.使用< include>标签重用布局

在一个应用程序里，为了风格和是哪个的统一，很多界面都会存在一些共同的UI，就可以使用<  include>标签重用布局

```
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:text="this is a common ui"
    android:textSize="20sp"></TextView>
```

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <include
        layout="@layout/activity_main"
        android:layout_width="100dp"
        android:layout_height="50dp" />
</RelativeLayout>
```

####-2.使用< ViewStud>实现view的延迟加载

除了把一个view作为公用的ui，还可以对他进行延时加载，用< ViewStud>就可以轻松实现，< ViewStud>是一个轻量级的组件，不仅不可视，而且大小为0

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="not often use" />
</RelativeLayout>
```

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ViewStub
        android:id="@+id/viewStub"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:layout="@layout/activity_main" />
</RelativeLayout>
```
```
ViewStub viewStub = (ViewStub) findViewById(R.id.viewStub);
```

种方式来重新显示这个view

- VISIBLE

通过调用ViewStub的setVisibility()方法来显示这个view

```
  viewStub.setVisibility(View.VISIBLE);
```

- inflate

通过调用ViewStub的inflate方法来显示这个view

```
View inflateView = viewStub.inflate();
```

这两种方式都可以让ViewStub重新展开，显示引用的布局，而唯一的区别就是inflate方法可以返回引用的布局

###5.Hierarchy Viewer

无论哪本将优化的书，他们都不得不提到Hierarchy Viewer，不过通常情况下，Hierarchy Viewer无法再真机上进行使用，他只能在工厂的Demo机上测试即非加密过的设备，Google大神-Romain Guy提供了一个开源的View service，通过这个程序，也能让普通的手机使用Hierarchy Viewer，进入sdk目录下的/tools/启动hierarchyviewer

选择要调试的进程，然后点击上面的load view Hierarchy按钮

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical">


                </LinearLayout>

            </LinearLayout>

        </LinearLayout>
    </LinearLayout>

</LinearLayout>
```

![这里写图片描述](http://img.blog.csdn.net/20160430135018944)

使用多层的线性布局，这个代码是多余的，通常情况下，我们关心ID为content的分支，这是setvontenview所设置的内容

![这里写图片描述](http://img.blog.csdn.net/20160430135028447)


当点击一个view的时候，可以显示改view的绘制情况，不过第一次显示的时候，各种显示时间都是NA，需要点击菜单中的“profile node”重新计算

此时就可以知道每个view所绘制的时长，并且在系统的下方给出三个不同颜色的小圆点，用来表示绘制的效率，绿黄红，好，中，差

通过Hierarchy工具就可以很快的找到视图树上多余的布局了，从而有目的去优化布局了，

##二.内存优化

###1.什么是内存

由于Android的沙箱机制,每个应用所分配的内存大小是有限度的,内存太低就会触发LMK-Low Memory Killer机制。那么到底什么是内存呢?通常情况下我们所说的内存是指手机的RAM,它包括以下几个部分

- 寄存器(Registers)
>速度最快的存储场所,因为寄存器位于处理器内部.在程序中无法控制

- 栈（Stack）
>存放基本类型的数据和对象的引用.但对像本身不存放在栈中,而是存放在堆中

- 堆内存（Heap）
>堆内存用来存放由new创建的对象和数组.在堆中分配的内存，由java虚拟机的自动垃圾回收（GC）管理

- 静态存储区域(static Field)
>静态存储区域就是指在固定的位置存放应用程,子运行时一直存在的数据，java在内存中专门划分了一个静态存储区域来管理一些特殊的 数据变量如静态的数据变量

- 常量池(Constant Pool)
>JVM虚拟机必须为每个被装载的类型维护一个常量池，常量池就是该类型所用到常量的一个有序集合，包括直接常量(基本类型，String)和对其他类型. 字段和方法的符号引用

堆和栈的区分。当定义一个变量，Java虚拟机就会在栈中为该变量分配内存空间，这部分内存空间会马上被用作新的空间进行分配
如果使用new的方式创建一个变量，那么就会在堆中为这个对象分配内存空间，即使该对象的作用域结束，这部分内存也不会立即被回收。而是等待系统Gc进行回收，堆的大小随着手机的不断发展而不断变大，在过程中.可以使用如下所示的代码来获得堆的大小，所谓的内存分析，正是分析Heap中的内存状态。

```
ActivityManager manager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
int heapSize = manager.getLargeMemoryClass();
```


###2.获取Android系统内存信息
####1.Process Stats
Process Stats是KK上新增的一个系统内存监视服务. 可以通过Setting-Developer options-Process Stats来开启该功能

同样也可以使用Dumpsys命令来获取这些信息 

```
agb shell dumpsys procstats
```

####2.Meminfo
> Meminof也是系统的一个非常重要的内存监视工具，可以通过setting-apps-running打开，如图

可以通过adb命令来实现

```
agb shell dumpsys meminfo
```

#####3.内存回收

java对于C/C+这类语言最大的优势就是不用手动管理系统资源,java创建了垃圾收集器线程来自动进行资源的管理。好处是大大降低了程序开发人员对内存管理的繁琐工作，但这也带来了很多问题，例如java的gc是系统自动进行的，但何时进行却是开发者无法控制的，即使调用system.gc方法,也只是建议系统进行GC,但是系统是否采纳你的建议，那就不一定了。JVM虚拟机虽然能够自动控制GC,但是再强大的算法，也难免会存在部分对象忘记回收的现象发生, 这就是造成内存泄漏的原因

####4.内存优化实例
下面来看两个内存优化的实例 分别从Bitmap和代码两个角度来对内存进行优化。

#####Bitmap 优化
Bitmap是造成内存占用过高甚至是OOM的最大威胁,下面给出一些使用Bitmap的技巧

- 使用适当分辨率和大小的图片

- 及时回收内存
用完Bitmap后,一定要及时使用bitmap.recycle()方法来释放资源，自Android3.0后，由于bitmap被放置到了堆内存由gc管理.就不需要释放了

- 使用图片缓存
通过内存缓存(LrcCache)和硬盘缓存(DiskLrcCache) 可以更高的使用bitmap

#####代码 优化

任何Java类,都将占用大约500字节的内存空间，创建一个类的实例会消耗大约15子节的内存。从代码的实现方式上,也可以对内存进行优化 这里同样总结了一些小的技巧

 - 对常量使用static修饰符
 - 使用静态方法,静态方法会比普通方法提高15%左右的访问速度
 - 减少不必要的成员变量 ,这点在Android Lint工具上已经集成检测了.如果一个变量可以定义为局部变量，则会建议你不要定义为成成变量.
 - 减少不必要的对象 使用基础类型会比使用对象更加节省资源, 同时更应该避免频繁创建短作用域的变量
 - 尽量不要使枚举、少使用迭代器。
 - 对Cursor,Receiver，Sensor,Fiie等对象,要非常注意对它们的创建.回收与注册解注册。
 - 避免使用IOC框架.IOC通常使用注解.反射来进行实现,虽然现在java对反射的效率已经进行了很好的优化.但大量使用反射依然会带来性能的下降.
 - 使用RenderScript，openGL来进行非常复杂的绘图操作.
 - 使用surfaceView来替View进行大量.频繁的绘图操作.
 - 尽量使用视图缓存，而不是每次都执行inflaler()方法解析视图

##三.Lint工具

Android Lint工具是Android studio中继承的一个代码提示工具，他可以给你的布局，代码提供非常强大的提示

![这里写图片描述](http://img.blog.csdn.net/20160430135112292)

Lint的功能非常的强大，应该养成写完代码之后检查lint的习惯

##四.使用Android Studio的Memory Monitor工具

Memory Monitor工具是Android studio上的一个内存监视工具，他可以很好的帮助我们进行内存实时分析，通过点击Android studio右下角的Memory Monitor标签就可以进行查看了

![这里写图片描述](http://img.blog.csdn.net/20160430135121964)


##五.使用TraceView工具优化APP性能

TraceView是一个Android下的可视化性能调查工具，它用来分析TraceView的日志

###1.生成TraceView日志的两种方法
一种是利用debug类帮助我们生成日志文件。
另一种方法就是利用Android device monitor工具辅助生成文件。

####1.通过代码生成精确范围的TraceView日志

通过调用Debug.startMethodTracing()方法开启监听。
通过调用Debug.stopMethodTracing()来停止监听。
我们可以使用这两个方法来包围要监听的代码块，例如在oncreate方法里调用开启，在ondesty()方法来结束，TraceView的日志将会保存到 /sdcard/ dmtrace.trace 目录下，因此, 需要在清单文件中增加权限

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

当然, 除了使用默认的输出日志名，也可以自定义路径和日志名

当要监听的内容执行完毕后，通过ADB命令将日志文件导出到本地，命令如下所示

```
adb pull /adcard/trace_log/local/LOG/
```

####2 通过Android Device Monltor生成TiaceView日志

![这里写图片描述](http://img.blog.csdn.net/20160430135614212)

打开Android studio的Android device monitor工具,选择要调试的进程,，点击工具栏中start method profi1ing’按钮，点击后会弹出提示, 选择监听的模式，TiaceView提供了以下两种监听方式。

- 整体监听

跟踪每个方法执行的全部过程，这种方式资源消耗较大

- 抽样监听

按照指定的平率进行抽样调查，这种方式需要较长的时间来获取较为精准的样本数据，执行一段时间之后，再次点击按钮则会取消监听

###2.打开TiaceView日志
>对于导出的TiaceView日志文件，可以使用SDK的sdk\tools\traceview.bat工具来打开或者在ADM工具中，选择File下open file。

###3.分析TiaceView日志
> TiaceView的分析界面共有两部分，上面是用于显示方法的时间和时间轴区域，下面是profile区域

####1.时间轴区域

![这里写图片描述](http://img.blog.csdn.net/20160430135137683)

时间轴区域显示了不同线程在不同的时间段的执行情况，在时间轴中，每一行都代表了一个独立的线程

####2.Profile区域
>Profile区域显示了你选择的色块所代表的方法和该色块所处的时间段的性能分析

>Profile区域主要显示一下的一些信息

- Incl CPU Time - 某方法占用CPU的时间
- Excl CPU Time - 某方法本身占用的CPU
- Incl Real Time - 某方法真正执行的时间
- Call+RecurCalls - 调用次数+递归回调的次数

>每个时间都包含两列,一个是实际的时间,另一个是百分比，分析的时候，通常从Incl CPUTime和Call+RecurCalls开始进行分析，对占用时间长的方法进行重点分析,如果占用时间长且Call+RecurCalls次数少, 那么就可以列为怀疑对象了

##六.使用MAT工具分析APP内存状态

MAT工具是一个分析内存的强力助手

##1.生成HPROF文件

首先打开Android Device Monitor工具，选择要监听的线程，并且点击菜单的uPDATE HPROF按钮，

![这里写图片描述](http://img.blog.csdn.net/20160430135825613)

在HEAP标签中，点击Cause GC

![这里写图片描述](http://img.blog.csdn.net/20160430135917614)

就会显示当前的内存状态，

这里有一个判断当前是否存在内存泄漏的小技巧，当我们不停地点击Cause GC按钮的时候，如果data object中的Total size有明显变化，就代表可能存在内存泄漏

上面是手动查看Heap状态, 下面点击菜单栏的Dump HPRoF Filc按钮，如图

![这里写图片描述](http://img.blog.csdn.net/20160430140539920)

竽待几秒钟后,系统就会生成一个hprof文件,我们要分析的就是这个文件。将它保存到PC上,默认名为包名.hprof不过对这个文件我们还不能直接使用MAT工具进行分析,需要进行格式转换。在命令行下,切换到SDK目录的platfrom一tools目录下,使用hproicoow工具

```
hprof-conv
```

![这里写图片描述](http://img.blog.csdn.net/20160430141054359)

执行

![这里写图片描述](http://img.blog.csdn.net/20160430141108859)

命名格式为"hprof-conf infile outfile"，使用生成的heap.hprof文件可以利用MAT工具进行分析了

###2.分析HPROF文件
打开MAT工具，选择open a heap Dump选项，等待文件导入

![这里写图片描述](http://img.blog.csdn.net/20160430141720627)

后面还可以进行更深人的分析，MAT功能强大,这里主要看以下几个功能

 - Histogram

Histograrn直方图,用于显示内存中每个对象的数量，大小和名称。点击后打开Histogram标签

在最上方一行,可以通过搜索过滤相应的关键字,这点在分析内存中是非常有用的,比如可以过滤game关键字

在选择的对象上单击鼠标右键，在跳出的快捷菜单中选择list objects-witch incoming references 选项查看具体的对象

 - Dominator Tree

Dominator Tree支配树会将内存中的对象按照带下进行排序的，并且显示对象之间的引用结构，点击打开Dominator Thee标签，对象已经按照Retained Heap进行排序了，即按照对象及所持有的引用内存总和进行排序，通过分析内存占用找到内存消耗的原因

##七.使用Dumpsys命令分析系统状态
使用Dumpsys命令可以列出Android系统相关的信息和服务状态，Dumpsys命令的功能非常强大,可使用的参数配置也非常多，关于Dumpsys的官方信息可以从如所示的网址来获取。

https://source.android.com/devices/input/dumpsys.html

使用Dumpsys命令时.只需要输人“adb Shell dumpsys + 参数就可以，我们来获取一下acticity栈的信息

```
adb shell dumpsys activity
```

下面的列表中,总结了一些常用的Dumpsys参数

![这里写图片描述](http://img.blog.csdn.net/20160430144107935)

配合Linux下的shell命令,如grep、find等,可以让Dumpsys命令发挥非常强大的作用

