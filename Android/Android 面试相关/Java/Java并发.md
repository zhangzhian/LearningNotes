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



## 九、深入理解ReentrantLock



## 十、Java并发集合：ArrayBlockingQueue



## 十一、Java并发集合：LinkedBlockingQueue



## 十二、Java并发集合：ConcurrentHashMap

