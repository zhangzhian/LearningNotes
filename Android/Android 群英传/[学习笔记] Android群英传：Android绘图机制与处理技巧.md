# [学习笔记] Android群英传：Android绘图机制与处理技巧
---
包含内容：
- Android屏幕相关知识
- Android绘图技巧
- Android图像处理技巧
- SurfaceView的使用

##一.屏幕的尺寸信息

###1.屏幕参数
屏幕通常具备以下参数

- 屏幕大小

>指屏幕对角线的长度，通常用寸来表示，例如4.7寸，5.5寸

- 分辨率

>分辨率是指实际屏幕的像素点个数，例如720X1280就是指屏幕的分辨率，宽有720个像素点，高有1280个像素点

- PPI

>每英寸像素又称为DPI，由对角线的的像素点数除以屏幕的大小所得

###2.系统屏幕密度
系统定义了几个标准的DPI

![这里写图片描述](http://img.blog.csdn.net/20160227221717064)

###3.独立像素密度dp
各种屏幕密度的不同,导致同样像素大小的长度，在不同密度的屏幕上显示长度不同。相同长度的屏幕，高密度的屏幕包含更多的像素点。在安卓系统中使用mdpi密度值为160的屏幕作为标准，在这个屏幕上，1px = 1dp，其他屏幕则可以通过比例进行换算，例如同样是100dp的长度，mdpi中为100px，而在hdpi中为150，我们也可以得出在各个密度值中的换算公式，在mdpi中 1dp = 1px, 在hdpi中， 1dp = 1.5px，在xhdpi中，1dp = 2px,在xxhdpi中1dp = 3px。
> 换算公式 l:m:h:xh:xxh = 3:4:6:8:12

###4.单位换算

```java
package com.lgl.playview;

import android.content.Context;


/**
 * dp，sp转换成px的工具类
 */
public class DisplayUtils {

    /**
     * 将px值转换成dpi或者dp值，保持尺寸不变
     *
     * @param content
     * @param pxValus
     * @return
     */
    public static int px2dip(Context content, float pxValus) {
        final float scale = content.getResources().getDisplayMetrics().density;
        return (int) (pxValus / scale + 0.5f);
    }

    /**
     * 将dip和dp转化成px,保证尺寸大小不变。
     *
     * @param content
     * @param pxValus
     * @return
     */
    public static int dip2px(Context content, float pxValus) {
        final float scale = content.getResources().getDisplayMetrics().density;
        return (int) (pxValus / scale + 0.5f);
    }

    /**
     * 将px转化成sp,保证文字大小不变。
     *
     * @param content
     * @param pxValus
     * @return
     */
    public static int px2sp(Context content, float pxValus) {
        final float fontScale = content.getResources().getDisplayMetrics().scaledDensity;
        return (int) (pxValus / fontScale + 0.5f);
    }

    /**
     * 将sp转化成px,保证文字大小不变。
     *
     * @param content
     * @param pxValus
     * @return
     */
    public static int sp2px(Context content, float pxValus) {
        final float fontScale = content.getResources().getDisplayMetrics().scaledDensity;
        return (int) (pxValus / fontScale + 0.5f);
    }
}

    /**
     * dp2px
     * @param dp
     * @return
     */
    protected int dp2px(int dp){
        return (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,dp,getResources().getDisplayMetrics());
    }

    /**
     * sp2px
     * @param dp
     * @return
     */
    protected int sp2px(int sp){
        return (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,sp,getResources().getDisplayMetrics());
    }
```

##二.2D绘图基础
系统通过提供的Canvas对象来提供绘图方法，它提供了各种绘制图像的API,drawLine,deawPoint,drawRect,drawVertices,drawAce,drawCircle等等

Paint:
```java
   setAntiAlias();            //设置画笔的锯齿效果

　　setColor();                //设置画笔的颜色

　　setARGB();                 //设置画笔的A、R、G、B值

　　setAlpha();                //设置画笔的Alpha值

　　setTextSize();             //设置字体的尺寸

　　setStyle();                //设置画笔的风格（空心或实心）

　　setStrokeWidth();          //设置空心边框的宽度

　　getColor();                //获取画笔的颜色

```


```java

public class RectView extends View {

    private Paint paint1,paint2;

    public RectView(Context context, AttributeSet attrs) {
        super(context, attrs);

        initView();
    }

    private void initView() {

        paint1 = new Paint();
        paint1.setColor(Color.BLUE);
        paint1.setAntiAlias(true);
        //空心
        paint1.setStyle(Paint.Style.STROKE);

        paint2 = new Paint();
        paint2.setColor(Color.BLUE);
        //实心
        paint2.setStyle(Paint.Style.FILL);

    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        canvas.drawRect(50,100,300,300,paint1);
        canvas.drawRect(350,400,700,700,paint2);
    }
}


```

##三.Android XML 绘图

###1.Bitmap

```xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher">

</bitmap>
```

通过这样引用图片就可以将图片直接转化成Bitmap让我们在程序中使用

###2.Shape

通过Shape可以绘制各种图形，下面展示一下shape的参数

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"

    android:shape="rectangle">
    <!--默认是rectangle-->

    <!--当shape= rectangle的时候使用-->
    <corners
        android:bottomLeftRadius="1dp"
        android:bottomRightRadius="1dp"
        android:radius="1dp"
        android:topLeftRadius="1dp"
        android:topRightRadius="1dp" />
    <!--半径，会被后面的单个半径属性覆盖，默认是1dp-->

    <!--渐变-->
    <gradient
        android:angle="1dp"
        android:centerColor="@color/colorAccent"
        android:centerX="1dp"
        android:centerY="1dp"
        android:gradientRadius="1dp"
        android:startColor="@color/colorAccent"
        android:type="linear"
        android:useLevel="true" />

    <!--内间距-->
    <padding
        android:bottom="1dp"
        android:left="1dp"
        android:right="1dp"
        android:top="1dp" />

    <!--大小，主要用于imageview用于scaletype-->
    <size
        android:width="1dp"
        android:height="1dp" />

    <!--填充颜色-->
    <solid android:color="@color/colorAccent" />

    <!--指定边框-->
  	<!--虚线间隔宽度-->
		<!--虚线宽度-->
    <stroke
        android:width="1dp"
        android:color="@color/colorAccent"
    android:dashWidth= "1dp" 
    android:dashGap= "1dp" />

</shape>

```

>shape可以说是xml绘图的精华所在，而且功能十分的强大，无论是扁平化，拟物化还是渐变，都是十分的OK，我们现在来做一个阴影的效果

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <gradient
        android:angle="45"
        android:endColor="#805FBBFF"
        android:startColor="#FF5DA2FF" />

    <padding
        android:bottom="7dp"
        android:left="7dp"
        android:right="7dp"
        android:top="7dp" />

    <corners android:radius="8dp" />

</shape>

```

###3.Layer
>Layer是在PhotoShop中是非常常用的功能，在Android中，我们同样可以实现图层的效果

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--图片1-->
    <item android:drawable="@mipmap/ic_launcher"/>

    <!--图片2-->
    <item
        android:bottom="10dp"
        android:top="10dp"
        android:right="10dp"
        android:left="10dp"
        android:drawable="@mipmap/ic_launcher"
        />

</layer-list>
```

###4.Selector
> Selector的作用是帮助开发者实现静态View的反馈，通过设置不同的属性呈现不同的效果

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 默认时候的背景-->
    <item android:drawable="@mipmap/ic_launcher" />

    <!-- 没有焦点时候的背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_window_focused="false" />

    <!-- 非触摸模式下获得焦点并点击时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_pressed="true" android:state_window_focused="true" />

    <!-- 触摸模式下获得焦点并点击时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_focused="false" android:state_pressed="true" />

    <!--选中时的图片背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_selected="true" />

    <!--获得焦点时的图片背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_focused="true" />
    
</selector>

```

下面这个例子就展示了在一个selector中使用shape作为他的item的例子，实现一个具体点击反馈效果的，圆角矩形的selector，代码如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <shape android:shape="rectangle">
            <!--填充颜色-->
            <solid android:color="#33444444" />
            <!--设置按钮的四个角为弧形-->
            <corners android:radius="5dp" />
            <!--间距-->
            <padding android:bottom="10dp" android:left="10dp" android:right="10dp" android:top="10dp" />
        </shape>
    </item>

    <item>
        <shape android:shape="rectangle">
            <!--填充颜色-->
            <solid android:color="#FFFFFF" />
            <!--设置按钮的四个角为弧形-->
            <corners android:radius="5dp" />
            <!--间距-->
            <padding android:bottom="10dp" android:left="10dp" android:right="10dp" android:top="10dp" />
        </shape>
    </item>
</selector>

```

##四.Android绘图技巧
>在学完Android的基本绘图之后我们来讲解一下常用的绘图技巧

###1.Canvas
Canvas作为绘制图形的直接对象，提供了一下几个非常有用的方法

- Canvas.save()
- Canvas.restore()
- Canvas.translate()
- Canvas.roate()

Canvas.save()这个方法，从字面上的意思可以理解为保存画布，作用就是讲之前的图像保存起来，让后续的操作能像在新的画布一样操作，这跟PS的图层基本差不多
Canvas.restore()这个方法，则可以理解为合并图层，就是讲之前保存下来的东西合并
Canvas.translate()画布平移，坐标系平移，调用translate（x,y）之后，则将原点（0,0）移动到（x,y）之后的所有绘图都是在这一点上执行的
Canvas.roate()画布旋转，坐标系旋转

**例子：画一个表盘：**
![这里写图片描述](http://img.blog.csdn.net/20180410153644816?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
将表盘分解

- 1.仪表盘——外面的大圆盘
- 2.刻度线——包含四个长的刻度线和其他短的刻度线
- 3.刻度值——包含长刻度线对应的大的刻度尺和其他小的刻度尺
- 4.指针——中间的指针，一粗一细两根

第一步，先画表盘。关键在于确定圆心和半径，这里直接居中


```java
// 画外圆
Paint paintCircle = new Paint();
paintCircle.setAntiAlias(true);
paintCircle.setStyle(Paint.Style.STROKE);
paintCircle.setStrokeWidth(5);
canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth / 2, paintCircle);
```

![这里写图片描述](http://img.blog.csdn.net/20180410153625384?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

画刻度尺，通过旋转画布——实际上是旋转了画图的坐标轴来绘绘制刻度

```java
	// 画刻度
		Paint paintDegree = new Paint();
		paintDegree.setStrokeWidth(3);
		for (int i = 0; i < 24; i++) {
			// 区别整点和非整点
			if (i == 0 || i == 6 || i == 12 || i == 18) {
				paintDegree.setStrokeWidth(5);
				paintDegree.setTextSize(30);
				canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2, mWidth,
						mHeight / 2 - mWidth / 2 + 60, paintDegree);
				String degree = String.valueOf(i);
				canvas.drawText(degree,
						mWidth / 2 - paintDegree.measureText(degree) / 2,
						mHeight / 2 - mWidth / 2 + 90, paintDegree);
			} else {
				paintDegree.setStrokeWidth(3);
				paintDegree.setTextSize(15);
				canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2, mWidth,
						mHeight / 2 - mWidth / 2 + 30, paintDegree);
				String degree = String.valueOf(i);
				canvas.drawText(degree,
						mWidth / 2 - paintDegree.measureText(degree) / 2,
						mHeight / 2 - mWidth / 2 + 60, paintDegree);
			}
			// 通过旋转画布简化坐标运算
			canvas.rotate(15, mWidth / 2, mHeight / 2);
		}
