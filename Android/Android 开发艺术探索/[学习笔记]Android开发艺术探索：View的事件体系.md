# [学习笔记]Android开发艺术探索：View的事件体系

## View基础知识

View是Android所有控件的基类；View是一种界面层的控件的一种抽象，代表了一个控件；ViewGroup继承自View。

View的位置主要由它的四个定点来决定，分别对应View的四个属性：top、left、right、bottom，这下坐标都是相对父容器而言的。获取方式getXXX()。

从3.0开始View增加了x、y、translationX、translationY；x和y是View左上角的坐标，translationX和translationY是View左上方相对父容器的偏移量。

 x = left + translationX；

 y = top + translationY；

 View平移的过程中，top和left表示的是原始左上角的位置信息；其值并不会改变，此时发生改变的是x、y、translationX和translationY。

**MotionEvent**是指用户手指触摸屏幕产生的一系列事件。

- ACTION_DOWN: 手指刚接触屏幕

- ACTION_DMOVE：手指在屏幕上移动

- ACTION_UP：手指从屏幕上松开的一瞬间

getX/getY获取相对当前View左上角的x和y坐标；getRawX/getRawY获取相对手机屏幕左上角的x和y坐标。

**TouchSlop**是系统能识别滑动的最小距离，是系统常量，当手指在屏幕上滑动，小于这个距离，系统不认为你在进行滑动操作；可通过ViewConfiguration.get(getContext()).getScaledTouchSlop()方法来获取;

**VelocityTracker**用于追踪手指在滑动过程中的速度。在View的onTouchEvent方法中追踪当前单击事件的速度。

```java
VelocityTracker velocityTracker = VelocityTracker.obtain(); 
velocityTracker.addMovement(event); 
```

想知道当前滑动速度时，获取速度前需计算速度，参数是时间间隔，单位ms。

```java
velocityTracker.computeCurrentVelocity(1000);  
//获取速度
int xVelocity = (int)velocityTracker.getXVelocity(); 
int yVelocity = (int)velocityTracker.getYVelocity();
```

速度 = （终点位置 - 起点位置）/ 时间段

**GestureDetector**用于辅助检测用户的单击、滑动、长按、双击等行为。建议：如果只是监听滑动相关的推荐在onTouchEvent中实现，如果需要监听双击，使用GeststureDetector。

**Scroller**用来实现View的弹性滑动，View的scrollTo/scrollBy是瞬间完成的，使用Scroller配合View的computeScroll方法配合使用达到弹性滑动的效果

## View的滑动

scrollTo和scrollBy只能改变View内容而不能改变View本身的位置。scrollBy内部也是调用了scrollTo，它是基于当前位置的相对滑动，scrollTo是基于所传递参数的绝对滑动。在滑动过程中mScrollX/mScrollY总是等于View边缘与View内容边缘的距离，这两个属性用getScrollX/getScrollY方法获取。滑动偏移量mScrollX和mScrollY的正负与实际滑动方向相反。

使用动画来移动View,主要是操作View的translationX和translationY属性。需要注意的是，View动画只是对View的影像做操作，它并不能真正改变View的位置参数，如果这个View设置了点击事件，点击动画后的新位置无法触发点击事件的，使用属性动画没有此问题，但3.0之前系统无属性动画。

```java
ObjectAnimator.ofFloat(mButton1, "translationX", 0, 100) .setDuration(1000).start();
```

改变布局参数实现滑动，即改变LayoutParams，或者设置一个空View。如想将一个View右平移100px，只需要将该View的LayoutParams里的marginLeft增加100px即可。

```java
ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) mBtn.getLayoutParams();
params.leftMargin += 100;
mBtn.requestLayout();
//或者mBtn.setLayoutParams(params);
```

**总结：**

1. **scrollTo/scrollBy**: 操作简单，适合对View内容的滑动
2. **动画**: 操作简单，适合没有交互的View和实现负责的动画效果
3. **改变布局参数**：操作稍微复杂，适合有交互的View

## 弹性滑动

**Scroller**不能直接完成View滑动，需要配合View的computeScroll方法才可以完成弹性滑动，它让View不断重绘，每一次重绘有一个时间间隔，通过这个时间间隔Scroller就可以得出View当前滑动的位置，知道了滑动位置就通过scrollTo来完成滑动。每一次滑动都会导致View小幅度滑动，多次小幅度滑动组成了弹性滑动，这就是Scroller的工作机制。 

