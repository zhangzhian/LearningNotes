# [学习笔记]Android开发艺术探索：View的工作原理

## 初识ViewRoot和DecorView

ViewRoot的实现是 ViewRootImpl 类，是连接WindowManager和DecorView的纽带， View的三大流程（mearsure、layout、draw）均是通过ViewRoot来完成。当Activity对象被创建完毕后，会将DecorView添加到Window中，同时创建 ViewRootImpl 对象，并将 ViewRootImpl 对象和DecorView建立连接。

```java
  root = new ViewRootImpl(view.getContext(),display); 
  root.setView(view,wparams, panelParentView);
```

View的三大流程:

1. measure用来测量View的宽高 

2. layout来确定View在父容器中的位置 

3. draw负责将View绘制在屏幕上

View的绘制流程从ViewRoot的 performTraversals 方法开始：

1. performTraversals会依次调用 performMeasure 、 performLayout 和 performDraw 三个方法，这三个方法分别完成顶级View的measure、layout和draw这三大流程。 

2. 其中 performMeasure 中会调用 measure 方法，在 measure 方法中又会调用 onMeasure 方法，在 onMeasure 方法中则会对所有子元素进行measure过程，这样就完成了一次measure过程；子元素会重复父容器的measure过程，如此反复完成了整个View数的遍历。另外两个过程同理。

Measure完成后, getMeasuredWidth / getMeasureHeight 方法来获取View测量后的宽/高。

Layout过程决定了View的四个顶点的坐标和实际View的宽高，完成后可通 过 getTop 、 getBotton 、 getLeft 和 getRight 拿到View的四个顶点坐标。

DecorView作为顶级View，其实是一个 FrameLayout ，它包含一个竖直方向 的 LinearLayout ，这个 LinearLayout 分为标题栏和内容栏两个部分。在Activity通过 setContextView所设置的布局文件其实就是被加载到内容栏之中的。这个内容栏的id 是 R.android.id.content ，通过 `ViewGroup content = findViewById(R.android.id.content);` 可以得到这个contentView。View层的事件都是先经 过DecorView，然后才传递到子View。

## 理解MeasureSpec

测量过程，系统将View的 LayoutParams 根据父容器所施加的规则转换成对应的 MeasureSpec，然后根据这个MeasureSpec来测量出View的宽高。

MeasureSpec代表一个32位int值，高2位代表SpecMode（测量模式），低30位代表 SpecSize（在某个测量模式下的规格大小）。

SpecMode有三种： 

1. UNSPECIFIED ：父容器不对View进行任何限制，要多大给多大，一般用于系统内部 

2. EXACTLY：父容器检测到View所需要的精确大小，这时候View的最终大小就是 SpecSize所指定的值，对应LayoutParams中的 match_parent 和具体数值这两种模式 

3. AT_MOST ：对应View的默认大小，不同View实现不同，View的大小不能大于父容 器的SpecSize，对应 LayoutParams 中的 wrap_content

View的MeasureSpec由父容器的MeasureSpec和自身的LayoutParams共同决定。 View的measure过程由ViewGroup传递而来，参考ViewGroup的 measureChildWithMargins 方 法，通过调用子元素的 getChildMeasureSpec 方法来得到子元素的MeasureSpec，再调用子元素的 measure 方法。

1. 当View采用固定宽/高时（即设置固定的dp/px）,不管父容器的MeasureSpec是什么， View的MeasureSpec都是EXACTLY模式，并且大小遵循我们设置的值。 
2. 当View的宽/高是 match_parent 时，View的MeasureSpec都是EXACTLY模式并且其大小等于父容器的剩余空间。 
3. 当View的宽/高是 wrap_content 时，View的MeasureSpec都是AT_MOST模式并且其大小不能超过父容器的剩余空间。 
4. 父容器的UNSPECIFIED模式，一般用于系统内部多次Measure时，表示一种测量的状态，一般来说我们不需要关注此模式。

## View的工作流程

### measure过程

分两种情况： 1. View通过 measure 方法就完成了测量过程 2. ViewGroup除了完成自己的测量过程还会便利调用所有子View的 measure 方法，而且各 个子View还会递归执行这个过程。

#### View的measure过程

```cpp
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {          
     setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(),      widthMeasureSpec),     
     getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));}
```

- setMeasuredDimension方法会设置View的宽/高的测量值
- getDefaultSize方法返回的大小就是measureSpec中的specSize，也就是View测量后的大小，绝大部分情况和View的最终大小(layout阶段确定)相同。
- getSuggestedMinimumWidth方法，作为getDefaultSize的第一个参数（建议宽度)

直接继承View的自定义控件需要重写 onMeasure 方法并设置 wrap_content （即specMode 是 AT_MOST 模式）时的自身大小，否则在布局中使用 wrap_content 相当于使 用 match_parent 。对于非 wrap_content 的情形，我们沿用系统的测量值即可。

#### ViewGroup的measure过程

