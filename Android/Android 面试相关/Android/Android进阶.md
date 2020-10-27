# Android进阶

## 一、Android多线程断点续传

### 1. 原理

其实断点续传的原理很简单，从字面上理解，所谓断点续传就是从停止的地方重新下载。

断点：线程停止的位置。

续传：从停止的位置重新下载。

用代码解析就是：

断点 ： 当前线程已经下载完成的数据长度。

续传 ： 向服务器请求上次线程停止位置之后的数据。

原理知道了，功能实现起来也简单。每当线程停止时就把已下载的数据长度写入记录文件，当重新下载时，从记录文件读取已经下载了的长度。而这个长度就是所需要的断点。

续传的实现也简单，可以通过设置网络请求参数，请求服务器从指定的位置开始读取数据。

而要实现这两个功能只需要使用到httpURLconnection里面的setRequestProperty方法便可以实现.

```java
public void setRequestProperty(String field, String newValue)
```

如下所示，便是向服务器请求500-1000之间的500个byte：

```Java
conn.setRequestProperty("Range", "bytes=" + 500 + "-" + 1000);
```

以上只是续传的一部分需求，当我们获取到下载数据时，还需要将数据写入文件，而普通发File对象并不提供从指定位置写入数据的功能，这个时候，就需要使用到RandomAccessFile来实现从指定位置给文件写入数据的功能。

```java
public void seek(long offset)
```

如下所示，便是从文件的的第100个byte后开始写入数据。

```java
raFile.seek(100);
```

而开始写入数据时还需要用到RandomAccessFile里面的另外一个方法

```java
public void write(byte[] buffer, int byteOffset, int byteCount)
```

该方法的使用和OutputStream的write的使用一模一样...

以上便是断点续传的原理。

### 2. 多线程断点续传

多线程断点续传便是在单线程的断点续传上延伸的。多线程断点续传是把整个文件分割成几个部分，每个部分由一条线程执行下载，而每一条下载线程都要实现断点续传功能。
为了实现文件分割功能，我们需要使用到httpURLconnection的另外一个方法：

```java
public int getContentLength()
```

当请求成功时，可以通过该方法获取到文件的总长度。
`每一条线程下载大小 = fileLength / THREAD_NUM`

如下图所示，描述的便是多线程的下载模型：

