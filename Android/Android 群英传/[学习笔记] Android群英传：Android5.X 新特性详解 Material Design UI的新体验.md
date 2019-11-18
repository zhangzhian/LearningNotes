#[学习笔记] Android群英传：Android5.X 新特性详解 Material Design UI的新体验

主要内容

- Android 5.X UI设计初步
- Android 5.X 新增特性分析

##一. Android 5.X UI设计初步
Android 5.X开始使用新的设计风格Material Design来统一整个Android系统设计风格，与之前的设计不同，这次的Material Design设计将Android带来一片全新的高度，同时Google在官网推出了新的设计指南，全面的讲解了Material Design的整个实现规范

![这里写图片描述](http://img.blog.csdn.net/20160430145458345)

###1.材料的形态模拟
材料的形态模拟是Material Design中最核心也是改变最大的设计，Google通过模拟自然界纸墨的形态变化，光线和阴影，纸和纸之间的空间层次关系，带来一种真实的感觉

![这里写图片描述](http://img.blog.csdn.net/20160430205855893)

###2.更加真实的动画
好的动画效果可以非常的有效的指引用户，暗示用户并给用户带来愉悦的使用体验，Android5.X中大量加入了各种新的动画效果，让整个设计风格更加自然，和谐，而各种新的转场动画，能更加有效的指引用户的视觉焦点，不至于因为复杂布局的界面重排而对整体效果产生影响，让使用者达到一个视觉连贯性

![这里写图片描述](http://img.blog.csdn.net/20160430210203381)

###3.大色块的使用
Material Design中用了大量高饱和度，适中亮度的大色块来突出界面的主次，并一扫Android4.X系列Holo主题的沉重感，让界面更加富有时尚感和视觉冲击力

此外，还有更多的设计风格，比如悬浮按钮，聚焦大图，无框按钮，波纹效果等新特性。

##2.Material Design主题

我们先来看看如何使用主题，MD一共有三种默认的主题可以设置

```xml
@android:style/Theme.Material (dark version)        
@android:style/Theme.Material.Light (light version)     
@android:style/Theme.Material.Light.DarkActionBar   
```

效果如图

![这里写图片描述](http://img.blog.csdn.net/20160430211323091)

同时，Android5.X提出来Color Palette的概念，让开发者可以设定系统区域的颜色，使得整个APP的颜色使得APP的颜色统一

![这里写图片描述](http://img.blog.csdn.net/20160430212301361)

通过如下所示的代码，可以通过自定义style的方式来创建自己的Color palette颜色主题，从而实现颜色的不同风格

```xml
<!-- inherit from the material theme-->
    <style name="AppTheme" parent="android:Theme.Material">
        <!-- Main theme color-->
        <!-- your app branding color for the app bar-->
        <item name="colorPrimary">#BEBEBE</item>
        <!-- derker variant for thr status bar and contextual app bars-->
        <item name="colorPrimaryDark">#FF5AEBFF</item>
        <!--theme ui controls like checkBoxs and text fields-->
        <item name="colorAccent">#FFFF4130</item>

    </style>
```

>看效果

![这里写图片描述](http://img.blog.csdn.net/20160430211705183)

##3.Palette
在Android的版本发展中，UI越来越成为Google的发展中心，这次的Android5.X创新的使用了Palette来提取颜色，从而让主题能够动态适应当前页面的色调，使得整个app的颜色基本和谐统一

Android内置了几种提取颜色的种类

- Vibrant（充满活力的）
- Vibrant dark（充满活力的黑）
- Vibrant light（充满活力的白）
- Muted(柔和的)
- Muted dark(柔和的黑)
- Muted light(柔和的白)

使用Palette的API，能够让我们从Bitmap中获取对应的色调，修改当前的主题色调，使用Palette首先需要在Android studio引用相关的依赖

```
compile 'com.android.support:palette-v7:21.0.+'
```

可以通过传递一个Bitmap对象给Palette，并调用它的Palette.generate()静态方法或者Palette.generateAsync()方法创建一个Palette，接下来，就可以使用getter方法来检索相应的色调，这些色调就是我们在上面列表所列出的色调

例子，演示如何通过加载的图片的柔和色调来改变状态栏和actionbar的色调，代码如下

```java

public class MainActivity extends AppCompatActivity {

    private ImageView iv_palette;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        setPalette();
    }

    /**
     * Palette获取颜色
     */
    private void setPalette() {
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher);
        //创建Palette对象
        Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener() {
            @Override
            public void onGenerated(Palette palette) {
                //通过Palette来获取对应的色调
                Palette.Swatch vibrant = palette.getDarkVibrantSwatch();
                //将颜色设置给相应的组件
                getSupportActionBar().setBackgroundDrawable(new ColorDrawable(vibrant.getRgb()));
                Window window = getWindow();
                window.setStatusBarColor(vibrant.getRgb());
            }
        });
    }
}

```

而且，可以使用不同的方法获取不同的色调颜色

```java
palette.getVibrantSwatch();
palette.getDarkMutedSwatch();
palette.getLightMutedSwatch();
palette.getMutedSwatch();
palette.getDarkVibrantSwatch();
palette.getLightVibrantSwatch();
```

##4.视图与阴影

Material Design的一个很重要的特点就是拟物扁平化，通过展现生活中的材料效果，恰当的使用阴影和光线，配合完美的动画效果，模拟出一个动感十足又美丽大胆的视觉效果

###1.阴影效果

以往的Android View通常有两个属性——X和Y，而在Android5.X中，Google为其增加了一个新的属性——Z，对应垂直方向上的高度变化

在Android 5.X中，View的Z值由两部分组成，elevation和translationZ(他们都是Android5.X新引入的属性)，elevation是静态的成员，translationZ可以在代码中使用来实现动画效果，他们的关系

```java
Z = elevation + translationZ;
```

通过下面的代码，演示了不同视图高度所显示的效果，xml代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="10dp"
        android:background="@mipmap/ic_launcher" />

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="10dp"
        android:background="@mipmap/ic_launcher"
        android:elevation="2dp" />

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="10dp"
        android:background="@mipmap/ic_launcher"
        android:elevation="10dp" />


</LinearLayout>

```

![这里写图片描述](http://img.blog.csdn.net/20160430215158885)

在程序中，我们也可以使用代码改变视图高度

```java
 view.setTranslationZ(xxx);
```

通常也会使用属性动画来为视图高度改变的时候增加动画效果

```java
	if(flag){
            view.animate().translationZ(100);
            flag = false;
        }else {
            view.animate().translationZ(0);
            flag = true;
        }
```

##5.Tinting 和 Clipping

Android5.X在对图像的操作有了更多的功能，下面来看看Android5.X的两个对操作图像的新功能——Tinting（着色） 和 Clipping（裁剪）

###1.Tinting（着色）
Tinting的使用非常的简单，只要在XML中配置好tint和tintMode就可以了，对于配置组合效果，只需要大家实际操作一下，就能非常清晰的理解处理效果，在下面的代码中，设置了几种不同的tint和tintMode效果，XML代码如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher" />

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher"
        android:tint="@android:color/holo_blue_bright" />

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher"
        android:tint="@android:color/holo_blue_bright"
        android:tintMode="add" />


    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher"
        android:tint="@android:color/holo_blue_bright"
        android:tintMode="multiply" />


</LinearLayout>

```

效果如下：

![这里写图片描述](http://img.blog.csdn.net/20160430220325791)

Tint通过修改图像的Alpha遮罩来修改图像的颜色，从而达到重新着色的目的，这一功能在一些图片处理的APP使用起来还是十分的方便的

###2. Clipping（裁剪）

Clipping可以让我们改变一个视图的外形，要使用Clipping，首先需要使用ViewOutlineProvider来修改outline作用给视图

下面这个例子，将一个正方形的textview通过Clipping裁剪成一个圆形的正方形和一个圆，XML代码如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv_rect"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="20dp"
        android:elevation="1dp"
        android:gravity="center" />

    <TextView
        android:id="@+id/tv_circle"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="20dp"
        android:elevation="1dp"
        android:gravity="center" />


</LinearLayout>

```

逻辑代码很简单

```java
    /**
     * Clipping裁剪
     */
    private void setClipping() {
        final View v1 = findViewById(R.id.tv_rect);
        View v2 = findViewById(R.id.tv_circle);

        //获取Outline
        ViewOutlineProvider vlp1 = new ViewOutlineProvider() {
            @Override
            public void getOutline(View view, Outline outline) {
                //修改outline为特定形状
                outline.setRoundRect(0, 0, view.getWidth(), view.getHeight(), 30);
            }
        };

        //获取Outline
        ViewOutlineProvider vlp2 = new ViewOutlineProvider() {
            @Override
            public void getOutline(View view, Outline outline) {
                //修改outline为特定形状
                outline.setOval(0, 0, view.getWidth(), view.getHeight());
            }
        };
        //重新设置形状
        v1.setOutlineProvider(vlp1);
        v2.setOutlineProvider(vlp2);
    }
```

效果

![这里写图片描述](http://img.blog.csdn.net/20160430221640355)

##6.列表和卡片
###1.RecyclerView

RecyclerView是support-v7中的一个新的组件，是一个强大的滑动控件，ViewHolder的封装实现，用户只要实现自己的viewholder就可以了，该组件会自动帮你回收服用的每一个item

要使用RecyclerView，首先需要在项目中引用依赖

```
compile 'com.android.support:recyclerview-v7:21.0.+'
```

在布局中使用RecyclerView与使用ListView基本类似，同样需要一个类似List itemd的布局，在Material Design中，通常与CardView配合使用，后面我们再讲CardView的使用

使用RecyclerView的重点和使用和ListView一样，需要使用一个合适的数据适配器来加载数据，RecyclerView中需要重写的很多方法都似曾相识，不过RecyclerView更加先进的是，它已经封装好了ViewHolder，只要实现功能就可以，，先看Adapter

```java
/**
 * RecyclerView的适配器
 */
public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerAdapter.ViewHolder> {

    private List<String> mData;

    public RecyclerAdapter(List<String> mData) {
        this.mData = mData;
    }

    public AdapterView.OnItemClickListener itemClickListener;

    public void setOnItemClickListener(AdapterView.OnItemClickListener itemClickListener) {
        this.itemClickListener = itemClickListener;
    }

    public interface OnItemClickListener {
        void onItemClic(View view, int position);
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        //将布局转化为View并传递给RecyclerView封装好的ViewHolder
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.rc_item, parent, false);
        return new ViewHolder(v);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        //建立起ViewHolder中视图和数据的关联
        holder.textView.setText(mData.get(position) + position);
    }

    @Override
    public int getItemCount() {
        return mData.size();
    }

    public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {

        private TextView textView;

        public ViewHolder(View itemView) {
            super(itemView);
            textView = (TextView) itemView;
            textView.setOnClickListener(this);
        }

        //通过接口回调来实现RecyclerView的点击事件
        @Override
        public void onClick(View v) {
            if (itemClickListener != null) {
                itemClickListener.onItemClick(null, v, getPosition(), R.id.tv_item);
            }
        }
    }
}

```

上面就是一个非常简单却典型的RecyclerView，通过onCreateViewHolder将List item的布局转换成View,并传递给RecyclerView封装好的ViewHolder，就可以将数据和视图关联起来了，但是有一点要注意的是，Android并没有给RecyclerView增进点击事件，所以需要自己写接口回调，代码如图

```java
 public AdapterView.OnItemClickListener itemClickListener;

    public void setOnItemClickListener(AdapterView.OnItemClickListener itemClickListener) {
        this.itemClickListener = itemClickListener;
    }

    public interface OnItemClickListener {
        void onItemClic(View view, int position);
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        //将布局转化为View并传递给RecyclerView封装好的ViewHolder
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.rc_item, parent, false);
        return new ViewHolder(v);
    }

```

类似ListView的List item视图

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#bebebe"
    android:textSize="40sp"
    android:layout_margin="3dp"
    android:gravity="center"
    android:id="@+id/tv_item"
    android:orientation="vertical">

</TextView>
```

Google在RecyclerView中定义了LayoutManager来帮助开发者更加方便的创建不同但的布局，下面的例子就演示如何创建水平和垂直布局，当然，你也可以通过自定义LayoutManager来创建自己的布局，核心代码如下：

```java
   mRcList.setLayoutManager(new LinearLayoutManager(this));
   mRcList.setLayoutManager(new GridLayoutManager(this));
```

>完整代码：

```java
public class MainActivity extends AppCompatActivity {

    private RecyclerView mRcList;
    private RecyclerAdapter mAdapter;
    private RecyclerView.LayoutManager mLayoutManager;
    private Spinner mSpinner;
    private List<String>mData = new ArrayList<String>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mRcList = (RecyclerView) findViewById(R.id.mRcList);
        mLayoutManager = new LinearLayoutManager(this);
        mRcList.setLayoutManager(mLayoutManager);
        mRcList.setHasFixedSize(true);
        //设置显示动画
        mRcList.setItemAnimator(new DefaultItemAnimator());

        mSpinner = (Spinner) findViewById(R.id.spinner);
        mSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                if(position == 0){
                    //设置为线性布局
                    mRcList.setLayoutManager(new LinearLayoutManager(MainActivity.this));
                }else if(position == 1){
                    //设置为表格布局
                    mRcList.setLayoutManager(new GridLayoutManager(MainActivity.this,3));
                }else if(position == 2 ){

                }
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });
        //增加测试数据
        mData.add("Android Test1");
        mData.add("Android Test2");
        mData.add("Android Test3");

        mAdapter = new RecyclerAdapter(mData);
        mRcList.setAdapter(mAdapter);

        mAdapter.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(final AdapterView<?> parent, View view, int position, long id) {
                //设置点击动画
                parent.animate().translationZ(15f).setDuration(300).setListener(new AnimatorListenerAdapter() {
                    @Override
                    public void onAnimationEnd(Animator animation) {
                        super.onAnimationEnd(animation);
                        parent.animate().translationZ(1f).setDuration(500).start();
                    }
                }).start();
            }
        });
    }

    public void addRecycler(View view){
        int position = mData.size();
        if(position>0){
            mAdapter.notifyDataSetChanged();
        }
    }

    public void delRecycler(View view){
        int position = mData.size();
        if(position>0){
            mData.remove(position-1);
            mAdapter.notifyDataSetChanged();
        }
    }
}

```

运行结果

![这里写图片描述](http://img.blog.csdn.net/20160430231233968)


###2.CardView

CardView曾经开始流行在Google+，后来越来越多的APP也引入了Card这样的布局方式，说到底，CardView也是一个容器布局，只是他提供了一种卡片的形式，Google所幸提供了一个CardView控件，方便大家使用这种布局，开发者可以设置大小和视图高度，圆角的角度等，在此之前，我们要添加依赖

```
 compile 'com.android.support:cardview-v7:21.0.0+'
```

同时要添加命名空间

```xml
xmlns:app="http://schemas.android.com/apk/res-auto"
```

举个例子

```xml
 <android.support.v7.widget.CardView xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/cardview"
        android:layout_width="match_parent"
        android:layout_height="80dp"
        android:layout_margin="8dp"
        app:cardBackgroundColor="@color/colorAccent"
        app:cardCornerRadius="8dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="TextView in CardView"
            android:textColor="@android:color/white"
            android:textSize="26sp" />
    </android.support.v7.widget.CardView>

```

运行的效果

![这里写图片描述](http://img.blog.csdn.net/20160430233232455)


##7.Activity过渡动画

曾经的Android在activity进行跳转的时候，只是很生硬的切换，即使通过 overridePendingTransition( int inId, int outId)这个方法来给Activity增加一些切换动画，效果也只是差强人意，而在Android5.X中，Google对动画效果进行了更深一步的诠释，为Activity的转场效果设计了更加丰富的动画效果

Android5.X提供了三种Transition类型

- 进入：一个进入的过渡动画决定Activity中的所有视图怎么进入屏幕
- 退出：一个退出的过渡动画决定Activity中的所有视图怎么退出屏幕
- 共享元素：一个共享元素过渡动画决定两个Activity之间的过渡，怎么共享他们的视图

其中，进人和退出效果包括

- explode(分解)一一从屏幕中间进或出,移动视图
- slide(滑动)——从屏幕边缘进或出，移动视图
- fade(淡出) 一一通过改变屏幕上视图的不透明度达到添加或者移除视图

共享元素包括

-  changeBounds——改变目标视图的布局边界
-  changeCliBounds——裁剪目标视图边界
-  changeTransfrom——改变目标视图的缩放比例和旋转角度
-  changeImageTransfrom——改变目标图片的大小和缩放比例

在Android5.X上，动画效果的种类变得更加丰富了

首先来看看普通的三种Activity过渡动画，要使用这些动画非常简单，例如从ActivityA转到ActivityB，只需要在ActivityA中将基本的startActivity(intent)方法改为如下代码即可

```java
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(MainActivity.this).toBundle());
```
而在AchvityB中，只需要设置下如下所示代码

```java
 getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
```

或者在样式文件中设置如下所示代码

```xml
 <item name="android:windowContentTransitions">true</item>
```

那么接下来就可以设置进人/退出ActivityB的具体的动画效果了， 代码如下所示

```java
getWindow().setEnterTransition(new Explode());
getWindow().setEnterTransition(new Slide());
getWindow().setEnterTransition(new Fade());
```

而对于共享元素的动画效果.可以借用开发者网上的一张图

![这里写图片描述](http://img.blog.csdn.net/20160501234929425)

要想在程序中使用共享元素的动画效果也很简单，首先需要在他的activity1布局中设置共享元素，增加元素代码

```xml
android:transitionName="XXX"
```
同时在activity2中，也增加一个相应的共享元素属性，如果只要一个共享元素，那么在activity1中可以这样写

```java
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this,view,"share").toBundle());
```

使用的参数就是前面普通动画的基础上增加了共享的的view和前面取的名字，如果由多个共享元素，那么我们可以通过

```java
 Pair.create()
```

来创建多个共享元素

```java
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, Pair.create(view,"share"),Pair.create(fab,"fab")).toBundle());
```

下面通过一个具体的实例来演示一下过渡动画，我们从BActivity跳转到CActivity

>BActivity的XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="explode"
        android:text="explode" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="slide"
        android:text="slide" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="fade"
        android:text="fade" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="share"
        android:text="share"
        android:transitionName="share" />

    <Button
        android:id="@+id/fab_button"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:background="@mipmap/ic_launcher"
        android:elevation="5dp"
        android:onClick="explode"
        android:transitionName="fab" />

</LinearLayout>
```

>实现逻辑

```java
/**
 * BActivity
 */
public class BActivity extends AppCompatActivity {

    private Intent intent;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_b);


    }

    //设置不同的动画效果
    public void explode(View view) {
        intent = new Intent(this, CActivity.class);
        intent.putExtra("flag", 0);
        startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this).toBundle());
    }

    //设置不同的动画效果
    public void slide(View view) {
        intent = new Intent(this, CActivity.class);
        intent.putExtra("flag", 1);
        startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this).toBundle());
    }

    //设置不同的动画效果
    public void fade(View view) {
        intent = new Intent(this, CActivity.class);
        intent.putExtra("flag", 2);
        startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this).toBundle());
    }

    //设置不同的动画效果
    public void share(View view) {
        View fab = findViewById(R.id.fab_button);
        intent = new Intent(this, CActivity.class);
        intent.putExtra("flag", 3);
        //创建单个共享元素
        startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, view, "share").toBundle());
        //创建多个共享单元
        startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, Pair.create(view, "share"), Pair.create(fab, "fab")).toBundle());
    }


}

