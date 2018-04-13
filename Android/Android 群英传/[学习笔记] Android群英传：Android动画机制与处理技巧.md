# [学习笔记] Android群英传：Android动画机制与处理技巧

学习的主要内容有：

- Android视图动画‘
- Android属性动画
- Android动画实例

##一.Android View动画框架
Animation动画框架定义了透明度，旋转，缩放个移动等几种动画，而且控制了整个的View，实现原理是每次绘制视图的时候View所在的ViewGroup中drawChild函数获取该View的Animation的Transformation值，然后调用了canvas.concat方法，通过矩阵运算完成动画帧，如果动画没有完成则继续调用invalidate（）方法启动下回绘制来驱动动画，从而完成整个动画的绘制。

视图动画提供了AlphaAnimation,RotateAnimatio，TranslateAnimation，ScaleAnimation四种动画方式,并提供了Animationset动画集合，混合使用多种动画。相比属性动画,视图动画的缺陷就是不具备交互性,当某个元件发生视图动画后，其响应事件的位置还依然在动画前的地方,所以视图动画只能做普通的显示效果,避免交互的发生，但是它的优点也非常明显,即效率比较高且使用方便。

###1.透明动画


```
AlphaAnimation al = new AlphaAnimation(0,1);
al.setDuration(2000);
alpha.startAnimation(al);
```

###2.旋转动画

```
 RotateAnimation ro = new RotateAnimation(0,300,100,100);
 ro.setDuration(2000);
 rotate.setAnimation(ro);
```

###3.平移动画

```
TranslateAnimation tr = new TranslateAnimation(0,200,0,300);
tr.setDuration(2000);
translate.setAnimation(tr);
```

###4.缩放动画

```
 ScaleAnimation sc = new ScaleAnimation(0,2,0,2);
 sc.setDuration(2000);
 scale.setAnimation(sc);
```

###5.动画集合

```
AnimationSet setAnimation = new AnimationSet(true);
setAnimation.setDuration(2000);

AlphaAnimation als = new AlphaAnimation(0,1);
als.setDuration(2000);
setAnimation.addAnimation(als);

RotateAnimation ros = new RotateAnimation(0,300,100,100);
ros.setDuration(2000);
setAnimation.addAnimation(ros);

set.startAnimation(setAnimation);
```


动画的监听：

```
AlphaAnimation al = new AlphaAnimation(0,1);
                al.setDuration(2000);
                alpha.startAnimation(al);

                al.setAnimationListener(new Animation.AnimationListener() {
                    @Override
                    public void onAnimationStart(Animation animation) {
                        Log.i("Animation","开始");
                    }

                    @Override
                    public void onAnimationEnd(Animation animation) {
                        Log.i("Animation","结束");
                        Toast.makeText(MainActivity.this,"动画结束",Toast.LENGTH_LONG).show();
                    }

                    @Override
                    public void onAnimationRepeat(Animation animation) {

                    }
                });

```


##二.Android属性动画分析
属性动画在Animator框架里的，用的最多的也就是AnimatorSet和ObjectAnimator配合，使用ObjectAnimator进行更精细化的控制，只控制一个对象的属性，而使用多个ObjectAnimator组合到AnimatorSet形成一个动画。

###1.ObjectAnimator
ObjectAnimator是属性动画框架中最重要的实行类,创建一个ObjectAnimator只需通过他
的静态工厂类直接返回一个ObjectAnimator对象，参数包括一个对象和对象的属性名字,但这
个属性必须有get和set函数，内部会通过Java反射机制来调用set函数修改对象属性值.同样
你也可以调用setIn设置相应的差值器。 


简单的平移动画的实现：

```
 ObjectAnimator ob = ObjectAnimator.ofFloat(view,"translationX",300);
                ob.setDuration(2000);
                ob.start();
```

通过ObjectAnimator的静态工厂方法，创建个ObjectAnimator对象，第一个参数自然是要操纵的view,第二个参数则是要操纵的属性 而最后个参数是一个可变数组参数，需要传递进去该属性变化的一个取值过程。

>在使用ObjectAnimator的时候，有一点是非常重要的，就是操纵的set，get方法，不然ObjectAnimator是无效的

- translationX和 translationY:这两个属性作为一种增量来控制着View对象从它布局容器的左上角坐标平移的位置。

- rotation、rotationX和rotationY:这三个属性控制View对象围绕支点进行2D和3D旋转

- scaleX和scaleY. 这两个属性控制着View对象围绕它的支点进行2D缩放。

- pivotX和pivotY:这两个属性控制着view对象的支点位置,围绕这个支点进行旋转和缩放变换处理，默认情况下,该支点的位置就是View对象的中心点。

