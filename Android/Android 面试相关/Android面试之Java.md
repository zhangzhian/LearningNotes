# Java

| 时间       | 版本  | 说明           |
| ---------- | ----- | -------------- |
| 2020.10.15 | 0.0.1 | 初创，结构输入 |
| 2021.01.06 | 0.0.2 | 完成一个初版   |

## 基础

### 一、Object

#### 1. 两个值相等的 Integer 对象，== 比较，判断是否相等

Integer对象值的范围为 `[-128 - 127]` 时，不会创建新的引用，指向同一个地址，地址值相等。

但是当超过这个范围后，就会 new 一个对象，引用指向的地址是不同的，地址值不相等。

#### 2. 下面的代码，再次使用对象 student 是否需要判空？

> Java 中方法参数的使用情况总结：
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

#### 4. Java 的基本数据类型都有哪些各占几个字节?

byte 1 char 2 short 2 int 4 float 4 double 8 long 8 boolean 1（boolean 类型比较特别可能只占一个 bit，多个 boolean 可能共同占用一个字节）



### 二、String

#### 1. 下面的代码， str 值最终为多少？换成 Integer 值又为多少，是否会被改变？

> - **考点**：Java 值传递 (第一节第 2 题相同)。编写代码测试，在 changeValue() 方法中修改入参，并**不会改变**之前的值；
> - **原理** ：Java 程序设计语言总是采用按值调用，方法得到的是所有参数值的一个拷贝，即方法**不能修改**传递给它的任何参数变量的内容。基本类型参数传递的是参数副本，对象类型参数传递的是**对象地址的副本；**
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

大部分情况下，效率是：`StringBuilder`>`StringBuffer`>`String`。

#### 3. Java中中文字符和英文字符的大小分别多少？在网络上传输大小又分别是多少？

Java采用unicode编码，2个字节来表示一个字符，这点与C语言中不同，C语言中采用ASCII，在大多数系统中，一个字符通常占1个字节，但是在0~127整数之间的字符映射，unicode向下兼容ASCII。而Java采用unicode来表示字符，一个中文或英文字符的unicode编码都占2个字节。但如果采用其他编码方式，一个字符占用的字节数则各不相同。

在 **GB 2312 编码或 GBK 编码**中，一个英文字母字符存储需要**1个字节**，一个汉字字符存储需要**2个字节**。 在**UTF-8编码**中，一个英文字母字符存储需要**1个字节**，一个汉字字符储存需要**3到4个字节**。在UTF-16编码中，一个英文字母字符存储需要2个字节，一个汉字字符储存需要3到4个字节（Unicode扩展区的一些汉字存储需要4个字节）。在UTF-32编码中，世界上任何字符的存储都需要4个字节。

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
String是不可变类，每当我们对String进行操作的时候，总是会创建新的字符串。操作String很耗资源,所以Java提供了两个工具类来操作String：StringBuffer和StringBuilder。

StringBuffer和StringBuilder是可变类，StringBuffer是线程安全的，StringBuilder则不是线程安全的。所以在多线程对同一个字符串操作的时候，我们应该选择用StringBuffer。由于不需要处理多线程的情况，StringBuilder的效率比StringBuffer高。

1） String常见的创建方式有两种

- String s1 = “Java”
- String s2 = new String("Java")

2）存储方式不同

- 第一种，s1会先去字符串常量池中找字符串"Java”，如果有相同的字符则直接返回常量句柄，如果没有此字符串则会先在常量池中创建此字符串，然后再返回常量句柄，或者说字符串引用。

- 第二种，s2是直接在堆上创建一个变量对象，但不存储到字符串池 ，调用`intern`方法才会把此字符串保存到常量池中



### 三、封装、继承、多态

#### 1. Java中抽象类和接口的特点？

共同点：

- 抽象类和接口都不能生成具体的实例。
- 都是作为上层使用。

不同点：

- 抽象类可以有属性和成员方法，接口不可以。
- 一个类只能继承一个类，但是可以实现多个接口。
- **抽象类中的变量是普通变量，接口中的变量是静态变量。**
- 抽象类表达的是`is-a`的关系，接口表达的是`like-a`的关系。

#### 2. 关于多态的理解？实现机制？

多态是面向对象的三大特性：继承、封装和多态之一。

多态的定义：允许不同类对同一消息做出响应。

多态存在的条件：

1. 要有继承。
2. 要有复写。
3. 父类引用指向子类对象。

Java中多态的实现方式：接口实现，继承父类进行方法重写，同一个类中的方法重载。

实现机制：靠的是父类或接口定义的引用变量可以指向子类或具体实现类的实例对象，而程序调用的方法在运行期才动态绑定，就是引用变量所指向的具体实例对象的方法，也就是内存里正在运行的那个对象的方法，而不是引用变量的类型中定义的方法。

#### 3. Java面向对象都有哪些特性以及你对这些特性的理解

**继承：**继承是从已有类得到继承信息创建新类的过程。提供继承信息的类被称为父类（超类、基类）；得到
继承信息的类被称为子类（派生类）。

继承让变化中的软件系统有了一定的延续性，同时继承也是封装程序中可变因素的重要手段。

**封装：**通常认为封装是把数据和操作数据的方法绑定起来，对数据的访问只能通过已定义的接口。面向对象的本质就是将现实世界描绘成一系列完全自治、封闭的对象。我们在类中编写的方法就是对实现细节的一种封装；我们编写一个类就是对数据和数据操作的封装。可以说，封装就是隐藏一切可隐藏的东西，只向外界提供最简单的编程接口。

**多态性：**多态性是指允许不同子类型的对象对同一消息作出不同的响应。简单的说就是用同样的对象引用调用同样的方法但是做了不同的事情。

多态性分为编译时的多态性和运行时的多态性。

如果将对象的方法视为对象向外界提供的服务，那么运行时的多态性可以解释为：当 A 系统访问 B 系统提供的服务时，B 系统有多种提供服务的方式， 但一切对 A 系统来说都是透明的。

方法重载（overload）实现的是编译时的多态性（也称为前绑定），而方法重写 （override）实现的是运行时的多态性（也称为后绑定）。