```

再看CActivity的XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="15dp">

    <View
        android:id="@+id/holder_view"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:background="?android:colorPrimary"
        android:transitionName="share" />

    <Button xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/fab_button"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:layout_alignParentEnd="true"
        android:layout_below="@id/holder_view"
        android:layout_marginRight="15dp"
        android:layout_marginTop="-26dp"
        android:background="@mipmap/ic_launcher"
        android:elevation="5dp"
        android:transitionName="fab" />

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/holder_view"
        android:paddingTop="10dp">

        <Button
            android:id="@+id/button1"
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:layout_alignParentStart="true"
            android:layout_marginTop="10dp" />

        <Button
            android:id="@+id/button"
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:layout_below="@id/button1"
            android:layout_marginTop="10dp" />
    </RelativeLayout>

</RelativeLayout>
```

实现逻辑就更简单了

```java

/**
 * CActivity
 */
public class CActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
        int flag = getIntent().getExtras().getInt("flag");
        switch (flag) {
            case 0:
                getWindow().setEnterTransition(new Explode());
                break;
            case 1:
                getWindow().setEnterTransition(new Slide());
                break;
            case 2:
                getWindow().setEnterTransition(new Fade());
                getWindow().setExitTransition(new Fade());
                break;
        }
        setContentView(R.layout.activity_c);
    }
}

```

