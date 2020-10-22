# Java并发

## 一、Java创建线程的三种方式

### 1. 继承Thread类创建线程类

（1）定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run()方法称为执行体。

（2）创建Thread子类的实例，即创建了线程对象。

（3）调用线程对象的start()方法来启动该线程。

```java
public class FirstThreadTest extends Thread {
    int i = 0;

    //重写run方法，run方法的方法体就是现场执行体
    public void run() {
        for (; i < 100; i++) {
            System.out.println(getName() + "  " + i);

        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "  : " + i);
            if (i == 20) {
                new FirstThreadTest().start();
                new FirstThreadTest().start();
            }
        }
    }

}
```

上述代码中Thread.currentThread()方法返回当前正在执行的线程对象。GetName()方法返回调用该方法的线程的名字。

### 2. 通过Runnable接口创建线程类

（1）定义runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体。

（2）创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。

（3）调用线程对象的start()方法来启动该线程。

```java
public class RunnableThreadTest implements Runnable {
    private int i;

    public void run() {
        for (i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 20) {
                RunnableThreadTest rtt = new RunnableThreadTest();
                new Thread(rtt, "新线程1").start();
                new Thread(rtt, "新线程2").start();
            }
        }

    }

}
```

### 3. 通过Callable和Future创建线程

（1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。

（2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call()方法的返回值。

（3）使用FutureTask对象作为Thread对象的target创建并启动新线程。

（4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值，调用get()方法会阻塞线程。

```java
public class CallableThreadTest implements Callable<Integer> {

    public static void main(String[] args) {
        CallableThreadTest ctt = new CallableThreadTest();
        FutureTask<Integer> ft = new FutureTask<>(ctt);
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " 的循环变量i的值" + i);
            if (i == 20) {
                new Thread(ft, "有返回值的线程").start();
            }
        }
        try {
            System.out.println("子线程的返回值：" + ft.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    @Override
    public Integer call() throws Exception {
        int i = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        return i;
    }
}
```

### 4. 创建线程的三种方式的对比

**采用实现Runnable、Callable接口的方式创见多线程时，优势是：**

线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。

在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。

**劣势是：**

编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

**使用继承Thread类的方式创建多线程时优势是：**

编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。

**劣势是：**

线程类已经继承了Thread类，所以不能再继承其他父类。



## 二、Java线程池

### 1. 概述

在我们的开发中经常会使用到多线程。例如在Android中，由于主线程的诸多限制，像网络请求等一些耗时的操作我们必须在子线程中运行。我们往往会通过new Thread来开启一个子线程，待子线程操作完成以后通过Handler切换到主线程中运行。这么以来我们无法管理我们所创建的子线程，并且无限制的创建子线程，它们相互之间竞争，很有可能由于占用过多资源而导致死机或者OOM。所以在Java中为我们提供了线程池来管理我们所创建的线程。

**线程池的优势**

①降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗；

②提高系统响应速度，当有任务到达时，无需等待新线程的创建便能立即执行；

③方便线程并发数的管控，线程若是无限制的创建，不仅会额外消耗大量系统资源，更是占用过多资源而阻塞系统或oom等状况，从而降低系统的稳定性。线程池能有效管控线程，统一分配、调优，提供资源使用率；

④更强大的功能，线程池提供了定时、定期以及可控线程数等功能的线程池，使用方便简单。

### 2. ThreadPoolExecutor

我们可以通过ThreadPoolExecutor来创建一个线程池。

```java
ExecutorService service = new ThreadPoolExecutor(....);
```

下面我们就来看一下ThreadPoolExecutor中的一个构造方法。

```Java
 public ThreadPoolExecutor(int corePoolSize,
 	int maximumPoolSize,
 	long keepAliveTime,
 	TimeUnit unit,
 	BlockingQueue<Runnable> workQueue,
 	ThreadFactory threadFactory,
 	RejectedExecutionHandler handler) 
```

**ThreadPoolExecutor参数含义**

**1. corePoolSize**

线程池中的核心线程数，默认情况下，核心线程一直存活在线程池中，即便他们在线程池中处于闲置状态。除非我们将ThreadPoolExecutor的**allowCoreThreadTimeOut**属性设为true的时候，这时候处于闲置的核心线程在等待新任务到来时会有超时策略，这个超时时间由keepAliveTime来指定。一旦超过所设置的超时时间，闲置的核心线程就会被终止。

**2. maximumPoolSize**

线程池中所容纳的最大线程数，如果活动的线程达到这个数值以后，后续的新任务将会被阻塞。包含核心线程数+非核心线程数。

**3. keepAliveTime**

非核心线程闲置时的超时时长，对于非核心线程，闲置时间超过这个时间，非核心线程就会被回收。只有对ThreadPoolExecutor的allowCoreThreadTimeOut属性设为true的时候，这个超时时间才会对核心线程产生效果。

**4. unit**

用于指定keepAliveTime参数的时间单位。他是一个枚举，可以使用的单位有天（TimeUnit.DAYS），小时（TimeUnit.HOURS），分钟（TimeUnit.MINUTES），毫秒(TimeUnit.MILLISECONDS)，微秒(TimeUnit.MICROSECONDS, 千分之一毫秒)和毫微秒(TimeUnit.NANOSECONDS, 千分之一微秒);

**5. workQueue**

线程池中保存等待执行的任务的阻塞队列。通过线程池中的execute方法提交的Runable对象都会存储在该队列中。我们可以选择下面几个阻塞队列。

| 阻塞队列              | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| ArrayBlockingQueue    | 基于数组实现的有界的阻塞队列,该队列按照FIFO（先进先出）原则对队列中的元素进行排序。 |
| LinkedBlockingQueue   | 基于链表实现的阻塞队列，该队列按照FIFO（先进先出）原则对队列中的元素进行排序。 |
| SynchronousQueue      | 内部没有任何容量的阻塞队列。在它内部没有任何的缓存空间。对于SynchronousQueue中的数据元素只有当我们试着取走的时候才可能存在。 |
| PriorityBlockingQueue | 具有优先级的无限阻塞队列。                                   |

我们还能够通过实现BlockingQueue接口来自定义我们所需要的阻塞队列。

**6. threadFactory**

线程工厂，为线程池提供新线程的创建。ThreadFactory是一个接口，里面只有一个newThread方法。 默认为DefaultThreadFactory类。

**7. handler**

是RejectedExecutionHandler对象，而RejectedExecutionHandler是一个接口，里面只有一个rejectedExecution方法。**当任务队列已满并且线程池中的活动线程已经达到所限定的最大值或者是无法成功执行任务，这时候ThreadPoolExecutor会调用RejectedExecutionHandler中的rejectedExecution方法。在ThreadPoolExecutor中有四个内部类实现了RejectedExecutionHandler接口。在线程池中它默认是AbortPolicy，在无法处理新任务时抛出RejectedExecutionException异常**。

下面是在ThreadPoolExecutor中提供的四个可选值。

| 可选值              | 说明                                       |
| ------------------- | ------------------------------------------ |
| CallerRunsPolicy    | 只用调用者所在线程来运行任务。             |
| AbortPolicy         | 直接抛出RejectedExecutionException异常。   |
| DiscardPolicy       | 丢弃掉该任务，不进行处理。                 |
| DiscardOldestPolicy | 丢弃队列里最近的一个任务，并执行当前任务。 |

我们也可以通过实现RejectedExecutionHandler接口来自定义我们自己的handler。如记录日志或持久化不能处理的任务。

**ThreadPoolExecutor的使用**

```java
ExecutorService service = new ThreadPoolExecutor(5, 10, 10, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
```

对于ThreadPoolExecutor有多个构造方法，对于上面的构造方法中的其他参数都采用默认值。可以通过execute和submit两种方式来向线程池提交一个任务。 **execute** 当我们使用execute来提交任务时，由于execute方法没有返回值，所以说我们也就无法判定任务是否被线程池执行成功。

```java
service.execute(new Runnable() {
	public void run() {
		System.out.println("execute方式");
	}
});
```

**submit**

当我们使用submit来提交任务时,它会返回一个future,我们就可以通过这个future来判断任务是否执行成功，还可以通过future的get方法来获取返回值。如果子线程任务没有完成，get方法会阻塞住直到任务完成，而使用`get(long timeout, TimeUnit unit)`方法则会阻塞一段时间后立即返回，这时候有可能任务并没有执行完。

```java
Future<Integer> future = service.submit(new Callable<Integer>() {

	@Override
	public Integer call() throws Exception {
		System.out.println("submit方式");
		return 2;
	}
});
try {
	Integer number = future.get();
} catch (ExecutionException e) {
	// TODO Auto-generated catch block
	e.printStackTrace();
}
```

**线程池关闭**

调用线程池的`shutdown()`或`shutdownNow()`方法来关闭线程池

shutdown原理：将线程池状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

shutdownNow原理：将线程池的状态设置成STOP状态，然后中断所有任务(包括正在执行的)的线程，并返回等待执行任务的列表。

**中断采用interrupt方法，所以无法响应中断的任务可能永远无法终止。** 但调用上述的两个关闭之一，isShutdown()方法返回值为true，当所有任务都已关闭，表示线程池关闭完成，则isTerminated()方法返回值为true。当需要立刻中断所有的线程，不一定需要执行完任务，可直接调用shutdownNow()方法。

### 3. 线程池执行流程

[![img](https://camo.githubusercontent.com/b84bf3f22d9a96e43f24cde542b9586afeb26d4f/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d633035373731393832616432376638362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/b84bf3f22d9a96e43f24cde542b9586afeb26d4f/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d633035373731393832616432376638362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430) ①如果在线程池中的线程数量没有达到核心的线程数量，这时候就回启动一个核心线程来执行任务。

②如果线程池中的线程数量已经超过核心线程数，这时候任务就会被插入到任务队列中排队等待执行。

③由于任务队列已满，无法将任务插入到任务队列中。这个时候如果线程池中的线程数量没有达到线程池所设定的最大值，那么这时候就会立即启动一个非核心线程来执行任务。

④如果线程池中的数量达到了所规定的最大值，那么就会拒绝执行此任务，这时候就会调用RejectedExecutionHandler中的rejectedExecution方法来通知调用者。

### 4. 四种线程池类

Java中四种具有不同功能常见的线程池。他们都是直接或者间接配置ThreadPoolExecutor来实现他们各自的功能。这四种线程池分别是newFixedThreadPool,newCachedThreadPool,newScheduledThreadPool和newSingleThreadExecutor。这四个线程池可以通过Executors类获取。

#### newFixedThreadPool

通过Executors中的newFixedThreadPool方法来创建，该线程池是一种线程数量固定的线程池。

```java
ExecutorService service = Executors.newFixedThreadPool(4);
```

在这个线程池中 **所容纳最大的线程数就是我们设置的核心线程数。** 如果线程池的线程处于空闲状态的话，它们并不会被回收，除非是这个线程池被关闭。如果所有的线程都处于活动状态的话，新任务就会处于等待状态，直到有线程空闲出来。

由于newFixedThreadPool只有核心线程，并且这些线程都不会被回收，也就是 **它能够更快速的响应外界请求** 。从下面的newFixedThreadPool方法的实现可以看出，newFixedThreadPool只有核心线程，并且不存在超时机制，采用LinkedBlockingQueue，所以对于任务队列的大小也是没有限制的。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
	return new ThreadPoolExecutor(nThreads, nThreads,
		0L, TimeUnit.MILLISECONDS,
		new LinkedBlockingQueue<Runnable>());
}
```

#### newCachedThreadPool

通过Executors中的newCachedThreadPool方法来创建。

```java
public static ExecutorService newCachedThreadPool() {
	return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
		60L, TimeUnit.SECONDS,
		new SynchronousQueue<Runnable>());
}
```

通过s上面的newCachedThreadPool方法在这里我们可以看出它的 **核心线程数为0，** 线程池的最大线程数Integer.MAX_VALUE。而Integer.MAX_VALUE是一个很大的数，也差不多可以说 **这个线程池中的最大线程数可以任意大。**

**当线程池中的线程都处于活动状态的时候，线程池就会创建一个新的线程来处理任务。该线程池中的线程超时时长为60秒，所以当线程处于闲置状态超过60秒的时候便会被回收。** 这也就意味着若是整个线程池的线程都处于闲置状态超过60秒以后，在newCachedThreadPool线程池中是不存在任何线程的，所以这时候它几乎不占用任何的系统资源。

对于newCachedThreadPool他的任务队列采用的是SynchronousQueue，上面说到在SynchronousQueue内部没有任何容量的阻塞队列。SynchronousQueue内部相当于一个空集合，我们无法将一个任务插入到SynchronousQueue中。所以说在线程池中如果现有线程无法接收任务,将会创建新的线程来执行任务。

#### newScheduledThreadPool

通过Executors中的newScheduledThreadPool方法来创建。

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

它的核心线程数是固定的，对于非核心线程几乎可以说是没有限制的，并且当非核心线程处于限制状态的时候就会立即被回收。

创建一个可定时执行或周期执行任务的线程池：

```java
ScheduledExecutorService service = Executors.newScheduledThreadPool(4);
service.schedule(new Runnable() {
	public void run() {
		System.out.println(Thread.currentThread().getName()+"延迟三秒执行");
	}
}, 3, TimeUnit.SECONDS);
service.scheduleAtFixedRate(new Runnable() {
	public void run() {
		System.out.println(Thread.currentThread().getName()+"延迟三秒后每隔2秒执行");
	}
}, 3, 2, TimeUnit.SECONDS);
```

输出结果：

> pool-1-thread-2延迟三秒后每隔2秒执行
> pool-1-thread-1延迟三秒执行
> pool-1-thread-1延迟三秒后每隔2秒执行
> pool-1-thread-2延迟三秒后每隔2秒执行
> pool-1-thread-2延迟三秒后每隔2秒执行

`schedule(Runnable command, long delay, TimeUnit unit)`：延迟一定时间后执行Runnable任务；

`schedule(Callable callable, long delay, TimeUnit unit)`：延迟一定时间后执行Callable任务；

`scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)`：延迟一定时间后，以间隔period时间的频率周期性地执行任务；

`scheduleWithFixedDelay(Runnable command, long initialDelay, long delay,TimeUnit unit)`:与scheduleAtFixedRate()方法很类似，但是不同的是scheduleWithFixedDelay()方法的周期时间间隔是以上一个任务执行结束到下一个任务开始执行的间隔，而scheduleAtFixedRate()方法的周期时间间隔是以上一个任务开始执行到下一个任务开始执行的间隔，也就是这一些任务系列的触发时间都是可预知的。

ScheduledExecutorService功能强大，对于定时执行的任务，建议多采用该方法。

#### newSingleThreadExecutor

通过Executors中的newSingleThreadExecutor方法来创建，**在这个线程池中只有一个核心线程**，对于任务队列没有大小限制，也就意味着**这一个任务处于活动状态时，其他任务都会在任务队列中排队等候依次执行**。

newSingleThreadExecutor将所有的外界任务统一到一个线程中支持，所以在这个任务执行之间我们不需要处理线程同步的问题。

```java
public static ExecutorService newSingleThreadExecutor() {
	return new FinalizableDelegatedExecutorService
	(new ThreadPoolExecutor(1, 1,
		0L, TimeUnit.MILLISECONDS,
		new LinkedBlockingQueue<Runnable>()));
}
```

### 5. 线程池的使用技巧

需要针对具体情况而具体处理，不同的任务类别应采用不同规模的线程池，任务类别可划分为CPU密集型任务、IO密集型任务和混合型任务。(N代表CPU个数)

| 任务类别      | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| CPU密集型任务 | 线程池中线程个数应尽量少，如配置N+1个线程的线程池。          |
| IO密集型任务  | 由于IO操作速度远低于CPU速度，那么在运行这类任务时，CPU绝大多数时间处于空闲状态，那么线程池可以配置尽量多些的线程，以提高CPU利用率，如2*N。 |
| 混合型任务    | 可以拆分为CPU密集型任务和IO密集型任务，当这两类任务执行时间相差无几时，通过拆分再执行的吞吐率高于串行执行的吞吐率，但若这两类任务执行时间有数据级的差距，那么没有拆分的意义 |



## 三、死锁

### 1. 死锁产生的条件

一般来说，要出现死锁问题需要满足以下条件：

1. 互斥条件：一个资源每次只能被一个线程使用。
2. 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺。
4. 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。

在JAVA编程中，有3种典型的死锁类型：
静态的锁顺序死锁，动态的锁顺序死锁，协作对象之间发生的死锁。

### 2. 静态的锁顺序死锁

a和b两个方法都需要获得A锁和B锁。一个线程执行a方法且已经获得了A锁，在等待B锁；另一个线程执行了b方法且已经获得了B锁，在等待A锁。这种状态，就是发生了静态的锁顺序死锁。

```java
//可能发生静态锁顺序死锁的代码
class StaticLockOrderDeadLock {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void a() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("function a");
            }
        }
    }

    public void b() {
        synchronized (lockB) {
            synchronized (lockA) {
                System.out.println("function b");
            }
        }
    }
}
```

**解决静态的锁顺序死锁的方法就是：所有需要多个锁的线程，都要以相同的顺序来获得锁。**

```java
//正确的代码
class StaticLockOrderDeadLock {
    private final Object lockA = new Object();
    private final Object lockB = new Object();

    public void a() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("function a");
            }
        }
    }

    public void b() {
        synchronized (lockA) {
            synchronized (lockB) {
                System.out.println("function b");
            }
        }
    }
}
```

### 3. 动态的锁顺序死锁：

动态的锁顺序死锁是指两个线程调用同一个方法时，传入的参数颠倒造成的死锁。如下代码，一个线程调用了transferMoney方法并传入参数accountA,accountB；另一个线程调用了transferMoney方法并传入参数accountB,accountA。此时就可能发生在静态的锁顺序死锁中存在的问题，即：第一个线程获得了accountA锁并等待accountB锁，第二个线程获得了accountB锁并等待accountA锁。

```java
//可能发生动态锁顺序死锁的代码
class DynamicLockOrderDeadLock {
    public void transefMoney(Account fromAccount, Account toAccount, Double amount) {
        synchronized (fromAccount) {
            synchronized (toAccount) {
                //...
                fromAccount.minus(amount);
                toAccount.add(amount);
                //...
            }
        }
    }
}
```

**动态的锁顺序死锁解决方案如下：使用System.identifyHashCode来定义锁的顺序。确保所有的线程都以相同的顺序获得锁。**

```java
//正确的代码
class DynamicLockOrderDeadLock {
    private final Object myLock = new Object();

