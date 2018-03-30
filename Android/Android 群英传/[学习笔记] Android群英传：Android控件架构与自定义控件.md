# [学习笔记] Android群英传：Android控件架构与自定义控件

---

##Android控件架构
- View树结构
![此处输入图片的描述][1]
- UI界面框架
![此处输入图片的描述][2]
- 标准视图树
![此处输入图片的描述][3]


##View的测量

MeasureSpec类，通过他来帮助我们测量View, MeasureSpec是一个32位的int值，其中高2位为测量模式，低30为测量的大小，在计算中使用位运算时为了提高并且优化效率

###测量模式
- EXACTLY：具体值和match_parent
- AT_MOST：wrap_parent
- UNSPECIFIED:将视图按照自己的意愿设置成任意的大小，没有任何限制

View默认的onMeasure只支持EXACTLY模式，在自定义控件的时候不重写这个方法，只能使用EXACTLY模式
###例子
```
    /**
     * 测量
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

```
重写onMeasure这个方法
```
 @Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {
        setMeasuredDimension(
                measureWidth(widthMeasureSpec),
                measureHeight(heightMeasureSpec));
    }
```
```
 private int measureWidth(int measureSpec) {
        int result = 0;
        //从MeasureSpec类中提取出具体的测量模式和大小
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else {
            result = 200;//设置一个默认值
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }
```
## View的绘制

### Canvas
画布，而onDraw()就含有参数Canvas了，在其他地方绘制的话，就需要new对象
```
    Canvas canvas = new Canvas(Bitmap);
```
Bitmap是和Canvas紧密联系的，这个过程我们称之为装载画布，这个bitmap用来储存画布的一些像素信息，而且我们可以用canvas.drawxxx()来绘制相关的基础图形
```
//绘制直线
canvas.drawLine(float startX, float startY, float stopX, float stopY, Paint paint);

//绘制矩形
canvas.drawRect(float left, float top, float right, float bottom, Paint paint);

//绘制圆形
canvas.drawCircle(float cx, float cy, float radius, Paint paint);

//绘制字符
canvas.drawText(String text, float x, float y, Paint paint);

//绘制图形
canvas.drawBirmap(Bitmap bitmap, float left, float top, Paint paint);
```
## ViewGroup的测量
- ViewGroup是用来管理View的，包括View的大小，当ViewGroup大小是包裹内容，ViewGroup会遍历所有的子View，来获取View的大小，从而决定自身的大小。在其他模式下，会通过具体的值来自定自身的大小。
- ViewGroup遍历所有的View，会调用所有的View的onMeasure()方法来获取测量结果，当子View测量完毕之后，就需要将子View放在合适的地方，这部分是由onLayout()来进行的，
- 自定义ViewGroup的时候，一般都要重写onLayout()方法控制子View显示位置的逻辑，同样，如果需要wrap_content属性，那就必须重写onLayout()方法了

##ViewGroup的绘制
ViewGroup在一般情况下是不会绘制的，因为他本身没有需要绘制的东西，如果不是指定ViewGroup的背景颜色，他连onDraw()都不会调用，但是ViewGroup会使用dispatchDraw()来绘制其他子View，其过程同样是遍历所有的子View，并调用子View的绘制方法来完成绘制的
```
    @Override
    protected void dispatchDraw(Canvas canvas)  {
        super.dispatchDraw(canvas);
    }
```

##自定义View

**在View中通常有以下比较重要的回调方法:**

1. onFinishInflate()：从XML加载组件后回调
2. onSizeChanged()：组件大小改变时回调
3. onMeasure()：回调该方法进行测量
4. onLayout()：回调该方法来确定显示的位置
5. onTouchEvent():监听到触摸时间时回调
6. onDraw():绘图

**一般实现自定义控件有三种方法:**

1. 对现有的控件进行扩展
2. 通过组件来实现新的控件
3. 重写View来实现全新的控件

### 对现有的控件进行扩展
在原生控件的基础上进行扩展，增加新功能、修改UI等。
```
 @Override
    protected void onDraw(Canvas canvas) {
        //在回调父类之前，实现自己的逻辑，对textview来说就是绘制文本内容前
        super.onDraw(canvas);
         //在回调父类之后，实现自己的逻辑，对textview来说就是绘制文本内容后
     }
```

