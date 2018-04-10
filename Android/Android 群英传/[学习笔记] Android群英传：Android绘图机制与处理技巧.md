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

```
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
```
   setAntiAlias();            //设置画笔的锯齿效果

　　setColor();                //设置画笔的颜色

　　setARGB();                 //设置画笔的A、R、G、B值

　　setAlpha();                //设置画笔的Alpha值

　　setTextSize();             //设置字体的尺寸

　　setStyle();                //设置画笔的风格（空心或实心）

　　setStrokeWidth();          //设置空心边框的宽度

　　getColor();                //获取画笔的颜色

```


```

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

```
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher">

</bitmap>
```

通过这样引用图片就可以将图片直接转化成Bitmap让我们在程序中使用

###2.Shape

通过Shape可以绘制各种图形，下面展示一下shape的参数

```
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
    <stroke
        android:width="1dp"
        android:color="@color/colorAccent" />
    <!--虚线宽度-->
    android:dashWidth= "1dp"

    <!--虚线间隔宽度-->
    android:dashGap= "1dp"

</shape>

```

>shape可以说是xml绘图的精华所在，而且功能十分的强大，无论是扁平化，拟物化还是渐变，都是十分的OK，我们现在来做一个阴影的效果

```
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

```
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

```
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

```
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

```
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

```
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

```

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