- x和y这是两个简单实用的属性，它描述了View对象在它的容器中的最终位置，它是最初的左上角坐标和 translationX和 translationY值的累计和

- alpha:它表示View对象的alpha透明度，默认值是1(不透明),0代表完全透明(不可见)

一个属性没有get，set方法，google在应用层提供了两种方案来解决这个问题：

  - 通过自定义一个属性类或者包装类,来间接地给这个属性增加get方法
  - 通过ValusAnimator来实现

使用包装类的方法给一个属性增加set，get方法：

```
 private static class WrapperView {

        private View mTarget;

        public WrapperView(View target) {
            mTarget = target;
        }

        public int getWidth() {
            return mTarget.getLayoutParams().width;
        }

        public void setWidth(int width) {
            mTarget.getLayoutParams().width = width;
            mTarget.requestLayout();
        }
    }
```

通过上面的代码就给一个属性包装上了一层，并且提供set，get方法

```
WrapperView vi = new WrapperView(alpha);
ObjectAnimator.ofInt(vi,"width",500).setDuration(2000).start();
```

###2.PropertyValuesHolder
类似视图动画中的AnimationSet，就是把动画给组合起来，在属性动画中，如果针对一个对象的多个属性，就同时需要多个动画了，可以使用PropertyValuesHolder来实现，比如需要在平移的过程中，同时改变x,y的缩放，可以这样实现

```
PropertyValuesHolder pvh1 = PropertyValuesHolder.ofFloat("translationX",300f);
PropertyValuesHolder pvh2 = PropertyValuesHolder.ofFloat("scaleX",1f,0,1f);
PropertyValuesHolder pvh3 = PropertyValuesHolder.ofFloat("scaleY",1f,0,1f);        ObjectAnimator.ofPropertyValuesHolder(alpha,pvh1,pvh2,pvh3).setDuration(2000).start();

```

###3.ValueAnimator
ValueAnimator是属性动画的核心所在，ObjectAnimator也是继承自它：

```
public final class ObjectAnimator extends ValueAnimator

```

>ValueAnimator本身不提供任何动画，像是一个数值发生器，用来产生一定具有规律的数字，从而让调用者控制动画的整个过程，我们举个例子来说明

```
ValueAnimator va = ValueAnimator.ofFloat(0,100);
                va.setTarget(value);
                va.setDuration(2000).start();
                va.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                    @Override
                    public void onAnimationUpdate(ValueAnimator animation) {
                        float values = (float) animation.getAnimatedValue();
                        Log.i("数值",values+"");
                    }
                });

```

###4.动画事件的监听

一个完整的动画是具有，start，repeat，end，cancel四个过程的：

```
 ob.addListener(new Animator.AnimatorListener() {
                    @Override
                    public void onAnimationStart(Animator animation) {
                        
                    }

                    @Override
                    public void onAnimationEnd(Animator animation) {

                    }

                    @Override
                    public void onAnimationCancel(Animator animation) {

                    }

                    @Override
                    public void onAnimationRepeat(Animator animation) {

                    }
                });
```

只关心动画结束，Android提供了一个AnimatorLisistenerAdapter来让你自己选择监听事件

```
va.addListener(new AnimatorListenerAdapter() {
                    @Override
                    public void onAnimationEnd(Animator animation) {
                        super.onAnimationEnd(animation);
                    }
                });
```


###5.AnimatorSet
AnimatorSet能更精准的控制顺序，同样是实现PropertyValuesHolder的动画，AnimatorSet是这样实现的

```
		ObjectAnimator animator1 = ObjectAnimator.ofFloat(alpha, "translationX", 300f);
        ObjectAnimator animator2 = ObjectAnimator.ofFloat(alpha, "scaleX", 1f, 0, 1f);
        ObjectAnimator animator3 = ObjectAnimator.ofFloat(alpha, "scaleY", 1f, 0, 1f);
        AnimatorSet set = new AnimatorSet();
        set.setDuration(2000);
        set.playTogether(animator1,animator2,animator3);
        set.start();
```

>在属性动画中，AnimatorSet正是通过playTogether()、playSequentially()、animSet.Play().with()、before()、after()等方法控制多个动画协同工作，从而控制播放顺序的

###6.在XML中定义动画

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000"
    android:propertyName="scaleX"
    android:valueFrom="1.0"
    android:valueTo="2.0"
    android:valueType="floatType">

</objectAnimator>
```

在代码中引用

```

    /**
     * 引用xml动画
     * @param v
     */
    private void scaleX(View v){
        Animator anim = AnimatorInflater
                .loadAnimator(this,R.animator.animator);
        anim.setTarget(v);
        anim.start();
    }