例子1：
```
public class MyTextView extends TextView {

    private Paint mPaint1, mPaint2;

    public MyTextView(Context context) {
        super(context);
        initView();
    }

    public MyTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public MyTextView(Context context, AttributeSet attrs,
                      int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        mPaint1 = new Paint();
        mPaint1.setColor(getResources().getColor(
                android.R.color.holo_blue_light));
        mPaint1.setStyle(Paint.Style.FILL);
        mPaint2 = new Paint();
        mPaint2.setColor(Color.YELLOW);
        mPaint2.setStyle(Paint.Style.FILL);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // 绘制外层矩形
        canvas.drawRect(
                0,
                0,
                getMeasuredWidth(),
                getMeasuredHeight(),
                mPaint1);
        // 绘制内层矩形
        canvas.drawRect(
                10,
                10,
                getMeasuredWidth() - 10,
                getMeasuredHeight() - 10,
                mPaint2);
        canvas.save();
        // 绘制文字前平移10像素
        canvas.translate(10, 0);
        // 父类完成的方法，即绘制文本
        super.onDraw(canvas);
        canvas.restore();
    }
}
```
例子2：
```

public class ShineTextView extends TextView {

    private LinearGradient mLinearGradient;
    private Matrix mGradientMatrix;
    private Paint mPaint;
    private int mViewWidth = 0;
    private int mTranslate = 0;

    public ShineTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (mViewWidth == 0) {
            mViewWidth = getMeasuredWidth();
            if (mViewWidth > 0) {
                mPaint = getPaint();
                mLinearGradient = new LinearGradient(
                        0,
                        0,
                        mViewWidth,
                        0,
                        new int[]{
                                Color.BLUE, 0xffffffff,
                                Color.BLUE},
                        null,
                        Shader.TileMode.CLAMP);
                mPaint.setShader(mLinearGradient);
                mGradientMatrix = new Matrix();
            }
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (mGradientMatrix != null) {
            mTranslate += mViewWidth / 5;
            if (mTranslate > 2 * mViewWidth) {
                mTranslate = -mViewWidth;
            }
            mGradientMatrix.setTranslate(mTranslate, 0);
            mLinearGradient.setLocalMatrix(mGradientMatrix);
            postInvalidateDelayed(100);
        }
    }
}
```

### 通过组件来实现新的控件

经常需要继承一个合适的ViewGroup，再给他添加指定功能的控件，从而组成一个新的合适的控件

1. 定义属性
需要在values下新建一个attrs.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="TopBar">
        <attr name="title" format="string" />
        <attr name="titleTextSize" format="dimension" />
        <attr name="titleTextColor" format="color" />
        <attr name="leftTextColor" format="color" />
        <attr name="leftBackground" format="reference|color" />
        <attr name="leftText" format="string" />
        <attr name="rightTextColor" format="color" />
        <attr name="rightBackground" format="reference|color" />
        <attr name="rightText" format="string" />
    </declare-styleable>
</resources>
```
用<declare-styleable>标签声明一些属性的，然后name确定引用名称。 <attr>标签声明具体自定义属性。

在构造方法中，通过下列代码获取xml中的自定义属性。
```
 TypedArray ta = context.obtainStyledAttributes(attrs,R.styleable.TopBar);
```
通过TypedArray对象的方法读取出相应的值设置属性
```
        mLeftTextColor = ta.getColor(R.styleable.TopBar_leftTextColor, 0);
```

获取完TypedArray的值之后，一般要调用recyle方法来避免重复创建时候的错误
```
        ta.recycle();
```
2. 组合控件
完整代码
```
public class TopBar extends RelativeLayout {

    // 包含topbar上的元素：左按钮、右按钮、标题
    private Button mLeftButton, mRightButton;
    private TextView mTitleView;

    // 布局属性，用来控制组件元素在ViewGroup中的位置
    private LayoutParams mLeftParams, mTitlepParams, mRightParams;

    // 左按钮的属性值，即我们在atts.xml文件中定义的属性
    private int mLeftTextColor;
    private Drawable mLeftBackground;
    private String mLeftText;
    // 右按钮的属性值，即我们在atts.xml文件中定义的属性
    private int mRightTextColor;
    private Drawable mRightBackground;
    private String mRightText;
    // 标题的属性值，即我们在atts.xml文件中定义的属性
    private float mTitleTextSize;
    private int mTitleTextColor;
    private String mTitle;

    // 映射传入的接口对象
    private topbarClickListener mListener;

