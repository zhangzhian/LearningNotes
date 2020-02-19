# [学习笔记]Android开发艺术探索：IPC机制



## Android IPC简介

IPC为进程间通讯，两个进程之间进行数据交换的过程。

IPC不是Android所独有的，任何一个操作系统都有对应的IPC机制。Windows上通过剪切板、管道、油槽等进行进程间通讯。Linux上通过命名空间、共享内容、信号量等进行进程间通讯。Android中没有完全继承于Linux，有特色的进程间通讯方式是Binder，还支持Socket。

## Android中的多进程模式

- 同一个应用，通过给四大组件在AndroidMenifest指定android:process属性，就可以开启多进程模式。
- 非常规的方式，通过JNI在Native层fork一个新的进程。
- 查看进程信息：DDMS；adb shell ps；adb shell ps | grep [包名]。
- 进程名以":"开头的属于当前应用的私有进程，其他应用的组件不可以和他跑在同一个进程里面。而进程名不以":"开头的进程属于全局进程，其他应用通过ShareUID方式可以和它跑在同一个进程中。
- 两个应用可以通过ShareUID跑在同一个进程并且签名相同，他们可以共享data目录、组件信息、共享内存数据。
- 多进程通讯的**问题**：

1. 静态成员和单例模式完全失效。

2. 线程同步机制完全失效。

3. SharedPreferences的可靠性下降

4. Application会多次创建

   问题1、2原因是因为进程不同，已经不是同一块内存了；
   问题3是因为SharedPreferences不支持两个进程同事进行读写操作，有一定几率导致数据丢失；
   问题4是当一个组件跑在一个新的进程中，系统会为他创建新的进程同时分配独立的虚拟机，其实就是启动一个应用的过程，因此相当于系统又把这个应用重新启动了一遍，Application也重新创建。

多进程中，不同的进程组件拥有独立的虚拟机，Application，内存。

实现跨进程通讯有很多方式，比如：通过Intent，共享文件、SharedPreferences、基于Binder的Messenger和AIDL、Socket等。

## IPC基础概念介绍

### Serializable接口

Serializable是Java所提供的一个序列化接口，空接口。

serialVersionUID是用来辅助序列化和反序列化过程的，原则上序列化后的数据中的serialVersionUID要和当前类的serialVersionUID相同才能正常的序列化。

- 静态成员变量属于类不属于对象，所以不会参加序列化过程；

- 其次用transient关键字标明的成员变量也不参加序列化过程。

重写如下两个方法可以重写系统默认的序列化和反序列化过程

```Java
private void writeObject(java.io.ObjectOutputStream out)throws IOException{
}
private void readObject(java.io.ObjectInputStream out)throws IOException,ClassNotFoundException{
}
```



### Parcelable接口

Android中特有的序列化方式，效率相对Serializable更高，占用内存相对也更少，但使用起来稍微麻烦点。

```java
public class User implements Parcelable {
    public int userId;
    public String userName;
    public boolean isMale;

    public Book book;

    public User() {
    }

    public User(int userId, String userName, boolean isMale) {
        this.userId = userId;
        this.userName = userName;
        this.isMale = isMale;
    }

    public int describeContents() {
        return 0;//返回当前对象的内容描述，含有文件描述符返回1，否则0
    }

    public void writeToParcel(Parcel out, int flags) {//将当前对象写入序列号结构中
        out.writeInt(userId);
        out.writeString(userName);
        out.writeInt(isMale ? 1 : 0);
        out.writeParcelable(book, 0);
    }

    public static final Parcelable.Creator<User> CREATOR = new Parcelable.Creator<User>() {
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        public User[] newArray(int size) {
            return new User[size];
        }
    };

    private User(Parcel in) {
        userId = in.readInt();
        userName = in.readString();
        isMale = in.readInt() == 1;
        book = in.readParcelable(Thread.currentThread().getContextClassLoader());
    }

    @Override
    public String toString() {
        return String.format(
                "User:{userId:%s, userName:%s, isMale:%s}, with child:{%s}",
                userId, userName, isMale, book);
    }

}
```

序列化功能由**writeToParcel**方法来完成，最终通过Parcel中的一系列write方法完成的。反序列化功能由CREATOR来完成，其内部标明了如何创建序列号对象和数组，并通过Parcel的一系列read方法来完成反序列化过程。内容描述功能由describeContents方法来完成，几乎所有情况都返回0，只有当前对象存在文件描述符时，才返回1。

Serializable是Java中的序列化接口，简单但开销大，序列化和反序列化需要大量的IO操作。

Parceable是Android中的序列化方式，使用起来麻烦，但是效率高。