我们运行一下就可以看到自己想要的效果了

![这里写图片描述](http://img.blog.csdn.net/20160502100708427)


##八.Material Design动画效果

动画已经成了UI设计中一个非常重要的一个组成部分，在Android5.X的Material Design中，更是使用了大量的动画效果，同时Google官方文档也有详细的说明

###1.Ripple效果
在Android5.X中，Material Design大量的使用了Ripple动画，即点就博文效果，可以通过如下代码设置波纹的背景

```xml
//波纹有边界
 android:background="?android:attr/selectableItemBackground"
 //波纹无边界
 android:background="?android:attr/selectableItemBackgroundBorderless"
```

波纹有边界是指波纹被限制在控件的边界中，而波纹超出边界，则是不会受到控件的限制，以圆形发散出去，我们做一个演示

XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/holo_blue_light"
    android:gravity="center"
    android:orientation="vertical">

    <Button
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="?android:attr/selectableItemBackground"
        android:text="有界波纹"
        android:textColor="@android:color/white" />


    <Button
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="?android:attr/selectableItemBackgroundBorderless"
        android:text="无界波纹"
        android:textColor="@android:color/white" />

</LinearLayout>
```

运行的效果

![这里写图片描述](http://img.blog.csdn.net/20160502103354437)

同样的，我们可以写一个xml文件来实现Ripple效果

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@android:color/holo_blue_dark">
    <item>
        <shape android:shape="oval">
            <solid android:color="@color/colorPrimary" />
        </shape>
    </item>
</ripple>
```

使用方法直接设置背景就可以了，运行效果

![这里写图片描述](http://img.blog.csdn.net/20160502103808388)


###2.Circular Reveal
这个动画效果是在Google I/O 大会上演示了好多次的，具体变现为一个View以圆形的形式展开，揭示出来，通过ViewAnimationUtils.createCircularReveal()来创建动画，代码如下：

```java
public static Animator createCircularReveal(View view, int centerX, int centerY, float startRadius, float endRadius) {

        return new RevealAnimator(view,centerX,centerY,startRadius,endRadius);
    }
```

RevealAnimator的使用特别简单，主要就是几个关键的坐标点

- centerX 动画开始的中心点X
- centerY 动画开始的中心点Y
- startRadius 动画开始半径
- endRadius   动画结束半径

通过一个实例去演示一下

XML

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <ImageView
        android:id="@+id/oval"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@mipmap/ic_launcher" />

    <ImageView
        android:id="@+id/rect"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@mipmap/ic_launcher" />

</LinearLayout>
```

逻辑代码

```java

/**
 * Circular Reveal
 */
public class CirActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_cir);

        final View oval = findViewById(R.id.oval);
        oval.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Animator animator = ViewAnimationUtils.createCircularReveal(oval, oval.getWidth() / 2, oval.getHeight() / 2, oval.getWidth(), 0);
                animator.setInterpolator(new AccelerateDecelerateInterpolator());
                animator.setDuration(2000);
                animator.start();
            }
        });


        final View rect = findViewById(R.id.rect);
        rect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Animator animator = ViewAnimationUtils.createCircularReveal(rect, 0, 0, 0, (float) Math.hypot(rect.getWidth(), rect.getHeight()));
                animator.setInterpolator(new AccelerateInterpolator());
                animator.setDuration(2000);
                animator.start();
            }
        });
    }
}

