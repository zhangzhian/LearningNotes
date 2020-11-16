# 设计模式

《Head First 设计模式》读书备忘笔记，整合了一些网上资料，记录了所有常见的算法模式实现的示例。适合有一定基础的读者，若无基础建议先看《Head First 设计模式》，是一本很好的入门资料。

- **设计原则**：找出应用中可能需要变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起。

- **设计原则**：针对接口编程，而不是针对实现编程。

- **设计原则**：多用组合，少用继承。
- **设计原则**：为了交互对象之间的松耦合设计而努力。

- **设计原则**：类应该对扩展开放，对修改关闭。
- **设计原则**：要依赖抽象，不要依赖具体类。
- **设计原则：**一个类应该只有一个引起变化的原因。

良好的OO设计必须具备：可复用，可扩充，可维护三个特性。设计模式可以让我们建造出具有良好OO设计质量的系统。

模式不是代码，而是针对设计问题的同意解决方案，你可以将其应用到特点应用中。

模式不是被发明，而是发现。

松耦合的设计之所以能让我们建立有弹性的OO系统，能够应对变化，是因为对象之间的互相依赖降到了最低。

在程序设计中，应该允许行为可以被扩展，而无需修改现有的代码。

组合和委托可在运行时动态地加上新的行为。

**依赖倒置原则**：要依赖抽象，不要依赖具体的类。

避免OO设计中违反依赖倒置原则的指导方针：

- 变量不可以持有具体的引用
- 不要让类派生自具体类
- 不要覆盖基类中已实现的方法

**最少知识原则（迪米特法则）**：只和你的“密友”谈话。设计一个系统，不管是任何对象，都需要注意交互的类有哪些，是如何交互的，不要让太多的类耦合在一起，减少对象之间的交互。

**开闭原则**：当应用的需求改变时，在不修改软件实体的源代码或者二进制代码的前提下，可以扩展模块的功能，使其满足新的需求。

**好莱坞原则**：别调用我们，我们会调用你。允许低层组件将自己挂钩到系统上，但是高层组件会决定when和how使用这些低层组件。

当一个模块或一个类被设计成只支持一组相关功能时，具有高内聚；反之，被设计成支持一组不想管功能时，具有低内聚。

## 零、策略模式

**策略模式定义**：定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

> 策略模式属于对象行为模式，它通过对算法进行封装，把使用算法的责任和算法的实现分割开来，并委派给不同的对象对这些算法进行管理。

**主要优点：**

1. 多重条件语句不易维护，而使用策略模式可以避免使用多重条件语句。
2. 策略模式提供了一系列的可供重用的算法族，恰当使用继承可以把算法族的公共代码转移到父类里面，从而避免重复的代码。
3. 策略模式可以提供相同行为的不同实现，客户可以根据不同时间或空间要求选择不同的。
4. 策略模式提供了对开闭原则的完美支持，可以在不修改原代码的情况下，灵活增加新算法。
5. 策略模式把算法的使用放到环境类中，而算法的实现移到具体策略类中，实现了二者的分离。

**主要缺点：**

1. 客户端必须理解所有策略算法的区别，以便适时选择恰当的算法类。
2. 策略模式造成很多的策略类。

**使用场景：** 

1. 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 

2. 一个系统需要动态地在几种算法中选择一种。 

3. 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。

**应用示例：** 

1. 诸葛亮的锦囊妙计，每一个锦囊就是一个策略。 

2. 旅行的出游方式，选择骑自行车、坐汽车，每一种旅行方式都是一个策略。

**策略模式的结构与实现**

1. 抽象策略（Strategy）类：定义了一个公共接口，各种不同的算法以不同的方式实现这个接口，环境角色使用这个接口调用不同的算法，一般使用接口或抽象类实现。
2. 具体策略（Concrete Strategy）类：实现了抽象策略定义的接口，提供具体的算法实现。
3. 环境（Context）类：持有一个策略类的引用，最终给客户端调用。