### Binder

#### Binder简介

直观来说，Binder是Android中一个类，实现了IBinder接口；

从IPC的角度来说，Binder是Android的一种跨进程的通讯方式；Binder还可以理解为一种虚拟的物理设备，设备驱动是/dev/binder；

从Android Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager、等等）和相应ManagerService的桥梁；

从Android应用层来说，Binder是客户端与服务端通讯的媒介。在Android开发中，Binder主要用于Service中，包括AIDL和Messenger，其中普通的Service的Binder不涉及进程间通讯；而Messenger的底层其实就是AIDL。

AIDL:

```java
Book.aidl:
package com.zza.stardust.app.ui.androidart;

parcelable Book;

IBookManager.aidl
package com.zza.stardust.app.ui.androidart;

import com.zza.stardust.app.ui.androidart.Book;

//  /build/generated/aidl_source_output_dir目录下的com.zza.stardust.app.ui.androidart包中
interface IBookManager {
     List<Book> getBookList();
     void addBook(in Book book);
}
```

生成的Java类:

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 * Original file: /app/src/main/aidl/com/zza/stardust/app/ui/androidart/IBookManager.aidl
 */
package com.zza.stardust.app.ui.androidart;
//gen目录下的com.zza.stardust.app.ui.androidart.aidl包中