```

![这里写图片描述](http://img.blog.csdn.net/20160502105927354)


###3. View state changes Animation
在Android5.X中，系统提供了视图与状态改变来设置一个视图状态的切换

- StaetListAnimator

StaetListAnimator作为视图改变时的动画效果，通常会使用Seletor来进行设置，但是以前我们设置Seletor的时候，通常是修改他的背景来达到反馈的效果，但是再现在Android5.X中就不需要这样了，可以使用动画来实现，我们用小例子来具体看看怎么实现的吧

在XML中定义一个StaetListAnimator

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:duration="2000" android:property="rotationX" android:valueTo="360" android:valuyeType="floatType" />
        </set>
    </item>

    <item android:state_pressed="false">
        <set>
            <objectAnimator android:duration="2000" android:property="rotationX" android:valueTo="0" android:valuyeType="floatType" />
        </set>
    </item>
</selector>
```

然后直接在布局中设置即可

```xml
 <Button
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:background="@drawable/animatorstate" />
```


- animated-selector
animated-selector同样是一个改变动画效果的动画，


##九.Toolbar
Toolbar和actionBar最大的区别就是Toolbar更加的自由，可控，这也是Google在逐渐使用Toolbar来取代actionBar的原因，要使用Toolbar就必须引用依赖

```
compile 'com.android.support:appcompat-v7:23.3.0'
```

