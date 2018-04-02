# [学习笔记] Android群英传：ListView 使用技巧

##一.Listview常用优化技巧

###1.使用ViewHolder模式提高效率
ViewHolder模式利用了ListView的视图缓存机制，避免每次调用getView()的时候去通过findViewById()实例化控件，需要在自定义的Adapter里面定义一个内部类ViewHolder即可，代码如下：

```
	public final class ViewHolder{
        public ImageView img;
        public TextView tv;
    }
```

完整的Adapter：

```
/**
 * 自定义Adapter
 * Created by LGL on 2016/3/10.
 */
public class MyAdapter extends BaseAdapter {

    private List<String> mData;
    private LayoutInflater mInflater;

    //构造方法
    public MyAdapter(Context context, List<String> mData) {
        this.mData = mData;
        mInflater = LayoutInflater.from(context);
    }

    //返回长度
    @Override
    public int getCount() {
        return mData.size();
    }

    @Override
    public Object getItem(int position) {
        return mData.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder = null;
        //判断是否有缓存
        if (convertView == null) {
            viewHolder = new ViewHolder();
            //通过LayoutInflater去实例化布局
            convertView = mInflater.inflate(R.layout.item, null);
            viewHolder.img = (ImageView) convertView.findViewById(R.id.img);
            viewHolder.tv = (TextView) convertView.findViewById(R.id.tv);
            convertView.setTag(viewHolder);
        } else {
            //通过TAG找到缓存的布局
            viewHolder = (ViewHolder) convertView.getTag();
        }
        //设置布局中要显示的东西
        viewHolder.tv.setText(mData.get(position));
        return convertView;
    }

    public final class ViewHolder {
        public ImageView img;
        public TextView tv;
    }
}

```
使用adapter：

```

public class MainActivity extends AppCompatActivity {

    private ListView listview;
    private MyAdapter myAdapter;
    private List<String> list = new ArrayList<String>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        listview = (ListView) findViewById(R.id.listview);

        for (int i = 0; i < 20; i++) {
            list.add( i + "");
        }
        myAdapter = new MyAdapter(this, list);
        listview.setAdapter(myAdapter);
    }
}

```


###2.设置项目间分割栏、
>ListView在各个项目之中，可以通过分割线进行区分的，系统也提供了两个属性来设置item间的颜色和高度

```
android:dividerHeight="10dp"
android:divider="@android:color/holo_blue_bright"
```

去掉这个分割线的话：

```
 android:divider="@null"
```

###3.隐藏listview的滚动条

```
 android:scrollbars="none"
```

###4.取消ListView的item点击效果

```
 android:listSelector="@android:color/transparent"
```

>这里：可以直接填#00000000

###5.设置ListView需要显示在第几项

```
listview.setSelection(15);
```

类似于平滑的效果了：

```
listview.smoothScrollBy(1,15);
listview.smoothScrollByOffset(15);
listview.smoothScrollToPosition(15);
```


###6.动态修改ListView

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <Button
        android:id="@+id/add_data"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="添加数据" />
</LinearLayout>

```

点击事件：

```
 @Override
    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.add_data:
                list.add("我是新增加的数据");
                //通知刷新
                myAdapter.notifyDataSetChanged();
                break;
        }
    }
```

###7.遍历所有的item
ListView作为一个ViewGroup,他提供了很多操纵子View的方法，最常用的就是getChilaAt()来获取View了

```
for (int j = 0; j < listview.getChildCount();j++){
            View v = listview.getChildAt(12);
        }

```

###8.处理空ListView
```
 <TextView
        android:id="@+id/tv"
        android:text="没有数据"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
```

在代码中设置如果是空数据的话就加载指定控件：

```
 listview.setEmptyView(findViewById(R.id.tv));
```

###9.ListView滑动监听
很多应用场景需要重写ListView的其实都是因为滑动监听，通过判断滑动事件来处理不同的逻辑，开发者通常还需要GestureDetector手势识别，VelocityTracker滑动速度检测来辅助监听，这里介绍两种监听方法，一种是OnTouchListener,另一中是OnScrollListener

####1. OnTouchListener
OnTouchListener是View的监听事件，包括ACTION_DOWN,UP,MOVE等，通过坐标的改变老发生不同的逻辑

```
listview.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                switch (motionEvent.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        //触摸时操作
                        break;
                    case MotionEvent.ACTION_MOVE:
                        //移动时操作
                        break;
                    case MotionEvent.ACTION_UP:
                        //离开时操作
                        break;
                }
                return true;
            }
        });