运行时的多态是面向对象最精髓的东西，要实现多态需要做两件事：方法重写（子类继承父类并重写父类中已有的或抽象的方法）；对象造型（用父类型引用引用子类型对象，这样同样的引用调用同样的方法就会根据子类对象的不同而表现出不同的行为）。



### 四、注解

#### 1. 编译期注解处理的是字节码还是java文件

java文件。原理就是读入Java源代码，解析注解，然后生成新的Java代码。新生成的Java代码最后被编译成Java字节码。

#### 2. 说说你对注解的了解，是怎么解析的

详见《Java基础总结》第四章

注解(Annotation)在JDK1.5之后增加的一个新特性。注解作为程序的元数据嵌入到程序。注解可以被解析工具或编译工具解析。

Annotation能被用来为程序元素（类、方法、成员变量等）设置元素据。Annotaion不影响程序代码的执行，无论增加、删除Annotation，代码都始终如一地执行。如果希望让程序中的Annotation起一定的作用，只有通过解析工具或编译工具对Annotation中的信息进行解析和处理。

通过反射技术来解析自定义注解。关于反射类位于包java.lang.reflect，其中有一个接口AnnotatedElement，该接口主要有如下几个实现类：Class，Constructor，Field，Method，Package。除此之外，该接口定义了注释相关的几个核心方法，如下：

| 返回值       | 方法                                                         | 解释                                                     |
| ------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| T            | getAnnotation(Class annotationClass)                         | 当存在该元素的指定类型注解，则返回响应注释，否则返回null |
| Annotation[] | getAnnotation()                                              | 返回此元素上存在的所有注释                               |
| Annotation[] | getDeclaredAnnotation()                                      | 返回直接存在于此元素上存在的所有注释                     |
| boolean      | isAnnotationPresent(Class<? extends Annotation> annotationClass) | 当存在该元素的指定类型注解，则返回true，否则返回false    |

因此，当获取了某个类的Class对象，然后获取其Field,Method等对象，通过上述4个方法提取其中的注解，然后获得注解的详细信息。

### 五、序列化



### 六、集合

#### 1. Java 集合，介绍下ArrayList 和 HashMap 的使用场景，底层实现原理

《Java基础》第一章7节和9节

ArrayList：以数组实现。节约空间，但数组有容量限制。超出限制时会**增加50%容量**，用System.arraycopy()复制到新的数组，因此最好能给出数组大小的预估值。**默认第一次插入元素时创建大小为10的数组**。按数组下标访问元素—get(i)/set(i,e) 的性能很高，这是数组的基本优势。直接在数组末尾加入元素—add(e)的性能也高，但如果按下标插入、删除元素—add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势。

HashMap：基于Map接口实现、允许null键/值、非同步、不保证有序(比如插入的顺序)、也不保证序不随时间变化。当bucket中的entries的数目大于`capacity*load factor`时就需要调整bucket的大小为当前的2倍。

通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过`Load Facotr`则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

#### 2. ArrayList 与 LinkedList 的区别

ArrayList 和 Vector 使用了数组的实现，可以认为 ArrayList 或者 Vector 封装了对内部数组的操作，比如向数组中添加，删除，插入新的元素或者数据的扩展和重定向。

LinkedList 使用了循环双向链表数据结构。与基于数组的 ArrayList 相比，这是两种截然不同的实现技术，这也决定了它们将适用于完全不同的工作场景。

LinkedList 链表由一系列表项连接而成。一个表项总是包含 3 个部分：元素内容，前驱表和后驱表。在 JDK 的实现中，无论 LikedList 是否 为空，链表内部都有一个 header 表项，它既表示链表的开始，也表示链表的结尾。表项 header 的后驱表项便是链表中第一个元素，表项 header 的前驱表项便是链表中最后一个元素。

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

当然这是存入的过程。这里需要注意的是：

- `key`的hash值计算过程是高16位不变，低16位和高16位取异或，让更多位参与进来，可以有效的减少碰撞的发生。
- 初始数组容量为16，默认不超过的比例为0.75。

#### 4. 讲讲LinkedHashMap的数据结构

- 底层是散列表和双向链表
- 允许为null，不同步
- 插入的顺序是有序的(底层链表致使有序)
- 装载因子和初始容量对LinkedHashMap影响是很大的

`put`函数在LinkedHashMap中未重新实现，只是实现了`afterNodeAccess`和`afterNodeInsertion`两个回调函数。`get`函数则重新实现并加入了`afterNodeAccess`来保证访问顺序。

LinkedHashMap的操作基本上都是为了维护好那个具有访问顺序的双向链表。

#### 5. 链表和数组使用场景
数组应用场景：数据比较少；经常做的运算是按序号访问数据元素；数组更容易实现，任何高级语言都支持；构建的线性表较稳定。
链表应用场景：对线性表的长度或者规模难以估计；频繁做插入删除操作；构建动态性比较强的线性表。

#### 6. 要对集合更新操作时，ArrayList 和 LinkedList 哪个更适合？

ArrayList 和 LinkedList 在性能上各有优缺点，都有各自所适用的地方，总的说来可以描述如下： 

- 对 ArrayList 和 LinkedList 而言，在列表末尾增加一个元素所花的开销都是固定的。对 ArrayList 而言，主要是在内部数组中增加一项，指向所添加的元素，偶尔可能会导致对数组重新进行分配；而对 LinkedList 而言，这个开销是统一的，分配一个内部 Entry 对象。
- 在 ArrayList 的中间插入或删除一个元素意味着这个列表中剩余的元素都会被移动；而在 LinkedList 的中间插入或删除一个元素的开销是固定的。
- LinkedList 不支持高效的随机元素访问。
- ArrayList 的空间浪费主要体现在在 list 列表的结尾预留一定的容量空间，而 LinkedList 的空间花费则体现在 它的每一个元素都需要消耗相当的空间

可以这样说：当操作是在一列数据的后面添加数据而不是在前面或中间,并且需要随机地访问其中的元素时,使用ArrayList 会提供比较好的性能；当你的操作是在一列数据的前面或中间添加或删除数据，并且按照顺序访问其中的元素时，应该使用 LinkedList 了。

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

Java泛型的处理几乎都在编译器中进行，编译器生成的字节码是不包涵泛型信息的，泛型类型信息将在编译处理是被擦除，这个过程即类型擦除。