同样的，我们要设置主题

```
Theme.AppCompat.Light.NoActionBar
```

然后在XML中

```java
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/colorPrimaryDark"/>
```

在代码中获取

```java
		 mToolbar = (Toolbar) findViewById(R.id.toolbar);
        mToolbar.setLogo(R.mipmap.ic_launcher);
        mToolbar.setTitle("主标题");
        mToolbar.setSubtitle("副标题");
        setSupportActionBar(mToolbar);
```

而menu都是一样的，就不写了，这样一个title就写出来了

![这里写图片描述](http://img.blog.csdn.net/20160502113126051)

具体实现以下，加入一个侧滑的效果，XML代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/colorPrimaryDark" />

    <android.support.v4.widget.DrawerLayout
        android:id="@+id/drawer"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!--主页内容-->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/holo_blue_bright"
            android:orientation="vertical">

            <Button
                android:layout_width="100dp"
                android:layout_height="match_parent"
                android:text="内容界面" />

        </LinearLayout>


        <!--侧滑菜单内容，必须制定水平重力-->
        <LinearLayout
            android:id="@+id/drawer_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="start"
            android:orientation="vertical">

            <Button
                android:layout_width="100dp"
                android:layout_height="match_parent"
                android:text="菜单界面" />

        </LinearLayout>


    </android.support.v4.widget.DrawerLayout>

</LinearLayout>
```


然后设置一下

```java
 mToolbar = (Toolbar) findViewById(R.id.toolbar);
        mToolbar.setLogo(R.mipmap.ic_launcher);
        mToolbar.setTitle("主标题");
        mToolbar.setSubtitle("副标题");
        setSupportActionBar(mToolbar);

        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer);
        mDrawerToggle = new ActionBarDrawerToggle(this,mDrawerLayout,mToolbar,0,0);
        mDrawerToggle.syncState();
        mDrawerLayout.setDrawerListener(mDrawerToggle);