```

####2. OnScrollListener
OnScrollListener是AbsListView的监听事件，他封装了很多ListView的相关信息

```
 listview.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView absListView, int scrollState) {
                               switch (scrollState) {
                    case AbsListView.OnScrollListener.SCROLL_STATE_IDLE:
                        //滚动停止
                        break;
                    case AbsListView.OnScrollListener.SCROLL_STATE_TOUCH_SCROLL:
                        //正在滚动
                        break;
                    case AbsListView.OnScrollListener.SCROLL_STATE_FLING:
                        //手指抛动时，即手指用力滑动的时候
                        break;
                }
            }

            @Override
            public void onScroll(AbsListView absListView, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                    //滚动的时候一直在调用
            }
        });
```
 OnScrollListener中有两个回调方法onScrollStateChanged()和onScroll()
onScrollStateChanged()，有三种模式:

- OnScrollListener.SCROLL_STATE_IDLE://滚动停止
- OnScrollListener.SCROLL_STATE_TOUCH_SCROLL://正在滚动
- OnScrollListener.SCROLL_STATE_FLING://手指抛动时，即手指用力滑动的时候

>当用户没有做手指抛动的动作时，这个方法只会调用2次，否则就调用三次，差别就在于手指抛动的这个状态

onScroll():

 **firstVisibleItem:**当前能看到的第一个item(包括部分显示的ListItem)的id(下标从0开始)
  **visibleItemCount:**可以看到的item(包括部分显示的ListItem)总数
 **totalItemCount:**表示ListView的ListItem总数
 
```
 if(firstVisibleItem+visibleItemCount == totalItemCount &&totalItemCount>0){
     //滚动到最后一行
     }
```

```

 if(firstVisibleItem>lastVisiblePosition){
     //上滑
  }else if(firstVisibleItem<lastVisiblePosition){
     //下滑
  }
  
  
```
>  lastVisiblePosition 为上次 firstVisibleItem;


ListView提供了获取位置信息的Api
```
 //获取可视区域内最后一个item的id
listview.getLastVisiblePosition();
 //获取可视区域内第一个item的id
 listview.getFirstVisiblePosition();
```

##二.ListView常用扩展
>虽然ListView应用很广泛，但是毕竟是一个显示的东西，扩展性肯定要的，我们接下来说几种常见的扩展

###1.具有弹性的ListView
>Android默认滑动到顶部或者底部只会有一个阴影，而在5.X之后改变成了半圆的阴影

![这里写图片描述](http://img.blog.csdn.net/20160320183200576)

>而在IOS上，列表是具有弹性的，即滚动到顶部或者底部，会再滚动一段距离，这样的设计感觉还是挺友好的，我们也来模仿一下
>
>我们在查看ListView的源码的时候会发现一个控制滑动到边缘的处理方法

```
@Override
    protected boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY, int scrollRangeX, int scrollRangeY, int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent) {
        return super.overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, maxOverScrollX, maxOverScrollY, isTouchEvent);
    }
```

>我們可以看到这样一个参数maxOverScrollY，就是他负责控制滑动的个数的，默认是0，我们重写ListView

```
package com.lgl.listviewdemo;

import android.content.Context;
import android.util.AttributeSet;
import android.util.DisplayMetrics;
import android.widget.ListView;

/**
 * 弹性ListView
 * Created by lgl on 16/3/20.
 */
public class ListViewScroll extends ListView {

    private int mMaxOverdistance ;

    public ListViewScroll(Context context, AttributeSet attrs) {
        super(context, attrs);

        //通过分辨率来调节滑动尺度
        DisplayMetrics metrics = context.getResources().getDisplayMetrics();
        float density = metrics.density;
        mMaxOverdistance = (int)(density*mMaxOverdistance);

    }


    @Override
    protected boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY, int scrollRangeX, int scrollRangeY, int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent) {
        return super.overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, mMaxOverdistance, maxOverScrollY, isTouchEvent);
    }
}

```

###2.自动显示，隐藏布局的ListView
>相信看过Google最新的应用，或者使用了MD风格的应用都知道，列表滑动可以躺actionbar显示或者隐藏，

**这一段作者写的很乱**

>这要是讲一下大概，需要使用ToolsBar,然后一个listview

```
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary" />
```

>不详细记了，在我讲MD风格的时候会详细介绍的

###3.聊天ListView
>相信大家都看过聊天的ListView吧，这个主要是adapter做手脚，其实设计者早就想到了这种方法，所以在继承BaseAdapter的时候还需要重写两个方法

```
 @Override
    public int getItemViewType(int position) {
        return super.getItemViewType(position);
    }

    @Override
    public int getViewTypeCount() {
        return super.getViewTypeCount();
    }
```

>我们这里大致的了解一个思路，我们可以设置一个tag来识别不同的方向，先写两个item,分别是左边的和右边的布局，然后，我们再来写个实体类Bean

####Bean
```
package com.lgl.listviewdemo;

import android.graphics.Bitmap;