public interface IBookManager extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.zza.stardust.app.ui.androidart.IBookManager {
        private static final java.lang.String DESCRIPTOR = "com.zza.stardust.app.ui.androidart.IBookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.zza.stardust.app.ui.androidart.IBookManager interface,
         * generating a proxy if needed.
         */
        public static com.zza.stardust.app.ui.androidart.IBookManager asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.zza.stardust.app.ui.androidart.IBookManager))) {
                return ((com.zza.stardust.app.ui.androidart.IBookManager) iin);
            }
            return new com.zza.stardust.app.ui.androidart.IBookManager.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getBookList: {
                    data.enforceInterface(descriptor);
                    java.util.List<com.zza.stardust.app.ui.androidart.Book> _result = this.getBookList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                case TRANSACTION_addBook: {
                    data.enforceInterface(descriptor);
                    com.zza.stardust.app.ui.androidart.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.zza.stardust.app.ui.androidart.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addBook(_arg0);
                    reply.writeNoException();
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.zza.stardust.app.ui.androidart.IBookManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public java.util.List<com.zza.stardust.app.ui.androidart.Book> getBookList() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<com.zza.stardust.app.ui.androidart.Book> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(com.zza.stardust.app.ui.androidart.Book.CREATOR);
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            @Override
            public void addBook(com.zza.stardust.app.ui.androidart.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    public java.util.List<com.zza.stardust.app.ui.androidart.Book> getBookList() throws android.os.RemoteException;

    public void addBook(com.zza.stardust.app.ui.androidart.Book book) throws android.os.RemoteException;

}

```

系统会根据AIDL文件生成同名的.java类；首先声明与AIDL文件相对应的几个接口方法，还申明每个接口方法相对应的int id来做为标识，这几个id在transact过程中标识客户端请求的到底是什么方法。接着会声明一个内部类Stub，这个Stub就是一个Binder类，当客户端和服务端处于同一个进程的时候，方法调用不会走transact过程，处于不同进程时，方法调用会走transact过程，这个逻辑由Stub的内部代理类Proxy来完成。所以核心实现在于它的内部类Stub和Stub的内部代理类Proxy，下面分析其中的方法：

**DESCRIPTOR**：Binder的唯一标识，一般用当前Binder的类名表示。

**asInterface(android.os.IBinder obj)**：将服务端的Binder对象转换成客户端所需要的AIDL接口类型的对象；如果客户端和服务端位于相同进程，那么此方法返回的就是服务端Stub对象本身，否则返回系统封装后的Stub.proxy对象。

**asBinder**：用于返回当前的Binder对象

**onTransact**：运行在服务端的Binder线程池中，当客户端发起跨进程通讯时，远程请求会通过系统底层封装交由此方法处理。

```java
 public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException 
```

- 服务端通过code确定客户端请求的目标方法是什么
- 接着从data取出目标方法所需要的参数，然后执行目标方法
- 执行后向reply写入返回值
- 如果返回false，服务端请求会失败，可以做权限验证

**Proxy#getBookList和Proxy#addBook**：

- 运行在客户端，首先该方法所需要的输入型对象Parcel _data对象，输出对象Parcel _reply对象和返回值对象List
- 然后把该方法的参数信息写入Parcel _data对象。
- 接着调研transact方法发起RPC，同时当前线程挂起
- 然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行，并从_reply取出RPC的返回结果，最后返回 _reply中的数据

Binder的两个重要方法**linkToDeath**和**unlinkToDeath**。通过linkToDeath可以给Binder设置一个死亡代理，当Binder死亡时，我们就会收到通知，然后就可以重新发起连接请求。声明一个DeathRecipient对象，DeathRecipient是一个接口，其内部只有一个方法binderDied，实现这个方法后就可以在Binder死亡的时候收到通知了。

```java
private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient(){
    @Override
    public void binderDied(){
        if(mBookManager == null){
            return;
        }
        mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
        mBookManager = null;
        // TODO：接下来重新绑定远程Service
    }
}
```

在客户端绑定远程服务成功后，给Binder设置死亡代理：

```undefined
mService = IBookManager.Stub.asInterface(binder);
binder.linkToDeath(mDeathRecipient,0);
```

## Android中的IPC方式

### 使用Bundle

由于Binder实现了Parcelable接口，所以可以方便的在不同进程中传输；Activity、Service和Receiver都支持在Intent中传递Bundle数据。

### 使用文件共享

两个进程通过读/写一个文件来交换数据

适合对数据同步要求性不高的场景，并要避免并发写这种场景或者处理好线程同步问题。

SharedPreferences是个特例，虽然也是文件的一种，但系统在内存中有一份SharedPreferences文件的缓存，因此在多线程模式下，系统的读/写就变得不可靠，高并发读写SharedPreferences有一定几率会丢失数据，因此不建议在多进程通信中使用SharedPreferences。

### 使用Messenger

可以在不同的进程之间传递Message对象。Messenger是轻量级的IPC方案，底层实现是AIDL，对AIDL进行了封装。Messenger 服务端是以串行的方式来处理客户端的请求的，不存在并发执行的情形。

服务端：

- 创建一个Service来处理客户端的连接请求
- 创建一个Handler并通过它来创建一个Messager对象
- 在Service的onBind中返回这个Messager对象底层的Binder即可

```java

public class MessengerService extends Service {
    public MessengerService() {
    }

    private static class MessengerHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    ToastUtil.show("receive msg from Client:" + msg.getData().getString("msg"));
                    Messenger client = msg.replyTo;
                    Message relpyMessage = Message.obtain(null, 1);
                    Bundle bundle = new Bundle();
                    bundle.putString("reply", "嗯，你的消息我已经收到，稍后会回复你。");
                    relpyMessage.setData(bundle);
                    try {
                        client.send(relpyMessage);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    private final Messenger mMessenger = new Messenger(new MessengerHandler());

    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }
}
```

客户端

- 绑定这个服务端的Server
- 用服务端返回的IBinder对象创建一个Messager对象
- 若需要回应客户端，需要创建一个Handler并创建一个新的Messager，并通过Messa的replyTo参数传递给服务端，服务端通过这个replyTo参数就可以回应客户端

```java
public class MessengerActivity extends Activity {

    private Messenger mService;
    private Messenger mGetReplyMessenger = new Messenger(new MessengerHandler());

    private static class MessengerHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    LogUtil.i("receive msg from Service:" + msg.getData().getString("reply"));
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder service) {
            mService = new Messenger(service);
            LogUtil.d("bind service");
            Message msg = Message.obtain(null, 1);
            Bundle data = new Bundle();
            data.putString("msg", "hello, this is client.");
            msg.setData(data);
            msg.replyTo = mGetReplyMessenger;
            try {
                mService.send(msg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        public void onServiceDisconnected(ComponentName className) {
        }
    };

    @Override
    protected void onDestroy() {
        unbindService(mConnection);
        super.onDestroy();
    }

   @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_messenger);
        findViewById(R.id.bt_start_service).setOnClickListener(v -> {
            Intent intent = new Intent();
            intent.setAction("com.zza.MessengerService.launch");
            intent.setPackage(this.getPackageName());//需要添加包名
            bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
        });
    }
}
```

### 使用AIDL

Messenger不适合处理大并发请求。Messenger主要作用传递消息，跨进程调用服务端方法，无法做到。

**服务端**首先创建一个Service用来监听客户端的连接请求，然后创建一个AIDL文件，将暴露给客户端的接口在AIDL文件中声明，最后在Service中实现这个AIDL接口即可。

**客户端**首先绑定服务端的Service，绑定成功后，将服务端返回的Binder对象转化成AIDL接口所属的类型，调用相对应的AIDL中的方法。

AIDL支持的数据类型：

- 基本数据类型；

- String、CharSequence；

- List： 只支持ArrayList,里面的元素必须都能被AIDL所支持；

- Map： 只支持HashMap，里面的元素（key和value）必须都能被AIDL所支持；
- Parcelable： 所有实现了Parcelable接口的对象；

- AIDL： 所有AIDL接口本身也可以在AIDL文件中使用。

自定义的Parcelable对象和AIDL对象必须显示的import进来（即使在同一个包）。

如果ALDL文件中用到了自定义的Parcelable类型，必须新建一个和它同名ALDL文件，并在其中声明它为Parcelable类型。

AIDL中除了基本数据类型，其他类型参数必须标上方向：in、out或inout。

AIDL接口中只支持方法，不支持声明静态常量。

为了方便AIDL开发，建议把所有和AIDL相关的类和文件都放在同一个包中，好处在于，当客户端是另一个应用的时候，我们可以直接把整个包复制到客户端工程中去。

AIDL的包结构在服务端和客户端要保持一致，否则会运行出错。

客户端的listener和服务端的listener不是同一个对象，RemoteCallbackList是系统专门提供用于删除跨进程listener的接口，RemoteCallbackList是泛型，支持管理任意的AIDL接口，因为所有AIDL接口都继承自android.os.IInterface接口。

需注意AIDL客户端发起RPC过程的时候，客户端的线程会挂起，如果是UI线程发起的RPC过程，如果服务端处理事件过长，就会导致ANR。

AIDL：

```java
package com.zza.stardust.app.ui.androidart.aidl;

parcelable Book;



package com.zza.stardust.app.ui.androidart.aidl;

import com.zza.stardust.app.ui.androidart.aidl.Book;
import com.zza.stardust.app.ui.androidart.aidl.IOnNewBookArrivedListener;

//  /build/generated/aidl_source_output_dir目录下的com.zza.stardust.app.ui.androidart包中
interface IBookManager {
     List<Book> getBookList();
     void addBook(in Book book);
     void registerListener(IOnNewBookArrivedListener listener);
     void unregisterListener(IOnNewBookArrivedListener listener);
}



package com.zza.stardust.app.ui.androidart.aidl;

import com.zza.stardust.app.ui.androidart.aidl.Book;

interface IOnNewBookArrivedListener {
    void onNewBookArrived(in Book newBook);
}

```

服务端：

```java
public class BookManagerService extends Service {

    private AtomicBoolean mIsServiceDestoryed = new AtomicBoolean(false);

    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
    // private CopyOnWriteArrayList<IOnNewBookArrivedListener> mListenerList =
    // new CopyOnWriteArrayList<IOnNewBookArrivedListener>();

    private RemoteCallbackList<IOnNewBookArrivedListener> mListenerList = new RemoteCallbackList<>();

    private Binder mBinder = new IBookManager.Stub() {

        @Override
        public List<Book> getBookList() throws RemoteException {
            //测试
            //SystemClock.sleep(5000);
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
        }

        public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
                throws RemoteException {
            int check = checkCallingOrSelfPermission("com.zza.permission.ACCESS_BOOK_SERVICE");
            LogUtil.d("check=" + check);
            if (check == PackageManager.PERMISSION_DENIED) {
                return false;
            }

            String packageName = null;
            String[] packages = getPackageManager().getPackagesForUid(
                    getCallingUid());
            if (packages != null && packages.length > 0) {
                packageName = packages[0];
            }
            LogUtil.d("onTransact: " + packageName);
            if (!packageName.startsWith("com.zza")) {
                return false;
            }

            return super.onTransact(code, data, reply, flags);
        }

        @Override
        public void registerListener(IOnNewBookArrivedListener listener)
                throws RemoteException {
            mListenerList.register(listener);

            final int N = mListenerList.beginBroadcast();
            mListenerList.finishBroadcast();
            LogUtil.d("registerListener, current size:" + N);
        }

        @Override
        public void unregisterListener(IOnNewBookArrivedListener listener)
                throws RemoteException {
            boolean success = mListenerList.unregister(listener);

            if (success) {
                LogUtil.d("unregister success.");
            } else {
                LogUtil.d("not found, can not unregister.");
            }
            final int N = mListenerList.beginBroadcast();
            mListenerList.finishBroadcast();
            LogUtil.d("unregisterListener, current size:" + N);
        }

        ;

    };

    @Override
    public void onCreate() {
        super.onCreate();
        mBookList.add(new Book(1, "Android"));
        mBookList.add(new Book(2, "Ios"));
        new Thread(new ServiceWorker()).start();
    }

    @Override
    public IBinder onBind(Intent intent) {
        int check = checkCallingOrSelfPermission("com.zza.permission.ACCESS_BOOK_SERVICE");
        LogUtil.d("onbind check=" + check);
        if (check == PackageManager.PERMISSION_DENIED) {
            return null;
        }
        return mBinder;
    }

    @Override
    public void onDestroy() {
        mIsServiceDestoryed.set(true);
        super.onDestroy();
    }

    private void onNewBookArrived(Book book) throws RemoteException {
        mBookList.add(book);
        final int N = mListenerList.beginBroadcast();
        for (int i = 0; i < N; i++) {
            IOnNewBookArrivedListener l = mListenerList.getBroadcastItem(i);
            if (l != null) {
                try {
                    l.onNewBookArrived(book);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
        mListenerList.finishBroadcast();
    }

    private class ServiceWorker implements Runnable {
        @Override
        public void run() {
            // do background processing here.....
            while (!mIsServiceDestoryed.get()) {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                int bookId = mBookList.size() + 1;
                Book newBook = new Book(bookId, "new book#" + bookId);
                try {
                    onNewBookArrived(newBook);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

客户端：

```java
public class BookManagerActivity extends MActivity {

    private static final int MESSAGE_NEW_BOOK_ARRIVED = 1;

    private IBookManager mRemoteBookManager;

    @SuppressLint("HandlerLeak")
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MESSAGE_NEW_BOOK_ARRIVED:
                    LogUtil.d("receive new book :" + msg.obj);
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    };

    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            LogUtil.d("binder died. tname:" + Thread.currentThread().getName());
            if (mRemoteBookManager == null)
                return;
            mRemoteBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
            mRemoteBookManager = null;
            // TODO:这里重新绑定远程Service
        }
    };

    private ServiceConnection mConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            mRemoteBookManager = bookManager;
            try {
                mRemoteBookManager.asBinder().linkToDeath(mDeathRecipient, 0);
                List<Book> list = bookManager.getBookList();
                LogUtil.d("query book list, list type:"
                        + list.getClass().getCanonicalName());
                LogUtil.d("query book list:" + list.toString());
                Book newBook = new Book(3, "Android进阶");
                bookManager.addBook(newBook);
                LogUtil.d("add book:" + newBook);
                List<Book> newList = bookManager.getBookList();
                LogUtil.d("query book list:" + newList.toString());
                bookManager.registerListener(mOnNewBookArrivedListener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        public void onServiceDisconnected(ComponentName className) {
            mRemoteBookManager = null;
            LogUtil.d("onServiceDisconnected. tname:" + Thread.currentThread().getName());
        }
    };

    private IOnNewBookArrivedListener mOnNewBookArrivedListener = new IOnNewBookArrivedListener.Stub() {

        @Override
        public void onNewBookArrived(Book newBook) throws RemoteException {
            mHandler.obtainMessage(MESSAGE_NEW_BOOK_ARRIVED, newBook)
                    .sendToTarget();
        }
    };

    @Override
    protected void onInit(Bundle savedInstanceState) {
        super.onInit(savedInstanceState);
        Intent intent = new Intent(this, BookManagerService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }

    public void onButton1Click(View view) {
        Toast.makeText(this, "click button1", Toast.LENGTH_SHORT).show();
        new Thread(new Runnable() {

            @Override
            public void run() {
                if (mRemoteBookManager != null) {
                    try {
                        List<Book> newList = mRemoteBookManager.getBookList();
                        runOnUiThread(() -> ToastUtil.show("size:" + newList.size()));
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }

    @Override
    protected void onDestroy() {
        if (mRemoteBookManager != null
                && mRemoteBookManager.asBinder().isBinderAlive()) {
            try {
                LogUtil.d("unregister listener:" + mOnNewBookArrivedListener);
                mRemoteBookManager
                        .unregisterListener(mOnNewBookArrivedListener);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
        unbindService(mConnection);
        super.onDestroy();
    }

    @Override
    protected BasePresenter createPresenter() {
        return null;
    }

    @Override
    protected int provideContentViewId() {
        return R.layout.activity_book_manager;
    }
}
```

### 使用ContentProvider

ContentProvider是Android专门用于不同应用之间进行数据共享的方式，适合跨进程通信，底层采用Binder实现。

ContentProvider主要以表格的形式来组织数据，可以包含多个表；ContentProvider支持普通文件，甚至可以采用内存中得一个对象来进行数据存储。

通过ContentProvider的notifyChange方法来通知外界当前ContentProvider中的数据已经发生改变。

query、update、insert、delete四大方法是存在多线程并发访问的，内部需要做好线程同步。



### 使用Socket





## Binder连接池





## 选用合适的IPC方式