```

![这里写图片描述](http://img.blog.csdn.net/20180410153636725?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

绘制两根指针

```java
		// 画指针
		Paint paintHour = new Paint();
		paintHour.setStrokeWidth(20);
		Paint paintMinute = new Paint();
		paintMinute.setStrokeWidth(10);
		canvas.save();
		canvas.translate(mWidth / 2, mHeight / 2);
		canvas.drawLine(0, 0, 100, 100, paintHour);
		canvas.drawLine(0, 0, 100, 200, paintMinute);
		canvas.restore();
```

完整的代码：

```java

public class DialView extends View {

	// 宽高
	private int mWidth;
	private int mHeight;

	// 构造方法
	public DialView(Context context, AttributeSet attrs) {
		super(context, attrs);
		// 获取屏幕的宽高
		WindowManager wm = (WindowManager) getContext().getSystemService(
				Context.WINDOW_SERVICE);
		mWidth = wm.getDefaultDisplay().getWidth();
		mHeight = wm.getDefaultDisplay().getHeight();
	}

	@Override
	protected void onDraw(Canvas canvas) {
		// TODO Auto-generated method stub
		super.onDraw(canvas);

		// 画外圆
		Paint paintCircle = new Paint();
		paintCircle.setAntiAlias(true);
		paintCircle.setStyle(Paint.Style.STROKE);
		paintCircle.setStrokeWidth(5);
		canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth / 2, paintCircle);

