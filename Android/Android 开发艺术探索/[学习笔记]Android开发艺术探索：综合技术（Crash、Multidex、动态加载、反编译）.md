# [学习笔记]Android开发艺术探索：综合技术（Crash、Multidex、动态加载、反编译）



## 使用CrashHandler来获取应用的crash信息

检测崩溃并了解详细的crash信息: 

首先需实现一个uncaughtExceptionHandler对象，在它的uncaughtException方法中获取异常信息并将其存储到SD卡或者上传到服务器中，然后调用Thread的setDefaultUncaughtExceptionHandler为当前进程的所有线程设置异常处理器。

```java
public class CrashHandler implements Thread.UncaughtExceptionHandler {
    private static final String TAG = "CrashHandler";
    private static final boolean DEBUG = true;

    private static final String PATH = Environment.getExternalStorageDirectory().getPath() + "/CrashTest/log/";
    private static final String FILE_NAME = "crash";
    private static final String FILE_NAME_SUFFIX = ".trace";

    private static CrashHandler sInstance = new CrashHandler();
    private Thread.UncaughtExceptionHandler mDefaultCrashHandler;
    private Context mContext;

    private CrashHandler() {
    }

    public static CrashHandler getInstance() {
        return sInstance;
    }

    public void init(Context context) {
        mDefaultCrashHandler = Thread.getDefaultUncaughtExceptionHandler();
        Thread.setDefaultUncaughtExceptionHandler(this);
        mContext = context.getApplicationContext();
    }

    /**
     * 这个是最关键的函数，当程序中有未被捕获的异常，系统将会自动调用#uncaughtException方法
     * thread为出现未捕获异常的线程，ex为未捕获的异常，有了这个ex，我们就可以得到异常信息。
     */
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        try {
            //导出异常信息到SD卡中
            dumpExceptionToSDCard(ex);
            uploadExceptionToServer();
            //这里可以通过网络上传异常信息到服务器，便于开发人员分析日志从而解决bug
        } catch (IOException e) {
            e.printStackTrace();
        }

        ex.printStackTrace();

        //如果系统提供了默认的异常处理器，则交给系统去结束我们的程序，否则就由我们自己结束自己
        if (mDefaultCrashHandler != null) {
            mDefaultCrashHandler.uncaughtException(thread, ex);
        } else {
            Process.killProcess(Process.myPid());
        }
    }

     private void dumpExceptionToSDCard(Throwable ex) throws IOException {
        //如果SD卡不存在或无法使用，则无法把异常信息写入SD卡
        if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
            if (DEBUG) {
                Log.w(TAG, "sdcard unmounted,skip dump exception");
                return;
            }
        }
        File dir = new File(PATH);
        if (!dir.exists()) {
            dir.mkdirs();
        }
        long current = System.currentTimeMillis();
        String time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(current));
        File file = new File(PATH + FILE_NAME + time + FILE_NAME_SUFFIX);

        try {
            PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
            pw.println(time);
            dumpPhoneInfo(pw);
            pw.println();
            ex.printStackTrace(pw);
            pw.close();
        } catch (Exception e) {
            Log.e(TAG, "dump crash info failed");
        }
    }

    	private void dumpPhoneInfo(PrintWriter pw) throws NameNotFoundException {
        PackageManager pm = mContext.getPackageManager();
        PackageInfo pi = pm.getPackageInfo(mContext.getPackageName(), PackageManager.GET_ACTIVITIES);
        pw.print("App Version: ");
        pw.print(pi.versionName);
        pw.print('_');
        pw.println(pi.versionCode);

        //android版本号
        pw.print("OS Version: ");
        pw.print(Build.VERSION.RELEASE);
        pw.print("_");
        pw.println(Build.VERSION.SDK_INT);

        //手机制造商
        pw.print("Vendor: ");
        pw.println(Build.MANUFACTURER);

        //手机型号
        pw.print("Model: ");
        pw.println(Build.MODEL);

        //cpu架构
        pw.print("CPU ABI: ");
        pw.println(Build.CPU_ABI);
    }

  

    private void uploadExceptionToServer() {
        //TODO 本方法用于将错误信息上传至服务器
    }
}
```