```

运行的效果

![这里写图片描述](http://img.blog.csdn.net/20160502114128620)


##十.Notification

Notification作为一个时间触发性的交互提示接口，让我们获得消息的时候，在状态栏，锁屏界面显示相应的消息

Google在Android5.X中，又进一步的改进了通知栏，优化了Notification，在5.X设备上一个标准的通知是这样的，长按是下图

![这里写图片描述](http://img.blog.csdn.net/20160502115934511)

长按会显示消息的来源

Notification会有一个从白色到灰色的动画切换效果，最终显示发出这个通知的调用者，而在我们的锁屏界面，也可以看到



![这里写图片描述](http://img.blog.csdn.net/20160502120144877)

我们分四重境界来讲解在Android5.X下使用Notification

###1.基本的Notification
通过Notification.Builder创建一个Notification的builder，代码如下

```java
 Notification.Builder builder = new Notification.Builder(this);
```

这个与AlertDialog的使用方法非常的相似，接下来，要点击Notification执行一个intent，由于这个intent的不是立即执行，而是用户触发的，所以用pendingintent来完成这个延时操作，代码如下：

```java
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.baidu.com"));
//构造pendingdintent
PendingIntent pendingIntent = PendingIntent.getActivities(this,0,intent,0);
```

这样点击PendingIntent 之后就会触发时间了，我们也可以给他增加属性

```java
		builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentIntent(pendingIntent);
        builder.setAutoCancel(true);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher));
        builder.setContentText("Title");
        builder.setContentText("内容");
        builder.setSubText("text");