![策略模式的结构图](http://c.biancheng.net/uploads/allimg/181116/3-1Q116103K1205.gif)

```java
//策略接口
public interface IStrategy {
    //定义的抽象算法方法 来约束具体的算法实现方法
    public void algorithmMethod();
}

// 具体的策略实现
public class ConcreteStrategy1 implements IStrategy {
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

**定义：**观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

> 观察者模式提供了一种对象设计，让主题和观察者之间松耦合。关于观察者的一切，主题只知道观察者实现了某个接口。使用观察者模式时，可以从被观察者处push或pull数据。Java内置的observable有多个观察者时，不可以依赖特定的通知顺序，observable实现采用的是继承，存在一些问题。

**主要优点**：

1. 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系。
2. 目标与观察者之间建立了一套触发机制。

**主要缺点**：

1. 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用。
2. 当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。

**应用示例：**

1. 拍卖的时候，拍卖师观察最高标价，然后通知给其他竞价者竞价。

2. 当“人民币汇率”升值时，进口公司的进口产品成本降低且利润率提升，出口公司的出口产品收入降低且利润率降低；当“人民币汇率”贬值时，进口公司的进口产品成本提升且利润率降低，出口公司的出口产品收入提升且利润率提升。

**使用场景：**

- 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
- 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
- 一个对象必须通知其他对象，而并不知道这些对象是谁。
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。

**模式的结构与实现**

实现观察者模式时要注意具体目标对象和具体观察者对象之间不能直接调用，否则将使两者之间紧密耦合起来，这违反了面向对象的设计原则。

1. 抽象主题（Subject）角色：也叫抽象目标类，它提供了一个用于保存观察者对象的聚集类和增加、删除观察者对象的方法，以及通知所有观察者的抽象方法。
2. 具体主题（Concrete  Subject）角色：也叫具体目标类，它实现抽象目标中的通知方法，当具体主题的内部状态发生改变时，通知所有注册过的观察者对象。
3. 抽象观察者（Observer）角色：它是一个抽象类或接口，它包含了一个更新自己的抽象方法，当接到具体主题的更改通知时被调用。
4. 具体观察者（Concrete Observer）角色：实现抽象观察者中定义的抽象方法，以便在得到目标的更改通知时更新自身的状态。



![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTYvMy0xUTExNjFBNjIyMVMuZ2lm)



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

**定义：**装饰者模式动态地将责任附加到对象上。若要扩展此功能，装饰者提供了比继承更有弹性的代替方案。

> 装饰者模式意味着一群装饰者类，这些类用来包装具体的组件。
>
> 装饰者反映出被装饰的组件类型（事实上，他们具有相同的类型，都经过接口或继承实现）。
>
> 装饰者可以在被装饰者的行为前面或后面加上自己的行为，甚至将被装饰者行为整个取代掉，而达到特定的目的。
>
> 可以用无数个装饰者包装一个组件。
>
> 装饰者一般对组件的客户是透明的，除非客户程序依赖与组件的具体类型。
>
> 装饰者会导致设计中出现许多小对象，过度使用，会让程序变的很复杂。

**主要优点：**

- 装饰器是继承的有力补充，比继承灵活，在不改变原有对象的情况下，动态的给一个对象扩展功能，即插即用
- 通过使用不用装饰类及这些装饰类的排列组合，可以实现不同效果
- 装饰器模式完全遵守开闭原则

**主要缺点：**

装饰模式会增加许多子类，过度使用会增加程序得复杂性。

**应用实例：** 

1. 孙悟空有 72 变，当他变成"庙宇"后，他的根本还是一只猴子，但是他又有了庙宇的功能。 

2. 不论一幅画有没有画框都可以挂在墙上，但是通常都是有画框的，并且实际上是画框被挂在墙上。在挂在墙上之前，画可以被蒙上玻璃，装到框子里；这时画、玻璃和画框形成了一个物体。

**使用场景：** 

1. 扩展一个类的功能。 

2. 动态增加功能，动态撤销。

**装饰模式的结构与实现**

通常情况下，扩展一个类的功能会使用继承方式来实现。但继承具有静态特征，耦合度高，并且随着扩展功能的增多，子类会很膨胀。如果使用组合关系来创建一个包装对象（即装饰对象）来包裹真实对象，并在保持真实对象的类结构不变的前提下，为其提供额外的功能，这就是装饰模式的目标。下面来分析其基本结构和实现方法。

1. 抽象构件（Component）角色：定义一个抽象接口以规范准备接收附加责任的对象。
2. 具体构件（ConcreteComponent）角色：实现抽象构件，通过装饰角色为其添加一些职责。
3. 抽象装饰（Decorator）角色：继承抽象构件，并包含具体构件的实例，可以通过其子类扩展具体构件的功能。
4. 具体装饰（ConcreteDecorator）角色：实现抽象装饰的相关方法，并给具体构件对象添加附加的责任。

装饰模式的结构图如下图所示。

![装饰模式的结构图](http://c.biancheng.net/uploads/allimg/181115/3-1Q115142115M2.gif)

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

**分为**：简单工厂模式，工厂模式和抽象工厂模式

**简单工厂模式**：其实不是一种设计模式，反而比较像是一种编程习惯，但仍可以将客户程序从具体实现解耦。

**工厂模式定义**：定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化到子类。

**抽象工厂模式定义**：提供了一个接口，用于创建相关或依赖对象的家族，而不需要明确制定具体类

> 工厂方法用来处理对象的创建，并将这样的行为封装在子类，客户程序中关于超类的代码就和子类对象创建代码解耦类。
>
> 所有工厂模式都是用来封装对象的创建，都通过减少应用程序和具体类之间的依赖促进松耦合。
>
> 工厂模式方法通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的。
>
> 工厂方法使用继承：把对象的创建委托给子类，子类实现工厂方法来创建对象。
>
> 抽象工厂使用对象耦合：对象的创建被实现在工厂接口所暴露出来的方法中。

### 1. 简单工厂模式

**优点：**

1. 工厂类包含必要的逻辑判断，可以决定在什么时候创建哪一个产品的实例。客户端可以免除直接创建产品对象的职责，很方便的创建出相应的产品。工厂和产品的职责区分明确。
2. 客户端无需知道所创建具体产品的类名，只需知道参数即可。
3. 也可以引入配置文件，在不修改客户端代码的情况下更换和添加新的具体产品类。

**缺点：**

1. 简单工厂模式的工厂类单一，负责所有产品的创建，职责过重，一旦异常，整个系统将受影响。且工厂类代码会非常臃肿，违背高聚合原则。
2. 使用简单工厂模式会增加系统中类的个数（引入新的工厂类），增加系统的复杂度和理解难度
3. 系统扩展困难，一旦增加新产品不得不修改工厂逻辑，在产品类型较多时，可能造成逻辑过于复杂
4. 简单工厂模式使用了 static 工厂方法，造成工厂角色无法形成基于继承的等级结构。

**应用场景：**

对于产品种类相对较少的情况，考虑使用简单工厂模式。使用简单工厂模式的客户端只需要传入工厂类的参数，不需要关心如何创建对象的逻辑，可以很方便地创建所需产品。

**模式的结构与实现：**

- 简单工厂（SimpleFactory）：是简单工厂模式的核心，负责实现创建所有实例的内部逻辑。工厂类的创建产品类的方法可以被外界直接调用，创建所需的产品对象。
- 抽象产品（Product）：是简单工厂创建的所有对象的父类，负责描述所有实例共有的公共接口。
- 具体产品（ConcreteProduct）：是简单工厂模式的创建目标。


其结构图如下图所示。

![简单工厂模式的结构图](http://c.biancheng.net/uploads/allimg/200908/5-200ZQ64244445.png)

```java
//抽象产品类（接口或抽象类）
public interface Product {
    void doSomething();
    void doAnything();
}

//具体产品类
public class ConcreteProductA implements Product {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductA doSomething");

    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductA doAnything");
    }
}
public class ConcreteProductB implements Product {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductB doSomething");

    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductB doAnything");
    }
}

//工厂类
public class Creator {

    public static Product createProduct(String type) {
        Product product = null;
        switch (type) {
            case "A":
                product = new ConcreteProductA();
                break;
            case "B":
                product = new ConcreteProductB();
                break;
        }
        return product;
    }
}

//客户端代码
public class Client {

    public static void main(String[] args) {
        Product productA = Creator.createProduct("A");
        productA.doSomething();
        productA.doAnything();
      
        Product productB = Creator.createProduct("B");
        productB.doSomething();
        productB.doAnything();
    }

}
```

### 2. 工厂模式

**优点：**

- 用户只需要知道具体工厂的名称就可得到所要的产品，无须知道产品的具体创建过程。
- 灵活性增强，对于新产品的创建，只需多写一个相应的工厂类。
- 典型的解耦框架。高层模块只需要知道产品的抽象类，无须关心其他实现类，满足迪米特法则、依赖倒置原则和里氏替换原则。

**缺点：**

- 类的个数容易过多，增加复杂度
- 增加了系统的抽象性和理解难度
- 抽象产品只能生产一种产品，此弊端可使用抽象工厂模式解决。

**应用场景：**

- 客户只知道创建产品的工厂名，而不知道具体的产品名。如 TCL 电视工厂、海信电视工厂等。
- 创建对象的任务由多个具体子工厂中的某一个完成，而抽象工厂只提供创建产品的接口。
- 客户不关心创建产品的细节，只关心产品的品牌

**模式的结构与实现：**

工厂方法模式由抽象工厂、具体工厂、抽象产品和具体产品等4个要素构成。

1. 抽象工厂（Abstract Factory）：提供了创建产品的接口，调用者通过它访问具体工厂的工厂方法 newProduct() 来创建产品。
2. 具体工厂（ConcreteFactory）：主要是实现抽象工厂中的抽象方法，完成具体产品的创建。
3. 抽象产品（Product）：定义了产品的规范，描述了产品的主要特性和功能。
4. 具体产品（ConcreteProduct）：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间一一对应。

![工厂方法模式的结构图](http://c.biancheng.net/uploads/allimg/181114/3-1Q114135A2M3.gif)

```java
//抽象产品类
public interface Product {
    void doSomething();
    void doAnything();
}