		// 画刻度
		Paint paintDegree = new Paint();
		paintDegree.setStrokeWidth(3);
		for (int i = 0; i < 24; i++) {
			// 区别整点和非整点
			if (i == 0 || i == 6 || i == 12 || i == 18) {
				paintDegree.setStrokeWidth(5);
				paintDegree.setTextSize(30);
				canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2,
						mWidth / 2, mHeight / 2 - mWidth / 2 + 60, paintDegree);
				String degree = String.valueOf(i);
				canvas.drawText(degree,
						mWidth / 2 - paintDegree.measureText(degree) / 2,
						mHeight / 2 - mWidth / 2 + 90, paintDegree);
			} else {
				paintDegree.setStrokeWidth(3);
				paintDegree.setTextSize(15);
				canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2,
						mWidth / 2, mHeight / 2 - mWidth / 2 + 30, paintDegree);
				String degree = String.valueOf(i);
				canvas.drawText(degree,
						mWidth / 2 - paintDegree.measureText(degree) / 2,
						mHeight / 2 - mWidth / 2 + 60, paintDegree);
			}
			// 通过旋转画布简化坐标运算
			canvas.rotate(15, mWidth / 2, mHeight / 2);
		}

		// 画指针
		Paint paintHour = new Paint();
		paintHour.setStrokeWidth(20);
		Paint paintMinute = new Paint();
		paintMinute.setStrokeWidth(10);
		canvas.save();
		canvas.translate(mWidth / 2, mHeight / 2);
		canvas.drawLine(0, 0, 100, 100, paintHour);
		canvas.drawLine(0, 0, 100, 200, paintMinute);
		canvas.restore();
	}

}

```

###2.Layer图层

![这里写图片描述](http://img.blog.csdn.net/20160327215159839)

Android通过saveLayer()方法，saveLayerAlpha()将一个图层入栈
使用restore(）方法，restoreToCount（）方法将一个图层出栈
入栈的时候，后面的所有才做都是发生在这个图层上的，而出栈的时候，则会把图层绘制在上层Canvas上

```java
@Override
	protected void onDraw(Canvas canvas) {
		// TODO Auto-generated method stub
		super.onDraw(canvas);
		canvas.drawColor(Color.WHITE);
		mPaint.setColor(Color.BLUE);
		canvas.drawCircle(150, 150, 100, mPaint);
		
		canvas.saveLayerAlpha(0, 0,400,400,127,LAYER_TYPE_NONE);
		mPaint.setColor(Color.RED);
		canvas.drawCircle(200, 200, 100, mPaint);
		canvas.restore();
	}
```

当绘制两个相交的圆时，就是图层
接下来将图层后面的透明度设置成127 255 0三个


![这里写图片描述](http://img.blog.csdn.net/20160327221258101)

##五.Android图像处理之色彩特效处理
Android对于图片的处理，最常用的数据结构是位图——Bitmap，包含了一张图片的所有数据，整个图片都是由点阵和颜色值去组成的，所谓的点阵就是一个包含像素的矩形，每一个元素对应的图片就是一个像素，而颜色值——ARGB，分别对应透明度红，绿，蓝，这四个通用的分量，他们共同决定了每个像素点显示的颜色

![这里写图片描述](http://img.blog.csdn.net/20160327221742455)

###1.色彩矩阵分析
在色彩处理中，通常用三个角度描述一张图片

- 色调——物体传播的颜色
- 饱和度——颜色的纯度，从0-100来进行描述
- 亮度——颜色的相对，明暗程度

在Android中，系统会使用一个颜色矩阵——ColorMatrix,来处理这些色彩的效果，Android中的颜色矩阵是4X5的数字矩阵，用来对颜色色彩进行处理，而对于每一个像素点，都有一个颜色分量矩阵来保存ARGB值

![这里写图片描述](http://img.blog.csdn.net/20160327222645743)

图中矩阵A就是一个4X5的颜色矩阵，在Android中，他们会以一段一维数组的形式来存储[a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t],则C就是一个颜色矩形分量，在处理图像中，使用矩阵乘法运算AC处理颜色分量矩阵

计算过程如下

![这里写图片描述](http://img.blog.csdn.net/20160327223840962)


- 第一行的abcde用来决定新的颜色值R——红色
- 第二行的fghij用来决定新的颜色值G——绿色
- 第三行的kimno用来决定新的颜色值B——蓝色
- 第四行的pqrst用来决定新的颜色值A——透明



矩阵变换的计算公式，以R分量为例，计算过程是

```java
R1 = a * R + b* G + c*B+d *A + e

```

如果让a = 1,b,c,d,e都等于0，那么计算的结果为R1 = R，因此我们可以构建一个矩阵

![这里写图片描述](http://img.blog.csdn.net/20160327231540399)

这个矩阵通常是用来作为初始的颜色矩阵来使用，不会对原有颜色进行任何改变

要变换颜色值的时候通常有两种方法：

- 直接改变颜色的offset，即修改颜色的分量
- 直接改变RGBA值的系数来改变颜色分量的值


####1.改变偏移量
修改R1的值，只要将第五列的值进行修改即可，即改变颜色的偏移量，其它值保存初始矩阵的值

![这里写图片描述](http://img.blog.csdn.net/20160327232024735)

在上面这个矩阵中，修改了R,G所对应的颜色偏移量，那么最后的处理结果，就是图像的红色绿色分别增加了100，红色混合绿色的到了黄色，所以最终的颜色处理结果就是让整个图片的色调偏黄色

####2.改变颜色系数

修改颜色分量中的某个系数值而其他只依然保存初始矩阵的值。

![这里写图片描述](http://img.blog.csdn.net/20160327232146096)

在上面这个矩阵中改变了G分量所对应的系数据g，这样在矩形运算后G分量会变成以前的两倍，最终效果就是图像的色调更加偏绿。

####3.改变色光属性
图像的色调，饱和度，亮度这三个属性在图像处理中使用的非常多，因此在颜色矩阵中也封装了一些API来快速调用这些参数

在Android中，系统封装了一个类——ColorMatrix，也就是颜色矩阵。通过这个类，可以很方便地通过改变矩阵值来处理颜色的效果。
创建一个ColorMatrix对象非常简单代码如下：

```java
ColorMatrix colorMatrix = new ColorMatrix();
```

处理不同的色光属性：

- 色调

setRotate()设置三个颜色的色调，第一个参数系统分别使用0，1，2来代表red green blue三个颜色的处理，第二个参数就是需要处理的值了

```java
		ColorMatrix colorMatrix = new ColorMatrix();
		colorMatrix.setRotate(0, 2);
		colorMatrix.setRotate(1, 4);
		colorMatrix.setRotate(2, 3);
