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


##六.Android 5.X SVG矢量动画机制

Google在Android5.X中增加了对SVG矢量图形的支持

首先，什么是SVG：

- 可伸缩矢量图形
- 定义用于网络的基于矢量的图形
- 使用xml格式定义图形
- 图片在放大或者改变尺寸的情况下其图形质量不会有所损失
- 万维网联盟的标准
- 与诸多DOM和XSL之类的W3C标准是一个整体

SVG放大不会失真，而且bitmap需要不同分辨率适配，SVG不需要

###1.< path >标签
使用< path >标签来创建SVG，就是用指令的方式来控制一支画笔，列入，移动画笔来到某一个坐标位置，画一条线，画一条曲线，结束
< path >标签所支持的指令大致有一下几种:

- M = moveto(M X,Y):将画笔移动到指定的坐标位置，但未发生绘制

- L = lineto(L X,Y):画直线到指定的位置

- H = horizontal lineto(H X):画水平线到指定的X坐标位置

- V = vertical lineto(V Y):画垂直线到指定的Y坐标

- C = curveto(C,X1,Y1,X2,Y2,ENDX,ENDY):三次贝塞尔曲线

- S = smooth curveto(S X2,Y2,ENDX,ENDY):三次贝塞尔曲线

- Q = quadratic Belzier curve(Q X Y,ENDX,ENDY):二次贝塞尔曲线

- T = smooth quadratic Belzier curvrto(T,ENDX,ENDY):映射前面路径的重点

- A = elliptical Are(A RX,RY,XROTATION,FLAG1FLAG2,X,Y):弧线

- Z = closepath() 关闭路径

>使用上面的指令时，需要注意的几点

- 坐标轴以（0,0）位中心，X轴水平向右，Y轴水平向下
- 所有指令大小写均可，大写绝对定位，参照全局坐标系，小写相对定位，参照父容器坐标系
- 指令和数据间的空格可以无视
- 同一指令出现多次可以用一个

###2.SVG常见指令
####L
绘制直线的指令是“L”，代表从当前点绘制直线到给定点，“L”之后的参数是一个点坐标，如“L 200 400”绘制直线，同时，还可以使用“H”和“V”指令来绘制水平竖直线，后面的参数是x坐标，y坐标。
####M
M指令类似Android绘图中的path类moveto方法，即代表画笔移动到某一点，但并不发生绘图动作
####A
A指令是用来绘制一条弧线，且允许弧线不闭合，可以把A指令绘制的弧度想象成椭圆的某一段A指令一下有七个指令

- RX，RY指所有的椭圆的半轴大小
- XROTATION 指椭圆的X轴和水平方向顺时针方向的夹角，可以想象成一个水平的椭圆饶中心点顺时针旋转XROTATION 的角度
- FLAG1 只有两个值，1表示大角度弧度，0为小角度弧度
- FLAG2 只有两个值，确定从起点到终点的方向1顺时针，0逆时针
- X，Y为终点坐标

###3.SVG编辑器

网上有很多在线的编辑器，通过可视化编辑图像之后，点击view source可以转换成SVG代码

地址：http://editor.method.ac/

