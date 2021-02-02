# 开发者营地|RecyclerView：详解

## 前言

RecyclerView是谷歌官方出的一个用于大量数据展示的新控件，可以用来代替传统的ListView，更加强大和灵活。

现阶段基本上已经使用RecyclerView替代了ListView。

本文先简单聊聊ListView，再分析RecyclerView的使用和缓存机制。

## ListView

### 1. 简介

ListView是`Android`中的一种列表视图组件，继承自`AdapterView`抽象类，集合多个`Item`以列表的形式展示给客户。

类图关系如下：

![img](https://upload-images.jianshu.io/upload_images/944365-a6f76b5bc3010aa0.png?imageMogr2/auto-orient/strip|imageView2/2/w/660/format/webp)

`ListView`仅作为容器（列表），用于装载和显示数据（即列表项`Item`），而容器内的具体数据（列表项`Item`）则是由 适配器（`Adapter`）提供。

> 适配器：作为`View` 和 数据之间的桥梁中介，将数据映射到要展示的`View`中

当需显示数据时，`ListView`会向`Adapter`取出数据，从而加载显示，具体如下图：

![img](https://upload-images.jianshu.io/upload_images/944365-9337bc62b916d9de.png?imageMogr2/auto-orient/strip|imageView2/2/w/750/format/webp)

总的来说，`ListView`负责以列表的形式显示`Adapter`提供的内容。

**缓存原理**

为了节省空间和时间，`ListView`不会为每一个数据创建一个视图，而是采用了Recycler组件，用于回收和复用 `View`

当屏幕需显示`x`个`Item`时，那么`ListView`会创建 `x+1`个视图；当第1个`Item`离开屏幕时，此`Item`的`View`被回收至缓存，入屏的`Item`的`View`会优先从该缓存中获取。

> 只有`Item`完全离开屏幕后才可复用，这也是为什么`ListView`要创建比屏幕需显示视图多1个的原因：缓冲显示视图x
>
> 即：第1个`Item`离开屏幕是有过程的，会有1个 **第1个Item的下半部分和第x+1个Item上半部分同时在屏幕中显示**的状态，此时仍无法使用缓存的`View`，只能继续用新创建的视图`View`

### 2.  使用

生成列表视图（ListView）的方式主要有两种：

- 直接用ListView进行创建
- 让Activity继承ListActivity

**xml文件配置信息**

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"   
    xmlns:tools="http://schemas.android.com/tools"   
    android:layout_width="match_parent"   
    android:layout_height="match_parent"   
    android:background="#FFE1FF"   
    android:orientation="vertical" >   
    <ListView   
        android:id="@+id/listView1"   
        android:layout_width="match_parent"   
        android:layout_height="match_parent" />   
</LinearLayout>  
```

AbsListView的常用属性和相关方法：

| 属性                       |                             说明                             |                                                         备注 |
| -------------------------- | :----------------------------------------------------------: | -----------------------------------------------------------: |
| android:choiceMode         |            列表的选择行为，默认：none没有选择行为            | 选择方式： none：不显示任何选中项  singleChoice：允许单选multipleChoice：允许多选multipleChoiceModal：允许多选 （把Activity里面adapter的第二个参数改成支持选择的布局） |
| android:drawSelectorOnTop  |                                                              |             如果该属性设置为true，选中的列表项将会显示在上面 |
| android:listSelector       |                    为点击到的Item设置图片                    |             如果该属性设置为true，选中的列表项将会显示在上面 |
| android：fastScrollEnabled |                     设置是否允许快速滚动                     | 如果该属性设置为true，将会显示滚动图标，并允许用户拖动该滚动图标进行快速滚动。 |
| android：listSelector      |              指定被选中的列表项上绘制的Drawable              |                                                              |
| android：scrollingCache    |                      滚动时是否使用缓存                      |                       如果设置为true，则在滚动时将会使用缓存 |
| android：stackFromBottom   |                 设置是否从底端开始排列列表项                 |                                                              |
| android：transcriptMode    | 指定列表添加新的选项的时候，是否自动滑动到底部，显示新的选项。 | disabled：取消transcriptMode模式；默认的normal：当接受到数据集合改变的通知，并且仅仅当最后一个选项已经显示在屏幕的时候，自动滑动到底部。 alwaysScroll：无论当前列表显示什么选项，列表将会自动滑动到底部显示最新的选项。 |

Listview提供的XML属性：

| XML属性                       |                             说明                             |                                      备注 |
| ----------------------------- | :----------------------------------------------------------: | ----------------------------------------: |
| android:divider               | 设置List列表项的分隔条（可用颜色分割，也可用图片（Drawable）分割 | 不设置列表之间的分割线，可设置属性为@null |
| android:dividerHeight         |                     用于设置分隔条的高度                     |                                           |
| android:background属性        |                        设置列表的背景                        |                                           |
| android：entries              |   指定一个数组资源，Android将根据该数组资源来生成ListView    |                                           |
| android：footerDividerEnabled |       如果设置成false，则不在footer View之前绘制分隔条       |                                           |
| andorid：headerDividerEnabled |       如果设置成false，则不再header View之前绘制分隔条       |                                           |

### 3. Adapter简介

Adapter本身是一个接口，Adapter接口及其子类的继承关系如下图：

![img](https://upload-images.jianshu.io/upload_images/944365-078700715f8460d7.png?imageMogr2/auto-orient/strip|imageView2/2/w/636/format/webp)

Adapter接口派生了ListAdapter和SpinnerAdapter两个子接口。

> 其中ListAdapter为AbsAdapter提供列表项，而SpinnerAdapter为AbsSpinner提供列表项

ArrayAdapter、SimpleAdapter、SimpleCursorAdapter、BaseAdapter都是常用的实现适配器的类。

- ArrayAdapter：简单、易用的Adapter，用于将数组绑定为列表项的数据源，支持泛型操作

- SimpleAdapter：功能强大的Adapter，用于将XML中控件绑定为列表项的数据源

- SimpleCursorAdapter：与SimpleAdapter类似，用于绑定游标（直接从数据数取出数据）作为列表项的数据源

- BaseAdapter：可自定义ListView，通用用于被扩展。扩展BaseAdapter可以对各个列表项进行最大程度的定制。

接下来以最常用的BaseAdapter进行介绍。

### 4. BaseAdapter

可自定义ListView，通用用于被扩展。扩展BaseAdapter可以对各个列表项进行最大程度的定制。

**使用步骤：**

1. 定义主xml布局
2. 根据需要定义ListView每行所实现的xml布局
3. 定义一个Adapter类继承BaseAdapter，重写里面的方法。
4. 定义一个HashMap构成的列表，将数据以键值对的方式存放在里面。
5. 构造Adapter对象，设置适配器。
6. 将LsitView绑定到Adapter上。

**先定义一个Adapter类继承BaseAdapter，并重写里面的方法**

> 使用BaseAdapter必须写一个类继承它，同时BaseAdapter是一个抽象类，继承它必须实现它的方法。

```java
class MyAdapter extends BaseAdapter {
    private LayoutInflater mInflater;//得到一个LayoutInfalter对象用来导入布局

 //构造函数
    public MyAdapter(Context context,ArrayList<HashMap<String, Object>> listItem) {
        this.mInflater = LayoutInflater.from(context);
        this.listItem = listItem;
    }//声明构造函数

    @Override
    public int getCount() {
        return listItem.size();
    }//这个方法返回了在适配器中所代表的数据集合的条目数

    @Override
    public Object getItem(int position) {
        return listItem.get(position);
    }//这个方法返回了数据集合中与指定索引position对应的数据项

    @Override
    public long getItemId(int position) {
        return position;
    }//这个方法返回了在列表中与指定索引对应的行id


    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        return null;
    }//这个方法返回了指定索引对应的数据项的视图，还没写完
}
```

其中，重点讲解重写的`getView`方式：

```java
    static class ViewHolder

        {
            public ImageView img;
            public TextView title;
            public TextView text;
            public Button btn;
        }

    @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            ViewHolder holder ;
            if(convertView == null)
            {
                holder = new ViewHolder();
                convertView = mInflater.inflate(R.layout.item, null);
                holder.img = (ImageView)convertView.findViewById(R.id.ItemImage);
                holder.title = (TextView)convertView.findViewById(R.id.ItemTitle);
                holder.text = (TextView)convertView.findViewById(R.id.ItemText);
                holder.btn = (Button) convertView.findViewById(R.id.ItemBottom);
                convertView.setTag(holder);
            }
            else {
                holder = (ViewHolder)convertView.getTag();

            }
            holder.img.setImageResource((Integer) listItem.get(position).get("ItemImage"));
            holder.title.setText((String) listItem.get(position).get("ItemTitle"));
            holder.text.setText((String) listItem.get(position).get("ItemText"));

            return convertView;
        }
