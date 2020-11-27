# Java

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 基础

### 一、Object

#### 1. 两个值相等的 Integer 对象，== 比较，判断是否相等

Integer对象值的范围为 `[-128 - 127]` 时，不会创建新的引用，指向同一个地址，地址值相等。

但是当超过这个范围后，就会 new 一个对象，引用指向的地址是不同的，地址值不相等。

#### 2. 下面的代码，再次使用对象 student 是否需要判空？

> **Java 中方法参数的使用情况总结：**
>
> - 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）；
> - 一个方法可以改变一个对象参数的状态；
> - 一个方法不能让对象参数引用一个新的对象

```csharp
public void test()  {
    Student student = new Student("Bobo", 15);
    changeValue1(student);   // student值未改变，不为null! 输出结果 student值为 name:Bobo、age:15
    // changeValue2(student);  // student值被改变，输出结果 student值为 name:Lily、age:20
    System.out.println("student值为 name: " + student.name + "、age:" + student.age);
}

public static void changeValue1(Student student) {
    student = null;
}

public static void changeValue2(Student student)  {    
     student.name = "Lily";    
     student.age = 20;
}
```

#### 3. equals和==的区别？equals和hashcode的关系？

- ==：基本类型比较值，引用类型比较地址。
- `equals`：默认情况下，`equals`作为对象中的方法，比较的是地址，不过可以根据业务，修改`equals`方法。

`equals`和`hashcode`之间的关系：

默认情况下，`equals`相等，`hashcode`必相等，`hashcode`相等，`equals`不是必相等。hashcode基于内存地址计算得出，可能会相等，虽然几率微乎其微。



### 二、String

#### 1. 下面的代码， str 值最终为多少？换成 Integer 值又为多少，是否会被改变？

> - **考点**：Java 值传递 (第 2 题相同)。编写代码测试，在 changeValue() 方法中修改入参，并**不会改变**之前的值；
> - **原理** ：[Java 程序设计语言总是采用按值调用](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FSnailclimb%2FJavaGuide%2Fblob%2Fmaster%2Fdocs%2Fessential-content-for-interview%2FPreparingForInterview%2F%E5%BA%94%E5%B1%8A%E7%94%9F%E9%9D%A2%E8%AF%95%E6%9C%80%E7%88%B1%E9%97%AE%E7%9A%84%E5%87%A0%E9%81%93Java%E5%9F%BA%E7%A1%80%E9%97%AE%E9%A2%98.md%23%E4%B8%80-%E4%B8%BA%E4%BB%80%E4%B9%88-java-%E4%B8%AD%E5%8F%AA%E6%9C%89%E5%80%BC%E4%BC%A0%E9%80%92)，方法得到的是所有参数值的一个拷贝，即方法**不能修改**传递给它的任何参数变量的内容。基本类型参数传递的是参数副本，对象类型参数传递的是**对象地址的副本；**
> - **题解**：在 changeValue() 中，对于对象类型参数，直接修改的是**对象地址副本**的值，所以之前变量的地址并未被修改！若修改的是对象实例里面的某个值，之前变量则会被修改



```rust
public void test() {
    String str = "123";
    changeValue(str); 
    System.out.println("str值为: " + str);  // str未被改变，str = "123"
}

public changeValue(String str) {
    str = "abc";
}
```

#### 2. String、StringBuffer和StringBuilder的区别？

- `String`：`String`属于不可变对象，每次修改都会生成新的对象。
- `StringBuilder`：可变对象，非多线程安全。
- `StringBuffer`：可变对象，多线程安全。

大部分情况下，效率是：`StringBuilder`>`StmiandringBuffer`>`String`。

#### 3. Java中中文字符和英文字符的大小分别多少？在网络上传输大小又分别是多少？

Java采用unicode编码，2个字节来表示一个字符，这点与C语言中不同，C语言中采用ASCII，在大多数系统中，一个字符通常占1个字节，但是在0~127整数之间的字符映射，unicode向下兼容ASCII。而Java采用unicode来表示字符，一个中文或英文字符的unicode编码都占2个字节。但如果采用其他编码方式，一个字符占用的字节数则各不相同。

在 GB 2312 编码或 GBK 编码中，一个英文字母字符存储需要1个字节，一个汉字字符存储需要2个字节。 在UTF-8编码中，一个英文字母字符存储需要1个字节，一个汉字字符储存需要3到4个字节。在UTF-16编码中，一个英文字母字符存储需要2个字节，一个汉字字符储存需要3到4个字节（Unicode扩展区的一些汉字存储需要4个字节）。在UTF-32编码中，世界上任何字符的存储都需要4个字节。

**简单来讲，一个字符表示一个汉字或英文字母，具体字符与字节之间的大小比例视编码情况而定。有时候读取的数据是乱码，就是因为编码方式不一致，需要进行转换，然后再按照unicode进行编码。**

网络上传输大小根据字节数再加上协议头尾的长度决定。

#### 4. String是java中的基本数据类型吗？是可变的吗？是线程安全的吗？

- `String`不是基本数据类型，java中把大数据类型是：`byte, short, int, long, char, float, double, boolean`
- `String`是不可变的
- `String`是不可变类，一旦创建了String对象，我们就无法改变它的值。因此，它是**线程安全**的，可以安全地用于多线程环境中

#### 5. 为什么要设计成不可变的呢？如果String是不可变的，那我们平时赋值是改的什么呢？

1）为什么设计不可变

- **安全**。由于String广泛用于java类中的参数，所以安全是非常重要的考虑点。包括线程安全，打开文件，存储数据密码等等。

- String的不变性保证哈希码始终一，所以在用于HashMap等类的时候就不需要重新计算哈希码，**提高效率**。

- 因为java字符串是不可变的，可以在java运行时**节省大量java堆空间**。因为不同的字符串变量可以引用池中的相同的字符串。如果字符串是可变得话，任何一个变量的值改变，就会反射到其他变量，那字符串池也就没有任何意义了。

2）平时使用双引号方式赋值的时候其实是返回的字符串引用,并不是改变了这个字符串对象