![这里写图片描述](http://img.blog.csdn.net/20160416230547587)

下载离线的SVG编辑器，例如：Inkscape，由很多强大的功能

###4.Android中使用SVG

Google在Android5.X后给我们提供了两个新的API来支持SVG

- VectorDrawable

- AnimatedVectorDrawable

其中，VectorDrawable可以创建基于XML的SVG图像，并且结合AnimatedVectorDrawable来实现动画效果

####1.VectorDrawable
>在XML中创建一个静态的SVG，通过这个结构

![这里写图片描述](http://img.blog.csdn.net/20160416231719716)

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100dp"
    android:viewportWidth="100dp">

</vector>
```

>这个代码之中包含了两组高宽，width,和height是表示SVG图像的具体大小，后面的是表示SVG图像划分的比例，后面再绘制path时所使用的参数，就是根据这两个值来进行转换的，比如上面的代码，将200dp划分100份，如果在绘图中使用坐标（50,50），则意味着该坐标为正中间，现在我们加上path标签

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100dp"
    android:viewportWidth="100dp">

    <group
        android:name="svg1"
        android:rotation="0">
        <path
            android:fillColor="@android:color/holo_blue_light"
            android:pathData="M 25 50 a 25,25 0 1,0 50,0" />

    </group>

</vector>
```

通过添加< path >标签绘制一个SVG齐总pathData就是图形所用到的指令了，先用M指令，将画笔移动到（25 ， 50）的位置，再通过A指令来绘制一个圆弧并且填充他，通过以上代码，就可以绘制一个SVG图形了



![这里写图片描述](http://img.blog.csdn.net/20160418204231456)

填充指令

![这里写图片描述](http://img.blog.csdn.net/20160418204901274)

####2.AnimatedVectorDrawable
AnimatedVectorDrawable的作用是给VectorDrawable提供动画效果，通过AnimatedVectorDrawable来连接静态的VectorDrawable和动态的objectAnimator

首先我们在xml中定义一个< animated-vector>，来申明对AnimatedVectorDrawable的使用，并且指明是作用在path或者group上

```
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/verctors">
    <target
        android:animation="@android:anim/fade_in"
        android:name="test">
    </target>

</animated-vector>
```

对应的vector即为静态的VectorDrawable

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="200dp"
    android:width="200dp"
    android:viewportWidth="100"
    android:viewportHeight="100">

    <group
        android:name="test"
        android:rotation="0"
        >

        <path
            android:strokeColor="@android:color/holo_blue_light"
            android:strokeWidth="2"
            android:pathData="M 25 50 a 25 , 25 0 1 , 0 50 ,0"
            >

        </path>
        
    </group>

</vector>
```

需要注意的是，AnimatedVectorDrawable中指明的target和name属性，必须与VectorDrawable中需要的name保持一致，这样系统能找到找到要实现的动画元素，最后，通过AnimatedVectorDrawable中的target和animation属性，将一个动画作用在对应的name上，objectanimator代码如下

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="4000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360">

</objectAnimator>
```

在< group>标签和< path>标签中添加rotation，fillColor，pathData属性，那么在objecyAnimator中，就可以通过指定，android:valueFrom=XXX，和android:property="XXX"属性，控制动画的起始值，唯一需要注意的是，如果指定属性为pathData，那么需要添加一个属性android:valueType-"pathType"来告诉系统进行pathData变换，类似的情况，可以使用rotation来进行旋转变换，使用fileColor实现变换颜色，使用pathData进行形状，位置的变换。

当所有的XML准备好之后，我们就可以直接给一个imageview设置背景了

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" 
        android:layout_gravity="center"
        android:src="@drawable/myver"/>

</LinearLayout>
```

在程序中，只要使用这个start方法开启动画就可以

###5.SVG动画实例
####1 线图动画

在android5.X之后，Google大量引入了线图动画，当页面发生改变的时候，页面的icon不再是生硬的切换，而是通过非常生动的动画，转换成另一种形态

要实现这样的一个效果，我们首先要创建一个SVG图形，即动态的VectorDrawable，要实现的效果就是上下两根线，然后他们形成一个X的效果

path1和path2分别绘制了两条直线

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">

    <group>
        <path
            android:name="path1"
            android:pathData="M 20,80 L 50,80 80 , 80"
            android:strokeColor="@android:color/holo_green_dark"
            android:strokeLineCap="round"
            android:strokeWidth="5" />

        <path
            android:name="path2"
            android:pathData="M 20,20 L 50,20 80 , 20"
            android:strokeColor="@android:color/holo_green_dark"
            android:strokeLineCap="round"
            android:strokeWidth="5" />

    </group>

</vector>
```

>path1和path2分别绘制了一条直线,

![这里写图片描述](http://img.blog.csdn.net/20160418210700953)

每条线都有三个点控制，接下来就是变换的动画了

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="5000"
    android:propertyName="pathData"
    android:valueFrom="M 20, 80 L 50 , 80 80 ,80"
    android:valueTo="M 20 ,80 L 50 ,50 80 ,80"
    android:valueType="pathType"
    android:interpolator="@android:anim/bounce_interpolator">

</objectAnimator>
```

在以上代码中，定义了一个pathType动画，并且指定了起点，中点，终点

```
android:valueFrom="M 20, 80 L 50 , 80 80 ,80"
android:valueTo="M 20 ,80 L 50 ,50 80 ,80"
```

这两个值是对应的起始点，需要注意的是，SVG的路径变换属性动画中，变换前后阶段属性必须相同，这也是前面需要使用的三个点看来绘制一条直线的原因，有了VectorDrawable和objectAnimator，现在只需要AnimatedVectorDrawable将他们加起来就起来

```
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/ver2">


    <target
        android:name="path1"
        android:animation="@anim/anim2" />


    <target
        android:name="path2"
        android:animation="@anim/anim2" />

</animated-vector>
```

最后只需要去启动动画就可以了

```
/**
 * 绘制SVG
 */
public class SVGActivity extends AppCompatActivity {

    private ImageView iv;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_svg);

        iv = (ImageView) findViewById(R.id.iv);
        iv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

            }
        });
    }

    private void anim() {
        Drawable drawable = iv.getDrawable();
        if (drawable instanceof Animatable) {
            ((Animatable) drawable).start();
        }
    }

}