```

- 饱和度

setSaturation（）来设置颜色的饱和度，参数即代表饱和度的值，当饱和度为0的时候是灰色的

```java
    ColorMatrix colorMatrix = new ColorMatrix();
    colorMatrix.setSaturation(10);
```

- 亮度

当三原色以相同的比例及及混合的时候，就会显示出白色，系统使用这个原理来改变一个图像的亮度,代码如下，当亮度为零时图像会变成全黑。

```java
ColorMatrix colorMatrix = new ColorMatrix();
colorMatrix.setScale(10, 10, 10, 10);
```

PostConcat方法来将矩阵的作用效果混合，从而叠加处理效果代码如下：

```java
		ColorMatrix colorMatrix = new ColorMatrix();
		colorMatrix.postConcat(colorMatrix);
		colorMatrix.postConcat(colorMatrix);
		colorMatrix.postConcat(colorMatrix);
```


例子中，我们通过seekBar来改变不同的数值，并将这些数值作用到对应的矩阵中，最后通过PostConcat方法混合处理

代码如下

```java
public class PrimaryColor extends Activity implements SeekBar.OnSeekBarChangeListener {

    private static int MAX_VALUE = 255;
    private static int MID_VALUE = 127;
    private ImageView mImageView;
    private SeekBar mSeekbarhue, mSeekbarSaturation, mSeekbarLum;
    private float mHue, mStauration, mLum;
    private Bitmap bitmap;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.primary_color);
        bitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test3);
        mImageView = (ImageView) findViewById(R.id.imageview);
        mSeekbarhue = (SeekBar) findViewById(R.id.seekbarHue);
        mSeekbarSaturation = (SeekBar) findViewById(R.id.seekbarSaturation);
        mSeekbarLum = (SeekBar) findViewById(R.id.seekbatLum);
        mSeekbarhue.setOnSeekBarChangeListener(this);
        mSeekbarSaturation.setOnSeekBarChangeListener(this);
        mSeekbarLum.setOnSeekBarChangeListener(this);
        mSeekbarhue.setMax(MAX_VALUE);
        mSeekbarSaturation.setMax(MAX_VALUE);
        mSeekbarLum.setMax(MAX_VALUE);
        mSeekbarhue.setProgress(MID_VALUE);
        mSeekbarSaturation.setProgress(MID_VALUE);
        mSeekbarLum.setProgress(MID_VALUE);
        mImageView.setImageBitmap(bitmap);
    }

    @Override
    public void onProgressChanged(SeekBar seekBar,
                                  int progress, boolean fromUser) {
        switch (seekBar.getId()) {
            case R.id.seekbarHue:
                mHue = (progress - MID_VALUE) * 1.0F / MID_VALUE * 180;
                break;
            case R.id.seekbarSaturation:
                mStauration = progress * 1.0F / MID_VALUE;
                break;
            case R.id.seekbatLum:
                mLum = progress * 1.0F / MID_VALUE;
                break;
        }
        mImageView.setImageBitmap(ImageHelper.handleImageEffect(
                bitmap, mHue, mStauration, mLum));
    }

    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {
    }

    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
    }
}
```
```java

public class ImageHelper {

    public static Bitmap handleImageEffect(Bitmap bm,
                                           float hue,
                                           float saturation,
                                           float lum) {
        Bitmap bmp = Bitmap.createBitmap(
                bm.getWidth(), bm.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bmp);
        Paint paint = new Paint();

        ColorMatrix hueMatrix = new ColorMatrix();
        hueMatrix.setRotate(0, hue);
        hueMatrix.setRotate(1, hue);
        hueMatrix.setRotate(2, hue);

        ColorMatrix saturationMatrix = new ColorMatrix();
        saturationMatrix.setSaturation(saturation);

        ColorMatrix lumMatrix = new ColorMatrix();
        lumMatrix.setScale(lum, lum, lum, 1);

        ColorMatrix imageMatrix = new ColorMatrix();
        imageMatrix.postConcat(hueMatrix);
        imageMatrix.postConcat(saturationMatrix);
        imageMatrix.postConcat(lumMatrix);

        paint.setColorFilter(new ColorMatrixColorFilter(imageMatrix));
        canvas.drawBitmap(bm, 0, 0, paint);
        return bmp;
    }
}
```
###2.Android颜色矩阵——ColorMatrix

模拟一个4X5的颜色矩阵，通过修改矩阵的值，验证前面所说的改变图像色彩效果的原理，对矩阵产生的作用效果更加清晰

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <ImageView
        android:id="@+id/imageview"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="2" />

    <GridLayout
        android:id="@+id/group"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="3"
        android:columnCount="5"
        android:rowCount="4" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >

        <Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:onClick="btnChange"
            android:text="Change" />

        <Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:onClick="btnReset"
            android:text="Reset" />
    </LinearLayout>

</LinearLayout>
```

要动态的去添加EditText

