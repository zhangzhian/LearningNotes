# 设计模式

| 时间       | 版本  | 说明 |
| ---------- | ----- | ---- |
| 2020.10.15 | 0.0.1 | 初创 |
|            |       |      |

#### 1. 常见的设计模式有哪些？用了哪些设计模式？

重点了解以下的几种常用的设计模式：

- 单例模式

- 工厂模式和抽象工厂模式：注意他们的区别。
- 责任链模式：View的事件分发和OkHttp的调用过程都使用到了责任链模式。
- 观察者模式：重要性不言而喻。
- 代理模式：建议了解一下动态代理。

#### 2. 六大原则

设计模式的六大原则是：

- 单一职责：合理分配类和函数的职责
- 开闭原则：开放扩展，关闭修改
- 里式替换：继承
- 依赖倒置：面向接口
- 接口隔离：控制接口的粒度
- 迪米特：一个类应该对其他的类了解最少

#### 3. 单例模式

单例模式被问到的几率很大，通常会问如下几种问题。

#### 4. 单例的常用写法有哪几种？

**懒汉模式**

```
public class SingleInstance {
	private static SingleInstance instance;
	private SingleInstance() {}
	public static synchronized SingleInstance getInstance() {
		if(instance == null) {
			instance = new SingleInstance();
		}
		return instance;
	}
}
```

该模式的主要问题是每次获取实例都需要同步，造成不必要的同步开销。 **DCL模式**

```
public class SingleInstance {
	private static SingleInstance instance;
	private SingleInstance() {}
	public static SingleInstance getInstance() {
		if(instance == null) {
			synchronized (SingleInstance.class) {
				if(instance == null) {
					instance = new SingleInstance();
				}
			}
		}
		return instance;
	}
}
```

高并发环境下可能会发生问题。 **静态内部类单例**

```
public class SingleInstance {
	private SingleInstance() {}
	public static SingleInstance getInstance() {
		return SingleHolder.instance;
	}
	
	private static class SingleHolder{
		private static final SingleInstance instance = new SingleInstance();
	}
}
```

**枚举单例**

```
public enum SingletonEnum {
	INSTANCE
}
```

优点：线程安全和反序列化不会生成新的实例

#### 5. DCL模式会有什么问题？

对象生成实例的过程中，大概会经过以下过程：

1. 为对象分配内存空间。
2. 初始化对象中的成员变量。
3. 将对象指向分配的内存空间（此时对象就不为null）。

由于Jvm会优化指令顺序，也就是说2和3的顺序是不能保证的。在多线程的情况下，当一个线程完成了1、3过程后，当前线程的时间片已用完，这个时候会切换到另一个线程，另一个线程调用这个单例，会使用这个还没初始化完成的实例。 解决方法是使用`volatile`关键字：

```
public class SingleInstance {
	private static volatile SingleInstance instance;
	private SingleInstance() {}
	public static SingleInstance getInstance() {
		if(instance == null) {
			synchronized (SingleInstance.class) {
				if(instance == null) {
					instance = new SingleInstance();
				}
			}
		}
		return instance;
	}
}
```

#### 6. Android源码中有哪些设计模式