```


####2 模拟三球仪
三球仪是天体文学中的星象仪器，用来模拟地球，月亮和太阳的运行轨迹，如图

![这里写图片描述](http://img.blog.csdn.net/20160418211632362)

实现例子：

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="200dp"
    android:height="200dp"
    android:viewportHeight="100"
    android:viewportWidth="100">

    <group
        android:name="sun"
        android:pivotX="60"
        android:pivotY="50"
        android:rotation="0">

        <path
            android:name="path_sun"
            android:fillColor="@android:color/holo_blue_light"
            android:pathData="M 50 ,50 a 10,10 0 1 , 0 20 , 0 a 10 ,10 0 1 , 0 -20 ,0" />

        <group
            android:name="earth"
            android:pivotX="75"
            android:pivotY="50"
            android:rotation="0">

            <path
                android:name="path_earth"
                android:fillColor="@android:color/holo_orange_dark"
                android:pathData="M 70 , 50 a 5 , 5 0  1 , 0 10 ,0 a 5 , 5 0 1,0 -10 ,0" />


            <group>
                <path
                    android:fillColor="@android:color/holo_green_dark"
                    android:pathData="M 90,50 m -5 0 a 4 , 4 0 1,0 8 0 a 4 , 4 0 1 , 0 - 8 , 0" />

            </group>

        </group>
    </group>

</vector>
```

![这里写图片描述](http://img.blog.csdn.net/20160418213000364)

>可以从代码冲发现，sun在这个group中，有一个earth的group，我们这里再定义一个动画

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="4000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360"
   >

</objectAnimator>
```

后面的跟前面的类似

####3.轨迹动画
android对SVG的支持带来了很多的特效，做一个搜索的放大镜效果吧，先定义好轨迹

```
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="160dp"
    android:height="30dp"
    android:viewportHeight="30"
    android:viewportWidth="160">

    <path
        android:name="search"
        android:pathData="M141 , 17 A9 ,9 0 1 , 1 ,142 , 16 L149 ,23"
        android:strokeAlpha="0.8"
        android:strokeColor="#ff3570be"
        android:strokeLineCap="square"
        android:strokeWidth="2" />

    <path
        android:name="bar"
        android:pathData="M0,23 L149 ,23"
        android:strokeAlpha="0.8"
        android:strokeColor="#ff3570be"
        android:strokeLineCap="square"
        android:strokeWidth="2"
        />


</vector>
```

![这里写图片描述](http://img.blog.csdn.net/20160418213839313)


##七.android动画特效

###1.卫星菜单

当点击小红点的时候，弹出菜单，并且带有一个缓冲的效果，这就是Google在MD中强调的动画过渡，要实现这个动画，其实是一个开始一个结束动画
```
 /**
     * 执行动画
     */
    private void statAnim(){
        ObjectAnimator animator0 = ObjectAnimator.ofFloat(iv0,"alpha",1F,0.5F);
        ObjectAnimator animator1 = ObjectAnimator.ofFloat(iv1,"translationY",200F);
        ObjectAnimator animator2 = ObjectAnimator.ofFloat(iv2,"translationX",200F);
        ObjectAnimator animator3 = ObjectAnimator.ofFloat(iv3,"translationY",-200F);
        ObjectAnimator animator4 = ObjectAnimator.ofFloat(iv4,"translationX",-200F);
        AnimatorSet set = new AnimatorSet();
        set.setInterpolator(new BounceInterpolator());
        set.playTogether(animator0,animator1,animator2,animator3,animator4);
        set.start();
        mFlag = false;
    }

```

>下面就是点击事件

```
iv0.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                   if (mFlag){
                       statAnim();
                   }else{
                       closeAnim();
                   }
            }
        });