ViewGroup是一个抽象类，没有重写View的 onMeasure 方法，但是它提供了一 个 measureChildren 方法。这是因为不同的ViewGroup子类有不同的布局特性，导致他们的测量细节各不相同，比如 LinearLayout 和 RelativeLayout ，因此ViewGroup没办法同一实现 onMeasure 方法。

measureChildren方法的流程： 

1. 取出子View的 LayoutParams
2. 通过 getChildMeasureSpec 方法来创建子元素的 MeasureSpec
3. 将 MeasureSpec 直接传递给View的measure方法来进行测量

View的measure过程是三大流程中最复杂的一个，measure完成以后，通过 getMeasuredWidth/Height 方法就可以正确获取到View的测量后宽/高。在某些情况下，系统可能需要多次measure才能确定最终的测量宽/高，所以在onMeasure中拿到的宽/高很可能不是准确的。同时View的measure过程和Activity的生命周期并不是同步执行，因此无法保证在 Activity的 onCreate、onStart、onResume 时某个View就已经测量完毕。所以有以下四种方式来 获取View的宽高： 

1. Activity/View#onWindowFocusChanged。 onWindowFocusChanged这个方法的含义 是：VieW已经初始化完毕了，宽高已经准备好了，需要注意：它会被调用多次，当 Activity的窗口得到焦点和失去焦点均会被调用。 

2. view.post(runnable)。 通过post将一个runnable投递到消息队列的尾部，当Looper调用此 runnable的时候，View也初始化好了。 

3. ViewTreeObserver。 使用 ViewTreeObserver 的众多回调可以完成这个功能，比如 OnGlobalLayoutListener 这个接口，当View树的状态发送改变或View树内部的View的 可见性发生改变时， onGlobalLayout 方法会被回调。需要注意的是，伴随着View树状态 的改变， onGlobalLayout 会被回调多次。 

4. view.measure(int widthMeasureSpec,int heightMeasureSpec)。

   (1) match_parent：

   ​	无法measure出具体的宽高，因为不知道父容器的剩余空间，无法测量出View的大小

   (2) 具体的数值（dp/px）:

   ```cpp
   int widthMeasureSpec = MeasureSpec.makeMeasureSpec(100,MeasureSpec.EXACTLY);
   int heightMeasureSpec = MeasureSpec.makeMeasureSpec(100,MeasureSpec.EXACTLY);
   view.measure(widthMeasureSpec,heightMeasureSpec);
   ```

   (3) wrap_content：

   ```cpp
   int widthMeasureSpec = MeasureSpec.makeMeasureSpec((1<<30)-1,MeasureSpec.AT_MOST);
   int heightMeasureSpec = MeasureSpec.makeMeasureSpec((1<<30)-1,MeasureSpec.AT_MOST);
   view.measure(widthMeasureSpec,heightMeasureSpec);
   ```

### layout过程

layout的作用是ViewGroup用来确定子View的位置，当ViewGroup的位置被确定后，它会在 onLayout中遍历所有的子View并调用其layout方法，在 layout 方法中， onLayout 方法又会被调用。 layout 方法确定View本身的位置，源码流程如下：

1. setFrame 确定View的四个顶点位置，即确定了View在父容器中的位置。
2. 调用 onLayout 方法，确定所有子View的位置。

### draw过程

View的绘制过程遵循如下几步：

1. 绘制背景 drawBackground(canvas)
2. 绘制自己 onDraw
3. 绘制children dispatchDraw 遍历所有子View的 draw 方法
4.  绘制装饰 onDrawScrollBars

View的绘制过程是通过dispatchDraw来实现的，它会遍历所有子元素的draw方法。

如果一个View不需要绘制任何内容，那么设置setWillNotDraw为true后，系统会进行相应的优化；ViewGroup默认为true，如果我们的自定义ViewGroup需要通过onDraw来绘制内容的时候，需要显示的关闭它。

## 自定义View

自定义View的分类

- 继承View 通过 onDraw 方法来实现一些效果，需要自己支持 wrap_content ，并且 padding也要去进行处理。

- 继承ViewGroup 实现自定义的布局方式，需要合适地处理ViewGroup的测量、布局这两 个过程，并同时处理子View的测量和布局过程。

- 继承特定的View子类（如TextView、Button） 扩展某种已有的控件的功能，且不需要自 己去管理 wrap_content 和padding。

- 继承特定的ViewGroup子类（如LinearLayout）

直接继承View或ViewGroup的控件， 需要在onMeasure中对wrap_content做特殊处理。

直接继承View的控件，如果不在draw方法中处理padding，那么padding属性就无法起作用。直接继承ViewGroup的控件也需要在onMeasure和onLayout中考虑padding和子元素margin的影响，不然padding和子元素的margin无效。

View内部提供了post系列的方法，完全可以替代Handler的作用。

View中有线程和动画，需要在View的onDetachedFromWindow中停止。

View带有滑动嵌套情形时，需要处理好滑动冲突