#### 6. 浅谈一下String, StringBuffer，StringBuilder的区别？String的两种创建方式，在JVM的存储方式相同吗？
String是不可变类，每当我们对String进行操作的时候，总是会创建新的字符串。操作String很耗资源,所以Java提供了两个工具类来操作String - StringBuffer和StringBuilder。

StringBuffer和StringBuilder是可变类，StringBuffer是线程安全的，StringBuilder则不是线程安全的。所以在多线程对同一个字符串操作的时候，我们应该选择用StringBuffer。由于不需要处理多线程的情况，StringBuilder的效率比StringBuffer高。

1） String常见的创建方式有两种

- String s1 = “Java”
- String s2 = new String("Java")

2）存储方式不同

- 第一种，s1会先去字符串常量池中找字符串"Java”，如果有相同的字符则直接返回常量句柄，如果没有此字符串则会先在常量池中创建此字符串，然后再返回常量句柄，或者说字符串引用。

- 第二种，s2是直接在堆上创建一个变量对象，但不存储到字符串池 ，调用`intern`方法才会把此字符串保存到常量池中



### 三、继承

#### 1. Java中抽象类和接口的特点？

共同点：

- 抽象类和接口都不能生成具体的实例。
- 都是作为上层使用。

不同点：

- 抽象类可以有属性和成员方法，接口不可以。
- 一个类只能继承一个类，但是可以实现多个接口。
- **抽象类中的变量是普通变量，接口中的变量是静态变量。**
- 抽象类表达的是`is-a`的关系，接口表达的是`like-a`的关系。

#### 2. 关于多态的理解？

多态是面向对象的三大特性：继承、封装和多态之一。

多态的定义：允许不同类对同一消息做出响应。

多态存在的条件：

1. 要有继承。
2. 要有复写。
3. 父类引用指向子类对象。

Java中多态的实现方式：接口实现，继承父类进行方法重写，同一个类中的方法重载。

#### 3. Java有什么特性，继承有什么用处，多态有什么用处





### 四、注解

#### 1.编译期注解处理的是字节码还是java文件



#### 2. @JavaScriptInterface为什么不通过多个方法来实现？



#### 3. 说说你对注解的了解，是怎么解析的



### 五、序列化

#### 1. Seriazable与Parceable的区别



### 六、集合

#### 1. Java 集合，介绍下ArrayList 和 HashMap 的使用场景，底层实现原理

#### 2. ArrayList 与 LinkedList 的区别

#### 3. HashMap的特点是什么？HashMap的原理？

HashMap的特点：

1. 基于Map接口，存放键值对。
2. 允许key/value为空。
3. 非多线程安全。
4. 不保证有序，也不保证使用的过程中顺序不会改变。

简单来讲，核心是**数组+链表/红黑树**，HashMap的原理就是存键值对的时候：

1. 通过键的Hash值确定数组的位置。
2. 找到以后，如果该位置无节点，直接存放。
3. 该位置有节点即位置发生冲突，遍历该节点以及后续的节点，比较`key`值，相等则覆盖。
4. 没有就新增节点，默认使用链表，相连节点数超过8的时候，在jdk 1.8中会变成红黑树。
5. 如果Hashmap中的数组使用情况超过一定比例，就会扩容，默认扩容两倍。

当然这是存入的过程，其他过程可以自行查阅。这里需要注意的是：

- `key`的hash值计算过程是高16位不变，低16位和高16位取异或，让更多位参与进来，可以有效的减少碰撞的发生。
- 初始数组容量为16，默认不超过的比例为0.75。

#### 4. 讲讲LinkedHashMap的数据结构

- 底层是散列表和双向链表
- 允许为null，不同步
- 插入的顺序是有序的(底层链表致使有序)
- 装载因子和初始容量对LinkedHashMap影响是很大的

`put`函数在LinkedHashMap中未重新实现，只是实现了`afterNodeAccess`和`afterNodeInsertion`两个回调函数。`get`函数则重新实现并加入了`afterNodeAccess`来保证访问顺序。

LinkedHashMap的操作基本上都是为了维护好那个具有访问顺序的双向链表。



### 七、泛型

#### 1. 说一下对泛型的理解？

泛型的本质是**参数化类型，在不创建新的类型的情况下，通过泛型指定不同的类型来控制形参具体限制的类型**。也就是说在泛型的使用中，操作的数据类型被指定为一个参数，这种参数可以被用在类、接口和方法中，分别被称为泛型类、泛型接口和泛型方法。

泛型是Java中的一种语法糖，能够在代码编写的时候起到类型检测的作用，但是虚拟机是不支持这些语法的。

泛型的优点：

1. 类型安全，避免类型的强转。
2. 提高了代码的可读性，不必要等到运行的时候才去强制转换。

#### 2. 什么是类型擦除？

不管泛型的类型传入哪一种类型实参，对于Java来说，都会被当成同一类处理，在内存中也只占用一块空间。通俗一点来说，就是泛型只作用于代码编译阶段，在编译过程中，对于正确检验泛型结果后，会将泛型的信息擦除，也就是说，成功编译过后的class文件是不包含任何泛型信息的。

