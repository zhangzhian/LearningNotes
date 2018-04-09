# [学习笔记] Android群英传：Android Scroll分析
---


##一.滑动效果的产生

###1.Android坐标系
在Android，系统将屏幕最左上角的顶点作为Android坐标系的原点，从这个点向右是X轴正方向，从这个点向下是Y轴正方向，如图

![这里写图片描述](http://img.blog.csdn.net/20160321233639365)

系统提供了`getLocationOnScreen(intlocation[])`来获取Android坐标中的位置，即该视图左上角Android的坐标，另外，在触摸事件中使用getRawX(),getRawY()方法来获取Android坐标系中的坐标。

###2.视图坐标系
视图坐标系，描述了子视图在父视图的位置关系，视图坐标系同样的以原点向右为X正方向，以原点向下为Y方法，在视图坐标系中，原点不再是Android坐标系中的屏幕左上角，而是以父视图左上角为坐标原点

![这里写图片描述](http://img.blog.csdn.net/20160321233650597)

>在触控事件中通过getX,getY来获取的坐标就是视图坐标中的坐标

###3.触控事件——MotionEvent
看MotionEvent中封装了一些常量，定义了触摸事件的不同类型。

```
	//单点触摸按下的动作
    public static final int ACTION_DOWN = 0;
    //单点触摸离开的动作
    public static final int ACTION_UP = 1;
    //单点触摸移动的动作
    public static final int ACTION_MOVE = 2;
    //单点触摸取消
    public static final int ACTION_CANCEL = 3;
    //单点触摸超出边界
    public static final int ACTION_OUTSIDE = 4;
    //多点触摸按下的动作
    public static final int ACTION_POINTER_DOWN = 5;
    //多点触摸离开的动作
    public static final int ACTION_POINTER_UP = 6;
```

通常情况下，我们会在onTouchEvent(MotionEvent event)方法中通过event.getAction()来获取触摸事件的类型，并使用switch来判断

```
 	@Override
    public boolean onTouchEvent(MotionEvent event) {
        //获取当前输入点的X,Y坐标（视图坐标）
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //处理输入的按下动作
                break;
            case MotionEvent.ACTION_MOVE:
                //处理输入的移动动作
                break;
            case MotionEvent.ACTION_UP:
                //处理输入的离开动作
                break;
        }
        return true;
    }
```

下图总结一些常用的API：

![这里写图片描述](http://img.blog.csdn.net/20160321233701756)

这些方法可以分成两个类别

  **View提供的获取坐标方法**
 
> getTop():获取到的是View自身的顶部到其父布局顶部的距离
> 
> getLeft():获取到的是View自身的左边到其父布局左边的距离
> 
> getRight():获取到的是View自身的右边到其父布局右边的距离
> 
> getBottom():获取到的是View自身的底部到其父布局底部的距离

---
 
  **MotionEvent提供的方法**

> getX():获取点击事件距离控件左边的距离，即视图坐标
> 
> getY():获取点击事件距离控件顶部的距离，即视图坐标
> 
> getRawX:获取点击事件整个屏幕左边的距离，即绝对坐标
> 
> getRawY:获取点击事件整个屏幕顶部的距离，即绝对坐标


##二.实现滑动的七种方法

通过实例来看看Android中如何实现滑动的效果：
定义一个View，简单的实现一个布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.zza.demo.DragView
        android:layout_width="100dp"
        android:layout_height="100dp" />
</RelativeLayout>

```
默认的显示:

![这里写图片描述](http://img.blog.csdn.net/20160321233714740)

###1.layout方法
在View的绘制上，会调用onLayout()方法来设置显示的位置，同样可以修改View的left,top,right,bottom四个属性来控制View的坐标

```
 //触摸事件
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int rawX = (int) event.getRawX();
        int rawY = (int) event.getRawY();
        int lastX = 0;
        int lastY = 0;
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //记录触摸点的坐标
                lastX = rawX;
                lastY = rawY;
                break;
            case MotionEvent.ACTION_MOVE:
                //计算偏移量
                int officeX = rawX - lastX;
                int officeY = rawY - lastY;
                //在当前的left,top,right,bottom基础上加上偏移量
                layout(getLeft()+officeX,getTop()+officeY,getRight()+officeX,getBottom()+officeY);
                //重新设置初始值
                lastX = rawX;
                lastY = rawY;
                break;
            case MotionEvent.ACTION_UP:
                //处理输入的离开动作
                break;
        }
        return true;
    }

```
###2.offsetLeftAndRight()与offsetTopAndBottom()
系统提供的一个对左右，上下移动的封装，当计算出偏移量的时候，只需要使用如下的代码就可以完成View的重新布局，效果和使用Layout（）方法是一样的

```
//同时对左右偏移
offsetLeftAndRight(officeX);
//同时对上下偏移
offsetTopAndBottom(officeY);

```

###3.LayoutParams
LayoutParams保留了一个View的布局参数，因此可以在程序中，通过改变LayoutParams来动态改变一个布局的位置参数，从而改变View位置的效果，我们可以很方便的在程序中使用getLayoutParams()来获取一个View的LayoutParams，当然，在计算偏移量的方法和Layout方法中计算offset是一样的，当获取到偏移量之后，可以通过setLayoutParams来改变LayoutParams:

```
LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
                layoutParams.leftMargin = getLeft()+officeX;
                layoutParams.topMargin = getTop()+officeY;
                setLayoutParams(layoutParams);
```

>注意，通过getLayoutParams()获取layoutParams时，需要根据View所在的跟布局的类型来设置不同的类型，比如View放在LinearLayout里那就是LinearLayout.LayoutParams，比如在RelativeLayout里就是 RelativeLayout.LayoutParams，不然系统是无法获取到layoutParams的.

在通过一个layoutParams来改变一个View的位置时，通常改变的是这个view的Margin属性，所以除了使用布局的layoutParams属性外，还需要 ViewGroup.MarginLayoutParams来实现这样的功能

```
ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
                layoutParams.leftMargin = getLeft()+officeX;
                layoutParams.topMargin = getTop()+officeY;
                setLayoutParams(layoutParams);
```

>用ViewGroup不用去管父布局是什么

###4.scrollTo与scrollBy
在一个View当中，系统提供了scrollTo与scrollBy这两种方式来实现移动一个View的位置
scrollTo(x,y);表示移动到一个具体的点
scrollBy(dx,dy);表示移动的增量

```
int officeX = rawX - lastX;
int officeY = rawY - lastY;
scrollBy(officeX,officeY);
```

>但是，当拖动View的时候，View并没有移动，只是移动了view的content，如果用ViewGroup使用to和by的话，那所有的子View都将移动，要是在View中使用的话，那么移动的就是View的内容了

在该View所在的ViewGroup中使用scrollBy方法来移动这个view

```
  ((View)getParent()).scrollBy(officeX,officeY);
```

>但是，拖动View的时候，View虽然移动了，并不是我们想要的跟随触摸点的移动而移动

当调用scrollBy的方法时，可以想象外面的ViewGroup在移动，具体的例子

![这里写图片描述](http://img.blog.csdn.net/20160322202614727)

上图，中间的矩形相当于屏幕，即可视区域，后面的content相当于画布，代表视图，只有视图的中间部分目前是可视的，其他部分都不可见，可见区域中设置一个button,他的坐标是（20.10），下面我们使用scrollBy(20.10)方法来进行移动，如图：

![这里写图片描述](http://img.blog.csdn.net/20160322221559982)

设置scrollBy(20.10)，偏移量均为XY的正方向，但是屏幕的可视区域，Button却向反方向移动了。
参考系选择的不同，产生不同效果。

>我们将scrollBy的参数dx,dy设置成正数，那么content将向坐标轴负方向移动，反之，则正方向

```
int officeX = rawX - lastX;
int officeY = rawY - lastY;
scrollBy(-officeX,-officeY);
```
###5.Scroller
Scroller就可以实现平滑的效果

- 初始化scroller
首先，通过他的构造方法来创建一个scroller对象

```
//初始化mScroller
mScroller = new Scroller(context);
```
- 重写computeScroll，实现模拟滑动
computeScroll这个方法，是使用Scroller的核心，系统在绘制View的同时，会在onDraw()方法中调用这个方法
通常情况下，computeScroll的代码可以利用标准的写法：

```
	/**
     * 模拟滑动
     */
    @Override
    public void computeScroll() {
        super.computeScroll();
        
        //判断Scroller是否执行完毕
        if(mScroller.computeScrollOffset()){
            ((View)getParent()).scrollTo(mScroller.getCurrX(),mScroller.getCurrY());
        }
        //通过重绘来不断调用computeScroll
        invalidate();
    }
```

Scroller类提供了computeScrollOffset（）来判断是否完成了整个页面的滑动，提供了getCurrX（），getCurrY（）来获取当前滑动坐标。
>注意invalidate()，只能在computeScroll中获得模拟过程中的scrollX,scrollY坐标，但computeScroll方法是不会自动调用的，只能通过invalidate——>OnDraw——>computeScroll来调用，所以需要这个invalidate，而当模拟过程结束的时候，computeScrollOffset返回的是false，从而结束循环

- startScroll开启模拟过程
使用平滑移动事件，使用Scroller类的startScroll（）方法来开启平滑过程，startScroll有两个重载的方法
	- public void startScroll(int startX,int startY,int dx,int dy)
	- public void startScroll(int startX,int startY,int dx)

实例，在构造分钟初始化Scroller对象，然后重写computeScroll方法，最后需要监听手指离开屏幕的事件，并在该事件之后调用startScroll()完成平移，所以我们在ACTION_UP中

```
case MotionEvent.ACTION_UP:
                //处理输入的离开动作
                View view = ((View)getParent());
                mScroller.startScroll(view.getScrollX(),view.getScrollY(),-view.getScrollX(),-view.getScrollY());
                invalidate();
                break;
```

###6.ViewDragHelper
Google在其support库中为我们提供了一个DrawerLayout和SlidingPaneLayout两个布局来帮助开发者实现侧滑效果，这两个布局背后，隐藏着一个ViewDragHelper，通过ViewDragHelper，基本可以实现各种不同的侧滑，拖放需求

>使用ViewDragHelper实现一个**QQ滑动侧边栏**的布局

- 初始化ViewDragHelper

首先是初始化ViewDragHelper，ViewDragHelper通常定义在一个ViewGroup中，通过其静态方法初始化

```
 mViewDragHelper = ViewDragHelper.create(this,callback);
```

>他的第一个参数是要监听的View,第二个参数是一个Callback回调，这个回调是整个业务的核心，

- 拦截事件

要重写拦截事件，将事件传递给ViewDragHelper进行处理

```
 //事件拦截
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {

        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }

    //触摸事件
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //将触摸事件传递给ViewDragHelper

        mViewDragHelper.processTouchEvent(event);

        return true;
    }
