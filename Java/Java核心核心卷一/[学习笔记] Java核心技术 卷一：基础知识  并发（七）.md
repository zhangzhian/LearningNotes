# [学习笔记] Java核心技术 卷一：基础知识  并发（七）

---

 - 当线程的 run方法执行方法体中最后一条语句后， 并经由执行 return 语句返冋时， 或者出现了在方法中没有捕获的异常时，线程将终止。 在 Java 的早期版本中， 还有一个 stop方法， 其他线程可以调用它终止线程。但是， 这个方法现在已经被弃用了。
 - 没有可以强制线程终止的方法。然而， interrupt 方法可以用来请求终止线程。当对一个线程调用interrupt方法时，线程的中断状态将被置位。
 - 线程可以有如下 6 种状态：
     - New (新创建）
     - Runnable (可运行）
     - Blocked (被阻塞）
     - Waiting (等待）
     - Timed waiting (计时等待）
     - Terminated (被终止）
 - 线程状态
![线程状态](http://img.blog.csdn.net/20171121092955379?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 - 可以用 setUncaughtExceptionHandler 方法为任何线程安装一个处理器。也可以用 Thread类的静态方法setDefaultUncaughtExceptionHandler 为所有线程安装一个默认的处理器。替换处理器可以使用日志API发送未捕获异常的报告到日志文件
 - 有两种机制防止代码块受并发访问的干扰。Java语言提供一个 synchronized 关键字达到这一目的，并且 Java SE 5.0 引入了 ReentrantLock 类。
 - 调用 signalAll 不会立即激活一个等待线程。它仅仅解除等待线程的阻塞， 以便这些线程可以在当前线程退出同步方法之后， 通过竞争实现对对象的访问。另一个方法 signal,则是随机解除等待集中某个线程的阻塞状态。
 - 用 Java 的术语来讲， 监视器具有如下特性：
 - 监视器是只包含私有域的类。
 - 每个监视器类的对象有一个相关的锁。
 - 使用该锁对所有的方法进行加锁。换句话说，如果客户端调用 obj.meth0d(), 那 么 obj对象的锁是在方法调用开始时自动获得， 并且当方法返回时自动释放该锁。因为所有的域是私有的，这样的安排可以确保一个线程在对对象操作时， 没有其他线程能访问该域。
 - 该锁可以有任意多个相关条件。
 - volatile 关键字为实例域的同步访问提供了一种免锁机制。如果声明一个域为 volatile ,那么编译器和虚拟机就知道该域是可能被另一个线程并发更新的。Volatile 变量不能提供原子性
 - java.util.concurrent 包提供了映射、 有序集和队列的高效实现：ConcurrentHashMap、ConcurrentSkipListMap 、 ConcurrentSkipListSet 和 ConcurrentLinkedQueue。这些集合使用复杂的算法，通过允许并发地访问数据结构的不同部分来使竞争极小化。
 - Runnable 封装一个异步运行的任务，可以把它想象成为一个没有参数和返回值的异步方法。Callable 与 Runnable 类似 ，但是有返回值。Callable 接口是一个参数化的类型， 只有一个方法 call。
 - Future 保存异步计算的结果。可以启动一个计算，将 Future 对象交给某个线程 ，然后忘掉它。 Future对象的所有者在结果计算好之后就可以获得它。FutureTask 包装器是一种非常便利的机制， 可将 Callable转换成 Future 和 Runnable, 它同时实现二者的接口。
 - 执行器 （Executor) 类有许多静态工厂方法用来构建线程池
 ![这里写图片描述](http://img.blog.csdn.net/20171121113907443?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 - 在使用连接池时应该做的事：
 - 调用 Executors 类中静态的方法 newCachedThreadPool 或 newFixedThreadPool。
 - 调用 submit 提交 Runnable 或 Callable 对象。
 - 如果想要取消一个任务， 或如果提交 Callable 对象， 那就要保存好返回的 Future
对象。
 - 当不再提交任何任务时，调用 shutdown。
 - 当用完一个线程池的时候， 调用 shutdown 。该方法启动该池的关闭序列。被关闭的执行器不再接受新的任务。当所有任务都完成以后，线程池中的线程死亡。另一种方法是调用shutdownNow。该池取消尚未开始的所有任务并试图中断正在运行的线程。
 - java.util.concurrent 包包含了几个能帮助人们管理相互合作的线程集的类。这些机制具有为线程之间的共用集结点模式（common rendezvous patterns) 提供的“ 预置功能”(canned functionality )。如果有一个相互合作的线程集满足这些行为模式之一， 那么应该直接重用合适的库类而不要试图提供手工的锁与条件的集合。
 ![同 步 器](http://img.blog.csdn.net/20171121140913566?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


