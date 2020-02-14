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



### 使用文件共享



### 使用Messenger



### 使用AIDL



### 使用ContentProvider



### 使用Socket





## Binder连接池





## 选用合适的IPC方式















