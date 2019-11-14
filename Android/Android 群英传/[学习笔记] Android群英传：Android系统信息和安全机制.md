#[学习笔记] Android群英传：Android系统信息和安全机制

---
主要内容：

- Android系统信息的获取
- PackageManager的使用
- ActivityManager的使用
- Android安全机制

##一. Android系统信息的获取
系统的配置信息，通常可以从以下两个方面获取

- android.os.Build
- SystemProperty

###1.android.os.Build
android.os.Build类里面的信息非常的丰富，它包含了系统编译时的大量设备、配置信息：

- Build.BOARD——主板
- Build.BRAND——Android系统定制商
- Build.SUPPORTED_ABIS——CPU指令集合
- Build.DEVICE——设备参数
- Build.DISPLAY——显示屏参数
- Build.FINGERPRINT——唯一编号
- Build.SERIAL——硬件序列号
- Build.ID——修订版本列表
- Build.MANUFACTURER——硬件制造商
- Build.MODEL——版本
- Build.HARDWARE——硬件名
- Build.PRODUCT——手机产品名
- Build.TAGS——描述Build的标签
- Build.TYPE——Builder类型
- Build.VERSION.CODENAME——当前开发代号
- Build.VERSION.INCREMENTAL——源码控制版本号
- Build.VERSION.RELEASE——版本字符串
- Build.VERSION.SDK_INT——版本号
- Build.HOST——host值
- Build.USER——User名
- Build.TIME——编译时间

###2.SystemProperty
SystemProperty类包含了许多系统配置属性值和参数，很多信息和上面通过android.os.Build获取的值是相同的

- os.version——OS版本
- os.name——OS名称
- os.arch——OS架构
- user.home——home属性
- user.name——name属性
- user.dir——Dir属性
- user.timezone——时区
- path.separator——路径分隔符
- line.separator——行分隔符
- file.separator——文件分隔符
- java.vendor.url——Java vender Url属性
- java.class.path——Java Class属性
- java.class.version——Java Class版本
- java.vendor——Java Vender属性
- java.version——Java版本
- java.home——Java Home属性

###3.Android系统信息实例

通过android.os.Build这个类，我们可以直接获取到Build提供的系统信息，而通过System.getProperty（“xxx”），我们可以访问到系统的属性