    public TopBar(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    public TopBar(Context context) {
        super(context);
    }

    public TopBar(Context context, AttributeSet attrs) {
        super(context, attrs);
        // 设置topbar的背景
        setBackgroundColor(0xFFF59563);
        // 通过这个方法，将你在atts.xml中定义的declare-styleable
        // 的所有属性的值存储到TypedArray中
        TypedArray ta = context.obtainStyledAttributes(attrs,
                R.styleable.TopBar);
        // 从TypedArray中取出对应的值来为要设置的属性赋值
        mLeftTextColor = ta.getColor(
                R.styleable.TopBar_leftTextColor, 0);
        mLeftBackground = ta.getDrawable(
                R.styleable.TopBar_leftBackground);
        mLeftText = ta.getString(R.styleable.TopBar_leftText);

        mRightTextColor = ta.getColor(
                R.styleable.TopBar_rightTextColor, 0);
        mRightBackground = ta.getDrawable(
                R.styleable.TopBar_rightBackground);
        mRightText = ta.getString(R.styleable.TopBar_rightText);

        mTitleTextSize = ta.getDimension(
                R.styleable.TopBar_titleTextSize, 10);
        mTitleTextColor = ta.getColor(
                R.styleable.TopBar_titleTextColor, 0);
        mTitle = ta.getString(R.styleable.TopBar_title);

        // 获取完TypedArray的值后，一般要调用
        // recyle方法来避免重新创建的时候的错误
        ta.recycle();

        mLeftButton = new Button(context);
        mRightButton = new Button(context);
        mTitleView = new TextView(context);

        // 为创建的组件元素赋值
        // 值就来源于我们在引用的xml文件中给对应属性的赋值
        mLeftButton.setTextColor(mLeftTextColor);
        mLeftButton.setBackground(mLeftBackground);
        mLeftButton.setText(mLeftText);

        mRightButton.setTextColor(mRightTextColor);
        mRightButton.setBackground(mRightBackground);
        mRightButton.setText(mRightText);

        mTitleView.setText(mTitle);
        mTitleView.setTextColor(mTitleTextColor);
        mTitleView.setTextSize(mTitleTextSize);
        mTitleView.setGravity(Gravity.CENTER);

        // 为组件元素设置相应的布局元素
        mLeftParams = new LayoutParams(
                LayoutParams.WRAP_CONTENT,
                LayoutParams.MATCH_PARENT);
        mLeftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT, TRUE);
        // 添加到ViewGroup
        addView(mLeftButton, mLeftParams);

        mRightParams = new LayoutParams(
                LayoutParams.WRAP_CONTENT,
                LayoutParams.MATCH_PARENT);
        mRightParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT, TRUE);
        addView(mRightButton, mRightParams);

        mTitlepParams = new LayoutParams(
                LayoutParams.WRAP_CONTENT,
                LayoutParams.MATCH_PARENT);
        mTitlepParams.addRule(RelativeLayout.CENTER_IN_PARENT, TRUE);
        addView(mTitleView, mTitlepParams);

        // 按钮的点击事件，不需要具体的实现，
        // 只需调用接口的方法，回调的时候，会有具体的实现
        mRightButton.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                mListener.rightClick();
            }
        });

        mLeftButton.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                mListener.leftClick();
            }
        });
    }

    // 暴露一个方法给调用者来注册接口回调
    // 通过接口来获得回调者对接口方法的实现
    public void setOnTopbarClickListener(topbarClickListener mListener) {
        this.mListener = mListener;
    }

    /**
     * 设置按钮的显示与否 通过id区分按钮，flag区分是否显示
     *
     * @param id   id
     * @param flag 是否显示
     */
    public void setButtonVisable(int id, boolean flag) {
        if (flag) {
            if (id == 0) {
                mLeftButton.setVisibility(View.VISIBLE);
            } else {
                mRightButton.setVisibility(View.VISIBLE);
            }
        } else {
            if (id == 0) {
                mLeftButton.setVisibility(View.GONE);
            } else {
                mRightButton.setVisibility(View.GONE);
            }
        }
    }

    // 接口对象，实现回调机制，在回调方法中
    // 通过映射的接口对象调用接口中的方法
    // 而不用去考虑如何实现，具体的实现由调用者去创建
    public interface topbarClickListener {
        // 左按钮点击事件
        void leftClick();
        // 右按钮点击事件
        void rightClick();
    }
}
```
3. 引用UI模板

自定义属需要自己的命名空间，例如：
```
xmlns:custom="http://schemas.android.com/apk/res-auto"
```
完整代码：
```
<com.xys.mytopbar.Topbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:custom="http://schemas.android.com/apk/res-auto"
    android:id="@+id/topBar"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    custom:leftBackground="@drawable/blue_button"
    custom:leftText="Back"
    custom:leftTextColor="#FFFFFF"
    custom:rightBackground="@drawable/blue_button"
    custom:rightText="More"
    custom:rightTextColor="#FFFFFF"
    custom:title="自定义标题"
    custom:titleTextColor="#123412"
    custom:titleTextSize="15sp">