![](http://upload-images.jianshu.io/upload_images/1824042-411c25f0cb1927de?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在多线程断点续传下载中，有一点需要特别注意：
由于文件是分成多个部分是被不同的线程的同时下载的，这就需要，每一条线程都分别需要有一个断点记录，和一个线程完成状态的记录；

![](http://upload-images.jianshu.io/upload_images/1824042-843517c30becdcd6?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

只有所有线程的下载状态都处于完成状态时，才能表示文件已经下载完成。
实现记录的方法多种多样，我这里采用的是JDK自带的Properties类来记录下载参数。

### 3. 断点续传结构

通过原理的了解，便可以很快的设计出断点续传工具类的基本结构图

![](http://upload-images.jianshu.io/upload_images/1824042-74a1e86dee5bd13c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**IDownloadListener.java**

```java
    package com.arialyy.frame.http.inf;
    import java.net.HttpURLConnection;

    /** 
     * 在这里面编写你的业务逻辑
     */
    public interface IDownloadListener {
        /**
         * 取消下载
         */
        public void onCancel();

        /**
         * 下载失败
         */
        public void onFail();

        /**
         * 下载预处理,可通过HttpURLConnection获取文件长度
         */
        public void onPreDownload(HttpURLConnection connection);

        /**
         * 下载监听
         */
        public void onProgress(long currentLocation);

        /**
         * 单一线程的结束位置
         */
        public void onChildComplete(long finishLocation);

        /**
         * 开始
         */
        public void onStart(long startLocation);

        /**
         * 子程恢复下载的位置
         */
        public void onChildResume(long resumeLocation);

        /**
         * 恢复位置
         */
        public void onResume(long resumeLocation);

        /**
         * 停止
         */
        public void onStop(long stopLocation);

        /**
         * 下载完成
         */
        public void onComplete();
    }
```

该类是下载监听接口

**DownloadListener.java**

```java
import java.net.HttpURLConnection;

/**
 * 下载监听
 */
public class DownloadListener implements IDownloadListener {

    @Override
    public void onResume(long resumeLocation) {

    }

    @Override
    public void onCancel() {

    }

    @Override
    public void onFail() {

    }

    @Override
    public void onPreDownload(HttpURLConnection connection) {

    }

    @Override
    public void onProgress(long currentLocation) {

    }

    @Override
    public void onChildComplete(long finishLocation) {

    }

    @Override
    public void onStart(long startLocation) {

    }

    @Override
    public void onChildResume(long resumeLocation) {

    }

    @Override
    public void onStop(long stopLocation) {

    }

    @Override
    public void onComplete() {

    }
}
```

**下载参数实体**

```java
    /**
     * 子线程下载信息类
     */
    private class DownloadEntity {
        //文件总长度
        long fileSize;
        //下载链接
        String downloadUrl;
        //线程Id
        int threadId;
        //起始下载位置
        long startLocation;
        //结束下载的文章
        long endLocation;
        //下载文件
        File tempFile;
        Context context;

        public DownloadEntity(Context context, long fileSize, String downloadUrl, File file, int threadId, long startLocation, long endLocation) {
            this.fileSize = fileSize;
            this.downloadUrl = downloadUrl;
            this.tempFile = file;
            this.threadId = threadId;
            this.startLocation = startLocation;
            this.endLocation = endLocation;
            this.context = context;
        }
    }
```

该类是下载信息配置类，每一条子线程的下载都需要一个下载实体来配置下载信息。

**下载任务线程**

```java
    /**
     * 多线程下载任务类
     */
    private class DownLoadTask implements Runnable {
        private static final String TAG = "DownLoadTask";
        private DownloadEntity dEntity;
        private String configFPath;

        public DownLoadTask(DownloadEntity downloadInfo) {
            this.dEntity = downloadInfo;
            configFPath = dEntity.context.getFilesDir().getPath() + "/temp/" + dEntity.tempFile.getName() + ".properties";
        }

        @Override
        public void run() {
            try {
                L.d(TAG, "线程_" + dEntity.threadId + "_正在下载【" + "开始位置 : " + dEntity.startLocation + "，结束位置：" + dEntity.endLocation + "】");
                URL url = new URL(dEntity.downloadUrl);
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                //在头里面请求下载开始位置和结束位置
                conn.setRequestProperty("Range", "bytes=" + dEntity.startLocation + "-" + dEntity.endLocation);
                conn.setRequestMethod("GET");
                conn.setRequestProperty("Charset", "UTF-8");
                conn.setConnectTimeout(TIME_OUT);
                conn.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.2; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)");
                conn.setRequestProperty("Accept", "image/gif, image/jpeg, image/pjpeg, image/pjpeg, application/x-shockwave-flash, application/xaml+xml, application/vnd.ms-xpsdocument, application/x-ms-xbap, application/x-ms-application, application/vnd.ms-excel, application/vnd.ms-powerpoint, application/msword, */*");
                conn.setReadTimeout(2000);  //设置读取流的等待时间,必须设置该参数
                InputStream is = conn.getInputStream();
                //创建可设置位置的文件
                RandomAccessFile file = new RandomAccessFile(dEntity.tempFile, "rwd");
                //设置每条线程写入文件的位置
                file.seek(dEntity.startLocation);
                byte[] buffer = new byte[1024];
                int len;
                //当前子线程的下载位置
                long currentLocation = dEntity.startLocation;
                while ((len = is.read(buffer)) != -1) {
                    if (isCancel) {
                        L.d(TAG, "++++++++++ thread_" + dEntity.threadId + "_cancel ++++++++++");
                        break;
                    }

                    if (isStop) {
                        break;
                    }

                    //把下载数据数据写入文件
                    file.write(buffer, 0, len);
                    synchronized (DownLoadUtil.this) {
                        mCurrentLocation += len;
                        mListener.onProgress(mCurrentLocation);
                    }
                    currentLocation += len;
                }
                file.close();
                is.close();

                if (isCancel) {
                    synchronized (DownLoadUtil.this) {
                        mCancelNum++;
                        if (mCancelNum == THREAD_NUM) {
                            File configFile = new File(configFPath);
                            if (configFile.exists()) {
                                configFile.delete();
                            }

                            if (dEntity.tempFile.exists()) {
                                dEntity.tempFile.delete();
                            }
                            L.d(TAG, "++++++++++++++++ onCancel +++++++++++++++++");
                            isDownloading = false;
                            mListener.onCancel();
                            System.gc();
                        }
                    }
                    return;
                }

                //停止状态不需要删除记录文件
                if (isStop) {
                    synchronized (DownLoadUtil.this) {
                        mStopNum++;
                        String location = String.valueOf(currentLocation);
                        L.i(TAG, "thread_" + dEntity.threadId + "_stop, stop location ==> " + currentLocation);
                        writeConfig(dEntity.tempFile.getName() + "_record_" + dEntity.threadId, location);
                        if (mStopNum == THREAD_NUM) {
                            L.d(TAG, "++++++++++++++++ onStop +++++++++++++++++");
                            isDownloading = false;
                            mListener.onStop(mCurrentLocation);
                            System.gc();
                        }
                    }
                    return;
                }

                L.i(TAG, "线程【" + dEntity.threadId + "】下载完毕");
                writeConfig(dEntity.tempFile.getName() + "_state_" + dEntity.threadId, 1 + "");
                mListener.onChildComplete(dEntity.endLocation);
                mCompleteThreadNum++;
                if (mCompleteThreadNum == THREAD_NUM) {
                    File configFile = new File(configFPath);
                    if (configFile.exists()) {
                        configFile.delete();
                    }
                    mListener.onComplete();
                    isDownloading = false;
                    System.gc();
                }
            } catch (MalformedURLException e) {
                e.printStackTrace();
                isDownloading = false;
                mListener.onFail();
            } catch (IOException e) {
                FL.e(this, "下载失败【" + dEntity.downloadUrl + "】" + FL.getPrintException(e));
                isDownloading = false;
                mListener.onFail();
            } catch (Exception e) {
                FL.e(this, "获取流失败" + FL.getPrintException(e));
                isDownloading = false;
                mListener.onFail();
            }
        }
```

这个是每条下载子线程的下载任务类，子线程通过下载实体对每一条线程进行下载配置，由于在多断点续传的概念里，停止表示的是暂停状态，而恢复表示的是线程从记录的断点重新进行下载，所以，线程处于停止状态时是不能删除记录文件的。

**下载入口**

```java
    /**
     * 多线程断点续传下载文件，暂停和继续
     *
     * @param context          必须添加该参数，不能使用全局变量的context
     * @param downloadUrl      下载路径
     * @param filePath         保存路径
     * @param downloadListener 下载进度监听 {@link DownloadListener}
     */
    public void download(final Context context, @NonNull final String downloadUrl, @NonNull final String filePath,
                         @NonNull final DownloadListener downloadListener) {
        isDownloading = true;
        mCurrentLocation = 0;
        isStop = false;
        isCancel = false;
        mCancelNum = 0;
        mStopNum = 0;
        final File dFile = new File(filePath);
        //读取已完成的线程数
        final File configFile = new File(context.getFilesDir().getPath() + "/temp/" + dFile.getName() + ".properties");
        try {
            if (!configFile.exists()) { //记录文件被删除，则重新下载
                newTask = true;
                FileUtil.createFile(configFile.getPath());
            } else {
                newTask = false;
            }
        } catch (Exception e) {
            e.printStackTrace();
            mListener.onFail();
            return;
        }
        newTask = !dFile.exists();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    mListener = downloadListener;
                    URL url = new URL(downloadUrl);
                    HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                    conn.setRequestMethod("GET");
                    conn.setRequestProperty("Charset", "UTF-8");
                    conn.setConnectTimeout(TIME_OUT);
                    conn.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.2; Trident/4.0; .NET CLR 1.1.4322; .NET CLR 2.0.50727; .NET CLR 3.0.04506.30; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729)");
                    conn.setRequestProperty("Accept", "image/gif, image/jpeg, image/pjpeg, image/pjpeg, application/x-shockwave-flash, application/xaml+xml, application/vnd.ms-xpsdocument, application/x-ms-xbap, application/x-ms-application, application/vnd.ms-excel, application/vnd.ms-powerpoint, application/msword, */*");
                    conn.connect();
                    int len = conn.getContentLength();
                    if (len < 0) {  //网络被劫持时会出现这个问题
                        mListener.onFail();
                        return;
                    }
                    int code = conn.getResponseCode();
                    if (code == 200) {
                        int fileLength = conn.getContentLength();
                        //必须建一个文件
                        FileUtil.createFile(filePath);
                        RandomAccessFile file = new RandomAccessFile(filePath, "rwd");
                        //设置文件长度
                        file.setLength(fileLength);
                        mListener.onPreDownload(conn);
                        //分配每条线程的下载区间
                        Properties pro = null;
                        pro = Util.loadConfig(configFile);
                        int blockSize = fileLength / THREAD_NUM;
                        SparseArray<Thread> tasks = new SparseArray<>();
                        for (int i = 0; i < THREAD_NUM; i++) {
                            long startL = i * blockSize, endL = (i + 1) * blockSize;
                            Object state = pro.getProperty(dFile.getName() + "_state_" + i);
                            if (state != null && Integer.parseInt(state + "") == 1) {  //该线程已经完成
                                mCurrentLocation += endL - startL;
                                L.d(TAG, "++++++++++ 线程_" + i + "_已经下载完成 ++++++++++");
                                mCompleteThreadNum++;
                                if (mCompleteThreadNum == THREAD_NUM) {
                                    if (configFile.exists()) {
                                        configFile.delete();
                                    }
                                    mListener.onComplete();
                                    isDownloading = false;
                                    System.gc();
                                    return;
                                }
                                continue;
                            }
                            //分配下载位置
                            Object record = pro.getProperty(dFile.getName() + "_record_" + i);
                            if (!newTask && record != null && Long.parseLong(record + "") > 0) {       //如果有记录，则恢复下载
                                Long r = Long.parseLong(record + "");
                                mCurrentLocation += r - startL;
                                L.d(TAG, "++++++++++ 线程_" + i + "_恢复下载 ++++++++++");
                                mListener.onChildResume(r);
                                startL = r;
                            }
                            if (i == (THREAD_NUM - 1)) {
                                endL = fileLength;//如果整个文件的大小不为线程个数的整数倍，则最后一个线程的结束位置即为文件的总长度
                            }
                            DownloadEntity entity = new DownloadEntity(context, fileLength, downloadUrl, dFile, i, startL, endL);
                            DownLoadTask task = new DownLoadTask(entity);
                            tasks.put(i, new Thread(task));
                        }
                        if (mCurrentLocation > 0) {
                            mListener.onResume(mCurrentLocation);
                        } else {
                            mListener.onStart(mCurrentLocation);
                        }
                        for (int i = 0, count = tasks.size(); i < count; i++) {
                            Thread task = tasks.get(i);
                            if (task != null) {
                                task.start();
                            }
                        }
                    } else {
                        FL.e(TAG, "下载失败，返回码：" + code);
                        isDownloading = false;
                        System.gc();
                        mListener.onFail();
                    }
                } catch (IOException e) {
                    FL.e(this, "下载失败【downloadUrl:" + downloadUrl + "】\n【filePath:" + filePath + "】" + FL.getPrintException(e));
                    isDownloading = false;
                    mListener.onFail();
                }
            }
        }).start();
    }
```

其实也没啥好说的，注释已经很完整了，需要注意两点
1、恢复下载时：`已下载的文件大小 = 该线程的上一次断点的位置 - 该线程起始下载位置`；
2、为了保证下载文件的完整性，只要记录文件不存在就需要重新进行下载；

**最终效果**

![](http://upload-images.jianshu.io/upload_images/1824042-1fc216854a28525f?imageMogr2/auto-orient/strip)

[Demo点我](https://github.com/AriaLyy/DownloadUtil)

## 二、Android全局异常处理

### 1. CrashHandler

```Java
/** 
 * 自定义的 异常处理类 , 实现了 UncaughtExceptionHandler接口  
 * 
 */  
public class CrashHandler implements UncaughtExceptionHandler {  
    // 需求是 整个应用程序 只有一个 MyCrash-Handler   
    private static CrashHandler INSTANCE ;  
    private Context context;  
      
    //1.私有化构造方法  
    private CrashHandler(){  
          
    }  
      
    public static synchronized CrashHandler getInstance(){  
        if (INSTANCE == null)  
            INSTANCE = new CrashHandler();  
        return INSTANCE;
    }

    public void init(Context context){  
        this.context = context;
    }  
      
  
    public void uncaughtException(Thread arg0, Throwable arg1) {  
        System.out.println("程序挂掉了 ");  
        // 在此可以把用户手机的一些信息以及异常信息捕获并上传,由于UMeng在这方面有很程序的api接口来调用，故没有考虑
          
        //干掉当前的程序   
        android.os.Process.killProcess(android.os.Process.myPid());  
    }  

}  
```

### 2. CrashApplication

```java
/** 
 * 在开发应用时都会和Activity打交道，而Application使用的就相对较少了。 
 * Application是用来管理应用程序的全局状态的，比如载入资源文件。 
 * 在应用程序启动的时候Application会首先创建，然后才会根据情况(Intent)启动相应的Activity或者Service。 
 * 在本文将在Application中注册未捕获异常处理器。 
 */  
public class CrashApplication extends Application {  
    @Override  
    public void onCreate() {  
        super.onCreate();  
        CrashHandler handler = CrashHandler.getInstance();  
        handler.init(getApplicationContext());
        Thread.setDefaultUncaughtExceptionHandler(handler);  
    }  
}  
```

### 3. 在AndroidManifest.xml中注册

```xml
<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
    package="org.wp.activity" android:versionCode="1" android:versionName="1.0">  
    <application android:icon="@drawable/icon" android:label="@string/app_name"  
        android:name=".CrashApplication" android:debuggable="true">  
        <activity android:name=".MainActivity" android:label="@string/app_name">  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN" />  
                <category android:name="android.intent.category.LAUNCHER" />  
            </intent-filter>  
        </activity>  
    </application>  
    <uses-sdk android:minSdkVersion="8" />  
</manifest> 
```

至此，可以测试下在出错的时候程序会直接闪退，并杀死后台进程。当然也可以自定义一些比较友好的出错UI提示，进一步提升用户体验。

## 三、Android MVP模式详解

### 1. MVP概述

**MVP，全称 Model-View-Presenter，即模型-视图-层现器**。

提到MVP，就必须要先介绍一下它的前辈MVC，因为MVP正是基于MVC的基础发展而来的。两个之间的关系也是源远流长。

**MVC，全称Model-View-Controller，即模型-视图-控制器。** 具体如下：

**View：对应于布局文件**

Model：业务逻辑和实体模型

Controllor：对应于Activity

但是View对应于布局文件，其实能做的事情特别少，实际上关于该布局文件中的数据绑定的操作，事件处理的代码都在Activity中，造成了Activity既像View又像Controller，使得Activity变得臃肿。

而当将架构改为MVP以后，Presenter的出现，将Actvity视为View层，Presenter负责完成View层与Model层的交互。现在是这样的：

**View 对应于Activity，负责View的绘制以及与用户交互**

Model 依然是业务逻辑和实体模型

Presenter 负责完成View于Model间的交互

下面两幅图通过数据与视图之间的交互清楚地展示了这种变化：

![img](http://upload-images.jianshu.io/upload_images/3985563-4fd0f30f81ae423e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MVC模式下实际上就是Activty与Model之间交互，View完全独立出来了。

![img](http://upload-images.jianshu.io/upload_images/3985563-7c936f2223dce1de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MVP模式通过Presenter实现数据和视图之间的交互，简化了Activity的职责。同时即避免了View和Model的直接联系，又通过Presenter实现两者之间的沟通。

**总结：MVP模式减少了Activity的职责，简化了Activity中的代码，将复杂的逻辑代码提取到了Presenter中进行处理，模块职责划分明显，层次清晰。与之对应的好处就是，耦合度更低，更方便的进行测试。**

**MVC和MVP的区别**

![img](http://upload-images.jianshu.io/upload_images/3985563-4472518336d7465e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**MVC中是允许Model和View进行交互的，而MVP中很明显，Model与View之间的交互由Presenter完成。还有一点就是Presenter与View之间的交互是通过接口的。**

还有一点注意：**MVC中V对应的是布局文件，MVP中V对应的是Activity。**

### 2. MVP的简单使用

大多数MVP模式的示例都使用登录案例进行介绍。因为简单方便，同时能提现出MVP的特点。今天我们也以此例进行学习。
使用MVP的好处之一就是模块职责划分明显，层次清晰。
该例的结构图即可展现此优点。

![img](http://upload-images.jianshu.io/upload_images/3985563-3f3d046d40d6bfbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**1.Model层**

在本例中，Model层负责对从登录页面获取地帐号密码进行验证（一般需要请求服务器进行验证，本例直接模拟这一过程）。
从上图的包结构图中可以看出，Model层包含内容：

①实体类bean

②接口，表示Model层所要执行的业务逻辑

③接口实现类，具体实现业务逻辑，包含的一些主要方法

下面以代码的形式一一展开。



①实体类bean

```java
public class User {
    private String password;
    private String username;

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    @Override
    public String toString() {
        return "User{" +
                "password='" + password + '\'' +
                ", username='" + username + '\'' +
                '}';
    }
}
```

封装了用户名、密码，方便数据传递。

②接口

```java
public interface LoginModel {
    void login(User user, OnLoginFinishedListener listener);
}
```

其中OnLoginFinishedListener 是presenter层的接口，方便实现回调presenter，通知presenter业务逻辑的返回结果，具体在presenter层介绍。

③接口实现类

```java
public class LoginModelImpl implements LoginModel {
    @Override
    public void login(User user, final OnLoginFinishedListener listener) {
        final String username = user.getUsername();
        final String password = user.getPassword();
        new Handler().postDelayed(new Runnable() {
            @Override public void run() {
                boolean error = false;
                if (TextUtils.isEmpty(username)){
                    listener.onUsernameError();//model层里面回调listener
                    error = true;
                }
                if (TextUtils.isEmpty(password)){
                    listener.onPasswordError();
                    error = true;
                }
                if (!error){
                    listener.onSuccess();
                }
            }
        }, 2000);
    }
}
```

实现Model层逻辑：延时模拟登陆（2s），如果用户名或者密码为空则登陆失败，否则登陆成功。

**2.View层**

视图：将Modle层请求的数据呈现给用户。一般的视图都只是包含用户界面(UI)，而不包含界面逻辑，界面逻辑由Presenter来实现。

从上图的包结构图中可以看出，View包含内容：

①接口，上面我们说过Presenter与View交互是通过接口。其中接口中方法的定义是根据Activity用户交互需要展示的控件确定的。

②接口实现类，将上述定义的接口中的方法在Activity中对应实现具体操作。

下面以代码的形式一一展开。

①接口

```java
public interface LoginView {
    //login是个耗时操作，我们需要给用户一个友好的提示，一般就是操作ProgressBar
    void showProgress();

    void hideProgress();
   //login当然存在登录成功与失败的处理，失败给出提示
    void setUsernameError();

    void setPasswordError();
   //login成功，也给个提示
    void showSuccess();
}
```

上述5个方法都是presenter根据model层返回结果需要view执行的对应的操作。

②接口实现类

即对应的登录的Activity，需要实现LoginView接口。

```java
public class LoginActivity extends AppCompatActivity implements LoginView, View.OnClickListener {
    private ProgressBar progressBar;
    private EditText username;
    private EditText password;
    private LoginPresenter presenter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        progressBar = (ProgressBar) findViewById(R.id.progress);
        username = (EditText) findViewById(R.id.username);
        password = (EditText) findViewById(R.id.password);
        findViewById(R.id.button).setOnClickListener(this);
       //创建一个presenter对象，当点击登录按钮时，让presenter去调用model层的login()方法，验证帐号密码
        presenter = new LoginPresenterImpl(this);
    }

    @Override
    protected void onDestroy() {
        presenter.onDestroy();
        super.onDestroy();
    }

    @Override
    public void showProgress() {
        progressBar.setVisibility(View.VISIBLE);
    }

    @Override
    public void hideProgress() {
        progressBar.setVisibility(View.GONE);
    }

    @Override
    public void setUsernameError() {
        username.setError(getString(R.string.username_error));
    }

    @Override
    public void setPasswordError() {
        password.setError(getString(R.string.password_error));
    }

    @Override
    public void showSuccess() {
         progressBar.setVisibility(View.GONE);
        Toast.makeText(this,"login success",Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onClick(View v) {
        User user = new User();
        user.setPassword(password.getText().toString());
        user.setUsername(username.getText().toString());
        presenter.validateCredentials(user);
    }

}
```

View层实现Presenter层需要调用的控件操作，方便Presenter层根据Model层返回的结果进行操作View层进行对应的显示。

**3.Presenter层**

Presenter是用作Model和View之间交互的桥梁。
从上图的包结构图中可以看出，Presenter包含内容：

①接口，包含Presenter需要进行Model和View之间交互逻辑的接口，以及上面提到的Model层数据请求完成后回调的接口。

②接口实现类，即实现具体的Presenter类逻辑。

下面以代码的形式一一展开。

①接口

```java
public interface OnLoginFinishedListener {
    void onUsernameError();

    void onPasswordError();

    void onSuccess();
}
```

当Model层得到请求的结果，需要回调Presenter层，让Presenter层调用View层的接口方法。

```java
public interface LoginPresenter {
    void validateCredentials(User user);

    void onDestroy();
}
```

登陆的Presenter 的接口，实现类为LoginPresenterImpl，完成登陆的验证，以及销毁当前view。

②接口实现类

```java
public class LoginPresenterImpl implements LoginPresenter, OnLoginFinishedListener {
    private LoginView loginView;
    private LoginModel loginModel;

    public LoginPresenterImpl(LoginView loginView) {
        this.loginView = loginView;
        this.loginModel = new LoginModelImpl();
    }

    @Override
    public void validateCredentials(User user) {
        if (loginView != null) {
            loginView.showProgress();
        }

        loginModel.login(user, this);
    }

    @Override
    public void onDestroy() {
        loginView = null;
    }

    @Override
    public void onUsernameError() {
        if (loginView != null) {
            loginView.setUsernameError();
            loginView.hideProgress();
        }
    }

    @Override
    public void onPasswordError() {
        if (loginView != null) {
            loginView.setPasswordError();
            loginView.hideProgress();
        }
    }

    @Override
    public void onSuccess() {
        if (loginView != null) {
            loginView.showSuccess();
        }
    }
}
```

由于presenter完成二者的交互，那么肯定需要二者的实现类（通过传入参数，或者new）。

presenter里面有个OnLoginFinishedListener， 其在Presenter层实现，给Model层回调，更改View层的状态， 确保 Model层不直接操作View层。

**示例展示：**

![img](http://upload-images.jianshu.io/upload_images/3985563-216404156964b311.gif?imageMogr2/auto-orient/strip)

[代码地址](https://github.com/LRH1993/MVPdemo)

### 3. 总结

MVP模式的整个核心流程：

View与Model并不直接交互，而是使用Presenter作为View与Model之间的桥梁。其中Presenter中同时持有View层的Interface的引用以及Model层的引用，而View层持有Presenter层引用。当View层某个界面需要展示某些数据的时候，首先会调用Presenter层的引用，然后Presenter层会调用Model层请求数据，当Model层数据加载成功之后会调用Presenter层的回调方法通知Presenter层数据加载情况，最后Presenter层再调用View层的接口将加载后的数据展示给用户。

![img](http://upload-images.jianshu.io/upload_images/3985563-03352e00ce8b4083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四、Android Binder机制及AIDL使用

### 1. 概述

Android系统中，涉及到多进程间的通信底层都是依赖于Binder IPC机制。例如当进程A中的Activity要向进程B中的Service通信，这便需要依赖于Binder IPC。不仅于此，整个Android系统架构中，大量采用了Binder机制作为IPC（进程间通信，Interprocess Communication）方案。

当然也存在部分其他的IPC方式，如管道、SystemV、Socket等。那么Android为什么不使用这些原有的技术，而是要使开发一种新的叫Binder的进程间通信机制呢？

**为什么要使用Binder？**

**性能方面**

在移动设备上（性能受限制的设备，比如要省电），广泛地使用跨进程通信对通信机制的性能有严格的要求，Binder相对于传统的Socket方式，更加高效。**Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次，共享内存方式一次内存拷贝都不需要，但实现方式又比较复杂。**

**安全方面**

传统的进程通信方式对于通信双方的身份并没有做出严格的验证，比如Socket通信的IP地址是客户端手动填入，很容易进行伪造。然而，Binder机制从协议本身就支持对通信双方做身份校检，从而大大提升了安全性。

### 2.  Binder

#### IPC原理

从进程角度来看IPC（Interprocess Communication）机制

![img](http://upload-images.jianshu.io/upload_images/3985563-a3722ee387793114.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个Android的进程，只能运行在自己进程所拥有的虚拟地址空间。例如，对应一个4GB的虚拟地址空间，其中3GB是用户空间，1GB是内核空间。当然内核空间的大小是可以通过参数配置调整的。对于用户空间，不同进程之间是不能共享的，而内核空间却是可共享的。Client进程向Server进程通信，恰恰是利用进程间可共享的内核内存空间来完成底层通信工作的。Client端与Server端进程往往采用ioctl(input/output control)等方法与内核空间的驱动进行交互。

#### Binder原理

Binder通信采用C/S架构，从组件视角来说，包含Client、Server、ServiceManager以及Binder驱动，其中ServiceManager用于管理系统中的各种服务。架构图如下所示：

![img](http://upload-images.jianshu.io/upload_images/3985563-5ff2c4816543c433.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Binder通信的四个角色**

**Client进程**：使用服务的进程。

**Server进程**：提供服务的进程。

**ServiceManager进程**：ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。

**Binder驱动**：驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。

**Binder运行机制**

图中Client/Server/ServiceManage之间的相互通信都是基于Binder机制。既然基于Binder机制通信，那么同样也是C/S架构，则图中的3大步骤都有相应的Client端与Server端。

**注册服务(addService)**：Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。

**获取服务(getService)**：Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。

**使用服务**：Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：Client是客户端，Server是服务端。

图中的Client，Server，Service Manager之间交互都是虚线表示，是由于它们彼此之间不是直接交互的，而是都通过与Binder驱动进行交互的，从而实现IPC通信（Interprocess Communication）方式。其中Binder驱动位于内核空间，Client，Server，Service Manager位于用户空间。Binder驱动和Service Manager可以看做是Android平台的基础架构，而Client和Server是Android的应用层，开发人员只需自定义实现Client、Server端，借助Android的基本平台架构便可以直接进行IPC通信。

**Binder运行的实例解释**

首先我们看看我们的程序跨进程调用系统服务的简单示例，实现浮动窗口部分代码：

```java
//获取WindowManager服务引用
WindowManager wm = (WindowManager) getSystemService(getApplication().WINDOW_SERVICE);
//布局参数layoutParams相关设置略...
View view = LayoutInflater.from(getApplication()).inflate(R.layout.float_layout, null);
//添加view
wm.addView(view, layoutParams);
```

**注册服务(addService)：** 在Android开机启动过程中，Android会初始化系统的各种Service，并将这些Service向ServiceManager注册（即让ServiceManager管理）。这一步是系统自动完成的。

**获取服务(getService)：** 客户端想要得到具体的Service直接向ServiceManager要即可。客户端首先向ServiceManager查询得到具体的Service引用，通常是Service引用的代理对象，对数据进行一些处理操作。即第2行代码中，得到的wm是WindowManager对象的引用。

**使用服务：** 通过这个引用向具体的服务端发送请求，服务端执行完成后就返回。即第6行调用WindowManager的addView函数，将触发远程调用，调用的是运行在systemServer进程中的WindowManager的addView函数。

**使用服务的具体执行过程**

![img](http://upload-images.jianshu.io/upload_images/3985563-727dd63017d2113b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. Client通过获得一个Server的代理接口，对Server进行调用。
2. 代理接口中定义的方法与Server中定义的方法是一一对应的。
3. Client调用某个代理接口中的方法时，代理接口的方法会将Client传递的参数打包成Parcel对象。
4. 代理接口将Parcel发送给内核中的Binder Driver。
5. Server会读取Binder Driver中的请求数据，如果是发送给自己的，解包Parcel对象，处理并将结果返回。
6. 整个的调用过程是一个同步过程，在Server处理的时候，Client会Block住。**因此Client调用过程不应在主线程。**

### 3. AIDL的简介

AIDL (Android Interface Definition Language) 是一种接口定义语言，用于生成可以在Android设备上两个进程之间进行进程间通信(Interprocess Communication, IPC)的代码。如果在一个进程中（例如Activity）要调用另一个进程中（例如Service）对象的操作，就可以使用AIDL生成可序列化的参数，来完成进程间通信。

**简言之，AIDL能够实现进程间通信，其内部是通过Binder机制来实现的，后面会具体介绍，现在先介绍AIDL的使用。**

### 4. AIDL的具体使用

AIDL的实现一共分为三部分，一部分是客户端，调用远程服务。一部分是服务端，提供服务。最后一部分，也是最关键的是AIDL接口，用来传递的参数，提供进程间通信。

先在服务端创建AIDL部分代码。

**AIDL文件**
通过如下方式新建一个AIDL文件

![img](http://upload-images.jianshu.io/upload_images/3985563-4ea35902ebf6fa51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**默认生成格式**

```java
interface IBookManager {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```

默认如下格式，由于本例要操作Book类，实现两个方法，添加书本和返回书本列表。

**定义一个Book类，实现Parcelable接口。**

```java
public class Book implements Parcelable {
    public int bookId;
    public String bookName;

    public Book() {
    }

    public Book(int bookId, String bookName) {
        this.bookId = bookId;
        this.bookName = bookName;
    }

    public int getBookId() {
        return bookId;
    }

    public void setBookId(int bookId) {
        this.bookId = bookId;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(this.bookId);
        dest.writeString(this.bookName);
    }

    protected Book(Parcel in) {
        this.bookId = in.readInt();
        this.bookName = in.readString();
    }

    public static final Parcelable.Creator<Book> CREATOR = new Parcelable.Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel source) {
            return new Book(source);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };
}
```

由于AIDL只支持数据类型:基本类型（int，long，char，boolean等），String，CharSequence，List，Map，其他类型必须使用import导入，即使它们可能在同一个包里，比如上面的Book。

**最终IBookManager.aidl 的实现**

```java
// Declare any non-default types here with import statements
import com.lvr.aidldemo.Book;

interface IBookManager {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);

    void addBook(in Book book);

    List<Book> getBookList();

}
```

**注意：如果自定义的Parcelable对象，必须创建一个和它同名的AIDL文件，并在其中声明它为parcelable类型。**

**Book.aidl**

```Java
// Book.aidl
package com.lvr.aidldemo;

parcelable Book;
```

以上就是AIDL部分的实现，一共三个文件。

然后Make Project ，SDK为自动为我们生成对应的Binder类。

在如下路径下：

![img](http://upload-images.jianshu.io/upload_images/3985563-53d6a0fdeafefa74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中该接口中有个重要的内部类Stub ，继承了Binder 类，同时实现了IBookManager接口。
这个内部类是接下来的关键内容。

```java
public static abstract class Stub extends android.os.Binder implements com.lvr.aidldemo.IBookManager{}
```

**服务端**

服务端首先要创建一个Service用来监听客户端的连接请求。然后在Service中实现Stub 类，并定义接口中方法的具体实现。

```java
//实现了AIDL的抽象函数
private IBookManager.Stub mbinder = new IBookManager.Stub() {
    @Override
    public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) throws RemoteException {
        //什么也不做
    }

    @Override
    public void addBook(Book book) throws RemoteException {
        //添加书本
        if (!mBookList.contains(book)) {
            mBookList.add(book);
        }
    }

    @Override
    public List<Book> getBookList() throws RemoteException {
        return mBookList;
    }
};
```

当客户端连接服务端，服务端就会调用如下方法：

```java
public IBinder onBind(Intent intent) {
    return mbinder;
}
```

就会把Stub实现对象返回给客户端，该对象是个Binder对象，可以实现进程间通信。
本例就不真实模拟两个应用之间的通信，而是让Service另外开启一个进程来模拟进程间通信。

``` xml
<service
    android:name=".MyService"
    android:process=":remote">
    <intent-filter>
        <category android:name="android.intent.category.DEFAULT" />
        <action android:name="com.lvr.aidldemo.MyService" />
    </intent-filter>
</service>
```

`android:process=":remote"`设置为另一个进程。`<action android:name="com.lvr.aidldemo.MyService"/>`是为了能让其他apk隐式bindService。**通过隐式调用的方式来连接service，需要把category设为default，这是因为，隐式调用的时候，intent中的category默认会被设置为default。**

**客户端**

**首先将服务端工程中的aidl文件夹下的内容整个拷贝到客户端工程的对应位置下，由于本例的使用在一个应用中，就不需要拷贝了，其他情况一定不要忘记这一步。**

客户端需要做的事情比较简单，首先需要绑定服务端的Service。

```java
Intent intentService = new Intent();
intentService.setAction("com.lvr.aidldemo.MyService");
intentService.setPackage(getPackageName());
intentService.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
MyClient.this.bindService(intentService, mServiceConnection, BIND_AUTO_CREATE);
Toast.makeText(getApplicationContext(), "绑定了服务", Toast.LENGTH_SHORT).show();
```

将服务端返回的Binder对象转换成AIDL接口所属的类型，接着就可以调用AIDL中的方法了。

```java
if (mIBookManager != null) {
    try {
        mIBookManager.addBook(new Book(18, "新添加的书"));
        Toast.makeText(getApplicationContext(), mIBookManager.getBookList().size() + "", Toast.LENGTH_SHORT).show();
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}
```

### 5. AIDL的工作原理

Binder机制的运行主要包括三个部分：注册服务、获取服务和使用服务。
其中注册服务和获取服务的流程涉及C的内容，由于个人能力有限，就不予介绍了。

本篇文章主要介绍使用服务时，AIDL的工作原理。

**① Binder对象的获取**

Binder是实现跨进程通信的基础，那么Binder对象在服务端和客户端是共享的，是同一个Binder对象。在客户端通过Binder对象获取实现了IInterface接口的对象来调用远程服务，然后通过Binder来实现参数传递。

那么如何维护实现了IInterface接口的对象和获取Binder对象呢？

**服务端获取Binder对象并保存IInterface接口对象**

Binder中两个关键方法：

```java
public class Binder implement IBinder {
    void attachInterface(IInterface plus, String descriptor)

    IInterface queryLocalInterface(Stringdescriptor) //从IBinder中继承而来
    ..........................
}
```

Binder具有被跨进程传输的能力是因为它实现了IBinder接口。系统会为每个实现了该接口的对象提供跨进程传输，这是系统给我们的一个很大的福利。

**Binder具有的完成特定任务的能力是通过它的IInterface的对象获得的**，我们可以简单理解attachInterface方法会将（descriptor，plus）作为（key,value）对存入Binder对象中的一个Map<String,IInterface>对象中，Binder对象可通过attachInterface方法持有一个IInterface对象（即plus）的引用，并依靠它获得完成特定任务的能力。queryLocalInterface方法可以认为是根据key值（即参数 descriptor）查找相应的IInterface对象。

在服务端进程，通过实现`private IBookManager.Stub mbinder = new IBookManager.Stub() {}`抽象类，获得Binder对象。
并保存了IInterface对象。

```java
public Stub() {
    this.attachInterface(this, DESCRIPTOR);
}
```

**客户端获取Binder对象并获取IInterface接口对象**

通过bindService获得Binder对象

```java
MyClient.this.bindService(intentService, mServiceConnection, BIND_AUTO_CREATE);
```

然后通过Binder对象获得IInterface对象。

```java
private ServiceConnection mServiceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder binder) {
        //通过服务端onBind方法返回的binder对象得到IBookManager的实例，得到实例就可以调用它的方法了
        mIBookManager = IBookManager.Stub.asInterface(binder);
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        mIBookManager = null;
    }
};
```

其中`asInterface(binder)`方法如下：

```java
public static com.lvr.aidldemo.IBookManager asInterface(android.os.IBinder obj) {
    if ((obj == null)) {
        return null;
    }
    android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
    if (((iin != null) && (iin instanceof com.lvr.aidldemo.IBookManager))) {
        return ((com.lvr.aidldemo.IBookManager) iin);
    }
    return new com.lvr.aidldemo.IBookManager.Stub.Proxy(obj);
}
```

先通过`queryLocalInterface(DESCRIPTOR);`查找到对应的IInterface对象，然后判断对象的类型，如果是同一个进程调用则返回IBookManager对象，由于是跨进程调用则返回Proxy对象，即Binder类的代理对象。

**② 调用服务端方法**

获得了Binder类的代理对象，并且通过代理对象获得了IInterface对象，那么就可以调用接口的具体实现方法了，来实现调用服务端方法的目的。

以addBook方法为例，调用该方法后，客户端线程挂起，等待唤醒：

```java
    @Override public void addBook(com.lvr.aidldemo.Book book) throws android.os.RemoteException
    {
        ..........
        //第一个参数：识别调用哪一个方法的ID
        //第二个参数：Book的序列化传入数据
        //第三个参数：调用方法后返回的数据
        //最后一个不用管
        mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
        _reply.readException();
    }
    ..........
}
```

省略部分主要完成对添加的Book对象进行序列化工作，然后调用`transact`方法。

Proxy对象中的transact调用发生后，会引起系统的注意，系统意识到Proxy对象想找它的真身Binder对象（系统其实一直存着Binder和Proxy的对应关系）。于是系统将这个请求中的数据转发给Binder对象，Binder对象将会在onTransact中收到Proxy对象传来的数据，于是它从data中取出客户端进程传来的数据，又根据第一个参数确定想让它执行添加书本操作，于是它就执行了响应操作，并把结果写回reply。代码概略如下：

```java
case TRANSACTION_addBook: {
    data.enforceInterface(DESCRIPTOR);
    com.lvr.aidldemo.Book _arg0;
    if ((0 != data.readInt())) {
        _arg0 = com.lvr.aidldemo.Book.CREATOR.createFromParcel(data);
    } else {
        _arg0 = null;
    }
    //这里调用服务端实现的addBook方法
    this.addBook(_arg0);
    reply.writeNoException();
    return true;
}
```

然后在`transact`方法获得`_reply`并返回结果，本例中的addList方法没有返回值。

客户端线程被唤醒。**因此调用服务端方法时，应开启子线程，防止UI线程堵塞，导致ANR。**



## 五、Android Parcelable和Serializable的区别

Serializable的作用是**为了保存对象的属性到本地文件、数据库、网络流、rmi以方便数据传输，当然这种传输可以是程序内的也可以是两个程序间的**。而Android的Parcelable的设计初衷是因为Serializable效率过慢，**为了在程序内不同组件间以及不同Android程序间(AIDL)高效**的传输数据而设计，这些数据仅在内存中存在，Parcelable是通过IBinder通信的消息的载体。

从上面的设计上我们就可以看出优劣了。

**效率及选择**

Parcelable的性能比Serializable好，在内存开销方面较小，所以**在内存间数据传输时推荐使用Parcelable**，如activity间传输数据，而Serializable可将数据持久化方便保存，所以**在需要保存或网络传输数据时选择Serializable**，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化。

**编程实现**

对于Serializable，类只需要实现Serializable接口，并提供一个序列化版本id(serialVersionUID)即可。而Parcelable则需要实现writeToParcel、describeContents函数以及静态的CREATOR变量，实际上就是**将如何打包和解包的工作自己来定义，而序列化的这些操作完全由底层实现**。

Parcelable的一个实现例子如下

```java
public class MyParcelable implements Parcelable {
     private int mData;
     private String mStr;

     public int describeContents() {
         return 0;
     }

     // 写数据进行保存
     public void writeToParcel(Parcel out, int flags) {
         out.writeInt(mData);
         out.writeString(mStr);
     }

     // 用来创建自定义的Parcelable的对象
     public static final Parcelable.Creator<MyParcelable> CREATOR
             = new Parcelable.Creator<MyParcelable>() {
         public MyParcelable createFromParcel(Parcel in) {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size) {
             return new MyParcelable[size];
         }
     };
     
     // 读数据进行恢复
     private MyParcelable(Parcel in) {
         mData = in.readInt();
         mStr = in.readString();
     }
 }
```

从上面我们可以看出Parcel的写入和读出顺序是一致的。如果元素是list读出时需要先new一个ArrayList传入，否则会报空指针异常。如下：

```java
list = new ArrayList<String>();
in.readStringList(list);
```

 PS: 在自己使用时，read数据时误将前面int数据当作long读出，结果后面的顺序错乱，报如下异常，当类字段较多时**务必保持写入和读取的类型及顺序一致**。

```
11-21 20:14:10.317: E/AndroidRuntime(21114): Caused by: java.lang.RuntimeException: Parcel android.os.Parcel@4126ed60: Unmarshalling unknown type code 3014773 at offset 164
```

**高级功能**

Serializable序列化不保存静态变量，可以使用Transient关键字对部分字段不进行序列化，也可以覆盖writeObject、readObject方法以实现序列化过程自定义。

## 六、APP启动过程

### 1. 流程概述

![img](http://upload-images.jianshu.io/upload_images/3985563-b7edc7b70c9c332f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**启动流程：**

①点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；

②system_server进程接收到请求后，向zygote进程发送创建进程的请求；

③Zygote进程fork出新的子进程，即App进程；

④App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；

⑤system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；

⑥App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；

⑦主线程在收到Message后，通过反射机制创建目标Activity，并回调Activity.onCreate()等方法。

⑧到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。

上面的一些列步骤简单介绍了一个APP启动到主页面显示的过程，可能这些流程中的一些术语看的有些懵，什么是Launcher，什么是zygote，什么是applicationThread.....

下面我们一一介绍。

### 2. 理论基础

#### 1) zygote

zygote意为“受精卵“。Android是基于Linux系统的，而在Linux中，所有的进程都是由init进程直接或者是间接fork出来的，zygote进程也不例外。

在Android系统里面，zygote是一个进程的名字。Android是基于Linux System的，当你的手机开机的时候，Linux的内核加载完成之后就会启动一个叫“init“的进程。在Linux System里面，所有的进程都是由init进程fork出来的，我们的zygote进程也不例外。

我们都知道，每一个App其实都是

● 一个单独的dalvik虚拟机

● 一个单独的进程

所以当系统里面的第一个zygote进程运行之后，在这之后再开启App，就相当于开启一个新的进程。而为了实现资源共用和更快的启动速度，Android系统开启新进程的方式，是通过fork第一个zygote进程实现的。所以说，除了第一个zygote进程，其他应用所在的进程都是zygote的子进程，这下你明白为什么这个进程叫“受精卵”了吧？因为就像是一个受精卵一样，它能快速的分裂，并且产生遗传物质一样的细胞！

#### 2) system_server

SystemServer也是一个进程，而且是由zygote进程fork出来的。

知道了SystemServer的本质，我们对它就不算太陌生了，这个进程是Android Framework里面两大非常重要的进程之一——另外一个进程就是上面的zygote进程。

为什么说SystemServer非常重要呢？因为系统里面重要的服务都是在这个进程里面开启的，比如
ActivityManagerService、PackageManagerService、WindowManagerService等等。

#### 3) ActivityManagerService

ActivityManagerService，简称AMS，服务端对象，负责系统中所有Activity的生命周期。

ActivityManagerService进行初始化的时机很明确，就是在SystemServer进程开启的时候，就会初始化ActivityManagerService。

**下面介绍下Android系统里面的服务器和客户端的概念。**

其实服务器客户端的概念不仅仅存在于Web开发中，在Android的框架设计中，使用的也是这一种模式。服务器端指的就是所有App共用的系统服务，比如我们这里提到的ActivityManagerService，和前面提到的PackageManagerService、WindowManagerService等等，这些基础的系统服务是被所有的App公用的，当某个App想实现某个操作的时候，要告诉这些系统服务，比如你想打开一个App，那么我们知道了包名和MainActivity类名之后就可以打开

```java
Intent intent = new Intent(Intent.ACTION_MAIN);  
intent.addCategory(Intent.CATEGORY_LAUNCHER);              
ComponentName cn = new ComponentName(packageName, className);              
intent.setComponent(cn);  
startActivity(intent);
```

但是，我们的App通过调用startActivity()并不能直接打开另外一个App，这个方法会通过一系列的调用，最后还是告诉AMS说：“我要打开这个App，我知道他的住址和名字，你帮我打开吧！”所以是AMS来通知zygote进程来fork一个新进程，来开启我们的目标App的。这就像是浏览器想要打开一个超链接一样，浏览器把网页地址发送给服务器，然后还是服务器把需要的资源文件发送给客户端的。

知道了Android Framework的客户端服务器架构之后，我们还需要了解一件事情，那就是我们的App和AMS(SystemServer进程)还有zygote进程分属于三个独立的进程，他们之间如何通信呢？

**App与AMS通过Binder进行IPC通信，AMS(SystemServer进程)与zygote通过Socket进行IPC通信。后面具体介绍。**

那么AMS有什么用呢？**在前面我们知道了，如果想打开一个App的话，需要AMS去通知zygote进程，除此之外，其实所有的Activity的开启、暂停、关闭都需要AMS来控制，所以我们说，AMS负责系统中所有Activity的生命周期。**

在Android系统中，**任何一个Activity的启动都是由AMS和应用程序进程（主要是ActivityThread）相互配合来完成的。AMS服务统一调度系统中所有进程的Activity启动，而每个Activity的启动过程则由其所属的进程具体来完成。**

#### 4) Launcher

当我们点击手机桌面上的图标的时候，App就由Launcher开始启动了。但是，你有没有思考过Launcher到底是一个什么东西？

**Launcher本质上也是一个应用程序，和我们的App一样，也是继承自Activity**

packages/apps/Launcher2/src/com/android/launcher2/Launcher.java

```java
public final class Launcher extends Activity
        implements View.OnClickListener, OnLongClickListener, LauncherModel.Callbacks,
                   View.OnTouchListener {
                   }
```

Launcher实现了点击、长按等回调接口，来接收用户的输入。既然是普通的App，那么我们的开发经验在这里就仍然适用，比如，我们点击图标的时候，是怎么开启的应用呢？**捕捉图标点击事件，然后startActivity()发送对应的Intent请求呗！是的，Launcher也是这么做的，就是这么easy！**

#### 5) Instrumentation和ActivityThread

每个Activity都持有Instrumentation对象的一个引用，但是整个进程只会存在一个Instrumentation对象。
Instrumentation这个类里面的方法大多数和Application和Activity有关，**这个类就是完成对Application和Activity初始化和生命周期的工具类。**Instrumentation这个类很重要，对Activity生命周期方法的调用根本就离不开他，他可以说是一个大管家。

ActivityThread，依赖于UI线程。App和AMS是通过Binder传递信息的，那么ActivityThread就是专门与AMS的外交工作的。

#### 6) ApplicationThread

前面我们已经知道了App的启动以及Activity的显示都需要AMS的控制，那么我们便需要和服务端的沟通，而这个沟通是双向的。

**客户端-->服务端**

![img](http://upload-images.jianshu.io/upload_images/3985563-f9b3071c35cdb5de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而且由于继承了同样的公共接口类，ActivityManagerProxy提供了与ActivityManagerService一样的函数原型，使用户感觉不出Server是运行在本地还是远端，从而可以更加方便的调用这些重要的系统服务。

**服务端-->客户端**

还是通过Binder通信，不过是换了另外一对，换成了ApplicationThread和ApplicationThreadProxy。

![img](http://upload-images.jianshu.io/upload_images/3985563-363b74b716570dd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

他们也都实现了相同的接口IApplicationThread

```java
  private class ApplicationThread extends ApplicationThreadNative {}

  public abstract class ApplicationThreadNative extends Binder implements IApplicationThread{}

  class ApplicationThreadProxy implements IApplicationThread {}
```

**好了，前面罗里吧嗦的一大堆，介绍了一堆名词，可能不太清楚，没关系，下面结合流程图介绍。**

### 3. 启动流程

#### 1.创建进程

①先从Launcher的startActivity()方法，通过Binder通信，调用ActivityManagerService的startActivity方法。

②一系列折腾，最后调用startProcessLocked()方法来创建新的进程。

③该方法会通过前面讲到的socket通道传递参数给Zygote进程。Zygote孵化自身。调用ZygoteInit.main()方法来实例化ActivityThread对象并最终返回新进程的pid。

④调用ActivityThread.main()方法，ActivityThread随后依次调用Looper.prepareLoop()和Looper.loop()来开启消息循环。

**方法调用流程图如下:**

![img](http://upload-images.jianshu.io/upload_images/3985563-25c23ee6ccb48048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**更直白的流程解释：**

![img](http://upload-images.jianshu.io/upload_images/3985563-ed91fd7c240e6bd3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

①App发起进程：当从桌面启动应用，则发起进程便是Launcher所在进程；当从某App内启动远程进程，则发送进程便是该App所在进程。发起进程先通过binder发送消息给system_server进程；

②system_server进程：调用Process.start()方法，通过socket向zygote进程发送创建新进程的请求；

③zygote进程：在执行ZygoteInit.main()后便进入runSelectLoop()循环体内，当有客户端连接时便会执行ZygoteConnection.runOnce()方法，再经过层层调用后fork出新的应用进程；

④新进程：执行handleChildProc方法，最后调用ActivityThread.main()方法。

#### 2.绑定Application

上面创建进程后，执行ActivityThread.main()方法，随后调用attach()方法。

将进程和指定的Application绑定起来。这个是通过上节的ActivityThread对象中调用bindApplication()方法完成的。该方法发送一个BIND_APPLICATION的消息到消息队列中, 最终通过handleBindApplication()方法处理该消息. 然后调用makeApplication()方法来加载App的classes到内存中。

**方法调用流程图如下：**

![img](http://upload-images.jianshu.io/upload_images/3985563-0eb6b9d2b091de3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**更直白的流程解释：**

![img](http://upload-images.jianshu.io/upload_images/3985563-d8def9358f4646e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 3.显示Activity界面

经过前两个步骤之后, 系统已经拥有了该application的进程。 后面的调用顺序就是普通的从一个已经存在的进程中启动一个新进程的activity了。

实际调用方法是realStartActivity(), 它会调用application线程对象中的scheduleLaunchActivity()发送一个LAUNCH_ACTIVITY消息到消息队列中, 通过 handleLaunchActivity()来处理该消息。在 handleLaunchActivity()通过performLaunchActiivty()方法回调Activity的onCreate()方法和onStart()方法，然后通过handleResumeActivity()方法，回调Activity的onResume()方法，最终显示Activity界面。

![img](http://upload-images.jianshu.io/upload_images/3985563-5222775558226c7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**更直白的流程解释：**

![img](http://upload-images.jianshu.io/upload_images/3985563-5f711b4bca6bf21b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4. Binder通信

![img](http://upload-images.jianshu.io/upload_images/3985563-cb3187996516846a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**ATP:** ApplicationThreadProxy

**AT:** ApplicationThread

**AMP:** ActivityManagerProxy

**AMS: **ActivityManagerService

①system_server进程中调用startProcessLocked方法,该方法最终通过socket方式,将需要创建新进程的消息告知Zygote进程,并阻塞等待Socket返回新创建进程的pid;

②Zygote进程接收到system_server发送过来的消息, 则通过fork的方法，将zygote自身进程复制生成新的进程，并将ActivityThread相关的资源加载到新进程app process,这个进程可能是用于承载activity等组件;

③ 在新进程app process向servicemanager查询system_server进程中binder服务端AMS, 获取相对应的Client端,也就是AMP. 有了这一对binder c/s对, 那么app process便可以通过binder向跨进程system_server发送请求,即attachApplication()

④system_server进程接收到相应binder操作后,经过多次调用,利用ATP向app process发送binder请求, 即bindApplication.
system_server拥有ATP/AMS, 每一个新创建的进程都会有一个相应的AT/AMP,从而可以跨进程 进行相互通信. 这便是进程创建过程的完整生态链。

以上大概介绍了一个APP从启动到主页面显示经历的流程，主要从宏观角度介绍了其过程，具体可结合源码理解。

## 七、Android性能优化总结

### 1. 布局优化

关于布局优化的思想很简单，就是**尽量减少布局文件的层级**。这个道理很浅显，布局中的层级少了，就意味着Android绘制时的工作量少了，那么程序的性能自然就提高了。

**①删除布局中无用的控件和层次，其次有选择地使用性能比较低的ViewGroup。**

关于有选择地使用性能比较低的ViewGroup,这就需要我们开发就实际灵活选择了。

例如：如果布局中既可以使用LinearLayout也可以使用RelativeLayout，那么就采用LinearLayout，这是因为RelativeLayout的功能比较复杂，它的布局过程需要花费更多的CPU时间。FrameLayout和LinearLayout一样都是一种简单高效的ViewGroup，因此可以考虑使用它们，但是**很多时候单纯通过一个LinearLayout或者FrameLayout无法实现产品效果，需要通过嵌套的方式来完成。这种情况下还是建议采用RelativeLayout,因为ViewGroup的嵌套就相当于增加了布局的层级，同样会降低程序的性能。**

**②采用<include>标签,<merge>标签,ViewStub。**

<include>标签主要用于布局重用。

<merge>标签一般和<include>配合使用，可以降低减少布局的层级。

ViewStub提供了按需加载的功能，当需要时才会将ViewStub中的布局加载到内存，提高了程序初始化效率。

**③避免多度绘制**

过度绘制（Overdraw）描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次重叠的 UI 结构里面，如果不可见的 UI 也在做绘制的操作，会导致某些像素区域被绘制了多次，同时也会浪费大量的 CPU 以及 GPU 资源。

如下所示，有些部分在布局时，会被重复绘制。

![img](http://upload-images.jianshu.io/upload_images/3985563-7f91a67f91bf3317.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关于过度绘制产生的一般场景及解决方案，参考：[Android 过度绘制优化](http://jaeger.itscoder.com/android/2016/09/29/android-performance-overdraw.html)

### 2. 绘制优化

绘制优化是指**View的onDraw方法要避免执行大量的操作，**这主要体现在两个方面：

**①onDraw中不要创建新的局部对象。**

因为onDraw方法可能会被频繁调用，这样就会在一瞬间产生大量的临时对象，这不仅占用了过多的内存而且还会导致系统更加频繁gc，降低了程序的执行效率。

**②onDraw方法中不要做耗时的任务，也不能执行成千上万次的循环操作，尽管每次循环都很轻量级，但是大量的循环仍然十分抢占CPU的时间片，这会造成View的绘制过程不流畅。**

按照Google官方给出的性能优化典范中的标准，View的绘制频率保证60fps是最佳的，这就要求每帧绘制时间不超过16ms(16ms = 1000/60)，虽然程序很难保证16ms这个时间，但是尽量降低onDraw方法中的复杂度总是切实有效的。

![img](http://upload-images.jianshu.io/upload_images/3985563-81b81fb8ab4d92db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3. 内存泄漏优化

内存泄漏是开发过程中的一个需要重视的问题，但是由于内存泄露问题对开发人员的经验和开发意识有较高的要求，因此也是开发人员最容易犯的错误之一。

内存泄露的优化分为两个方面：

**①在开发过程中避免写出有内存泄漏的代码**

**②通过一些分析工具比如MAT来找出潜在的内存泄露，然后解决。**

对应于两种不同情况，一个是了解内存泄漏的可能场景以及如何规避，二是怎么查找内存泄漏。

**1) 那么我们就先了解什么是内存泄漏?这样我们才能知道如何避免。**

大家都知道，java是有垃圾回收机制的，这使得java程序员比C++程序员轻松了许多，存储申请了，不用心心念念要加一句释放，java虚拟机会派出一些回收线程兢兢业业不定时地回收那些不再被需要的内存空间（注意回收的不是对象本身，而是对象占据的内存空间）。

**Q1：什么叫不再被需要的内存空间？**

**答：**Java没有指针，全凭引用来和对象进行关联，通过引用来操作对象。如果一个对象没有与任何引用关联，那么这个对象也就不太可能被使用到了，回收器便是把这些“无任何引用的对象”作为目标，回收了它们占据的内存空间。

**Q2：如何分辨为对象无引用？**

**答：**2种方法

**引用计数法**直接计数，简单高效，Python便是采用该方法。但是如果出现 两个对象相互引用，即使它们都无法被外界访问到，计数器不为0它们也始终不会被回收。为了解决该问题，java采用的是b方法。

**可达性分析法**这个方法设置了一系列的“GC Roots”对象作为索引起点，如果一个对象 与起点对象之间均无可达路径，那么这个不可达的对象就会成为回收对象。这种方法处理 两个对象相互引用的问题，如果两个对象均没有外部引用，会被判断为不可达对象进而被回收。

**Q3：有了回收机制，放心大胆用不会有内存泄漏？**

**答：**答案当然是No！

虽然垃圾回收器会帮我们干掉大部分无用的内存空间，但是对于还保持着引用，但逻辑上已经不会再用到的对象，垃圾回收器不会回收它们。这些对象积累在内存中，直到程序结束，就是我们所说的“内存泄漏”。
当然了，用户对单次的内存泄漏并没有什么感知，但当泄漏积累到内存都被消耗完，就会导致卡顿，崩溃。

下面这张图可以帮助我们更好地理解对象的状态，以及内存泄漏的情况

![img](http://upload-images.jianshu.io/upload_images/3985563-2cb740a394402ae0.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

左边未引用的对象是会被GC回收的，右边被引用的对象不会被GC回收，但是未使用的对象中除了未引用的对象，还包括已被引用的一部分对象，那么内存泄漏久发生这部分已被引用但未使用的对象。

**2) Android一般在什么情况下会出现内存泄漏？**

①集合类泄漏
②单例/静态变量造成的内存泄漏
③匿名内部类/非静态内部类
④资源未关闭造成的内存泄漏

大概可以分为以上几类，还有一些经常会听到的Hanlder,AsyncTask引起内存泄漏，都属于上述③中的情况。

那么上述四种情况是怎么造成的内存泄漏，具体是什么原因，以及Android中一些知名的引起内存泄漏的原因，以及解决方法是怎么样的？

**3) Android怎么分析内存泄漏？**

上面介绍了内存泄漏的场景，对应的有一些解决方案。

那么在内存泄漏已经发生的情况下，我们该如何解决呢？

我们可以通过MAT(Memory Analyzer Tool)，或者 LeakCanary来检测Android中的内存泄漏。

### 4. 响应速度优化

响应速度优化的核心思想就是**避免在主线程中做耗时操作**。

如果有耗时操作，可以开启子线程执行，即采用异步的方式来执行耗时操作。

如果在主线程中做太多事情，会导致Activity启动时出现黑屏现象，甚至ANR。

![img](http://upload-images.jianshu.io/upload_images/3985563-a1e005753b8e32a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Android规定，Activity如果5秒钟之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，而BroadcastReceiver如果10秒钟之内还未执行完操作也会出现ANR。**

为了避免ANR，可以开启子线程执行耗时操作，但是子线程不能更新UI，所以需要子线程与主线程进行通信来解决子线程执行耗时任务后，通知主线程更新UI的场景。关于这部分，需要掌握Handler消息机制，AsyncTask，IntentService等内容。

然而，在实际开发中，ANR仍然不可避免的发生了，而且很难从代码上发现，这时候就要用到ANR日志分析。当一个进程发生了ANR之后，系统会在/data/anr目录下创建一个文件traces.txt，通过分析这个文件就能定位出ANR的原因。

### 5. ListView/RecycleView及Bitmap优化

![img](http://upload-images.jianshu.io/upload_images/3985563-d51c4fec20e776cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**ListView/RecycleView的优化思想主要从以下几个方面入手：**

①使用ViewHolder模式来提高效率

②异步加载：耗时的操作放在异步线程中

③ListView/RecycleView的滑动时停止加载和分页加载

具体优化建议及详情，参考：[ListView的优化](http://www.jianshu.com/p/f0408a0f0610)

**Bitmap优化**

![img](http://upload-images.jianshu.io/upload_images/3985563-eab380aea4795930.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

主要是对加载图片进行压缩，避免加载图片多大导致OOM出现。

### 6. 线程优化

线程优化的思想就是**采用线程池，避免程序中存在大量的Thread。**线程池可以重用内部的线程，从而避免了线程的创建和销毁锁带来的性能开销，同时线程池还能有效地控制线程池的最大并法术，避免大量的线程因互相抢占系统资源从而导致阻塞现象的发生。因此在实际开发中，尽量采用线程池，而不是每次都要创建一个Thread对象。

![img](http://upload-images.jianshu.io/upload_images/3985563-7dda79e4c0ec6d78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7. 其他性能优化建议

①避免过度的创建对象

②不要过度使用枚举，枚举占用的内存空间要比整型大

③常量请使用static final来修饰

④使用一些Android特有的数据结构，比如SparseArray和Pair等

⑤适当采用软引用和弱引用

⑥采用内存缓存和磁盘缓存

⑦尽量采用静态内部类，这样可以避免潜在的由于内部类而导致的内存泄漏。



## 八、Android 内存泄漏总结

内存管理的目的就是让我们在开发中怎么有效的避免我们的应用出现内存泄漏的问题。内存泄漏大家都不陌生了，简单粗俗的讲，就是该被释放的对象没有释放，一直被某个或某些实例所持有却不再被使用导致 GC 不能回收

### 1. Java 内存分配策略

Java 程序运行时的内存分配策略有三种,分别是静态分配,栈式分配,和堆式分配，对应的，三种存储策略使用的内存空间主要分别是静态存储区（也称方法区）、栈区和堆区。

- 静态存储区（方法区）：主要存放静态数据、全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
- 栈区 ：当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
- 堆区 ： 又称动态内存分配，通常就是指在程序运行时直接 new 出来的内存。这部分内存在不使用时将会由 Java 垃圾回收器来负责回收。

**栈与堆的区别：**

在方法体内定义的（局部变量）一些基本类型的变量和对象的引用变量都是在方法的栈内存中分配的。当在一段方法块中定义一个变量时，Java 就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给它的内存空间也将被释放掉，该内存空间可以被重新使用。

堆内存用来存放所有由 new 创建的对象（包括该对象其中的所有成员变量）和数组。在堆中分配的内存，将由 Java 垃圾回收器来自动管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，这个特殊的变量就是我们上面说的引用变量。我们可以通过这个引用变量来访问堆中的对象或者数组。

举个例子:

```java
public class Sample() {
    int s1 = 0;
    Sample mSample1 = new Sample();

    public void method() {
        int s2 = 1;
        Sample mSample2 = new Sample();
    }
}

Sample mSample3 = new Sample();

```

Sample 类的局部变量 s2 和引用变量 mSample2 都是存在于栈中，但 mSample2 指向的对象是存在于堆上的。
mSample3 指向的对象实体存放在堆上，包括这个对象的所有成员变量 s1 和 mSample1，而它自己存在于栈中。

结论：

局部变量的基本数据类型和引用存储于栈中，引用的对象实体存储于堆中。—— 因为它们属于方法中的变量，生命周期随方法而结束。

成员变量全部存储于堆中（包括基本数据类型，引用和引用的对象实体）—— 因为它们属于类，类对象终究是要被new出来使用的。

了解了 Java 的内存分配之后，我们再来看看 Java 是怎么管理内存的。

### 2. Java是如何管理内存

Java的内存管理就是对象的分配和释放问题。在 Java 中，程序员需要通过关键字 new 为每个对象申请内存空间 (基本类型除外)，所有的对象都在堆 (Heap)中分配空间。另外，对象的释放是由 GC 决定和执行的。在 Java 中，内存的分配是由程序完成的，而内存的释放是由 GC 完成的，这种收支两条线的方法确实简化了程序员的工作。但同时，它也加重了JVM的工作。这也是 Java 程序运行速度较慢的原因之一。因为，GC 为了能够正确释放对象，GC 必须监控每一个对象的运行状态，包括对象的申请、引用、被引用、赋值等，GC 都需要进行监控。

监视对象状态是为了更加准确地、及时地释放对象，而释放对象的根本原则就是该对象不再被引用。

为了更好理解 GC 的工作原理，我们可以将对象考虑为有向图的顶点，将引用关系考虑为图的有向边，有向边从引用者指向被引对象。另外，每个线程对象可以作为一个图的起始顶点，例如大多程序从 main 进程开始执行，那么该图就是以 main 进程顶点开始的一棵根树。在这个有向图中，根顶点可达的对象都是有效对象，GC将不回收这些对象。如果某个对象 (连通子图)与这个根顶点不可达(注意，该图为有向图)，那么我们认为这个(这些)对象不再被引用，可以被 GC 回收。
以下，我们举一个例子说明如何用有向图表示内存管理。对于程序的每一个时刻，我们都有一个有向图表示JVM的内存分配情况。以下右图，就是左边程序运行到第6行的示意图。

[![img](http://www.ibm.com/developerworks/cn/java/l-JavaMemoryLeak/1.gif)](javascript:;)

Java使用有向图的方式进行内存管理，可以消除引用循环的问题，例如有三个对象，相互引用，只要它们和根进程不可达的，那么GC也是可以回收它们的。这种方式的优点是管理内存的精度很高，但是效率较低。另外一种常用的内存管理技术是使用计数器，例如COM模型采用计数器方式管理构件，它与有向图相比，精度行低(很难处理循环引用的问题)，但执行效率很高。

### 3. 什么是Java中的内存泄露

在Java中，内存泄漏就是存在一些被分配的对象，这些对象有下面两个特点，首先，这些对象是可达的，即在有向图中，存在通路可以与其相连；其次，这些对象是无用的，即程序以后不会再使用这些对象。如果对象满足这两个条件，这些对象就可以判定为Java中的内存泄漏，这些对象不会被GC所回收，然而它却占用内存。

在C++中，内存泄漏的范围更大一些。有些对象被分配了内存空间，然后却不可达，由于C++中没有GC，这些内存将永远收不回来。在Java中，这些不可达的对象都由GC负责回收，因此程序员不需要考虑这部分的内存泄露。

通过分析，我们得知，对于C++，程序员需要自己管理边和顶点，而对于Java程序员只需要管理边就可以了(不需要管理顶点的释放)。通过这种方式，Java提高了编程的效率。

[![img](http://www.ibm.com/developerworks/cn/java/l-JavaMemoryLeak/2.gif)](javascript:;)

因此，通过以上分析，我们知道在Java中也有内存泄漏，但范围比C++要小一些。因为Java从语言上保证，任何对象都是可达的，所有的不可达对象都由GC管理。

对于程序员来说，GC基本是透明的，不可见的。虽然，我们只有几个函数可以访问GC，例如运行GC的函数System.gc()，但是根据Java语言规范定义， 该函数不保证JVM的垃圾收集器一定会执行。因为，不同的JVM实现者可能使用不同的算法管理GC。通常，GC的线程的优先级别较低。JVM调用GC的策略也有很多种，有的是内存使用到达一定程度时，GC才开始工作，也有定时执行的，有的是平缓执行GC，有的是中断式执行GC。但通常来说，我们不需要关心这些。除非在一些特定的场合，GC的执行影响应用程序的性能，例如对于基于Web的实时系统，如网络游戏等，用户不希望GC突然中断应用程序执行而进行垃圾回收，那么我们需要调整GC的参数，让GC能够通过平缓的方式释放内存，例如将垃圾回收分解为一系列的小步骤执行，Sun提供的HotSpot JVM就支持这一特性。

同样给出一个 Java 内存泄漏的典型例子，

```Java
Vector v = new Vector(10);
for (int i = 1; i < 100; i++) {
    Object o = new Object();
    v.add(o);
    o = null;   
}
```

在这个例子中，我们循环申请Object对象，并将所申请的对象放入一个 Vector 中，如果我们仅仅释放引用本身，那么 Vector 仍然引用该对象，所以这个对象对 GC 来说是不可回收的。因此，如果对象加入到Vector 后，还必须从 Vector 中删除，最简单的方法就是将 Vector 对象设置为 null。

### 4. Android中常见的内存泄漏汇总

- **集合类泄漏**

  集合类如果仅仅有添加元素的方法，而没有相应的删除机制，导致内存被占用。如果这个集合类是全局性的变量 (比如类中的静态属性，全局性的 map 等即有静态引用或 final 一直指向它)，那么没有相应的删除机制，很可能导致集合所占用的内存只增不减。比如上面的典型例子就是其中一种情况，稍不注意还是很容易出现这种情况，比如我们都喜欢通过 HashMap 做一些缓存之类的事，这种情况就要多留一些心眼。

- **单例造成的内存泄漏**

  由于单例的静态特性使得其生命周期跟应用的生命周期一样长，所以如果使用不恰当的话，很容易造成内存泄漏。比如下面一个典型的例子，

```java
public class AppManager {
  private static AppManager instance;
  private Context context;
  private AppManager(Context context) {
  this.context = context;
  }
  public static AppManager getInstance(Context context) {
    if (instance == null) {
    instance = new AppManager(context);
    }
    return instance;
  }
}

```

这是一个普通的单例模式，当创建这个单例的时候，由于需要传入一个Context，所以这个Context的生命周期的长短至关重要：

1、如果此时传入的是 Application 的 Context，因为 Application 的生命周期就是整个应用的生命周期，所以这将没有任何问题。

2、如果此时传入的是 Activity 的 Context，当这个 Context 所对应的 Activity 退出时，由于该 Context 的引用被单例对象所持有，其生命周期等于整个应用程序的生命周期，所以当前 Activity 退出时它的内存并不会被回收，这就造成泄漏了。

正确的方式应该改为下面这种方式：

```java
public class AppManager {
  private static AppManager instance;
  private Context context;
  private AppManager(Context context) {
    this.context = context.getApplicationContext();// 使用Application 的context
 }
  public static AppManager getInstance(Context context) {
    if (instance == null) {
      instance = new AppManager(context);
    }
    return instance;
  }
}

```

或者这样写，连 Context 都不用传进来了：

```java
在你的 Application 中添加一个静态方法，getContext() 返回 Application 的 context，

...

context = getApplicationContext();

...  
   /**
     * 获取全局的context
     * @return 返回全局context对象
     */
    public static Context getContext(){
        return context;
    }

public class AppManager {
  private static AppManager instance;
  private Context context;
  private AppManager() {
    this.context = MyApplication.getContext();// 使用Application 的context
  }
  public static AppManager getInstance() {
    if (instance == null) {
      instance = new AppManager();
    }
    return instance;
  }
}

```

- **匿名内部类/非静态内部类和异步线程**

  非静态内部类创建静态实例造成的内存泄漏

  有的时候我们可能会在启动频繁的Activity中，为了避免重复创建相同的数据资源，可能会出现这种写法：

  ```java
  public class MainActivity extends AppCompatActivity {
    private static TestResource mResource = null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      if(mManager == null){
        mManager = new TestResource();
      }
      //...
      }
    class TestResource {
    //...
    }
  }
  
  ```

这样就在Activity内部创建了一个非静态内部类的单例，每次启动Activity时都会使用该单例的数据，这样虽然避免了资源的重复创建，不过这种写法却会造成内存泄漏，因为非静态内部类默认会持有外部类的引用，而该非静态内部类又创建了一个静态的实例，该实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，导致Activity的内存资源不能正常回收。正确的做法为：

将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，请按照上面推荐的使用Application 的 Context。当然，Application 的 context 不是万能的，所以也不能随便乱用，对于有些地方则必须使用 Activity 的 Context，对于Application，Service，Activity三者的Context的应用场景如下：

[![img](http://img.blog.csdn.net/20151123144226349)](javascript:;)

**其中：** NO1表示 Application 和 Service 可以启动一个 Activity，不过需要创建一个新的 task 任务队列。而对于 Dialog 而言，只有在 Activity 中才能创建

- 匿名内部类

  android开发经常会继承实现Activity/Fragment/View，此时如果你使用了匿名类，并被异步线程持有了，那要小心了，如果没有任何措施这样一定会导致泄露

  ```java
  public class MainActivity extends Activity {
    ...
    Runnable ref1 = new MyRunable();
    Runnable ref2 = new Runnable() {
        @Override
        public void run() {
  
        }
    };
     ...
  }
  
  ```

ref1和ref2的区别是，ref2使用了匿名内部类。我们来看看运行时这两个引用的内存：

[![img](http://img2.tbcdn.cn/L1/461/1/fb05ff6d2e68f309b94dd84352c81acfe0ae839e)](javascript:;)

可以看到，ref1没什么特别的。
但ref2这个匿名类的实现对象里面多了一个引用：
this$0这个引用指向MainActivity.this，也就是说当前的MainActivity实例会被ref2持有，如果将这个引用再传入一个异步线程，此线程和此Acitivity生命周期不一致的时候，就造成了Activity的泄露。

- **Handler 造成的内存泄漏**

  Handler 的使用造成的内存泄漏问题应该说是最为常见了，很多时候我们为了避免 ANR 而不在主线程进行耗时操作，在处理网络任务或者封装一些请求回调等api都借助Handler来处理，但 Handler 不是万能的，对于 Handler 的使用代码编写一不规范即有可能造成内存泄漏。另外，我们知道 Handler、Message 和 MessageQueue 都是相互关联在一起的，万一 Handler 发送的 Message 尚未被处理，则该 Message 及发送它的 Handler 对象将被线程 MessageQueue 一直持有。
  由于 Handler 属于 TLS(Thread Local Storage) 变量, 生命周期和 Activity 是不一致的。因此这种实现方式一般很难保证跟 View 或者 Activity 的生命周期保持一致，故很容易导致无法正确释放。

  举个例子：

  ```java
  public class SampleActivity extends Activity {
  
    private final Handler mLeakyHandler = new Handler() {
      @Override
      public void handleMessage(Message msg) {
        // ...
      }
    }
  
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
  
      // Post a message and delay its execution for 10 minutes.
      mLeakyHandler.postDelayed(new Runnable() {
        @Override
        public void run() { /* ... */ }
      }, 1000 * 60 * 10);
  
      // Go back to the previous Activity.
      finish();
    }
  }
  
  ```

在该 SampleActivity 中声明了一个延迟10分钟执行的消息 Message，mLeakyHandler 将其 push 进了消息队列 MessageQueue 里。当该 Activity 被 finish() 掉时，延迟执行任务的 Message 还会继续存在于主线程中，它持有该 Activity 的 Handler 引用，所以此时 finish() 掉的 Activity 就不会被回收了从而造成内存泄漏（因 Handler 为非静态内部类，它会持有外部类的引用，在这里就是指 SampleActivity）。

修复方法：在 Activity 中避免使用非静态内部类，比如上面我们将 Handler 声明为静态的，则其存活期跟 Activity 的生命周期就无关了。同时通过弱引用的方式引入 Activity，避免直接将 Activity 作为 context 传进去，见下面代码：

```java
public class SampleActivity extends Activity {

  /**
   * Instances of static inner classes do not hold an implicit
   * reference to their outer class.
   */
  private static class MyHandler extends Handler {
    private final WeakReference<SampleActivity> mActivity;

    public MyHandler(SampleActivity activity) {
      mActivity = new WeakReference<SampleActivity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
      SampleActivity activity = mActivity.get();
      if (activity != null) {
        // ...
      }
    }
  }

  private final MyHandler mHandler = new MyHandler(this);

  /**
   * Instances of anonymous classes do not hold an implicit
   * reference to their outer class when they are "static".
   */
  private static final Runnable sRunnable = new Runnable() {
      @Override
      public void run() { /* ... */ }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Post a message and delay its execution for 10 minutes.
    mHandler.postDelayed(sRunnable, 1000 * 60 * 10);

    // Go back to the previous Activity.
    finish();
  }
}

```

综述，即推荐使用静态内部类 + WeakReference 这种方式。每次使用前注意判空。

前面提到了 WeakReference，所以这里就简单的说一下 Java 对象的几种引用类型。

Java对引用的分类有 Strong reference, SoftReference, WeakReference, PhatomReference 四种。

[![img](https://gw.alicdn.com/tps/TB1U6TNLVXXXXchXFXXXXXXXXXX-644-546.jpg)](javascript:;)

在Android应用的开发中，为了防止内存溢出，在处理一些占用内存大而且声明周期较长的对象时候，可以尽量应用软引用和弱引用技术。

软/弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。利用这个队列可以得知被回收的软/弱引用的对象列表，从而为缓冲器清除已失效的软/弱引用。

假设我们的应用会用到大量的默认图片，比如应用中有默认的头像，默认游戏图标等等，这些图片很多地方会用到。如果每次都去读取图片，由于读取文件需要硬件操作，速度较慢，会导致性能较低。所以我们考虑将图片缓存起来，需要的时候直接从内存中读取。但是，由于图片占用内存空间比较大，缓存很多图片需要很多的内存，就可能比较容易发生OutOfMemory异常。这时，我们可以考虑使用软/弱引用技术来避免这个问题发生。以下就是高速缓冲器的雏形：

首先定义一个HashMap，保存软引用对象。

```java
private Map <String, SoftReference<Bitmap>> imageCache = new HashMap <String, SoftReference<Bitmap>> ();
```

再来定义一个方法，保存Bitmap的软引用到HashMap。

[![img](https://gw.alicdn.com/tps/TB1oW_FLVXXXXXuaXXXXXXXXXXX-679-717.jpg)](javascript:;)

使用软引用以后，在OutOfMemory异常发生之前，这些缓存的图片资源的内存空间可以被释放掉的，从而避免内存达到上限，避免Crash发生。

如果只是想避免OutOfMemory异常的发生，则可以使用软引用。如果对于应用的性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用。

另外可以根据对象是否经常使用来判断选择软引用还是弱引用。如果该对象可能会经常使用的，就尽量用软引用。如果该对象不被使用的可能性更大些，就可以用弱引用。

前面所说的，创建一个静态Handler内部类，然后对 Handler 持有的对象使用弱引用，这样在回收时也可以回收 Handler 持有的对象，但是这样做虽然避免了 Activity 泄漏，不过 Looper 线程的消息队列中还是可能会有待处理的消息，所以我们在 Activity 的 Destroy 时或者 Stop 时应该**移除消息队列 MessageQueue 中的消息**。

下面几个方法都可以移除 Message：

```java
public final void removeCallbacks(Runnable r);

public final void removeCallbacks(Runnable r, Object token);

public final void removeCallbacksAndMessages(Object token);

public final void removeMessages(int what);

public final void removeMessages(int what, Object object);

```

- **尽量避免使用 static 成员变量**

  如果成员变量被声明为 static，那我们都知道其生命周期将与整个app进程生命周期一样。

  这会导致一系列问题，如果你的app进程设计上是长驻内存的，那即使app切到后台，这部分内存也不会被释放。按照现在手机app内存管理机制，占内存较大的后台进程将优先回收，因为如果此app做过进程互保保活，那会造成app在后台频繁重启。当手机安装了你参与开发的app以后一夜时间手机被消耗空了电量、流量，你的app不得不被用户卸载或者静默。
  这里修复的方法是：

不要在类初始时初始化静态成员。可以考虑lazy初始化。
架构设计上要思考是否真的有必要这样做，尽量避免。如果架构需要这么设计，那么此对象的生命周期你有责任管理起来。

- **避免 override finalize()**

  1、finalize 方法被执行的时间不确定，不能依赖与它来释放紧缺的资源。时间不确定的原因是：

  - 虚拟机调用GC的时间不确定
  - Finalize daemon线程被调度到的时间不确定

  2、finalize 方法只会被执行一次，即使对象被复活，如果已经执行过了 finalize 方法，再次被 GC 时也不会再执行了，原因是：

  含有 finalize 方法的 object 是在 new 的时候由虚拟机生成了一个 finalize reference 在来引用到该Object的，而在 finalize 方法执行的时候，该 object 所对应的 finalize Reference 会被释放掉，即使在这个时候把该 object 复活(即用强引用引用住该 object )，再第二次被 GC 的时候由于没有了 finalize reference 与之对应，所以 finalize 方法不会再执行。

  3、含有Finalize方法的object需要至少经过两轮GC才有可能被释放。

  详情见这里 [深入分析过dalvik的代码](http://blog.csdn.net/kai_gong/article/details/24188803)

- **资源未关闭造成的内存泄漏**

  对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄漏。

- **一些不良代码造成的内存压力**

  有些代码并不造成内存泄露，但是它们，或是对没使用的内存没进行有效及时的释放，或是没有有效的利用已有的对象而是频繁的申请新内存。

  比如：构造 Adapter 时，没有使用缓存的 convertView ,每次都在创建新的 converView。这里推荐使用 ViewHolder。


### 5. 工具分析

Java 内存泄漏的分析工具有很多，但众所周知的要数 MAT(Memory Analysis Tools) 和 YourKit 了。由于篇幅问题，我这里就只对 [MAT](http://www.eclipse.org/mat/) 的使用做一下介绍。--> [MAT 的安装](http://www.ibm.com/developerworks/cn/opensource/os-cn-ecl-ma/index.html)

- MAT分析heap的总内存占用大小来初步判断是否存在泄露

  打开 DDMS 工具，在左边 Devices 视图页面选中“Update Heap”图标，然后在右边切换到 Heap 视图，点击 Heap 视图中的“Cause GC”按钮，到此为止需检测的进程就可以被监视。

  [![img](https://gw.alicdn.com/tps/TB1pdf2LVXXXXXeXXXXXXXXXXXX-690-514.jpg)](javascript:;)

  Heap视图中部有一个Type叫做data object，即数据对象，也就是我们的程序中大量存在的类类型的对象。在data object一行中有一列是“Total Size”，其值就是当前进程中所有Java数据对象的内存总量，一般情况下，这个值的大小决定了是否会有内存泄漏。可以这样判断：

  进入某应用，不断的操作该应用，同时注意观察data object的Total Size值，正常情况下Total Size值都会稳定在一个有限的范围内，也就是说由于程序中的的代码良好，没有造成对象不被垃圾回收的情况。

  所以说虽然我们不断的操作会不断的生成很多对象，而在虚拟机不断的进行GC的过程中，这些对象都被回收了，内存占用量会会落到一个稳定的水平；反之如果代码中存在没有释放对象引用的情况，则data object的Total Size值在每次GC后不会有明显的回落。随着操作次数的增多Total Size的值会越来越大，直到到达一个上限后导致进程被杀掉。

- MAT分析hprof来定位内存泄露的原因所在

  这是出现内存泄露后使用MAT进行问题定位的有效手段。

  A)Dump出内存泄露当时的内存镜像hprof，分析怀疑泄露的类：

  [![img](https://gw.alicdn.com/tps/TB1r2zZLVXXXXcHXXXXXXXXXXXX-640-167.jpg)](javascript:;)

  B)分析持有此类对象引用的外部对象

  [![img](https://gw.alicdn.com/tps/TB17XvOLVXXXXbiXFXXXXXXXXXX-640-90.png)](javascript:;)

  C)分析这些持有引用的对象的GC路径

  [![img](https://gw.alicdn.com/tps/TB10yTwLVXXXXaRapXXXXXXXXXX-640-278.png)](javascript:;)

  D)逐个分析每个对象的GC路径是否正常

  [![img](https://gw.alicdn.com/tps/TB1CWTQLVXXXXamXFXXXXXXXXXX-640-90.png)](javascript:;)

  从这个路径可以看出是一个antiRadiationUtil工具类对象持有了MainActivity的引用导致MainActivity无法释放。此时就要进入代码分析此时antiRadiationUtil的引用持有是否合理（如果antiRadiationUtil持有了MainActivity的context导致节目退出后MainActivity无法销毁，那一般都属于内存泄露了）。

- MAT对比操作前后的hprof来定位内存泄露的根因所在

  为查找内存泄漏，通常需要两个 Dump结果作对比，打开 Navigator History面板，将两个表的 Histogram结果都添加到 Compare Basket中去

  A） 第一个HPROF 文件(usingFile > Open Heap Dump ).

  B）打开Histogram view.

  C）在NavigationHistory view里 (如果看不到就从Window >show view>MAT- Navigation History ), 右击histogram然后选择Add to Compare Basket .

  [![img](https://gw.alicdn.com/tps/TB1p1rULVXXXXbyXpXXXXXXXXXX-525-212.png)](javascript:;)

  D）打开第二个HPROF 文件然后重做步骤2和3.

  E）切换到Compare Basket view, 然后点击Compare the Results (视图右上角的红色”!”图标)。

  [![img](https://gw.alicdn.com/tps/TB1p0zKLVXXXXX.XVXXXXXXXXXX-640-98.png)](javascript:;)

  F）分析对比结果

  [![img](https://gw.alicdn.com/tps/TB1lwDMLVXXXXcUXFXXXXXXXXXX-640-115.png)](javascript:;)

  可以看出两个hprof的数据对象对比结果。

  通过这种方式可以快速定位到操作前后所持有的对象增量，从而进一步定位出当前操作导致内存泄露的具体原因是泄露了什么数据对象。

  注意：

  如果是用 MAT Eclipse 插件获取的 Dump文件，不需要经过转换则可在MAT中打开，Adt会自动进行转换。

  而手机SDk Dump 出的文件要经过转换才能被 MAT识别，Android SDK提供了这个工具 hprof-conv (位于 sdk/tools下)

  首先，要通过控制台进入到你的 android sdk tools 目录下执行以下命令：

  ./hprof-conv xxx-a.hprof xxx-b.hprof

  例如 hprof-conv input.hprof out.hprof

  此时才能将out.hprof放在eclipse的MAT中打开。

下面将给介绍一个工具 -- LeakCanary 。

### 6. 使用 LeakCanary 检测 Android 的内存泄漏

 [LeakCanary](https://github.com/square/leakcanary) 是国外一位大神 Pierre-Yves Ricau 开发的一个用于检测内存泄露的开源类库。一般情况下，在对战内存泄露中，我们都会经过以下几个关键步骤：

1、了解 OutOfMemoryError 情况。

2、重现问题。

3、在发生内存泄露的时候，把内存 Dump 出来。

4、在发生内存泄露的时候，把内存 Dump 出来。

5、计算这个对象到 GC roots 的最短强引用路径。

6、确定引用路径中的哪个引用是不该有的，然后修复问题。

如果有一个类库能在发生 OOM 之前把这些事情全部都搞定，然后你只要修复这些问题就好了。LeakCanary 做的就是这件事情。你可以在 debug 包中轻松检测内存泄露。

一起来看这个例子（摘自 LeakCanary 中文使用说明，下面会附上所有的参考文档链接）：

```java
class Cat {
}

class Box {
  Cat hiddenCat;
}
class Docker {
    // 静态变量，将不会被回收，除非加载 Docker 类的 ClassLoader 被回收。
    static Box container;
}

// ...

Box box = new Box();

// 薛定谔之猫
Cat schrodingerCat = new Cat();
box.hiddenCat = schrodingerCat;
Docker.container = box;

```

创建一个RefWatcher，监控对象引用情况。

```java
// 我们期待薛定谔之猫很快就会消失（或者不消失），我们监控一下
refWatcher.watch(schrodingerCat);
```

当发现有内存泄露的时候，你会看到一个很漂亮的 leak trace 报告:

- GC ROOT static Docker.container
- references Box.hiddenCat
- leaks Cat instance

只需要添加一行代码就行了。然后 LeakCanary 就会自动侦测 activity 的内存泄露了。

```java
public class ExampleApplication extends Application {
  @Override public void onCreate() {
    super.onCreate();
    LeakCanary.install(this);
  }
}

```

然后你会在通知栏看到一个界面，以很直白的方式将内存泄露展现在我们的面前。

**Demo**

一个非常简单的 LeakCanary demo: [一个非常简单的 LeakCanary demo: https://github.com/liaohuqiu/leakcanary-demo](https://github.com/liaohuqiu/leakcanary-demo)

**接入**

在 build.gradle 中加入引用，不同的编译使用不同的引用：

```java
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3'
 }

```

**如何使用**

使用 RefWatcher 监控那些本该被回收的对象。

```java
RefWatcher refWatcher = {...};

// 监控
refWatcher.watch(schrodingerCat);

```

LeakCanary.install() 会返回一个预定义的 RefWatcher，同时也会启用一个 ActivityRefWatcher，用于自动监控调用 Activity.onDestroy() 之后泄露的 activity。

在Application中进行配置 ：

```java
public class ExampleApplication extends Application {

  public static RefWatcher getRefWatcher(Context context) {
    ExampleApplication application = (ExampleApplication) context.getApplicationContext();
    return application.refWatcher;
  }

  private RefWatcher refWatcher;

  @Override public void onCreate() {
    super.onCreate();
    refWatcher = LeakCanary.install(this);
  }
}

```

使用 RefWatcher 监控 Fragment：

```java
public abstract class BaseFragment extends Fragment {

  @Override public void onDestroy() {
    super.onDestroy();
    RefWatcher refWatcher = ExampleApplication.getRefWatcher(getActivity());
    refWatcher.watch(this);
  }
}

```

使用 RefWatcher 监控 Activity：

```java
public class MainActivity extends AppCompatActivity {

    ......
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
            //在自己的应用初始Activity中加入如下两行代码
        RefWatcher refWatcher = ExampleApplication.getRefWatcher(this);
        refWatcher.watch(this);

        textView = (TextView) findViewById(R.id.tv);
        textView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startAsyncTask();
            }
        });

    }

    private void async() {

        startAsyncTask();
    }

    private void startAsyncTask() {
        // This async task is an anonymous class and therefore has a hidden reference to the outer
        // class MainActivity. If the activity gets destroyed before the task finishes (e.g. rotation),
        // the activity instance will leak.
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                // Do some slow work in background
                SystemClock.sleep(20000);
                return null;
            }
        }.execute();
    }


}

```

**工作机制**

1.RefWatcher.watch() 创建一个 KeyedWeakReference 到要被监控的对象。

2.然后在后台线程检查引用是否被清除，如果没有，调用GC。

3.如果引用还是未被清除，把 heap 内存 dump 到 APP 对应的文件系统中的一个 .hprof 文件中。

4.在另外一个进程中的 HeapAnalyzerService 有一个 HeapAnalyzer 使用HAHA 解析这个文件。

5.得益于唯一的 reference key, HeapAnalyzer 找到 KeyedWeakReference，定位内存泄露。

6.HeapAnalyzer 计算 到 GC roots 的最短强引用路径，并确定是否是泄露。如果是的话，建立导致泄露的引用链。

7.引用链传递到 APP 进程中的 DisplayLeakService， 并以通知的形式展示出来。

ok,这里就不再深入了，想要了解更多就到 [github 主页](https://github.com/square/leakcanary) 这去哈。

### 7. 总结

- 对 Activity 等组件的引用应该控制在 Activity 的生命周期之内； 如果不能就考虑使用 getApplicationContext 或者 getApplication，以避免 Activity 被外部长生命周期的对象引用而泄露。
- 尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context )，即使要使用，也要考虑适时把外部成员变量置空；也可以在内部类中使用弱引用来引用外部类的变量。
- 对于生命周期比Activity长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄漏：
  - 将内部类改为静态内部类
  - 静态内部类中使用弱引用来引用外部类的成员变量
- Handler 的持有的引用对象最好使用弱引用，资源释放时也可以清空 Handler 里面的消息。比如在 Activity onStop 或者 onDestroy 的时候，取消掉该 Handler 对象的 Message和 Runnable.
- 在 Java 的实现过程中，也要考虑其对象释放，最好的方法是在不使用某对象时，显式地将此对象赋值为 null，比如使用完Bitmap 后先调用 recycle()，再赋为null,清空对图片等资源有直接引用或者间接引用的数组（使用 array.clear() ; array = null）等，最好遵循谁创建谁释放的原则。
- 正确关闭资源，对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。
- 保持对对象生命周期的敏感，特别注意单例、静态对象、全局性集合等的生命周期。



## 九、Android布局优化之include、merge、ViewStub的使用



## 十、Android权限处理



## 十一、Android热修复原理



## 十二、Android插件化入门指南



## 十三、VirtualApk解析



## 十四、Android推送技术解析



## 十五、Android Apk安装过程



## 十六、PopupWindow和Dialog区别