startScroll() 把参数保存下来，然后invalidate()会调用，导致View重绘，draw()中调用computeScroll()，该方法为空实现，去调用scrollTo(x,y)实现滑动 和 postInvalidate() 继续重绘，反复下去完成弹性滑动。

通过**动画**可以直接实现弹性滑动

```java
            ValueAnimator animator = ValueAnimator.ofInt(0,
                    1).setDuration(1000);
            animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animator) {
                    float fraction = animator.getAnimatedFraction();
                    mButton1.scrollTo(startX + (int) (deltaX * fraction), 0);
                }
            });
 						animator.start();
```



使用**延时策略**完成滑动，核心思想就是通过发送一系列延时消息从而达到一种渐进式的效果。用Handler或View的postDelayed方法，postDelayed发送延时消息，然后消息中进行View滑动，接连不断的发送这种延时消息，达到弹性滑动的效果。也可以使用线程的sleep方法来实现。

```java
    private static final int MESSAGE_SCROLL_TO = 1;
    private static final int FRAME_COUNT = 30;
    private static final int DELAYED_TIME = 33;
    private int mCount = 0;
    
    @SuppressLint("HandlerLeak")
    private Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MESSAGE_SCROLL_TO: {
                    mCount++;
                    if (mCount <= FRAME_COUNT) {
                        float fraction = mCount / (float) FRAME_COUNT;
                        int scrollX = (int) (fraction * 100);
                        mButton1.scrollTo(scrollX, 0);
                        mHandler.sendEmptyMessageDelayed(MESSAGE_SCROLL_TO, DELAYED_TIME);
                    }
                    break;
                }

                default:
                    break;
            }
        }
    };

		mHandler.sendEmptyMessageDelayed(MESSAGE_SCROLL_TO, DELAYED_TIME);

```

## View的事件分发机制

- dispatchTouchEvent(MotionEvent event) 用来处理事件的分发，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法影响，表示是否消耗该事件。
- onInterceptTouchEvent(MotionEvent event) 在dispatchTouchEvent方法内部调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那在同一个事件序列中，此方法不会再次调用，返回结果表示是否拦截当前事件。
- onTouchEvent(MotionEvent event) 在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，在同一事件序列里，当前View无法再次接收到事件。
- 三者关系可以用如下伪代码表示

```java
public boolean dispatchTouchEvent(MotionEvent ev){    
    boolean consume = false;
    if(onInterceptTouchEvent(ev)){
        consume = onTouchEvent(ev);
    }else{
        consume = child.dispatchTouchEvent(ev);   
    }
    return consume;
}
```



- 对于一个根ViewGroup，点击事件产生后，首先会传递给它，这时他的dispatchTouchEvent会调用，如果它的onInterceptTouchEvent返回true表示要拦截当前事件，接下来事件会交给这个ViewGroup处理，它的onTouchEvent就会被调用，如果这个ViewGroup的onInterceptTouchEvent返回false,则事件会继续传递给子元素，子元素的dispatchTouchEvent会调用，如此反复直到事件被处理。

- 当一个View需要处理事件时，如果设置了OnTouchListener,那么OnTouchListener的onTouch方法会回调，如果onTouch返回false,则当前View的onTouchEvent方法会被调用；如果返回true,那么onTouchEvent方法将不会调用。由此可见，OnTouchListener优先级高于onTouchEvent。OnClickListener优先级处在事件传递的尾端。

- 一个点击事件产生后，传递顺序：Activity->Window->View；如果一个View的onTouchEvent返回false,那么它的父容器的onTouchEvent会被调用，以此类推，所有元素都不处理该事件，最终将传递给Activity处理，即Activity的onTouchEvent会被调用。

- 同一个事件序列是指从手指触摸屏幕那一刻开始，到手指离开屏幕那一刻（down->move...move->up)。

- 一个事件序列只能被一个View拦截且消耗，同一个事件序列所有事件都会直接交给它处理，并且它的onInterceptTouchEvent不会再被调用。

- 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN（onTouchEvent返回了false），那么同一事件序列中其他事件都不会再交给它来处理，事件将重新交给他的父元素处理，即父元素的onTouchEvent会被调用。

- 如果某个View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以收到后续事件，最终这些消失的点击事件会传递给Activity处理。

- ViewGroup默认不拦截任何事件，ViewGroup的onInterceptTouchEvent方法默认返回false。

- View没有onInterceptTouchEvent方法，一旦有事件传递给它，那么它的onTouchEvent方法就会被调用。