![这里写图片描述](http://img.blog.csdn.net/20160427215950431)

```
		tv_message.append("主板:"+ Build.BOARD+"\n");
        tv_message.append("Android系统定制商:"+ Build.BRAND+"\n");
        tv_message.append("CPU指令集:"+ Build.SUPPORTED_ABIS+"\n");
        tv_message.append("设置参数:"+ Build.DEVICE+"\n");
        tv_message.append("显示屏参数:"+ Build.DISPLAY+"\n");
        tv_message.append("唯一编号:"+ Build.SERIAL+"\n");
```

System.getProperty:

![这里写图片描述](http://img.blog.csdn.net/20160427220057916)

```
		tv.append("OS版本："+System.getProperty("os.version")+"\n");
        tv.append("OS名称："+System.getProperty("os.name")+"\n");
        tv.append("OS架构："+System.getProperty("os.arch")+"\n");
        tv.append("Home属性："+System.getProperty("user.home")+"\n");
        tv.append("Name属性："+System.getProperty("user.name")+"\n");
```


在system/build.prop中，包含了很多的RO值，打开命令窗，通过cat build.prop可以看到。这里我们可以看到很多前面通过android.os.Build所获取到的系统信息，同时，在adb shell中，还可以通过getprop来获取对应的值

android系统还在另外一个非常重要的目录来存储系统信息——/proc目录，在adb shell中进入/proc目录，通过ll命令查看文件信息

##二.Android APK应用程序信息获取值PackageManager

PackageManager

![这里写图片描述](http://img.blog.csdn.net/20160428214218256)

最里面的框就代表整个Activity的信息，系统提供了ActivityInfo类来进行封装

Android提供了PackageManager来负责管理所有已安装的App，PackageManager可以获得AndroidManifest中不同节点的封装信息，下面是一些常用的封装信息：

- ActivityInfo
>ActivityInfo封装在了Mainfest文件中的< activity ></ activity>和< eceiver></ receiver>之间的所有信息，包括name、icon、label、launchMode等。

- ServiceInfo
>ServiceInfo与ActivityInfo类似，封装了< service></ service>之间的所有信息。

- ApplicationInfo
>它封装了< application></ application>之间的信息，特别的是，ApplicationInfo包含了很多Flag，FLAG_SYSTEM表示为系统应用，FLAG_EXTERNAL_STORAGE表示为安装在SDcard上的应用，通过这些flag可以很方便的判断应用的类型。

- PackageInfo
>PackageInfo包含了所有的Activity和Service信息。

- ResolveInfo
>ResolveInfo包含了< intent>信息的上级信息，所以它可以返回ActivityInfo、ServiceInfo等包含了< intent>的信息，经常用来帮助找到那些包含特定intent条件的信息，如带分享功能、播放功能的应用。

下面就是PackageManager中封装的用来获取这些信息的方法：

- getPackageManager()——通过这个方法可以返回一个PackageManager对象。
- getApplicationInfo()——以ApplicationInfo的形式返回指定包名的ApplicationInfo。
- getApplicationIcon()——返回指定包名的Icon。
- getInstalledApplications()——以ApplicationInfo的形式返回安装的应用。
- getInstalledPackages()——以PackageInfo的形式返回安装的应用。
- queryIntentActivities()——返回指定Intent的ResolveInfo对象、Activity集合。
- queryIntentServices()——返回指定Intent的ResolveInfo对象、Service集合。
- resolveActivity()——返回指定Intent的Activity。
- resolveService()——返回指定Intent的Service。

根据ApplicationInfo的flag来判断App的类型：

- 如果当前应用的flags & ApplicationInfo.FLAG_SYSTEM != 0则为系统应用

- 如果flags & ApplicationInfo.FLAG_SYSTEM <= 0 则为第三方应用

- 特殊的当系统应用升级后也会成为第三方应用，此时 flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_APP != 0;

- 如果flags & ApplicationInfo.FLAG_EXTERNAL_STORAGE != 0 则为安装在SDCard上的应用。

通过一个实例来分析，先封装一个Bean来保存我们需要的字段

```java
/**
 * Bean
 */
public class PMAPPInfo {

    //应用名
    private String appLabel;
    //图标
    private Drawable appIcon;
    //包名
    private String pkgName;
    
    //构造方法
    public PMAPPInfo(){
        
    }

    public String getAppLabel() {
        return appLabel;
    }

    public void setAppLabel(String appLabel) {
        this.appLabel = appLabel;
    }

    public Drawable getAppIcon() {
        return appIcon;
    }

    public void setAppIcon(Drawable appIcon) {
        this.appIcon = appIcon;
    }

    public String getPkgName() {
        return pkgName;
    }

    public void setPkgName(String pkgName) {
        this.pkgName = pkgName;
    }
}

```
通过上面的方法判断各种类型的应用，主布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">


    <Button
        android:id="@+id/btn_all"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="All APP"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/btn_other"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Other App"
        android:textAllCaps="false" />

    <Button
        android:id="@+id/btn_system"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="System App"
        android:textAllCaps="false" />

    <ListView
        android:id="@+id/mListView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"></ListView>

</LinearLayout>
```

有一个listview，所以我们需要一个adapter和一个item

```java

/**
 * 数据源
 */
public class PkgAdapter extends BaseAdapter {
    private Context mContext;
    private List<PMAPPInfo> mList;

    public PkgAdapter(Context context) {
        mContext = context;
        mList = new ArrayList<>();
    }

    public void addAll(List<PMAPPInfo> list) {
        mList.clear();
        mList.addAll(list);
        notifyDataSetChanged();
    }

    @Override
    public int getCount() {
        return mList.size();
    }

    @Override
    public PMAPPInfo getItem(int position) {
        return mList.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        PMAPPInfo item = getItem(position);
        if (convertView == null) {
            holder = new ViewHolder();
            convertView = LayoutInflater.from(mContext).inflate(R.layout.list_item, parent, false);
            holder.mIcon = (ImageView) convertView.findViewById(R.id.iv_icon);
            holder.mLabel = (TextView) convertView.findViewById(R.id.tv_label);
            holder.mPkgName = (TextView) convertView.findViewById(R.id.tv_pkg_name);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        holder.mIcon.setImageDrawable(item.getAppIcon());
        holder.mLabel.setText(item.getAppLabel());
        holder.mPkgName.setText(item.getPkgName());
        return convertView;
    }

    static class ViewHolder {
        ImageView mIcon;
        TextView mLabel;
        TextView mPkgName;
    }
}

```

item

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/empty"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_vertical"
    android:orientation="horizontal">

    <ImageView
        android:id="@+id/iv_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_marginLeft="15dp"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tv_label"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <TextView
            android:id="@+id/tv_pkg_name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

    </LinearLayout>

</LinearLayout>
```

主程序的逻辑就是

```java


/**
 * 软件列表
 */
public class APP extends AppCompatActivity implements View.OnClickListener{

    private PackageManager mPackageManager;

    private ListView mListView;
    private PkgAdapter mAdapter;
    List<PMAPPInfo> result;

    protected void onCreate( Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_app);

        mPackageManager = getPackageManager();
        mListView = (ListView) findViewById(R.id.mListView);
        mAdapter = new PkgAdapter(this);
        mListView.setEmptyView(findViewById(R.id.empty));
        mListView.setAdapter(mAdapter);
        findViewById(R.id.btn_all).setOnClickListener(this);
        findViewById(R.id.btn_other).setOnClickListener(this);
        findViewById(R.id.btn_system).setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_all:
                result = getAppInfo(R.id.btn_all);
                break;
            case R.id.btn_other:
                result = getAppInfo(R.id.btn_other);
                break;
            case R.id.btn_system:
                result = getAppInfo(R.id.btn_system);
                break;
            default:
                result = new ArrayList<>();
        }
        mAdapter.addAll(result);
    }

    private List<PMAPPInfo> getAppInfo(int flag) {
        List<ApplicationInfo> appInfos = mPackageManager.getInstalledApplications(
                PackageManager.GET_UNINSTALLED_PACKAGES);
        List<PMAPPInfo> list = new ArrayList<>();
        //根据不同的flag来切换显示不同的App类型
        switch (flag) {
            case R.id.btn_all:
                list.clear();
                for (ApplicationInfo appInfo : appInfos) {
                    list.add(makeAppInfo(appInfo));
                }

                break;
            case R.id.btn_other:
                list.clear();
                for (ApplicationInfo appInfo : appInfos) {
                    if ((appInfo.flags & ApplicationInfo.FLAG_SYSTEM) <= 0) {
                        list.add(makeAppInfo(appInfo));
                    } else if ((appInfo.flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_APP) != 0){
                        list.add(makeAppInfo(appInfo));
                    }
                }
                break;
            case R.id.btn_system:
                list.clear();
                for (ApplicationInfo appInfo : appInfos) {
                    if ((appInfo.flags & ApplicationInfo.FLAG_SYSTEM) != 0) {
                        list.add(makeAppInfo(appInfo));
                    }
                }
                break;
        }
        return list;
    }

    private PMAPPInfo makeAppInfo(ApplicationInfo appInfo) {
        PMAPPInfo info = new PMAPPInfo();
        info.setAppIcon(appInfo.loadIcon(mPackageManager));
        info.setAppLabel(appInfo.loadLabel(mPackageManager).toString());
        info.setPkgName(appInfo.packageName);
        return info;
    }
}

```

##三.Android APK应用程序信息获取值ActivityManager
packagemanager获取的是所有的应用包名，ActivityManager获取运行的应用程序信息

ActivityManager也封装的部分Bean对象：

- ActivityManager.MemoryInfo——MemoryInfo有几个非常重要的字段：availMem（系统可用内存），totalMem（总内存），threshold（低内存的阈值，即区分是否低内存的临界值），lowMemory（是否处于低内存）。

- Debug.MemoryInfo——这个MemoryInfo用于统计进程下的内存信息。
- RunningAppProcessInfo——运行进程的信息，存储的字段有：processName（进程名），pid（进程pid），uid（进程uid），pkgList（该进程下的所有包）

- RunningServiceInfo——运行的服务信息，在它里面同样包含了一些服务进程信息，同时还有一些其他信息。activeSince（第一次被激活的时间、方式），foreground（服务是否在后台执行）。



```java

public class AMProcessInfo {
    
    private String pid;
    private String uid;
    private String memorySize;
    private String processName;
    
    public  AMProcessInfo(){
        
    }

    public String getPid() {
        return pid;
    }

    public void setPid(String pid) {
        this.pid = pid;
    }

    public String getUid() {
        return uid;
    }

    public void setUid(String uid) {
        this.uid = uid;
    }

    public String getProcessName() {
        return processName;
    }

    public void setProcessName(String processName) {
        this.processName = processName;
    }

    public String getMemorySize() {
        return memorySize;
    }

    public void setMemorySize(String memorySize) {
        this.memorySize = memorySize;
    }
}

```

>然后我们用一个方法就可以获取到了

```java
     /**
     * 正在运行
     * @return
     */
    private List<ActivityManager.RunningAppProcessInfo> getRunningProcossInfo(){
        mAMProcessInfo = new ArrayList<>();
        List<ActivityManager.RunningAppProcessInfo>appRunningList = activityManager.getRunningAppProcesses();
        
        for (int i = 0;i<appRunningList.size();i++){
            ActivityManager.RunningAppProcessInfo info = appRunningList.get(i);
            int pid = info.pid;
            int uid = info.uid;
            String procossName = info.processName;
            int[]memoryPid = new int[]{pid};
            Debug.MemoryInfo[] memoryInfos = activityManager.getProcessMemoryInfo(memoryPid);
            
            int memorySize = memoryInfos[0].getTotalPss();
            
            AMProcessInfo processInfo = new AMProcessInfo();
            processInfo.setPid(""+pid);
            processInfo.setUid(""+uid);
            processInfo.setMemorySize(""+memorySize);
            processInfo.setProcessName(procossName);
            appRunningList.add(processInfo);
        }
        return  appRunningList;
    }
```


##四.解析Packages.xml获取系统信息

熟悉Android开机启动流程的朋友大概知道，在系统初始化到时候，packagemanager的底层实现类packagemanagerService会去扫描系统的一些特定目录，并且解析其中的Apk文件，同时Android把他获取到的应用信息保存到xml中，做成一个应用的花名册，就是data/system/apckages.xml，我们用adb pull命令把他导出来，里面的信息也太多了，但是我们只要知道结果根节点就可以了

- < permissions>标签
>permissions标签定义了现在系统所有的权限，也分两类，系统定义的和app定义的

- < package>标签
>package代表的是一个apk的属性
 - name:APK的包名
 - cadePath:APK安装路径，主要在system/app和data/app两种，前者放系统级别的，后者放系统安装的
 - userid:用户ID
 - version:版本

- < perms>标签
对应apk的清单文件，记录apk的权限信息

##五.Android安全机制

###1.Android安全机制介绍
Android开发者在Android系统中简历了五道防线来保护Android的安全

#### 1.第一道防线
代码安全机制——代码混淆proguard
由于java语言的特殊性，即使是编译成apk的应用程序也存在反编译的风险，而proguard则是在代码从上对app的第一道程序，他混淆关键代码，替换命名，让破坏者阅读难，同样也可以压缩代码，优化编译后的字节

####2.第二道防线
应用接入权限控制——清单文件权限声明，权限检查机制
任何app在使用Android受限资源的时候，都需要显示向系统生命权限，只有当一个应用app具有相应的权限，才能申请相应的资源，通过权限机制的检查并且使用并且使用系统的Binder对象完成对系统服务的调用

不足，如以下几项:
- 被授予的权限无法停止
- 在应用声明app使用权限的时候，用户无法针对部分权限进行限制
- 权限的判断机制与用户的安全理念相关

>Android系统通常按照以下顺序来检查操作者的权限.
>首先,判断permission名称.如果为空则直接返回PERMISSION_DENIED
>其次。判断Uid,如果为0则为root权限，不做权限控制，如果为systyemsystem service的uid则为系统服务.不做权限控制:如果Uid与参数中的请求uid不同则返回PERMISSION_DENIED
>最后,通过调用packagemanageservice.checkUidPermission()方法来判断该uid是否具有相应的权限，该方法会去xml的权限列表和系统级的权限进行查找
通过上面的步骤Android就确定了使用者是否具有某项使用权限

####3. 第三道防线
应用签名机制一数字证书。
Android中所有的app都会有个数字证书,这就是app的签名.数字证书用于保护app的作者和其app的信任关系，只有拥有相同数字签名的app，才会在升级时被认为是同一app，而且Android系统不会安装没有签名的App

####4. 第四道防线
Linux内核层安全机制一一Uid 访问权限控制。
Animid本质是基于Linux内核开发的，所以Android同样继承了Linux的安全特性，比如Linux文件系统的权限控制是由user,group,other与读，写，执行的不同组合来实现的，同样,Android也实现了这套机制”通常情况下.只有system，root用户才有权限访问到系统文件，而一般用户无法访问。

####5. 第五道防线
Android虚拟机沙箱机制——沙箱隔流
Android的App运行在虚拟机中，因此才有沙箱机制，可以让应用之间相互隔离，通常情况下.不同的应用之间不能互相访问.每个App都单独的运行在虚似机中，与其他应用完全隔离.在实现安全机制的基础上，也让应用之间能够互不影响,即时一个应用崩溃，也不会导致其他应用异常

###2.Android系统安全隐患

####-1.代码漏洞
这个问题存在世界上所有的程序中，没有谁敢保证自己的程序没有bug，有漏洞，如果遇到这种问题，大家只能尽快的升级版本，更新补丁，才能杜绝利用漏洞的攻击装，比如Android的LaunchAnyWhere,FakeId，这些都是bug，就是在编写的时候产生的漏洞，只有期待官方的更新了

####2. root风险
Root权限是指Android的系统管理员权限,类似于windows系统中的Administrator。具有Root权限的用户可以访问和修改手机中几乎所有的文件

####3. 安全机制不健全
由于Android的权限管理机制并不完美,所以很多手机开发商，通常会在RoM中增加自己的一套权限管理工具来帮助用户控制手机中应用的权限

####4.用户安全意识

用户对于安全隐患的察觉里也是保护手机安全的一个重要因素。用户可以通过在正规的应用市场下载和安装应用时通过列出来的应用权限申请信息来大致判断一个应用的安全性，用户在安装不明来源的应用时，如果一个娱乐类型的app生命权限的时候不仅要联系人又要短信权限，这个时候就需要警惕了，用户也可以在市场上下载一些安全类App，如LEB安全大师，360安全等, 虽然这些软件会加重系统负担，但是为了安全也是值得的

####5. Android开发原则与安全

Android与ioS系统一个非常显著的区别就是_一个是开放系统，一个是封闭系统，开放自然有开放的好处,技术进步快，产品丰富，封闭也有封闭的好处，安全性高，可控性高，Google本着开源的精神开放了Android的源代码，但随之而来的各种安全问题也让Android倍受诟病,过度的开放与可定制化,不仅造成了Android的碎片化严重, 同时也给很多不法应用以可乘之机，但可喜的是,随着Android的发展日益壮大，Google也在着手处理开发与安全的问题,相信在不久的将来,这一矛盾会越来越小

###3.Android Apk反编译

Android的应用程序APk文件,说到底也是一个压缩文件，那么可以通过解压缩，获得里面的文件内容。让我们先来找一个apk，然后使用解压缩工具,最后就会得到一些文件。

![这里写图片描述](http://img.blog.csdn.net/20160428233512594)

首先三个重量级的工具，

![这里写图片描述](http://img.blog.csdn.net/20160428234452804)

>apktools:反编译
>下载地址：http://ibotpeaches.github.io/Apktool/install/
>dex2jar 这个工具用于将dex文件转换成jar文件 
下载地址：http://sourceforge.net/projects/dex2jar/files/
jd-gui 这个工具用于将jar文件转换成java代码 
下载地址：http://jd.benow.ca/

这三 个工具分别负责反编译不同的部分

####1.apktool
首先我们来反编译apk的xml文件，使用的是apktool，我们进入所在的目录执行反编译命令

![这里写图片描述](http://img.blog.csdn.net/20160428234701633)

格式非常的简单，参数d是指decode，并写入要反编译的目录，执行后，就会生成一个对应apk名字的文件夹，这个时候你进去看xml的代码就不会有错误了

![这里写图片描述](http://img.blog.csdn.net/20160428234822274)

这个工具可以方便我们汉化们重新打包的命令是b，选择文件夹即可

####2.Dex2jar,jd-gui

现在需要这两个工具了，我们回到apk的文件夹，里面有一个非常重要的文件classes.dex,这个就是源代码了，我们把它复制到Dex2jar的目录下

![这里写图片描述](http://img.blog.csdn.net/20160428235209681)

>并且执行如下命令

![这里写图片描述](http://img.blog.csdn.net/20160428235342229)

>这里就生成了一个jar文件，然后我们就可以用最后的jd-gui去查看了

![这里写图片描述](http://img.blog.csdn.net/20160428235519479)


>通过以上的方式，我们就可以完美的编译一个应用程序了，

###3.Android APK 加密
>由于java字节的特殊性，他很容易反编译，为了能够保护好代码，我们通常会使用一些措施，比如说混淆，而在Android studio中，可以很方便的使用ProGuard，在Gradle Scripts目录下

```java
 buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```

这里的minifyEnabled属性就是控制是否启动ProGuard，这个属性以前叫做runProgyard,在Android studio1.1的时候改成minifyEnabled，设置成true，就可以混淆了，位于SDK目录下的tools/proguard/proguard-android.txt目录下，大部分的情况下使用使用这个默认的混淆就好了，后面亦不过分是项目中自定义的混淆，可以在项目的app文件夹下找到这个文件，在这根文件里可以定义引用的第三方依赖库和混淆规则，配置好ProGuard之后，用AS到处apk即可