//具体产品类
public class ConcreteProductA implements Product {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductA doSomething");

    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductA doAnything");
    }
}
public class ConcreteProductB implements Product {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductB doSomething");

    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductB doAnything");
    }
}

//抽象工厂类
public interface Creator {
    Product createProduct();
}

//具体工厂类
public class ConcreteCreatorA implements Creator {
    @Override
    public Product createProduct() {
        return new ConcreteProductA();
    }
}
public class ConcreteCreatorB implements Creator {
    @Override
    public Product createProduct() {
        return new ConcreteProductB();
    }
}

//客户端代码
public static void main(String[] args) {
        Creator creatorA = new ConcreteCreatorA();
        Product productA = creatorA.createProduct();
        productA.doSomething();
        productA.doAnything();

        Creator creatorB = new ConcreteCreatorB();
        Product productB = creatorB.createProduct();
        productB.doSomething();
        productB.doAnything();
    }
```



### 3. 抽象工厂模式

抽象工厂模式是工厂方法模式的升级版本，工厂方法模式只生产一个等级的产品，而抽象工厂模式可生产多个等级的产品。

使用抽象工厂模式一般要满足以下条件。

- 系统中有多个产品族，每个具体工厂创建同一族但属于不同等级结构的产品。
- 系统一次只可能消费其中某一族产品，即同族的产品一起使用。

抽象工厂模式除了具有工厂方法模式的优点外，**其他主要优点:**

- 可以在类的内部对产品族中相关联的多等级产品共同管理，而不必专门引入多个新的类来进行管理。
- 当需要产品族时，抽象工厂可以保证客户端始终只使用同一个产品的产品组。
- 抽象工厂增强了程序的可扩展性，当增加一个新的产品族时，不需要修改原代码，满足开闭原则。

**缺点：**

当产品族中需要增加一个新的产品时，所有的工厂类都需要进行修改。增加了系统的抽象性和理解难度。

**应用场景：**

抽象工厂模式最早的应用是用于创建属于不同操作系统的视窗构件。如Java的 AWT 中的 Button 和 Text 等构件在 Windows 和 UNIX 中的本地实现是不同的。

抽象工厂模式通常适用于以下场景：

1. 当需要创建的对象是一系列相互关联或相互依赖的产品族时，如电器工厂中的电视机、洗衣机、空调等。
2. 系统中有多个产品族，但每次只使用其中的某一族产品。如有人只喜欢穿某一个品牌的衣服和鞋。
3. 系统中提供了产品的类库，且所有产品的接口相同，客户端不依赖产品实例的创建细节和内部结构。

**模式的结构与实现：**

抽象工厂模式同工厂方法模式一样，也是由抽象工厂、具体工厂、抽象产品和具体产品等 4 个要素构成，但抽象工厂中方法个数不同，抽象产品的个数也不同。现在我们来分析其基本结构和实现方法。

1. 抽象工厂（Abstract Factory）：提供了创建产品的接口，它包含多个创建产品的方法 newProduct()，可以创建多个不同等级的产品。
2. 具体工厂（Concrete Factory）：主要是实现抽象工厂中的多个抽象方法，完成具体产品的创建。
3. 抽象产品（Product）：定义了产品的规范，描述了产品的主要特性和功能，抽象工厂模式有多个抽象产品。
4. 具体产品（ConcreteProduct）：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间是多对一的关系。

![抽象工厂模式的结构图](http://c.biancheng.net/uploads/allimg/181114/3-1Q11416002NW.gif)



````java
//抽象产品类
//产品A家族
public interface ProductA {
    void doSomething();
    void doAnything();
}

//产品B家族
public interface ProductB {
    void doSomething();
    void doAnything();
}

//具体产品类
//产品A家族，产品等级1
public class ConcreteProductA1 implements ProductA {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductA1 doSomething");
    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductA1 doAnything");
    }
}

//产品A家族，产品等级2
public class ConcreteProductA2 implements ProductA {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductA2 doSomething");
    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductA2 doAnything");
    }
}

//产品B家族，产品等级2
public class ConcreteProductB1 implements ProductB {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductB1 doSomething");
    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductB1 doAnything");
    }
}

//产品B家族，产品等级2
public class ConcreteProductB2 implements ProductB {
    @Override
    public void doSomething() {
        System.out.println("ConcreteProductB2 doSomething");
    }

    @Override
    public void doAnything() {
        System.out.println("ConcreteProductB2 doAnything");
    }
}

//抽象工厂类
public interface Creator {
    /**
     * 创建A产品家族
     * @return
     */
    ProductA createProductA();

    /**
     * 创建B产品家族
     * @return
     */
    ProductB createProductB();

    // ...
    // 有N个产品族，在抽象工厂类中就应该有N个创建方法
}

//有N个产品族，在抽象工厂类中就应该有N个创建方法
//具体工厂类
public class ConcreteCreator1 implements Creator {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB1();
    }
}
public class ConcreteCreator2 implements Creator {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA2();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB2();
    }
}

//有M个产品等级，就应该有M个具体工厂实现
//客户端代码
public class Client {
    public static void main(String[] args) {
        Creator creator1 = new ConcreteCreator1();

        ProductA productA1 = creator1.createProductA();
        productA1.doSomething();
        productA1.doAnything();

        ProductB productB1 = creator1.createProductB();
        productB1.doSomething();
        productB1.doAnything();

        Creator creator2 = new ConcreteCreator2();

        ProductA productA2 = creator2.createProductA();
        productA2.doSomething();
        productA2.doAnything();

        ProductB productB2 = creator2.createProductB();
        productB2.doSomething();
        productB2.doAnything();
    }
}
````



## 四、单例模式

**定义：**确保一个类只有一个实例，并提供一个全局访问点。

**优点：**

- 单例模式可以保证内存里只有一个实例，减少了内存的开销。
- 可以避免对资源的多重占用。
- 单例模式设置全局访问点，可以优化和共享资源的访问。

**缺点：**

- 单例模式一般没有接口，扩展困难。如果要扩展，则除了修改原来的代码，没有第二种途径，违背开闭原则。
- 在并发测试中，单例模式不利于代码调试。在调试过程中，如果单例中的代码没有执行完，也不能模拟生成一个新的对象。
- 单例模式的功能代码通常写在一个类中，如果功能设计不合理，则很容易违背单一职责原则。

**应用场景**

对于 Java来说，单例模式可以保证在一个 JVM 中只存在单一实例。

- 需要频繁创建的一些类，使用单例可以降低系统的内存压力，减少 GC。
- 某类只要求生成一个对象的时候，如一个班中的班长、每个人的身份证号等。
- 某些类创建实例时占用资源较多，或实例化耗时较长，且经常使用。
- 某类需要频繁实例化，而创建的对象又频繁被销毁的时候，如多线程的线程池、网络连接池等。
- 频繁访问数据库或文件的对象。
- 对于一些控制硬件级别的操作，或者从系统上来讲应当是单一控制逻辑的操作，如果有多个实例，则系统会完全乱套。
- 当对象需要被共享的场合。由于单例模式只允许创建一个对象，共享该对象可以节省内存，并加快对象访问速度。如 Web 中的配置对象、数据库的连接池等。

**使用场景**

一般情况下，建议使用饿汉方式。只有在要明确实现 lazy loading 效果时，才会使用静态内部类。如果涉及到反序列化创建对象时，可以尝试使用枚举方式。如果有其他特殊的需求，可以考虑使用DCL方式。

**结构与实现**

- 单例类：包含一个实例且能自行创建这个实例的类。
- 访问类：使用单例的类。

![单例模式的结构图](http://c.biancheng.net/uploads/allimg/181113/3-1Q1131K441K2.gif)

**多线程下的单例模式：**

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

- 用“双重检查锁DCL”，在`getInstance()`中减少使用同步

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

这种方式采用双锁机制，安全且在多线程情况下能保持高性能。getInstance() 的性能对应用程序很关键。

- 静态内部类

```java
public class Singleton {  
    private static class SingletonHolder {  
         private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    	return SingletonHolder.INSTANCE;  
    }  
}
```

这种方式能达到双检锁方式一样的功效，但实现更简单。对静态域使用延迟初始化，应使用这种方式而不是双检锁方式。

- 枚举

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}
```