```

###7.View的animate方法

在Android3.0，Google给view增加了animate方法直接来驱动属性动画，代码如下，其实animate就是属性动画的一种缩写

```
 view.animate().alpha(0).y(300).setDuration(2000).withStartAction(new Runnable() {
                    @Override
                    public void run() {

                    }
                }).withEndAction(new Runnable() {
                    @Override
                    public void run() {
                        runOnUiThread(new Runnable() {
                            @Override
                            public void run() {

                            }
                        });
                    }
                }).start();
```

##三.Android布局动画

布局动画，就是作用在ViewGruop中给View添加的过渡效果，最简单的方法是在xml中打开

```
 android:animateLayoutChanges="true"
```

可以通过LayoutAnimationController来定义

```
		ll = (LinearLayout) findViewById(R.id.ll);
        //设置过渡动画
        ScaleAnimation sa = new ScaleAnimation(0, 1, 0, 1);
        sa.setDuration(2000);
        LayoutAnimationController lc = new LayoutAnimationController(sa, 0.5f);
        lc.setOrder(LayoutAnimationController.ORDER_NORMAL);
        //设置布局动画
        ll.setLayoutAnimation(lc);
```

- LayoutAnimationController.ORDER_NORMAL 顺序
- LayoutAnimationController.ORDER_RANDOM 随机
- LayoutAnimationController.ORDER_REVEESE 反序


##四.Interpolators(插值器)
通过插值器可以定义动画变换速率，类似物理中的加速度，其作用主要是目标变化对应的变化，同样的一个动画变换起始值，在不同的插值器的作用下，每个单位时间内所达到的变换值都是不一样的

 [Interpolator插值器详解][1]

##五.自定义动画
>创建自定义动画很简单，只需要实现applyTransformation的逻辑就可以，不过通常情况下，我们还要覆盖父类的initialize方法来完成一些初始化工作，

```
@Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
        super.applyTransformation(interpolatedTime, t);
    }
```

第一个参数interpolatedTime是前面说的插值器的时间因子，这个因子是由动画当前完成的百分比和当前时间对应的差值计算的，取值范围在0-1.0，第二个参数是矩阵的封装类，一般使用这个类获取当前的矩阵对象，代码如下

```
 Matrix matrix = t.getMatrix();
```

通过改变获得的matrix 对象，可以将动画效果实现，而对于matrix的变换操作，基本上可以实现任何效果

实现一个电视机关闭的效果：

```
 @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {

        Matrix matrix = t.getMatrix();
        matrix.preScale(1, 1 - interpolatedTime, width,height);
        super.applyTransformation(interpolatedTime, t);
    }
```

完整代码：
```
public class CustomTV extends Animation {

    private int mCenterWidth;
    private int mCenterHeight;
    private Camera mCamera = new Camera();
    private float mRotateY = 0.0f;

    @Override
    public void initialize(int width,
                           int height,
                           int parentWidth,
                           int parentHeight) {

        super.initialize(width, height, parentWidth, parentHeight);
        // 设置默认时长
        setDuration(1000);
        // 动画结束后保留状态
        setFillAfter(true);
        // 设置默认插值器
        setInterpolator(new AccelerateInterpolator());
        mCenterWidth = width / 2;
        mCenterHeight = height / 2;
    }

    // 暴露接口-设置旋转角度
    public void setRotateY(float rorateY) {
        mRotateY = rorateY;
    }

    @Override
    protected void applyTransformation(
            float interpolatedTime,
            Transformation t) {
        final Matrix matrix = t.getMatrix();
        matrix.preScale(1,
                1 - interpolatedTime,
                mCenterWidth,
                mCenterHeight);
    }
}

```
结合矩阵，并且使用Canmera来实现一个3D的效果，要注意的是，这里所指的Camera不是相机，而这个类封装了 openGl的3D动画，我们继续用代码来实现

```
  @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {

        super.initialize(width, height, parentWidth, parentHeight);
        setDuration(2000);
        setFillAfter(true);
        setInterpolator(new BounceInterpolator());
        w = width / 2;
        h = height / 2;

    }

    //暴露接口-设置旋转角度
    public void setRotateY(float rotateY) {
        mRotateY = rotateY;
    }

    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {

        Matrix matrix = t.getMatrix();
	    //matrix.preScale(1, 1 - interpolatedTime, w,h);
        mCamera.save();
        //设置旋转角度
        mCamera.rotate(0, 180, 360);
        mCamera.restore();
        //通过pre方法设置矩形作用前的偏移量来改变旋转中心
        matrix.preTranslate(w, h);
        matrix.postTranslate(-w, -h);

        super.applyTransformation(interpolatedTime, t);

    }
```
