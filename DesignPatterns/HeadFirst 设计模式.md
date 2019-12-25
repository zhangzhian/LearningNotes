#设计模式

《Head First 设计模式》读书笔记。

## 零、策略模式

设计原则：找出应用中可能需要变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起。

设计原则：针对接口编程，而不是针对实现编程。

设计原则：多用组合，少用继承。

良好的OO设计必须具备：可复用，可扩充，可维护三个特性。           

策略模式：定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让那个算法的变化独立于使用算法的客户。

![](https://img-blog.csdnimg.cn/2019030212151566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3R1Z2FuZ2thaQ==,size_16,color_FFFFFF,t_70)

```java

//策略接口
public interface IStrategy {
    //定义的抽象算法方法 来约束具体的算法实现方法
    public void algorithmMethod();
}

// 具体的策略实现
public class ConcreteStrategy implements IStrategy {
    //具体的算法实现
    @Override
    public void algorithmMethod() {
        System.out.println("this is ConcreteStrategy method...");
    }
}

// 具体的策略实现2
public class ConcreteStrategy2 implements IStrategy {
     //具体的算法实现
    @Override
    public void algorithmMethod() {
        System.out.println("this is ConcreteStrategy2 method...");
    }
}

/**
 * 策略上下文
 */
public class StrategyContext {
    //持有一个策略实现的引用
    private IStrategy strategy;
    //使用构造器注入具体的策略类
    public StrategyContext(IStrategy strategy) {
        this.strategy = strategy;
    }
 
    public void contextMethod(){
        //调用策略实现的方法
        strategy.algorithmMethod();
    }
}

//外部客户端
public class Client {
    public static void main(String[] args) {
        //1.创建具体测策略实现
        IStrategy strategy = new ConcreteStrategy2();
        //2.在创建策略上下文的同时，将具体的策略实现对象注入到策略上下文当中
        StrategyContext ctx = new StrategyContext(strategy);
        //3.调用上下文对象的方法来完成对具体策略实现的回调
        ctx.contextMethod();
    }
}
```



## 一、观察者模式

**定义:**
观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新


观察者模式提供了一种对象设计，让主题和观察者之间松耦合

对于观察者的一切，主题只知道观察者实现了某个接口

设计原则：为了交互对象之间的松耦合设计而努力

松耦合的设计之所以能让我们建立有弹性的OO系统，能够应对变化，是因为对象之间的互相依赖降到了最低

使用观察者模式时，可以从被观察者处push或pull数据

observable有多个观察者时，不可以依赖特定的通知顺序，observable实现采用的是继承，存在一些问题

![](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161A6221S.gif)



```java
public interface Subject {
		void attach(Observer observer);
		void detach(Observer observer);
		void notifyObserver(String newState);
}

public class ConcreteSubject implements Subject {
    //保存注册的观察者对象
		private List<Observer> mObervers = new ArrayList<>();

		//注册观察者对象
		public void attach(Observer observer) {
				mObervers.add(observer);
		}

		//注销观察者对象
		public void detach(Observer observer) {
				mObervers.remove(observer);
		}

		//通知所有注册的观察者对象
		public void notifyObserver(String newState) {
				for (Observer observer : mObervers) {
						observer.update(newState);
				}
		}
}

public interface Observer {
    void update(String newState);
}

public class ObserverA implements Observer {

    //观察者状态
    private String observerState;

    @Override
    public void update(String newState) {
        //更新观察者状态，让它与目标状态一致
        observerState = newState;
        System.out.println("接收到消息：" + newState + "；我是A模块！！");
    }
}

public class ObserverB implements Observer {
    //观察者状态
    private String observerState;

    @Override
    public void update(String newState) {
        //更新观察者状态，让它与目标状态一致
        observerState = newState;
        System.out.println("接收到消息：" + newState + "；我是B模块！！");
    }
}

public class ObserverC implements Observer {
    //观察者状态
    private String observerState;

    @Override
    public void update(String newState) {
        //更新观察者状态，让它与目标状态一致
        observerState = newState;
        System.out.println("接收到消息：" + newState + "；我是C模块！！");
    }
}

public class ObserverTest
{
    public static void main(String[] args)
    {
        Subject subject=new ConcreteSubject();
        Observer obs1=new ObserverA();
        Observer obs2=new ObserverB();
        Observer obs3=new ObserverC();
        subject.attach(obs1);
        subject.attach(obs2);
        subject.attach(obs3);
        subject.notifyObserver("123");
    }
}

```



## 二、装饰者模式

**定义：**

装饰者模式动态地将责任附加到对象上。若要扩展此功能，装饰者提供了比继承更有弹性的代替方案。

在程序设计中，应该允许行代码可以被扩展，而无需修改现有的代码

组合和委托可在允许时动态地加上新的行为

装饰者模式意味着一群装饰者类，这些类用来包装具体的组件

装饰者反映出被装饰的组件类型（事实上，他们具有相同的类型，都经过接口或继承实现）。

装饰者可以在被装饰者的行为前面或后面加上自己的行为，甚至将被装饰者行为整个取代掉，而达到特定的目的

可以用无数个装饰者包装一个组件

装饰者一般对组件的客户是透明的，除非客户程序依赖与组件的具体类型

装饰者会导致设计中出现许多小对象，过度使用，会让程序变的很复杂

设计原则：类应该对扩展开放，对修改关闭

![](https://img-blog.csdn.net/20140818201029976?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhzaHVsaW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

```java
// 男人
public interface Man {
    public void getManDesc();
}

// 普通男人
public class NormalMan implements Man{
    private String name = null;
    
    public NormalMan(String name) {
        this.name = name;
    }
    
    @Override
    public void getManDesc() {
        System.out.print(name + ": ");
    }
}

// 附加属性装饰者
public abstract class AttachedPropertiesDecorator implements Man{
    private Man man;
    
    public AttachedPropertiesDecorator(Man man) {
        this.man = man;
    }
    
    public void getManDesc() {
        man.getManDesc();
    }
}

// 小车装饰者
public class CarDecorator extends AttachedPropertiesDecorator{
    private String car = "有车";
    
    public CarDecoratorImpl(Man man) {
        super(man);
    }
    
    public void addCar() {
        System.out.print(car + " ");
    }
    
    @Override
    public void getManDesc() {
        super.getManDesc();
        addCar();
    }
}

// 房子装饰者
public class HouseDecorator extends AttachedPropertiesDecorator{
    private String house = "有房";
    
    public HouseDecoratorImpl(Man man) {
        super(man);
    }
    
    public void addHouse() {
        System.out.print(house + " ");
    }
    
    @Override
    public void getManDesc() {
        super.getManDesc();
        addHouse();
    }
}

// 存款装饰者
public class DepositDecorator extends AttachedPropertiesDecorator{
    private String deposit = "有存款";
    
    public DepositDecoratorImpl(Man man) {
        super(man);
    }
    
    public void addDeposit() {
        System.out.print(deposit + " ");
    }
    
    @Override
    public void getManDesc() {
        super.getManDesc();
        addDeposit();
    }
}

public class DecoratorTest {

    public static void main(String[] args) {
        Man man = new NormalMan("张三");
        man = new CarDecorator(man);
        man = new HouseDecorator(man);
        man = new DepositDecorator(man);
        System.out.println("层层装饰:");
        man.getManDesc();
    }
}
```



## 三、工厂模式

分为：简单工厂模式，工厂模式和抽象工厂模式

6简单工厂模式其实不是一种设计模式，反而比较像是一种编程习惯，但仍可以将客户程序从具体实现解耦

工厂方法用来处理对象的创建，并将这样的行为封装在子类，客户程序中关于超类的代码就和子类对象创建代码解耦类

所有工厂模式都是用来封装对象的创建，都通过减少应用程序和具体类之间的依赖促进松耦合

工厂模式方法通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的

工厂方法使用继承：把对象的创建委托给子类，子类实现工厂方法来创建对象

**工厂方法模式**：定义了一个创建对象的接口，但由于子类决定要实例化的类是哪一个。工厂方法让类把实例化到子类。

依赖倒置原则：要依赖抽象，不要依赖具体的类

避免OO设计中违反依赖倒置原则的指导方针：

- 变量不可以持有具体的引用
- 不要让类派生自具体类
- 不要覆盖基类中以实现的方法

**抽象工厂模式**：提供了一个接口，用于创建相关或依赖对象的家族，而不需要明确制定具体类

抽象工厂使用对象耦合：对象的创建被实现在工厂接口所暴露出来的方法中

https://segmentfault.com/a/1190000019485423?utm_source=tag-newest

## 四、单例模式

单例模式：确保一个类只有一个实例，并提供一个全局访问点

多线程下的单例模式：

- 如果`getInstance()`的性能对应用不是很关键，就什么也别做
- 使用“饿汉式”创建实例，而不用“懒汉式”实例化的做法

```java
public class Singleton {
	private static Singleton uniqueInstance = new Singleton();

  private Singleton() {}

  public static Singleton getInstance() {
    return uniqueInstance;
  }
}
```



- 用“双重检查锁”，在`getInstance()`中减少使用同步

```java
public class Singleton {
	private volatile static Singleton uniqueInstance;
 
	private Singleton() {}
 
	public static Singleton getInstance() {
		if (uniqueInstance == null) {
			synchronized (Singleton.class) {
				if (uniqueInstance == null) {
					uniqueInstance = new Singleton();
				}
			}
		}
		return uniqueInstance;
	}
}

```

## 五、命令模式

封装调用

定义：将请求封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

空对象：当不想返回一个有意义的对象，可以使用空对象，将处理null的责任转移给空对象。

当命令支持撤销时，该命令必须提供和execute()方法相反的方法undo() 方法

宏命令：在命令中添加命令的集合，是命令的一种简单延伸，允许调用多个命令。支持注销

命令模式将发出请求的对象和执行请求的对象接偶，在被解耦的两者之间时通过命令对象进行沟通的，命令对象封装了接收者和一个或一组动作

调用者通过调用命令对象的execute（）发出请求，这会使接收者的动作被调用

调用者可以接受命令当做参数，甚至可以在运行时动态进行

实际操作时，直接实现请求，而非将工作直接委托给接收者

![](http://c.biancheng.net/uploads/allimg/181116/3-1Q11611335E44.gif)

```java
public class CommandPattern
{
    public static void main(String[] args)
    {
        Command cmd=new ConcreteCommand();
        Invoker ir=new Invoker(cmd);
        System.out.println("客户访问调用者的call()方法...");
        ir.call();
    }
}
//调用者
class Invoker
{
    private Command command;
    public Invoker(Command command)
    {
        this.command=command;
    }
    public void setCommand(Command command)
    {
        this.command=command;
    }
    public void call()
    {
        System.out.println("调用者执行命令command...");
        command.execute();
    }
}
//抽象命令
interface Command
{
    public abstract void execute();
}
//具体命令
class ConcreteCommand implements Command
{
    private Receiver receiver;
    ConcreteCommand()
    {
        receiver=new Receiver();
    }
    public void execute()
    {
        receiver.action();
    }
}
//接收者
class Receiver
{
    public void action()
    {
        System.out.println("接收者的action()方法被调用...");
    }
}
```



![](https://www.runoob.com/wp-content/uploads/2014/08/command_pattern_uml_diagram.jpg)

```java
public interface Order {
   void execute();
}
public class Stock {
   
   private String name = "ABC";
   private int quantity = 10;
 
   public void buy(){
      System.out.println("Stock [ Name: "+name+", 
         Quantity: " + quantity +" ] bought");
   }
   public void sell(){
      System.out.println("Stock [ Name: "+name+", 
         Quantity: " + quantity +" ] sold");
   }
}
public class BuyStock implements Order {
   private Stock abcStock;
 
   public BuyStock(Stock abcStock){
      this.abcStock = abcStock;
   }
 
   public void execute() {
      abcStock.buy();
   }
}
public class SellStock implements Order {
   private Stock abcStock;
 
   public SellStock(Stock abcStock){
      this.abcStock = abcStock;
   }
 
   public void execute() {
      abcStock.sell();
   }
}
 
public class Broker {
   private List<Order> orderList = new ArrayList<Order>(); 
 
   public void takeOrder(Order order){
      orderList.add(order);      
   }
 
   public void placeOrders(){
      for (Order order : orderList) {
         order.execute();
      }
      orderList.clear();
   }
}
                         
public class CommandPatternDemo {
   public static void main(String[] args) {
      Stock abcStock = new Stock();
 
      BuyStock buyStockOrder = new BuyStock(abcStock);
      SellStock sellStockOrder = new SellStock(abcStock);
 
      Broker broker = new Broker();
      broker.takeOrder(buyStockOrder);
      broker.takeOrder(sellStockOrder);
 
      broker.placeOrders();
   }
}                         
```



## 六、适配器模式与外观模式

适配器模式将一个类的接口，转化成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。

外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。

意图：

装饰者模式：不改变接口，但加入责任。

适配器模式：将一个接口转换成为另一个接口。

外观模式：让接口更接单。

适配器模式和外观模式的主要差异在意图上。而不在包装类的数量上，适配器是转化接口，外观模式是提供子系统的更简单的接口，从复杂的子系统中解耦。

最少知识原则：只和你的“密友”谈话。设计一个系统，不管是任何对象，都需要注意交互的类有哪些，是如何交互的，不要让太多的类耦合在一起，减少对象之间的交互。

适配器模式有两种形态：对象适配器和类适配器。类适配器用到多重继承（不适用于Java）。



##七、模版方法模式

模版方法定义了一个算法的步骤，并允许字类为一个或多个步骤提供实现。

模版方法模式在一个方法中定义了一个算法的骨架，而将一些步骤延迟到子类中。模版方法使得字类可以在不改变算法结构的情况下，重信定义算法中的某些步骤。

钩子（hook）是一种被声明在抽象类中的方法，但只有控的或者默认的实现。可以让子类有能力对算法的不同点进行挂钩。

好莱坞原则：别调用我们，我们会调用你。允许低层组件将自己挂钩到系统上，但是高层组件会决定when和how使用这些低层组件。

模版方法为我们提供一种代码复用的重要技巧。

模版方法的抽象类可以定义具体方法，抽象方法和钩子。

为了防止子类改变模版方法中的算法，可以将模版发放声明final。

策略模式和模版方法模式都封装算法，一个用组合，一个基础。

工厂方法是版模版方法的一种特殊版本。



## 八、迭代器与组合模式

迭代器模式：提供类一种方法访问一个聚合对象中的各个元素，而又不暴露其内部的表示。

迭代器将遍历聚合的工作封装到一个对象中。

当使用迭代器的时候，我们依赖聚合提供遍历。

设计原则：一个类应该只有一个引起变化的原因。

当一个模块或一个类被设计成只支持一组相关功能时，具有高内聚；反之，被设计成支持一组不想管功能时，具有低内聚。

组合模式：允许你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。

组合模式以单一设计原则获取透明性。

组合结构内的任意对象成为组件，组件可以时组合也可以时叶节点。

## 九、状态模式

状态模式：允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。

和程序状态机PSM不同，状态模式用类代表状态。

context会将行为委托给当前状态对象。

通过将每个状态分装进一个类，我们把以后需要做的任务改变局部化了。

状态模式和策略模式有相同的类图，但是它们的意图不同。

策略模式通常会用行为或算法来配置context类。

状态模式允许Context随状态的改变而改变行为。

状态转换可以由State类或Context类控制。

使用状态模式通常会导致设计中类的数目大量增加。

状态类可以被多Context实例共享。把每个状态都指定到静态的实例变量中。



## 十、代理模式

代理模式：为另一个对象提供一个替身或占位符以控制对这个对象的访问。

使用代理模式创建代表对象，让代表对象控制某对象的访问，对代理的对象可以时远程的对象、创建开销大的对象或事需要安全控制的对象。

代理在结构上类似装饰者，但目的不同。装饰者为对象加上行为，而代理则是控制访问。

远程代理控制访问远程对象。

虚拟代理控制访问创建开销大的资源。

保护代理基于权限控制对资源的访问。

动态代理：Java在java.lang.reflect中有自己的代理支持，利用这个包可以在运行时动态创建一个代理类，实现一个或者多个接口，并将方法的调用转发到指定的类。

防火墙代理：控制网络资源的访问，保护主题免于侵害。

智能代理：当主题被引用时，进行额外的动作，例如计算一个对象引用的次数。

缓存代理：为开销大的运算结果提供暂时存储，允许多个客户共享结果，以减少计算或网络延迟。

同步代理：在多线程的情况下为主题提供安全的访问。

复杂隐藏代理：用来隐藏一个类的复杂集合的复杂度，并进行访问控制。有时也称外观代理。

写入时复杂代理：用来控制对象的复制，方法是延迟对象的复制，知道客户真的需要为止。这是虚拟代理的变体。

##十一、复合模式

模式通常被一起使用，并被组合在一个设计解决方案中

复合模式在一个解决方案中结合两个或多个模式，以解决一般或者重复发生的问题。

MVC是复合模式，结合了观察者模式、策略模式和组合模式。

模型使用观察者模式，以便观察者更新，同时保持两者之间的解耦。

视图使用组合模式实现用户界面，用户界面通常组合了嵌套的组件。

适配器模式用来将新的模型适配成已有的视图和控制器。

Model 2是MVC在Web上的应用。



##十二、其他模式

### 1. 桥接模式

桥接模式(Bridge Pattern)：将抽象部分与它的实现部分分离，使它们都可以独立地变化。

桥接模式，不只改变实现，还能改变抽象

优点：将实现解耦；抽象和实现可以独立扩展；对于“具体的抽象类”所做的改变，不会影响到客户

用途：适合使用载需要跨越多个平台的图形和窗口系统上；当需要用于不同的方式改变接口和实现时

缺点：增加了复杂度

![](https://img2018.cnblogs.com/blog/1475571/201901/1475571-20190112180526113-1204626425.png)



```java
public interface Implementor
{
    public void operationImpl();
} 

public abstract class Abstraction
{
    protected Implementor impl;
    
    public void setImpl(Implementor impl)
    {
        this.impl=impl;
    }
    
    public abstract void operation();
}

public class RefinedAbstraction extends Abstraction
{
    public void operation()
    {
        //代码
        impl.operationImpl();
        //代码
    }
}
```

![](https://img2018.cnblogs.com/blog/1475571/201901/1475571-20190112180712208-505786819.png)



```java
//抽象类
public abstract class Pen
{
    protected Color color;
    public void setColor(Color color)
    {
        this.color=color;
    }
    public abstract void draw(String name);
} 

//扩充抽象类
public class SmallPen extends Pen
{
    public void draw(String name)
    {
        String penType="小号毛笔绘制";
        this.color.bepaint(penType,name);            
    }    
}

//扩充抽象类
public class MiddlePen extends Pen
{
    public void draw(String name)
    {
        String penType="中号毛笔绘制";
        this.color.bepaint(penType,name);            
    }    
}

//扩充抽象类
public class BigPen extends Pen
{
    public void draw(String name)
    {
        String penType="大号毛笔绘制";
        this.color.bepaint(penType,name);            
    }    
}

//实现类接口
public interface Color
{
    void bepaint(String penType,String name);
}

//扩充实现类
public class Red implements Color
{
    public void bepaint(String penType,String name)
    {
        System.out.println(penType + "红色的"+ name + ".");
    }
}

//扩充实现类
public class Green implements Color
{
    public void bepaint(String penType,String name)
    {
        System.out.println(penType + "绿色的"+ name + ".");
    }
}

//扩充实现类
public class Blue implements Color
{
    public void bepaint(String penType,String name)
    {
        System.out.println(penType + "蓝色的"+ name + ".");
    }
}

//扩充实现类
public class White implements Color
{
    public void bepaint(String penType,String name)
    {
        System.out.println(penType + "白色的"+ name + ".");
    }
}

//扩充实现类
public class Black implements Color
{
    public void bepaint(String penType,String name)
    {
        System.out.println(penType + "黑色的"+ name + ".");
    }
}

//客户端
public class Client
{
    public static void main(String a[])
    {
        Color color;
        Pen pen;
        
        color = new Blue();
        pen = new SmallPen();
        
        pen.setColor(color);
        pen.draw("鲜花");
    }
}
```



### 2. 生成器模式

使用生成器模式封装一个产品的构造过程，并允许按步骤构造

优点：将一个复杂对象的创建过程封装起来；允许对象通过多个步骤创建，并且可以改变过程；向客户隐藏产品内部的表现；产品的实现可以被替换，因为客户只看到一个抽象的接口

用途：经常被用来创建组合结构

缺点：与工厂模式相比，采用生成器创建对象的客户，需要更多的领域知识

![](https://img-blog.csdnimg.cn/20190603112106513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rhbmd5dXpoaWRhbw==,size_16,color_FFFFFF,t_70)

```java
//抽象类或接口
public abstract class Builder {

    public abstract void buildPartA();
    public abstract void buildPartB();
    public abstract Product getBuildResult();
    
}
//指挥者类，用来指挥建造过程
public class Director {
    public void construct(Builder builder) {
	    builder.buildPartA();
	    builder.buildPartB();
    }
}
//具体建造者类
public class ConcreteBuilder1 extends Builder {
    private Product product = new Product();

    @Override
    public void buildPartA() {
	    product.add("部件A");
    }

    @Override
    public void buildPartB() {
	    product.add("部件B");
    }

    @Override
    public Product getBuildResult() {
	    return product;
    }
}
// 具体建造者类，建造的对象时Product，通过build使Product完善
public class ConcreteBuilder2 extends Builder {
    private Product product = new Product();

    @Override
    public void buildPartA() {
	    product.add("部件X");
    }

    @Override
    public void buildPartB() {
	    product.add("部件Y");
    }

    @Override
    public Product getBuildResult() {
	   return product;
    }
}
//产品类，由多个部件组成
public class Product {
    List<String> parts = new ArrayList<String>();

    // 添加产品部件
    public void add(String part) {
	    parts.add(part);
    }

    // 列举所有的产品部件
    public void show() {
	    System.out.println("---产品 创建---");

		for (String part : parts) {
		    System.out.println(part);
		}
    }
}
//建造客户端
public class BuilderClient {

    public static void main(String[] args) {
	    Director director = new Director();
	    Builder builder1 = new ConcreteBuilder1();
	    Builder builder2 = new ConcreteBuilder2();

      director.construct(builder1);
      Product product = builder1.getBuildResult();
      product.show();
      
    }
}
```

### 3. 责任链模式

当你想要让一个以上的对象有机会能够处理某个请求的时候，就使用责任链模式。

优点：将请求的发送者和接受者解耦；可以简化你的对象，因为不需要知道链的机构；通过改变链内的成员或调动它们的次序，允许动态地新增或者删除责任

用途：经常被使用在窗口系统中，处理鼠标和键盘之类的事件；

缺点：并不保证请求一定被执行；如果哪有任何对象处理它的话，可能落到链尾短之外；可能不容易观察运行时的特征，有碍于除错

![](https://upload-images.jianshu.io/upload_images/4807654-f9ce0fd7b19be529.png?imageMogr2/auto-orient/strip|imageView2/2/w/994)

```java
/**
 * 抽象处理器
 */
public abstract class Handler {
    //下一个处理器
    private Handler nextHandler;

    //处理方法
    public abstract void handleRequest();

    public Handler getNextHandler() {
        return nextHandler;
    }
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }
}

/**
 * 具体处理器.
 */
public class ConcreteHandler extends Handler {

    @Override
    public void handleRequest() {
        System.out.println(this.toString()+"处理器处理");
        if (getNextHandler()!=null){   //判断是否存在下一个处理器
            getNextHandler().handleRequest();   //存在则调用下一个处理器
        }
    }

}

/**
 * 测试
 */
public class Client {
    public static void main(String[] args) {
        Handler h1 = new ConcreteHandler();
        Handler h2 = new ConcreteHandler();
        h1.setNextHandler(h2);   //h1的下一个处理器是h2
        h1.handleRequest();
    }
}
```



![](https://www.runoob.com/wp-content/uploads/2014/08/chain_pattern_uml_diagram.jpg)

```java
/**
 * 抽象处理器.
 */
public abstract class AbstractLogger {
    public static final int INFO = 1;    //一级日志
    public static final int DEBUG = 2;   //二级日志包括一级
    public static final int ERROR = 3;   //三级包括前两个

    protected int level;
    //责任链下一个元素
    protected AbstractLogger nextLogger ;
    public void setNextLogger(AbstractLogger nextLogger){
        this.nextLogger = nextLogger;
    }

    //不同级别的记录方法不一样,这里给一个抽象的记录方法
    abstract protected void write(String message);

    //调用责任链处理器的记录方法.并且判断下一个责任链元素是否存在,若存在,则执行下一个方法.
    public void logMessage(int level,String message){
        if (this.level <= level){    //根据传进来的日志等级,判断哪些责任链元素要去记录
            write(message);
        }
        if (nextLogger != null){
            nextLogger.logMessage(level,message);   //进行下一个责任链元素处理
        }
    }
}

/**
 * 控制台处理器.
 */
public class ConsoleLogger extends AbstractLogger {
    public ConsoleLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("Standard Console::Logger :"+message);
    }
}

/**
 * 文件处理器.
 */
public class FileLogger extends AbstractLogger {
    public FileLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("File Console::Logger"+message);
    }
}

/**
 * error日志处理器.
 */
public class ErrorLogger extends AbstractLogger {
    public ErrorLogger(int level) {
        this.level = level;
    }

    @Override
    protected void write(String message) {
        System.out.println("Error Console::Logger: " + message);
    }
}

/**
 * 处理链.
 */
public class ChainPatternDemo {

    public static AbstractLogger getChainOfLoggers() {

        AbstractLogger errorLogger = new ErrorLogger(AbstractLogger.ERROR);
        AbstractLogger fileLogger = new FileLogger(AbstractLogger.DEBUG);
        AbstractLogger consoleLogger = new ConsoleLogger(AbstractLogger.INFO);

        errorLogger.setNextLogger(fileLogger);
        fileLogger.setNextLogger(consoleLogger);

        return  errorLogger;
    }
}

public class Main {
    public static void main(String[] args) {
        AbstractLogger logger = ChainPatternDemo.getChainOfLoggers();
        logger.logMessage(1,"一级日志记录");
        System.out.println("--------------------------------");
        logger.logMessage(2,"二级日志记录");
        System.out.println("--------------------------------");
        logger.logMessage(3,"三级日志记录");
    }
}
```

### 4. 蝇量模式（享元模式）

如果相让某个类的一个实例能用来提供许多“虚拟实例”，就使用蝇量模式

优点：减少运行时对象实例的个数，节省内存；将需多“虚拟”对象的状态集中管理

用途：当一个类游许多的实例，而这些实例能被同一方法控制的时候，我们就可以使用蝇量模式

缺点：一旦实现了它，单个逻辑实例将无法拥有独立而不同的行为

![](http://c.biancheng.net/uploads/allimg/181115/3-1Q115161342242.gif)

```java
public class FlyweightPattern
{
    public static void main(String[] args)
    {
        FlyweightFactory factory=new FlyweightFactory();
        Flyweight f01=factory.getFlyweight("a");
        Flyweight f02=factory.getFlyweight("a");
        Flyweight f03=factory.getFlyweight("a");
        Flyweight f11=factory.getFlyweight("b");
        Flyweight f12=factory.getFlyweight("b");       
        f01.operation(new UnsharedConcreteFlyweight("第1次调用a。"));       
        f02.operation(new UnsharedConcreteFlyweight("第2次调用a。"));       
        f03.operation(new UnsharedConcreteFlyweight("第3次调用a。"));       
        f11.operation(new UnsharedConcreteFlyweight("第1次调用b。"));       
        f12.operation(new UnsharedConcreteFlyweight("第2次调用b。"));
    }
}
//非享元角色
class UnsharedConcreteFlyweight
{
    private String info;
    UnsharedConcreteFlyweight(String info)
    {
        this.info=info;
    }
    public String getInfo()
    {
        return info;
    }
    public void setInfo(String info)
    {
        this.info=info;
    }
}
//抽象享元角色
interface Flyweight
{
    public void operation(UnsharedConcreteFlyweight state);
}
//具体享元角色
class ConcreteFlyweight implements Flyweight
{
    private String key;
    ConcreteFlyweight(String key)
    {
        this.key=key;
        System.out.println("具体享元"+key+"被创建！");
    }
    public void operation(UnsharedConcreteFlyweight outState)
    {
        System.out.print("具体享元"+key+"被调用，");
        System.out.println("非享元信息是:"+outState.getInfo());
    }
}
//享元工厂角色
class FlyweightFactory
{
    private HashMap<String, Flyweight> flyweights=new HashMap<String, Flyweight>();
    public Flyweight getFlyweight(String key)
    {
        Flyweight flyweight=(Flyweight)flyweights.get(key);
        if(flyweight!=null)
        {
            System.out.println("具体享元"+key+"已经存在，被成功获取！");
        }
        else
        {
            flyweight=new ConcreteFlyweight(key);
            flyweights.put(key, flyweight);
        }
        return flyweight;
    }
}
```

![](https://www.runoob.com/wp-content/uploads/2014/08/flyweight_pattern_uml_diagram-1.jpg)

```java
public interface Shape {
   void draw();
}

public class Circle implements Shape {
   private String color;
   private int x;
   private int y;
   private int radius;
 
   public Circle(String color){
      this.color = color;     
   }
 
   public void setX(int x) {
      this.x = x;
   }
 
   public void setY(int y) {
      this.y = y;
   }
 
   public void setRadius(int radius) {
      this.radius = radius;
   }
 
   @Override
   public void draw() {
      System.out.println("Circle: Draw() [Color : " + color 
         +", x : " + x +", y :" + y +", radius :" + radius);
   }
}

public class ShapeFactory {
   private static final HashMap<String, Shape> circleMap = new HashMap<>();
 
   public static Shape getCircle(String color) {
      Circle circle = (Circle)circleMap.get(color);
 
      if(circle == null) {
         circle = new Circle(color);
         circleMap.put(color, circle);
         System.out.println("Creating circle of color : " + color);
      }
      return circle;
   }
}

public class FlyweightPatternDemo {
   private static final String colors[] = 
      { "Red", "Green", "Blue", "White", "Black" };
   public static void main(String[] args) {
 
      for(int i=0; i < 20; ++i) {
         Circle circle = 
            (Circle)ShapeFactory.getCircle(getRandomColor());
         circle.setX(getRandomX());
         circle.setY(getRandomY());
         circle.setRadius(100);
         circle.draw();
      }
   }
   private static String getRandomColor() {
      return colors[(int)(Math.random()*colors.length)];
   }
   private static int getRandomX() {
      return (int)(Math.random()*100 );
   }
   private static int getRandomY() {
      return (int)(Math.random()*100);
   }
}
```



### 5. 解释器模式

使用解释器模式为语言创建解释器

优点：将每一个语法规则表示成一个类，方便实现语言；因为语法由许多类表示，所以可以轻松改变或扩展此语言；通过在类的结构中加入新的方法，可以在解释的同时增加新的行为，例如打印格式的美化或者进行复杂的程序验证

用途：实现一个简单语言；有一个简单语法，而且简单比效率重要；可以处理脚本语言和编程语言

缺点：语法规则的数目太大时，这个模式可能会变得非常繁杂。在这种情况下，使用解析器/编译器的产生器可能更适合。

![](http://c.biancheng.net/uploads/allimg/181119/3-1Q119150626422.gif)

```java
//抽象表达式类
interface AbstractExpression
{
    public Object interpret(String info);    //解释方法
}
//终结符表达式类
class TerminalExpression implements AbstractExpression
{
    public Object interpret(String info)
    {
        //对终结符表达式的处理
    }
}
//非终结符表达式类
class NonterminalExpression implements AbstractExpression
{
    private AbstractExpression exp1;
    private AbstractExpression exp2;
    public Object interpret(String info)
    {
        //非对终结符表达式的处理
    }
}
//环境类
class Context
{
    private AbstractExpression exp;
    public Context()
    {
        //数据初始化
    }
    public void operation(String info)
    {
        //调用相关表达式类的解释方法
    }
}
```

【例】用解释器模式设计一个“韶粵通”公交车卡的读卡器程序。
说明：假如“韶粵通”公交车读卡器可以判断乘客的身份，如果是“韶关”或者“广州”的“老人” “妇女”“儿童”就可以免费乘车，其他人员乘车一次扣 2 元。

```
<expression> ::= <city>的<person>
<city> ::= 韶关|广州
<person> ::= 老人|妇女|儿童
```

![](http://c.biancheng.net/uploads/allimg/181119/3-1Q119150Q6401.gif)

```java
/*文法规则
  <expression> ::= <city>的<person>
  <city> ::= 韶关|广州
  <person> ::= 老人|妇女|儿童
*/
public class InterpreterPatternDemo
{
    public static void main(String[] args)
    {
        Context bus=new Context();
        bus.freeRide("韶关的老人");
        bus.freeRide("韶关的年轻人");
        bus.freeRide("广州的妇女");
        bus.freeRide("广州的儿童");
        bus.freeRide("山东的儿童");
    }
}
//抽象表达式类
interface Expression
{
    public boolean interpret(String info);
}
//终结符表达式类
class TerminalExpression implements Expression
{
    private Set<String> set= new HashSet<String>();
    public TerminalExpression(String[] data)
    {
        for(int i=0;i<data.length;i++)set.add(data[i]);
    }
    public boolean interpret(String info)
    {
        if(set.contains(info))
        {
            return true;
        }
        return false;
    }
}
//非终结符表达式类
class AndExpression implements Expression
{
    private Expression city=null;    
    private Expression person=null;
    public AndExpression(Expression city,Expression person)
    {
        this.city=city;
        this.person=person;
    }
    public boolean interpret(String info)
    {
        String s[]=info.split("的");       
        return city.interpret(s[0])&&person.interpret(s[1]);
    }
}
//环境类
class Context
{
    private String[] citys={"韶关","广州"};
    private String[] persons={"老人","妇女","儿童"};
    private Expression cityPerson;
    public Context()
    {
        Expression city=new TerminalExpression(citys);
        Expression person=new TerminalExpression(persons);
        cityPerson=new AndExpression(city,person);
    }
    public void freeRide(String info)
    {
        boolean ok=cityPerson.interpret(info);
        if(ok) System.out.println("您是"+info+"，您本次乘车免费！");
        else System.out.println(info+"，您不是免费人员，本次乘车扣费2元！");   
    }
}
```

### 6. 中介者模式

使用中介者模式来集中相关对象之间复杂的沟通和控制方式

优点：通过将对象彼此解耦，可以增加对象的复用性；通过将控制逻辑集中，可以简化系统维护；可以让对象之间所传递的消息变得简单而且大幅度减少

用途：中介者常常被用来协调相关的GUI组件

缺点：如果设计不当，中介者对象本身会变得过于复杂

![](http://c.biancheng.net/uploads/allimg/181116/3-1Q1161I532V0.gif)

```java
import java.util.ArrayList;
import java.util.List;

public class MediatorPattern
{
    public static void main(String[] args)
    {
        Mediator md=new ConcreteMediator();
        Colleague c1,c2;
        c1=new ConcreteColleague1();
        c2=new ConcreteColleague2();
        md.register(c1);
        md.register(c2);
        c1.send();
        System.out.println("-------------");
        c2.send();
    }
}
//抽象中介者
abstract class Mediator
{
    public abstract void register(Colleague colleague);
    public abstract void relay(Colleague cl); //转发
}
//具体中介者
class ConcreteMediator extends Mediator
{
    private List<Colleague> colleagues=new ArrayList<Colleague>();
    public void register(Colleague colleague)
    {
        if(!colleagues.contains(colleague))
        {
            colleagues.add(colleague);
            colleague.setMediator(this);
        }
    }
    public void relay(Colleague cl)
    {
        for(Colleague ob:colleagues)
        {
            if(!ob.equals(cl))
            {
                ((Colleague)ob).receive();
            }
        }
    }
}
//抽象同事类
abstract class Colleague
{
    protected Mediator mediator;
    public void setMediator(Mediator mediator)
    {
        this.mediator=mediator;
    }
    public abstract void receive();
    public abstract void send();
}
//具体同事类
class ConcreteColleague1 extends Colleague
{
    public void receive()
    {
        System.out.println("具体同事类1收到请求。");
    }
    public void send()
    {
        System.out.println("具体同事类1发出请求。");
        mediator.relay(this); //请中介者转发
    }
}
//具体同事类
class ConcreteColleague2 extends Colleague
{
    public void receive()
    {
        System.out.println("具体同事类2收到请求。");
    }
    public void send()
    {
        System.out.println("具体同事类2发出请求。");
        mediator.relay(this); //请中介者转发
    }
}
```

### 7. 备忘录模式

当你需要让对象返回之前的状态时，就使用备忘录模式

优点：将被存储的状态放在外面，不要和关键对象混在一起，这样可以帮助维护内聚；保持关键对象的数据封装；提供了容易实现的恢复能力

用途：备忘录用户存储状态

缺点：使用备忘录时，存储和恢复状态的过程可能相当耗时；在Java系统中可以考虑使用序列化机制存储系统状态

![](http://c.biancheng.net/uploads/allimg/181119/3-1Q119130413927.gif)



```java
public class MementoPattern
{
    public static void main(String[] args)
    {
        Originator or=new Originator();
        Caretaker cr=new Caretaker();       
        or.setState("S0"); 
        System.out.println("初始状态:"+or.getState());           
        cr.setMemento(or.createMemento()); //保存状态      
        or.setState("S1"); 
        System.out.println("新的状态:"+or.getState());        
        or.restoreMemento(cr.getMemento()); //恢复状态
        System.out.println("恢复状态:"+or.getState());
    }
}
//备忘录
class Memento
{ 
    private String state; 
    public Memento(String state)
    { 
        this.state=state; 
    }     
    public void setState(String state)
    { 
        this.state=state; 
    }
    public String getState()
    { 
        return state; 
    }
}
//发起人
class Originator
{ 
    private String state;     
    public void setState(String state)
    { 
        this.state=state; 
    }
    public String getState()
    { 
        return state; 
    }
    public Memento createMemento()
    { 
        return new Memento(state); 
    } 
    public void restoreMemento(Memento m)
    { 
        this.setState(m.getState()); 
    } 
}
//管理者
class Caretaker
{ 
    private Memento memento;       
    public void setMemento(Memento m)
    { 
        memento=m; 
    }
    public Memento getMemento()
    { 
        return memento; 
    }
}
```

### 8. 原型模式

### 9. 访问者模式

