这种实现方式没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。

Effective Java 作者 Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象，绝对防止多次实例化。

不过，由于 JDK1.5 之后才加入 enum 特性，用这种方式写不免让人感觉生疏，在实际工作中，也很少用。

不能通过 reflection attack 来调用私有构造方法。

## 五、命令模式

**定义：**将请求封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

>命令模式将发出请求的对象和执行请求的对象解偶，在被解耦的两者之间时通过命令对象进行沟通的，命令对象封装了接收者和一个或一组动作。
>
>调用者通过调用命令对象的execute() 发出请求，这会使接收者的动作被调用.
>
>当命令支持撤销时，该命令必须提供和execute() 方法相反的方法undo() 方法
>
>调用者可以接受命令当做参数，甚至可以在运行时动态进行。
>
>实际操作时，可以直接实现请求，而非将工作直接委托给接收者。

- 空对象：当不想返回一个有意义的对象，可以使用空对象，将处理null的责任转移给空对象。

- 宏命令：在命令中添加命令的集合，是命令的一种简单延伸，允许调用多个命令。支持撤销。

**优点：**

1. 通过引入中间件（抽象接口）降低系统的耦合度。
2. 扩展性良好，增加或删除命令非常方便。采用命令模式增加与删除命令不会影响其他类，且满足“开闭原则”。
3. 可以实现宏命令。命令模式可以与组合模式结合，将多个命令装配成一个组合命令，即宏命令。
4. 方便实现 Undo 和 Redo 操作。命令模式可以与后面介绍的备忘录模式结合，实现命令的撤销与恢复。
5. 可以在现有命令的基础上，增加额外功能。比如日志记录，结合装饰器模式会更加灵活。

**缺点：**

1. 可能产生大量具体的命令类。因为每一个具体操作都需要设计一个具体命令类，这会增加系统的复杂性。
2. 命令模式的结果其实就是接收方的执行结果，但是为了以命令的形式进行架构、解耦请求与实现，引入了额外类型结构（引入了请求方与抽象命令接口），增加了理解上的困难。不过这也是设计模式的通病，抽象必然会额外增加类的数量，代码抽离肯定比代码聚合更加难理解。

**应用实例**

用命令模式实现客户去餐馆吃早餐的实例。分析：客户去餐馆可选择的早餐有肠粉、河粉和馄饨等，客户可向服务员选择以上早餐中的若干种，服务员将客户的请求交给相关的厨师去做。这里的点早餐相当于“命令”，服务员相当于“调用者”，厨师相当于“接收者”，所以用命令模式实现比较合适。

**使用场景**

认为是命令的地方都可以使用命令模式，比如： 1、GUI 中每一个按钮都是一条命令。 2、模拟 CMD。

**命令模式的结构与实现**

可以将系统中的相关操作抽象成命令，使调用者与实现者相关分离，其结构如下。

命令模式包含以下主要角色。

1. 抽象命令类（Command）角色：声明执行命令的接口，拥有执行命令的抽象方法 execute()。
2. 具体命令类（Concrete Command）角色：是抽象命令类的具体实现类，它拥有接收者对象，并通过调用接收者的功能来完成命令要执行的操作。
3. 实现者/接收者（Receiver）角色：执行命令功能的相关操作，是具体命令对象业务的真正实现者。
4. 调用者/请求者（Invoker）角色：是请求的发送者，它通常拥有很多的命令对象，并通过访问命令对象来执行相关请求，它不直接访问接收者。





![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTYvMy0xUTExNjExMzM1RTQ0LmdpZg)

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
    public ConcreteCommand()
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

## 六、适配器模式和外观模式

**定义：**

适配器模式将一个类的接口，转化成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。

外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。

> 适配器模式有两种形态：对象适配器和类适配器。
>
> 类适配器用到多重继承（不适用于Java，但可以定义一个适配器类来实现当前系统的业务接口，同时又继承现有组件库中已经存在的组件）。

**意图区别：**

装饰者模式：不改变接口，但加入责任。

适配器模式：将一个接口转换成为另一个接口。

外观模式：让接口更接单。

适配器模式和外观模式的主要差异在意图上。而不在包装类的数量上，适配器是转化接口，外观模式是提供子系统的更简单的接口，从复杂的子系统中解耦。

**优点：**

适配器模式

- 客户端通过适配器可以透明地调用目标接口。
- 复用了现存的类，程序员不需要修改原有代码而重用现有的适配者类。
- 将目标类和适配者类解耦，解决了目标类和适配者类接口不一致的问题。
- 在很多业务场景中符合开闭原则。

外观模式

- 降低了子系统与客户端之间的耦合度，使得子系统的变化不会影响调用它的客户类。

- 对客户屏蔽了子系统组件，减少了客户处理的对象数目，并使得子系统使用起来更加容易。

- 降低了大型软件系统中的编译依赖性，简化了系统在不同平台之间的移植过程，因为编译一个子系统不会影响其他的子系统，也不会影响外观对象。

**缺点：**

适配器模式

- 适配器编写过程需要结合业务场景全面考虑，可能会增加系统的复杂性。
- 增加代码阅读难度，降低代码可读性，过多使用适配器会使系统代码变得凌乱。

外观模式

- 不能很好地限制客户使用子系统类，很容易带来未知风险。

- 增加新的子系统可能需要修改外观类或客户端的源代码，违背了“开闭原则”。

**应用实例：**

适配器模式 

1、美国电器 110V，中国 220V，就要有一个适配器将 110V 转化为 220V。 

2、JAVA JDK 1.1 提供了 Enumeration 接口，而在 1.2 中提供了 Iterator 接口，想要使用 1.2 的 JDK，则要将以前系统的 Enumeration 接口转化为 Iterator 接口，这时就需要适配器模式。

外观模式

1、去医院看病，可能要去挂号、门诊、划价、取药，让患者或患者家属觉得很复杂，如果有提供接待人员，只让接待人员来处理，就很方便。

**使用场景：**