```

  * 重写方式：使用ViewHolder实现更加具体的缓存：View组件缓存
  * 具体原理：通过`setTag()`设置将一个ViewHolder绑定到 convertView 作为 `getView()`的输入参数和返回参数，从而形成了Adapter的itemView重用机制，减少了重绘View的次数
  * 优点：重用View时就不用通过 `findViewById()` 重新寻找View组件；同时也减少了重绘View的次数，是ListView使用的最优化方案

### 5. 完整示例

定义主xml的布局activity_main.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:orientation="vertical" >
    <ListView
        android:id="@+id/listView1"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

定义ListView每行所实现的xml布局（item布局）：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent" 
android:layout_height="match_parent">
    <ImageView
        android:layout_alignParentRight="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/ItemImage"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="按钮"
        android:id="@+id/ItemBottom"
        android:focusable="false"
        android:layout_toLeftOf="@+id/ItemImage" />
    <TextView android:id="@+id/ItemTitle"
        android:layout_height="wrap_content"
        android:layout_width="fill_parent"
        android:textSize="20sp"/>
    <TextView android:id="@+id/ItemText"
        android:layout_height="wrap_content"
        android:layout_width="fill_parent"
        android:layout_below="@+id/ItemTitle"/>
</RelativeLayout>
```

定义一个Adapter类继承BaseAdapter，重写里面的方法：

