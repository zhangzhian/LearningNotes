# Java

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
|            |       |                |

## 基础

### 一、Object
### 二、String
### 三、继承
### 四、注解
### 五、序列化
### 六、集合
### 七、泛型
### 八、反射机制
### 九、IO
### 十、深拷贝和浅拷贝

### 十一、其他

## 并发
### 一、线程Thread
### 二、线程池
### 三、锁
### 四、Synchronized
### 五、ReentranLock
### 六、线程间通信
### 七、Java内存模型
### 八、阻塞队列

### 九、其他

## JVM
### 一、Java内存模型
### 二、类加载器
### 三、垃圾回收









#### 1. 两个值相等的 Integer 对象，== 比较，判断是否相等

Integer对象值的范围为 `[-128 - 127]` 时，不会创建新的引用，指向同一个地址，地址值相等。

但是当超过这个范围后，就会 new 一个对象，引用指向的地址是不同的，地址值不相等。

#### synchronized 修饰 static 方法、普通方法、类、方法块区别

#### synchronized 底层实现原理

#### volatile 的作用和原理

#### 一个 int 变量用 volatile 修饰，多线程去操作 i++，是否线程安全？如何保证 i++ 线程安全？AtomicInteger 的底层实现原理？

使用 AtomicInteger 可以使 i++ 线程安全

#### 说下对线程池的理解，以及创建线程池的几个关键参数

#### Java 集合，介绍下ArrayList 和 HashMap 的使用场景，底层实现原理

#### ArrayList 与 LinkedList 的区别



#### 下面的代码， str 值最终为多少？换成 Integer 值又为多少，是否会被改变？

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



#### 下面的代码，再次使用对象 student 是否需要判空？

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

public changeValue1(Student student) {
    student = null;
}

public static void changeValue2(Student student)  {    
     student.name = "Lily";    
     student.age = 20;
}
```

#### Java 的几种引用类型，弱引用的使用场景？