```

- 处理computeScroll（）

使用ViewDragHelper也是需要重写computeScroll的，因为ViewDragHelper内部也是通过Scroller来实现平移的，我们可以这样使用

```
    @Override
    public void computeScroll() {
        if(mViewDragHelper.continueSettling(true)){
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }

```

- 处理回调Cakkback


```
//侧滑回调
    private ViewDragHelper.Callback callback = new ViewDragHelper.Callback() {
        //何时开始触摸
        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            //如果当前触摸的child是mMainView开始检测
            return mMainView == child;
        }

        //处理水平滑动
        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            return left;
        }

        //处理垂直滑动
        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            return 0;
        }

        //拖动结束后调用
        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            super.onViewReleased(releasedChild, xvel, yvel);
            //手指抬起后缓慢的移动到指定位置
            if(mMainView.getLeft() <500){
                //关闭菜单
                mViewDragHelper.smoothSlideViewTo(mMainView,0,0);
                ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
            }else{
                //打开菜单
                mViewDragHelper.smoothSlideViewTo(mMainView,300,0);
                ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
            }
        }
    };
```

下面自定义一个viewGroup来完成整个编码的实例

```
/**
 * 侧滑
 */
public class DragViewGroup extends FrameLayout{

    //侧滑类
    private ViewDragHelper mViewDragHelper;
    private View mMenuView,mMainView;
    private int mWidth;