适配器模式

需要开发的具有某种业务功能的组件在现有的组件库中已经存在，但它们与当前系统的接口规范不兼容，如果重新开发这些组件成本又很高，这时用适配器模式能很好地解决这些问题。

外观模式

1、为复杂的模块或子系统提供外界访问的模块。 2、子系统相对独立。 3、预防低水平人员带来的风险

**模式的结构与实现**

对象适配器模式可釆用将现有组件库中已经实现的组件引入适配器类中，该类同时实现当前系统的业务接口。

1. 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口。
2. 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。
3. 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

类适配器模式

![类适配器模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTEwNDUzNTFjLmdpZg)

对象适配器模式

![对象适配器模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTEwNDYxMDVBLmdpZg)

类适配器模式的代码如下:

```java
//目标接口
interface Target
{
    public void request();
}
//适配者接口
class Adaptee
{
    public void specificRequest()
    {       
        System.out.println("适配者中的业务代码被调用！");
    }
}
//类适配器类
class ClassAdapter extends Adaptee implements Target
{
    public void request()
    {
        specificRequest();
    }
}
//客户端代码
public class ClassAdapterTest
{
    public static void main(String[] args)
    {
        System.out.println("类适配器模式测试：");
        Target target = new ClassAdapter();
        target.request();
    }
}
```

对象适配器模式的代码:

```java
//对象适配器类
class ObjectAdapter implements Target
{
    private Adaptee adaptee;
    public ObjectAdapter(Adaptee adaptee)
    {
        this.adaptee=adaptee;
    }
    public void request()
    {
        adaptee.specificRequest();
    }
}
//客户端代码
public class ObjectAdapterTest
{
    public static void main(String[] args)
    {
        System.out.println("对象适配器模式测试：");
        Adaptee adaptee = new Adaptee();
        Target target = new ObjectAdapter(adaptee);
        target.request();
    }
}
```

外观（Facade）模式包含以下主要角色：

1. 外观（Facade）角色：为多个子系统对外提供一个共同的接口。
2. 子系统（Sub System）角色：实现系统的部分功能，客户可以通过外观角色访问它。
3. 客户（Client）角色：通过一个外观角色访问各个子系统的功能。

外观模式的结构图:

![外观模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTE1MjE0MzUwOS5naWY)

```java
public class FacadePattern
{
    public static void main(String[] args)
    {
        Facade f=new Facade();
        f.method();
    }
}
//外观角色
class Facade
{
    private SubSystem01 obj1=new SubSystem01();
    private SubSystem02 obj2=new SubSystem02();
    private SubSystem03 obj3=new SubSystem03();
    public void method()
    {
        obj1.method1();
        obj2.method2();
        obj3.method3();
    }
}
//子系统角色
class SubSystem01
{
    public  void method1()
    {
        System.out.println("子系统01的method1()被调用！");
    }   
}
//子系统角色
class SubSystem02
{
    public  void method2()
    {
        System.out.println("子系统02的method2()被调用！");
    }   
}
//子系统角色
class SubSystem03
{
    public  void method3()
    {
        System.out.println("子系统03的method3()被调用！");
    }   
}
```



## 七、模版方法模式

**定义：**模版方法定义了一个算法的步骤，并允许子类为一个或多个步骤提供实现。

> 模版方法模式在一个方法中定义了一个算法的骨架，而将一些步骤延迟到子类中。模版方法使得字类可以在不改变算法结构的情况下，重信定义算法中的某些步骤。
>
> 模版方法为我们提供一种代码复用的重要技巧。
>
> 模版方法的抽象类可以定义具体方法，抽象方法和钩子。
>
> 钩子（hook）是一种被声明在抽象类中的方法，但只有空的或者默认的实现。可以让子类有能力对算法的不同点进行挂钩。
>
> 为了防止子类改变模版方法中的算法，可以将模版声明final。
>

策略模式和模版方法模式都封装算法，策略模式用组合，模版方法模式用继承。

工厂方法是版模版方法的一种特殊版本。

**优点：**

1. 它封装了不变部分，扩展可变部分。它把认为是不变部分的算法封装到父类中实现，而把可变部分算法由子类继承实现，便于子类继续扩展。
2. 它在父类中提取了公共的部分代码，便于代码复用。
3. 部分方法是由子类实现的，因此子类可以通过扩展方式增加相应的功能，符合开闭原则。

**缺点：**

1. 对每个不同的实现都需要定义一个子类，这会导致类的个数增加，系统更加庞大，设计也更加抽象，间接地增加了系统实现的复杂度。
2. 父类中的抽象方法由子类实现，子类执行的结果会影响父类的结果，这导致一种反向的控制结构，它提高了代码阅读的难度。
3. 由于继承关系自身的缺点，如果父类添加新的抽象方法，则所有子类都要改一遍。

**应用实例：** 

1、在造房子的时候，地基、走线、水管都一样，只有在建筑的后期才有加壁橱加栅栏等差异。 

2、西游记里面菩萨定好的 81 难，这就是一个顶层的逻辑骨架。 

3、spring 中对 Hibernate 的支持，将一些已经定好的方法封装起来，比如开启事务、获取 Session、关闭 Session 等，程序员不重复写那些已经规范好的代码，直接丢一个实体就可以保存。

**使用场景：** 1、有多个子类共有的方法，且逻辑相同。 2、重要的、复杂的方法，可以考虑作为模板方法。

**模式的结构与实现**

模板方法模式需要注意抽象类与具体子类之间的协作。

它用到了虚函数的多态性技术以及“不用调用我，让我来调用你”的反向控制技术。

模板方法模式包含以下主要角色：

1）抽象类/抽象模板（Abstract Class）

抽象模板类，负责给出一个算法的轮廓和骨架。它由一个模板方法和若干个基本方法构成。这些方法的定义如下。

① 模板方法：定义了算法的骨架，按某种顺序调用其包含的基本方法。

② 基本方法：是整个算法中的一个步骤，包含以下几种类型。

- 抽象方法：在抽象类中声明，由具体子类实现。
- 具体方法：在抽象类中已经实现，在具体子类中可以继承或重写它。
- 钩子方法：在抽象类中已经实现，包括用于判断的逻辑方法和需要子类重写的空方法两种。

2）具体子类/具体实现（Concrete Class）

具体实现类，实现抽象类中所定义的抽象方法和钩子方法，它们是一个顶级逻辑的一个组成步骤。



![模板方法模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTYvMy0xUTExNjA5NTQwNTMwOC5naWY)

```java
public class TemplateMethodPattern
{
    public static void main(String[] args)
    {
        AbstractClass tm=new ConcreteClass();
        tm.TemplateMethod();
    }
}
//抽象类
abstract class AbstractClass
{
    public void TemplateMethod() //模板方法
    {
        SpecificMethod();
        abstractMethod1();          
        abstractMethod2();
    }  
    public void SpecificMethod() //具体方法
    {
        System.out.println("抽象类中的具体方法被调用...");
    }   
    public abstract void abstractMethod1(); //抽象方法1
    public abstract void abstractMethod2(); //抽象方法2
}
//具体子类
class ConcreteClass extends AbstractClass
{
    public void abstractMethod1()
    {
        System.out.println("抽象方法1的实现被调用...");
    }   
    public void abstractMethod2()
    {
        System.out.println("抽象方法2的实现被调用...");
    }
}
```