通常情况下，Java是通过以下方式处理泛型：Java编译器通过Code sharing方式为每个泛型类型创建唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类形实例映射到唯一的字节码表示是通过类型擦除（type erasue）实现的。

> Code sharing：对每个泛型类只生成唯一的一份目标代码；该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。

#### 4. 泛型有什么优点？

- 类型安全，避免类型的强转。

- 提高了代码的可读性，不必要等到运行的时候才去强制转换。



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

Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

Java中的反射首先是能够获取到 Java 中要反射类的字节码 ， 获取字节码有三种方法:

- `Class.forName(className)` 
- `类名.class`
- `this.getClass()`

然后将字节码中的方法，变量，构造函数等映射成相应的 Method、Filed、Constructor 等类，这些类提供了丰富的方法可以被我们所使用。

#### 3. 代理模式与装饰模式的区别，手写一个静态代理，一个动态代理

对装饰器模式来说，装饰者（decorator）和被装饰者（decoratee）都实现同一个接口。

对代理模式来说，代理类（proxy class）和真实处理的类（real class）都实现同一个接口。

他们之间的边界确实比较模糊，两者都是对类的方法进行扩展，具体区别如下：

- 装饰器模式强调的是增强自身，在被装饰之后你能够在被增强的类上使用增强后的功能。增强后你还是你，只不过能力更强了而已；代理模式强调要让别人帮你去做一些本身与你业务没有太多关系的职责（记录日志、设置缓存）。代理模式是为了实现对象的控制，因为被代理的对象往往难以直接获得或者是其内部不想暴露出来。

- 装饰模式是以对客户端透明的方式扩展对象的功能，是继承方案的一个替代方案；代理模式则是给一个对象提供一个代理对象，并由代理对象来控制对原有对象的引用；

- 装饰模式是为装饰的对象增强功能；而代理模式对代理的对象施加控制，但不对对象本身的功能进行增强；

静态代理:

```java
public class ProxyDemo {
    public static void main(String args[]){
        RealSubject subject = new RealSubject();
        Proxy p = new Proxy(subject);
        p.request();
    }
}

interface Subject{
    void request();
}

class RealSubject implements Subject{
    public void request(){
        System.out.println("request");
    }
}

class Proxy implements Subject{
    private Subject subject;
    public Proxy(Subject subject){
        this.subject = subject;
    }
    public void request(){
        System.out.println("PreProcess");
        subject.request();
        System.out.println("PostProcess");
    }
}
```

动态代理:

```java
public class DynamicProxyDemo {
    public static void main(String[] args) {
        //1.创建目标对象
        RealSubject realSubject = new RealSubject();    
        //2.创建调用处理器对象
        ProxyHandler handler = new ProxyHandler(realSubject);    
       //3.动态生成代理对象
        Subject proxySubject = (Subject)Proxy.newProxyInstance(RealSubject.class.getClassLoader(),
                                                        RealSubject.class.getInterfaces(), handler);   
        //4.通过代理对象调用方法   
        proxySubject.request();    
    }
}

/**
 * 主题接口
 */
interface Subject{
    void request();
}

/**
 * 目标对象类
 */
class RealSubject implements Subject{
    public void request(){
        System.out.println("====RealSubject Request====");
    }
}
/**
 * 代理类的调用处理器
 */
class ProxyHandler implements InvocationHandler{
    private Subject subject;
    public ProxyHandler(Subject subject){
        this.subject = subject;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        //定义预处理的工作，当然你也可以根据 method 的不同进行不同的预处理工作
        System.out.println("====before====");
       //调用RealSubject中的方法
        Object result = method.invoke(subject, args);
        System.out.println("====after====");
        return result;
    }
}
```

#### 4. 动态代理有什么作用

- 不需要为`RealSubject`写一个形式完全一样的代理类。

- 使用一些动态代理的方法可以在运行时制定代理类的逻辑，从而提升系统的灵活性。

#### 5. 反射可以反射final修饰的字段吗？

- 当final修饰的成员变量在定义的时候就初始化了值，那么java反射机制就已经不能动态修改它的值了。

- 当final修饰的成员变量在定义的时候并没有初始化值的话，那么就还能通过java反射机制来动态修改它的值。

### 九、IO

#### 1. Java 中有几种类型的流 

字节流和字符流。

字节流继承于 InputStream 和 OutputStream，字符流继承于 Reader 和 Writer。

#### 2. 字节流如何转为字符流?

字节输入流转字符输入流通过 InputStreamReader 实现，该类的构造函数可以传入 InputStream 对象。 

字节输出流转字符输出流通过 OutputStreamWriter 实现，该类的构造函数可以传入 OutputStream 对象。

#### 3. 如何将一个 java 对象序列化到文件里

在 java 中能够被序列化的类必须先实现 Serializable 接口，该接口没有任何抽象方法只是起到一个标记作用

```java
//对象输出流
ObjectOutputStream objectOutputStream = new ObjectOutputStream(new
FileOutputStream(new File("D://obj")));
objectOutputStream.writeObject(new User("zhangsan", 100));
objectOutputStream.close();
//对象输入流
ObjectInputStream objectInputStream = new ObjectInputStream(new
FileInputStream(new File("D://obj")));
User user = (User)objectInputStream.readObject();
System.out.println(user);
objectInputStream.close();
```

#### 4. 字节流和字符流的区别

字节流和字符流的区别：

- 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。

- 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。

只要是处理纯文本数据，就优先考虑使用字符流。 除此之外都使用字节流。



### 十、深拷贝和浅拷贝

#### 1. 为什么要用 clone？

首先，克隆对象是很有必要的，当一个对象需要被多人操作，但是又不想相互影响，需要保持原对象的状态，这时，就会克隆很多个相同的对象。

####  2. new 和 clone 一个对象的过程区别？

new 操作符的本意是分配内存。程序执行到 new 操作符时， 首先去看 new 操作符后面的类型，因为知道了类型，才能知道要分配多大的内存空间。分配完内存之后，再调用构造函数，填充对象的各个域，这一步叫做对象的初始化，构造方法返回后，一个对象创建完毕，可以把他的引用（地址）发布到外部，在外部就可以使用这个引用操纵这个对象。