    public DragViewGroup(Context context) {
        super(context);
        initView();

    }

    public DragViewGroup(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public DragViewGroup(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    //初始化数据
    private void initView() {
        mViewDragHelper = ViewDragHelper.create(this,callback);
    }

    //XML加载组建后回调
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mMenuView = getChildAt(0);
        mMainView = getChildAt(1);
    }


    //组件大小改变时回调
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = mMenuView.getMeasuredWidth();
    }

    //事件拦截
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {

        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }

    //触摸事件
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //将触摸事件传递给ViewDragHelper

        mViewDragHelper.processTouchEvent(event);

        return true;
    }

    //侧滑回调
    private ViewDragHelper.Callback callback = new ViewDragHelper.Callback() {
        //何时开始触摸
        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            //如果当前触摸的child是mMainView开始检测
            return mMainView == child;
        }

        //处理水平滑动
        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            return left;
        }

        //处理垂直滑动
        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            return 0;
        }

        //拖动结束后调用
        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            super.onViewReleased(releasedChild, xvel, yvel);
            //手指抬起后缓慢的移动到指定位置
            if(mMainView.getLeft() <500){
                //关闭菜单
                mViewDragHelper.smoothSlideViewTo(mMainView,0,0);
                ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
            }else{
                //打开菜单
                mViewDragHelper.smoothSlideViewTo(mMainView,300,0);
                ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
            }
        }
    };

    @Override
    public void computeScroll() {
        if(mViewDragHelper.continueSettling(true)){
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }
}

```

在Cakkback中系统提供了很多的方法来监听

- onViewCaptured

```
        //用户触摸到view回调
        @Override
        public void onViewCaptured(View capturedChild, int activePointerId) {
            super.onViewCaptured(capturedChild, activePointerId);
        }
```

- onViewDragStateChanged

```
        //拖拽状态改变时，比如idle,dragging
        @Override
        public void onViewDragStateChanged(int state) {
            super.onViewDragStateChanged(state);
        }
```

- onViewPositionChanged

```
//位置发生改变，常用语滑动scale效果
        @Override
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
            super.onViewPositionChanged(changedView, left, top, dx, dy);
        }
```