```

通过下面的代码发布通知栏

```java
 //通过NotificationManager来发出
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        notificationManager.notify(0,builder.build());
```

这样我们很轻松的就创建了一个通知栏

![这里写图片描述](http://img.blog.csdn.net/20160502121915412)

###2.折叠式Notification

折叠式Notification也是一种自定义视图的Notification，常常用于显示文本，他拥有两个视图状态，我们可以用RemoteView来帮助我们创建一个Notification视图，代码如下：

```java
		//通过RemoteViews创建自定义视图
        RemoteViews contentView = new RemoteViews(getPackageName(),R.layout.notification);
        contentView.setTextViewText(R.id.textView,"通知栏");
```

其中notification的布局是这样的

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:textColor="#ff43aebe" />

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher" />

</LinearLayout>
```

通过如下代码，可以讲一个视图指定为Notification正常状态下的视图

```
  notification.contentView = contentView;
```

另一个展开的代码

```
notification.bigContentView = expandedView;
```

完整代码

```java

/**
 * Notification
 * Created by LGL on 2016/5/2.
 */
public class NotificationActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_notification);

        Notification.Builder builder = new Notification.Builder(this);

        Intent intents = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.baidu.com"));
        //构造pendingdintent
        PendingIntent pendingIntent = PendingIntent.getActivities(this, 0, new Intent[]{intents}, 0);

        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentIntent(pendingIntent);
        builder.setAutoCancel(true);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));

        //通过RemoteViews创建自定义视图
        RemoteViews contentView = new RemoteViews(getPackageName(), R.layout.notification);
        contentView.setTextViewText(R.id.textView, "通知栏");


        Notification notification = builder.build();
        //指定视图
        notification.contentView = contentView;

        RemoteViews expandedView = new RemoteViews(getPackageName(), R.layout.notification_expanded);

        notification.bigContentView = expandedView;


        //通过NotificationManager来发出
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        notificationManager.notify(0, builder.build());


    }
}

```