    public void transefMoney(final Account fromAccount, final Account toAccount, final Double amount) {
        class Helper {
            public void transfer() {
                //...
                fromAccount.minus(amount);
                toAccount.add(amount);
                //...
            }
        }
        int fromHash = System.identityHashCode(fromAccount);
        int toHash = System.identityHashCode(toAccount);

        if (fromHash < toHash) {
            synchronized (fromAccount) {
                synchronized (toAccount) {
                    new Helper().transfer();
                }
            }
        } else if (fromHash > toHash) {
            synchronized (toAccount) {
                synchronized (fromAccount) {
                    new Helper().transfer();
                }
            }
        } else {
            synchronized (myLock) {
                synchronized (fromAccount) {
                    synchronized (toAccount) {
                        new Helper().transfer();
                    }
                }
            }
        }

    }
}
```

### 4. 协作对象之间发生的死锁：

有时，死锁并不会那么明显，比如两个相互协作的类之间的死锁，比如下面的代码：一个线程调用了Taxi对象的setLocation方法，另一个线程调用了Dispatcher对象的getImage方法。此时可能会发生，第一个线程持有Taxi对象锁并等待Dispatcher对象锁，另一个线程持有Dispatcher对象锁并等待Taxi对象锁。

```java
//可能发生死锁
class Taxi {
    private Point location, destination;
    private final Dispatcher dispatcher;

    public Taxi(Dispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }

    public synchronized Point getLocation() {
        return location;
    }

    public synchronized void setLocation(Point location) {
        this.location = location;
        if (location.equals(destination))
            dispatcher.notifyAvailable(this);//外部调用方法，可能等待Dispatcher对象锁
    }
}

class Dispatcher {
    private final Set<Taxi> taxis;
    private final Set<Taxi> availableTaxis;

    public Dispatcher() {
        taxis = new HashSet<Taxi>();
        availableTaxis = new HashSet<Taxi>();
    }

    public synchronized void notifyAvailable(Taxi taxi) {
        availableTaxis.add(taxi);
    }

    public synchronized Image getImage() {
        Image image = new Image();
        for (Taxi t : taxis)
            image.drawMarker(t.getLocation());//外部调用方法，可能等待Taxi对象锁
        return image;
    }
}
```

上面的代码中， **我们在持有锁的情况下调用了外部的方法，这是非常危险的（可能发生死锁）。为了避免这种危险的情况发生，** 我们使用开放调用。如果调用某个外部方法时不需要持有锁，我们称之为开放调用。

**解决协作对象之间发生的死锁：需要使用开放调用，即避免在持有锁的情况下调用外部的方法。**

```java
//正确的代码
class Taxi {
    private Point location, destination;
    private final Dispatcher dispatcher;

    public Taxi(Dispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }

    public synchronized Point getLocation() {
        return location;
    }

    public void setLocation(Point location) {
        boolean flag = false;
        synchronized (this) {
            this.location = location;
            flag = location.equals(destination);
        }
        if (flag)
            dispatcher.notifyAvailable(this);//使用开放调用
    }
}

class Dispatcher {
    private final Set<Taxi> taxis;
    private final Set<Taxi> availableTaxis;

    public Dispatcher() {
        taxis = new HashSet<Taxi>();
        availableTaxis = new HashSet<Taxi>();
    }

    public synchronized void notifyAvailable(Taxi taxi) {
        availableTaxis.add(taxi);
    }

    public Image getImage() {
        Set<Taxi> copy;
        synchronized (this) {
            copy = new HashSet<Taxi>(taxis);
        }
        Image image = new Image();
        for (Taxi t : copy)
            image.drawMarker(t.getLocation());//使用开放调用
        return image;
    }
}
```

### 5. 总结

综上，是常见的3种死锁的类型。即：静态的锁顺序死锁，动态的锁顺序死锁，协作对象之间的死锁。在写代码时，要确保线程在获取多个锁时采用一致的顺序。同时，要避免在持有锁的情况下调用外部方法。



## 四、Synchronized/ReentrantLock

### 1. 线程同步问题的产生及解决方案

**问题的产生：**

Java允许多线程并发控制，当多个线程同时操作一个可共享的资源变量时（如数据的增删改查），将会导致数据不准确，相互之间产生冲突。

如下例：假设有一个卖票系统，一共有100张票，有4个窗口同时卖。

```java
public class Ticket implements Runnable {
    // 当前拥有的票数
    private int num = 100;

    public void run() {
        while (true) {
            if (num > 0) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                }
                // 输出卖票信息
                System.out.println(Thread.currentThread().getName() + ".....sale...." + num--);
            }
        }
    }
}
public class Nothing {

    public static void main(String[] args) {
        Ticket t = new Ticket();//创建一个线程任务对象。
        //创建4个线程同时卖票
        Thread t1 = new Thread(t);
        Thread t2 = new Thread(t);
        Thread t3 = new Thread(t);
        Thread t4 = new Thread(t);
        //启动线程
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}
```

输出部分结果：

> Thread-1.....sale....2
> Thread-0.....sale....3
> Thread-2.....sale....1
> Thread-0.....sale....0
> Thread-1.....sale....0
> Thread-3.....sale....1

显然上述结果是不合理的，对于同一张票进行了多次售出。这就是多线程情况下，出现了数据“**脏读**”情况。即多个线程访问余票num时，当一个线程获得余票的数量，要在此基础上进行-1的操作之前，其他线程可能已经卖出多张票，导致获得的num不是最新的，然后-1后更新的数据就会有误。这就需要线程同步的实现了。

**问题的解决：**

因此加入同步锁以避免在该线程没有完成操作之前，被其他线程的调用，从而保证了该变量的唯一性和准确性。

一共有两种锁，来实现线程同步问题，分别是：`synchronized`和`ReentrantLock`。下面我们就带着上述问题，看看这两种锁是如何解决的。

### 2. synchronized关键字

**synchronized简介**

- synchronized实现同步的基础：Java中每个对象都可以作为锁。当线程试图访问同步代码时，必须先获得**对象锁**，退出或抛出异常时必须释放锁。

- Synchronzied实现同步的表现形式分为：**代码块同步** 和 **方法同步**。

**synchronized原理**

JVM基于进入和退出`Monitor`对象来实现 **代码块同步** 和 **方法同步** ，两者实现细节不同。

**代码块同步：** 在编译后通过将`monitorenter`指令插入到同步代码块的开始处，将`monitorexit`指令插入到方法结束处和异常处，通过反编译字节码可以观察到。任何一个对象都有一个`monitor`与之关联，线程执行`monitorenter`指令时，会尝试获取对象对应的`monitor`的所有权，即尝试获得对象的锁。

**方法同步：** synchronized方法在`method_info结构`有`ACC_synchronized`标记，线程执行时会识别该标记，获取对应的锁，实现方法同步。

两者虽然实现细节不同，但本质上都是对一个对象的监视器（monitor）的获取。**任意一个对象都拥有自己的监视器**，当同步代码块或同步方法时，执行方法的线程必须先获得该对象的监视器才能进入同步块或同步方法，没有获取到监视器的线程将会被阻塞，并进入同步队列，状态变为`BLOCKED`。当成功获取监视器的线程释放了锁后，会唤醒阻塞在同步队列的线程，使其重新尝试对监视器的获取。

**对象、监视器、同步队列和执行线程间的关系如下图：**

[![img](https://camo.githubusercontent.com/57e44b9c59d9441a163e7d4b53d241fb589b3d1e/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d633338383132643866343538313064632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/57e44b9c59d9441a163e7d4b53d241fb589b3d1e/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d633338383132643866343538313064632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

**synchronized的使用场景**

**①方法同步**

```
public synchronized void method1
```

锁住的是该对象,类的其中一个实例，当该对象(仅仅是这一个对象)在不同线程中执行这个同步方法时，线程之间会形成互斥。达到同步效果，但如果不同线程同时对该类的不同对象执行这个同步方法时，则线程之间不会形成互斥，因为他们拥有的是不同的锁。

**②代码块同步**

```
synchronized(this){ //TODO }
```

描述同①

**③方法同步**

```
public synchronized static void method3
```

锁住的是该类，当所有该类的对象(多个对象)在不同线程中调用这个static同步方法时，线程之间会形成互斥，达到同步效果。

**④代码块同步**

```
synchronized(Test.class){ //TODO}
```

同③

**⑤代码块同步**

```
synchronized(o) {}
```

这里面的o可以是一个任何Object对象或数组，并不一定是它本身对象或者类，谁拥有o这个锁，谁就能够操作该块程序代码。

**解决线程同步的实例**

针对上述方法，具体的解决方式如下：

```java
public class Ticket implements Runnable {
    // 当前拥有的票数
    private int num = 100;

    public void run() {
        while (true) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
            }
            synchronized (this) {
                // 输出卖票信息
                if (num > 0) {
System.out.println(Thread.currentThread().getName() + ".....sale...." + num--);
                }

            }
        }
    }
}
```

输出部分结果：

> Thread-2.....sale....10
> Thread-1.....sale....9
> Thread-3.....sale....8
> Thread-0.....sale....7
> Thread-2.....sale....6
> Thread-1.....sale....5
> Thread-2.....sale....4
> Thread-1.....sale....3
> Thread-3.....sale....2
> Thread-0.....sale....1

可以看出实现了线程同步。同时改了一下逻辑，在进入到同步代码块时，先判断现在是否有没有票，然后再买票，防止出现没票还要售出的情况。通过同步代码块实现了线程同步，其他方法也一样可以实现该效果。

### 3. ReentrantLock锁

ReentrantLock，一个可重入的互斥锁，它具有与使用synchronized方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。（重入锁后面介绍）

**Lock接口**

Lock，锁对象。在Java中锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源（但有的锁可以允许多个线程并发访问共享资源，比如读写锁，后面我们会分析）。在Lock接口出现之前，Java程序是靠`synchronized`关键字（后面分析）实现锁功能的，而JAVA SE5.0之后并发包中新增了`Lock`接口用来实现锁的功能，它提供了与`synchronized`关键字类似的同步功能，只是在使用时需要显式地获取和释放锁，缺点就是缺少像`synchronized`那样隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性，可中断的获取锁以及超时获取锁等多种`synchronized`关键字所不具备的同步特性。

Lock接口的主要方法（还有两个方法比较复杂，暂不介绍）：

> **void lock():** 执行此方法时，如果锁处于空闲状态，当前线程将获取到锁。相反，如果锁已经被其他线程持有，将禁用当前线程，直到当前线程获取到锁。
> **boolean tryLock()：** 如果锁可用，则获取锁，并立即返回true，否则返回false. 该方法和lock()的区别在于，tryLock()只是"试图"获取锁，如果锁不可用，不会导致当前线程被禁用，当前线程仍然继续往下执行代码。而lock()方法则是一定要获取到锁，如果锁不可用，就一直等待，在未获得锁之前,当前线程并不继续向下执行. 通常采用如下的代码形式调用tryLock()方法：
> **void unlock()：** 执行此方法时，当前线程将释放持有的锁. 锁只能由持有者释放，如果线程并不持有锁，却执行该方法，可能导致异常的发生.
> **Condition newCondition()：** 条件对象，获取等待通知组件。该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的await()方法，而调用后，当前线程将缩放锁。

**ReentrantLock的使用**

关于ReentrantLock的使用很简单，只需要显示调用，获得同步锁，释放同步锁即可。

```java
ReentrantLock lock = new ReentrantLock(); //参数默认false，不公平锁
.....................
lock.lock(); //如果被其它资源锁定，会在此等待锁释放，达到暂停的效果
try {
    //操作
} finally {
    lock.unlock();  //释放锁
}
```

**解决线程同步的实例**

针对上述方法，具体的解决方式如下：

```java
public class Ticket implements Runnable {
    // 当前拥有的票数
    private int num = 100;
    ReentrantLock lock = new ReentrantLock();

    public void run() {
        while (true) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
            }

            lock.lock();
            // 输出卖票信息
            if (num > 0) {
                System.out.println(Thread.currentThread().getName() + ".....sale...." + num--);
            }
            lock.unlock();

        }
    }
}
```

### 4. 重入锁

当一个线程得到一个对象后，再次请求该对象锁时是可以再次得到该对象的锁的。

具体概念就是：自己可以再次获取自己的内部锁。

Java里面内置锁(synchronized)和Lock(ReentrantLock)都是可重入的。

```Java
public class SynchronizedTest {
    public void method1() {
        synchronized (SynchronizedTest.class) {
            System.out.println("方法1获得ReentrantTest的锁运行了");
            method2();
        }
    }
    public void method2() {
        synchronized (SynchronizedTest.class) {
            System.out.println("方法1里面调用的方法2重入锁,也正常运行了");
        }
    }
    public static void main(String[] args) {
        new SynchronizedTest().method1();
    }
}
```

上面便是synchronized的重入锁特性，即调用method1()方法时，已经获得了锁，此时内部调用method2()方法时，由于本身已经具有该锁，所以可以再次获取。

```java
public class ReentrantLockTest {
    private Lock lock = new ReentrantLock();
    public void method1() {
        lock.lock();
        try {
            System.out.println("方法1获得ReentrantLock锁运行了");
            method2();
        } finally {
            lock.unlock();
        }
    }
    public void method2() {
        lock.lock();
        try {
            System.out.println("方法1里面调用的方法2重入ReentrantLock锁,也正常运行了");
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) {
        new ReentrantLockTest().method1();
    }
}
```

上面便是ReentrantLock的重入锁特性，即调用method1()方法时，已经获得了锁，此时内部调用method2()方法时， **由于本身已经具有该锁，所以可以再次获取**。

### 5. 公平锁

CPU在调度线程的时候是在等待队列里随机挑选一个线程，由于这种随机性所以是无法保证线程**先到先得**的（synchronized控制的锁就是这种非公平锁）。但这样就会产生饥饿现象，即有些线程（优先级较低的线程）可能永远也无法获取CPU的执行权，优先级高的线程会不断的强制它的资源。那么如何解决饥饿问题呢，这就需要公平锁了。公平锁可以保证线程**按照时间的先后顺序**执行，避免饥饿现象的产生。但公平锁的效率比较低，因为要实现顺序执行，需要维护一个有序队列。

ReentrantLock便是一种公平锁，通过在构造方法中传入true就是公平锁，传入false，就是非公平锁。

```
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

以下是使用公平锁实现的效果：

```java
public class LockFairTest implements Runnable{
    //创建公平锁
    private static ReentrantLock lock=new ReentrantLock(true);
    public void run() {
        while(true){
            lock.lock();
            try{
                System.out.println(Thread.currentThread().getName()+"获得锁");
            }finally{
                lock.unlock();
            }
        }
    }
    public static void main(String[] args) {
        LockFairTest lft=new LockFairTest();
        Thread th1=new Thread(lft);
        Thread th2=new Thread(lft);
        th1.start();
        th2.start();
    }
}
```

输出结果：

> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁
> Thread-1获得锁
> Thread-0获得锁

这是截取的部分执行结果，分析结果可看出两个线程是交替执行的，几乎不会出现同一个线程连续执行多次。

### 6. synchronized和ReentrantLock的比较

**区别：**

1）Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；

2）synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁；

3）Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；

4）通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

5）Lock可以提高多个线程进行读操作的效率。

**总结：ReentrantLock相比synchronized，增加了一些高级的功能。但也有一定缺陷。**

在ReentrantLock类中定义了很多方法，比如：

```
isFair()      //判断锁是否是公平锁

isLocked()    //判断锁是否被任何线程获取了

isHeldByCurrentThread()   //判断锁是否被当前线程获取了

hasQueuedThreads()   //判断是否有线程在等待该锁
```

**两者在锁的相关概念上区别：**

**1)可中断锁**

顾名思义，就是可以响应中断的锁。

在Java中，**synchronized就不是可中断锁，而Lock是可中断锁**。如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

`lockInterruptibly()`的用法体现了Lock的可中断性。

**2)公平锁**

公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该锁（并不是绝对的，大体上是这种顺序），这种就是公平锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。ReentrantLock可以设置成公平锁。

**3)读写锁**

读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。

正因为有了读写锁，才使得多个线程之间的读操作可以并发进行，不需要同步，而写操作需要同步进行，提高了效率。

ReadWriteLock就是读写锁，它是一个接口，ReentrantReadWriteLock实现了这个接口。

可以通过readLock()获取读锁，通过writeLock()获取写锁。

**4)绑定多个条件**

一个ReentrantLock对象可以同时绑定多个Condition对象，而在synchronized中，锁对象的wait()和notify()或notifyAll()方法可以实现一个隐含的条件，如果要和多余一个条件关联的时候，就不得不额外地添加一个锁，而ReentrantLock则无须这么做，只需要多次调用new Condition()方法即可。

**性能比较**

在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而 **当竞争资源非常激烈时（即有大量线程同时竞争），此时ReentrantLock的性能要远远优于synchronized** 。所以说，在具体使用时要根据适当情况选择。

在JDK1.5中，synchronized是性能低效的。因为这是一个重量级操作，它对性能最大的影响是阻塞的是实现，挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作给系统的并发性带来了很大的压力。相比之下使用Java提供的ReentrankLock对象，性能更高一些。到了JDK1.6，发生了变化，对synchronize加入了很多优化措施，有自适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在JDK1.6上synchronize的性能并不比Lock差。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地，所以还是提倡在synchronized能实现需求的情况下，优先考虑使用synchronized来进行同步。