</com.xys.mytopbar.Topbar>
```

通过<include>标签使用这个UI模板

### 重写View来实现全新的控件
通常需要基础View类，重写onDraw()，onMeasure()等方式实现绘制，同事重写onTouchEvent()等触控时间来事先交互逻辑。
CircleProgressView:
```

public class CircleProgressView extends View {

    private int mMeasureHeigth;
    private int mMeasureWidth;

    private Paint mCirclePaint;
    private float mCircleXY;
    private float mRadius;

    private Paint mArcPaint;
    private RectF mArcRectF;
    private float mSweepAngle;
    private float mSweepValue = 66;

    private Paint mTextPaint;
    private String mShowText;
    private float mShowTextSize;

    public CircleProgressView(Context context, AttributeSet attrs,
                              int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public CircleProgressView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public CircleProgressView(Context context) {
        super(context);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {

        mMeasureWidth = MeasureSpec.getSize(widthMeasureSpec);
        mMeasureHeigth = MeasureSpec.getSize(heightMeasureSpec);
        setMeasuredDimension(mMeasureWidth, mMeasureHeigth);
        initView();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // 绘制圆
        canvas.drawCircle(mCircleXY, mCircleXY, mRadius, mCirclePaint);
        // 绘制弧线
        canvas.drawArc(mArcRectF, 270, mSweepAngle, false, mArcPaint);
        // 绘制文字
        canvas.drawText(mShowText, 0, mShowText.length(),
                mCircleXY, mCircleXY + (mShowTextSize / 4), mTextPaint);
    }

    private void initView() {
        float length = 0;
        if (mMeasureHeigth >= mMeasureWidth) {
            length = mMeasureWidth;
        } else {
            length = mMeasureHeigth;
        }

        mCircleXY = length / 2;
        mRadius = (float) (length * 0.5 / 2);
        mCirclePaint = new Paint();
        mCirclePaint.setAntiAlias(true);
        mCirclePaint.setColor(getResources().getColor(
                android.R.color.holo_blue_bright));

        mArcRectF = new RectF(
                (float) (length * 0.1),
                (float) (length * 0.1),
                (float) (length * 0.9),
                (float) (length * 0.9));
        mSweepAngle = (mSweepValue / 100f) * 360f;
        mArcPaint = new Paint();
        mArcPaint.setAntiAlias(true);
        mArcPaint.setColor(getResources().getColor(
                android.R.color.holo_blue_bright));
        mArcPaint.setStrokeWidth((float) (length * 0.1));
        mArcPaint.setStyle(Style.STROKE);

        mShowText = setShowText();
        mShowTextSize = setShowTextSize();
        mTextPaint = new Paint();
        mTextPaint.setTextSize(mShowTextSize);
        mTextPaint.setTextAlign(Paint.Align.CENTER);
    }

    private float setShowTextSize() {
        this.invalidate();
        return 50;
    }

    private String setShowText() {
        this.invalidate();
        return "Android Skill";
    }

    public void forceInvalidate() {
        this.invalidate();
    }

    public void setSweepValue(float sweepValue) {
        if (sweepValue != 0) {
            mSweepValue = sweepValue;
        } else {
            mSweepValue = 25;
        }
        this.invalidate();
    }
}

```
VolumeView:
```

public class VolumeView extends View {

    private int mWidth;
    private int mRectWidth;
    private int mRectHeight;
    private Paint mPaint;
    private int mRectCount;
    private int offset = 5;
    private double mRandom;
    private LinearGradient mLinearGradient;

    public VolumeView(Context context) {
        super(context);
        initView();
    }

    public VolumeView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public VolumeView(Context context, AttributeSet attrs,
                      int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    private void initView() {
        mPaint = new Paint();
        mPaint.setColor(Color.BLUE);
        mPaint.setStyle(Paint.Style.FILL);
        mRectCount = 12;
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = getWidth();
        mRectHeight = getHeight();
        mRectWidth = (int) (mWidth * 0.6 / mRectCount);
        mLinearGradient = new LinearGradient(
                0,
                0,
                mRectWidth,
                mRectHeight,
                Color.YELLOW,
                Color.BLUE,
                Shader.TileMode.CLAMP);
        mPaint.setShader(mLinearGradient);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        for (int i = 0; i < mRectCount; i++) {
            mRandom = Math.random();
            float currentHeight = (float) (mRectHeight * mRandom);
            canvas.drawRect(
                    (float) (mWidth * 0.4 / 2 + mRectWidth * i + offset),
                    currentHeight,
                    (float) (mWidth * 0.4 / 2 + mRectWidth * (i + 1)),
                    mRectHeight,
                    mPaint);
        }
        postInvalidateDelayed(300);
    }
}
```

## 自定义ViewGroup
自定义ViewGroup是需要onMeasure()来测量的，然后重写onLayout()来确定位置，重写onTouchEvent()来相应事件

MyScrollView:
```
public class MyScrollView extends ViewGroup {

    private int mScreenHeight;
    private Scroller mScroller;
    private int mLastY;
    private int mStart;
    private int mEnd;

    public MyScrollView(Context context) {
        super(context);
        initView(context);
    }

    public MyScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context);
    }

    public MyScrollView(Context context, AttributeSet attrs,
                        int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context);
    }