在**编译期。**[我眼中的Java-Type体系](https://www.jianshu.com/p/e8eeff12c306)

#### 3. 泛型是怎么解析的

#### 4. 泛型有什么优点？

#### 



### 八、反射机制

#### 1. 动态代理和静态代理

静态代理很简单，运用的就是代理模式：

![代理模式](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6cfb98cfb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



声明一个接口，再分别实现一个真实的主题类和代理主题类，通过让代理类持有真实主题类，从而控制用户对真实主题的访问。

动态代理指的是在运行时动态生成代理类，即代理类的字节码在运行时生成并载入当前的ClassLoader。

动态代理的原理是使用反射，思路和上面的一致。

使用动态代理的好处：

1. 不需要为`RealSubject`写一个形式完全一样的代理类。
2. 使用一些动态代理的方法可以在运行时制定代理类的逻辑，从而提升系统的灵活性。



#### 2. 反射是什么，在哪里用到，怎么利用反射创建一个对象



#### 3. 代理模式与装饰模式的区别，手写一个静态代理，一个动态代理



#### 4. 动态代理有什么作用

#### 5. 反射可以反射final修饰的字段吗？




### 九、IO
### 十、深拷贝和浅拷贝

### 十一、其他



#### 1. 静态方法，静态对象为什么不能继承



#### 2. 有用过什么加密算法？AES,RAS什么原理？



#### 3. 共享内存用过吗？



#### 4. 如何封装一个字符串转数字的工具类

####  5. Java 中的引用类型分类和区别，虚引用的使用场景？

#### 6. Base64算法是什么，是加密算法吗？

- `Base64`是一种将二进制数据转换成64种字符组成的字符串的编码算法，主要用于非文本数据的传输，比如图片。可以将图片这种二进制数据转换成具体的字符串，进行保存和传输。
- 严格来说，不算。虽然它确实把一段二进制数据转换成另外一段数据，但是他的加密和解密是公开的，也就无秘密可言了。所以我更倾向于认为它是一种编码，每个人都可以用base64对二进制数据进行编码和解码。
- `面试加分项`：为了减少混淆，方便复制，减少数据长度，就衍生出一种base58编码。去掉了base64中一些容易混淆的数字和字母（数字0，字母O，字母I，数字1，符号+，符号/）
   大名鼎鼎的比特币就是用的改进后的base58编码，即`Base58Check`编码方式，有了校验机制，加入了hash值。

#### 7. Base64底层



#### 8. Linux进程和Java进程有何区别



## 并发
### 一、线程

#### 1. 线程的状态有哪些？

线程的状态有：

- `New`：新创建的线程，进入了初始状态
- `Runnable`：准备就绪的线程，由于CPU分配的时间片的关系，此时的任务不在执行过程中
- `Running`：正在执行的任务获得了cpu 时间片 ，执行程序代码
- `Block`：被阻塞的任务，通常是为了等待某个时间的发生（比如说某项资源就绪）之后再继续运行
- `Terminated`：终止的任务，线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。

附上一张状态转换的图：

![线程状态转换](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6f77cf96c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 2. 线程中wait和sleep的区别？

`wait`方法既释放cpu，又释放锁。 `sleep`方法只释放cpu，但是不释放锁。

#### 3. 线程和进程的区别？

线程是CPU调度的最小单位，一个进程中可以包含多个线程，在Android中，一个进程通常是一个App，App中会有一个主线程，主线程可以用来操作界面元素，如果有耗时的操作，必须开启子线程执行，不然会出现ANR，除此以外，进程间的数据是独立的，线程间的数据可以共享。

#### 4. 线程间同步的方法



#### 5. 如何让两个线程循环交替打印



#### 6. 怎么中止一个线程，Thread.Interupt一定有效吗？

#### 7. 实现一个线程同步的计数器

#### 8. 为什么多线程同时访问（读写）同个变量，会有并发问题？
- Java 内存模型规定了所有的变量都存储在主内存中，每条线程有自己的工作内存。
- 线程的工作内存中保存了该线程中用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。
- 线程访问一个变量，首先将变量从主内存拷贝到工作内存，对变量的写操作，不会马上同步到主内存。
- 不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步。

#### 9. 说说原子性，可见性，有序性分别是什么意思？
原子性：在一个操作中，CPU 不可以在中途暂停然后再调度，即不被中断操作，要么执行完成，要么就不执行。

可见性：多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

有序性：程序执行的顺序按照代码的先后顺序执行。

#### 10. 实际项目过程中，有用到多线程并发问题的例子吗？

有，比如**单例模式**。由于单例模式的特殊性，可能被程序中不同地方多个线程**同时调用**，所以为了避免多线程并发问题，一般要采用`volatile+Synchronized`的方式进行变量，方法保护。

```java
    private volatile static Singleton singleton;

    public static Singleton getSingleton() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }

        }
        return singleton;
    }
```

### 二、线程池

#### 1. 说下对线程池的理解，以及创建线程池的几个关键参数

线程池的构造函数如下：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
	this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
}
```

**参数解释：**

- `corePoolSize`：核心线程数量，不会释放。
- `maximumPoolSize`：允许使用的最大线程池数量，非核心线程数量，闲置时会释放。
- `keepAliveTime`：闲置线程允许的最大闲置时间。
- `unit`：闲置时间的单位。
- `workQueue`：阻塞队列，不同的阻塞队列有不同的特性。
- `ThreadFactory`：线程工厂。定制一个线程，可以设置线程的优先级、名字、是否后台线程、状态等。一般无须设置该参数。
- `RejectedExecutionHandler`：拒绝策略。这是当前任务队列和线程池都满了时所采取的应对策略，默认是AbordPolicy，表示无法处理新任务，并抛出RejectedExecutionException异常。

**拒绝策略有四种：**

- `AbordPolicy`:无法处理新任务，并抛出RejectedExecutionException异常。
- `CallerRunsPolicy`:用调用者所在的线程来处理任务。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。
- `DiscardPolicy`：不能执行的任务，并将该任务删除。
- `DiscardOldestPolicy`:丢弃队列最近的任务，并执行当前的任务。

**线程池分为四个类型：**

- `CachedThreadPool`：闲置线程超时会释放，没有闲置线程的情况下，每次都会创建新的线程。

  ```java
      public static ExecutorService newCachedThreadPool() {
          return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                        60L, TimeUnit.SECONDS,
                                        new SynchronousQueue<Runnable>());
      }
  ```

  - 线程数量不定，只有非核心线程，最大线程数`任意大`：传入核心线程数量的参数为0，最大线程数为Integer.MAX_VALUE；
  - 有新任务时使用`空闲线程`执行，没有空闲线程则创建新的线程来处理。
  - 该线程池的每个空闲线程都有超时机制，时常为60s（参数：60L, TimeUnit.SECONDS），空闲超过60s则回收空闲线程。
  - 适合执行大量的耗时较少的任务，当所有线程闲置`超过60s`都会被停止，所以这时几乎不占用系统资源。

- `FixedThreadPool`：线程池只能存放指定数量的线程池，线程不会释放，可重复利用。

  ```java
      public static ExecutorService newFixedThreadPool(int nThreads) {
          return new ThreadPoolExecutor(nThreads, nThreads,
                                        0L, TimeUnit.MILLISECONDS,
                                        new LinkedBlockingQueue<Runnable>());
      }
  ```

  - 线程`数量固定`且都是核心线程：核心线程数量和最大线程数量都是nThreads；

  - 都是核心线程且不会被回收，快速相应外界请求；

  - 没有超时机制，任务队列也没有大小限制；

  - 新任务使用`核心线程`处理，如果没有空闲的核心线程，则排队等待执行。

- `SingleThreadExecutor`：单线程的线程池。

  ```java
      public static ExecutorService newSingleThreadExecutor() {
          return new FinalizableDelegatedExecutorService
              (new ThreadPoolExecutor(1, 1,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>()));
      }
  ```

  - 只有`一个核心线程`，所有任务在同一个线程按顺序执行。
  - 所有的外界任务统一到一个线程中，所以不需要处理线程同步的问题。

- `ScheduledThreadPool`：可定时和重复执行的线程池。

  ```java
      private static final long DEFAULT_KEEPALIVE_MILLIS = 10L;
  
      public ScheduledThreadPoolExecutor(int corePoolSize) {
          super(corePoolSize, Integer.MAX_VALUE,
                DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                new DelayedWorkQueue());
      }
  ```

  - 核心线程`数量固定`，非核心线程数量`无限制`；
  - 非核心线程闲置超过10s会被回收；
  - 主要用于执行定时任务和具有固定周期的重复任务；

**执行任务流程：**

- 如果线程池中的线程数量未达到`核心线程的数量`，会直接启动一个核心线程来执行任务。
- 如果线程池中的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到`任务队列`中排队等待执行。
- 如果任务队列无法插入新任务，说明任务队列已满，如果未达到规定的最大线程数量，则启动一个`非核心线程`来执行任务。
- 如果线程数量超过规定的最大值，则执行`拒绝策略`-RejectedExecutionHandler。

#### 2. 与新建一个线程相比，线程池的特点？

1. 节省开销： 线程池中的线程可以重复利用。
2. 速度快：任务来了就能开始，省去创建线程的时间。
3. 线程可控：线程数量可空和任务可控。
4. 功能强大：可以定时和重复执行任务。

#### 3. 线程池的工作流程？



![线程池工作流程](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a7092b526a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

简而言之：

1. 任务来了，优先考虑核心线程。
2. 核心线程满了，进入阻塞队列。
3. 阻塞队列满了，考虑非核心线程（图上好像少了这个过程）。
4. 非核心线程满了，再触发拒绝任务。

#### 4. 为什么有newSingleThread?



### 三、锁

#### 1. 死锁触发的四大条件？

- 互斥锁

- 请求与保持

- 不可剥夺

- 循环的请求与等待

#### 2. synchronized关键字的使用？synchronized的参数放入对象和Class有什么区别？

synchronized 修饰 static 方法、普通方法、类、方法块区别？

`synchronized`关键字的用法：

- 修饰方法
- 修饰代码块：需要自己提供锁对象，锁对象包括对象本身、对象的Class和其他对象。

放入对象和Class的区别是：

1. 锁住的对象不同：成员方法锁住的实例对象，静态方法锁住的是Class。
2. 访问控制不同：如果锁住的是实例，只会针对同一个对象方法进行同步访问，多线程访问同一个对象的synchronized代码块是串行的，访问不同对象是并行的。如果锁住的是类，多线程访问的不管是同一对象还是不同对象的synchronized代码块是都是串行的。

#### 3. synchronized的（底层）原理？

任何一个对象都有一个`monitor`与之相关联，JVM基于进入和退出`mointor`对象来实现代码块同步和方法同步，两者实现细节不同：

- 代码块同步：在编译字节码的时候，代码块起始的地方插入`monitorenter` 指令，异常和代码块结束处插入`monitorexit`指令，线程在执行`monitorenter`指令的时候尝试获取`monitor`对象的所有权，获取不到的情况下就是阻塞
- 方法同步：`synchronized`方法在`method_info`结构有`AAC_synchronized`标记，线程在执行的时候获取对应的锁，从而实现同步方法

#### 4. synchronized和Lock的区别？

主要区别：

1. `synchronized`是Java中的关键字，是Java的内置实现；`Lock`是Java中的接口。
2. `synchronized`遇到异常会释放锁；`Lock`需要在发生异常的时候调用成员方法`Lock#unlock()`方法。
3. `synchronized`是不可以中断的，`Lock`可中断。
4. `synchronized`不能去尝试获得锁，没有获得锁就会被阻塞； `Lock`可以去尝试获得锁，如果未获得可以尝试处理其他逻辑。
5. `synchronized`多线程效率不如`Lock`，不过Java在1.6以后已经对`synchronized`进行大量的优化，所以性能上来讲，其实差不了多少。

#### 5. 悲观锁和乐观锁的举例？以及它们的相关实现？

悲观锁和乐观锁的概念：

- 悲观锁：悲观锁会认为，修改共享数据的时候其他线程也会修改数据，因此只在不会受到其他线程干扰的情况下执行。这样会导致其他有需要锁的线程挂起，等到持有锁的线程释放锁
- 乐观锁：每次不加锁，每次直接修改共享数据假设其他线程不会修改，如果发生冲突就直接重试，直到成功为止

举例：

- `悲观锁`：典型的悲观锁是独占锁，有`synchronized`、`ReentrantLock`。
- `乐观锁`：典型的乐观锁是CAS，实现CAS的`atomic`为代表的一系列类

#### 6. CAS是什么？底层原理？

`CAS`全称Compare And Set，核心的三个元素是：内存位置、预期原值和新值，执行CAS的时候，会将内存位置的值与预期原值进行比较，如果一致，就将原值更新为新值，否则就不更新。 底层原理：是借助CPU底层指令`cmpxchg`实现原子操作。

#### 7. 平常有用到什么锁，synchronized底层原理是什么

#### 8. synchronized是公平锁还是非公平锁,ReteranLock是公平锁吗？是怎么实现的

#### 9. JMM可见性，原子性，有序性，synchronized可以保证什么？

#### 10. 构造一个出现死锁的情况

#### 11. synchronized 的 4 种状态

> - [不可不说的Java“锁”事](https://links.jianshu.com/go?to=https%3A%2F%2Ftech.meituan.com%2F2018%2F11%2F15%2Fjava-lock.html)
> - 访问 synchronized 修饰static方法、synchronized(this|object) 是否会冲突受干扰

#### 12. 访问 synchronized 修饰 static 方法、synchronized (类.class) 是否会冲突受干扰

> [synchronized 详解](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F594a24defe88c2006aa01f1c%23heading-5)

**Synchronized修饰非静态方法**，实际上是对调用该方法的对象加锁，俗称“对象锁”。也就是锁住的是这个对象，即this。如果同一个对象在两个线程分别访问对象的两个同步方法，就会产生互斥，这就是对象锁，一个对象一次只能进入一个操作。

**Synchronized修饰静态方法**，实际上是对该类对象加锁，俗称“类锁”。也就是锁住的是这个类，即xx.class。如果一个对象在两个线程中分别调用一个静态同步方法和一个非静态同步方法，由于静态方法会收到类锁限制，但是非静态方法会收到对象限制，所以两个方法并不是同一个对象锁，因此不会排斥。



### 五、ReentranLock

#### 1.synchronized跟ReentranLock有什么区别？

#### 2.synchronized与ReentranLock发生异常的场景.



### 六、线程间通信

#### 1. notify和notifyAll方法的区别？

`notify`随机唤醒一个线程，`notifyAll`唤醒所有等待的线程，让他们竞争锁。

#### 2. wait/notify和Condition类实现的等待通知有什么区别？

`synchronized`与`wait/notify`结合的等待通知只有一个条件，而Condition类可以实现多个条件等待。

#### 3. 多线程间的有序性、可见性和原子性是什么意思？

- `原子性`：执行一个或者多个操作的时候，要么全部执行，要么都不执行，并且中间过程中不会被打断。Java中的原子性可以通过独占锁和CAS去保证
- `可见性`：指多线程访问同一个变量的时候，一个线程修改了变量的值，其他线程能够立刻看得到修改的值。锁和`volatile`能够保证可见性
- `有序性`：程序执行的顺序按照代码先后的顺序执行。锁和`volatile`能够保证有序性

#### 4. happens-before原则有哪些？

Java内存模型具有一些先天的有序性，它通常叫做happens-before原则。

如果两个操作的先后顺序不能通过happens-before原则推倒出来，那就不能保证它们的先后执行顺序，虚拟机就可以随意打乱执行指令。`happens-before`原则有：

1. 程序次序规则：单线程程序的执行结果得和看上去代码执行的结果要一致。
2. 锁定规则：一个锁的`lock`操作一定发生在上一个`unlock`操作之后。
3. volatile规则：对`volatile`变量的写操作一定先行于后面对这个变量的对操作。
4. 传递规则：A发生在B前面，B发生在C前面，那么A一定发生在C前面。
5. 线程启动规则：线程的`start`方法先行发生于线程中的每个动作。
6. 线程中断规则：对线程的`interrupt`操作先行发生于中断线程的检测代码。
7. 线程终结原则：线程中所有的操作都先行发生于线程的终止检测。
8. 对象终止原则：一个对象的初始化先行发生于他的`finalize()`方法的执行。

前四条规则比较重要。

### 七、Volatile 

#### 1. volatile 的作用和原理

**可见性** 如果对声明了`volatile`的变量进行写操作的时候，JVM会向处理器发送一条`Lock`前缀的指令，将这个变量所在缓存行的数据写入到系统内存。

多处理器的环境下，其他处理器的缓存还是旧的，为了保证各个处理器一致，会通过嗅探在总线上传播的数据来检测自己的数据是否过期，如果过期，会强制重新将系统内存的数据读取到处理器缓存。

**有序性** `Lock`前缀的指令相当于一个内存栅栏，它确保指令排序的时候，不会把后面的指令拍到内存栅栏的前面，也不会把前面的指令排到内存栅栏的后面。

#### 2. 一个 int 变量用 volatile 修饰，多线程去操作 i++，是否线程安全？如何保证 i++ 线程安全？AtomicInteger 的底层实现原理？

使用 AtomicInteger 可以使 i++ 线程安全

#### 3. 说说双重校验锁，以及volatile的作用

先回顾下双重校验锁的原型，也就是单例模式的实现：

```java
public class Singleton {
    private volatile static Singleton mSingleton;
    private Singleton() {
    }

    public Singleton getInstance() {
        if (null == mSingleton) {
            synchronized (Singleton.class) {
                if (null == mSingleton) {
                    mSingleton = new Singleton();
                }
            }
        }
        return mSingleton;
	}
}
```
有几个疑问需要解决：

为什么要加锁？为什么不直接给getInstance方法加锁？为什么需要双重判断是否为空？为什么还要加volatile修饰变量？接下来一一解答：

- 如果不加锁的话，是线程不安全的，也就是有可能多个线程同时访问getInstance方法会得到两个实例化的对象。

- 如果给getInstance方法加锁，就每次访问mSingleton都需要加锁，增加了性能开销

- 第一次判空是为了判断是否已经实例化，如果已经实例化就直接返回变量，不需要加锁了。第二次判空是因为走到加锁这一步，如果线程A已经实例化，等B获得锁，进入的时候其实对象已经实例化完成了，如果不二次判空就会再次实例化。

- 加volatile是为了禁止指令重排。指令重排指的是在程序运行过程中，并不是完全按照代码顺序执行的，会考虑到性能等原因，将不影响结果的指令顺序有可能进行调换。所以初始化的顺序本来是这三步：

  1）分配内存空间 2）初始化对象 3）将对象指向分配的空间

  如果进行了指令重排，由于不影响结果，所以2和3有可能被调换。所以就变成了：

  1）分配内存空间 2）将对象指向分配的空间 3）初始化对象

  就有可能会导致，假如线程A中已经进行到第二步，线程B进入第二次判空的时候，判断mSingleton不为空，就直接返回了，但是实际此时mSingleton还没有初始化。

#### 4. synchronized和volatile的区别

- `volatile`本质是在告诉jvm当前变量在寄存器中的值是不确定的,需要从主存中读取,`synchronized`则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住.
- `volatile`仅能使用在变量级别,`synchronized`则可以使用在变量,方法.
- `volatile`仅能实现变量的修改可见性,而`synchronized`则可以保证变量的修改可见性和原子性.
- `volatile`不会造成线程的阻塞,而`synchronized`可能会造成线程的阻塞.
- 当一个域的值依赖于它之前的值时，`volatile`就无法工作了，如n=n+1,n++等，也就是不保证原子性。
- 使用`volatile`而不是`synchronized`的唯一安全的情况是类中只有一个可变的域。





### 八、阻塞队列

#### 1. 通常的阻塞队列有哪几种，特点是什么？

- `ArrayBlockQueue`：基于数组实现的有界的FIFO(先进先出)阻塞队列。
- `LinkedBlockQueue`：基于链表实现的无界的FIFO(先进先出)阻塞队列。
- `SynchronousQueue`：内部没有任何缓存的阻塞队列。
- `PriorityBlockingQueue`：具有优先级的无限阻塞队列。

#### 2. ConcurrentHashMap的原理

数据结构的实现跟HashMap一样，不做介绍。

JDK 1.8之前采用的是分段锁，核心类是一个`Segment`，`Segment`继承了`ReentrantLock`，每个`Segment对象`管理若干个桶，多个线程访问同一个元素的时候只能去竞争获取锁。

JDK 1.8采用了`CAS + synchronized`，插入键值对的时候如果当前桶中没有Node节点，使用CAS方式进行更新，如果有Node节点，则使用synchronized的方式进行更新。

### 九、协程，纤程(Java Kotlin)

#### 1.说说你对协程的理解

在我看来，协程和线程一样都是用来解决`并发任务（异步任务）`的方案。
所以协程和线程是属于一个层级的概念，但是对于`kotlin`中的协程，又与广义的协程有所不同。
kotlin中的协程其实是对线程的一种`封装`，或者说是一种线程框架，为了让异步任务更好更方便使用。

#### 2. 说下协程具体的使用

比如在一个异步任务需要回调到主线程的情况，普通线程需要通过`handler`切换线程然后进行UI更新等，一旦多个任务需要`顺序调用`，那更是很不方便，比如以下情况：

```kotlin
//客户端顺序进行三次网络异步请求，并用最终结果更新UI
thread{
    iotask1(parameter) { value1 ->
        iotask1(value1) { value2 ->
            iotask1(value2) { value3 ->
                runOnUiThread{
                    updateUI(value3) 
                }      
        } 
    }              
}
}
```

简直是`魔鬼调用`，如果不止3次，而是5次，6次，那还得了。。

而用协程就能很好解决这个问题：

```kotlin
//并发请求
GlobalScope.launch(Dispatchers.Main) {
    //三次请求并发进行
    val value1 = async { request1(parameter1) }
    val value2 = async { request2(parameter2) }
    val value3 = async { request3(parameter3) }
    //所有结果全部返回后更新UI
    updateUI(value1.await(), value2.await(), value3.await())
}

//切换到io线程
suspend fun request1(parameter : Parameter){withContext(Dispatcher.IO){}}
suspend fun request2(parameter : Parameter){withContext(Dispatcher.IO){}}
suspend fun request3(parameter : Parameter){withContext(Dispatcher.IO){}}
```

就像是同一个线程中顺序执行的效果一样，再比如我要按顺序执行一次异步任务，然后完成后更新UI，一共三个异步任务。
如果正常写应该怎么写？

```kotlin
thread{
    iotask1() { value1 ->
        runOnUiThread{
            updateUI1(value1) 
            iotask2() { value2 ->
            runOnUiThread{
                updateUI2(value2) 
                iotask3() { value3 ->
                runOnUiThread{
                    updateUI3(value3) 
                } 
                }   
            }      
            }   

        }
    }
}
```

晕了晕了，不就是一次异步任务，一次UI更新吗。怎么这么麻烦，来，用协程看看怎么写：

```kotlin
    GlobalScope.launch (Dispatchers.Main) {
        ioTask1()
        ioTask1()
        ioTask1()
        updateUI1()
        updateUI2()
        updateUI3()
    }

    suspend fun ioTask1(){
        withContext(Dispatchers.IO){}
    }
    suspend fun ioTask2(){
        withContext(Dispatchers.IO){}
    }
    suspend fun ioTask3(){
        withContext(Dispatchers.IO){}
    }

    fun updateUI1(){
    }
    fun updateUI2(){
    }
    fun updateUI3(){
    }
```

#### 3.协程怎么取消

- 取消`协程作用域`将取消它的所有子协程。

```kotlin
// 协程作用域 scope
val job1 = scope.launch { … }
val job2 = scope.launch { … }
scope.cancel()
```

- 取消`子协程`

```kotlin
// 协程作用域 scope
val job1 = scope.launch { … }
val job2 = scope.launch { … }
job1.cancel()
```

但是调用了`cancel`并不代表协程内的工作会马上停止，他并不会组织代码运行。
 比如上述的`job1`，正常情况处于`active`状态，调用了`cancel`方法后，协程会变成`Cancelling`状态，工作完成之后会变成`Cancelled` 状态，所以可以通过判断协程的状态来停止工作。

Jetpack 中定义的协程作用域`（viewModelScope 和 lifecycleScope）`可以帮助你自动取消任务，下次再详细说明，其他情况就需要自行进行绑定和取消了。

#### 5. 你了解协程吗？协程有什么作用？可以完全取代rxjava吗？

#### 6. 讲一个协程的scope与context，协程的+号代表什么



### 十、其他

#### 1. 源码中有哪里用到了AtomicInt

#### 2. AQS了解吗？

#### 3. 管道了解吗？





## JVM
### 一、Java内存模型

#### 1. Jvm内存区域是如何划分的？

内存区域划分：

- `程序计数器`：当前线程的字节码执行位置的指示器，线程私有。
- `Java虚拟机栈`：描述的Java方法执行的内存模型，每个方法在执行的同时会创建一个栈帧，存储着局部变量、操作数栈、动态链接和方法出口等，线程私有。
- `本地方法栈`：本地方法执行的内存模型，线程私有。
- `Java堆`：所有对象实例分配的区域。
- `方法区`：所有已经被虚拟机加载的类的信息、常量、静态变量和即时编辑器编译后的代码数据。

#### 2. Jvm内存模型是怎么样的？

1. Java规定所有变量的内存都需要存储在主内存。

2. 每个线程都有自己的工作内存，线程中使用的所有变量以及对变量的操作都基于工作内存，工作内存中的所有变量都从主内存读取过来的。

3. 不同线程间的工作内存无法进行直接交流，必须通过主内存完成。

   ![Java内存模型](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a70e9f7985?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

   主内存和工作内存之间的交互协议，即变量如何从主内存传递到工作内存、工作内存如何将变量传递到主内存，Java内存模型定义了8种操作来完成，并且每一种操作都是原子的，不可再分的。

|   类型   | 说明                                                         |
| :------: | :----------------------------------------------------------- |
|  `lock`  | 作用于主内存的变量，把一个变量标识一个线程独占的状态         |
| `unlock` | 作用于主内存的变量，把一个处于锁定状态的变量释放出来         |
|  `read`  | 把一个变量从主内存传输到工作内存，以便随后的`load`使用       |
|  `load`  | 把`read`操作读取的变量存储到工作内存的变量副本中             |
|  `use`   | 把工作内存中的变量的值传递给执行引擎，每当虚拟机执行到一个需要使用变量的字节码指令的时候都会执行这个操作 |
| `assign` | 把一个从执行引擎中接收到的变量赋值给工作内存中的变量，每当虚拟机遇到赋值的字节码指令都会执行这个操作 |
| `store`  | 把工作内存中的一个变量的值传递给主内存，以便以后的`write`使用 |
| `write`  | 把`store`传递过来的工作内存中的变量写入到主内存中的变量      |

#### 3. String s1 = "abc"和String s2 = new String("abc")的区别，生成对象的情况

1. 指向方法区：`"abc"`是常量，所以它会在方法区中分配内存，如果方法区已经给`"abc"`分配过内存，则s1会直接指向这块内存区域。
2. 指向Java堆：`new String("abc")`是重新生成了一个Java实例，它会在Java堆中分配一块内存。

所以s1和s2的内存地址肯定不一样，但是内容一样。

#### 4. 说说Java的内存分区

### 二、类加载器

#### 1. 类加载的过程？

类加载的过程可以分为：

1. `加载`：将类的全限定名转化为二进制流，再将二进制流转化为方法区中的类型信息，从而生成一个Class对象。
2. `验证`：对类的验证，包括格式、字节码、属性等。
3. `准备`：为类变量分配内存并设置初始值。
4. `解析`：将常量池的符号引用转化为直接引用。
5. `初始化`：执行类中定义的Java程序代码，包括类变量的赋值动作和构造函数的赋值。
6. `使用`
7. `卸载`

只有加载、验证、准备、初始化和卸载的这个五个阶段的顺序是确定的。

[**Java 类的加载过程 - 三个主要步骤：加载、链接、初始化：**](https://links.jianshu.com/go?to=https%3A%2F%2Fapp.yinxiang.com%2FHome.action%23n%3D1f028e40-c11f-4f3f-ac22-bdebe8c71011%26s%3Ds38%26b%3Ded4692de-7e6e-4e29-ae3a-d4d01138be52%26ses%3D4%26sh%3D1%26sds%3D5%26)

**（1）加载 -** 将字节码数据从不同的数据源读取到 JVM 中，并映射为 JVM 认可的数据结构 (Class 对象)

- 由于类加载器的代理机制，**启动类加载过程**的类加载器和真正**完成类加载工作**的类加载器，有可能不同；
- 启动类的加载过程通过调用loadClass()来实现，称为初始加载器 (initiating loader)；而完成类的加载工作通过调用defineClass()来实现，称为类的定义加载器 (defining loader)。在 Java 虚拟机判断两个类是否相同的时候，使用的是类的定义加载器；
- loadClass() 抛出的是  java.lang.ClassNotFoundException 异常，而 defineClass() 抛出的是 java.lang.NoClassDefFoundError 异常；
- 类加载器在成功加载某个类之后，会把得到的 java.lang.Class 类的实例缓存起来。下次再请求加载该类的时候，类加载器会直接使用缓存的类的实例，而不会尝试再次加载 (即 loadClass()不会被重复调用)

**（2）链接 -** 将原始的类定义信息平滑地转化入 JVM 运行的过程中

- 验证：核验字节信息是符合 Java 虚拟机规范；
- 准备：创建类或接口中的静态变量并初始化，侧重分配所需要的内存空间（与初始化阶段区分开）；
- 解析：替换常量池中的符号引用为直接引用，类、接口、方法和字段等各个方面的解析等

**（3）初始化 -** 真正执行类初始化的代码逻辑，包括静态字段赋值的动作，以及类中静态初始化块内的逻辑。编译器在编译阶段就会把这部分逻辑整理好，父类型的初始化逻辑优先于当前类型的逻辑

#### 2. 类加载的机制，以及为什么要这样设计？双亲委派模型？

类加载的机制是双亲委派模型。大部分Java程序需要使用的类加载器包括：

- `启动类加载器`：由C++语言实现，负责加载Java中的核心类。
- `扩展类加载器`：负责加载Java扩展的核心类之外的类。
- `应用程序类加载器`：负责加载用户类路径上指定的类库。
- `自定义类加载器`: 用户自定义

双亲委派模型如下：

![双亲委派模型](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a726174d8e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

双亲委派模型要求出了顶层的启动类加载器之外，其他的类加载器都有自己的父加载器，通过组合实现。

双亲委派模型的工作流程： 当一个类加载的任务来临的时候，先交给父类加载器完成，父类加载器交给父父类加载器完成，知道传递给启动类加载器，如果完成不了的情况下，再依次往下传递类加载的任务。

这样设计的原因： 双亲委派模型能够保证Java程序的稳定运行，不同层次的类加载器具有不同优先级，所有的对象的父类Object，无论哪一个类加载器加载，最后都会交给启动类加载器，保证安全。

举例：

- 当Application ClassLoader 收到一个类加载请求时，他首先不会自己去尝试加载这个类，而是将这个请求委派给父类加载器Extension ClassLoader去完成。
- 当Extension ClassLoader收到一个类加载请求时，他首先也不会自己去尝试加载这个类，而是将请求委派给父类加载器Bootstrap ClassLoader去完成。
- 如果Bootstrap ClassLoader加载失败(在<JAVA_HOME>\lib中未找到所需类)，就会让Extension ClassLoader尝试加载。
- 如果Extension ClassLoader也加载失败，就会使用Application ClassLoader加载。
- 如果Application ClassLoader也加载失败，就会使用自定义加载器去尝试加载。
- 如果均加载失败，就会抛出ClassNotFoundException异常。

这么设计的原因是为了防止危险代码的植入，比如String类，如果在AppClassLoader就直接被加载，就相当于会被篡改了，所以都要经过老大，也就是BootstrapClassLoader进行检查，已经加载过的类就不需要再去加载了。



#### 3. 属性先加载还是方法先加载

#### 4. PathClassLoader与DexClassLoader有什么区别

#### 5. class文件的组成？常量池里面有什么内容？

#### 6. 自动装箱发生在什么时候？编译期还是运行期

#### 7.java和字节码有什么区别？

#### 8. Java 虚拟机类加载器分类，类加载器的代理机制有什么好处？

（1）类加载器分类

启动类加载器：加载 Java 的核心库，是用原生代码来实现的，并不继承自 java.lang.ClassLoader；

扩展类加载器：加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类；

系统/应用类加载器：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它；

注：类加载器树状组织结构，除了引导类加载器之外，所有的类加载器都有一个父类加载器。类加载器 Java 类如同其它的 Java 类一样，也是要由类加载器来加载的。

（2）类加载器的代理机制

原理：类加载器在尝试自己去查找某个类的字节代码并定义它时，会先代理给其父类加载器，由父类加载器先去尝试加载这个类，依次类推；

作用：代理模式是为了保证 Java 核心库的类型安全。对于Java 核心库的类的加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的。

> 传送门：[深入探讨 Java 类加载器](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fjava%2Fj-lo-classloader%2F)

#### 9. Java 虚拟机是如何判定两个 Java 类是相同的？

Java 虚拟机不仅要看类的全名是否相同，还要看加载此类的类加载器 (defining loader) 是否一样。即便是同样的字节代码，被不同的类加载器加载之后所得到的类，也是不同的；

不同的类加载器为相同名称的类创建了额外的名称空间。相同名称的类可以并存在 Java 虚拟机中，只需要用不同的类加载器来加载它们即可。不同类加载器加载的类之间是不兼容的，这就相当于在 Java 虚拟机内部创建了一个个相互隔离的 Java 类空间。







### 三、垃圾回收

#### 1. 如何判断对象可回收？

判断一个对象可以回收通常采用的算法是引用几算法和可达性算法。由于互相引用导致的计数不好判断，Java采用的可达性算法。

可达性算法的思路是：通过一些列被成为GC Roots的对象作为起始点，自上往下从这些起点往下搜索，搜索所有走过的路径称为引用链，如果一个对象没有跟任何引用链相关联的时候，则证明该对象不可用，所以这些对象就会被判定为可以回收。

可以被当作GC Roots的对象包括：

- Java虚拟机栈中的引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法中JNI引用的对象

[Java 垃圾收集的原理：](https://links.jianshu.com/go?to=https%3A%2F%2Fapp.yinxiang.com%2FHome.action%23n%3D6add1e9f-f832-438c-ae3d-6f0f1fca8108%26s%3Ds38%26b%3Ded4692de-7e6e-4e29-ae3a-d4d01138be52%26ses%3D4%26sh%3D1%26sds%3D5%26)

- 自动垃圾收集的前提是清楚哪些内存可以被释放，主要有两个方面，最主要部分就是**对象实例**，存储在堆上的；另一个是**方法区中的元数据**等信息，例如类型不再使用，卸载该 Java 类比较合理；
- **对象实例收集**主要是两种基本算法，[引用计数](https://links.jianshu.com/go?to=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0)和可达性分析，**Java 选择的可达性分析**。**JVM 会把虚拟机栈和本地方法栈中**正在引用的对象**、**静态属性引用的对象**和**常量**，作为 GC Roots。

#### 2. GC的常用算法？

- `标记 - 清除`：首先标记出需要回收的对象，标记完成后统一回收所有被标记的对象。容易产生碎片空间。
- `复制算法`：它将可用的内存分为两块，每次只用其中的一块，当需要内存回收的时候，将存活的对象复制到另一块内存，然后将当前已经使用的内存一次性回收掉。需要浪费一半的内存。
- `标记 - 整理`：让存活的对象向一端移动，之后清除边界外的内存。
- `分代搜集`：根据对象存活的周期，Java堆会被分为新生代和老年代，根据不同年代的特性，选择合适的GC收集算法。

#### 3. Minar GC和Full GC的区别？

- `Minar GC`：频率高、针对新生代。
- `Full GC`：频率低、发生在老年代、通常会伴随一次Minar GC和速度慢。

#### 4. 说一下四种引用以及他们的区别？

- `强引用`：强引用还在，垃圾搜集器就不会回收被引用的对象。
- `软引用`：对于软引用关联的对象，在系统发生内存溢出异常之前，将会把这些对象列进回收范围进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。
- `弱引用`：被若引用关联的对象只能存活到下一次GC之前。
- `虚引用`：为对象设置虚引用的目的仅仅是为了GC之前收到一个系统通知。



#### 5. 垃圾回收机制与jvm结构

#### 6. 圾回收的GCRoot是什么？

#### 