## 五、生产者/消费者模式

### 1. 线程间通信的两种方式

#### wait()/notify()

Object类中相关的方法有notify方法和wait方法。因为wait和notify方法定义在Object类中，因此会被所有的类所继承。这些方法都是**final**的，即它们都是不能被重写的，不能通过子类覆写去改变它们的行为。

**①wait()方法：** 让当前线程进入等待，并释放锁。

**②wait(long)方法：** 让当前线程进入等待，并释放锁，不过等待时间为long，超过这个时间没有对当前线程进行唤醒，将**自动唤醒**。

**③notify()方法：** 让当前线程通知那些处于等待状态的线程，当前线程执行完毕后释放锁，并从其他线程中唤醒其中一个继续执行。

**④notifyAll()方法：** 让当前线程通知那些处于等待状态的线程，当前线程执行完毕后释放锁，将唤醒所有等待状态的线程。

**wait()方法使用注意事项**

①当前的线程必须拥有当前对象的monitor，也即lock，就是锁，才能调用wait()方法，否则将抛出异常java.lang.IllegalMonitorStateException。

②线程调用wait()方法，释放它对锁的拥有权，然后等待另外的线程来通知它（通知的方式是notify()或者notifyAll()方法），这样它才能重新获得锁的拥有权和恢复执行。

③要确保调用wait()方法的时候拥有锁，即，wait()方法的调用必须放在synchronized方法或synchronized块中。

**notify()方法使用注意事项**

①如果多个线程在等待，它们中的一个将会选择被唤醒。这种选择是随意的，和具体实现有关。（线程等待一个对象的锁是由于调用了wait()方法）。

②被唤醒的线程是不能被执行的，需要等到当前线程放弃这个对象的锁，当前线程会在方法执行完毕后释放锁。

**wait()/notify()协作的两个注意事项**

①通知过早

如果通知过早，则会打乱程序的运行逻辑。

```java
public class MyRun {
    private String lock = new String("");
    public Runnable runnableA = new Runnable() {

        @Override
        public void run() {
            try {
                synchronized (lock) {
                    System.out.println("begin wait");
                    lock.wait();
                    System.out.println("end wait");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    };
    public Runnable runnableB = new Runnable() {
        @Override
        public void run() {
            synchronized (lock) {
                System.out.println("begin notify");
                lock.notify();
                System.out.println("end notify");
            }
        }
    };
}
```

两个方法，分别执行wait()/notify()方法。

```java
public static void main(String[] args) throws InterruptedException {
        MyRun run = new MyRun();
        Thread bThread = new Thread(run.runnableB);
        bThread.start();
        Thread.sleep(100);
        Thread aThread = new Thread(run.runnableA);
        aThread.start();
    }
```

如果notify()方法先执行，将导致wait()方法释放锁进入等待状态后，永远无法被唤醒，影响程序逻辑。应避免这种情况。

②等待wait的条件发生变化

在使用wait/notify模式时，还需要注意另外一种情况，也就是wait等待条件发生了变化，也容易造成程序逻辑的混乱。

**Add类，执行加法操作，然后通知Subtract类**

```java
public class Add {
    private String lock;

    public Add(String lock) {
        super();
        this.lock = lock;
    }
    public void add(){
        synchronized (lock) {
            ValueObject.list.add("anyThing");
            lock.notifyAll();
        }
    }
}
```

**Subtract类，执行减法操作，执行完后进入等待状态，等待Add类唤醒notify**

```java
public class Subtract {
    private String lock;

    public Subtract(String lock) {
        super();
        this.lock = lock;
    }
    public void subtract(){
        try {
            synchronized (lock) {
                if(ValueObject.list.size()==0){
                    System.out.println("wait begin ThreadName="+Thread.currentThread().getName());
                    lock.wait();
                    System.out.println("wait end ThreadName="+Thread.currentThread().getName());
                }
                ValueObject.list.remove(0);
                System.out.println("list size ="+ValueObject.list.size());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**线程ThreadAdd**

```java
public class ThreadAdd extends Thread{
    private Add pAdd;

    public ThreadAdd(Add pAdd) {
        super();
        this.pAdd = pAdd;
    }
    @Override
    public void run() {
        pAdd.add();
    }

}
```

**线程ThreadSubtract**

```java
public class ThreadSubtract extends Thread{
    private Subtract rSubtract;

    public ThreadSubtract(Subtract rSubtract) {
        super();
        this.rSubtract = rSubtract;
    }
    @Override
    public void run() {
        rSubtract.subtract();
    }

}
```

**先开启两个ThreadSubtract线程，由于list中没有元素，进入等待状态。再开启一个ThreadAdd线程，向list中增加一个元素，然后唤醒两个ThreadSubtract线程。**

```java
public static void main(String[] args) throws InterruptedException {
        String lock = new String("");
        Add add = new Add(lock);
        Subtract subtract = new Subtract(lock);
        ThreadSubtract subtractThread1 = new ThreadSubtract(subtract);
        subtractThread1.setName("subtractThread1");
        subtractThread1.start();
        ThreadSubtract subtractThread2 = new ThreadSubtract(subtract);
        subtractThread2.setName("subtractThread2");
        subtractThread2.start();
        Thread.sleep(1000);
        ThreadAdd addThread = new ThreadAdd(add);
        addThread.setName("addThread");
        addThread.start();
    }