    private void initView(Context context) {
        WindowManager wm = (WindowManager) context.getSystemService(
                Context.WINDOW_SERVICE);
        DisplayMetrics dm = new DisplayMetrics();
        wm.getDefaultDisplay().getMetrics(dm);
        mScreenHeight = dm.heightPixels;
        mScroller = new Scroller(context);
    }

    @Override
    protected void onLayout(boolean changed,
                            int l, int t, int r, int b) {
        int childCount = getChildCount();
        // 设置ViewGroup的高度
        MarginLayoutParams mlp = (MarginLayoutParams) getLayoutParams();
        mlp.height = mScreenHeight * childCount;
        setLayoutParams(mlp);
        for (int i = 0; i < childCount; i++) {
            View child = getChildAt(i);
            if (child.getVisibility() != View.GONE) {
                child.layout(l, i * mScreenHeight,
                        r, (i + 1) * mScreenHeight);
            }
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec,
                             int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int count = getChildCount();
        for (int i = 0; i < count; ++i) {
            View childView = getChildAt(i);
            measureChild(childView,
                    widthMeasureSpec, heightMeasureSpec);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mLastY = y;
                mStart = getScrollY();
                break;
            case MotionEvent.ACTION_MOVE:
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                int dy = mLastY - y;
                if (getScrollY() < 0) {
                    dy = 0;
                }
                if (getScrollY() > getHeight() - mScreenHeight) {
                    dy = 0;
                }
                scrollBy(0, dy);
                mLastY = y;
                break;
            case MotionEvent.ACTION_UP:
                int dScrollY = checkAlignment();
                if (dScrollY > 0) {
                    if (dScrollY < mScreenHeight / 3) {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, -dScrollY);
                    } else {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, mScreenHeight - dScrollY);
                    }
                } else {
                    if (-dScrollY < mScreenHeight / 3) {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, -dScrollY);
                    } else {
                        mScroller.startScroll(
                                0, getScrollY(),
                                0, -mScreenHeight - dScrollY);
                    }
                }
                break;
        }
        postInvalidate();
        return true;
    }
	private int checkAlignment() {
        int mEnd = getScrollY();
        boolean isUp = ((mEnd - mStart) > 0) ? true : false;
        int lastPrev = mEnd % mScreenHeight;
        int lastNext = mScreenHeight - lastPrev;
        if (isUp) {
            //向上的
            return lastPrev;
        } else {
            return -lastNext;
        }
    }
    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            scrollTo(0, mScroller.getCurrY());
            postInvalidate();
        }
    }
}
```

## 事件拦截分析

在MotionEvent中封装了很多东西，比如获取坐标点event.getX()和getRawX()。MotionEvent提供的手势，常用的有DOWN,UP,MOVE。
一般ViewGroup需要重写三个方法:
```
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return super.onTouchEvent(event);
    }
```
View重写两个：
```
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return super.onTouchEvent(event);
    }
```
默认返回值都为false
事件传递的返回值：true，拦截，不继续；false，不拦截，继续流程
事件处理的返回值：true，处理 ；false，给上级处理

  [1]: http://img-blog.csdn.net/20160308223320045
  [2]: http://img-blog.csdn.net/20160308223337794
  [3]: http://img-blog.csdn.net/20160308224421308