```java
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.HashMap;

class MyAdapter extends BaseAdapter {
    private LayoutInflater mInflater;//得到一个LayoutInfalter对象用来导入布局 
    ArrayList<HashMap<String, Object>> listItem;

    public MyAdapter(Context context,ArrayList<HashMap<String, Object>> listItem) {
        this.mInflater = LayoutInflater.from(context);
        this.listItem = listItem;
    }//声明构造函数

    @Override
    public int getCount() {
        return listItem.size();
    }//这个方法返回了在适配器中所代表的数据集合的条目数

    @Override
    public Object getItem(int position) {
        return listItem.get(position);
    }//这个方法返回了数据集合中与指定索引position对应的数据项

    @Override
    public long getItemId(int position) {
        return position;
    }//这个方法返回了在列表中与指定索引对应的行id

//利用convertView+ViewHolder来重写getView()
    static class ViewHolder
    {
        public ImageView img;
        public TextView title;
        public TextView text;
        public Button btn;
    }//声明一个外部静态类
    @Override
    public View getView(final int position, View convertView, final ViewGroup parent) {
        ViewHolder holder ;
        if(convertView == null)
        {
            holder = new ViewHolder();
            convertView = mInflater.inflate(R.layout.item, null);
            holder.img = (ImageView)convertView.findViewById(R.id.ItemImage);
            holder.title = (TextView)convertView.findViewById(R.id.ItemTitle);
            holder.text = (TextView)convertView.findViewById(R.id.ItemText);
            holder.btn = (Button) convertView.findViewById(R.id.ItemBottom);
            convertView.setTag(holder);
        }
        else {
            holder = (ViewHolder)convertView.getTag();

        }
        holder.img.setImageResource((Integer) listItem.get(position).get("ItemImage"));
        holder.title.setText((String) listItem.get(position).get("ItemTitle"));
        holder.text.setText((String) listItem.get(position).get("ItemText"));
        holder.btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                System.out.println("你点击了选项"+position);//bottom会覆盖item的焦点，所以要在xml里面配置android:focusable="false"
            }
        });

        return convertView;
    }//这个方法返回了指定索引对应的数据项的视图
}
```

在MainActivity里：