```java
	private Bitmap bitmap;
	private GridLayout mGroup;
	private ImageView mImageView;
	// 高宽
	private int mEtWidth, mEtHeight;

	private EditText [] mEts = new EditText[20];




    bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.logo);
    		mImageView = (ImageView) findViewById(R.id.imageview);
    		mGroup = (GridLayout) findViewById(R.id.group);
    		mImageView.setImageBitmap(bitmap);
    
    		mGroup.post(new Runnable() {
    
    			@Override
    			public void run() {
    				// 获取高宽信息
    				mEtWidth = mGroup.getWidth() / 5;
    				mEtHeight = mGroup.getHeight() / 4;
    				addEts();
    				initMatrix();
    			}
    
    		});

	}

	/**
	 * 添加输入框
	 */
	private void addEts() {
		for (int i = 0; i < 20; i++) {
			EditText editText = new EditText(this);
			mEts[i] = editText;
			mGroup.addView(editText, mEtWidth, mEtHeight);
		}
	}

	/**
	 * 初始化颜色矩阵为初始状态
	 */
	private void initMatrix() {
		for (int i = 0; i < 20; i++) {
			if (i % 6 == 0) {
				mEts[i].setText(String.valueOf(1));
			} else {
				mEts[i].setText(String.valueOf(0));
			}
		}
	}
	
```

>**注意：**无法在onCreate里获得视图的宽高值，所以通过View的Post方法，在视图创建完成后获得其宽高值

接下来，只需要获得修改后的edittext值，并将矩阵值设置给颜色矩阵即可
```java

	/**
	 * 获得矩阵值
	 */
	private void getMatrix() {
		for (int i = 0; i < 20; i++) {
			mColorMatrix[i] = Float.valueOf(mEts[i].getText().toString());
		}
	}

	/**
	 * 将矩阵值设置到图像
	 */
	private void setImageMatrix() {
		Bitmap bitmap = Bitmap.createBitmap(mEtWidth, mEtHeight,
				Bitmap.Config.ARGB_8888);
		ColorMatrix colorMatrix = new ColorMatrix();
		colorMatrix.set(mColorMatrix);

		Canvas canvas = new Canvas(bitmap);
		Paint paint = new Paint();
		paint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
		canvas.drawBitmap(bitmap, 0, 0, paint);
		mImageView.setImageBitmap(bitmap);
	}

```

最后设置两个按钮的点击事件

```java
    /**
	 * 作用点击事件
	 */
	public void btnChange(View view) {
		getMatrix();
		setImageMatrix();
	}

	/**
	 * 重置矩阵效果
	 */
	public void btnReset(View view) {
		initMatrix();
		getMatrix();
		setImageMatrix();
	}

```

###3.常用图象颜色矩阵处理效果

 - 灰度效果
 - 反转效果
 - 怀旧效果
 - 去色效果
 - 高饱和度

###4.像素点分析
作为更加精确的图像处理方式，可以通过改变每个像素点的具体ARGB值，达到处理一张图片效果的目的，这里要注意的是，传递进来的原始图片是不能修改的，一般根据原始图片生成一张新的图片来修改

在Android中，系统系统提供了Bitmap.getPixels()方法来帮我们提取整个Bitmap中的像素密度点，并保存在一个数组中，该方法如下

```java
    bitmap.getPixels(pixels,offset, stride, x, y, width, height);
```

参数的具体含义如下：

- pixels ——接收位图颜色值的数组，
- offset——写入到pixels[]第一个像素索引值，
- stride——pixels[]中的行间距
- x——从位图中读取的第一个像素的ｘ坐标
- y——从图中读取的第一个像素的的y坐标
- width——从每一行读取的像素宽度
- height——读取的行数

通常情况下，可以这样

```java
 bitmap.getPixels(oldPx, 0, bitmap.getWidth(), 0, 0, width, height);
```

接下来，我们可以获取每一个像素具体的ARGB值，代码如下

```java
		color = oldPx[i];
		r = Color.red(color);
		g = Color.green(color)
		b = Color.blue(color);
		a = Color.alpha(color);
```

当获取到具体的颜色值后，就可以通过相应的算法去修改这个ARGB值了，从而重构一张图片

```java
		r1 = (int) (0.393 * r + 0.769 * g + 0.189 * b);
        g1 = (int) (0.349 * r + 0.686 * g + 0.168 * b);
        b1 = (int) (0.272 * r + 0.534 * g + 0.131 * b);
```

再通过如下代码将新的RGBA值合成像素点

```java
 newPx[i] = Color.argb(a, r1, b1, g1);
```

最后将处理后的像素点重新设置成新的bitmap

```java
bmp.setPixels(newPx, 0, width, 0, 0, width, height);
```

###5.常用图像图片像素点处理效果

####1.底片效果

若在ABC三个像素点上，要求B点对应的底片效果算法，代码如下

```java
		B.r = 255 - B.r;
        B.g = 255 - B.g;
        B.b = 255 - B.b;
```

实际代码如下

```java
    /**
     * 底片效果
     *
     * @param bm
     * @return
     */
    public  Bitmap handleImageNegative(Bitmap bm) {
        int width = bm.getWidth();
        int height = bm.getHeight();
        int color;
        int r, g, b, a;

        Bitmap bmp = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);

        int[] oldPx = new int[width * height];
        int[] newPx = new int[width * height];

        bm.getPixels(oldPx, 0, width, 0, 0, width, height);

        for (int i = 0; i < width * height; i++) {
            color = oldPx[i];
            r = Color.red(color);
            g = Color.green(color);
            b = Color.blue(color);
            a = Color.alpha(color);

            r = 255 - r;
            g = 255 - g;
            b = 255 - b;

            if (r > 255) {
                r = 255;
            } else if (r < 0) {
                r = 0;
            }

            if (g > 255) {
                g = 255;
            } else if (g < 0) {
                g = 0;
            }

            if (b > 255) {
                b = 255;
            } else if (b < 0) {
                b = 0;
            }
            newPx[i] = Color.argb(a, r, g, b);
        }
        bmp.setPixels(newPx, 0, width, 0, 0, width, height);
        return bmp;
    }

```

####2.老照片效果

```java
 r = (int) (0.393 * r + 0.769 * g + 0.189 * b);
 g = (int) (0.349 * r + 0.686 * g + 0.168 * b);
 b = (int) (0.272 * r + 0.534 * g + 0.131 * b);
```

####3.浮雕效果

```java
B.r = C.r - B.r + 127;
B.g = C.g - B.g + 127;
B.b = C.b - B.b + 127;
```

##六.Android图像处理之图像特效处理

###1.Android变形矩阵——Matrix
图像颜色的处理，Android系统提供了ColorMatrix颜色矩阵来帮助我们进行图像处理，而对于图像的图形变换，Android系统也可以通过矩阵来帮忙，每个像素点都表达了其xy的信息，Android的变化就是一个3X3的矩阵