```


###2.计时器动画

通过这个示例，了解一下ValueAnimator的效果，当用户点击后，数字不断增加，好的，我们开始

```
/**
 * 绘制SVG
 */
public class SVGActivity extends AppCompatActivity {

    private TextView tv;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_svg);

        tv = (TextView) findViewById(R.id.tv);
        tv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                tvTimer(tv);
            }
        });
    }

    private void  tvTimer(final  View view){
        ValueAnimator va = ValueAnimator.ofInt(0,100);
        va.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                ( (TextView)view).setText("$"+(Integer)animation.getAnimatedValue());
            }
        });
        va.setDuration(3000);
        va.start();
    }

}



```


###3.下拉展开动画

实现一个展开的动画，首先，XML是这样的

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/holo_blue_bright"
        android:gravity="center_vertical"
        android:onClick="llClick"
        android:orientation="horizontal">

        <ImageView
            android:id="@+id/app_icon"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:src="@mipmap/ic_launcher" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="5dp"
            android:gravity="left"
            android:text="Click me"
            android:textSize="30sp" />

    </LinearLayout>

    <LinearLayout
        android:id="@+id/hidden_view"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="@android:color/holo_orange_light"
        android:orientation="horizontal"
        android:visibility="gone">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:src="@mipmap/ic_launcher" />

        <TextView
            android:id="@+id/tv_hidden"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:gravity="center"
            android:text="I Am Hidden"
            android:textSize="20sp" />

    </LinearLayout>

</LinearLayout>
```


```
package com.lgl.animations;

import android.animation.ValueAnimator;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;

/**
 * 下拉展开动画
 * Created by LGL on 2016/4/18.
 */
public class AnimaActivity extends AppCompatActivity {

    private LinearLayout mHiddenView;
    private float mDensity;
    private int mHiddenViewMeasuredHeight;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_anima);

        mHiddenView = (LinearLayout) findViewById(R.id.hidden_view);
        //获取像素密度
        mDensity = getResources().getDisplayMetrics().density;
        //获取布局的高度
        mHiddenViewMeasuredHeight = (int) (mDensity*40+0.5);
    }

    public  void llClick(View view){
        if (mHiddenView.getVisibility() == View.GONE){
            animOpen(mHiddenView);
        }else{
            animClose(mHiddenView);
        }
    }

    private void animOpen(final  View view){
        view.setVisibility(View.VISIBLE);
        ValueAnimator va = createDropAnim(view,0,mHiddenViewMeasuredHeight);
        va.start();
    }


    private void animClose(final  View view){
        int origHeight = view.getHeight();
        ValueAnimator va = createDropAnim(view,origHeight,0);
        va.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                view.setVisibility(View.GONE);
            }
        });
        va.start();
    }


    private ValueAnimator createDropAnim(final  View view,int start,int end) {
        ValueAnimator va = ValueAnimator.ofInt(start, end);
        va.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                int value = (int) animation.getAnimatedValue();
                ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
                layoutParams.height = value;
                view.setLayoutParams(layoutParams);
            }
        });
        return  va;
    }

}

```