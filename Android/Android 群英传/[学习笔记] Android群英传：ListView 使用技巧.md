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

###1.具有弹性的ListView
Android默认滑动到顶部或者底部只会有一个阴影，而在5.X之后改变成了半圆的阴影

![](http://img.blog.csdn.net/20160320183200576)

在IOS上，列表是具有弹性的，即滚动到顶部或者底部，会再滚动一段距离。

在查看ListView的源码的时候会发现一个控制滑动到边缘的处理方法

```
@Override
    protected boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY, int scrollRangeX, int scrollRangeY, int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent) {
        return super.overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, maxOverScrollX, maxOverScrollY, isTouchEvent);
    }
```

>我們可以看到这样一个参数maxOverScrollY，就是他负责控制滑动的个数的，默认是0，我们重写ListView

```
/**
 * 弹性ListView
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
列表滑动actionbar显示或者隐藏
需要使用ToolsBar,然后一个listview

```
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary" />
```
```
public class ScrollHideListView extends Activity {

    private Toolbar mToolbar;
    private ListView mListView;
    private String[] mStr = new String[20];
    private int mTouchSlop;
    private float mFirstY;
    private float mCurrentY;
    private int direction;
    private ObjectAnimator mAnimator;
    private boolean mShow = true;

    View.OnTouchListener myTouchListener = new View.OnTouchListener() {
        @Override
        public boolean onTouch(View v, MotionEvent event) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    mFirstY = event.getY();
                    break;
                case MotionEvent.ACTION_MOVE:
                    mCurrentY = event.getY();
                    if (mCurrentY - mFirstY > mTouchSlop) {
                        direction = 0;// down
                    } else if (mFirstY - mCurrentY > mTouchSlop) {
                        direction = 1;// up
                    }
                    if (direction == 1) {
                        if (mShow) {
                            toolbarAnim(1);//hide
                            mShow = !mShow;
                        }
                    } else if (direction == 0) {
                        if (!mShow) {
                            toolbarAnim(0);//show
                            mShow = !mShow;
                        }
                    }
                    break;
                case MotionEvent.ACTION_UP:
                    break;
            }
            return false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.scroll_hide);
        mTouchSlop = ViewConfiguration.get(this).getScaledTouchSlop();
        mToolbar = (Toolbar) findViewById(R.id.toolbar);
        mListView = (ListView) findViewById(R.id.listview);
        for (int i = 0; i < mStr.length; i++) {
            mStr[i] = "Item " + i;
        }
        View header = new View(this);
        header.setLayoutParams(new AbsListView.LayoutParams(
                AbsListView.LayoutParams.MATCH_PARENT,
                (int) getResources().getDimension(
                        R.dimen.abc_action_bar_default_height_material)));
        mListView.addHeaderView(header);
        mListView.setAdapter(new ArrayAdapter<String>(
                ScrollHideListView.this,
                android.R.layout.simple_expandable_list_item_1,
                mStr));
        mListView.setOnTouchListener(myTouchListener);
    }

    private void toolbarAnim(int flag) {
        if (mAnimator != null && mAnimator.isRunning()) {
            mAnimator.cancel();
        }
        if (flag == 0) {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(), 0);
        } else {
            mAnimator = ObjectAnimator.ofFloat(mToolbar,
                    "translationY", mToolbar.getTranslationY(),
                    -mToolbar.getHeight());
        }
        mAnimator.start();
    }
}

```
###3.聊天ListView
继承BaseAdapter的时候需要重写两个方法
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

实体类Bean

####Bean
```
/**
 * 实体类
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

public class ChatAdapter extends BaseAdapter{

    private List<Bean>mData;
    private LayoutInflater mInflater;

    //构造方法
    public ChatAdapter(Context context,List<Bean>mData){
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

ChatListViewActivity：

```
/**
 * 聊天ListView
 */
public class ChatListViewActivity extends AppCompatActivity {

    private ListView mListview;

    @Override
    public void onCreate(Bundle savedInstanceState, PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
        setContentView(R.layout.activity_chat);

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

        ChatAdapter adapter = new ChatAdapter(this,data);
        mListview.setAdapter(adapter);
    }
}

```

###4.动态改变ListView的布局

```
public class FocusListViewAdapter extends BaseAdapter {

    private List<String> mData;
    private Context mContext;
    private int mCurrentItem = 0;

    public FocusListViewAdapter(Context context, List<String> data) {
        this.mContext = context;
        this.mData = data;
    }

    public int getCount() {
        return mData.size();
    }

    public Object getItem(int position) {
        return mData.get(position);
    }

    public long getItemId(int position) {
        return position;
    }

    public View getView(int position, View convertView, ViewGroup parent) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.VERTICAL);
        if (mCurrentItem == position) {
            layout.addView(addFocusView(position));
        } else {
            layout.addView(addNormalView(position));
        }
        return layout;
    }

    public void setCurrentItem(int currentItem) {
        this.mCurrentItem = currentItem;
    }

    private View addFocusView(int i) {
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.drawable.ic_launcher);
        return iv;
    }

    private View addNormalView(int i) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.HORIZONTAL);
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.drawable.in_icon);
        layout.addView(iv, new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT));
        TextView tv = new TextView(mContext);
        tv.setText(mData.get(i));
        layout.addView(tv, new LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT));
        layout.setGravity(Gravity.CENTER);
        return layout;
    }
```

```
 listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                myAdapter.setCurrentItem(position);
                myAdapter.notifyDataSetChanged();
            }
        });
```