clone 在第一步是和 new 相似的， 都是分配内存，调用 clone 方法时，分配的内存和原对象（即调用 clone 方 法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone 方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部。

#### 3. clone 对象的使用？深拷贝和浅拷贝？

实现：

```java
public class 对象 implements Cloneable{
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return (对象)super.clone();
    }
}
```

使用：`对象#clone();`

深拷贝：如果想要深拷贝一个对象，这个对象必须要实现 Cloneable 接口，实现 clone 方法，并且在 clone 方法内部，把该对象引用的其他对象也要 clone 一份，这就要求这个被引用的对象必须也要实现 Cloneable 接口并且实现 clone 方法。

### 十一、异常处理

#### 1. Java 中异常分为哪些种类

分为编译时异常 (CheckedException)  和运行时异常(RuntimeException) 。

只有 java 语言提供了编译时异常，Java 认为编译时异常都是可以被处理的异常， 所以 Java 程序必须显式处理 编译时异常。如果程序没有处理 Checked 异常，该程序在编译时就会发生错误无法编译。这体现了 Java 的设计哲学：没有完善错误处理的代码根本没有机会被执行。

对编译时异常处理方法有两种：

- 当前方法知道如何处理该异常，则用 try...catch 块来处理该异常。

- 当前方法不知道如何处理，则在定义该方法是声明抛出该异常。 

运行时异常只有当代码在运行时才发行的异常，编译时不需要 try catch。Runtime 如除数是 0 和数组下标越界等，其产生频繁，处理麻烦，若显示申明或者捕获将会对程序的可读性和运行效率影响很大。所以由系统自动 检测并将它们交给缺省的异常处理程序。当然如果你有处理要求也可以显示捕获它们。

#### 2. 调用下面的方法，得到的返回值是什么?

```java
public int getNum(){
	try {
    	int a = 1/0;
   	 	return 1;
    } catch (Exception e) {
    	return 2;
    }finally{
    	return 3;
    }
}
```

上面返回值是 3。

#### 3. error 和 exception 的区别

Error 类和 Exception 类的父类都是 Throwable 类，他们的区别是：

Error 类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢等。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和和预防，遇到这样的错误，建议让程序终止。 

Exception 类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常。

Exception 类又分为运行时异常（Runtime Exception）和编译时异常(Checked Exception )，运行时异常: ArithmaticException,IllegalArgumentException，编译能通过，但是一运行就终止了，程序不会处理运行时异常， 出现这类异常，程序会终止。而受检查的异常，要么用 try。。。catch 捕获，要么用 throws 字句声明抛出，交给它 的父类处理，否则编译不会通过

#### 4. 请写出你最常见的 5 个 RuntimeException

`NullPointerException`

`IndexOutOfBoundsException`

`ClassCastException`

`ClassNotFoundException`

`IllegalArgumentException`

`UnsupportedOperationException`



### 十二、其他

#### 1. 静态方法，静态对象能不能继承?

静态属性和静态方法是可以被继承的，但是不能被子类重写。

如果父类中定义的静态方法在子类中被重新定义，那么在父类中定义的静态方法将被隐藏。可以使用语法：父类名.静态方法调用隐藏的静态方法。 

如果父类中含有一个静态方法，且在子类中也含有一个返回类型、方法名、参数列表均与之相同的静态方法，那么该子类实际上只是将父类中的该同名方法进行了隐藏，而非重写。换句话说，父类和子类中含有的其实是两个没有关系的方法，它们的行为也并不具有多态性 

因此，通过一个指向子类对象的父类引用变量来调用父子同名的静态方法时，只会调用父类的静态方法。

#### 2. 如何封装一个字符串转数字的工具类？

```java
/**
 * 数字转换算法工具
 */
public class DecimalUtil {
    private static Pattern pattern = Pattern.compile("^[-\\+]?[.\\d]*$");
 
    /**
     * 四舍五入算法 取整
     */
    public static String roundHalfUp(String str) {
        if (TextUtils.isEmpty(str)) {
            return "0";
        }
        if (!isNum(str)) {
            return "0";
        }
        BigDecimal decimal = new BigDecimal(str);
        double num = decimal.setScale(0, BigDecimal.ROUND_HALF_UP).doubleValue();
        DecimalFormat format = new DecimalFormat("##0");
        return format.format(num);
    }
 
    /**
     * 四舍五入算法 scale 精度,保留几位小数 0, 1 ,2 , 3.....
     */
    public static double roundHalfUp(String str, int scale) {
        if (TextUtils.isEmpty(str)) {
            return 0;
        }
        if (!isNum(str)) {
            return 0;
        }
        BigDecimal decimal = new BigDecimal(str);
        return decimal.setScale(scale, BigDecimal.ROUND_UP).doubleValue();
    }
 
    /**
     * 字符串转换成整形
     */
    public static int string2Int(String str) {
        if (TextUtils.isEmpty(str)) {
            return 0;
        }
        if (!isNum(str)) {
            return 0;
        }
        return Integer.parseInt(str);
    }
 
    /**
     * 字符串转换成float
     */
    public static float string2Float(String str) {
        if (TextUtils.isEmpty(str)) {
            return 0;
        }
        if (!isNum(str)) {
            return 0;
        }
        return Float.parseFloat(str);
    }
 
    /**
     * 判断字符串是否可以转成数字
     */
    public static boolean isNum(String str) {
        if (null == str || "".equals(str)) {
            return false;
        }
 
        return pattern.matcher(str).matches();
    }
}
```

####  4. Java 中的引用类型分类和区别，虚引用的使用场景？

强引用：当内存不足，JVM开始垃圾回收，对于强引用的对象，就算是出现了OOM也不会对该对象进行回收。

软引用：是一种相对强引用弱化了一些的引用，需要用`java.lang.ref.SoftReference`类来实现，对于只有软引用的对象来说，当系统内存充足时它不会被回收，当系统内存不足时它会被回收。

弱引用：需要用`java.lang.ref.WeakReference`类来实现，它比软引用的生存期更短，对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。

虚引用：需要`java.lang.ref.PhantomReference`类来实现。顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，虚引用必须和引用队列 ReferenceQueue 联合使用。

