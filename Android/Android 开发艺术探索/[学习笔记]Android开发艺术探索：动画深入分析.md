# [学习笔记]Android开发艺术探索：动画深入分析

Android动画分为三种：  View动画、帧动画、属性动画

## View动画

View动画的作用对象是View，支持四种动画效果：平移 、缩放、旋转、透明。四种变换效果对应着Animation四个子类： TranslateAnimation 、 ScaleAnimation 、 RotateAnimation 和 AlphaAnimation 。这四种动画皆可以通过XML定义，也可以通过代码来动态创建。

set标签表示动画集合，对应AnimationSet类，可以包含一个或若干个动画，内部还可以嵌套其他动画集合。

android:interpolator 表示动画集合所采用的插值器，插值器影响动画速度，比如非匀速动画就需要通过插值器来控制动画的播放过程。

android:shareInterpolator 表示集合中的动画是否和集合共享同一个插值器，如果集合不指定插值器，那么子动画就需要单独指定所需的插值器或默认值。

translate、 scale 、  rotate、 alpha 这几个子标签分别代表四种变换效果。

Animation通过setAnimationListener方法可以给View动画添加过程监听。

**自定义View动画**需要继承 Animation 这个抽象类，重写它的 initialize 和 applyTransformation 方法。 在 initialize 方法中做一些初始化工作，在 applyTransformation 中进行相应的矩阵变换即可，很多时候需要采用 Camera 来简化矩阵变换的过程。自定义View动画的过程主要是矩阵变 换的过程。

**帧动画**是顺序播放一组预先定义好的图片，使用简单，但容易引起OOM，所以在使用帧动画 时应尽量避免使用过多尺寸较大的图片。

## View动画的特殊使用场景

**LayoutAnimation**作用于ViewGroup,为ViewGroup指定一个动画，当他的子元素出场的时候都会具有这种动画，ListView上用的多，LayoutAnimation也是一个View动画。

anim_layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:animationOrder="normal"
    android:delay="0.3" android:animation="@anim/anim_item"/>
```

anim_item:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="300"
    android:shareInterpolator="true">
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />
    <translate
        android:fromXDelta="300"
        android:toXDelta="0" />
</set>
```

使用LayoutAnimation:

第一种方法、为需要的ViewGroup指定android:layoutAnimation属性

```xml
<ListView
       android:id="@+id/lv"
       android:layout_width="match_parent"
       android:layout_height="0dp"
       android:layout_weight="1"
       android:layoutAnimation="@anim/anim_layout"/>
```

第二种方法、通过LayoutAnimationController来实现

```java
Animation animation = AnimationUtils.loadAnimation(this, R.anim.anim_item);
LayoutAnimationController controller = new LayoutAnimationController(animation);
controller.setDelay(0.5f);
controller.setOrder(LayoutAnimationController.ORDER_NORMAL);
listview.setLayoutAnimation(controller);
```

在startActivity(Intent)或finish()之后调用overridePendingTransition(int enterAnim,int exitAnim)方法。

Fragment也可以添加切换动画，通过FragmentTransaction中的setCustomAnimations()方法来添加；需考虑兼容性使用View动画，属性动画是API11新引入的。

## 属性动画

API 11后加入，可以在一个时间间隔内完成对象从一个属性值到另一个属性值的改变。因此与 View动画相比，属性动画几乎无所不能，只要对象有这个属性，它都能实现动画效果。API11 以下可以通过 nineoldandroids 库来兼容以前版本。

属性动画有以下三种使用方法：

1. ObjectAnimator:

```java
ObjectAnimator.ofFloat(view,"translationY",values).start();
```

2. ValueAnimator:

```java
ValueAnimator colorAnim = ObjectAnimator.ofInt(view,"backgroundColor",/*red*/0xff
ff8080,/*blue*/0xff8080ff);
colorAnim.setDuration(2000);
colorAnim.setEvaluator(new ArgbEvaluator());
colorAnim.setRepeatCount(ValueAnimator.INFINITE);
colorAnim.setRepeatMode(ValueAnimator.REVERSE);
colorAnim.start();
```

3. AnimatorSet

```java
AnimatorSet set = new AnimatorSet();
set.playTogether(animator1,animator2,animator3); 
set.setDuration(3*1000).start();
```

