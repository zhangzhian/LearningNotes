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





