虚引用的主要作用是跟踪对象被垃圾回收的状态。仅仅是提供了一种确保对象被finalize以后，做某些事情的机制。

#### 5. Base64算法是什么，是加密算法吗？

- `Base64`是一种将二进制数据转换成64种字符组成的字符串的编码算法，主要用于非文本数据的传输，比如图片。可以将图片这种二进制数据转换成具体的字符串，进行保存和传输。
- 严格来说，不算。虽然它确实把一段二进制数据转换成另外一段数据，但是他的加密和解密是公开的，也就无秘密可言了。所以我更倾向于认为它是一种编码，每个人都可以用base64对二进制数据进行编码和解码。
- `面试加分项`：为了减少混淆，方便复制，减少数据长度，就衍生出一种base58编码。去掉了base64中一些容易混淆的数字和字母（数字0，字母O，字母I，数字1，符号+，符号/）
   大名鼎鼎的比特币就是用的改进后的base58编码，即`Base58Check`编码方式，有了校验机制，加入了hash值。

#### 6. Base64底层

Base64的原理比较简单，每当我们使用Base64时都会先定义一个类似这样的数组：

```
['A', 'B', 'C', ... 'a', 'b', 'c', ... '0', '1', ... '+', '/']
```

上面就是Base64的索引表，字符选用了"A-Z、a-z、0-9、+、/" 64个可打印字符，这是标准的Base64协议规定。在日常使用中我们还会看到“=”或“==”号出现在Base64的编码结果中，“=”在此是作为填充字符出现。

- 第一步，将待转换的字符串每三个字节分为一组，每个字节占8bit，那么共有24个二进制位。
- 第二步，将上面的24个二进制位每6个一组，共分为4组。
- 第三步，在每组前面添加两个0，每组由6个变为8个二进制位，总共32个二进制位，即四个字节。
- 第四步，根据Base64编码对照表（见下图）获得对应的值。

```
0　A　　17　R　　　34　i　　　51　z
1　B　　18　S　　　35　j　　　52　0
2　C　　19　T　　　36　k　　　53　1
3　D　　20　U　　　37　l　　　54　2
4　E　　21　V　　　38　m　　　55　3
5　F　　22　W　　　39　n　　　56　4
6　G　　23　X　　　40　o　　　57　5
7　H　　24　Y　　　41　p　　　58　6
8　I　　25　Z　　　42　q　　　59　7
9　J　　26　a　　　43　r　　　60　8
10　K　　27　b　　　44　s　　　61　9
11　L　　28　c　　　45　t　　　62　+
12　M　　29　d　　　46　u　　　63　/
13　N　　30　e　　　47　v
14　O　　31　f　　　48　w　　　
15　P　　32　g　　　49　x
16　Q　　33　h　　　50　y
```

从上面的步骤我们发现：

- Base64字符表中的字符原本用6个bit就可以表示，现在前面添加2个0，变为8个bit，会造成一定的浪费。因此，Base64编码之后的文本，要比原文大约三分之一。
- 为什么使用3个字节一组呢？因为6和8的最小公倍数为24，三个字节正好24个二进制位，每6个bit位一组，恰好能够分为4组。

如果字节数不足三个:

- 两个字节：两个字节共16个二进制位，依旧按照规则进行分组。此时总共16个二进制位，每6个一组，则第三组缺少2位，用0补齐，得到三个Base64编码，第四组完全没有数据则用“=”补上。因此，上图中“BC”转换之后为“QKM=”；
- 一个字节：一个字节共8个二进制位，依旧按照规则进行分组。此时共8个二进制位，每6个一组，则第二组缺少4位，用0补齐，得到两个Base64编码，而后面两组没有对应数据，都用“=”补上。因此，上图中“A”转换之后为“QQ==”；

Base64就是用6位（2的6次幂就是64）表示字符，因此成为Base64。同理，Base32就是用5位，Base16就是用4位。

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

synchronized关键字

ReentrantLock锁

Volatile关键字

CAS原子操作

#### 5. 如何让两个线程循环交替打印

synchronized锁+AtomicInteger

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class StrangePrinter {

    private int max;
    private AtomicInteger status = new AtomicInteger(1); // AtomicInteger保证可见性，也可以用volatile

    public StrangePrinter(int max) {
        this.max = max;
    }

    public static void main(String[] args) {
        StrangePrinter strangePrinter = new StrangePrinter(100);
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.submit(strangePrinter.new MyPrinter("Print1", 0));
        executorService.submit(strangePrinter.new MyPrinter("Print2", 1));
        executorService.shutdown();
    }

    class MyPrinter implements Runnable {
        private String name;
        private int type; // 打印的类型，0：代表打印奇数，1：代表打印偶数

        public MyPrinter(String name, int type) {
            this.name = name;
            this.type = type;
        }

        @Override
        public void run() {
            if (type == 1) {
                while (status.get() <= max) {
                    synchronized (StrangePrinter.class) { // 加锁，保证下面的操作是一个原子操作
                        // 打印偶数
                        if (status.get() <= max && status.get() % 2 == 0) { // 打印偶数
                            System.out.println(name + " - " + status.getAndIncrement());
                        }
                    }
                }
            } else {
                while (status.get() <= max) {
                    synchronized (StrangePrinter.class) { // 加锁
                        // 打印奇数
                        if (status.get() <= max && status.get() % 2 != 0) { // 打印奇数
                            System.out.println(name + " - " + status.getAndIncrement());
                        }
                    }
                }
            }
        }
    }
}
```

这里需要注意两点：

1. 用AtomicInteger保证多线程数据可见性。
2. 不要觉得synchronized加锁是多余的，如果没有加锁，线程1和线程2就可能出现不是交替打印的情况。如果没有加锁，设想线程1打印完了一个奇数后，线程2去打印下一个偶数，当执行完`status.getAndIncrement()`后，此时status又是奇数了，当此时cpu将线程2挂起，调度线程1，就会出现线程2还没来得及打印偶数，线程1就已经打印了下一个奇数的情况。就不符合题目要求了。因此这里加锁是必须的，保证代码块中的是一个原子操作。

使用Object的wait和notify实现

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class StrangePrinter2 {

    Object odd = new Object(); // 奇数条件锁
    Object even = new Object(); // 偶数条件锁
    private int max;
    private AtomicInteger status = new AtomicInteger(1); // AtomicInteger保证可见性，也可以用volatile

    public StrangePrinter2(int max) {
        this.max = max;
    }

    public static void main(String[] args) {
        StrangePrinter2 strangePrinter = new StrangePrinter2(100);
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.submit(strangePrinter.new MyPrinter("偶数Printer", 0));
        executorService.submit(strangePrinter.new MyPrinter("奇数Printer", 1));
        executorService.shutdown();
    }

    class MyPrinter implements Runnable {
        private String name;
        private int type; // 打印的类型，0：代表打印奇数，1：代表打印偶数

        public MyPrinter(String name, int type) {
            this.name = name;
            this.type = type;
        }

        @Override
        public void run() {
            if (type == 1) {
                while (status.get() <= max) { // 打印奇数
                    if (status.get() % 2 == 0) { // 如果当前为偶数，则等待
                        synchronized (odd) {
                            try {
                                odd.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    } else {
                        System.out.println(name + " - " + status.getAndIncrement()); // 打印奇数
                        synchronized (even) { // 通知偶数打印线程
                            even.notify();
                        }
                    }
                }
            } else {
                while (status.get() <= max) { // 打印偶数
                    if (status.get() % 2 != 0) { // 如果当前为奇数，则等待
                        synchronized (even) {
                            try {
                                even.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                    } else {
                        System.out.println(name + " - " + status.getAndIncrement()); // 打印偶数
                        synchronized (odd) { // 通知奇数打印线程
                            odd.notify();
                        }
                    }
                }
            }
        }
    }
}
```