也可以通过在xml中定义在 res/animator/ 目录下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
<objectAnimator
....../>
<animator
....../>
</set>
```

```java
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(context , R.animator.ani
m);
set.setTarget(view);
set.start();
```

set 标签对应 AnimatorSet ，  animator对应 ValueAnimator ，而objectAnimator则对 应 ObjectAnimator 。

### 理解插值器和估值器

**时间插值器**（TimeInterpolator）的作用是**根据时间流逝的百分比来计算出当前属性值改变的百分比**，系统预置的有LinearInterpolator（线性插值器：匀速动画），AccelerateDecelerateInterpolator（加速减速插值器：动画两头慢中间快），DecelerateInterpolator(减速插值器：动画越来越慢）。

**估值器**（TypeEvaluator）的作用是**根据当前属性改变的百分比来计算改变后的属性值**。系统预置有IntEvaluator 、FloatEvaluator 、ArgbEvaluator。

整形估值器源码:

```java
public class IntEvaluator implements TypeEvaluator<Integer> {
    public Integer evaluate(float fraction, Integer startValue, Integer endValue) {
        int startInt = startValue;
        return (int)(startInt + fraction * (endValue - startInt));
    }
}
```

插值器和估值器除了系统提供之外，我们还可以自定义。自定义插值器需要实现Interpolator或者TimeInterpolator；自定义估值器算法需要实现TypeEvaluator。

### 属性动画的监听器

```csharp
public static interface AnimatorListener {
    void onAnimationStart(Animator animation); //动画开始
    void onAnimationEnd(Animator animation); //动画结束
    void onAnimationCancel(Animator animation); //动画取消
    void onAnimationRepeat(Animator animation); //动画重复播放
}
```

为了方便开发，系统提供了AnimatorListenerAdapter类，它是AnimatorListener的适配器类， 可以有选择的实现以上4个方法。

AnimatorUpdateListener会监听整个动画的过程，动画由许多帧组成的，每播放一帧，onAnimationUpdate就会调用一次。

```java
/**
* Implementors of this interface can add themselves as update listeners
* to an <code>ValueAnimator</code> instance to receive callbacks on every animati
on
* frame, after the current frame's values have been calculated for that
* <code>ValueAnimator</code>.
*/
public static interface AnimatorUpdateListener {
    /**
    * <p>Notifies the occurrence of another frame of the animation.</p>
    *
    * @param animation The animation which was repeated.
    */
    void onAnimationUpdate(ValueAnimator animation);
}
```

### 对任意属性做动画

属性动画要求作用的对象提供该属性的get和set方法，属性动画根据外界传递的该属性的初始值和最终值，通过多次调用set方法来实现动画效果。

如果你的对象没有对应的get和set方法

- 请给你的对象加上get和set方法，如果你有权限的话（如果直接使用系统的类，是无法加上的）；
- 用一个类来包装原始对象，间接为其提供get和set方法；

```cpp
public class ViewWrapper {
      private View target;
      public ViewWrapper(View target) {
          this.target = target;
      }
      public int getWidth() {
          return target.getLayoutParams().width;
      }
      public void setWidth(int width) {
          target.getLayoutParams().width = width;
          target.requestLayout();
      }
}
```

- 采用ValueAnimator，监听动画过程，自己实现属性的改变；

```java
   /** 使用ValueAnimator 监听动画过程 自己实现属性改变
     *
     * @param target 作用的View
     * @param start 动画起始值
     * @param end 动画终止值
     */
    private void startValueAnimator(final View target, final int start, final int end) {
        ValueAnimator valueAnimator = ValueAnimator.ofInt(1, 100);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            private IntEvaluator mEvaluation = new IntEvaluator();//新建一个整形估值起 方便估值使用

            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //获得当前动画的进度值 1~100之间
                int currentValue = (int) animation.getAnimatedValue();
                Log.e("Anim", "currentValue: " + currentValue);
                //获得当前进度占整个动画过程的比例，浮点型，0~1之间
                float fraction = animation.getAnimatedFraction();
                //调用估值器，通过比例计算出宽度 ，再设置给Button
                int targetWidth = mEvaluation.evaluate(fraction, start, end);
                target.getLayoutParams().width = targetWidth;
                target.requestLayout();
            }
        });
    }
```

### 属性动画的工作原理

通过反射调用get/set方法；属性动画需要运行在有Looper的线程中。

## 使用动画的注意事项

- 使用帧动画时，当图片数量较多且图片分辨率较大的时候容易出现**OOM**，需注意，尽量避免使用帧动画。

- 使用无限循环的属性动画时，在Activity退出时及时停止，否则将导致Activity无法释放从而造成**内存泄露**。

- View动画是对View的影像做动画，并不是真正的改变了View的状态，因此有时候会出现动画完成后View无法隐藏（setVisibility(View.GONE）失效），这时候调用view.clearAnimation()清理View动画即可解决。

- 不要使用px，使用px会导致不同设备上有不同的效果。

- View动画是对View的影像做动画，View的真实位置没有变动，也就导致点击View动画后的位置触摸事件不会响应，属性动画不存在这个问题。

- 使用动画的过程中，使用硬件加速可以提高动画的流畅度。

- 动画在3.0以下的系统存在兼容性问题，特殊场景可能无法正常工作，需做好适配工作。