- 定义一个HashMap构成的列表，将数据以键值对的方式存放在里面
- 构造Adapter对象，设置适配器
- 将LsitView绑定到Adapter上

```java
package scut.learnlistview;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SimpleAdapter;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    private ListView lv;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        lv = (ListView) findViewById(R.id.listView1);
        //定义一个以HashMap为内容的动态数组
        ArrayList<HashMap<String, Object>> listItem = new ArrayList<HashMap<String, Object>>();
        //在数组中存放数据
        for (int i = 0; i < 100; i++) {
            HashMap<String, Object> map = new HashMap<String, Object>();
            map.put("ItemImage", R.mipmap.ic_launcher);//加入图片
            map.put("ItemTitle", "第" + i + "行");
            map.put("ItemText", "这是第" + i + "行");
            listItem.add(map);
        }
        MyAdapter adapter = new MyAdapter(this, listItem);
        lv.setAdapter(adapter);//为ListView绑定适配器

        lv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> arg0, View arg1, int arg2, long arg3) {
                System.out.println("你点击了第" + arg2 + "行");//设置系统输出点击的行
            }
        });

}
}
```

### 6. 进阶使用

步骤1：添加头部 / 尾部视图header_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="70dp"
    android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:text="header"
        android:textSize="20dp"
        android:gravity="center"/>

</LinearLayout>
```

步骤2：添加到ListView中

```java
private ListView lv;

View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.header_view, null);

lv.addHeaderView(view);
// lv.addFooterView(view); // 添加到底部View
```

## RecyclerView

### 1. 基础用法

RecyclerView是Android一个更强大的控件,其不仅可以实现和ListView同样的效果，更加强大和灵活。

`RecyclerView`属于新增的控件，若要使用RecyclerView,第一步是要在`build.gradle`中添加对应的依赖库。

> 现阶段已经移到AndroidX中

activity_main.xml：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
    />
</LinearLayout>
```

> 由于`RecyclerView`不是内置在系统SDK中,需要把其完整的包名路径写出来

Fruit.java

```java
public class Fruit {

    private String name;
    private int imageId;

    public Fruit(String name, int imageId){
        this.name = name;
        this.imageId = imageId;

    }

    public String getName() {
        return name;
    }

    public int getImageId() {
        return imageId;
    }
}
```

fruit_item.xml

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"

    >
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/fruit_image"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/fruitname"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

新增适配器 FruitAdapter：为`RecyclerView`新增适配器`FruitAdapter`,并让其继承于`RecyclerView.Adapter`,把泛型指定为`FruitAdapter.ViewHolder`。

```java
public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {

    private  List<Fruit> mFruitList;
    static class ViewHolder extends RecyclerView.ViewHolder{
        ImageView fruitImage;
        TextView fruitName;

        public ViewHolder (View view)
        {
            super(view);
            fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
            fruitName = (TextView) view.findViewById(R.id.fruitname);
        }

    }

    public  FruitAdapter (List <Fruit> fruitList){
        mFruitList = fruitList;
    }

    @Override

    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType){
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.fruit_item,parent,false);
        ViewHolder holder = new ViewHolder(view);
        return holder;
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position){
		Fruit fruit = mFruitList.get(position);
        holder.fruitImage.setImageResource(fruit.getImageId());
        holder.fruitName.setText(fruit.getName());
    }

    @Override
    public int getItemCount(){
        return mFruitList.size();
    }
```

- 定义内部类`ViewHolder`,并继承`RecyclerView.ViewHolder`。传入的View参数通常是RecyclerView子项的最外层布局。

- FruitAdapter构造函数,用于把要展示的数据源传入,并赋予值给全局变量mFruitList。