## 八、迭代器

**定义：**迭代器模式提供类一种方法访问一个聚合对象中的各个元素，而又不暴露其内部的表示。

> 迭代器将遍历聚合的工作封装到一个对象中。当使用迭代器的时候，我们依赖聚合提供遍历。

**优点：**

1. 访问一个聚合对象的内容而无须暴露它的内部表示。
2. 遍历任务交由迭代器完成，这简化了聚合类。
3. 它支持以不同方式遍历一个聚合，甚至可以自定义迭代器的子类以支持新的遍历。
4. 增加新的聚合类和迭代器类都很方便，无须修改原有代码。
5. 封装性良好，为遍历不同的聚合结构提供一个统一的接口。

**缺点：**增加了类的个数，这在一定程度上增加了系统的复杂性。

**应用实例：**JAVA 中的 iterator。

**使用场景：** 

1. 访问一个聚合对象的内容而无须暴露它的内部表示。 

2. 需要为聚合对象提供多种遍历方式。 

3. 为遍历不同的聚合结构提供一个统一的接口。

**模式的结构与实现**

1. 抽象聚合（Aggregate）角色：定义存储、添加、删除聚合对象以及创建迭代器对象的接口。
2. 具体聚合（ConcreteAggregate）角色：实现抽象聚合类，返回一个具体迭代器的实例。
3. 抽象迭代器（Iterator）角色：定义访问和遍历聚合元素的接口，通常包含 hasNext()、first()、next() 等方法。
4. 具体迭代器（Concretelterator）角色：实现抽象迭代器接口中所定义的方法，完成对聚合对象的遍历，记录遍历的当前位置。

![迭代器模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTYvMy0xUTExNjFQVTk1MjguZ2lm)



```c
public class IteratorPattern
{
    public static void main(String[] args)
    {
        Aggregate ag=new ConcreteAggregate(); 
        ag.add("中山大学"); 
        ag.add("华南理工"); 
        ag.add("韶关学院");
        System.out.print("聚合的内容有：");
        Iterator it=ag.getIterator(); 
        while(it.hasNext())
        { 
            Object ob=it.next(); 
            System.out.print(ob.toString()+"\t"); 
        }
        Object ob=it.first();
        System.out.println("\nFirst："+ob.toString());
    }
}
//抽象聚合
interface Aggregate
{ 
    public void add(Object obj); 
    public void remove(Object obj); 
    public Iterator getIterator(); 
}
//具体聚合
class ConcreteAggregate implements Aggregate
{ 
    private List<Object> list=new ArrayList<Object>(); 
    public void add(Object obj)
    { 
        list.add(obj); 
    }
    public void remove(Object obj)
    { 
        list.remove(obj); 
    }
    public Iterator getIterator()
    { 
        return(new ConcreteIterator(list)); 
    }     
}
//抽象迭代器
interface Iterator
{
    Object first();
    Object next();
    boolean hasNext();
}
//具体迭代器
class ConcreteIterator implements Iterator
{ 
    private List<Object> list=null; 
    private int index=-1; 
    public ConcreteIterator(List<Object> list)
    { 
        this.list=list; 
    } 
    public boolean hasNext()
    { 
        if(index<list.size()-1)
        { 
            return true;
        }
        else
        {
            return false;
        }
    }
    public Object first()
    {
        index=0;
        Object obj=list.get(index);;
        return obj;
    }
    public Object next()
    { 
        Object obj=null; 
        if(this.hasNext())
        { 
            obj=list.get(++index); 
        } 
        return obj; 
    }   
}
```



## 九、组合模式

**定义：**组合模式允你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。

> 组合模式以单一设计原则获取透明性。
>
> 组合结构内的任意对象成为组件，组件可以是组合也可以是叶节点。