- View的onTouchEvent方法默认消耗事件（返回true）,除非他是不可点击的（clickable和longClickable同时为false）。View的longClickable属性默认都为false,clickable属性分情况，Button默认为true，TextView默认为false。

- onClick发生的前提是View可点击，并且它收到了down和up事件。

- 事件传递过程是由内而外，事件总是先传递给父元素，然后在由父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素干预父元素的事件分发过程，但ACTION_DOWN事件除外。

## View的滑动冲突

1.常见滑动冲突场景

- 场景1 —— 外部滑动方向与内部滑动方向不一致，比如ViewPager中包含ListView;
- 场景2 —— 外部滑动方向与内部滑动方向一致，比如ScrollView中包含ListView;
- 场景3 —— 上面两种情况的嵌套

2.滑动冲突处理规则

通过判断是水平滑动还是竖直滑动来判断到底应该谁来拦截事件；可以根据水平和竖直两个方向的距离差或速度差来做判断。

对于场景一，处理的规则是：当用户左右（上下）滑动时，需要让外部的View拦截点击 事件，当用户上下（左右）滑动的时候，需要让内部的View拦截点击事件。根据滑动的方向判断谁来拦截事件。

对于场景二，由于滑动方向一致，这时候只能在业务上找到突破点，根据业务需求，规 定什么时候让外部View拦截事件，什么时候由内部View拦截事件。

场景三的情况相对比较复杂，同样根据需求在业务上找到突破点。

3.滑动冲突解决方式

- 外部拦截法 —— 即点击事件先经过父容器的拦截处理，如果父容器需要此事件就拦截，不需要就不拦截，需要重写父容器的onInterceptTouchEvent方法；在onInterceptTouchEvent方法中，首先ACTION_DOWN这个事件，父容器必须返回false,即不拦截ACTION_DOWN事件，因为一旦父容器拦截了ACTION_DOWN,那么后续的ACTION_MOVE/ACTION_UP都会直接交给父容器处理；其次是ACTION_MOVE,根据需求来决定是否要拦截;最后ACTION_UP事件,这里必须要返回false,在这里没有多大意义。

```java
public boolean onInterceptTouchEvent (MotionEvent event){
      boolean intercepted = false;
      int x = (int) event.getX();
      int y = (int) event.getY();
      switch (event.getAction()) {
          case MotionEvent.ACTION_DOWN:
          	intercepted = false;
          break;
          case MotionEvent.ACTION_MOVE:
            if (父容器需要当前事件） {
              intercepted = true;
            } else {
              intercepted = flase;
            }
          break;
          case MotionEvent.ACTION_UP:
          	intercepted = false;
          break;
          default : break;
      }
      mLastXIntercept = x;
      mLastYIntercept = y;
      return intercepted;
}
```

- 内部拦截法 —— 内部拦截法是指父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗，否则就交由父容器进行处理。这种方法与Android事件分发机制不一致，需要配合requestDisallowInterceptTouchEvent方法才能正常工作。

```java
public boolean dispatchTouchEvent ( MotionEvent event ) {
      int x = (int) event.getX();
      int y = (int) event.getY();
      switch (event.getAction) {
          case MotionEvent.ACTION_DOWN:
          		parent.requestDisallowInterceptTouchEvent(true);
          break;
          case MotionEvent.ACTION_MOVE:
              int deltaX = x - mLastX;
              int deltaY = y - mLastY;
              if (父容器需要此类点击事件) {
              		parent.requestDisallowInterceptTouchEvent(false);
              }
          break;
          case MotionEvent.ACTION_UP:
          break;
          default : break;
      }
      mLastX = x;
      mLastY = y;
      return super.dispatchTouchEvent(event);
}
```

除了子元素需要做处理外，父元素也要默认拦截除了ACTION_DOWN以外的其他事件， 这样当子元素调用parent.requestDisallowInterceptTouchEvent(false)方法时，父元素才能继续拦截所需的事件。（ACTION_DOWN事件不受requestDisallowInterceptTouchEvent方法影响,所以一旦父元素拦截ACTION_DOWN事件,那么所有元素都无法传递到子元素去）因此，父元素要做以下修改：

```java
public boolean onInterceptTouchEvent (MotionEvent event) {
    int action = event.getAction();
    if(action == MotionEvent.ACTION_DOWN) {
    		return false;
    } else {
    		return true;
    }
}
```