- FruitAdapter继承`RecyclerView.Adapter`。因为必须重写`onCreateViewHolder()`,`onBindViewHolder()`和`getItemCount()`三个方法
  - `onCreateViewHolder()`用于创建ViewHolder实例,并把加载的布局传入到构造函数去,再把ViewHolder实例返回。
  - `onBindViewHolder()`则是用于对子项的数据进行赋值,会在每个子项被滚动到屏幕内时执行。`position`得到当前项的Fruit实例。
  - `getItemCount()`返回RecyclerView的子项数目

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private List<Fruit> fruitList = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initFruits();
        RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
    	//GridLayoutManager layoutManager = new GridLayoutManager(this,5);//网格
        //StaggeredGridLayoutManager layoutManager = 
        //new StaggeredGridLayoutManager(3,StaggeredGridLayoutManager.VERTICAL);//瀑布流
       	//layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);//横向
        recyclerView.setLayoutManager(layoutManager);
        FruitAdapter adapter = new FruitAdapter(fruitList);
        recyclerView.setAdapter(adapter);
        //设置分隔线  
        //recyclerView.addItemDecoration( new DividerGridItemDecoration(this ));  
        //设置增加或删除条目的动画  
        //recyclerView.setItemAnimator( new DefaultItemAnimator());  
    }

    private void initFruits() {
		Fruit apple = new Fruit("Apple", R.drawable.apple_pic);
        fruitList.add(apple);
        ...
    }
}
```

`LayoutManager`用于指定RecyclerView的布局方式。`LinearLayoutManager`指的是线性布局。`RecyclerView`还提供了`GridLayoutManager`(网格布局)和`StaggeredGridLayoutManager`(瀑布流布局)

RecyclerView 的点击事件，修改FruitAdapter.java：

```java
public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {

    ...
	
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType){
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.fruit_item,parent,false);
        final ViewHolder holder = new ViewHolder(view);
        holder.fruitView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                int position = holder.getAdapterPosition();
                Fruit fruit = mFruitList.get(position);
                Toast.makeText(view.getContext(), "you clicked view" + fruit.getName(), Toast.LENGTH_SHORT).show();
            }
        });

        holder.fruitImage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                int position = holder.getAdapterPosition();
                Fruit fruit = mFruitList.get(position);
                Toast.makeText(view.getContext(), "you clicked image" + fruit.getName(), Toast.LENGTH_SHORT).show();
            }
        });
        return holder;
    }

  ...
}
```

### 2. 优缺点

- RecyclerView封装了ViewHolder的回收复用，也就是说RecyclerView标准化了ViewHolder，编写Adapter面向的是ViewHolder而不再是View了，复用的逻辑被封装了，写起来更加简单。直接省去了listview中`convertView.setTag(holder)`和`convertView.getTag()`这些繁琐的步骤。
- 提供了一种插拔式的体验，高度的解耦，异常的灵活，针对一个Item的显示RecyclerView专门抽取出了相应的类，来控制Item的显示，使其的扩展性非常强。
- 设置布局管理器以控制Item的布局方式，横向、竖向以及瀑布流方式
- 可设置Item的间隔样式（可绘制），通过继承RecyclerView的ItemDecoration这个类，然后针对自己的业务需求去书写代码。
- 可以控制Item增删的动画，可以通过ItemAnimator类进行控制，当然针对增删的动画，RecyclerView有其自己默认的实现。

但是关于Item的点击和长按事件，需要用户自己去实现。

### 3. 缓存机制

#### RecyclerView缓存架构图

 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62aee1745ef7435bb93293bc49aafbcf~tplv-k3u1fbpfcp-zoom-1.image)

RecyclerView缓存是一个四级缓存的架构。当然，从RecyclerView的代码注释来看，官方认为只有三级缓存，即mCachedViews是一级缓存，mViewCacheExtension是二级缓存，mRecyclerPool是三级缓存。从开发者的角度来看，mAttachedScrap和mChangedScrap对开发者是不透明的，官方并未暴露出任何可以改变他们行为的方法。

mCacheViews可以通过如下方法，改变缓存的大小：

```java
public void setItemViewCacheSize(int size) {
    mRecycler.setViewCacheSize(size);
}
```

mViewCacheExtension则是一个抽象类，你完全可以自定义一个子类，修改获取缓存的策略。但是这个类只提供了获取缓存的接口，没有提供保存缓存的接口，对开发者要求甚高，而且使用RecyclerPool都能很好的实现一般的缓存需求。所以该接口，基本没用。

```java
public abstract static class ViewCacheExtension {
  public abstract View getViewForPositionAndType(Recycler recycler, int position,
          int type);
  }
}
```

RecyclerViewPool类提供了修改不同类型View的最大缓存数量，这对开发者很透明。

```java
public void setMaxRecycledViews(int viewType, int max) {
    ScrapData scrapData = getScrapDataForType(viewType);
    scrapData.mMaxScrap = max;
    final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
    while (scrapHeap.size() > max) {
        scrapHeap.remove(scrapHeap.size() - 1);
    }
}
```

#### RecyclerView#Recycler源码

RecyclerView的缓存功能是定义在`RecyclerView#Recycler`中的。