```

输出结果

> wait begin ThreadName=subtractThread1
> wait begin ThreadName=subtractThread2
> wait end ThreadName=subtractThread2
> Exception in thread "subtractThread1" list size =0
> wait end ThreadName=subtractThread1
> java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
> at java.util.ArrayList.rangeCheck(Unknown Source)
> at java.util.ArrayList.remove(Unknown Source)
> at com.lvr.communication.Subtract.subtract(Subtract.java:18)
> at com.lvr.communication.ThreadSubtract.run(ThreadSubtract.java:12)

**当第二个ThreadSubtract线程执行减法操作时，抛出下标越界异常。**

**原因分析：一开始两个ThreadSubtract线程等待状态，当ThreadAdd线程添加一个元素并唤醒所有线程后，第一个ThreadSubtract线程接着原来的执行到的地点开始继续执行，删除一个元素并输出集合大小。同样，第二个ThreadSubtract线程也如此，可是此时集合中已经没有元素了，所以抛出异常。**

**解决办法：从等待状态被唤醒后，重新判断条件，看看是否扔需要进入等待状态，不需要进入再进行下一步操作。即把if()判断，改成while()。**

```java
public void subtract(){
        try {
            synchronized (lock) {
                while(ValueObject.list.size()==0){
                    System.out.println("wait begin ThreadName="+Thread.currentThread().getName());
                    lock.wait();
                    System.out.println("wait end ThreadName="+Thread.currentThread().getName());
                }
                ValueObject.list.remove(0);
                System.out.println("list size ="+ValueObject.list.size());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

这是线程间协作中经常出现的一种情况，需要避免。

#### Condition实现等待/通知

关键字synchronized与wait()和notify()/notifyAll()方法相结合可以实现等待/通知模式，类似ReentrantLock也可以实现同样的功能，但需要借助于Condition对象。

关于Condition实现等待/通知就不详细介绍了，可以完全类比wait()/notify()，基本使用和注意事项完全一致。
就只简单介绍下类比情况：

**condition.await()————>lock.wait()**

**condition.await(long time, TimeUnit unit)————>lock.wait(long timeout)**

**condition.signal()————>lock.notify()**

**condition.signaAll()————>lock.notifyAll()**

**特殊之处：synchronized相当于整个ReentrantLock对象只有一个单一的Condition对象情况。而一个ReentrantLock却可以拥有多个Condition对象，来实现通知部分线程。**

**具体实现方式：**
假设有两个Condition对象：ConditionA和ConditionB。那么由ConditionA.await()方法进入等待状态的线程，由ConditionA.signalAll()通知唤醒；由ConditionB.await()方法进入等待状态的线程，由ConditionB.signalAll()通知唤醒。篇幅有限，代码示例就不写了。

### 2. 生产者/消费者模式实现

#### 一生产与一消费

下面情形是一个生产者，一个消费者的模式。假设场景：一个String对象，其中生产者为其设置值，消费者拿走其中的值，不断的循环往复，实现生产者/消费者的情形。

**wait()/notify()实现**

生产者

```java
public class Product {
    private String lock;

    public Product(String lock) {
        super();
        this.lock = lock;
    }
    public void setValue(){
        try {
            synchronized (lock) {
                if(!StringObject.value.equals("")){
                    //有值，不生产
                    lock.wait();
                }
                String  value = System.currentTimeMillis()+""+System.nanoTime();
                System.out.println("set的值是："+value);
                StringObject.value = value;
                lock.notify();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

消费者

```java
public class Consumer {
    private String lock;

    public Consumer(String lock) {
        super();
        this.lock = lock;
    }
    public void getValue(){
        try {
            synchronized (lock) {
                if(StringObject.value.equals("")){
                    //没值，不进行消费
                    lock.wait();
                }
                System.out.println("get的值是："+StringObject.value);
                StringObject.value = "";
                lock.notify();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

生产者线程

```java
public class ThreadProduct extends Thread{
    private Product product;

    public ThreadProduct(Product product) {
        super();
        this.product = product;
    }
    @Override
    public void run() {
        //死循环，不断的生产
        while(true){
            product.setValue();
        }
    }

}
```

消费者线程

```java
public class ThreadConsumer extends Thread{
    private Consumer consumer;

    public ThreadConsumer(Consumer consumer) {
        super();
        this.consumer = consumer;
    }
    @Override
    public void run() {
        //死循环，不断的消费
        while(true){
            consumer.getValue();
        }
    }

}
```

开启生产者/消费者模式

```java
public class Test {

    public static void main(String[] args) throws InterruptedException {
        String lock = new String("");
        Product product = new Product(lock);
        Consumer consumer = new Consumer(lock);
        ThreadProduct pThread = new ThreadProduct(product);
        ThreadConsumer cThread = new ThreadConsumer(consumer);
        pThread.start();
        cThread.start();
    }

}
```

输出结果：

> set的值是：148827033184127168687409691
> get的值是：148827033184127168687409691
> set的值是：148827033184127168687449887
> get的值是：148827033184127168687449887
> set的值是：148827033184127168687475117
> get的值是：148827033184127168687475117

Condition方式实现类似。

#### 多生产与多消费

**特殊情况：** 按照上述一生产与一消费的情况，通过创建多个生产者和消费者线程，实现多生产与多消费的情况，将会出现“假死”。

**具体原因：** 多个生产者和消费者线程。当全部运行后，生产者线程生产数据后，可能唤醒的同类即生产者线程。此时可能会出现如下情况：所有生产者线程进入等待状态，然后消费者线程消费完数据后，再次唤醒的还是消费者线程，直至所有消费者线程都进入等待状态，此时将进入“假死”。

**解决方法：** 将notify()或signal()方法改为notifyAll()或signalAll()方法，这样就不怕因为唤醒同类而进入“假死”状态了。

**Condition方式实现** 生产者

```java
public class Product {
    private ReentrantLock lock;
    private Condition condition;

    public Product(ReentrantLock lock, Condition condition) {
        super();
        this.lock = lock;
        this.condition = condition;
    }

    public void setValue() {
        try {
            lock.lock();
            while (!StringObject.value.equals("")) {
                // 有值，不生产
                condition.await();
            }
            String value = System.currentTimeMillis() + "" + System.nanoTime();
            System.out.println("set的值是：" + value);
            StringObject.value = value;
            condition.signalAll();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }


    }
}
```

消费者

```java
public class Consumer {
    private ReentrantLock lock;
    private Condition condition;

    public Consumer(ReentrantLock lock,Condition condition) {
        super();
        this.lock = lock;
        this.condition = condition;
    }
    public void getValue(){
        try {
                lock.lock();
                while(StringObject.value.equals("")){
                    //没值，不进行消费
                    condition.await();
                }
                System.out.println("get的值是："+StringObject.value);
                StringObject.value = "";
                condition.signalAll();

        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
```

生产者线程和消费者线程与一生产一消费的模式相同。

开启多生产/多消费模式

```java
public static void main(String[] args) throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();
        Condition newCondition = lock.newCondition();
        Product product = new Product(lock,newCondition);
        Consumer consumer = new Consumer(lock,newCondition);
        for(int i=0;i<3;i++){
            ThreadProduct pThread = new ThreadProduct(product);
            ThreadConsumer cThread = new ThreadConsumer(consumer);
            pThread.start();
            cThread.start();
        }

    }
```

输出结果:

> set的值是：148827212374628960540784817
> get的值是：148827212374628960540784817
> set的值是：148827212374628960540810047
> get的值是：148827212374628960540810047

可见交替地进行get/set实现多生产/多消费模式。

**注意：相比一生产一消费的模式，改动了两处。①signal()-->signalAll()避免进入“假死”状态。②if()判断-->while()循环，重新判断条件，避免逻辑混乱。**

以上就是Java线程间通信的相关知识，以生产者/消费者模式为例，讲解线程间通信的使用以及注意事项。



## 六、volatile关键字

### 1. Java内存模型

想要理解`volatile`为什么能确保可见性，就要先理解Java中的内存模型是什么样的。

Java内存模型规定了**所有的变量都存储在主内存中**。**每条线程中还有自己的工作内存，线程的工作内存中保存了被该线程所使用到的变量（这些变量是从主内存中拷贝而来）**。**线程对变量的所有操作（读取，赋值）都必须在工作内存中进行。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成**。

[![img](https://camo.githubusercontent.com/fed046c512b637dd64c7f90d52b63ab435039f6c/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d386133643961316239346239376538332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/fed046c512b637dd64c7f90d52b63ab435039f6c/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d386133643961316239346239376538332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

基于此种内存模型，便产生了多线程编程中的数据“脏读”等问题。

举个简单的例子：在java中，执行下面这个语句：

```
i  = 10;
```

**执行线程必须先在自己的工作线程中对变量i所在的缓存行进行赋值操作，然后再写入主存当中。而不是直接将数值10写入主存当中。**

比如同时有2个线程执行这段代码，假如初始时i的值为10，那么我们希望两个线程执行完之后i的值变为12。但是事实会是这样吗？

**可能存在下面一种情况：初始时，两个线程分别读取i的值存入各自所在的工作内存当中，然后线程1进行加1操作，然后把i的最新值11写入到内存。此时线程2的工作内存当中i的值还是10，进行加1操作之后，i的值为11，然后线程2把i的值写入内存。**

最终结果i的值是11，而不是12。这就是著名的缓存一致性问题。通常称这种被多个线程访问的变量为共享变量。

那么如何确保共享变量在多线程访问时能够正确输出结果呢？

在解决这个问题之前，我们要先了解并发编程的三大概念：**原子性，有序性，可见性**。

### 2. 原子性

**定义**

原子性：即一个操作或者多个操作，要么全部执行，并且执行的过程不会被任何因素打断，要么就都不执行。

**实例**

一个很经典的例子就是银行账户转账问题：

比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。

试想一下，如果这2个操作不具备原子性，会造成什么样的后果。假如从账户A减去1000元之后，操作突然中止。这样就会导致账户A虽然减去了1000元，但是账户B没有收到这个转过来的1000元。

所以这2个操作必须要具备原子性才能保证不出现一些意外的问题。

同样地反映到并发编程中会出现什么结果呢？

举个最简单的例子，大家想一下假如为一个32位的变量赋值过程不具备原子性的话，会发生什么后果？

```
i = 9;
```

假若一个线程执行到这个语句时，我暂且假设为一个32位的变量赋值包括两个过程：为低16位赋值，为高16位赋值。

那么就可能发生一种情况：当将低16位数值写入之后，突然被中断，而此时又有一个线程去读取i的值，那么读取到的就是错误的数据。

**Java中的原子性**

在Java中，**对基本数据类型的变量的读取和赋值操作是原子性操作**，即这些操作是不可被中断的，要么执行，要么不执行。

上面一句话虽然看起来简单，但是理解起来并不是那么容易。看下面一个例子i：

请分析以下哪些操作是原子性操作：

```java
x = 10;        //语句1
y = x;         //语句2
x++;           //语句3
x = x + 1;     //语句4
```

咋一看，可能会说上面的4个语句中的操作都是原子性操作。其实只有语句1是原子性操作，其他三个语句都不是原子性操作。

语句1是直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中。

**语句2实际上包含2个操作，它先要去读取x的值，再将x的值写入工作内存**，虽然读取x的值以及将x的值写入工作内存，这2个操作都是原子性操作，但是合起来就不是原子性操作了。

同样的，**x++和 x = x+1包括3个操作：读取x的值，进行加1操作，写入新的值**。

所以上面4个语句只有语句1的操作具备原子性。

也就是说，**只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。**

从上面可以看出，Java内存模型只保证了基本读取和赋值是原子性操作，**如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。由于synchronized和Lock能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。**

### 3. 可见性

**定义**

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

**实例**

举个简单的例子，看下面这段代码：

```
//线程1执行的代码
int i = 0;
i = 10;

//线程2执行的代码
j = i;
```

由上面的分析可知，当线程1执行 i =10这句时，会先把i的初始值加载到工作内存中，然后赋值为10，那么在线程1的工作内存当中i的值变为10了，却没有立即写入到主存当中。

此时线程2执行 j = i，它会先去主存读取i的值并加载到线程2的工作内存当中，注意此时内存当中i的值还是0，那么就会使得j的值为0，而不是10.

这就是可见性问题，线程1对变量i修改了之后，线程2没有立即看到线程1修改的值。

**Java中的可见性**

对于可见性，Java提供了volatile关键字来保证可见性。

**当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。**

而普通的共享变量不能保证可见性，**因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。**

另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且**在释放锁之前会将对变量的修改刷新到主存当中**。因此可以保证可见性。

### 4. 有序性

**定义**

有序性：即程序执行的顺序按照代码的先后顺序执行。

**实例**

举个简单的例子，看下面这段代码：

```java
int i = 0;

boolean flag = false;

i = 1;                //语句1
flag = true;          //语句2
```

上面代码定义了一个int型变量，定义了一个boolean类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句1是在语句2前面的，那么JVM在真正执行这段代码的时候会保证语句1一定会在语句2前面执行吗？不一定，为什么呢？这里可能会发生指令重排序（Instruction Reorder）。

下面解释一下什么是指令重排序，**一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。**

比如上面的代码中，语句1和语句2谁先执行对最终的程序结果并没有影响，那么就有可能在执行过程中，语句2先执行而语句1后执行。

但是要注意，虽然处理器会对指令进行重排序，但是它会保证程序最终结果会和代码顺序执行结果相同，那么它靠什么保证的呢？再看下面一个例子：

```
int a = 10;    //语句1
int r = 2;    //语句2
a = a + 3;    //语句3
r = a*a;     //语句4
```

这段代码有4个语句，那么可能的一个执行顺序是： 　　

[![img](https://camo.githubusercontent.com/e8d120c10e2d3e5f7bf8d3de270c5abe949a91a3/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d646365666362313339306362336564382e6a70673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/e8d120c10e2d3e5f7bf8d3de270c5abe949a91a3/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d646365666362313339306362336564382e6a70673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430) 　 

那么可不可能是这个执行顺序呢： 语句2  语句1  语句4  语句3

**不可能，因为处理器在进行重排序时是会考虑指令之间的数据依赖性，如果一个指令Instruction 2必须用到Instruction 1的结果，那么处理器会保证Instruction 1会在Instruction 2之前执行。**

虽然重排序不会影响单个线程内程序执行的结果，但是多线程呢？下面看一个例子：

```java
//线程1:

context = loadContext();   //语句1
inited = true;             //语句2

 //线程2:
while(!inited ){
   sleep()
}
doSomethingwithconfig(context);
```

上面代码中，由于语句1和语句2没有数据依赖性，因此可能会被重排序。假如发生了重排序，在线程1执行过程中先执行语句2，而此是线程2会以为初始化工作已经完成，那么就会跳出while循环，去执行doSomethingwithconfig(context)方法，而此时context并没有被初始化，就会导致程序出错。

从上面可以看出，**指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。**

也就是说，**要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。**

**Java中的有序性**

在Java内存模型中，允许编译器和处理器对指令进行重排序，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

在Java里面，可以通过volatile关键字来保证一定的“有序性”。另外可以通过synchronized和Lock来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。

另外，Java内存模型具备一些先天的“有序性”，**即不需要通过任何手段就能够得到保证的有序性，这个通常也称为 happens-before 原则。如果两个操作的执行次序无法从happens-before原则推导出来，那么它们就不能保证它们的有序性，虚拟机可以随意地对它们进行重排序。**

**下面就来具体介绍下happens-before原则（先行发生原则）：**

①程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作

②锁定规则：一个unLock操作先行发生于后面对同一个锁的lock操作

③volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作

④传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C

⑤线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作

⑥线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生

⑦线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行

⑧对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始

这8条规则中，前4条规则是比较重要的，后4条规则都是显而易见的。

下面我们来解释一下前4条规则：

对于程序次序规则来说，就是一段程序代码的执行**在单个线程中看起来是有序的**。注意，虽然这条规则中提到“书写在前面的操作先行发生于书写在后面的操作”，这个应该是程序看起来执行的顺序是按照代码顺序执行的，**但是虚拟机可能会对程序代码进行指令重排序**。虽然进行重排序，但是最终执行的结果是与程序顺序执行的结果一致的，它只会对不存在数据依赖性的指令进行重排序。因此，**在单个线程中，程序执行看起来是有序执行的**，这一点要注意理解。事实上，**这个规则是用来保证程序在单线程中执行结果的正确性，但无法保证程序在多线程中执行的正确性。**

第二条规则也比较容易理解，也就是说无论在单线程中还是多线程中，**同一个锁如果处于被锁定的状态，那么必须先对锁进行了释放操作，后面才能继续进行lock操作。**

第三条规则是一条比较重要的规则。直观地解释就是，**如果一个线程先去写一个变量，然后一个线程去进行读取，那么写入操作肯定会先行发生于读操作。**

第四条规则实际上就是体现happens-before原则**具备传递性**。

### 5. 深入理解volatile关键字

#### volatile保证可见性

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

1）保证了**不同线程对这个变量进行操作时的可见性**，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

2）**禁止进行指令重排序。**

先看一段代码，假如线程1先执行，线程2后执行：

```java
//线程1
boolean stop = false;
while(!stop){
    doSomething();
}
 
//线程2
stop = true;
```

这段代码是很典型的一段代码，很多人在中断线程时可能都会采用这种标记办法。但是事实上，这段代码会完全运行正确么？即一定会将线程中断么？不一定，也许在大多数时候，这个代码能够把线程中断，但是也有可能会导致无法中断线程（虽然这个可能性很小，但是只要一旦发生这种情况就会造成死循环了）。

下面解释一下这段代码为何有可能导致无法中断线程。在前面已经解释过，每个线程在运行过程中都有自己的工作内存，那么线程1在运行的时候，会将stop变量的值拷贝一份放在自己的工作内存当中。

**那么当线程2更改了stop变量的值之后，但是还没来得及写入主存当中，线程2转去做其他事情了，那么线程1由于不知道线程2对stop变量的更改，因此还会一直循环下去。**

但是用volatile修饰之后就变得不一样了：

第一：使用volatile关键字会**强制将修改的值立即写入主存**；

第二：使用volatile关键字的话，当线程2进行修改时，**会导致线程1的工作内存中缓存变量stop的缓存行无效**（反映到硬件层的话，就是CPU的L1或者L2缓存中对应的缓存行无效）；

第三：由于线程1的工作内存中缓存变量stop的缓存行无效，所以**线程1再次读取变量stop的值时会去主存读取**。

那么在线程2修改stop值时（当然这里包括2个操作，修改线程2工作内存中的值，然后将修改后的值写入内存），会使得线程1的工作内存中缓存变量stop的缓存行无效，然后线程1读取时，发现自己的缓存行无效，它会等待缓存行对应的主存地址被更新之后，然后去对应的主存读取最新的值。

那么线程1读取到的就是最新的正确的值。

#### volatile不能确保原子性

下面看一个例子：

```java
public class Nothing {

    private volatile int inc = 0;
    private volatile static int count = 10;

    private void increase() {
        ++inc;
    }

    public static void main(String[] args) {
        int loop = 10;
        Nothing nothing = new Nothing();
        while (loop-- > 0) {
            nothing.operation();
        }
    }

    private void operation() {
        final Nothing test = new Nothing();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000000; j++) {
                    test.increase();
                }
                --count;
            }).start();
        }

        // 保证前面的线程都执行完
        while (count > 0) {

        }
        System.out.println("最后的数据为：" + test.inc);
    }

}
```

运行结果为：

```
最后的数据为：5919956
最后的数据为：3637231
最后的数据为：2144549
最后的数据为：2403538
最后的数据为：1762639
最后的数据为：2878721
最后的数据为：2658645
最后的数据为：2534078
最后的数据为：2031751
最后的数据为：2924506
```

大家想一下这段程序的输出结果是多少？也许有些朋友认为是1000000。但是事实上运行它会发现每次运行结果都不一致，都是一个小于1000000的数字。

可能有的朋友就会有疑问，不对啊，上面是对变量inc进行自增操作，由于volatile保证了可见性，那么在每个线程中对inc自增完之后，在其他线程中都能看到修改后的值啊，所以有10个线程分别进行了1000000次操作，那么最终inc的值应该是1000000*10=10000000。

这里面就有一个误区了，**volatile关键字能保证可见性没有错，但是上面的程序错在没能保证原子性。** 可见性只能保证每次读取的是最新的值，但是volatile没办法保证对变量的操作的原子性。

在前面已经提到过，**自增操作是不具备原子性的，它包括读取变量的原始值、进行加1操作、写入工作内存**。那么就是说自增操作的三个子操作可能会分割开执行，就有可能导致下面这种情况出现：

假如某个时刻变量inc的值为10，**线程1对变量进行自增操作，线程1先读取了变量inc的原始值，然后线程1被阻塞了**；

然后线程2对变量进行自增操作，线程2也去读取变量inc的原始值，**由于线程1只是对变量inc进行读取操作，而没有对变量进行修改操作，所以不会导致线程2的工作内存中缓存变量inc的缓存行无效，也不会导致主存中的值刷新，** 所以线程2会直接去主存读取inc的值，发现inc的值时10，然后进行加1操作，并把11写入工作内存，最后写入主存。

然后线程1接着进行加1操作，由于已经读取了inc的值，注意此时在线程1的工作内存中inc的值仍然为10，所以线程1对inc进行加1操作后inc的值为11，然后将11写入工作内存，最后写入主存。

那么两个线程分别进行了一次自增操作后，inc只增加了1。

**根源就在这里，自增操作不是原子性操作，而且volatile也无法保证对变量的任何操作都是原子性的。**

**解决方案：可以通过synchronized或lock，进行加锁，来保证操作的原子性。也可以通过AtomicInteger。**

在java 1.5的java.util.concurrent.atomic包下提供了一些**原子操作类**，即对基本数据类型的 自增（加1操作），自减（减1操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作。**atomic是利用CAS来实现原子性操作的（Compare And Swap）**，CAS实际上是**利用处理器提供的CMPXCHG指令实现的，而处理器执行CMPXCHG指令是一个原子性操作。**

#### volatile保证有序性

在前面提到volatile关键字能禁止指令重排序，所以volatile能在一定程度上保证有序性。

volatile关键字禁止指令重排序有两层意思：

1）当程序执行到volatile变量的读操作或者写操作时，**在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行**；

2）在进行指令优化时，**不能将在对volatile变量的读操作或者写操作的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。**

可能上面说的比较绕，举个简单的例子：

```java
// x、y为非volatile变量
// flag为volatile变量

x = 2;        //语句1
y = 0;        //语句2
flag = true;  //语句3
x = 4;        //语句4
y = -1;       //语句5
```

由于**flag变量为volatile变量**，那么在进行指令重排序的过程的时候，**不会将语句3放到语句1、语句2前面，也不会讲语句3放到语句4、语句5后面。但是要注意语句1和语句2的顺序、语句4和语句5的顺序是不作任何保证的。**

并且volatile关键字能保证，**执行到语句3时，语句1和语句2必定是执行完毕了的，且语句1和语句2的执行结果对语句3、语句4、语句5是可见的。**

那么我们回到前面举的一个例子：

```java
//线程1:
context = loadContext();   //语句1
inited = true;             //语句2

//线程2:
while(!inited ){
  sleep()
}
doSomethingwithconfig(context);
```

前面举这个例子的时候，提到有可能语句2会在语句1之前执行，那么久可能导致context还没被初始化，而线程2中就使用未初始化的context去进行操作，导致程序出错。

这里如果用volatile关键字对inited变量进行修饰，就不会出现这种问题了，**因为当执行到语句2时，必定能保证context已经初始化完毕。**

### 6. volatile的实现原理

**可见性**

处理器为了提高处理速度，不直接和内存进行通讯，而是将系统内存的数据独到内部缓存后再进行操作，但操作完后不知什么时候会写到内存。

如果**对声明了volatile变量进行写操作时，JVM会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写会到系统内存。** 这一步确保了如果有其他线程对声明了volatile变量进行修改，则立即更新主内存中数据。

**但这时候其他处理器的缓存还是旧的，所以在多处理器环境下，为了保证各个处理器缓存一致，每个处理会通过嗅探在总线上传播的数据来检查 自己的缓存是否过期，** 当处理器发现自己缓存行对应的内存地址被修改了，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作时，**会强制重新从系统内存把数据读到处理器缓存里。** 这一步确保了其他线程获得的声明了volatile变量都是从主内存中获取最新的。

**有序性**

Lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），它确保**指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；**即在执行到内存屏障这句指令时，在它前面的操作已经全部完成。

### 7. volatile的应用场景

synchronized关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized，但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。通常来说，使用volatile必须具备以下2个条件：

1）对变量的写操作不依赖于当前值

2）该变量没有包含在具有其他变量的不变式中

下面列举几个Java中使用volatile的几个场景。

**①.状态标记量**

```java
volatile boolean flag = false;
 //线程1
while(!flag){
    doSomething();
}
  //线程2
public void setFlag() {
    flag = true;
}
```

根据状态标记，终止线程。

**②.单例模式中的double check**

```java
class Singleton {
    private volatile static Singleton instance = null;

    private Singleton() {

    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

**为什么要使用volatile 修饰instance？**

主要在于instance = new Singleton()这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情:

1.给 instance 分配内存

2.调用 Singleton 的构造函数来初始化成员变量

3.将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）。

但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后顺理成章地报错。

## 七、CAS原子操作

### 1. 乐观锁与悲观锁

我们都知道，cpu是时分复用的，也就是把cpu的时间片，分配给不同的thread/process轮流执行，时间片与时间片之间，需要进行cpu切换，也就是会发生进程的切换。切换涉及到清空寄存器，缓存数据。然后重新加载新的thread所需数据。当一个线程被挂起时，加入到阻塞队列，在一定的时间或条件下，在通过notify()，notifyAll()唤醒回来。 **在某个资源不可用的时候，就将cpu让出，把当前等待线程切换为阻塞状态。等到资源(比如一个共享数据）可用了，那么就将线程唤醒，让他进入runnable状态等待cpu调度。这就是典型的悲观锁的实现。** 独占锁是一种悲观锁，synchronized就是一种独占锁，它假设最坏的情况，认为一个线程修改共享数据的时候其他线程也会修改该数据，因此只在确保其它线程不会造成干扰的情况下执行，会导致其它所有需要锁的线程挂起，等待持有锁的线程释放锁。

**但是，由于在进程挂起和恢复执行过程中存在着很大的开销**。当一个线程正在等待锁时，它不能做任何事，所以悲观锁有很大的缺点。举个例子，如果一个线程需要某个资源，但是这个资源的占用时间很短，当线程第一次抢占这个资源时，可能这个资源被占用，如果此时挂起这个线程，可能立刻就发现资源可用，然后又需要花费很长的时间重新抢占锁，时间代价就会非常的高。

所以就有了乐观锁的概念，他的核心思路就是，**每次不加锁而是假设修改数据之前其他线程一定不会修改，如果因为修改过产生冲突就失败就重试，直到成功为止。** 在上面的例子中，某个线程可以不让出cpu，而是一直while循环，如果失败就重试，直到成功为止。所以，当数据争用不严重时，乐观锁效果更好。比如CAS就是一种乐观锁思想的应用。

### 2. CAS

**CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。执行CAS操作的时候，将内存位置的值与预期原值比较，如果相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。**

举个CAS操作的应用场景的一个例子，当一个线程需要修改共享变量的值。完成这个操作，先取出共享变量的值赋给A，然后基于A的基础进行计算，得到新值B，完了需要更新共享变量的值了，这个时候就可以调用CAS方法更新变量值了。

在java中可以通过锁和循环CAS的方式来实现原子操作。Java中`java.util.concurrent.atomic`包相关类就是 CAS的实现，atomic包里包括以下类：

| 类名                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| **AtomicBoolean**                     | 可以用原子方式更新的 `boolean` 值。                          |
| **AtomicInteger**                     | 可以用原子方式更新的 `int` 值。                              |
| **AtomicIntegerArray**                | 可以用原子方式更新其元素的 `int` 数组。                      |
| **AtomicIntegerFieldUpdater<T>**      | 基于反射的实用工具，可以对指定类的指定 `volatile int` 字段进行原子更新。 |
| **AtomicLong**                        | 可以用原子方式更新的 `long` 值。                             |
| **AtomicLongArray**                   | 可以用原子方式更新其元素的 `long` 数组。                     |
| **AtomicLongFieldUpdater<T>**         | 基于反射的实用工具，可以对指定类的指定 `volatile long` 字段进行原子更新。 |
| **AtomicMarkableReference<V>**        | `AtomicMarkableReference` 维护带有标记位的对象引用，可以原子方式对其进行更新。 |
| **AtomicReference<V>**                | 可以用原子方式更新的对象引用。                               |
| **AtomicReferenceArray<E>**           | 可以用原子方式更新其元素的对象引用数组。                     |
| **AtomicReferenceFieldUpdater<T，V>** | 基于反射的实用工具，可以对指定类的指定 `volatile` 字段进行原子更新。 |
| **AtomicStampedReference<V>**         | `AtomicStampedReference` 维护带有整数“标志”的对象引用，可以用原子方式对其进行更新。 |

下面我们来已AtomicInteger的源码为例来看看CAS操作：

```java
public final int getAndAdd(int delta) {
	for (; ; ) {
		int current = get();
		int next =t current + delta;
		if (compareAndSet(current, next))
			return current;
	}
}
```

这里很显然使用CAS操作（for(;;)里面），他每次都从内存中读取数据，+1操作，然后两个值进行CAS操作。如果成功则返回，否则失败重试，直到修改成功为止。上面源码最关键的地方有两个，一个for循环，它代表着一种宁死不屈的精神，不成功誓不罢休。还有就是compareAndSet：

```java
public final boolean compareAndSet(int expect, int update) {
	return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

compareAndSet方法内部是调用Java本地方法compareAndSwapInt来实现的，而compareAndSwapInt方法内部又是借助C来调用CPU的底层指令来保证在硬件层面上实现原子操作的。在intel处理器中，CAS是通过调用**cmpxchg**指令完成的。这就是我们常说的**CAS操作**（compare and swap）。

### 3. CAS的问题

CAS虽然很高效的解决原子操作，但是CAS仍然存在三大问题。ABA问题，循环时间长开销大和只能保证一个共享变量的原子操作。

- ABA问题。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

- 循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

- 只能保证一个共享变量的原子操作。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。

## 八、AbstractQueuedSynchronizer详解

### 1. AQS简介

#### 1.1 AQS介绍

AbstractQueuedSynchronizer提供了一个基于FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架。该同步器（以下简称同步器）利用了一个int来表示状态，期望它能够成为实现大部分同步需求的基础。使用的方法是继承，子类通过继承同步器并需要实现它的方法来管理其状态，管理的方式就是通过类似acquire和release的方式来操纵状态。然而多线程环境中对状态的操纵必须确保原子性，因此子类对于状态的把握，需要使用这个同步器提供的以下三个方法对状态进行操作：

```java
java.util.concurrent.locks.AbstractQueuedSynchronizer.getState()
java.util.concurrent.locks.AbstractQueuedSynchronizer.setState(int)
java.util.concurrent.locks.AbstractQueuedSynchronizer.compareAndSetState(int, int)
```

子类推荐被定义为自定义同步装置的内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干acquire之类的方法来供使用。该同步器即可以作为排他模式也可以作为共享模式，当它被定义为一个排他模式时，其他线程对其的获取就被阻止，而共享模式对于多个线程获取都可以成功。

子类推荐被定义为自定义同步装置的内部类，同步器自身没有实现任何同步接口，它仅仅是定义了若干acquire之类的方法来供使用。该同步器即可以作为排他模式也可以作为共享模式，当它被定义为一个排他模式时，其他线程对其的获取就被阻止，而共享模式对于多个线程获取都可以成功。

#### 1.2 AQS用处

[![img](https://camo.githubusercontent.com/3b1e6962810537203ee0c1cf9df037bd6d2dbdbe/687474703a2f2f6c756f6a696e70696e672e636f6d2f696d672f4151535f757365732e6a7067)](http://luojinping.com/img/AQS_uses.jpg)

#### 1.3 同步器与锁

**同步器是实现锁的关键，利用同步器将锁的语义实现，然后在锁的实现中聚合同步器。** 可以这样理解：锁的API是面向使用者的，它定义了与锁交互的公共行为，而每个锁需要完成特定的操作也是透过这些行为来完成的（比如：可以允许两个线程进行加锁，排除两个以上的线程），但是实现是依托给同步器来完成；同步器面向的是线程访问和资源控制，它定义了线程对资源是否能够获取以及线程的排队等操作。**锁和同步器很好的隔离了二者所需要关注的领域，严格意义上讲，同步器可以适用于除了锁以外的其他同步设施上（包括锁）。** 同步器的开始提到了其实现依赖于一个FIFO队列，那么队列中的元素Node就是保存着线程引用和线程状态的容器，每个线程对同步器的访问，都可以看做是队列中的一个节点。Node的主要包含以下成员变量：

```java
Node {
    int waitStatus;
    Node prev;
    Node next;
    Node nextWaiter;
    Thread thread;
}
```

以上五个成员变量主要负责保存该节点的线程引用，同步等待队列（以下简称sync队列）的前驱和后继节点，同时也包括了同步状态。

| 属性名称        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| int waitStatus  | 表示节点的状态。其中包含的状态有： CANCELLED，值为1，表示当前的线程被取消；SIGNAL，值为-1，表示当前节点的后继节点包含的线程需要运行，也就是unpark；CONDITION，值为-2，表示当前节点在等待condition，也就是在condition队列中；PROPAGATE，值为-3，表示当前场景下后续的acquireShared能够得以执行；值为0，表示当前节点在sync队列中，等待着获取锁。 |
| Node prev       | 前驱节点，比如当前节点被取消，那就需要前驱节点和后继节点来完成连接。 |
| Node next       | 后继节点。                                                   |
| Node nextWaiter | 存储condition队列中的后继节点。                              |
| Thread thread   | 入队列时的当前线程。                                         |

节点成为sync队列和condition队列构建的基础，在同步器中就包含了sync队列。同步器拥有三个成员变量：sync队列的头结点head、sync队列的尾节点tail和状态state。对于锁的获取，请求形成节点，将其挂载在尾部，而锁资源的转移（释放再获取）是从头部开始向后进行。对于同步器维护的状态state，多个线程对其的获取将会产生一个链式的结构。

[![img](https://camo.githubusercontent.com/e00923b637179b9c991410eece69d2cfd558e0fa/687474703a2f2f69666576652e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031332f31302f32312e706e67)](http://ifeve.com/wp-content/uploads/2013/10/21.png)

**API说明**

实现自定义同步器时，需要使用同步器提供的getState()、setState()和compareAndSetState()方法来操纵状态的变迁。

| 方法名称                                    | 描述                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| protected boolean tryAcquire(int arg)       | 排它的获取这个状态。这个方法的实现需要查询当前状态是否允许获取，然后再进行获取（使用compareAndSetState来做）状态。 |
| protected boolean tryRelease(int arg)       | 释放状态。                                                   |
| protected int tryAcquireShared(int arg)     | 共享的模式下获取状态。                                       |
| protected boolean tryReleaseShared(int arg) | 共享的模式下释放状态。                                       |
| protected boolean isHeldExclusively()       | 在排它模式下，状态是否被占用。                               |

实现这些方法必须是非阻塞而且是线程安全的，推荐使用该同步器的父类java.util.concurrent.locks.AbstractOwnableSynchronizer来设置当前的线程。 开始提到同步器内部基于一个FIFO队列，对于一个独占锁的获取和释放有以下伪码可以表示。 获取一个排他锁。

```java
while(获取锁) {
    if (获取到) {
        退出while循环
    } else {
        if(当前线程没有入队列) {
            那么入队列
        }
        阻塞当前线程
    }
}
释放一个排他锁。
if (释放成功) {
    删除头结点
    激活原头结点的后继节点
}
```

#### 1.4 Mutex 示例

下面通过一个排它锁的例子来深入理解一下同步器的工作原理，而只有掌握同步器的工作原理才能够更加深入了解其他的并发组件。 排他锁的实现，一次只能一个线程获取到锁。

```java
class Mutex implements Lock, java.io.Serializable {
   // 内部类，自定义同步器
   private static class Sync extends AbstractQueuedSynchronizer {
     // 是否处于占用状态
     protected boolean isHeldExclusively() {
       return getState() == 1;
     }
     // 当状态为0的时候获取锁
     public boolean tryAcquire(int acquires) {
       assert acquires == 1; // Otherwise unused
       if (compareAndSetState(0, 1)) {
         setExclusiveOwnerThread(Thread.currentThread());
         return true;
       }
       return false;
     }
     // 释放锁，将状态设置为0
     protected boolean tryRelease(int releases) {
       assert releases == 1; // Otherwise unused
       if (getState() == 0) throw new IllegalMonitorStateException();
       setExclusiveOwnerThread(null);
       setState(0);
       return true;
     }
     // 返回一个Condition，每个condition都包含了一个condition队列
     Condition newCondition() { return new ConditionObject(); }
   }
   // 仅需要将操作代理到Sync上即可
   private final Sync sync = new Sync();
   public void lock()                { sync.acquire(1); }
   public boolean tryLock()          { return sync.tryAcquire(1); }
   public void unlock()              { sync.release(1); }
   public Condition newCondition()   { return sync.newCondition(); }
   public boolean isLocked()         { return sync.isHeldExclusively(); }
   public boolean hasQueuedThreads() { return sync.hasQueuedThreads(); }
   public void lockInterruptibly() throws InterruptedException {
     sync.acquireInterruptibly(1);
   }
   public boolean tryLock(long timeout, TimeUnit unit)
       throws InterruptedException {
     return sync.tryAcquireNanos(1, unit.toNanos(timeout));
   }
 }
```

可以看到Mutex将Lock接口均代理给了同步器的实现。 使用方将Mutex构造出来之后，调用lock获取锁，调用unlock进行解锁。下面以Mutex为例子，详细分析以下同步器的实现逻辑。

### 2. 独占模式

#### 2.1 acquire

实现分析 public final void acquire(int arg) 该方法以排他的方式获取锁，对中断不敏感，完成synchronized语义。

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```

上述逻辑主要包括：

1. 尝试获取（调用tryAcquire更改状态，需要保证原子性）； 在tryAcquire方法中使用了同步器提供的对state操作的方法，利用compareAndSet保证只有一个线程能够对状态进行成功修改，而没有成功修改的线程将进入sync队列排队。
2. 如果获取不到，将当前线程构造成节点Node并加入sync队列； 进入队列的每个线程都是一个节点Node，从而形成了一个双向队列，类似CLH队列，这样做的目的是线程间的通信会被限制在较小规模（也就是两个节点左右）。
3. 再次尝试获取，如果没有获取到那么将当前线程从线程调度器上摘下，进入等待状态。 使用LockSupport将当前线程unpark，关于LockSupport后续会详细介绍。

```java
private Node addWaiter(Node mode) {
	Node node = new Node(Thread.currentThread(), mode);
	// 快速尝试在尾部添加
	Node pred = tail;
	if (pred != null) {
		node.prev = pred;
		if (compareAndSetTail(pred, node)) {
			pred.next = node;
			return node;
		}
	}
	enq(node);
	return node;
}

private Node enq(final Node node) {
	for (;;) {
		Node t = tail;
		if (t == null) { // Must initialize
			if (compareAndSetHead(new Node()))
				tail = head;
		} else {
			node.prev = t;
			if (compareAndSetTail(t, node)) {
			t.next = node;
			return t;
		}
	}
}
```

上述逻辑主要包括：

1. 使用当前线程构造Node； 对于一个节点需要做的是将当节点前驱节点指向尾节点（current.prev = tail），尾节点指向它（tail = current），原有的尾节点的后继节点指向它（t.next = current）而这些操作要求是原子的。上面的操作是利用尾节点的设置来保证的，也就是compareAndSetTail来完成的。

- 先行尝试在队尾添加；

  如果尾节点已经有了，然后做如下操作：

  1. 分配引用T指向尾节点；
  2. 将节点的前驱节点更新为尾节点（current.prev = tail）；
  3. 如果尾节点是T，那么将当尾节点设置为该节点（tail = current，原子更新）；
  4. T的后继节点指向当前节点（T.next = current）。 注意第3点是要求原子的。 这样可以以最短路径O(1)的效果来完成线程入队，是最大化减少开销的一种方式。

- 如果队尾添加失败或者是第一个入队的节点。

  如果是第1个节点，也就是sync队列没有初始化，那么会进入到enq这个方法，进入的线程可能有多个，或者说在addWaiter中没有成功入队的线程都将进入enq这个方法。

  可以看到enq的逻辑是确保进入的Node都会有机会顺序的添加到sync队列中，而加入的步骤如下：

  1. 如果尾节点为空，那么原子化的分配一个头节点，并将尾节点指向头节点，这一步是初始化；
  2. **然后是重复在addWaiter中做的工作，但是在一个for(;;)的循环中，直到当前节点入队为止。**

进入sync队列之后，接下来就是要进行锁的获取，或者说是访问控制了，只有一个线程能够在同一时刻继续的运行，而其他的进入等待状态。而每个线程都是一个独立的个体，它们自省的观察，当条件满足的时候（自己的前驱是头结点并且原子性的获取了状态），那么这个线程能够继续运行。

```java
final boolean acquireQueued(final Node node, int arg) {
	boolean failed = true;
	try {
		boolean interrupted = false;
		for (;;) {
			final Node p = node.predecessor();
			if (p == head &&tryAcquire(arg)) {
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return interrupted;
			}
			if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
				interrupted = true;
                }
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```

上述逻辑主要包括：

1. 获取当前节点的前驱节点； 需要获取当前节点的前驱节点，而头结点所对应的含义是当前占有锁且正在运行。
2. 当前驱节点是头结点并且能够获取状态，代表该当前节点占有锁； 如果满足上述条件，那么代表能够占有锁，根据节点对锁占有的含义，设置头结点为当前节点。
3. 否则进入等待状态。 如果没有轮到当前节点运行，那么将当前线程从线程调度器上摘下，也就是进入等待状态。 这里针对acquire做一下总结：
4. 状态的维护； 需要在锁定时，需要维护一个状态(int类型)，而对状态的操作是原子和非阻塞的，通过同步器提供的对状态访问的方法对状态进行操纵，并且利用compareAndSet来确保原子性的修改。
5. 状态的获取； 一旦成功的修改了状态，当前线程或者说节点，就被设置为头节点。
6. sync队列的维护。 在获取资源未果的过程中条件不符合的情况下(不该自己，前驱节点不是头节点或者没有获取到资源)进入睡眠状态，停止线程调度器对当前节点线程的调度。 这时引入的一个释放的问题，也就是说使睡眠中的Node或者说线程获得通知的关键，就是前驱节点的通知，而这一个过程就是释放，释放会通知它的后继节点从睡眠中返回准备运行。 下面的流程图基本描述了一次acquire所需要经历的过程：

[![img](https://camo.githubusercontent.com/da70d9baa1ed377d98ce785f11f9c354b055d389/687474703a2f2f696d67322e746263646e2e636e2f4c312f3436312f312f745f393638335f313337393332383534325f3932383139313734382e706e67)](http://img2.tbcdn.cn/L1/461/1/t_9683_1379328542_928191748.png)

如上图所示，其中的判定退出队列的条件，判定条件是否满足和休眠当前线程就是完成了自旋spin的过程。

#### 2.2 release

public final boolean release(int arg) 在unlock方法的实现中，使用了同步器的release方法。相对于在之前的acquire方法中可以得出调用acquire，保证能够获取到锁（成功获取状态），而release则表示将状态设置回去，也就是将资源释放，或者说将锁释放。

```
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

上述逻辑主要包括：

1. 尝试释放状态； tryRelease能够保证原子化的将状态设置回去，当然需要使用compareAndSet来保证。如果释放状态成功过之后，将会进入后继节点的唤醒过程。
2. 唤醒当前节点的后继节点所包含的线程。 通过LockSupport的unpark方法将休眠中的线程唤醒，让其继续acquire状态。

```java
private void unparkSuccessor(Node node) {
	// 将状态设置为同步状态
	int ws = node.waitStatus;
	if (ws < 0) 		
		compareAndSetWaitStatus(node, ws, 0); 	
	
	/* 
	* 获取当前节点的后继节点，如果满足状态，那么进行唤醒操作 
	* 如果没有满足状态，从尾部开始找寻符合要求的节点并将其唤醒 
	*/
	
	Node s = node.next; 	
	if (s == null || s.waitStatus > 0) {
		s = null;
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)
				s = t;
		}
	if (s != null)
		LockSupport.unpark(s.thread);
}
```

上述逻辑主要包括，该方法取出了当前节点的next引用，然后对其线程(Node)进行了唤醒，这时就只有一个或合理个数的线程被唤醒，被唤醒的线程继续进行对资源的获取与争夺。 回顾整个资源的获取和释放过程： 在获取时，维护了一个sync队列，每个节点都是一个线程在进行自旋，而依据就是自己是否是首节点的后继并且能够获取资源； 在释放时，仅仅需要将资源还回去，然后通知一下后继节点并将其唤醒。 这里需要注意，队列的维护（首节点的更换）是依靠消费者（获取时）来完成的，也就是说在满足了自旋退出的条件时的一刻，这个节点就会被设置成为首节点。

#### 2.3 tryAcquire

protected boolean tryAcquire(int arg) tryAcquire是自定义同步器需要实现的方法，也就是自定义同步器非阻塞原子化的获取状态，如果锁该方法一般用于Lock的tryLock实现中，这个特性是synchronized无法提供的。

public final void acquireInterruptibly(int arg) 该方法提供获取状态能力，当然在无法获取状态的情况下会进入sync队列进行排队，这类似acquire，但是和acquire不同的地方在于它能够在外界对当前线程进行中断的时候提前结束获取状态的操作，换句话说，就是在类似synchronized获取锁时，外界能够对当前线程进行中断，并且获取锁的这个操作能够响应中断并提前返回。一个线程处于synchronized块中或者进行同步I/O操作时，对该线程进行中断操作，这时该线程的中断标识位被设置为true，但是线程依旧继续运行。 如果在获取一个通过网络交互实现的锁时，这个锁资源突然进行了销毁，那么使用acquireInterruptibly的获取方式就能够让该时刻尝试获取锁的线程提前返回。而同步器的这个特性被实现Lock接口中的lockInterruptibly方法。根据Lock的语义，在被中断时，lockInterruptibly将会抛出InterruptedException来告知使用者。

```java
public final void acquireInterruptibly(int arg)
	throws InterruptedException {
	if (Thread.interrupted())
		throw new InterruptedException();
	if (!tryAcquire(arg))
		doAcquireInterruptibly(arg);
}

private void doAcquireInterruptibly(int arg)
	throws InterruptedException {
	final Node node = addWaiter(Node.EXCLUSIVE);
	boolean failed = true;
	try {
		for (;;) {
			final Node p = node.predecessor();
			if (p == head && tryAcquire(arg)) {
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return;
			}
			// 检测中断标志位
			if (shouldParkAfterFailedAcquire(p, node) &&
			parkAndCheckInterrupt())
				throw new InterruptedException();
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```

上述逻辑主要包括：

1. 检测当前线程是否被中断； 判断当前线程的中断标志位，如果已经被中断了，那么直接抛出异常并将中断标志位设置为false。
2. 尝试获取状态； 调用tryAcquire获取状态，如果顺利会获取成功并返回。
3. 构造节点并加入sync队列； 获取状态失败后，将当前线程引用构造为节点并加入到sync队列中。退出队列的方式在没有中断的场景下和acquireQueued类似，当头结点是自己的前驱节点并且能够获取到状态时，即可以运行，当然要将本节点设置为头结点，表示正在运行。
4. 中断检测。 在每次被唤醒时，进行中断检测，如果发现当前线程被中断，那么抛出InterruptedException并退出循环。

#### 2.4 doAcquireNanos

private boolean doAcquireNanos(int arg, long nanosTimeout) throws InterruptedException 该方法提供了具备有超时功能的获取状态的调用，如果在指定的nanosTimeout内没有获取到状态，那么返回false，反之返回true。可以将该方法看做acquireInterruptibly的升级版，也就是在判断是否被中断的基础上增加了超时控制。 针对超时控制这部分的实现，主要需要计算出睡眠的delta，也就是间隔值。间隔可以表示为nanosTimeout = 原有nanosTimeout – now（当前时间）+ lastTime（睡眠之前记录的时间）。如果nanosTimeout大于0，那么还需要使当前线程睡眠，反之则返回false。

```java
private boolean doAcquireNanos(int arg, long nanosTimeout)
throws InterruptedException {
	long lastTime = System.nanoTime();
	final Node node = addWaiter(Node.EXCLUSIVE);
	boolean failed = true;
	try {
		for (;;) {
			final Node p = node.predecessor();
			if (p == head &&tryAcquire(arg)) {
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return true;
			}
			if (nanosTimeout <= 0) 				return false; 			if (shouldParkAfterFailedAcquire(p, node) && nanosTimeout > spinForTimeoutThreshold)
			LockSupport.parkNanos(this, nanosTimeout);
			long now = System.nanoTime();
			//计算时间，当前时间减去睡眠之前的时间得到睡眠的时间，然后被
			//原有超时时间减去，得到了还应该睡眠的时间
			nanosTimeout -= now - lastTime;
			lastTime = now;
			if (Thread.interrupted())
				throw new InterruptedException();
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```

上述逻辑主要包括：

1. 加入sync队列； 将当前线程构造成为节点Node加入到sync队列中。
2. 条件满足直接返回； 退出条件判断，如果前驱节点是头结点并且成功获取到状态，那么设置自己为头结点并退出，返回true，也就是在指定的nanosTimeout之前获取了锁。
3. 获取状态失败休眠一段时间； 通过LockSupport.unpark来指定当前线程休眠一段时间。
4. 计算再次休眠的时间； 唤醒后的线程，计算仍需要休眠的时间，该时间表示为nanosTimeout = 原有nanosTimeout – now（当前时间）+ lastTime（睡眠之前记录的时间）。其中now – lastTime表示这次睡眠所持续的时间。
5. 休眠时间的判定。 唤醒后的线程，计算仍需要休眠的时间，并无阻塞的尝试再获取状态，如果失败后查看其nanosTimeout是否大于0，如果小于0，那么返回完全超时，没有获取到锁。 如果nanosTimeout小于等于1000L纳秒，则进入快速的自旋过程。那么快速自旋会造成处理器资源紧张吗？结果是不会，经过测算，开销看起来很小，几乎微乎其微。Doug Lea应该测算了在线程调度器上的切换造成的额外开销，因此在短时1000纳秒内就让当前线程进入快速自旋状态，如果这时再休眠相反会让nanosTimeout的获取时间变得更加不精确。 上述过程可以如下图所示：

[![img](https://camo.githubusercontent.com/f21e8d65f3fda25be4db2e5b8ade65d4cb418756/687474703a2f2f696d67322e746263646e2e636e2f4c312f3436312f312f745f393638335f313337393332383839315f3536383033343038312e706e67)](http://img2.tbcdn.cn/L1/461/1/t_9683_1379328891_568034081.png)

上述这个图中可以理解为在类似获取状态需要排队的基础上增加了一个超时控制的逻辑。每次超时的时间就是当前超时剩余的时间减去睡眠的时间，而在这个超时时间的基础上进行了判断，如果大于0那么继续睡眠（等待），可以看出这个超时版本的获取状态只是一个近似超时的获取状态，因此任何含有超时的调用基本结果就是近似于给定超时。

### 3. 共享模式

#### 3.1 acquireShared

public final void acquireShared(int arg) 调用该方法能够以共享模式获取状态，共享模式和之前的独占模式有所区别。以文件的查看为例，如果一个程序在对其进行读取操作，那么这一时刻，对这个文件的写操作就被阻塞，相反，这一时刻另一个程序对其进行同样的读操作是可以进行的。如果一个程序在对其进行写操作，那么所有的读与写操作在这一时刻就被阻塞，直到这个程序完成写操作。 以读写场景为例，描述共享和独占的访问模式，如下图所示：

[![img](https://camo.githubusercontent.com/1655b0d50856e347338d8ef1bba1dbb157c3bc94/687474703a2f2f696d67312e746263646e2e636e2f4c312f3436312f312f745f393638335f313337393332383935395f313730323338383033312e706e67)](http://img1.tbcdn.cn/L1/461/1/t_9683_1379328959_1702388031.png)

上图中，红色代表被阻塞，绿色代表可以通过。

```java
public final void acquireShared(int arg) {
	if (tryAcquireShared(arg) < 0)	
      doAcquireShared(arg); 
} 
private void doAcquireShared(int arg) { 	
  final Node node = addWaiter(Node.SHARED); 	
  boolean failed = true; 	
  try { 		
    boolean interrupted = false; 		
    for (;;) { 			
      final Node p = node.predecessor(); 			
      if (p == head) { 				
        int r = tryAcquireShared(arg); 				
        if (r >= 0) {
          setHeadAndPropagate(node, r);
          p.next = null; // help GC
          if (interrupted)
            selfInterrupt();
			failed = false;
			return;
				}
			}
      if (shouldParkAfterFailedAcquire(p, node) &&
parkAndCheckInterrupt())
			interrupted = true;
    }
  } finally {
    if (failed)
	cancelAcquire(node);
	}
}
```

上述逻辑主要包括：

1. 尝试获取共享状态； 调用tryAcquireShared来获取共享状态，该方法是非阻塞的，如果获取成功则立刻返回，也就表示获取共享锁成功。
2. 获取失败进入sync队列； 在获取共享状态失败后，当前时刻有可能是独占锁被其他线程所把持，那么将当前线程构造成为节点（共享模式）加入到sync队列中。
3. 循环内判断退出队列条件； 如果当前节点的前驱节点是头结点并且获取共享状态成功，这里和独占锁acquire的退出队列条件类似。
4. 获取共享状态成功； 在退出队列的条件上，和独占锁之间的主要区别在于获取共享状态成功之后的行为，而如果共享状态获取成功之后会判断后继节点是否是共享模式，如果是共享模式，那么就直接对其进行唤醒操作，也就是同时激发多个线程并发的运行。
5. 获取共享状态失败。 通过使用LockSupport将当前线程从线程调度器上摘下，进入休眠状态。 对于上述逻辑中，节点之间的通知过程如下图所示：

[![img](https://camo.githubusercontent.com/1ddfdc594b8b0d2ee70f84852b9b7c65c64ad6b3/687474703a2f2f696d67312e746263646e2e636e2f4c312f3436312f312f745f393638335f313337393332393231375f313534323936373532342e706e67)](http://img1.tbcdn.cn/L1/461/1/t_9683_1379329217_1542967524.png)

上图中，绿色表示共享节点，它们之间的通知和唤醒操作是在前驱节点获取状态时就进行的，红色表示独占节点，它的被唤醒必须取决于前驱节点的释放，也就是release操作，可以看出来图中的独占节点如果要运行，必须等待前面的共享节点均释放了状态才可以。而独占节点如果获取了状态，那么后续的独占式获取和共享式获取均被阻塞。

#### 3.2 releaseShared

public final boolean releaseShared(int arg) 调用该方法释放共享状态，每次获取共享状态acquireShared都会操作状态，同样在共享锁释放的时候，也需要将状态释放。比如说，一个限定一定数量访问的同步工具，每次获取都是共享的，但是如果超过了一定的数量，将会阻塞后续的获取操作，只有当之前获取的消费者将状态释放才可以使阻塞的获取操作得以运行。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

上述逻辑主要就是调用同步器的tryReleaseShared方法来释放状态，并同时在doReleaseShared方法中唤醒其后继节点。

#### 3.3 一个例子 TwinsLock

在上述对同步器AbstractQueuedSynchronizer进行了实现层面的分析之后，我们通过一个例子来加深对同步器的理解： 设计一个同步工具，该工具在同一时刻，只能有两个线程能够并行访问，超过限制的其他线程进入阻塞状态。 对于这个需求，可以利用同步器完成一个这样的设定，定义一个初始状态，为2，一个线程进行获取那么减1，一个线程释放那么加1，状态正确的范围在[0，1，2]三个之间，当在0时，代表再有新的线程对资源进行获取时只能进入阻塞状态（注意在任何时候进行状态变更的时候均需要以CAS作为原子性保障）。由于资源的数量多于1个，同时可以有两个线程占有资源，因此需要实现tryAcquireShared和tryReleaseShared方法，这里谢谢luoyuyou和同事小明指正，已经修改了实现。

```java
public class TwinsLock implements Lock {
	private final Sync	sync	= new Sync(2);

	private static final class Sync extends AbstractQueuedSynchronizer {
		private static final long	serialVersionUID	= -7889272986162341211L;

		Sync(int count) {
			if (count <= 0) {
				throw new IllegalArgumentException("count must large than zero.");
			}
			setState(count);
		}

		public int tryAcquireShared(int reduceCount) {
			for (;;) {
				int current = getState();
				int newCount = current - reduceCount;
				if (newCount < 0 || compareAndSetState(current, newCount)) {
					return newCount;
				}
			}
		}

		public boolean tryReleaseShared(int returnCount) {
			for (;;) {
				int current = getState();
				int newCount = current + returnCount;
				if (compareAndSetState(current, newCount)) {
					return true;
				}
			}
		}
	}

	public void lock() {
		sync.acquireShared(1);
	}

	public void lockInterruptibly() throws InterruptedException {
		sync.acquireSharedInterruptibly(1);
	}

	public boolean tryLock() {
		return sync.tryAcquireShared(1) >= 0;
	}

	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
		return sync.tryAcquireSharedNanos(1, unit.toNanos(time));
	}

	public void unlock() {
		sync.releaseShared(1);
	}

	@Override
	public Condition newCondition() {
		return null;
	}
}
```

上述测试用例的逻辑主要包括：

1. 打印线程 Worker在两次睡眠之间打印自身线程，如果一个时刻只能有两个线程同时访问，那么打印出来的内容将是成对出现。
2. 分隔线程 不停的打印换行，能让Worker的输出看起来更加直观。 该测试的结果是在一个时刻，仅有两个线程能够获得到锁，并完成打印，而表象就是打印的内容成对出现。

### 4. 总结

AQS简核心是通过一个共享变量来同步状态，变量的状态由子类去维护，而AQS框架做的是：

- 线程阻塞队列的维护
- 线程阻塞和唤醒

共享变量的修改都是通过Unsafe类提供的CAS操作完成的。 AbstractQueuedSynchronizer类的主要方法是acquire和release，典型的模板方法， 下面这4个方法由子类去实现：

```java
protected boolean tryAcquire(int arg)
protected boolean tryRelease(int arg)
protected int tryAcquireShared(int arg)
protected boolean tryReleaseShared(int arg)
```

acquire方法用来获取锁，返回true说明线程获取成功继续执行，一旦返回false则线程加入到等待队列中，等待被唤醒，release方法用来释放锁。 一般来说实现的时候这两个方法被封装为lock和unlock方法。

## 九、深入理解ReentrantLock

java5之后，并发包中新增了Lock接口（以及相关实现类）用来实现锁的功能，它提供了与synchronized关键字类似的同步功能。既然有了synchronized这种内置的锁功能，为何要新增Lock接口？先来想象一个场景：手把手的进行锁获取和释放，先获得锁A，然后再获取锁B，当获取锁B后释放锁A同时获取锁C，当锁C获取后，再释放锁B同时获取锁D，以此类推，这种场景下，synchronized关键字就不那么容易实现了，而使用Lock却显得容易许多。

### 1. 定义

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private final Sync sync;
    abstract static class Sync extends AbstractQueuedSynchronizer {

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
    }
    //默认非公平锁
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    //fair为false时，采用公平锁策略
    public ReentrantLock(boolean fair) {    
        sync = fair ? new FairSync() : new NonfairSync();
    }
    public void lock() {
        sync.lock();
    }
    public void unlock() {    sync.release(1);}
    public Condition newCondition() {    
        return sync.newCondition();
    }
    ...
}
```

从源代码可以Doug lea巧妙的采用组合模式把lock和unlock方法委托给同步器完成。

### 2. 使用方式

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
lock.lock();
try {
  while(条件判断表达式) {
      condition.wait();
  }
 // 处理逻辑
} finally {
    lock.unlock();
}
```

需要显示的获取锁，并在finally块中显示的释放锁，目的是保证在获取到锁之后，最终能够被释放。

### 3. 非公平锁实现

在非公平锁中，每当线程执行lock方法时，都尝试利用CAS把state从0设置为1。

那么Doug lea是如何实现锁的非公平性呢？ 我们假设这样一个场景：

1. 持有锁的线程A正在running，队列中有线程BCDEF被挂起并等待被唤醒；
2. 在某一个时间点，线程A执行unlock，唤醒线程B；
3. 同时线程G执行lock，这个时候会发生什么？线程B和G拥有相同的优先级，这里讲的优先级是指获取锁的优先级，同时执行CAS指令竞争锁。如果恰好线程G成功了，线程B就得重新挂起等待被唤醒。

通过上述场景描述，我们可以看书，即使线程B等了很长时间也得和新来的线程G同时竞争锁，如此的不公平。

```java
static final class NonfairSync extends Sync {
    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    public final void acquire(int arg) {    
        if (!tryAcquire(arg) && 
                acquireQueued(addWaiter(Node.EXCLUSIVE), arg))       
          selfInterrupt();
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

下面我们用线程A和线程B来描述非公平锁的竞争过程。

1. 线程A和B同时执行CAS指令，假设线程A成功，线程B失败，则表明线程A成功获取锁，并把同步器中的exclusiveOwnerThread设置为线程A。

2. 竞争失败的线程B，在nonfairTryAcquire方法中，会再次尝试获取锁，

   在这段时间如果线程A释放锁，线程B就可以直接获取锁而不用挂起

   。完整的执行流程如下：

   [![img](https://camo.githubusercontent.com/f62090cd63608583c37306104a48b9e5bcf8abcf/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323138343935312d356264373033383239636264633536612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/f62090cd63608583c37306104a48b9e5bcf8abcf/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323138343935312d356264373033383239636264633536612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

### 4. 公平锁实现

在公平锁中，每当线程执行lock方法时，如果同步器的队列中有线程在等待，则直接加入到队列中。 场景分析：

1. 持有锁的线程A正在running，对列中有线程BCDEF被挂起并等待被唤醒；
2. 线程G执行lock，队列中有线程BCDEF在等待，线程G直接加入到队列的对尾。

所以每个线程获取锁的过程是公平的，等待时间最长的会最先被唤醒获取锁。

```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

### 5. 重入锁实现

重入锁，即线程可以重复获取已经持有的锁。在非公平和公平锁中，都对重入锁进行了实现。

```java
    if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
```

### 6. 条件变量Condition

条件变量很大一个程度上是为了解决Object.wait/notify/notifyAll难以使用的问题。

```java
public class ConditionObject implements Condition, java.io.Serializable {
    /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;
    public final void signal() {}
    public final void signalAll() {}
    public final void awaitUninterruptibly() {}  
    public final void await() throws InterruptedException {}
}
```

1. Synchronized中，所有的线程都在同一个object的条件队列上等待。而ReentrantLock中，每个condition都维护了一个条件队列。
2. 每一个**Lock**可以有任意数据的**Condition**对象，**Condition**是与**Lock**绑定的，所以就有**Lock**的公平性特性：如果是公平锁，线程为按照FIFO的顺序从*Condition.await*中释放，如果是非公平锁，那么后续的锁竞争就不保证FIFO顺序了。
3. Condition接口定义的方法，**await**对应于**Object.wait**，**signal**对应于**Object.notify**，**signalAll**对应于**Object.notifyAll**。特别说明的是Condition的接口改变名称就是为了避免与Object中的*wait/notify/notifyAll*的语义和使用上混淆。

先看一个condition在生产者消费者的应用场景：

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by j_zhan on 2016/7/13.
 */
public class Queue<T> {
    private final T[] items;
    private final Lock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();
    private int head, tail, count;
    public Queue(int maxSize) {
        items = (T[]) new Object[maxSize];
    }
    public Queue() {
        this(10);
    }

    public void put(T t) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) {
                //数组满时，线程进入等待队列挂起。线程被唤醒时，从这里返回。
                notFull.await(); 
            }
            items[tail] = t;
            if (++tail == items.length) {
                tail = 0;
            }
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();
            }
            T o = items[head];
            items[head] = null;//GC
            if (++head == items.length) {
                head = 0;
            }
            --count;
            notFull.signal();
            return o;
        } finally {
            lock.unlock();
        }
    }
}
```

假设线程AB在并发的往items中插入数据，当items中元素存满时。如果线程A获取到锁，继续添加数据，满足count == items.length条件，导致线程A执行await方法。 **ReentrantLock**是独占锁，同一时刻只有一个线程能获取到锁，所以在lock.lock()和lock.unlock()之间可能有一次释放锁的操作（同样也必然还有一次获取锁的操作）。在Queue类中，不管take还是put，在线程持有锁之后只有await()方法有可能释放锁，然后挂起线程，一旦条件满足就被唤醒，再次获取锁。具体实现如下：

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}

private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

await实现逻辑：

1. 将线程A加入到条件等待队列中，如果最后一个节点是取消状态，则从对列中删除。
2. 线程A释放锁，实质上是线程A修改AQS的状态state为0，并唤醒AQS等待队列中的线程B，线程B被唤醒后，尝试获取锁，接下去的过程就不重复说明了。
3. 线程A释放锁并唤醒线程B之后，如果线程A不在AQS的同步队列中，线程A将通过LockSupport.park进行挂起操作。
4. 随后，线程A等待被唤醒，当线程A被唤醒时，会通过acquireQueued方法竞争锁，如果失败，继续挂起。如果成功，线程A从await位置恢复。

假设线程B获取锁之后，执行了take操作和条件变量的signal，signal通过某种实现唤醒了线程A，具体实现如下：

```java
 public final void signal() {
     if (!isHeldExclusively())
         throw new IllegalMonitorStateException();
     Node first = firstWaiter;
     if (first != null)
         doSignal(first);
 }

 private void doSignal(Node first) {
     do {
         if ((firstWaiter = first.nextWaiter) == null)
             lastWaiter = null;
         first.nextWaiter = null;
     } while (!transferForSignal(first) &&
              (first = firstWaiter) != null);
 }

 final boolean transferForSignal(Node node) {
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
    Node p = enq(node); //线程A插入到AQS的等待队列中
    int ws = p.waitStatus;
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

signal实现逻辑：

1. 接着上述场景，线程B执行了signal方法，取出条件队列中的第一个非CANCELLED节点线程，即线程A。另外，signalAll就是唤醒条件队列中所有非CANCELLED节点线程。遇到CANCELLED线程就需要将其从队列中删除。
2. 通过CAS修改线程A的waitStatus，表示该节点已经不是等待条件状态，并将线程A插入到AQS的等待队列中。
3. 唤醒线程A，线程A和别的线程进行锁的竞争。

### 7. 总结

1. ReentrantLock提供了内置锁类似的功能和内存语义。
2. 此外，ReetrantLock还提供了其它功能，包括定时的锁等待、可中断的锁等待、公平性、以及实现非块结构的加锁、Condition，对线程的等待和唤醒等操作更加灵活，一个ReentrantLock可以有多个Condition实例，所以更有扩展性，不过ReetrantLock需要显示的获取锁，并在finally中释放锁，否则后果很严重。
3. ReentrantLock在性能上似乎优于Synchronized，其中在jdk1.6中略有胜出，在1.5中是远远胜出。那么为什么不放弃内置锁，并在新代码中都使用ReetrantLock？
4. 在java1.5中， 内置锁与ReentrantLock相比有例外一个优点：在线程转储中能给出在哪些调用帧中获得了哪些锁，并能够检测和识别发生死锁的线程。Reentrant的非块状特性任然意味着，获取锁的操作不能与特定的栈帧关联起来，而内置锁却可以。
5. 因为内置锁时JVM的内置属性，所以未来更可能提升synchronized而不是ReentrantLock的性能。例如对线程封闭的锁对象消除优化，通过增加锁粒度来消除内置锁的同步。



## 十、Java并发集合：ArrayBlockingQueue

### 1. Queue接口和BlockingQueue接口回顾

#### 1.1 Queue接口回顾

> 在Queue接口中，除了继承Collection接口中定义的方法外，它还分别额外地定义**插入、删除、查询**这3个操作，其中每一个操作都以两种不同的形式存在，每一种形式都对应着一个方法。

方法说明：

|             | Throws exception | Returns special value |
| ----------- | ---------------- | --------------------- |
| **Insert**  | add(e)           | offer(e)              |
| **Remove**  | remove()         | poll()                |
| **Examine** | element()        | peek()                |

- add方法在将一个元素插入到队列的尾部时，如果出现队列已经满了，那么就会抛出IllegalStateException,而使用offer方法时，如果队列满了，则添加失败,返回false,但并不会引发异常。

- remove方法是获取队列的头部元素并且删除，如果当队列为空时，那么就会抛出NoSuchElementException。而poll在队列为空时，则返回一个null。

- element方法是从队列中获取到队列的第一个元素，但不会删除，但是如果队列为空时，那么它就会抛出NoSuchElementException。peek方法与之类似，只是不会抛出异常，而是返回false。

#### 1.2 BlockingQueue接口回顾

> BlockingQueue是JDK1.5出现的接口，它在原来的Queue接口基础上提供了更多的额外功能：当获取队列中的头部元素时，如果队列为空，那么它将会使执行线程处于等待状态；当添加一个元素到队列的尾部时，如果队列已经满了，那么它同样会使执行的线程处于等待状态。

前面我们在说Queue接口时提到过，它针对于相同的操作提供了2种不同的形式，而BlockingQueue更夸张，针对于相同的操作提供了4种不同的形式。

> 该四种形式分别为：
>
> 1. 抛出异常
> 2. 返回一个特殊值(可能是null或者是false，取决于具体的操作)
> 3. 阻塞当前执行直到其可以继续
> 4. 当线程被挂起后，等待最大的时间，如果一旦超时，即使该操作依旧无法继续执行，线程也不会再继续等待下去。

对应的方法说明:

|             | Throws exception | Returns special value | Blocks | Times out            |
| ----------- | ---------------- | --------------------- | ------ | -------------------- |
| **Insert**  | add(e)           | offer(e)              | put(e) | offer(e, time, unit) |
| **Remove**  | remove()         | poll()                | take() | poll(time, unit)     |
| **Examine** | element()        | peek()                | 无     | 无                   |

BlockingQueue虽然比起Queue在操作上提供了更多的支持，但是它在使用有如下的几点:

> 1. BlockingQueue中是不允许添加null的，该接受在声明的时候就要求所有的实现类在接收到一个null的时候，都应该抛出NullPointerException。
> 2. BlockingQueue是线程安全的，因此它的所有和队列相关的方法都具有原子性。但是对于那么从Collection接口中继承而来的批量操作方法，比如addAll(Collection e)等方法，BlockingQueue的实现通常没有保证其具有原子性，因此我们在使用的BlockingQueue，应该尽可能地不去使用这些方法。
> 3. BlockingQueue主要应用于生产者与消费者的模型中，其元素的添加和获取都是极具规律性的。但是对于remove(Object o)这样的方法，虽然BlockingQueue可以保证元素正确的删除，但是这样的操作会非常响应性能，因此我们在没有特殊的情况下，也应该避免使用这类方法。

### 2. ArrayBlockingQueue深入分析

首先让我们看看API对其的描述。 **注意:这里使用的JDK版本为1.7，不同的JDK版本在实现上存在不同**

[![img](https://camo.githubusercontent.com/2ced146e8c65f5bca72e71c3e0fb0ad536927e45/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d636135303862383733356334303235372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/2ced146e8c65f5bca72e71c3e0fb0ad536927e45/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d636135303862383733356334303235372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

首先让我们看下ArrayBlockingQueue的核心组成：

```java
/** 底层维护队列元素的数组 */
    final Object[] items;

    /**  当读取元素时数组的下标(这里称为读下标) */
    int takeIndex;

    /** 添加元素时数组的下标 (这里称为写小标)*/
    int putIndex;

    /** 队列中的元素个数 */
    int count;

    /**用于并发控制的工具类**/
    final ReentrantLock lock;

    /** 控制take操作时是否让线程等待 */
    private final Condition notEmpty;

    /** 控制put操作时是否让线程等待 */
    private final Condition notFull;
```

> take方法分析

```java
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        /*
            尝试获取锁，如果此时锁被其他线程锁占用，那么当前线程就处于Waiting的状态。           
            注意:当方法是支持线程中断响应的如果其他线程此时中断当前线程，
            那么当前线程就会抛出InterruptedException 
         */
        lock.lockInterruptibly();
        try {
            /*
          如果此时队列中的元素个数为0,那么就让当前线程wait,并且释放锁。
          注意:这里使用了while进行重复检查，是为了防止当前线程可能由于其他未知的原因被唤醒。
          (通常这种情况被称为"spurious wakeup")
            */    
        while (count == 0)
                notEmpty.await();
            //如果队列不为空，则从队列的头部取元素
            return extract();
        } finally {
             //完成锁的释放
            lock.unlock();
        }
    }
```

> extract方法分析

```java
    /*
      根据takeIndex来获取当前的元素,然后通知其他等待的线程。
      Call only when holding lock.(只有当前线程已经持有了锁之后，它才能调用该方法)
     */
    private E extract() {
        final Object[] items = this.items;

        //根据takeIndex获取元素,因为元素是一个Object类型的数组,因此它通过cast方法将其转换成泛型。
        E x = this.<E>cast(items[takeIndex]);

        //将当前位置的元素设置为null
        items[takeIndex] = null;

        //并且将takeIndex++,注意：这里因为已经使用了锁，因此inc方法中没有使用到原子操作
        takeIndex = inc(takeIndex);

        //将队列中的总的元素减1
        --count;
        //唤醒其他等待的线程
        notFull.signal();
        return x;
    }
```

> put方法分析

```java
    public void put(E e) throws InterruptedException {
        //首先检查元素是否为空，否则抛出NullPointerException
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        //进行锁的抢占
        lock.lockInterruptibly();
        try {
            /*当队列的长度等于数组的长度,此时说明队列已经满了,这里同样
              使用了while来方式当前线程被"伪唤醒"。*/
            while (count == items.length)
                //则让当前线程处于等待状态
                notFull.await();
            //一旦获取到锁并且队列还未满时，则执行insert操作。
            insert(e);
        } finally {
            //完成锁的释放
            lock.unlock();
        }
    }

      //检查元素是否为空
     private static void checkNotNull(Object v) {
        if (v == null)
            throw new NullPointerException();
      }

     //该方法的逻辑非常简单
    private void insert(E x) {
        //将当前元素设置到putIndex位置   
        items[putIndex] = x;
        //让putIndex++
        putIndex = inc(putIndex);
        //将队列的大小加1
        ++count;
        //唤醒其他正在处于等待状态的线程
        notEmpty.signal();
    }
```

**注:ArrayBlockingQueue其实是一个循环队列** 我们使用一个图来简单说明一下:

> ```
> 黄色表示数组中有元素
> ```

[![img](https://camo.githubusercontent.com/91cd14f54005c9d7361d0bfd537506bcdf6ee1aa/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d373464643363663666643262653433322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/91cd14f54005c9d7361d0bfd537506bcdf6ee1aa/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d373464643363663666643262653433322e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

当再一次执行put的时候,其结果为：

[![img](https://camo.githubusercontent.com/2e721616910d1241b8bdacf90a86b36a13af6cdf/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d626462653366653466343364353133302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/2e721616910d1241b8bdacf90a86b36a13af6cdf/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d626462653366653466343364353133302e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

此时放入的元素会从头开始置，我们通过其incr方法更加清晰的看出其底层的操作：

```java
    /**
     * Circularly increment i.
     */
    final int inc(int i) {
        //当takeIndex的值等于数组的长度时,就会重新置为0，这个一个循环递增的过程
        return (++i == items.length) ? 0 : i;
    }
```

至此，ArrayBlockingQueue的核心部分就分析完了，其余的队列操作基本上都是换汤不换药的，此处不再一一列举。



## 十一、Java并发集合：LinkedBlockingQueue

LinkedBlockingQueue作为JDK中BlockingQueue家族系列中一员，由于其作为固定大小线程池(Executors.newFixedThreadPool())底层所使用的阻塞队列，分析它的目的主要在于2点：

1. 与ArrayBlockingQueue进行类比学习，加深各种数据结构的理解
2. 了解底层实现，能够更好地理解每一种阻塞队列对线程池性能的影响，做到真正的知其然，且知其所以然

### 1、LinkedBlockingQueue深入分析

LinkedBlockingQueue，见名之意，它是由一个基于链表的阻塞队列，首先看一下的核心组成：

```java
    // 所有的元素都通过Node这个静态内部类来进行存储，这与LinkedList的处理方式完全一样
    static class Node<E> {
        //使用item来保存元素本身
        E item;
        //保存当前节点的后继节点
        Node<E> next;
        Node(E x) { item = x; }
    }
    /**
        阻塞队列所能存储的最大容量
        用户可以在创建时手动指定最大容量,如果用户没有指定最大容量
        那么最默认的最大容量为Integer.MAX_VALUE.
    */
    private final int capacity;

    /** 
        当前阻塞队列中的元素数量
        PS:如果你看过ArrayBlockingQueue的源码,你会发现
        ArrayBlockingQueue底层保存元素数量使用的是一个
        普通的int类型变量。其原因是在ArrayBlockingQueue底层
        对于元素的入队列和出队列使用的是同一个lock对象。而数
        量的修改都是在处于线程获取锁的情况下进行操作，因此不
        会有线程安全问题。
        而LinkedBlockingQueue却不是，它的入队列和出队列使用的是两个    
        不同的lock对象,因此无论是在入队列还是出队列，都会涉及对元素数
        量的并发修改，(之后通过源码可以更加清楚地看到)因此这里使用了一个原子操作类
        来解决对同一个变量进行并发修改的线程安全问题。
    */
    private final AtomicInteger count = new AtomicInteger(0);

    /**
     * 链表的头部
     * LinkedBlockingQueue的头部具有一个不变性:
     * 头部的元素总是为null，head.item==null   
     */
    private transient Node<E> head;

    /**
     * 链表的尾部
     * LinkedBlockingQueue的尾部也具有一个不变性:
     * 即last.next==null
     */
    private transient Node<E> last;

    /**
     元素出队列时线程所获取的锁
     当执行take、poll等操作时线程需要获取的锁
    */
    private final ReentrantLock takeLock = new ReentrantLock();

    /**
    当队列为空时，通过该Condition让从队列中获取元素的线程处于等待状态
    */
    private final Condition notEmpty = takeLock.newCondition();

    /** 
      元素入队列时线程所获取的锁
      当执行add、put、offer等操作时线程需要获取锁
    */
    private final ReentrantLock putLock = new ReentrantLock();

    /** 
     当队列的元素已经达到capactiy，通过该Condition让元素入队列的线程处于等待状态
    */
    private final Condition notFull = putLock.newCondition();
```

通过上面的分析，我们可以发现LinkedBlockingQueue在入队列和出队列时使用的不是同一个Lock，这也意味着它们之间的操作不会存在互斥操作。在多个CPU的情况下，它们可以做到真正的在同一时刻既消费、又生产，能够做到并行处理。

下面让我们看下LinkedBlockingQueue的构造方法：

```java
    /**
     * 如果用户没有显示指定capacity的值，默认使用int的最大值
     */
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
    /**
     可以看到,当队列中没有任何元素的时候,此时队列的头部就等于队列的尾部,
     指向的是同一个节点,并且元素的内容为null
    */
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }

    /*
    在初始化LinkedBlockingQueue的时候，还可以直接将一个集合
    中的元素全部入队列，此时队列最大容量依然是int的最大值。
    */
    public LinkedBlockingQueue(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock putLock = this.putLock;
        //获取锁
        putLock.lock(); // Never contended, but necessary for visibility
        try {
            //迭代集合中的每一个元素,让其入队列,并且更新一下当前队列中的元素数量
            int n = 0;
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (n == capacity)
                    throw new IllegalStateException("Queue full");
                //参考下面的enqueue分析        
                enqueue(new Node<E>(e));
                ++n;
            }
            count.set(n);
        } finally {
            //释放锁
            putLock.unlock();
        }
    }

    /**
     * 等价于如下内容:
     * last.next=node;
     * last = node;
     * 其实也没有什么花样:
       就是让新入队列的元素成为原来的last的next，让进入的元素称为last
     *
     */
    private void enqueue(Node<E> node) {
        // assert putLock.isHeldByCurrentThread();
        // assert last.next == null;
        last = last.next = node;
    }
```

在分析完LinkedBlockingQueue的核心组成之后,下面让我们再看下核心的几个操作方法,首先分析一下元素入队列的过程:

```java
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        // Note: convention in all put/take/etc is to preset local var

        /*注意上面这句话,约定所有的put/take操作都会预先设置本地变量,
        可以看到下面有一个将putLock赋值给了一个局部变量的操作
        */
        int c = -1;
        Node<E> node = new Node(e);
        /* 
         在这里首先获取到putLock,以及当前队列的元素数量
         即上面所描述的预设置本地变量操作
        */
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        /*
            执行可中断的锁获取操作,即意味着如果线程由于获取
            锁而处于Blocked状态时，线程是可以被中断而不再继
            续等待，这也是一种避免死锁的一种方式，不会因为
            发现到死锁之后而由于无法中断线程最终只能重启应用。
        */
        putLock.lockInterruptibly();
        try {
            /*
            当队列的容量到底最大容量时,此时线程将处于等待状
            态，直到队列有空闲的位置才继续执行。使用while判
            断依旧是为了放置线程被"伪唤醒”而出现的情况,即当
            线程被唤醒时而队列的大小依旧等于capacity时，线
            程应该继续等待。
            */
            while (count.get() == capacity) {
                notFull.await();
            }
            //让元素进行队列的末尾,enqueue代码在上面分析过了
            enqueue(node);
            //首先获取原先队列中的元素个数,然后再对队列中的元素个数+1.
            c = count.getAndIncrement();
            /*注:c+1得到的结果是新元素入队列之后队列元素的总和。
            当前队列中的总元素个数小于最大容量时,此时唤醒其他执行入队列的线程
            让它们可以放入元素,如果新加入元素之后,队列的大小等于capacity，
            那么就意味着此时队列已经满了,也就没有必须要唤醒其他正在等待入队列的线程,因为唤醒它们之后，它们也还是继续等待。
            */
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            //完成对锁的释放
            putLock.unlock();
        }
        /*当c=0时，即意味着之前的队列是空队列,出队列的线程都处于等待状态，
        现在新添加了一个新的元素,即队列不再为空,因此它会唤醒正在等待获取元素的线程。
        */
        if (c == 0)
            signalNotEmpty();
    }

    /*
    唤醒正在等待获取元素的线程,告诉它们现在队列中有元素了
    */
    private void signalNotEmpty() {
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lock();
        try {
            //通过notEmpty唤醒获取元素的线程
            notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
    }
```

看完put方法，下面再看看下offer是如何处理的方法：

```java
    /**
    在BlockingQueue接口中除了定义put方法外(当队列元素满了之后就会阻塞，
    直到队列有新的空间可以方法线程才会继续执行)，还定义一个offer方法，
    该方法会返回一个boolean值，当入队列成功返回true,入队列失败返回false。
    该方法与put方法基本操作基本一致，只是有细微的差异。
    */
     public boolean offer(E e) {
        if (e == null) throw new NullPointerException();
        final AtomicInteger count = this.count;
        /*
            当队列已经满了，它不会继续等待,而是直接返回。
            因此该方法是非阻塞的。
        */
        if (count.get() == capacity)
            return false;
        int c = -1;
        Node<E> node = new Node(e);
        final ReentrantLock putLock = this.putLock;
        putLock.lock();
        try {
            /*
            当获取到锁时，需要进行二次的检查,因为可能当队列的大小为capacity-1时，
            两个线程同时去抢占锁，而只有一个线程抢占成功，那么此时
            当线程将元素入队列后，释放锁，后面的线程抢占锁之后，此时队列
            大小已经达到capacity，所以将它无法让元素入队列。
            下面的其余操作和put都一样，此处不再详述
            */
            if (count.get() < capacity) {
                enqueue(node);
                c = count.getAndIncrement();
                if (c + 1 < capacity)
                    notFull.signal();
            }
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
        return c >= 0;
    }
```

BlockingQueue还定义了一个限时等待插入操作，即在等待一定的时间内，如果队列有空间可以插入，那么就将元素入队列，然后返回true,如果在过完指定的时间后依旧没有空间可以插入，那么就返回false，下面是限时等待操作的分析:

```java
        /**
         通过timeout和TimeUnit来指定等待的时长
         timeout为时间的长度,TimeUnit为时间的单位
        */
        public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        if (e == null) throw new NullPointerException();
        //将指定的时间长度转换为毫秒来进行处理
        long nanos = unit.toNanos(timeout);
        int c = -1;
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
                //如果等待的剩余时间小于等于0，那么直接返回
                if (nanos <= 0)
                    return false;
            /*
              通过condition来完成等待，此时当前线程会完成锁的，并且处于等待状态
              直到被其他线程唤醒该线程、或者当前线程被中断、
              等待的时间截至才会返回，该返回值为从方法调用到返回所经历的时长。
              注意：上面的代码是condition的awitNanos()方法的通用写法，
              可以参看Condition.awaitNaos的API文档。
              下面的其余操作和put都一样，此处不再详述
            */
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(new Node<E>(e));
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
        return true;
    }
```

通过上面的分析，我们应该比较清楚地知道了LinkedBlockingQueue的入队列的操作，其主要是通过获取到putLock锁来完成，当队列的数量达到最大值，此时会导致线程处于阻塞状态或者返回false(根据具体的方法来看)；如果队列还有剩余的空间，那么此时会新创建出一个Node对象，将其设置到队列的尾部，作为LinkedBlockingQueue的last元素。

在分析完入队列的过程之后，我们接下来看看LinkedBlockingQueue出队列的过程；由于BlockingQueue的方法都具有对称性，此处就只分析take方法的实现，其余方法的实现都如出一辙：

```java
 public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        //通过takeLock获取锁，并且支持线程中断
        takeLock.lockInterruptibly();
        try {
            //当队列为空时，则让当前线程处于等待
            while (count.get() == 0) {
                notEmpty.await();
            }
            //完成元素的出队列
            x = dequeue();
            /*            
               队列元素个数完成原子化操作-1,可以看到count元素会
               在插入元素的线程和获取元素的线程进行并发修改操作。
            */
            c = count.getAndDecrement();
            /*
              当一个元素出队列之后，队列的大小依旧大于1时
              当前线程会唤醒其他执行元素出队列的线程,让它们也
              可以执行元素的获取
            */
            if (c > 1)
                notEmpty.signal();
        } finally {
            //完成锁的释放
            takeLock.unlock();
        }
        /*
            当c==capaitcy时，即在获取当前元素之前，
            队列已经满了，而此时获取元素之后，队列就会
            空出一个位置，故当前线程会唤醒执行插入操作的线
            程通知其他中的一个可以进行插入操作。
        */
        if (c == capacity)
            signalNotFull();
        return x;
    }


    /**
     * 让头部元素出队列的过程
     * 其最终的目的是让原来的head被GC回收，让其的next成为head
     * 并且新的head的item为null.
     * 因为LinkedBlockingQueue的头部具有一致性:即元素为null。
     */
    private E dequeue() {
        Node<E> h = head;
        Node<E> first = h.next;
        h.next = h; // help GC
        head = first;
        E x = first.item;
        first.item = null;
        return x;
    }
```

[![img](https://camo.githubusercontent.com/40bbf1e02471cf9ac84dae5efbe4e73f9d77ef08/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d396265613330653239306138383232632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/40bbf1e02471cf9ac84dae5efbe4e73f9d77ef08/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313638343337302d396265613330653239306138383232632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

### 2. LinkedBlockingQueue与ArrayBlockingQueue的比较

ArrayBlockingQueue由于其底层基于数组，并且在创建时指定存储的大小，在完成后就会立即在内存分配固定大小容量的数组元素，因此其存储通常有限，故其是一个**“有界“**的阻塞队列；

而LinkedBlockingQueue可以由用户指定最大存储容量，也可以无需指定，如果不指定则最大存储容量将是Integer.MAX_VALUE，即可以看作是一个**“无界”**的阻塞队列，由于其节点的创建都是动态创建，并且在节点出队列后可以被GC所回收，因此其具有灵活的伸缩性。但是由于ArrayBlockingQueue的有界性，因此其能够更好的对于性能进行预测，而LinkedBlockingQueue由于没有限制大小，当任务非常多的时候，不停地向队列中存储，就有可能导致内存溢出的情况发生。

其次，ArrayBlockingQueue中在入队列和出队列操作过程中，使用的是同一个lock，所以即使在多核CPU的情况下，其读取和操作的都无法做到并行，而LinkedBlockingQueue的读取和插入操作所使用的锁是两个不同的lock，它们之间的操作互相不受干扰，因此两种操作可以并行完成，故LinkedBlockingQueue的吞吐量要高于ArrayBlockingQueue。

### 3. 选择LinkedBlockingQueue的理由

```java
    /**
        下面的代码是Executors创建固定大小线程池的代码，其使用了
        LinkedBlockingQueue来作为任务队列。
    */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

JDK中选用LinkedBlockingQueue作为阻塞队列的原因就在于其**无界性**。因为线程大小固定的线程池，其线程的数量是不具备伸缩性的，当任务非常繁忙的时候，就势必会导致所有的线程都处于工作状态，如果使用一个有界的阻塞队列来进行处理，那么就非常有可能很快导致队列满的情况发生，从而导致任务无法提交而抛出RejectedExecutionException，而使用无界队列由于其良好的存储容量的伸缩性，可以很好的去缓冲任务繁忙情况下场景，即使任务非常多，也可以进行动态扩容，当任务被处理完成之后，队列中的节点也会被随之被GC回收，非常灵活。

## 十二、Java并发集合：ConcurrentHashMap

HashMap是我们平时开发过程中用的比较多的集合，但它是非线程安全的，在涉及到多线程并发的情况，进行put操作有可能会引起死循环，导致CPU利用率接近100%。

```java
final HashMap<String, String> map = new HashMap<String, String>(2);
for (int i = 0; i < 10000; i++) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            map.put(UUID.randomUUID().toString(), "");
        }
    }).start();
}
```

解决方案有Hashtable和Collections.synchronizedMap(hashMap)，不过这两个方案基本上是对读写进行加锁操作，一个线程在读写元素，其余线程必须等待，性能可想而知。

所以，Doug Lea给我们带来了并发安全的ConcurrentHashMap，它的实现是依赖于 Java 内存模型，所以我们在了解 ConcurrentHashMap 的之前必须了解一些底层的知识：

1. java内存模型
2. java中的CAS
3. AbstractQueuedSynchronizer
4. ReentrantLock

本文源码是JDK8的版本，与之前的版本有较大差异。

### 1. JDK1.6分析

ConcurrentHashMap采用 分段锁的机制，实现并发的更新操作，底层采用数组+链表+红黑树的存储结构。 其包含两个核心静态内部类 Segment和HashEntry。

1. Segment继承ReentrantLock用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶。
2. HashEntry 用来封装映射表的键 / 值对；
3. 每个桶是由若干个 HashEntry 对象链接起来的链表。

一个 ConcurrentHashMap 实例中包含由若干个 Segment 对象组成的数组，下面我们通过一个图来演示一下 ConcurrentHashMap 的结构：

[![img](https://camo.githubusercontent.com/f907a168e28a2ed0421e7878df61123fbde4045b/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323138343935312d373238633331396634386562653335612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/f907a168e28a2ed0421e7878df61123fbde4045b/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323138343935312d373238633331396634386562653335612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

### 2. JDK1.8分析

1.8的实现已经抛弃了Segment分段锁机制，利用CAS+Synchronized来保证并发更新的安全，底层依然采用数组+链表+红黑树的存储结构。

[![img](https://camo.githubusercontent.com/b95a7f90cf8d557de4f89bb5e035aa8a7d0ffe32/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323138343935312d336432333635636135393936323734662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/b95a7f90cf8d557de4f89bb5e035aa8a7d0ffe32/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f323138343935312d336432333635636135393936323734662e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

#### 2.1 重要概念

在开始之前，有些重要的概念需要介绍一下：

1. **table**：默认为null，初始化发生在第一次插入操作，默认大小为16的数组，用来存储Node节点数据，扩容时大小总是2的幂次方。

2. **nextTable**：默认为null，扩容时新生成的数组，其大小为原数组的两倍。

3. sizeCtl

   ：默认为0，用来控制table的初始化和扩容操作，具体应用在后续会体现出来。

   - **-1 **代表table正在初始化
   - **-N **表示有N-1个线程正在进行扩容操作
   - 其余情况： 1、如果table未初始化，表示table需要初始化的大小。 2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍，居然用这个公式算0.75（n - (n >>> 2)）。

4. Node

   ：保存key，value及key的hash值的数据结构。

   ```java
   class Node<K,V> implements Map.Entry<K,V> {
     final int hash;
     final K key;
     volatile V val;
     volatile Node<K,V> next;
     ... 省略部分代码
   }
   ```

   其中value和next都用volatile修饰，保证并发的可见性。

5. ForwardingNode

   ：一个特殊的Node节点，hash值为-1，其中存储nextTable的引用。

   ```java
   final class ForwardingNode<K,V> extends Node<K,V> {
     final Node<K,V>[] nextTable;
     ForwardingNode(Node<K,V>[] tab) {
         super(MOVED, null, null, null);
         this.nextTable = tab;
     }
   }
   ```

   只有table发生扩容的时候，ForwardingNode才会发挥作用，作为一个占位符放在table中表示当前节点为null或则已经被移动。

#### 2.2 实例初始化

实例化ConcurrentHashMap时带参数时，会根据参数调整table的大小，假设参数为100，最终会调整成256，确保table的大小总是2的幂次方，算法如下：

```java
ConcurrentHashMap<String, String> hashMap = new ConcurrentHashMap<>(100);
private static final int tableSizeFor(int c) {
    int n = c - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

**注意**，ConcurrentHashMap在构造函数中只会初始化sizeCtl值，并不会直接初始化table，而是延缓到第一次put操作。

#### 2.3 table初始化

前面已经提到过，table初始化操作会延缓到第一次put行为。但是put是可以并发执行的，Doug Lea是如何实现table只初始化一次的？让我们来看看源码的实现。

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
//如果一个线程发现sizeCtl<0，意味着另外的线程执行CAS操作成功，当前线程只需要让出cpu时间片
        if ((sc = sizeCtl) < 0) 
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

sizeCtl默认为0，如果ConcurrentHashMap实例化时有传参数，sizeCtl会是一个2的幂次方的值。所以执行第一次put操作的线程会执行Unsafe.compareAndSwapInt方法修改sizeCtl为-1，有且只有一个线程能够修改成功，其它线程通过Thread.yield()让出CPU时间片等待table初始化完成。

#### 2.4 put操作

假设table已经初始化完成，put操作采用CAS+synchronized实现并发插入或更新操作，具体实现如下。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        ...省略部分代码
    }
    addCount(1L, binCount);
    return null;
}
```

1. hash算法

   ```
   static final int spread(int h) {return (h ^ (h >>> 16)) & HASH_BITS;}
   ```

2. table中定位索引位置，n是table的大小

   ```
   int index = (n - 1) & hash
   ```

3. 获取table中对应索引的元素f。 Doug Lea采用Unsafe.getObjectVolatile来获取，也许有人质疑，直接table[index]不可以么，为什么要这么复杂？ 在java内存模型中，我们已经知道每个线程都有一个工作内存，里面存储着table的副本，虽然table是volatile修饰的，但不能保证线程每次都拿到table中的最新元素，Unsafe.getObjectVolatile可以直接获取指定内存的数据，保证了每次拿到数据都是最新的。

4. 如果f为null，说明table中这个位置第一次插入元素，利用Unsafe.compareAndSwapObject方法插入Node节点。

   - 如果CAS成功，说明Node节点已经插入，随后addCount(1L, binCount)方法会检查当前容量是否需要进行扩容。
   - 如果CAS失败，说明有其它线程提前插入了节点，自旋重新尝试在这个位置插入节点。

5. 如果f的hash值为-1，说明当前f是ForwardingNode节点，意味有其它线程正在扩容，则一起进行扩容操作。

6. 其余情况把新的Node节点按链表或红黑树的方式插入到合适的位置，这个过程采用同步内置锁实现并发，代码如下:

   ```java
   synchronized (f) {
    if (tabAt(tab, i) == f) {
        if (fh >= 0) {
            binCount = 1;
            for (Node<K,V> e = f;; ++binCount) {
                K ek;
                if (e.hash == hash &&
                    ((ek = e.key) == key ||
                     (ek != null && key.equals(ek)))) {
                    oldVal = e.val;
                    if (!onlyIfAbsent)
                        e.val = value;
                    break;
                }
                Node<K,V> pred = e;
                if ((e = e.next) == null) {
                    pred.next = new Node<K,V>(hash, key,
                                              value, null);
                    break;
                }
            }
        }
        else if (f instanceof TreeBin) {
            Node<K,V> p;
            binCount = 2;
            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                           value)) != null) {
                oldVal = p.val;
                if (!onlyIfAbsent)
                    p.val = value;
            }
        }
    }
   }
   ```

   在节点f上进行同步，节点插入之前，再次利用tabAt(tab, i) == f判断，防止被其它线程修改。

   1. 如果f.hash >= 0，说明f是链表结构的头结点，遍历链表，如果找到对应的node节点，则修改value，否则在链表尾部加入节点。
   2. 如果f是TreeBin类型节点，说明f是红黑树根节点，则在树结构上遍历元素，更新或增加节点。
   3. 如果链表中节点数binCount >= TREEIFY_THRESHOLD(默认是8)，则把链表转化为红黑树结构。

#### 2.5 table扩容

当table容量不足的时候，即table的元素数量达到容量阈值sizeCtl，需要对table进行扩容。 整个扩容分为两部分：

1. 构建一个nextTable，大小为table的两倍。
2. 把table的数据复制到nextTable中。

这两个过程在单线程下实现很简单，但是ConcurrentHashMap是支持并发插入的，扩容操作自然也会有并发的出现，这种情况下，第二步可以支持节点的并发复制，这样性能自然提升不少，但实现的复杂度也上升了一个台阶。

先看第一步，构建nextTable，毫无疑问，这个过程只能只有单个线程进行nextTable的初始化，具体实现如下：

```java
private final void addCount(long x, int check) {
    ... 省略部分代码
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```

通过Unsafe.compareAndSwapInt修改sizeCtl值，保证只有一个线程能够初始化nextTable，扩容后的数组长度为原来的两倍，但是容量是原来的1.5。

节点从table移动到nextTable，大体思想是遍历、复制的过程。

1. 首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd。
2. 如果f == null，则在table中的i位置放入fwd，这个过程是采用Unsafe.compareAndSwapObjectf方法实现的，很巧妙的实现了节点的并发移动。
3. 如果f是链表的头节点，就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上，移动完成，采用Unsafe.putObjectVolatile方法给table原位置赋值fwd。
4. 如果f是TreeBin节点，也做一个反序处理，并判断是否需要untreeify，把处理的结果分别放在nextTable的i和i+n的位置上，移动完成，同样采用Unsafe.putObjectVolatile方法给table原位置赋值fwd。

遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。

#### 2.6 红黑树构造

**注意**：如果链表结构中元素超过TREEIFY_THRESHOLD阈值，默认为8个，则把链表转化为红黑树，提高遍历查询效率。

```java
if (binCount != 0) {
    if (binCount >= TREEIFY_THRESHOLD)
        treeifyBin(tab, i);
    if (oldVal != null)
        return oldVal;
    break;
}
```

接下来我们看看如何构造树结构，代码如下：

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

可以看出，生成树节点的代码块是同步的，进入同步代码块之后，再次验证table中index位置元素是否被修改过。 1、根据table中index位置Node链表，重新生成一个hd为头结点的TreeNode链表。 2、根据hd头结点，生成TreeBin树结构，并把树结构的root节点写到table的index位置的内存中，具体实现如下：

```java
TreeBin(TreeNode<K,V> b) {
    super(TREEBIN, null, null, null);
    this.first = b;
    TreeNode<K,V> r = null;
    for (TreeNode<K,V> x = b, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        if (r == null) {
            x.parent = null;
            x.red = false;
            r = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = r;;) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);
                    TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    r = balanceInsertion(r, x);
                    break;
                }
            }
        }
    }
    this.root = r;
    assert checkInvariants(root);
}
```

主要根据Node节点的hash值大小构建二叉树。这个红黑树的构造过程实在有点复杂，感兴趣的同学可以看看源码。

#### 2.7 get操作

get操作和put操作相比，显得简单了许多。

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

1. 判断table是否为空，如果为空，直接返回null。
2. 计算key的hash值，并获取指定table中指定位置的Node节点，通过遍历链表或则树结构找到对应的节点，返回value值。

### 3. 总结

ConcurrentHashMap 是一个并发散列映射表的实现，它允许完全并发的读取，并且支持给定数量的并发更新。相比于 HashTable 和同步包装器包装的 HashMap，使用一个全局的锁来同步不同线程间的并发访问，同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器，这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。