![组合模式树形结构图](http://c.biancheng.net/uploads/allimg/201019/5-201019124253553.png)

**优点：**

1. 组合模式使得客户端代码可以一致地处理单个对象和组合对象，无须关心自己处理的是单个对象，还是组合对象，这简化了客户端代码；
2. 更容易在组合体内加入新的对象，客户端不会因为加入了新的对象而更改源代码，满足“开闭原则”；

**缺点：**

1. 设计较复杂，客户端需要花更多时间理清类之间的层次关系；
2. 不容易限制容器中的构件；
3. 不容易用继承的方法来增加构件的新功能；

**应用实例：** 

1. 算术表达式包括操作数、操作符和另一个操作数，其中，另一个操作数也可以是操作数、操作符和另一个操作数。
2. 在 JAVA AWT 和 SWING 中，对于 Button 和 Checkbox 是树叶，Container 是树枝。

**使用场景：**部分、整体场景，如树形菜单，文件、文件夹的管理。

**组合模式的结构与实现**

1. 抽象构件（Component）角色：它的主要作用是为树叶构件和树枝构件声明公共接口，并实现它们的默认行为。在透明式的组合模式中抽象构件还声明访问和管理子类的接口；在安全式的组合模式中不声明访问和管理子类的接口，管理工作由树枝构件完成。（总的抽象类或接口，定义一些通用的方法，比如新增、删除）
2. 树叶构件（Leaf）角色：是组合中的叶节点对象，它没有子节点，用于继承或实现抽象构件。
3. 树枝构件（Composite）角色 / 中间构件：是组合中的分支节点对象，它有子节点，用于继承和实现抽象构件。它的主要作用是存储和管理子部件，通常包含 Add()、Remove()、GetChild() 等方法。

组合模式分为透明式的组合模式和安全式的组合模式。

**透明式的组合模式**

在该方式中，由于抽象构件声明了所有子类中的全部方法，所以客户端无须区别树叶对象和树枝对象，对客户端来说是透明的。但其缺点是：树叶构件本来没有 Add()、Remove() 及 GetChild() 方法，却要实现它们（空实现或抛异常），这样会带来一些安全性问题。



![透明式的组合模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTFHNjJMMTcuZ2lm)

**安全式的组合模式**

在该方式中，将管理子构件的方法移到树枝构件中，抽象构件和树叶构件没有对子对象的管理方法，这样就避免了上一种方式的安全性问题，但由于叶子和分支有不同的接口，客户端在调用时要知道树叶对象和树枝对象的存在，所以失去了透明性。

![安全式的组合模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTFHRjUyMjEuZ2lm)

透明组合模式

```java
public class CompositePattern
{
    public static void main(String[] args)
    {
        Component c0 = new Composite(); 
        Component c1 = new Composite(); 
        Component leaf1 = new Leaf("1"); 
        Component leaf2 = new Leaf("2"); 
        Component leaf3 = new Leaf("3");          
        c0.add(leaf1); 
        c0.add(c1);
        c1.add(leaf2); 
        c1.add(leaf3);          
        c0.operation(); 
    }
}
//抽象构件
interface Component
{
    public void add(Component c);
    public void remove(Component c);
    public Component getChild(int i);
    public void operation();
}
//树叶构件
class Leaf implements Component
{
    private String name;
    public Leaf(String name)
    {
        this.name=name;
    }
    public void add(Component c){ }           
    public void remove(Component c){ }   
    public Component getChild(int i)
    {
        return null;
    }   
    public void operation()
    {
        System.out.println("树叶"+name+"：被访问！"); 
    }
}
//树枝构件
class Composite implements Component
{
    private ArrayList<Component> children=new ArrayList<Component>();   
    public void add(Component c)
    {
        children.add(c);
    }   
    public void remove(Component c)
    {
        children.remove(c);
    }   
    public Component getChild(int i)
    {
        return children.get(i);
    }   
    public void operation()
    {
        for(Object obj:children)
        {
            ((Component)obj).operation();
        }
    }    
}
```

安全组合模式

安全式的组合模式与透明式组合模式的实现代码类似，只要对其做简单修改就可以了。

首先修改 Component 代码，只保留层次的公共行为。

```java
interface Component {
    public void operation();
}
```

然后修改客户端代码，将树枝构件类型更改为 Composite 类型，以便获取管理子类操作的方法。

```java
public class CompositePattern {
    public static void main(String[] args) {
        Composite c0 = new Composite();
        Composite c1 = new Composite();
        Component leaf1 = new Leaf("1");
        Component leaf2 = new Leaf("2");
        Component leaf3 = new Leaf("3");
        c0.add(leaf1);
        c0.add(c1);
        c1.add(leaf2);
        c1.add(leaf3);
        c0.operation();
    }
}
```



## 十、状态模式

**定义：**状态模式允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。

> 和程序状态机PSM不同，状态模式用类代表状态。
>
> context会将行为委托给当前状态对象。
>
> 通过将每个状态分装进一个类，我们把以后需要做的任务改变局部化了。
>
> 状态转换可以由State类或Context类控制。
>
> 状态类可以被多Context实例共享。把每个状态都指定到静态的实例变量中。

状态模式和策略模式有相同的类图，但是它们的意图不同。

- 策略模式通常会用行为或算法来配置context类。 

- 状态模式允许Context随状态的改变而改变行为。

**优点：**

1. 结构清晰，状态模式将与特定状态相关的行为局部化到一个状态中，并且将不同状态的行为分割开来，满足“单一职责原则”。
2. 将状态转换显示化，减少对象间的相互依赖。将不同的状态引入独立的对象中会使得状态转换变得更加明确，且减少对象间的相互依赖。
3. 状态类职责明确，有利于程序的扩展。通过定义新的子类很容易地增加新的状态和转换。

**缺点：**

1. 状态模式的使用必然会增加系统的类与对象的个数。
2. 状态模式的结构与实现都较为复杂，如果使用不当会导致程序结构和代码的混乱。
3. 状态模式对开闭原则的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源码，否则无法切换到新增状态，而且修改某个状态类的行为也需要修改对应类的源码。

**应用实例：** 

1. 打篮球的时候运动员可以有正常状态、不正常状态和超常状态。

2. 曾侯乙编钟中，'钟是抽象接口','钟A'等是具体状态，'曾侯乙编钟'是具体环境（Context）

**使用场景：** 

1. 行为随状态改变而改变的场景。 

2. 条件、分支语句的代替者。

**状态模式的结构与实现**

1. 环境类（Context）角色：也称为上下文，它定义了客户端需要的接口，内部维护一个当前状态，并负责具体状态的切换。
2. 抽象状态（State）角色：定义一个接口，用以封装环境对象中的特定状态所对应的行为，可以有一个或多个行为。
3. 具体状态（Concrete State）角色：实现抽象状态所对应的行为，并且在需要的情况下进行状态切换。

![状态模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTYvMy0xUTExNjE1NDEyVTU1LmdpZg)

```java
public class StatePatternClient
{
    public static void main(String[] args)
    {       
        Context context=new Context();    //创建环境       
        context.Handle();    //处理请求
        context.Handle();
        context.Handle();
        context.Handle();
    }
}
//环境类
class Context
{
    private State state;
    //定义环境类的初始状态
    public Context()
    {
        this.state=new ConcreteStateA();
    }
    //设置新状态
    public void setState(State state)
    {
        this.state=state;
    }
    //读取状态
    public State getState()
    {
        return(state);
    }
    //对请求做处理
    public void Handle()
    {
        state.Handle(this);
    }
}
//抽象状态类
abstract class State
{
    public abstract void Handle(Context context);
}
//具体状态A类
class ConcreteStateA extends State
{
    public void Handle(Context context)
    {
        System.out.println("当前状态是 A.");
        context.setState(new ConcreteStateB());
    }
}
//具体状态B类
class ConcreteStateB extends State
{
    public void Handle(Context context)
    {
        System.out.println("当前状态是 B.");
        context.setState(new ConcreteStateA());
    }
}
```



## 十一、代理模式

**定义：**代理模式为另一个对象提供一个替身或占位符以控制对这个对象的访问。

> 使用代理模式创建代表对象，让代表对象控制某对象的访问，对代理的对象可以时远程的对象、创建开销大的对象或事需要安全控制的对象。

和装饰器模式的区别：代理在结构上类似装饰者，但目的不同。装饰者为对象加上行为，而代理则是控制访问。

和适配器模式的区别：适配器模式主要改变所考虑对象的接口，而代理模式不能改变所代理类的接口。

**分类：**

远程代理：控制访问远程对象。

虚拟代理：控制访问创建开销大的资源。

保护代理：基于权限控制对资源的访问。

动态代理：Java在java.lang.reflect中有自己的代理支持，利用这个包可以在运行时动态创建一个代理类，实现一个或者多个接口，并将方法的调用转发到指定的类。

防火墙代理：控制网络资源的访问，保护主题免于侵害。

智能代理：当主题被引用时，进行额外的动作，例如计算一个对象引用的次数。

缓存代理：为开销大的运算结果提供暂时存储，允许多个客户共享结果，以减少计算或网络延迟。

同步代理：在多线程的情况下为主题提供安全的访问。

复杂隐藏代理：用来隐藏一个类的复杂集合的复杂度，并进行访问控制。有时也称外观代理。

写入时复杂代理：用来控制对象的复制，方法是延迟对象的复制，知道客户真的需要为止。这是虚拟代理的变体。

**优点：**

- 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；
- 代理对象可以扩展目标对象的功能；
- 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了程序的可扩展性

**缺点：**

- 代理模式会造成系统设计中类的数量增加
- 在客户端和目标对象之间增加一个代理对象，会造成请求处理速度变慢；
- 增加了系统的复杂度；

**应用实例：**

1. Windows 里面的快捷方式

2. 买火车票不一定在火车站买，也可以去代售点。

**使用场景：**

按职责来划分，通常有以下使用场景： 1、远程代理。 2、虚拟代理。 3、Copy-on-Write 代理。 4、保护（Protect or Access）代理。 5、Cache代理。 6、防火墙（Firewall）代理。 7、同步化（Synchronization）代理。 8、智能引用（Smart Reference）代理。

**模式的结构**

1. 抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法。
2. 真实主题（Real Subject）类：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。
3. 代理（Proxy）类：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。



![代理模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTA5MzAxMTUyMy5naWY)

```java
public class ProxyTest
{
    public static void main(String[] args)
    {
        Proxy proxy=new Proxy();
        proxy.Request();
    }
}
//抽象主题
interface Subject
{
    void Request();
}
//真实主题
class RealSubject implements Subject
{
    public void Request()
    {
        System.out.println("访问真实主题方法...");
    }
}
//代理
class Proxy implements Subject
{
    private RealSubject realSubject;
    public void Request()
    {
        if (realSubject==null)
        {
            realSubject=new RealSubject();
        }
        preRequest();
        realSubject.Request();
        postRequest();
    }
    public void preRequest()
    {
        System.out.println("访问真实主题之前的预处理。");
    }
    public void postRequest()
    {
        System.out.println("访问真实主题之后的后续处理。");
    }
}
```



## 十二、其他模式

### 1. 桥接模式

桥接模式(Bridge Pattern)：将抽象部分与它的实现部分分离，使它们都可以独立地变化。

桥接模式，不只改变实现，还能改变抽象

优点：将实现解耦；抽象和实现可以独立扩展；对于“具体的抽象类”所做的改变，不会影响到客户

用途：适合使用载需要跨越多个平台的图形和窗口系统上；当需要用于不同的方式改变接口和实现时

缺点：增加了复杂度

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTQ3NTU3MS8yMDE5MDEvMTQ3NTU3MS0yMDE5MDExMjE4MDUyNjExMy0xMjA0NjI2NDI1LnBuZw?x-oss-process=image/format,png)



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

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTQ3NTU3MS8yMDE5MDEvMTQ3NTU3MS0yMDE5MDExMjE4MDcxMjIwOC01MDU3ODY4MTkucG5n?x-oss-process=image/format,png)



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

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80ODA3NjU0LWY5Y2UwZmQ3YjE5YmU1MjkucG5n?x-oss-process=image/format,png)

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