然后在Application初始化的时候为线程设置CrashHandler，这样之后，Crash就会通过我们自己的异常处理器来处理异常了。

```java
public class BaseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        CrashHandler crashHandler = CrashHandler.getInstance();
        crashHandler.init(this);
    } 
}
```

## 使用multidex来解决方法数越界

在Android中单个dex文件所能包含的最大方法数为65536，这个是包含Android FrameWork、依赖jar包以及应用本身代码中的所有方法。达到这个65536后，编译器编译时会抛出DexIndexOverflowException异常。

Google提供了multidex解决方案。在Android5.0之前需要引入Google提供的android-support-multidex.jar；从5.0开始系统默认支持了multidex，它可以从apk文件中加载多个dex文件。

**使用步骤**：

1. 修改对应工程目录下的build.gradle文件，在defaultConfig中添加multiDexEnabled true这个配置项。

2. 在build.gradle的dependencies中添加multidex的依赖：

   'compile 'com.android.support:multidex:1.0.0'

```php
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"
    defaultConfig {
        applicationId "cn.hudp.androiddevartnote"
        minSdkVersion 14
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
        multiDexEnabled true  //关键部分
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile 'com.android.support:multidex:1.0.0' //关键部分
}
```

代码中加入支持multidex功能。

- 第一种方案，在manifest文件中指定Application为MultiDexApplication。
- 第二种方案，让应用的Application继承MultiDexApplication。
- 第三种方案，重写attachBaseContext方法，这个方法比onCreate还要先执行。

```java
public class BaseApplication extends Application {
  @Override
  protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        MultiDex.install(this);
    }
}
```

采用上面的配置项后，如果这个应用方法数没有越界，那么Gradle是不会生成多个dex文件的，当方法数越界后，Gradle就会在apk中打包2个或多个dex文件。当需要指定主dex文件中所包含的类，这时候就需要通过--multi-dex-list来选项来实现这个功能。

```c
//在对应工程目录下的build.gradle文件，加入
afterEvaluate {
    println "afterEvaluate"
    tasks.matching {
        it.name.startsWith('dex')
    }.each { dx ->
        def listFile = project.rootDir.absolutePath + '/app/maindexlist.txt'
        println "root dir:" + project.rootDir.absolutePath
        println "dex task found: " + dx.name
        if (dx.additionalParameters == null) {
            dx.additionalParameters = []
        }
        dx.additionalParameters += '--multi-dex'
        dx.additionalParameters += '--main-dex-list=' + listFile
        dx.additionalParameters += '--minimal-main-dex'
    }
} 
```

maindexlist.txt

```c
com/ryg/multidextest/TestApplication.class
com/ryg/multidextest/MainActivity.class
  
// multidex 这9个类必须在主Dex中
android/support/multidex/MultiDex.class
android/support/multidex/MultiDexApplication.class
android/support/multidex/MultiDexExtractor.class
android/support/multidex/MultiDexExtractor$1.class
android/support/multidex/MultiDex$V4.class
android/support/multidex/MultiDex$V14.class
android/support/multidex/MultiDex$V19.class
android/support/multidex/ZipUtil.class
android/support/multidex/ZipUtil$CentralDirectory.class
```

需要注意multidex的jar中的9个类必须要打包到主dex中，因为Application的attachBaseContext方法中需要用到MultiDex.install(this)需要用到MultiDex。

Multidex的缺点：

1. 启动速度会降低，由于应用启动时会加载额外的dex文件，这将导致应用的启动速度降低，甚至产生ANR现象。
2. 因为Dalvik linearAlloc的bug，可以导致使用multidex的应用无法在Android4.0之前的手机上运行，需要做大量兼容性测试。

## Android动态加载

各种插件化方案都需要解决3个基础性问题：

1. 资源访问，因为插件中凡是以R开头的资源文件都不能访问了。
2. Activity的生命周期管理，因为宿主动态将Activity.java加载到内存的时候，是不具备Activity的任何特性的，只是一个普通的java类。
3. ClassLoader的管理，为了避免多个ClassLoader加载了同一个类所引发的类型转换错误。

## 反编译初步

1. 使用dex2jar和jd-gui反编译apk
2. 使用apktool对apk进行二次打包