###3.悬挂式Notification
悬挂式Notification是Android5.X新增加的方式，Google希望通过这个方式来给用户带来更好的体验，这种被称为Headsup的Notification方式，可以在屏幕的上方产生Notification而不打扰用户的操作

在Android Sample中，Google给我们展示了这个项目，代码如下：

```java
Notification.Builder builder = new Notification.Builder(this).setSmallIcon(R.mipmap.ic_launcher).setPriority(Notification.PRIORITY_DEFAULT).setCategory(Notification.CATEGORY_MESSAGE).setContentTitle("Headsup Notification").setContentText("I am Headsup Notification");
        Intent push = new Intent();
        push.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        push.setClass(this,MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivities(this,0, new Intent[]{push},PendingIntent.FLAG_CANCEL_CURRENT);
        builder.setContentText("Android 5.X Headsup Notification").setFullScreenIntent(pendingIntent,true);
        NotificationManager nm = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        nm.notify(0,builder.build());

```

效果

![这里写图片描述](http://img.blog.csdn.net/20160502131330033)


###4.显示等级的Notification

>最后一重境界也就是新加入的一种模式，显示登记，具体分三级

- VISIBILITY_PRIVATE -没有锁屏的时候显示
- VISIBILITY_PUBLIC -  表明在任何情况下都会显示
- VISIBILITY_SECRET - 表明在pin,password等安全锁和没有锁屏的情况下显示