![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cucnVub29iLmNvbS93cC1jb250ZW50L3VwbG9hZHMvMjAxNC8wOC9jaGFpbl9wYXR0ZXJuX3VtbF9kaWFncmFtLmpwZw?x-oss-process=image/format,png)

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

![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTUvMy0xUTExNTE2MTM0MjI0Mi5naWY)

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

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cucnVub29iLmNvbS93cC1jb250ZW50L3VwbG9hZHMvMjAxNC8wOC9mbHl3ZWlnaHRfcGF0dGVybl91bWxfZGlhZ3JhbS0xLmpwZw?x-oss-process=image/format,png)

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

![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTkvMy0xUTExOTE1MDYyNjQyMi5naWY)

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

![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTkvMy0xUTExOTE1MFE2NDAxLmdpZg)

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

![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTYvMy0xUTExNjFJNTMyVjAuZ2lm)

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

![](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTkvMy0xUTExOTEzMDQxMzkyNy5naWY)



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

当创建给定类的实例过程很昂贵或很复杂时，就使用原型模式

优点：向客户隐藏制造新实例的复杂性；提供让客户能够产生未知类型对象的选项；在某些环境下，复杂对象比创建新对象更有效

用途：在一个复杂的类层次中，当系统必须从其中的许多类型创建新对象时，可以考虑原型。

缺点：对象的复制有时有点复杂

![原型模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTQvMy0xUTExNDEwMUZhMjIuZ2lm)

原型模式的克隆分为浅克隆和深克隆，Java 中的 Object 类提供了浅克隆的 clone() 方法，具体原型类只要实现 Cloneable 接口就可实现对象的浅克隆，这里的 Cloneable 接口就是抽象原型类。

```java
//具体原型类
class Realizetype implements Cloneable
{
    Realizetype()
    {
        System.out.println("具体原型创建成功！");
    }
    public Object clone() throws CloneNotSupportedException
    {
        System.out.println("具体原型复制成功！");
        return (Realizetype)super.clone();
    }
}
//原型模式的测试类
public class PrototypeTest
{
    public static void main(String[] args)throws CloneNotSupportedException
    {
        Realizetype obj1=new Realizetype();
        Realizetype obj2=(Realizetype)obj1.clone();
        System.out.println("obj1==obj2?"+(obj1==obj2));//false
    }
}
```



### 9. 访问者模式

当你想要为一个对象的组合增加新的能力，且封装并不重要时，就使用访问者模式

优点：允许你对组合结构加入新的操作，而无需改变结构本身；想要加入新的操作，相对容易；访问者所进行的操作，其代码是集中在一起的

缺点：当采用访问者模式的时候，就会打破组合类的封装；对组合结构的改变更加困难

![访问者（Visitor）模式的结构图](https://imgconvert.csdnimg.cn/aHR0cDovL2MuYmlhbmNoZW5nLm5ldC91cGxvYWRzL2FsbGltZy8xODExMTkvMy0xUTExOTEwMTM1WTI1LmdpZg)

```java
public class VisitorPattern
{
    public static void main(String[] args)
    {
        ObjectStructure os=new ObjectStructure();
        os.add(new ConcreteElementA());
        os.add(new ConcreteElementB());
        Visitor visitor=new ConcreteVisitorA();
        os.accept(visitor);
        System.out.println("------------------------");
        visitor=new ConcreteVisitorB();
        os.accept(visitor);
    }
}
//抽象访问者
interface Visitor
{
    void visit(ConcreteElementA element);
    void visit(ConcreteElementB element);
}
//具体访问者A类
class ConcreteVisitorA implements Visitor
{
    public void visit(ConcreteElementA element)
    {
        System.out.println("具体访问者A访问-->"+element.operationA());
    }
    public void visit(ConcreteElementB element)
    {
        System.out.println("具体访问者A访问-->"+element.operationB());
    }
}
//具体访问者B类
class ConcreteVisitorB implements Visitor
{
    public void visit(ConcreteElementA element)
    {
        System.out.println("具体访问者B访问-->"+element.operationA());
    }
    public void visit(ConcreteElementB element)
    {
        System.out.println("具体访问者B访问-->"+element.operationB());
    }
}
//抽象元素类
interface Element
{
    void accept(Visitor visitor);
}
//具体元素A类
class ConcreteElementA implements Element
{
    public void accept(Visitor visitor)
    {
        visitor.visit(this);
    }
    public String operationA()
    {
        return "具体元素A的操作。";
    }
}
//具体元素B类
class ConcreteElementB implements Element
{
    public void accept(Visitor visitor)
    {
        visitor.visit(this);
    }
    public String operationB()
    {
        return "具体元素B的操作。";
    }
}
//对象结构角色
class ObjectStructure
{   
    private List<Element> list=new ArrayList<Element>();   
    public void accept(Visitor visitor)
    {
        Iterator<Element> i=list.iterator();
        while(i.hasNext())
        {
            ((Element) i.next()).accept(visitor);
        }      
    }
    public void add(Element element)
    {
        list.add(element);
    }
    public void remove(Element element)
    {
        list.remove(element);
    }
}
```

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)