```java
 public final class Recycler {
        final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
        ArrayList<ViewHolder> mChangedScrap = null;

        final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();

        private final List<ViewHolder>
                mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap);

        private int mRequestedCacheMax = DEFAULT_CACHE_SIZE;
        int mViewCacheMax = DEFAULT_CACHE_SIZE;

        RecycledViewPool mRecyclerPool;

        private ViewCacheExtension mViewCacheExtension;

        static final int DEFAULT_CACHE_SIZE = 2;
} 
```

##### mAttachedScrap

mAttachedScrap的对应数据结构是ArrayList，在`LayoutManager#onLayoutChildren`方法中，对views进行布局时，会将RecyclerView上的Views全部暂存到该集合中，以备后续使用，该缓存中的ViewHolder的特性是，如果和RV上的position或者itemId匹配上了，那么认为是干净的ViewHolder，是可以直接拿出来使用的，无需调用onBindViewHolder方法。该ArrayList的大小是没有限制的，屏幕上有多少个View，就会创建多大的集合。触发该层级缓存的场景一般是调用notifyItemXXX方法。调用notifyDataSetChanged方法，只有当Adapter hasStableIds返回true，会触发该层级的缓存使用。

##### mChangedScrap

mChangedScrap和mAttachedScrap是同一级的缓存，他们是平等的。但是mChangedScrap的调用场景是`notifyItemChanged`和`notifyItemRangeChanged`，只有发生变化的ViewHolder才会放入到mChangedScrap中。mChangedScrap缓存中的ViewHolder是需要调用onBindViewHolder方法重新绑定数据的。

那么此时就有个问题了，为什么同一级别的缓存需要设计两个不同的缓存？区别就是，mChangedScrap中的ViewHolder在RV填充满的情况下，是不会强行填充到RV上的。那么有办法可以让发生改变的ViewHolder进入mAttachedScrap缓存吗？当然可以。调用`notifyItemChanged(int position, Object payload)`方法可以，实现局部刷新功能，payload不为空，那么发生改变的ViewHolder是会被分离到mAttachedScrap中的。

##### mUnmodifiableAttachedScrap

`mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap)`是对mAttachedScrap的封装，它将mAttachedScrap暴露给开发者调用，它的特性就是只可读不能写。

##### mCachedViews

mCachedViews对应的数据结构也是ArrayList但是该缓存对集合的大小是有限制的，默认是2。该缓存中ViewHolder的特性和mAttachedScrap中的特性是一样的，只要position或者itemId对应上了，那么它就是干净的，无需重新绑定数据。开发者可以调用`setItemViewCacheSize(size)`方法来改变缓存的大小。该层级缓存触发的一个常见的场景是滑动RV。当然`notifyXXX`也会触发该缓存。该缓存和mAttachedScrap一样特别高效。

##### mViewCacheExtension

mViewCacheExtension开发者自己实现的意义不大，基本上所有你想做的，都可以通过RecyclerViewPool来实现。

##### mRecyclerPool

mRecyclerPool缓存可以针对多ItemType，设置缓存大小。默认每个ItemType的缓存个数是5。而且该缓存可以给多个RecyclerView共享。




## 区别

#### 基础使用

ListView：继承重写BaseAdapter类；自定义ViewHolder与convertView的优化（判断是否为null）；

RecyclerView：继承重写RecyclerView.Adapter与RecyclerView.ViewHolder；设置LayoutManager，以及layout的布局效果

区别：

- ViewHolder的编写规范化，ListView是需要自己定义的，而RecyclerView是规范好的
- RecyclerView复用item全部搞定，不需要想ListView那样setTag()与getTag()
- RecyclerView多了一些LayoutManager工作，但实现了布局效果多样化

#### 布局效果

- ListView 的布局比较单一，只有一个纵向效果；