使用ReentrantLock+Condition实现

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class StrangePrinter3 {

    private int max;
    private AtomicInteger status = new AtomicInteger(1); // AtomicInteger保证可见性，也可以用volatile
    private ReentrantLock lock = new ReentrantLock();
    private Condition odd = lock.newCondition();
    private Condition even = lock.newCondition();

    public StrangePrinter3(int max) {
        this.max = max;
    }

    public static void main(String[] args) {
        StrangePrinter3 strangePrinter = new StrangePrinter3(100);
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.submit(strangePrinter.new MyPrinter("偶数Printer", 0));
        executorService.submit(strangePrinter.new MyPrinter("奇数Printer", 1));
        executorService.shutdown();
    }

    class MyPrinter implements Runnable {
        private String name;
        private int type; // 打印的类型，0：代表打印奇数，1：代表打印偶数

        public MyPrinter(String name, int type) {
            this.name = name;
            this.type = type;
        }

        @Override
        public void run() {
            if (type == 1) {
                while (status.get() <= max) { // 打印奇数
                    lock.lock();
                    try {
                        if (status.get() % 2 == 0) {
                            odd.await();
                        }
                        if (status.get() <= max) {
                            System.out.println(name + " - " + status.getAndIncrement()); // 打印奇数
                        }
                        even.signal();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            } else {
                while (status.get() <= max) { // 打印偶数
                    lock.lock();
                    try {
                        if (status.get() % 2 != 0) {
                            even.await();
                        }
                        if (status.get() <= max) {
                            System.out.println(name + " - " + status.getAndIncrement()); // 打印偶数
                        }
                        odd.signal();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            }
        }
    }
}
```

这里的实现思路其实和使用Object的wait和notify机制差不多。

通过flag标识实现

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class StrangePrinter4 {

    private int max;
    private AtomicInteger status = new AtomicInteger(1); // AtomicInteger保证可见性，也可以用volatile
    private boolean oddFlag = true;

    public StrangePrinter4(int max) {
        this.max = max;
    }

    public static void main(String[] args) {
        StrangePrinter4 strangePrinter = new StrangePrinter4(100);
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.submit(strangePrinter.new MyPrinter("偶数Printer", 0));
        executorService.submit(strangePrinter.new MyPrinter("奇数Printer", 1));
        executorService.shutdown();
    }

    class MyPrinter implements Runnable {
        private String name;
        private int type; // 打印的类型，0：代表打印奇数，1：代表打印偶数

        public MyPrinter(String name, int type) {
            this.name = name;
            this.type = type;
        }

        @Override
        public void run() {
            if (type == 1) {
                while (status.get() <= max) { // 打印奇数
                    if (oddFlag) {
                        System.out.println(name + " - " + status.getAndIncrement()); // 打印奇数
                        oddFlag = !oddFlag;
                    }
                }
            } else {
                while (status.get() <= max) { // 打印偶数
                    if (!oddFlag) {
                        System.out.println(name + " - " + status.getAndIncrement()); // 打印奇数
                        oddFlag = !oddFlag;
                    }
                }
            }
        }
    }
}
```

这是最简单最高效的实现方式，因为不需要加锁。比前面两种实现方式都要好一些。

#### 6. 怎么中止一个线程，Thread.Interupt一定有效吗？

停止一个线程可以用Thread.stop()方法，但最好不要用它。虽然它确实可以停止一个正在运行的线程，但是这个方法是不安全的，而且是已被废弃的方法。

在Java中有以下3种方法可以终止正在运行的线程：

1. 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop方法强行终止，但是不推荐这个方法，因为stop和suspend及resume一样都是过期作废的方法。
3. 使用interrupt方法中断线程。

调用Thread.Interupt方法不一定能中断线程，需要与线程中检测interrupt状态的方法配合使用。

无阻塞的情况下要保证interrupt状态仅在while判断时重置，不能受其他部分影响。

```java
	while(!Thread.interrupted()) {	// interrupt状态，要保证循环中不会重置这个值
		// do sth...
	}
```

有阻塞的情况下要保证interrupt状态不被吞，可以在catch块中再次调用interrupt()方法设置interrupt状态。

```java
	while(!Thread.interrupted()) {	// interrupt状态，要保证循环中不会重置这个值
		// do sth...
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			// do sth...
			// 在这里调了一次interrupt()，此时线程未处于阻塞状态，会设置interrupt状态
			Thread.currentThread().interrupt();
		}
	}
```

#### 7. 实现一个线程同步的计数器

synchronized

```java
public class Test {
	//公共变量
	int count=0;
	public static void main(String[] args){
		//new一个实现Runnable的类
		Test test=new Test();
		//创建1个任务
		MyRunnable myRunnable=test.new MyRunnable();
		//创建5个线程
		for(int i=0;i<4;i++){
			new Thread(myRunnable).start();
		}
	}
	//创建一个实现Runnable的类
	class MyRunnable implements Runnable{
		public void run() {
			while(true){
				//锁住的是同一对象
				synchronized(this){
					if(count>=1000){
						break;
					}
					System.out.println(Thread.currentThread().getName()+":count:"+(++count));
					//测试时，线程更容易切换
					Thread.yield();
				}
			}
		}
		
	}
 
}
```

AtomicInteger

```java
    static AtomicInteger count=new AtomicInteger(0);
    //公共变量
    //int count=0;
    public static void main(String[] args){
        //new一个实现Runnable的类
        Solution015 test=new Solution015();
        //创建1个任务
        MyRunnable myRunnable=test.new MyRunnable();
        //创建5个线程
        for(int i=0;i<4;i++){
            new Thread(myRunnable).start();
        }
    }
    //创建一个实现Runnable的类
    class MyRunnable implements Runnable{
        public void run() {
            while(true){
                if(count.get()>=1000){
                    break;
                }
                System.out.println(Thread.currentThread().getName()+":count:"+count.getAndIncrement());
                //测试时，线程更容易切换
                Thread.yield();
            }
        }
    }
```

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

单线程线程池中只有一个工作线程，可以保证添加的任务都以指定顺序执行（先进先出、后进先出、优先级）。

主要有两个优点：

- 可以通过共享的线程池很方便地提交任务进行异步执行，而不用自己管理线程的生命周期；

- 可以使用任务队列并指定任务的执行顺序，很容易做到任务管理的功能。

### 三、锁

#### 1. 死锁触发的四大条件？

- 互斥锁

- 请求与保持

- 不可剥夺

- 循环的请求与等待

#### 2. synchronized关键字的使用？

synchronized的参数放入对象和Class有什么区别？synchronized 修饰 static 方法、普通方法、类、方法块区别？

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

#### 7. synchronized是公平锁还是非公平锁,ReteranLock是公平锁吗？

synchronized是非公平锁；ReteranLock是非公平锁还是公平锁和设置有关。

#### 8.可见性，原子性，有序性，synchronized可以保证什么？

可见性，原子性，有序性都可以保证。

#### 9. 构造一个出现死锁的情况

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

#### 10. synchronized 的 4 种状态

> - [不可不说的Java“锁”事](https://links.jianshu.com/go?to=https%3A%2F%2Ftech.meituan.com%2F2018%2F11%2F15%2Fjava-lock.html)
> - 访问 synchronized 修饰static方法、synchronized(this|object) 是否会冲突受干扰

#### 11. 访问 synchronized 修饰 static 方法、synchronized (类.class) 是否会冲突受干扰

> [synchronized 详解](https://links.jianshu.com/go?to=https%3A%2F%2Fjuejin.im%2Fpost%2F594a24defe88c2006aa01f1c%23heading-5)

**Synchronized修饰非静态方法**，实际上是对调用该方法的对象加锁，俗称“对象锁”。也就是锁住的是这个对象，即this。如果同一个对象在两个线程分别访问对象的两个同步方法，就会产生互斥，这就是对象锁，一个对象一次只能进入一个操作。

**Synchronized修饰静态方法**，实际上是对该类对象加锁，俗称“类锁”。也就是锁住的是这个类，即xx.class。如果一个对象在两个线程中分别调用一个静态同步方法和一个非静态同步方法，由于静态方法会收到类锁限制，但是非静态方法会收到对象限制，所以两个方法并不是同一个对象锁，因此不会排斥。

#### 12. 同一个类中的 2 个方法都加了同步锁，多个线程能同时访问同一个类中的这两个方法吗？

这个问题需要考虑到 Lock 与 synchronized 两种实现锁的不同情形：

#### 13. 什么情况下导致线程死锁，遇到线程死锁该怎么解决？

死锁是指多个线程因竞争资源而造成的一种僵局（互相等待），若无外力作用，这些进程都将无法向前推进。

死锁产生的必要条件： 

- 互斥条件：线程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个线程所 占有。此时若有其他线程请求该资源，则请求线程只能等待。 

- 不剥夺条件：线程所获得的资源在未使用完毕之前，不能被其他线程强行夺走，即只能由获得该资源的线程 自己来释放（只能是主动释放)。 

- 请求和保持条件：线程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他线程占有， 此时请求进程被阻塞，但对自己已获得的资源保持不放。

- 循环等待条件：存在一种线程资源的循环等待链，链中每一个线程已获得的资源同时被链中下一个线程所请 求。

在有些情况下死锁是可以避免的。两种用于避免死锁的技术：

1）加锁顺序（线程按照一定的顺序加锁）

2）加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）



### 四、线程间通信

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

### 五、Volatile 

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
- 使用`volatile`而不是`synchronized`的唯一安全的情况是：类中只有一个可变的域。

### 六、阻塞队列

#### 1. 通常的阻塞队列有哪几种，特点是什么？

- `ArrayBlockQueue`：基于数组实现的有界的FIFO(先进先出)阻塞队列。
- `LinkedBlockQueue`：基于链表实现的无界的FIFO(先进先出)阻塞队列。
- `SynchronousQueue`：内部没有任何缓存的阻塞队列。
- `PriorityBlockingQueue`：具有优先级的无限阻塞队列。

#### 2. ConcurrentHashMap的原理

数据结构的实现跟HashMap一样，不做介绍。

JDK 1.8之前采用的是分段锁，核心类是一个`Segment`，`Segment`继承了`ReentrantLock`，每个`Segment对象`管理若干个桶，多个线程访问同一个元素的时候只能去竞争获取锁。

JDK 1.8采用了`CAS + synchronized`，插入键值对的时候如果当前桶中没有Node节点，使用CAS方式进行更新，如果有Node节点，则使用synchronized的方式进行更新。

### 七、协程，纤程(Java Kotlin)

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



### 八、其他

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

#### 4. heap 和 stack 有什么区别

**申请方式** 

stack:由系统自动分配。例如，声明在函数中一个局部变量 int b; 系统自动在栈中为 b 开辟空间 

heap:需要程序员自己申请，并指明大小，在 c 中 malloc 函数，对于 Java 需要手动 new Object()的形式开辟

**申请后系统的响应** 

stack：只要栈的剩余空间大于所申请空间，系统将为程序提供内存，否则将报异常提示栈溢出。 

heap：首先应该知道操作系统有一个记录空闲内存地址的链表，当系统收到程序的申请时， 会遍历该链表，寻找第一个空间大于所申请空间的堆结点，然后将该结点从空闲结点链表中删除，并将该结点的空间分配给程序。另外，由于找到的堆结点的大小不一定正好等于申请的大小，系统会自动的将多余的那部分重新放入空闲链表中。

**申请大小的限制** 

stack：栈是向低地址扩展的数据结构，是一块连续的内存的区域。这句话的意思是栈顶的地址和栈的最大容量是系统预先规定好的，如果 申请的空间超过栈的剩余空间时，将提示 overflow。因此，能从栈获得的空间较小。

heap：堆是向高地址扩展的数据结构，是不连续的内存区域。这是由于系统是用链表来存储的空闲内存地址的， 自然是不连续的，而链表的遍历方向是由低地址向高地址。堆的大小受限于计算机系统中有效的虚拟内存。由此可见， 堆获得的空间比较灵活，也比较大。

**申请效率的比较**

stack：由系统自动分配，速度较快。但程序员是无法控制的。 

heap：由 new 分配的内存，一般速度比较慢，而且容易产生内存碎片,不过用起来最方便。

**heap 和 stack 中的存储内容**

stack： 在函数调用时，第一个进栈的是主函数中后的下一条指令（函数调用语句的下一条可执行语句）的地址， 然后是函数的各个参数，在大多数的 C 编译器中，参数是由右往左入栈的，然后是函数中的局部变量。注意静态变量 是不入栈的。 

当本次函数调用结束后，局部变量先出栈，然后是参数，最后栈顶指针指向最开始存的地址，也就是主函数中的 下一条指令，程序由该点继续运行。 

heap：一般是在堆的头部用一个字节存放堆的大小。堆中的具体内容有程序员安排。

**数据结构层面的区别**

还有就是数据结构方面的堆和栈，这些都是不同的概念。这里的堆实际上指的就是（满足堆性质的）优先队列的 一种数据结构，第 1 个元素有最高的优先权；

栈实际上就是满足先进后出的性质的数学或数据结构。 虽然堆栈，堆栈的说法是连起来叫，但是他们还是有很大区别的，连着叫只是由于历史的原因。

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

1. 类加载过程（静态属性、静态方法声明--静态属性赋值、静态代码块）注意先父类后子类
2. 实例化过程（普通属性、普通方法声明--普通属性赋值、构造代码块--构造方法中的代码）也是先父类后子类
3. 如果类的加载过程中调用了实例化过程（如new了本类对象），则回暂停类加载过程先执行实例化过程，执行完再回到类加载过程。

#### 4. PathClassLoader与DexClassLoader有什么区别

[谈谈 Android 中的 PathClassLoader 和 DexClassLoader](https://juejin.cn/post/6844903929562529800)

PathClassLoader 和 DexClassLoader **都能加载外部的 dex／apk**，只不过区别是 DexClassLoader 可以**指定 optimizedDirectory**，也就是 dex2oat 的产物 .odex 存放的位置，而 PathClassLoader 只能使用系统默认位置。但是这个 optimizedDirectory 在 Android 8.0 以后也被舍弃了，只能使用系统默认的位置了。

#### 5. class文件的组成？常量池里面有什么内容？

[Java .class文件结构与常量池](https://zhuanlan.zhihu.com/p/117621212)

#### 6. 自动装箱发生在什么时候？编译期还是运行期

编译时自动完成替换的，装箱阶段自动替换为了 valueOf 方法，拆箱阶段自动替换为了 xxxValue 方法。对于 Integer 类型的 valueOf 方法参数如果是 -128~127 之间的值会直接返回内部缓存池中已经存在对象的引用，参数是其他范围值则返回新建对象；而 Double 类型与 Integer 类型类似，一样会调用 Double 的 valueOf 方法，但是 Double 的区别在于不管传入的参数值是多少都会 new 一个对象来表达该数值（因为在指定范围内浮点型数据个数是不确定的，整型等个数是确定的，所以可以 Cache）。

注意：Integer、Short、Byte、Character、Long 的 valueOf 方法实现类似，而 Double 和 Float 比较特殊，每次返回新包装对象，对于两边都是包装类型的比较 = = 比较的是引用，equals 比较的是值，对于两边有一边是表达式（包含算数运算）则 = = 比较的是数值（自动触发拆箱过程），对于包装类型 equals 方法不会进行类型转换。

#### 7.java和字节码有什么区别？

.java源程序是指未编译的按照一定的程序设计语言规范书写的文本文件

字节码文件就是以.CLASS文件结尾的文件，是通过JAVAC命令编译过生成的。因为JAVA不是编译型语言，所以它zhuan需要去解释字节码shu文件才能够运行。

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

#### 10. 类什么时候被初始化？

1）创建类的实例，也就是 new 一个对象

2）访问某个类或接口的静态变量，或者对该静态变量赋值 

3）调用类的静态方法 

4）反射（Class.forName("com.zza.test")） 

5）初始化一个类的子类（会首先初始化子类的父类） 

6）JVM 启动时标明的启动类，即文件名和类名相同的那个类

类的初始化步骤：

1）如果这个类还没有被加载和链接，那先进行加载和链接 

2）假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一 次），那就初始化直接的父类（不适用于接口） 

3）加入类中存在初始化语句（如 static 变量和 static 块），那就依次执行这些初始化语句。

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
- **对象实例收集**主要是两种基本算法，[引用计数](https://links.jianshu.com/go?to=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2F%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0)和可达性分析，Java 选择的可达性分析。JVM 会把虚拟机栈和本地方法栈中正在引用的对象、静态属性引用的对象和常量，作为 GC Roots。

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

#### 5. 圾回收的GCRoot是什么？

JVM 会把虚拟机栈和本地方法栈中正在引用的对象、静态属性引用的对象和常量，作为 GC Roots。