![这里写图片描述](http://img.blog.csdn.net/20160330222354958)


```
 X1 = a x X +b x Y +c;
 Y1 = d x X +e x Y +f;
 1 = g x X +hx Y + i;
```

>通常情况下，会让g = h = 0,i = 1;这样使1 = g x X + h x Y + i恒成立，因此只需要关注这几个参数就行

与色彩变换矩阵的初始化矩阵一样，图形变换矩阵也需要一个初始矩阵，很明显就是对角线元素，其他元素为0的矩阵

 ![这里写图片描述](http://img.blog.csdn.net/20160330223339477)

>图像的变形处理包含以下四类基本变换

- Translate——平移变换
- Rotate——旋转变换
- Scale——缩放变换
- Skew——错切变换

####1.平移变换

![这里写图片描述](http://img.blog.csdn.net/20160330224213215)

当p(x0,y0)平移到p(x,y)时，坐标发生了如下的变化

```
X = X0 + ^X;
Y = Y0 + ^Y;
```

![这里写图片描述](http://img.blog.csdn.net/20180412101559971?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
通过平移变换矩阵计算我们可以发现如下平移公式

```
X = X0 + ^X;
Y = Y0 + ^Y;
```

####2.旋转变换
旋转变换实际上就是一个点围绕一个中心旋转到一个新的点上

![这里写图片描述](http://img.blog.csdn.net/20160330230027394)

通过计算，可以还原以上等式

![这里写图片描述](http://img.blog.csdn.net/20160331111539445)

前面是以坐标原点为旋转中心旋转变换，如果以任一点0为旋转中心进行旋转变换，通常需要以下三个步骤

- 将坐标原点平移到0点
- 使用前面讲的以坐标原点为中心的旋转方法进行旋转变换
- 将坐标原点还原

通过上述三个步骤，就可以以任一点进行旋转了


####3.缩放变换
缩放效果的计算公式

```
x = K1 X x0;
y = K2 X y0;
```

写成矩阵的话

![这里写图片描述](http://img.blog.csdn.net/20160331112137853)

通过计算，就可以还原上述等式了

#### 4.错切变换
错切变换（skew）在数学上又称为Shear mapping(可译为“剪切变换”)，或者Transvection(缩并)，它是一种比较特殊的线性变换，错切变换的效果就是让所有的X坐标标尺不变，一般分水平和垂直

![这里写图片描述](http://img.blog.csdn.net/20160331112958294)


通过了一个一位数组将一个一位数组转换为变换矩阵

```java
private float [] mImageMatrix = new float[9];
Matrix matrix = new Matrix();
matrix.setValues(mImageMatrix);
canvas.drawBitmap(mBitmmap,matrix,null);
```


###2.像素块分析
进行图像的处理方式也有两种，即前面讲的使用矩阵来进行图像变换和马上要使用到的drawBitmapMesh()方法来处理，操作像素块的原理，该方法如下

```java
canvas.drawBitmapMesh(Bitmap bitmap,int meshWidth,int meshHeight,float [] verts,int vertOffset,int [] color,int colorOffset,Paint paint);
```
- bitmap:将要扭曲的图像
- meshWidth:需要的横向网格数目
- meshHeight:需要的纵向网格数目
- verts:网格交叉点的坐标数组
- vertOffset:verts数组中开始跳过的(X,Y)坐标对的数目

要想使用drawBitmapMesh()的方法先要将突破分成若干份，所以在图像各画N-1条线，将图片分成N块

```java
mBitmap = BitmapFactory.decodeResource(getResources(),R.drawable.nice);

        float bitmapWidth = mBitmap.getWidth();
        float bitmapHeight = mBitmap.getHeight();

        int index = 0;
        for (int y = 0;y<=HEIGHT;y++){
            float fy = bitmapHeight*y/HEIGHT;
            for (int x = 0;x<= WIDTH;x++){
                float fx = bitmapWidth*x/WIDTH;
                orig[index*2+0] = verts[index*2+0] = fx;
                //这里人为将坐标+100是为了让图像下移，避免扭曲后被屏幕遮挡
                orig[index*2+1] = verts[index*2+1] = fx+100;
                index +=1;
            }
        }
```

接下来，在onDraw()方法中改变交叉点的纵坐标的值，为了实现旗帜飘飘的效果，使用sin x，来改变交叉点的坐标值，使横坐标不变，将其变化后的值保存在数组中

```java
private void flagWava(){
        for (int j = 0; j<=HEIGHT;j++){
            for (int i = 0; i<=WIDTH;i++){
                verts[(j*(WIDTH+1)+i)*2+0] +=0;
                float offsetY = (float)Math.sin((float)i/WIDTH*2*Math.PI);
                verts[(j*(WIDTH+1)+i)*2+1] =  orig[(j*(WIDTH+i)+i)*2+1]+offsetY*A;
            }
        }
    }
```

然后

```
canvas.drawBitmapMesh(mBitmap,WIDTH,HEIGHT,verts,0,null,0,null);
```

为了能够让图片动起来，可以利用正弦函数的周期性来实现，在获取纵坐标的偏移量的时候给函数增加一个周期

```
 float offsetY = (float)Math.sin((float)i/WIDTH*2*Math.PI+Math.PI*k);
```

在重绘 d 时候，给K值增加

```
        flagWava();
        k+=0.1f;   
        canvas.drawBitmapMesh(mBitmap,WIDTH,HEIGHT,verts,0,null,0,null);
        invalidate();
```

每次重绘的时候，通过改变相位来改变偏移量，从而造成一个动态的效果，就好像旗帜在空中飘扬一样

>使用drawBitmapMesh（）方法可以创建很多复杂的特效，但是对他的使用也相对来讲比较复杂，需要开发者对图像处理有很深的功底，同时对算法要求比较高，才能做出特效

##七.Android图像处理之画笔特效处理
>不管是在我们的世界里，还是在Android的世界里，要想画出好的图片，就必须账务画笔的特效，今天我们就来玩玩这个Paint的特殊Get

###1.PorterDuffXfermode

![这里写图片描述](https://upload-images.jianshu.io/upload_images/2041548-d964105abf4be5d9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/312)

>**注意：** PorterDuffXfermode设置两个图层交集区域的显示方法des是先画的图像，src是后画的

```java
mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.nice);
        mOut = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(mOut);
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        canvas.drawRoundRect(0, 0, mBitmap.getWidth(), mBitmap.getHeight(), 80, 80, mPaint);
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
        canvas.drawBitmap(mBitmap,0,0,mPaint);
```

类似于刮刮卡的效果,其实效果就是两张图片，上面那张一刮就显示下面的那张：
首先要做的就是初始化一些数据

```java
	/**
     * 初始化
     */
    private void init() {
        mPaint = new Paint();
        mPaint.setAlpha(0);
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
        mPaint.setStrokeWidth(50);
        mPaint.setStrokeCap(Paint.Cap.ROUND);

        mPath = new Path();
        mBgBitmap = BitmapFactory.decodeResource(getResources(),R.drawable.nice);
        mFgBitmap = Bitmap.createBitmap(mBgBitmap.getWidth(), mBgBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        mCanvas = new Canvas(mFgBitmap);
        mCanvas.drawColor(Color.GRAY);
    }
```

给paint设置一些属性，让他更圆一点，然后我们再监听触摸时间

```java
	/**
     * 触摸事件
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                mPath.reset();
                mPath.moveTo(event.getX(),event.getY());
                break;
            case MotionEvent.ACTION_UP:
                mPath.lineTo(event.getX(),event.getY());
                break;
        }
        mCanvas.drawPath(mPath,mPaint);
        invalidate();
        return true;
    }
```

最后使用DST_IN来完善覆盖即可，这里我把源码放上来

```java

/**
 * 涂鸦
 */
public class PlayView extends View {

    private Bitmap mBgBitmap, mFgBitmap;
    private Paint mPaint;
    private Canvas mCanvas;
    private Path mPath;

    /**
     * 构造方法
     *
     * @param context
     * @param attrs
     */
    public PlayView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    /**
     * 初始化
     */
    private void init() {
        mPaint = new Paint();
        mPaint.setAlpha(0);
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
        mPaint.setStrokeWidth(50);
        mPaint.setStrokeCap(Paint.Cap.ROUND);

        mPath = new Path();
        mBgBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.nice);
        mFgBitmap = Bitmap.createBitmap(mBgBitmap.getWidth(), mBgBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        mCanvas = new Canvas(mFgBitmap);
        mCanvas.drawColor(Color.GRAY);
    }

    /**
     * 触摸事件
     *
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPath.reset();
                mPath.moveTo(event.getX(), event.getY());
                break;
            case MotionEvent.ACTION_UP:
                mPath.lineTo(event.getX(), event.getY());
                break;
        }
        mCanvas.drawPath(mPath, mPaint);
        invalidate();
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawBitmap(mBgBitmap, 0, 0, null);
        canvas.drawBitmap(mFgBitmap, 0, 0, null);
    }
}

```

>注意：使用PorterDuffXfermode的时候最好把硬件加速给关掉，因为有些不支持

###2.Shader
>Shader又被称为着色器。渲染器，它可以实现渲染，渐变等效果，Android中的Shader包括以下几种

- BitmapShader:位图Shader
- LinearGradient:线性Shader
- RadialGradient:光束Shader
- SweepGradient:梯形Shader
- ComposeShader:混合Shader

BitmapShader所产生的是一个图像，有点类似PS里面的图像填充，他的作用是通过Paint对画布进行制定bitmap的填充，填充式有一下几个模式供选择：

- CLAMP拉伸——拉伸的是图片最后的哪一个像素，不断重复
- REPEAT重复——横向，纵向不断重复
- MIRROR镜像——横向不断翻转重复，纵向不断翻转重复



```java
 mBitmap = BitmapFactory.decodeResource(getResources(),R.drawable.nice);
        mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP,Shader.TileMode.CLAMP);
        mPaint = new Paint();
        mPaint.setShader(mBitmapShader);
        canvas.drawCircle(500,250,200,mPaint);
```

看LinearGradient，理解起来就是线性渐变的意思，例子：

```java
mPaint = new Paint();
        mPaint.setShader(new LinearGradient(0,0,400,400, Color.BLUE,Color.YELLOW, Shader.TileMode.REPEAT));
        canvas.drawRect(0,0,400,400,mPaint);
```

倒影的例子：

```java

/**
 * 倒影
 */
public class InvertedView extends View {

    private Bitmap mSrcBitmap, mRefBitmap;
    private Paint mPaint;
    private PorterDuffXfermode xfermode;

    /**
     * 构造方法
     *
     * @param context
     * @param attrs
     */
    public InvertedView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initRes(context);
    }

    /**
     * 初始化
     *
     * @param context
     */
    private void initRes(Context context) {
        mSrcBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.nice);
        Matrix matrix = new Matrix();
        matrix.setScale(1F, -1F);
        mRefBitmap = Bitmap.createBitmap(mSrcBitmap, 0, 0, mSrcBitmap.getWidth(), mSrcBitmap.getHeight(), matrix, true);

        mPaint = new Paint();
        mPaint.setShader(new LinearGradient(0, mSrcBitmap.getHeight(), 0, mSrcBitmap.getHeight() + mSrcBitmap.getHeight() / 4, 0XDD000000, 0X10000000, Shader.TileMode.CLAMP));
        xfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
    }

    /**
     * 绘制
     *
     * @param canvas
     */
    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawColor(Color.BLACK);
        canvas.drawBitmap(mSrcBitmap, 0, 0, null);
        canvas.drawBitmap(mRefBitmap, 0, mSrcBitmap.getHeight(), null);
        mPaint.setXfermode(xfermode);
        //绘制渐变效果矩形
        canvas.drawRect(0, mSrcBitmap.getHeight(), mRefBitmap.getWidth(), mSrcBitmap.getHeight() * 2, mPaint);
        mPaint.setXfermode(null);
    }
}

```

###3.PathEffect

![这里写图片描述](http://img.blog.csdn.net/20160402205535417)

PathEffect就是指，用各种笔触效果绘制路径，Android系统提供的从上到下

- CornerPathEffect:拐弯处变圆滑
- DiscretePathEffect：线段上有杂点
- DashPathEffect：虚线
- PathDashPathEffect：虚线可以设置点的形状
- ComposePathEffect：组合效果

简单的实例：

```java

/**
 * PathEffect
 */
public class PathEffectView extends View{

    private Path mPath;
    private PathEffect [] mEffect = new PathEffect[6];
    private Paint mPaint;

    /**
     * 构造方法
     * @param context
     * @param attrs
     */
    public PathEffectView(Context context, AttributeSet attrs) {
        super(context, attrs);

        init();
    }

    /**
     * 初始化
     */
    private void init() {
        mPaint = new Paint();
        mPath = new Path();
        mPath.moveTo(0,0);
        for (int i = 0; i<= 30;i++){
            mPath.lineTo(i*35,(float)(Math.random()*100));
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        mEffect[0] = null;
        mEffect[1] = new CornerPathEffect(30);
        mEffect[2] = new DiscretePathEffect(3.0F,5.0F);
        mEffect[3] = new DashPathEffect(new float[]{20,10,5,10},0);
        Path path = new Path();
        path.addRect(0,0,8,8,Path.Direction.CCW);
        mEffect[4]= new PathDashPathEffect(path,12,0,PathDashPathEffect.Style.ROTATE);
        mEffect[5] = new ComposePathEffect(mEffect[3],mEffect[1]);
        for (int i = 0; i<mEffect.length;i++){
            mPaint.setPathEffect(mEffect[i]);
            canvas.drawPath(mPath,mPaint);
            canvas.translate(0,200);
        }
    }
}

```

##八.View之孪生兄弟——SurfaceView
###1.SurfaceView和View的区别
Surfaceview组件和View的区别主要体现

- 1.View主要用于自动更新的情况下,而surfaceVicw主要适用于被动更新,例如频繁刷新
- 2.View在主线程中刷新，而surfaceView通常会通过一 个子线程来进行页面刷新。
- 3.View在绘制的时候没有双缓冲机制,而surfaceVicw在底层实现机制中就已经实现了双缓冲机制；

>**总结：**自定义View需要频繁刷新，或者刷新时数据处理量比较大，就可以考虑使用surfaceVicw取代View了
###2.surfaceView的使用
SurfaeView在使用时，有一套使用的模板代码，大部分的surfaceView绘图操作都可以套用这样的模板代码来进行编写。

>通常情况下 使用以下步骤来创建一个surfaceView的模板。

- 创建surfaceView

创建自定义的surfaceView，实现它的两个接口

```java
public class SurfaView extends SurfaceView implements SurfaceHolder.Callback,Runnable
```

通过实现它的两个几口，就会有三个回调方法

```java
@Override
    public void surfaceCreated(SurfaceHolder holder) {

    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {

    }
```
分别对应的是SurfaceView的创建，改变和销毁的过程

对于Runnable接口，会实现他的回调

```java
	@Override
    public void run() {

    }
```

 - 初始化SurfaceView

在自定义的SurfaceView构造方法中，需要对SurfaceView进行出还刷，我们首先要定义三个成员变量

```java
	 //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘制的Canvas
    private Canvas mCanvas;
     //子线程标志位
    private boolean mIsDrawing;
```

初始化方法就是对回调初始化以及注册

```java
mHolder = getHolder();
mHolder.addCallback(this);
```

另外两个成员变量一Canvas和标志位。 使用Canvas来进行绘图，另一个标志位则是用来控制子线程的

- 使用SurfaceView

通过SurfaceHolder对象的lockCanvas方法，就可以获得当前Canvas绘图对象，就可以进行绘制，要注意的是Canvas对象还是继续上次的Canvas对象,而不是一个新的对象。因此，之前的操作都将被保留，如果需要擦除,则可以在绘制前, 通过drawColor方法来进行清屏操作

绘制的时候，充分利用SurfaceView的回调方法，在surfaceCreated方法中开启子线程进行绘制，而子线程开启了一个while(mIsDrawing)的循环来不停的绘制，而在具体的逻辑中，通过lookCanvas()方法获取Canvas对象进行绘制，，整个代码模块如下：

```java

/**
 * SurfaceView的使用
 */
public class SurfaView extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘制的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;

    /**
     * 构造方法
     *
     * @param context
     * @param attrs
     */
    public SurfaView(Context context, AttributeSet attrs) {
        super(context, attrs);

        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
        }
    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
        } catch (Exception e) {

        } finally {
            if (mCanvas != null) {
                //提交
                mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
}

```

> **注意：**绘制的时候mHolder.unlockCanvasAndPost(mCanvas);放在finally中，来保证每次都能将任务提交

###3.SurfaceView实例


####1.正弦曲线

```java
/**
 * SurfaceView的使用
 */
public class SurfaView extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘制的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;
    private int x, y = 0;
    private Path mPath;
    private Paint mPaint;

    /**
     * 构造方法
     *
     * @param context
     * @param attrs
     */
    public SurfaView(Context context, AttributeSet attrs) {
        super(context, attrs);

        mPath = new Path();
        mPaint = new Paint();
        mPaint.setStrokeWidth(20);
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
            x += 1;
            y = (int) (100 * Math.sin(x * 2 * Math.PI / 180) + 400);
            mPath.lineTo(x, y);
        }
    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath, mPaint);
        } catch (Exception e) {

        } finally {
            if (mCanvas != null) {
                //提交
                mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
}

```

####2.绘图板
>实现一个画板

```java

/**
 * SurfaceView的使用
 */
public class SurfaView extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘制的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;
    private int x, y = 0;
    private Path mPath;
    private Paint mPaint;

    /**
     * 构造方法
     *
     * @param context
     * @param attrs
     */
    public SurfaView(Context context, AttributeSet attrs) {
        super(context, attrs);

        mPath = new Path();
        mPaint = new Paint();
        mPaint.setStrokeWidth(20);
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
//            x += 1;
//            y = (int) (100 * Math.sin(x * 2 * Math.PI / 180) + 400);
//            mPath.lineTo(x, y);
        }
    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath, mPaint);
        } catch (Exception e) {

        } finally {
            if (mCanvas != null) {
                //提交
                mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }

    /**
     * 触摸事件
     *
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {

        int x = (int) event.getX();
        int y = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPath.moveTo(x, y);

                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(x, y);
                break;
            case MotionEvent.ACTION_UP:

                break;
        }
        return true;
    }
}

```