- RecyclerView 的布局效果丰富， 可以在LayoutMananger中设置：线性布局（纵向，横向），表格布局，瀑布流布局

- 在RecyclerView 中，如果存在的LayoutManager不能满足需求，可以在LayoutManager的API中自定义Layout，例如：scrollToPosition(), setOrientation(), getOrientation(), findViewByPosition()等等；

#### 空数据处理

- 在ListView中有个`setEmptyView()` 用来处理Adapter中数据为空的情况

- 但是在RecyclerView中没有这个API，所以在RecyclerView中需要进行一些数据判断来实现数据为空的情况；

#### HeaderView 与 FooterView

- 在ListView中可以通过`addHeaderView()` 与 `addFooterView()`来添加头部item与底部item，来当我们需要实现下拉刷新或者上拉加载的情况；而且这两个API不会影响Adapter的编写；

- 但是RecyclerView中并没有这两个API，所以当我们需要在RecyclerView添加头部item或者底部item的时候，我们可以在Adapter中自己编写，根据ViewHolder的Type与View来实现自己的Header，Footer与普通的item，但是这样就会影响到Adapter的数据，比如position，添加了Header与Footer后，实际的position将大于数据的position；

#### 局部刷新 

- 在ListView中通常刷新数据是用`notifyDataSetChanged()` ，但是这种刷新数据是全局刷新的（每个item的数据都会重新加载一遍），这样一来就会非常消耗资源
- RecyclerView中可以实现局部刷新，例如：`notifyItemChanged()`；
- 但是如果要在ListView实现局部刷新，依然是可以实现的，当一个item数据刷新时，我们可以在Adapter中，实现一个`onItemChanged()`方法，在方法里面获取到这个item的position（可以通过`getFirstVisiblePosition()`），然后调用`getView()`方法来刷新这个item的数据；

#### 动画效果

- 在RecyclerView中，已经封装好API来实现自己的动画效果；有许多动画API，例如：`notifyItemChanged()`, `notifyDataInserted()`, `notifyItemMoved()`等等；如果我们需要淑贤自己的动画效果，我们可以通过相应的接口实现自定义的动画效果（`RecyclerView.ItemAnimator`类），然后调用`RecyclerView.setItemAnimator()` (默认的有`SimpleItemAnimator`与`DefaultItemAnimator`）；
- 但是ListView并没有实现动画效果，但我们可以在Adapter自己实现item的动画效果；

#### Item点击事件

- 在ListView中有`onItemClickListener()`, `onItemLongClickListener()`, `onItemSelectedListener()`, 但是添加HeaderView与FooterView后就不一样了，因为HeaderView与FooterView都会算进position中，这时会发现position会出现变化，可能会抛出数组越界，为了解决这个问题，我们在`getItemId()`方法（在该方法中HeaderView与FooterView返回的值是-1）中通过返回id来标志对应的item，而不是通过position来标记；但是我们可以在Adapter中针对每个item写在`getView()`中会比较合适；

- 而在RecyclerView中，提供了唯一一个API：`addOnItemTouchListener()`，监听item的触摸事件；我们可以通过RecyclerView的`addOnItemTouchListener()`加上系统提供的`Gesture Detector`来实现像ListView那样监听某个item某个操作方法；

#### 嵌套滚动机制

- 在事件分发机制中，Touch事件在进行分发的时候，由父View向子View传递，一旦子View消费这个事件的话，那么接下来的事件分发的时候，父View将不接受，由子View进行处理；但是与Android的事件分发机制不同，嵌套滚动机制（Nested Scrolling）可以弥补这个不足，能让子View与父View同时处理这个Touch事件，主要实现在于`NestedScrollingChild`与`NestedScrollingParent`这两个接口；而在RecyclerView中，实现的是NestedScrollingChild，所以能实现嵌套滚动机制；

- ListView就没有实现嵌套滚动机制；

#### 缓存

- RecyclerView比ListView多两级缓存，支持多个ItemView缓存，支持开发者自定义缓存处理逻辑，支持所有RecyclerView共用同一个RecyclerViewPool(缓存池)。
- ListView和RecyclerView缓存机制基本一致，但缓存使用不同