/**
 * 实体类
 * Created by lgl on 16/3/20.
 */
public class Bean {

    private int type;
    private String text;
    private Bitmap icon;

    public Bean() {
    }

    public int getType() {
        return type;
    }

    public void setType(int type) {
        this.type = type;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public Bitmap getIcon() {
        return icon;
    }

    public void setIcon(Bitmap icon) {
        this.icon = icon;
    }
}

```

>现在就可以编写我们的Adapter了

####SpeakAdapter

```
package com.lgl.listviewdemo;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.List;

/**
 * Created by lgl on 16/3/20.
 */
public class SpeakAdapter extends BaseAdapter{

    private List<Bean>mData;
    private LayoutInflater mInflater;

    //构造方法
    public SpeakAdapter(Context context,List<Bean>mData){
        this.mData = mData;
        mInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return mData.size();
    }

    @Override
    public Object getItem(int position) {
        return mData.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if(convertView == null){
            //区分左右
            if(getItemViewType(position) == 0){
                viewHolder = new ViewHolder();
                convertView = mInflater.inflate(R.layout.left_item,null);
                viewHolder.icon = (ImageView) convertView.findViewById(R.id.left_icon);
                viewHolder.text = (TextView) convertView.findViewById(R.id.tv_left);
            }else{
                viewHolder = new ViewHolder();
                convertView = mInflater.inflate(R.layout.right_item,null);
                viewHolder.icon = (ImageView) convertView.findViewById(R.id.right_icon);
                viewHolder.text = (TextView) convertView.findViewById(R.id.tv_right);
            }
            convertView.setTag(viewHolder);
        }else{
            viewHolder = (ViewHolder) convertView.getTag();
        }

        viewHolder.icon.setImageBitmap(mData.get(position).getIcon());
        viewHolder.text.setText(mData.get(position).getText());

        return convertView;
    }

    @Override
    public int getItemViewType(int position) {
        Bean bean = mData.get(position);
        return bean.getType();
    }

    @Override
    public int getViewTypeCount() {
        return 2;
    }

    public final class  ViewHolder{
        public ImageView icon;
        public TextView text;
    }
}

```

>这里大致的思路也就是在getview中区分，现在我们的SpeakListViewActivity中就可以这样写

```
package com.lgl.listviewdemo;

import android.graphics.BitmapFactory;
import android.os.Bundle;
import android.os.PersistableBundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.ListView;

import java.util.ArrayList;
import java.util.List;

/**
 * 聊天ListView
 * Created by lgl on 16/3/20.
 */
public class SpeakListViewActivity extends AppCompatActivity {

    private ListView mListview;

    @Override
    public void onCreate(Bundle savedInstanceState, PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        setContentView(R.layout.activity_speak);

        mListview = (ListView) findViewById(R.id.listview);
        Bean bean1 = new Bean();
        bean1.setType(0);
        bean1.setIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
        bean1.setText("我是左边");

        Bean bean2 = new Bean();
        bean2.setType(1);
        bean2.setIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
        bean2.setText("我是右边");

        List<Bean>data = new ArrayList<Bean>();
        data.add(bean1);
        data.add(bean2);

        SpeakAdapter adapter = new SpeakAdapter(this,data);
        mListview.setAdapter(adapter);
    }
}

```

>把数据装载好了就行，这里只是演示


###4.动态改变ListView的布局
**都是什么鬼，写的不是很详细呀**
>通常情况下，如果要动态的改变点击item的布局来达到Focus的效果，一般有两种方法，一是将两个布局写在一起，通过布局的显示隐藏来达到切换布局的效果，另外一种则是在getView的时候，通过判断来选择不同的加载不同的布局，两种方法都有利弊，关键还是要看看应用场景，所以我们还是得在Adapter下手脚

```
 private View addFocusView(int i){
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.mipmap.ic_launcher);
        return iv;
    }

    private View addNormalView(int i){
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.HORIZONTAL);
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.mipmap.ic_launcher);
        layout.addView(iv,new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT,LinearLayout.LayoutParams.WRAP_CONTENT));
        TextView tv = new TextView(mContext);
        tv.setText(list.get(i));
        layout.addView(tv, new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT));
        layout.setGravity(Gravity.CENTER);
        return layout;
    }
```

>在这两个方法中就可以根据item的不同位置显示不同的信息了，下面我们回到adapter中

```
@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.VERTICAL);
        if(mCurrentItem == position){
            layout.addView(addFocusView(position));
        }else {
            layout.addView(addNormalView(position));
        }

        return convertView;
    } 
```

>这样就可以自由选择了
>那我们现在就来监听他的点击逻辑吧

```
 listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                myAdapter.setCurrentItem(position);
                myAdapter.notifyDataSetChanged();
            }
        });
```




