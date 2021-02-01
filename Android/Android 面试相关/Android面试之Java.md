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

#### 5. 浮点数的精准计算

BigDecimal类进行商业计算，Float和Double只能用来做科学计算或者是工程计算。

#### 6. Object类的 equal 和 hashcode 方法重写，为什么？

**为什么要重写equals()方法？**

1. Object类中equals方法比较的是两个对象的引用地址，只有对象的引用地址指向同一个地址时，才认为这两个地址是相等的，否则这两个对象就不想等。
2. 如果有两个对象，他们的属性是相同的，但是地址不同，这样使用equals()比较得出的结果是不相等的，而我们需要的是这两个对象相等，因此默认的equals()方法是不符合我们的要求的，这个时候我们就需要对equals()方法进行重写以满足我们的预期结果。
3. 在java的集合框架中需要用到equals()方法进行查找对象，如果集合中存放的是自定义类型，并且没有重写equals()方法，则会调用Object父类中的equals()方法按照地址比较，往往会出现错误的结果，此时我们应该根据业务需求重写equals()方法。

**为什么要重写hashCode()方法？**

1. hashCode()方法用于散列数据的快速存储，HashSet/HashMap/Hashtable类存储数据时都是根据存储对象的hashcode值来进行分类存储的，一般先根据hashcode值在集合中进行分类，在根据equals()方法判断对象是否相同。
2. HashMap对象是根据其Key的hashCode来获取对应的Value。
3. 生成一个好的hashCode值能提高HashSet查找的性能，差的hashCode值不但不能提高性能，甚至可能造成错误。比如hashCode方法中返回常量，会让HashSet的查找效率退化为List集合的查找效率；hashCode方法中返回随机数，会让查找结果变的不可预测。
4. 好的hashCode生成方式是让对象中的关键属性与质数相乘,并将积相加获取。

**为什么java中在重写equals()方法后必须对hashCode()方法进行重写？**

1. 为了维护hashCode()方法的equals协定，该协定指出：如果根据 equals()方法，两个对象是相等的，那么对这两个对象中的每个对象调用 hashCode方法都必须生成相同的整数结果；而两个hashCode()返回的结果相等，两个对象的equals()方法不一定相等。
2. HashMap对象是根据其Key的hashCode来获取对应的Value。
3. 在重写父类的equals()方法时，也重写hashcode()方法，使相等的两个对象获取的HashCode值也相等，这样当此对象做Map类中的Key时，两个equals为true的对象其获取的value都是同一个，比较符合实际。

#### 7.  在重写equals方法时，需要遵循哪些约定，具体介绍一下？

重写equals方法时需要遵循通用约定：自反性、对称性、传递性、一致性、非空性

1）自反性

对于任何非null的引用值x,x.equals(x)必须返回true。---这一点基本上不会有啥问题

2）对称性

对于任何非null的引用值x和y，当且仅当x.equals(y)为true时，y.equals(x)也为true。

3）传递性

对于任何非null的引用值x、y、z。如果x.equals(y)==true,y.equals(z)==true,那么x.equals(z)==true。

4） 一致性

对于任何非null的引用值x和y，只要equals的比较操作在对象所用的信息没有被修改，那么多次调用x.equals(y)就会一致性地返回true,或者一致性的返回false。

5）非空性

所有比较的对象都不能为空。

```java
   @Override
    public boolean equals(Object o) {
        //自反性
        if (this == o) return true;
        //任何对象不等于null，比较是否为同一类型
        if (!(o instanceof Person)) return false;
        //强制类型转换
        Person person = (Person) o;
        //比较属性值
        return getId() == person.getId() &&
                Objects.equals(getName(), person.getName()) &&
                Objects.equals(getSex(), person.getSex());
    }
```

#### 8. 重写hashCode()方法?

重写hashCode()方法需要遵循hashCode()协定：

1. 一致性：在Java应用程序执行期间，在对同一对象多次调用hashCode方法时，必须一致地返回相同的整数，前提是将对象进行hashcode比较时所用的信息没有被修改。
2. equals：如果根据equals()方法比较，两个对象是相等的，那么对这两个对象中的每个对象调用hashCode()方法都必须生成相同的整数结果，注：这里说的equals()方法是指Object类中未被子类重写过的equals()方法。
3. 附加：如果根据equals()方法比较，两个对象不相等，那么对这两个对象中的任一对象上调用hashCode方法不一定生成不同的整数结果。但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。

```java
	@Override
    public int hashCode() {
        return Objects.hash(getId(), getName(), getSex());
    }
```



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

StringBuffer和StringBuilder是可变类，StringBuffer是线程安全的，StringBuilder则不是线程安全的。所以在多线程对同一个字符串操作的时候，我们应该选择用StringBuffer。由于不需要处理多线程的情况，StringBuilder的效率比StringBuffer高。

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

- 因为java字符串是不可变的，可以在java运行时**节省大量Java堆空间**。因为不同的字符串变量可以引用池中的相同的字符串。如果字符串是可变得话，任何一个变量的值改变，就会反射到其他变量，那字符串池也就没有任何意义了。

2）平时使用双引号方式赋值的时候其实是返回的字符串引用，并不是改变了这个字符串对象

#### 6. String的两种创建方式，在JVM的存储方式相同吗？
1） String常见的创建方式有两种

- String s1 = “Java”
- String s2 = new String("Java")

2）存储方式不同

- 第一种，s1会先去字符串常量池中找字符串"Java”，如果有相同的字符则直接返回常量句柄，如果没有此字符串则会先在常量池中创建此字符串，然后再返回常量句柄，或者说字符串引用。

- 第二种，s2是直接在堆上创建一个变量对象，但不存储到字符串池 ，调用`intern`方法才会把此字符串保存到常量池中

#### 7. Java String可以有多长？

分配到栈：

```java
String longString = "aaa...aaa";
```

分配到堆：

```java
byte[] bytes = loadFromFile(new File("superLongText.txt");
String superLongString = new String(bytes);
```

源文件：*.java

```java
String longString = "aaa...aaa";
字节数 <= 65535
```

字节码：*.class

```c
CONSTANT_Utf8_info { 
    u1 tag; 
    u2 length;
    (0~65535) u1 bytes[length]; 
    最多65535个字节 
}
```

javac的编译器有问题，< 65535应该改为 < = 65535。

**Java String 栈分配**

- 受字节码限制，字符串最终的字节数不超过65535。
- Latin字符，受Javac代码限制，最多65534个。
- 非Latin字符最终对应字节个数差异较大，最多字节个数是65535。
- 如果运行时方法区设置较小，也会受到方法区大小的限制。

**Java String 堆分配**

- 受虚拟机指令限制，字符数理论上限为Integer.MAX_VALUE。
- 受虚拟机实现限制，实际上限可能会小于Integer.MAX_VALUE。
- 如果堆内存较小，也会受到堆内存的限制。

new String(bytes)内部是采用了一个字符数组，其对应的虚拟机指令是newarray [int] ，数组理论最大个数为Integer.MAX_VALUE，有些虚拟机需要一些头部信息，所以MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8。

总结：

**Java String字面量形式**

- 字节码中CONSTANT_Utf8_info的限制
- Javac源码逻辑的限制
- 方法区大小的限制

**Java String运行时创建在堆上的形式**

- Java虚拟机指令newarray的限制
- Java虚拟机堆内存大小的限制

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

多态的定义：允许不同类对同一消息做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用）

多态存在的条件：

1. 要有继承。
2. 要有复写。
3. 父类引用指向子类对象。

Java中多态的实现方式：接口实现，继承父类进行方法重写，同一个类中的方法重载。

实现多态的技术称为：动态绑定，是指在执行期间判断所引用对象的实际类型，根据其实际的类型调用其相应的方法。

多态的作用：消除类型之间的耦合关系。

实现机制：靠的是父类或接口定义的引用变量可以指向子类或具体实现类的实例对象，而程序调用的方法在运行期才动态绑定。这个方法就是引用变量所指向的具体实例对象的方法，也就是内存里正在运行的那个对象的方法，而不是引用变量的类型中定义的方法。

多态的好处：

1.**可替换性**。多态对已存在代码具有可替换性。例如，多态对圆Circle类工作，对其他任何圆形几何体，如圆环，也同样工作。

2.**可扩充性**。多态对代码具有可扩充性。增加新的子类不影响已存在类的多态性、继承性，以及其他特性的运行和操作。实际上新加子类更容易获得多态功能。例如，在实现了圆锥、半圆锥以及半球体的多态基础上，很容易增添球体类的多态性。

3.**接口性**。多态是超类通过方法签名，向子类提供了一个共同接口，由子类来完善或者覆盖它而实现的。

4.**灵活性**。它在应用中体现了灵活多样的操作，提高了使用效率。

5.**简化性**。多态简化对应用软件的代码编写和修改过程，尤其在处理大量对象的运算和操作时，这个特点尤为突出和重要。

Java中多态的实现方式：接口实现，继承父类进行方法重写，同一个类中进行方法重载。

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

注解(Annotation)在JDK1.5之后增加的一个新特性。注解作为程序的元数据嵌入到程序。注解可以被解析工具或编译工具解析。

Annotation能被用来为程序元素（类、方法、成员变量等）设置**元素据**。Annotaion不影响程序代码的执行，无论增加、删除Annotation，代码都始终如一地执行。如果希望让程序中的Annotation起一定的作用，只有通过解析工具或编译工具对Annotation中的信息进行解析和处理。

通过反射技术来解析自定义注解。关于反射类位于包java.lang.reflect，其中有一个接口AnnotatedElement，该接口主要有如下几个实现类：Class，Constructor，Field，Method，Package。除此之外，该接口定义了注释相关的几个核心方法，如下：

| 返回值       | 方法                                                         | 解释                                                     |
| ------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| T            | getAnnotation(Class annotationClass)                         | 当存在该元素的指定类型注解，则返回响应注释，否则返回null |
| Annotation[] | getAnnotation()                                              | 返回此元素上存在的所有注释                               |
| Annotation[] | getDeclaredAnnotation()                                      | 返回直接存在于此元素上存在的所有注释                     |
| boolean      | isAnnotationPresent(Class<? extends Annotation> annotationClass) | 当存在该元素的指定类型注解，则返回true，否则返回false    |

因此，当获取了某个类的Class对象，然后获取其Field，Method等对象，通过上述4个方法提取其中的注解，然后获得注解的详细信息。

**自定义注解类型**

通过@interface关键字的方式，其实通过该方式会隐含地继承`Annotation`接口。

`@Documented`： @Documented 用户指定被该元Annotation修饰的Annotation类将会被javadoc工具提取成文档

`@Inherited`：@Inherited 指定被它修饰的Annotation将具有继承性。如果某个类使用了 @Xxx注解（定义该Annotation时使用了 @Inherited 修饰）修饰，则其子类将自 动被@Xxx修饰。

`@Retention`：表示该注解类型的注解保留的时长。当注解类型声明中没有 `@Retention` 元注解，则默认保留策略为`RetentionPolicy.CLASS`。保留策略(RetentionPolicy)是枚举类型，共定义3种保留策略

![img](https://camo.githubusercontent.com/6ed7adc069270fc38dc3975727e7c52eb0ef6f05/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d383238666536386663646638333462342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

`@Target`：表示该注解类型的所适用的程序元素类型。当注解类型声明中没有`@Target`元注解，则默认为可适用所有的程序元素。
								![img](https://camo.githubusercontent.com/9dec3ec2d382a6c13ed4f7742e9abf9b8a653c11/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d376234353764663231343366613564642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

示例：

```java
@Documented
@Target(ElementType.METHOD)
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotataion{
    String name();
    String website() default "hello";
    int revision() default 1;
}
```

```java
public class AnnotationDemo {
	@AuthorAnno(name="xxx", website="hello", revision=1)
    public static void main(String[] args) {
    	System.out.println("I am main method");
    }
    @SuppressWarnings({ "unchecked", "deprecation" })
    @AuthorAnno(name="xxx", website="hello", revision=2)
    public void demo(){
    	System.out.println("I am demo method");
    }
}

```

由于该注解的保留策略为 `RetentionPolicy.RUNTIME` ，故可在运行期通过反射机 制来使用，否则无法通过反射机制来获取。

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

#### 7. HashMap底层为什么是线程不安全的？

并发场景下使用时容易出现死循环，在 HashMap 扩容的时候会调用 resize() 方法，就是这里的并发操作容易在一个桶上形成环形链表；这样当获取一个不存在的 key 时，计算出的 index 正好是环形链表的下标就会出现死循环；

在 1.7 中 hash 冲突采用的头插法形成的链表，在并发条件下会形成循环链表，一旦有查询落到了这个链表上，当获取不到值时就会死循环。

#### 8. 集合框架，List，Map，Set都有哪些具体的实现类，区别都是什么?

Java集合里使用接口来定义功能，是一套完善的继承体系。Iterator是所有集合的总接口，其他所有接口都继承于它，该接口定义了集合的遍历操作，Collection接口继承于Iterator，是集合的次级接口（Map独立存在），定义了集合的一些通用操作。

Java集合的类结构图如下所示：

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095d7a20a86d81?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

List：有序、可重复；

Set：无序，不可重复；

Map：键值对，键唯一，值多个；

- List和Set都是继承自Collection接口，Map则不是;

- List特点：元素有放入顺序，元素可重复；Set特点：元素无放入顺序，元素不可重复，重复元素会盖掉；Map适合储存键值对的数据。

  > 元素虽然无放入顺序，但是元素在set中位置是由该元素的HashCode决定的，其位置其实是固定，加入Set 的Object必须定义equals()方法；另外list支持for循环，也就是通过下标来遍历，也可以使用迭代器，但是set只能用迭代，因为他无序，无法用下标取得想要的值

- 线程安全集合类与非线程安全集合类

  - LinkedList、ArrayList、HashSet是非线程安全的，Vector是线程安全的;

  - HashMap是非线程安全的，HashTable是线程安全的;

#### 9. ArrayList与Vector的区别和适用场景

ArrayList有三个构造方法：

```java
public ArrayList(intinitialCapacity)// 构造一个具有指定初始容量的空列表。   
public ArrayList()// 构造一个初始容量为10的空列表。
public ArrayList(Collection<? extends E> c)// 构造一个包含指定 collection 的元素的列表  
```

Vector有四个构造方法：

```java
public Vector() // 使用指定的初始容量和等于零的容量增量构造一个空向量。    
public Vector(int initialCapacity) // 构造一个空向量，使其内部数据数组的大小，其标准容量增量为零。    
public Vector(Collection<? extends E> c)// 构造一个包含指定 collection 中的元素的向量  
public Vector(int initialCapacity, int capacityIncrement)// 使用指定的初始容量和容量增量构造一个空的向量
```

ArrayList和Vector都是用数组实现的，主要有这么四个区别：

- **Vector是多线程安全的**，线程安全就是说多线程访问代码，不会产生不确定的结果。而ArrayList不是，这可以从源码中看出，Vector类中的方法很多有synchronied进行修饰，这样就导致了Vector在效率上无法与ArrayLst相比；
- 两个都是采用的线性连续空间存储元素，但是当空间充足的时候，两个类的增加方式是不同。
- Vector可以设置增长因子，而ArrayList不可以。
- Vector是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。

适用场景：

- Vector是线程同步的，所以它也是线程安全的，而ArraList是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用ArrayList效率比较高。

- 如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数据，用Vector有一定的优势。

#### 10. HashSet与TreeSet的区别和适用场景

TreeSet： 是红黑树的树据结构实现的，TreeSet中的数据是自动排好序的，不允许放入null值。

HashSet： 是哈希表实现的，HashSet中的数据是无序的可以放入null，但只能放入一个null，两者中的值都不重复，就如数据库中唯一约束。HashSet要求放入的对象必须实现HashCode()方法，并且，放入的对象，是以hashcode码作为标识的，而具有相同内容的String对象，hashcode是一样，所以放入的内容不能重复但是同一个类的对象可以放入不同的实例。

适用场景分析:

HashSet是基于Hash算法实现的，其性能通常都优于TreeSet。为快速查找而设计的Set，我们通常都应该使用HashSet，在我们需要排序的功能时，我们才使用TreeSet。

#### 11. HashMap与TreeMap、HashTable的区别及适用场景

HashMap：基于哈希表(散列表)实现。非线程安全，使用HashMap要求的键类明确定义了hashCode()和equals()[可以重写hasCode()和equals()]，为了优化HashMap空间的使用，您可以调优初始容量和负载因子。其中散列表的冲突处理主分两种，一种是开放定址法，另一种是链表法。HashMap实现中采用的是链表法。

HashTable:同样是基于哈希表实现的，同样每个元素是一个key-value对，其内部也是通过单链表解决冲突问题，容量不足（超过了阀值）时，同样会自动增长，线程安全的，能用于多线程环境中。

TreeMap：非线程安全基于红黑树实现。TreeMap没有调优选项，因为该树总处于平衡状态。

适用场景分析：

HashMap和HashTable：HashMap去掉了HashTable的contain方法，但是加上了containsValue()和containsKey()方法。HashTable是同步的，而HashMap是非同步的，效率上比HashTable要高。HashMap允许空键值，而HashTable不允许。

HashMap：适用于Map中插入、删除和定位元素。

Treemap：适用于按自然顺序或自定义顺序遍历键(key)。 

#### 12. Set集合从原理上如何保证不重复？

1）在往set中添加元素时，如果指定元素不存在，则添加成功。

2）具体来讲：当向HashSet中添加元素的时候，首先计算元素的hashcode值，然后用这个（元素的hashcode）%（HashMap集合的大小）+1计算出这个元素的存储位置，如果这个位置为空，就将元素添加进去；如果不为空，则用equals方法比较元素是否相等，相等就不添加，否则找一个空位添加。

#### 13. HashMap和HashTable的主要区别是什么？，两者底层实现的数据结构是什么？

HashMap和HashTable的区别：

二者都实现了Map 接口，是将唯一的键映射到特定的值上，主要区别在于：

1) HashMap 没有排序，允许一个null 键和多个null 值，而HashTable不允许；

2) HashMap 把Hashtable 的contains 方法去掉了，改成containsvalue 和containsKey, 因为contains 方法容易让人引起误解；

3) Hashtable 继承自Dictionary 类，HashMap 是Java1.2 引进的Map 接口的实现；

4) Hashtable 的方法是Synchronized 的，而HashMap 不是，在多个线程访问Hashtable 时，不需要自己为它的方法实现同步，而HashMap 就必须为之提供额外的同步。Hashtable 和HashMap 采用的hash/rehash 算法大致一样，所以性能不会有很大的差异。

HashMap和Hashtable的底层实现数据结构：都是数组 + 链表结构实现的（jdk8以前）

#### 14. HashMap、ConcurrentHashMap、hash()相关原理解析？

**HashMap 1.7的原理：**

HashMap 底层是基于 数组 + 链表 组成的，不过在 jdk1.7 和 1.8 中具体实现稍有不同。

负载因子：

- 给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。
- 因此通常建议能提前预估 HashMap 的大小最好，尽量的减少扩容带来的性能损耗。

其实真正存放数据的是 Entry<K,V>[] table，Entry 是 HashMap 中的一个静态内部类，它有key、value、next、hash（key的hashcode）成员变量。

put 方法：

- 判断当前数组是否需要初始化。
- 如果 key 为空，则 put 一个空值进去。
- 根据 key 计算出 hashcode。
- 根据计算出的 hashcode 定位出所在桶。
- 如果桶是一个链表则需要遍历判断里面的 hashcode、key 是否和传入 key 相等，如果相等则进行覆盖，并返回原来的值。
- 如果桶是空的，说明当前位置没有数据存入，新增一个 Entry 对象写入当前位置。（当调用 addEntry 写入 Entry 时需要判断是否需要扩容。如果需要就进行两倍扩充，并将当前的 key 重新 hash 并定位。而在 createEntry 中会将当前位置的桶传入到新建的桶中，如果当前桶有值就会在位置形成链表。）

get 方法：

- 首先也是根据 key 计算出 hashcode，然后定位到具体的桶中。
- 判断该位置是否为链表。
- 不是链表就根据 key、key 的 hashcode 是否相等来返回值。
- 为链表则需要遍历直到 key 及 hashcode 相等时候就返回值。
- 啥都没取到就直接返回 null 。

**HashMap 1.8的原理：**

当 Hash 冲突严重时，在桶上形成的链表会变的越来越长，这样在查询时的效率就会越来越低；时间复杂度为 O(N)，因此 1.8 中重点优化了这个查询效率。

TREEIFY_THRESHOLD 用于判断是否需要将链表转换为红黑树的阈值。

HashEntry 修改为 Node。

put 方法：

- 判断当前桶是否为空，空的就需要初始化（在resize方法 中会判断是否进行初始化）。
- 根据当前 key 的 hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突就直接在当前位置创建一个新桶即可。
- 如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 key、key 的 hashcode 与写入的 key 是否相等，相等就赋值给 e,在第 8 步的时候会统一进行赋值及返回。
- 如果当前桶为红黑树，那就要按照红黑树的方式写入数据。
- 如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。
- 接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
- 如果在遍历过程中找到 key 相同时直接退出遍历。
- 如果 e != null 就相当于存在相同的 key,那就需要将值覆盖。
- 最后判断是否需要进行扩容。

get 方法：

- 首先将 key hash 之后取得所定位的桶。
- 如果桶为空则直接返回 null 。
- 否则判断桶的第一个位置(有可能是链表、红黑树)的 key 是否为查询的 key，是就直接返回 value。
- 如果第一个不匹配，则判断它的下一个是红黑树还是链表。
- 红黑树就按照树的查找方式返回值。
- 不然就按照链表的方式遍历匹配返回值。

修改为红黑树之后查询效率直接提高到了 O(logn)。但是 HashMap 原有的问题也都存在，比如在并发场景下使用时容易出现死循环：

- 在 HashMap 扩容的时候会调用 resize() 方法，就是这里的并发操作容易在一个桶上形成环形链表；这样当获取一个不存在的 key 时，计算出的 index 正好是环形链表的下标就会出现死循环：在 1.7 中 hash 冲突采用的头插法形成的链表，在并发条件下会形成循环链表，一旦有查询落到了这个链表上，当获取不到值时就会死循环。

**ConcurrentHashMap 1.7原理：**

ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

put 方法:

首先是通过 key 定位到 Segment，之后在对应的 Segment 中进行具体的 put。

- 虽然 HashEntry 中的 value 是用 volatile 关键词修饰的，但是并不能保证并发的原子性，所以 put 操作时仍然需要加锁处理。

- 首先第一步的时候会尝试获取锁，如果获取失败肯定就有其他线程存在竞争，则利用 scanAndLockForPut() 自旋获取锁:

  尝试自旋获取锁。 如果重试的次数达到了 MAX_SCAN_RETRIES 则改为阻塞锁获取，保证能获取成功。

- 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。

- 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。

- 为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。

- 最后会使用unlock()解除当前 Segment 的锁。

get 方法：

- 只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。
- 由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。
- ConcurrentHashMap 的 get 方法是非常高效的，因为整个过程都不需要加锁。

**ConcurrentHashMap 1.8原理：**

1.7 已经解决了并发问题，并且能支持 N 个 Segment 这么多次数的并发，但依然存在 HashMap 在 1.7 版本中的问题：那就是查询遍历链表效率太低。和 1.8 HashMap 结构类似：其中抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。

CAS：

如果obj内的value和expect相等，就证明没有其他线程改变过这个变量，那么就更新它为update，如果这一步的CAS没有成功，那就采用自旋的方式继续进行CAS操作。

问题：

- 目前在JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
- 如果CAS不成功，则会原地自旋，如果长时间自旋会给CPU带来非常大的执行开销。

put 方法：

- 根据 key 计算出 hashcode 。
- 判断是否需要进行初始化。
- 如果当前 key 定位出的 Node为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
- 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。
- 如果都不满足，则利用 synchronized 锁写入数据。
- 最后，如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树。

get 方法：

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。

1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（O(logn)），甚至取消了 ReentrantLock 改为了 synchronized，这样可以看出在新版的 JDK 中对 synchronized 优化是很到位的。

#### 15. HashMap何时扩容？

当向容器添加元素的时候，会判断当前容器的元素个数，如果大于等于阈值---即大于当前数组的长度乘以加载因子的值的时候，就要自动扩容。

#### 16. 扩容的算法是什么？

扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组。

#### 17. Hashmap如何解决散列碰撞？

Java中HashMap是利用“拉链法”处理HashCode的碰撞问题。在调用HashMap的put方法或get方法时，都会首先调用hashcode方法，去查找相关的key，当有冲突时，再调用equals方法。hashMap基于hasing原理，我们通过put和get方法存取对象。当我们将键值对传递给put方法时，他调用键对象的hashCode()方法来计算hashCode，然后找到bucket（哈希桶）位置来存储对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。HashMap使用链表来解决碰撞问题，当碰撞发生了，对象将会存储在链表的下一个节点中。hashMap在每个链表节点存储键值对对象。当两个不同的键却有相同的hashCode时，他们会存储在同一个bucket位置的链表中。键对象的equals()来找到键值对。

#### 18. ArrayMap跟SparseArray在HashMap上面的改进？

HashMap要存储完这些数据将要不断的扩容，而且在此过程中也需要不断的做hash运算，这将对我们的内存空间造成很大消耗和浪费。

**SparseArray:**

SparseArray比HashMap更省内存，在某些条件下性能更好，主要是因为它避免了对key的自动装箱（int转为Integer类型），它内部则是通过两个数组来进行数据存储的，一个存储key，另外一个存储value，为了优化性能，它内部对数据还采取了压缩的方式来表示稀疏数组的数据，从而节约内存空间，我们从源码中可以看到key和value分别是用数组表示：

```java
private int[] mKeys;
private Object[] mValues;
```

同时，SparseArray在存储和读取数据时候，使用的是二分查找法。也就是在put添加数据的时候，会使用二分查找法和之前的key比较当前我们添加的元素的key的大小，然后按照从小到大的顺序排列好，所以，SparseArray存储的元素都是按元素的key值从小到大排列好的。 而在获取数据的时候，也是使用二分查找法判断元素的位置，所以，在获取数据的时候非常快，比HashMap快的多。

**ArrayMap**:

ArrayMap利用两个数组，mHashes用来保存每一个key的hash值，mArrray大小为mHashes的2倍，依次保存key和value。

```java
mHashes[index] = hash;
mArray[index<<1] = key;
mArray[(index<<1)+1] = value;
```

当插入时，根据key的hashcode()方法得到hash值，计算出在mArrays的index位置，然后利用二分查找找到对应的位置进行插入，当出现哈希冲突时，会在index的相邻位置插入。

**假设数据量都在千级以内的情况下：**

1、如果key的类型已经确定为int类型，那么使用SparseArray，因为它避免了自动装箱的过程，如果key为long类型，它还提供了一个LongSparseArray来确保key为long类型时的使用

2、如果key类型为其它的类型，则使用ArrayMap。



### 七、泛型

#### 1. 说一下对泛型的理解？

泛型的本质是**参数化类型，在不创建新的类型的情况下，通过泛型指定不同的类型来控制形参具体限制的类型**。也就是说在泛型的使用中，操作的数据类型被指定为一个参数，这种参数可以被用在类、接口和方法中，分别被称为泛型类、泛型接口和泛型方法。

> 当创建了带泛型声明的接口、父类之后，可以为该接口创建实现类，或者从该父类派生子类，需要注意：使用这些接口、父类派生子类时不能再包含类型形参，需要 传入具体的类型。

泛型是Java中的一种语法糖，能够在代码编写的时候起到类型检测的作用，但是虚拟机是不支持这些语法的。通过泛型使得在编译阶段完成一些类 型转换的工作，避免在运行时强制类型转换而出现 ClassCastException ，即类型转换异常。

泛型的优点：

1. 类型安全，避免类型的强转换。类型错误现在在编译期间就被捕获到了，而不是在运行时当作`java.lang.ClassCastException`展示出来，将类型检查从运行时挪到编译时有助于开发者更容易找到错误，并提高程序的可靠性。
2. 提高了代码的可读性，不必要等到运行的时候才去强制转换。

#### 2. 什么是类型擦除？什么时候擦除？

不管泛型的类型传入哪一种类型实参，对于Java来说，都会被当成同一类处理，在内存中也只占用一块空间。通俗一点来说，就是泛型只作用于代码编译阶段，在编译过程中，对于正确检验泛型结果后，会将泛型的信息擦除，也就是说，成功编译过后的class文件是不包含任何泛型信息的。

在**编译期。**

#### 3. 泛型是怎么解析的?

Java泛型的处理几乎都在编译器中进行，编译器生成的字节码是不包涵泛型信息的，泛型类型信息将在编译处理是被擦除，这个过程即类型擦除。

通常情况下，Java是通过以下方式处理泛型：Java编译器通过Code sharing方式为每个泛型类型创建唯一的字节码表示，并且将该泛型类型的实例都映射到这个唯一的字节码表示上。将多种泛型类形实例映射到唯一的字节码表示是通过类型擦除（type erasue）实现的。

> Code sharing：对每个泛型类只生成唯一的一份目标代码；该泛型类的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。

#### 4. 泛型有什么优点？

- 类型安全，避免类型的强转。

- 提高了代码的可读性，不必要等到运行的时候才去强制转换。



### 八、反射机制

#### 1. 动态代理和静态代理？

静态代理：代理类是在编译时就实现好的。也就是说 Java 编译完成后代理类是一 个实际的 class 文件。 

动态代理：代理类是在运行时生成的。也就是说 Java 编译完之后并没有实际的 class 文件，而是在运行时动态生成的类字节码，并加载到JVM中。

静态代理很简单，运用的就是代理模式：

![代理模式](https://user-gold-cdn.xitu.io/2020/4/24/171ab7a6cfb98cfb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



声明一个接口，再分别实现一个真实的主题类和代理主题类，通过让代理类持有真实主题类，从而控制用户对真实主题的访问。

动态代理指的是在运行时动态生成代理类，即代理类的字节码在运行时生成并载入当前的ClassLoader。

动态代理的原理是使用反射，思路和上面的一致。

使用动态代理的好处：

1. 不需要为`RealSubject`写一个形式完全一样的代理类。
2. 使用一些动态代理的方法可以在运行时制定代理类的逻辑，从而提升系统的灵活性。

动态代理涉及的主要类:

主要涉及两个类，这两个类都是`java.lang.reflect`包下的类，内部主要通过反射来实现的。

`java.lang.reflect.Proxy`：这是生成代理类的主类，通过 Proxy 类生成的代理类都继 承了 Proxy 类。 Proxy提供了用户创建动态代理类和代理对象的静态方法，它是所有动态代理类的 父类。

`java.lang.reflect.InvocationHandler`：这里称他为"调用处理器"，它是一个接口。 当调用动态代理类中的方法时，将会直接转接到执行自定义的InvocationHandler中的`invoke()`方法。即我们动态生成的代理类需要完成的具体内容需要自己定义一个 类，而这个类必须实现 InvocationHandler 接口，通过重写`invoke()`方法来执行具体内容。

```java
//创建一个InvocationHandler对象
InvocationHandler handler = new MyInvocationHandler(.args..);
//使用Proxy生成一个动态代理类
Class proxyClass = Proxy.getProxyClass(RealSubject.class.getClassLoader(),RealSubject.class.getInterfaces(), handler);
//获取proxyClass类中一个带InvocationHandler参数的构造器
Constructor constructor = proxyClass.getConstructor(InvocationHandler.class);
//调用constructor的newInstance方法来创建动态实例
RealSubject real = (RealSubject)constructor.newInstance(handler);
```

```java
//创建一个InvocationHandler对象
InvocationHandler handler = new MyInvocationHandler(.args..);
//使用Proxy直接生成一个动态代理对象
RealSubject real = Proxy.newProxyInstance(RealSubject.class.getClassLoader(),RealSubject.class.getInterfaces(), handler);
```

InvocationHandler 接口中有方法：

` invoke(Object proxy, Method method, Object[] args) `

这个函数是在代理对象调用任何一个方法时都会调用的，方法不同会导致第二个参数method不同，第一个参数是代理对象（表示哪个代理对象调用了method方法），第二个参数是 Method 对象（表示哪个方法被调用了），第三个参数是指定调用方法的参数。

#### 2. 反射是什么，在哪里用到，怎么利用反射创建一个对象？

Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为Java语言的反射机制。

**Java 反射机制的功能**

1.在运行时判断任意一个对象所属的类。 

2.在运行时构造任意一个类的对象。 

3.在运行时判断任意一个类所具有的成员变量和方法。 

4.在运行时调用任意一个对象的方法。

5.生成动态代理。 

**Java 反射机制的应用场景**

1.逆向代码 ，例如反编译 

2.与注解相结合的框架，例如Retrofit 

3.单纯的反射机制应用框架 例如EventBus 

4.动态生成类框架 例如Gson

Java中的反射首先是能够获取到 Java 中要反射类的字节码 ， 获取字节码有三种方法:

```java
//第一种方式 通过Class类的静态方法forName()来实现
class1 = Class.forName("com.xxx.Person");
//第二种方式 通过类的class属性
class1 = Person.class;
//第三种方式 通过对象getClass方法
Person person = new Person();
Class<?> class1 = person.getClass();
```

然后将字节码中的方法，变量，构造函数等映射成相应的 Method、Filed、Constructor 等类，这些类提供了丰富的方法可以被我们所使用。

**生成类的实例对象**

1.使用Class对象的`newInstance()`方法来创建该Class对象对应类的实例。这种方式 要求该Class对象的对应类有默认构造器，而执行`newInstance()`方法时实际上是利用默认构造器来创建该类的实例。 

2.先使用Class对象获取指定的Constructor对象，再调用Constructor对象的`newInstance()`方法来创建该Class对象对应类的实例。通过这种方式可以选择使用 指定的构造器来创建实例。

```java
//第一种方式 Class对象调用newInstance()方法生成
Object obj = class1.newInstance();
//第二种方式 对象获得对应的Constructor对象，再通过该Constructor对象的ne
wInstance()方法生成
Constructor<?> constructor = class1.getDeclaredConstructor(Strin
g.class);//获取指定声明构造函数
obj = constructor.newInstance("hello");
```

**调用类的方法**

1.通过Class对象的`getMethods()`方法或者`getMethod()`方法获得指定方法，返回 Method数组或对象。 

2.调用Method对象中的 `Object invoke(Object obj, Object... args)` 方法。 第一个参数对应调用该方法的实例对象，第二个参数对应该方法的参数。

```java
// 生成新的对象：用newInstance()方法
Object obj = class1.newInstance();
//首先需要获得与该方法对应的Method对象
Method method = class1.getDeclaredMethod("setAge", int.class);
//调用指定的函数并传递参数
method.invoke(obj, 28)
```

当通过Method的`invoke()`方法来调用对应的方法时，Java会要求程序必须有调用 该方法的权限。如果程序确实需要调用某个对象的private方法，则可以先调用 Method 对象的如下方法。

`setAccessible(boolean flag)`：将Method对象的acessible设置为指定的布尔值。 值为true，指示该Method在使用时应该取消Java语言的访问权限检查；值为 false，则知识该Method在使用时要实施Java语言的访问权限检查。

**访问成员变量值**

1.通过Class对象的`getFields()`方法或者`getField()`方法获得指定方法，返回Field数组或对象。

2.Field提供了两组方法来读取或设置成员变量的值： `getXXX(Object obj):`获取obj对象的该成员变量的值。此处的XXX对应8种基本类 型。如果该成员变量的类型是引用类型，则取消get后面的XXX。 `setXXX(Object obj,XXX val)`：将obj对象的该成员变量设置成val值。

```java
//生成新的对象：用newInstance()方法
Object obj = class1.newInstance();
//获取age成员变量
Field field = class1.getField("age");
//将obj对象的age的值设置为10
field.setInt(obj, 10);
//获取obj对象的age的值
field.getInt(obj);
```



#### 3. 代理模式与装饰模式的区别，手写一个静态代理，一个动态代理？

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

#### 4. 反射可以反射final修饰的字段吗？

- 当final修饰的成员变量在定义的时候就初始化了值，那么java反射机制就已经不能动态修改它的值了。
- 当final修饰的成员变量在定义的时候并没有初始化值的话，那么就还能通过java反射机制来动态修改它的值。

#### 5. 静态代理和动态代理什么场景使用？

- 静态代理使用场景：四大组件同AIDL与AMS进行跨进程通信
- 动态代理使用场景：Retrofit使用了动态代理极大地提升了扩展性和可维护性。




### 九、IO

#### 1. Java 中有几种类型的流 

字节流和字符流。

字节流继承于 InputStream 和 OutputStream，字符流继承于 Reader 和 Writer。

![img](https://camo.githubusercontent.com/fdb7d34b4c62c0dde7adca91c766e224ee3e11f3/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d333863336561343536326436646265332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

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

浅拷贝：是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

深拷贝：如果想要深拷贝一个对象，这个对象必须要实现 Cloneable 接口，实现 clone 方法，并且在 clone 方法内部，把该对象引用的其他对象也要 clone 一份，这就要求这个被引用的对象必须也要实现 Cloneable 接口并且实现 clone 方法。深拷贝会拷贝所有的属性，并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。

### 十一、异常处理

#### 1. Java 中异常分为哪些种类

![img](https://camo.githubusercontent.com/6f4720a573cb252a0e7aa0bd08364fa9752f67e6/687474703a2f2f696d616765732e636e6974626c6f672e636f6d2f626c6f672f3439373633342f3230313430322f3131313232383038353932363232302e6a7067)

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

#### 3. Error 和 Exception 的区别

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

#### 5. NoClassDefFoundError 和 ClassNotFoundException 有什么区别？

ClassNotFoundException的产生原因主要是： Java支持使用反射方式在运行时动态加载类，例如使用Class.forName方法来动态地加载类时，可以将类名作为参数传递给上述方法从而将指定类加载到JVM内存中，如果这个类在类路径中没有被找到，那么此时就会在运行时抛出ClassNotFoundException异常。 解决该问题需要确保所需的类连同它依赖的包存在于类路径中，常见问题在于类名书写错误。 另外还有一个导致ClassNotFoundException的原因就是：当一个类已经某个类加载器加载到内存中了，此时另一个类加载器又尝试着动态地从同一个包中加载这个类。通过控制动态类加载过程，可以避免上述情况发生。

NoClassDefFoundError产生的原因在于： 如果JVM或者ClassLoader实例尝试加载（可以通过正常的方法调用，也可能是使用new来创建新的对象）类的时候却找不到类的定义。要查找的类在编译的时候是存在的，运行的时候却找不到了。这个时候就会导致NoClassDefFoundError. 造成该问题的原因可能是打包过程漏掉了部分类，或者jar包出现损坏或者篡改。解决这个问题的办法是查找那些在开发期间存在于类路径下但在运行期间却不在类路径下的类

### 十二、类

#### 1. Java的匿名内部类有哪些限制？

匿名内部类的概念和用法：

- 匿名内部类的名字：没有人类认知意义上的名字
- 只能继承一个父类或实现一个接口
- 包名.OuterClassN，表示定位的第一个匿名内部类。外部类加N，N是匿名内部类的顺序。

语言规范以及语言的横向对比等：

匿名内部类的继承结构：Java中的匿名内部类不可以继承，只有内部类才可以有实现继承、实现接口的特性。而Kotlin是的匿名内部类是支持继承的，如

```java
val runnableFoo = object: Foo(),Runnable { 
        override fun run() { 
        
        } 
}
```

内存泄漏的切入点：

匿名内部类的构造方法：

- 匿名内部类会默认持有外部类的引用，可能会导致内存泄漏。
- 由编译器生成的。

其参数列表包括

- 外部对象（定义在非静态域内）
- 父类的外部对象（父类非静态）
- 父类的构造方法参数（父类有构造方法且参数列表不为空）
- 外部捕获的变量（方法体内有引用外部final变量）

Lambda转换(SAM类型，仅支持单一接口类型）：

如果CallBack是一个interface，不是抽象类，则可以转换为Lambda表达式。

```
CallBack callBack = () -> { 
        ... 
};
```

总结：

- 没有人类认知意义上的名字。
- 只能继承一个父类或实现一个接口。
- 父类是非静态的类型，则需父类外部实例来初始化。
- 如果定义在非静态作用域内，会引用外部类实例。
- 只能捕获外部作用域内的final变量。
- 创建时只有单一方法的接口可以用Lambda转换。

#### 2. 为什么Java里的匿名内部类只能访问final修饰的外部变量？

匿名内部类用法：

```java
public class TryUsingAnonymousClass {
    public void useMyInterface() {
        final Integer number = 123;
        System.out.println(number);

        MyInterface myInterface = new MyInterface() {
            @Override
            public void doSomething() {
                System.out.println(number);
            }
        };
        myInterface.doSomething();

        System.out.println(number);
    }
}
```

译后的结果

```java
class TryUsingAnonymousClass$1
        implements MyInterface {
    private final TryUsingAnonymousClass this$0;
    private final Integer paramInteger;

    TryUsingAnonymousClass$1(TryUsingAnonymousClass this$0, Integer paramInteger) {
        this.this$0 = this$0;
        this.paramInteger = paramInteger;
    }

    public void doSomething() {
        System.out.println(this.paramInteger);
    }
}
```

因为匿名内部类最终会编译成一个单独的类，而被该类使用的变量会以构造函数参数的形式传递给该类，例如：Integer paramInteger，**如果变量不定义成final的，paramInteger在匿名内部类被可以被修改，进而造成和外部的paramInteger不一致的问题**，为了避免这种不一致的情况，因次Java规定匿名内部类只能访问final修饰的外部变量。

#### 3. 什么是内部类？内部类的作用。

内部类可以有多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。

在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。

创建内部类对象并不依赖于外围类对象的创建。

内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。

内部类提供了更好的封装，除了该外围类，其他类都不能访问。

#### 4. 抽象类和接口区别？

**共同点**

- 是上层的抽象层。
- 都不能被实例化。
- 都能包含抽象的方法，这些抽象的方法用于描述类具备的功能，但是不提供具体的实现。

**区别**

1. 抽象类和接口要实例化，抽象类变量必须指向实现所有抽象方法的子类对象，接口变量必须指向实现所有接口方法的类对象。
2. 抽象类要被子类继承，接口要被类实现。
3. 接口里定义的变量只能是公共的静态的常量，抽象类中的变量是普通变量。
4. 抽象类里可以没有抽象方法。
5. 接口可以被类多实现（被其他接口多继承），抽象类只能被单继承。
6. 接口中没有 `this` 指针，没有构造函数，不能拥有实例字段（实例变量）或实例方法。
7. 抽象类不能在Java 8 的 lambda 表达式中使用。

要注意的是， 在 JDK V1.8 及之后的版本中，在 Interface 中增加了 defalut 方法，即接口默认方法。该新特性允许我们在接口中添加一个非抽象的方法实现，而这样做的方法只需要使用关键字default修饰该默认实现方法即可。一个简单的示例如下所示：

```java
public interface Formula {
    double calculate(int a);
    default double sqrt(int a){
        return Math.sqrt(a);
    }
}
```

该特性又叫扩展方法。通过该特性，我们将能够很方便的实现接口默认实现类。

#### 5. 接口的意义？

规范、扩展、回调。

#### 6. 静态方法，静态对象能不能继承?

结论：java中静态属性和静态方法可以被继承，但是不可以被重写而是被隐藏。

原因：

1)  静态方法和属性是属于类的，调用的时候直接通过类名.方法名完成，不需要继承机制即可以调用。如果子类里面定义了静态方法和属性，那么这时候父类的静态方法或属性称之为"隐藏"。如果你想要调用父类的静态方法和属性，直接通过父类名.方法或变量名完成，至于是否继承一说，子类是有继承静态方法和属性，但是跟实例方法和属性不太一样，存在"隐藏"的这种情况。

2)  多态之所以能够实现依赖于继承、接口和重写、重载（继承和重写最为关键）。有了继承和重写就可以实现父类的引用指向不同子类的对象。重写的功能是："重写"后子类的优先级要高于父类的优先级，但是“隐藏”是没有这个优先级之分的。

3) 静态属性、静态方法和非静态的属性都可以被继承和隐藏而不能被重写，因此不能实现多态，不能实现父类的引用可以指向不同子类的对象。非静态方法可以被继承和重写，因此可以实现多态。

#### 7. 抽象类的意义?

为其子类提供一个公共的类型，封装子类中的重复内容，定义抽象方法，子类虽然有不同的实现 但是定义是一致的。

#### 8. 静态内部类、非静态内部类的理解？

静态内部类：只是为了降低包的深度，方便类的使用，静态内部类适用于包含在类当中，但又不依赖与外在的类，不用使用外在类的非静态属性和方法，只是为了方便管理类结构而定义。在创建静态内部类的时候，不需要外部类对象的引用。

非静态内部类：持有外部类的引用，可以自由使用外部类的所有变量和方法。

#### 9. 静态内部类的设计意图

静态内部类与非静态内部类之间存在一个最大的区别：非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。

没有这个引用就意味着：

它的创建是不需要依赖于外围类的。 它不能使用任何外围类的非static成员变量和方法。




### 十三、其他

#### 1. 如何封装一个字符串转数字的工具类？

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

#### 3. Java里的幂等性了解吗？

幂等性原本是数学上的一个概念，即：f(x) = f(f(x))，对同一个系统，使用同样的条件，一次请求和重复的多次请求对系统资源的影响是一致的。

幂等性最为常见的应用就是电商的客户付款，试想一下如果你在付款的时候因为网络等各种问题失败了，然后去重复的付了一次，是一种多么糟糕的体验。幂等性就是为了解决这样的问题。

实现幂等性可以使用Token机制。

核心思想是为每一次操作生成一个唯一性的凭证，也就是token。一个token在操作的每一个阶段只有一次执行权，一旦执行成功则保存执行结果。对重复的请求，返回同一个结果。

例如：电商平台上的订单id就是最适合的token。当用户下单时，会经历多个环节，比如生成订单，减库存，减优惠券等等。每一个环节执行时都先检测一下该订单id是否已经执行过这一步骤，对未执行的请求，执行操作并缓存结果，而对已经执行过的id，则直接返回之前的执行结果，不做任何操 作。这样可以在最大程度上避免操作的重复执行问题，缓存起来的执行结果也能用于事务的控制等。

#### 4. java为什么跨平台？

因为Java程序编译之后的代码不是能被硬件系统直接运行的代码，而是一种“中间码”——字节码。然后不同的硬件平台上安装有不同的Java虚拟机(JVM)，由JVM来把字节码再“翻译”成所对应的硬件平台能够执行的代码。因此对于Java编程者来说，不需要考虑硬件平台是什么。所以Java可以跨平台。

#### 5. final，finally，finalize的区别？

final 可以用来修饰类、方法、变量，分别有不同的意义，final 修饰的 class 代表不可以继承扩展，final 的变量是不可以修改的，而 final 的方法也是不可以重写的（override）。

finally 则是 Java 保证重点代码一定要被执行的一种机制。我们可以使用 try-finally 或者 try-catch-finally 来进行类似关闭 JDBC 连接、保证 unlock 锁等动作。

finalize 是基础类 java.lang.Object 的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated。Java 平台目前在逐步使用 java.lang.ref.Cleaner 来替换掉原有的 finalize 实现。Cleaner 的实现利用了幻象引用（PhantomReference），这是一种常见的所谓 post-mortem 清理机制。利用幻象引用和引用队列，我们可以保证对象被彻底销毁前做一些类似资源回收的工作，比如关闭文件描述符（操作系统有限的资源），它比 finalize 更加轻量、更加可靠。

#### 6. transient？

1）一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2）transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被 transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现 Serializable接口。

3）被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被 transient修饰，均不能被序列化。

> 在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是 Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接 口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所 要序列化的变量，这与是否被transient修饰无关。因此第二个例子输出的是变量 content初始化的内容，而不是null。

#### 7. Java中异常捕获机制中的finally语句是不是一定会被执行？

至少有两种情况下finally语句是不会被执行的： 

- try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执 行，这也说明了finally语句被执行的必要而非充分条件是：相应的try语句一定被执行到。

- 在try块中有 `System.exit(0)` 这样的语句，`System.exit(0)` 是终止 Java虚拟机JVM的，连JVM都停止了，所有都结束了，当然finally语句也不会被执 行到。

1）finally语句是在try的return语句执行之后，return 返回之前执行。

2）finally块中的return语句会覆盖try块中的return返回。

3）如果finally语句中没有return语句覆盖返回值，那么 原来的返回值可能因为finally里的修改而改变也可能不变。（值传递）

4）try块里的return语句在异常的情况下不会被执行， 这样具体返回哪个看情况。

5）当发生异常后，catch中的return执行情况与未发生异常时try中return的执行情况完全一样。

总结：finally块的语句在try或catch中的return语句执行之后返回之前执行且finally里的修改语句可能影响也可能不影响try或catch中 return已经确定的返回 值，若finally里也有return语句则覆盖try或catch中的return语句直接返回。


## 并发
### 一、线程

#### 1. 线程的状态有哪些？

线程状态流程图

![image](https://user-gold-cdn.xitu.io/2020/3/1/17095d9647fd213b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- `NEW`：创建状态，线程创建之后，但是还未启动。
- `RUNNABLE`：运行状态，处于运行状态的线程，但有可能处于等待状态，例如等待CPU、IO等。
- `WAITING`：等待状态，一般是调用了wait()、join()、LockSupport.spark()等方法。
- `TIMED_WAITING`：超时等待状态，也就是带时间的等待状态。一般是调用了wait(time)、join(time)、LockSupport.sparkNanos()、LockSupport.sparkUnit()等方法。
- `BLOCKED`：阻塞状态，**等待锁的释放**，例如调用了synchronized增加了锁。
- `TERMINATED`：终止状态，一般是线程完成任务后退出或者异常终止。

NEW、WAITING、TIMED_WAITING都比较好理解，重点说一说RUNNABLE运行态和BLOCKED阻塞态。

线程进入RUNNABLE运行态一般分为五种情况：

- 线程调用`sleep(time)`后结束了休眠时间
- 线程调用的阻塞IO已经返回，阻塞方法执行完毕
- 线程成功的获取了资源锁
- 线程正在等待某个通知，成功的获得了其他线程发出的通知
- 线程处于挂起状态，然后调用了resume()恢复方法，解除了挂起。

线程进入BLOCKED阻塞态一般也分为五种情况：

- 线程调用`sleep()`方法主动放弃占有的资源
- 线程调用了阻塞式IO的方法，在该方法返回前，该线程被阻塞。
- 线程视图获得一个资源锁，但是该资源锁正被其他线程锁持有。
- 线程正在等待某个通知
- 线程调度器调用`suspend()`方法将该线程挂起

和线程状态相关的一些方法。

- `sleep()`方法让当前正在执行的线程在指定时间内暂停执行，正在执行的线程可以通过Thread.currentThread()方法获取。
- `yield()`方法放弃线程持有的CPU资源，将其让给其他任务去占用CPU执行时间。但放弃的时间不确定，有可能刚刚放弃，马上又获得CPU时间片。
- `wait()`方法是当前执行代码的线程进行等待，将当前线程放入预执行队列，并在wait()所在的代码处停止执行，直到接到通知或者被中断为止。该方法可以使得调用该方法的线程释放共享资源的锁， 然后从运行状态退出，进入等待队列，直到再次被唤醒。该方法只能在同步代码块里调用，否则会抛出IllegalMonitorStateException异常。wait(long millis)方法等待某一段时间内是否有线程对锁进行唤醒，如果超过了这个时间则自动唤醒。
- `notify()`方法用来通知那些可能等待该对象的对象锁的其他线程，该方法可以随机唤醒等待队列中等同一共享资源的一个线程，并使该线程退出等待队列，进入可运行状态。
- `notifyAll()`方法可以使所有正在等待队列中等待同一共享资源的全部线程从等待状态退出，进入可运行状态，一般会是优先级高的线程先执行，但是根据虚拟机的实现不同，也有可能是随机执行。
- `join()`方法可以让调用它的线程正常执行完成后，再去执行该线程后面的代码，它具有让线程排队的作用。

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

停止一个线程可以用`Thread.stop()`方法，但最好不要用它。虽然它确实可以停止一个正在运行的线程，但是这个方法是不安全的，而且是已被废弃的方法。

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

#### 11. 多线程的使用场景？使用多线程就一定效率高吗？

有时候使用多线程并不是为了提高效率，而是使得CPU能同时处理多个事件。

- 为了不阻塞主线程,启动其他线程来做事情,比如APP中的耗时操作都不在UI线程中做。
- 实现更快的应用程序,即主线程专门监听用户请求,子线程用来处理用户请求,以获得大的吞吐量.感觉这种情况，多线程的效率未必高。这种情况下的多线程是为了不必等待，可以并行处理多条数据。比如JavaWeb的就是主线程专门监听用户的HTTP请求，然启动子线程去处理用户的HTTP请求。
- 某种虽然优先级很低的服务，但是却要不定时去做。比如Jvm的垃圾回收。
- 某种任务，虽然耗时，但是不消耗CPU的操作时间，开启个线程，效率会有显著提高。比如读取文件，然后处理。磁盘IO是个很耗费时间，但是不耗CPU计算的工作。所以可以一个线程读取数据，一个线程处理数据。肯定比一个线程读取数据，然后处理效率高。因为两个线程的时候充分利用了CPU等待磁盘IO的空闲时间。

#### 12. 怎么安全停止一个线程任务？原理是什么？线程池里有类似机制吗？

**终止线程**

1、使用volatile boolean变量退出标志，使线程正常退出，也就是当run方法完成后线程终止。（推荐）

2、使用`interrupt()`方法中断线程，但是线程不一定会终止。

3、使用stop方法强行终止线程。不安全主要是：`thread.stop()`调用之后，创建子线程的线程就会抛出ThreadDeatherror的错误，并且会释放子线程所持有的所有锁。

**终止线程池**

`ExecutorService`线程池就提供了`shutdown`和`shutdownNow`这样的生命周期方法来关闭线程池自身以及它拥有的所有线程。

1、`shutdown`关闭线程池

线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。

2、`shutdownNow`关闭线程池并中断任务

终止等待执行的线程，并返回它们的列表。试图停止所有正在执行的线程，试图终止的方法是调用`Thread.interrupt()`，但是大家知道，如果线程中没有`sleep` 、`wait`、`Condition`、`定时锁`等应用,` interrupt()`方法是无法中断当前的线程的。所以，`ShutdownNow()`并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。

#### 13. ThreadLocal的原理?

ThreadLocal是一个关于创建线程局部变量的类。使用场景如下所示：

- **实现单个线程单例以及单个线程上下文信息存储，比如交易id等。**
- **实现线程安全，非线程安全的对象使用ThreadLocal之后就会变得线程安全，因为每个线程都会有一个对应的实例。 承载一些线程相关的数据，避免在方法中来回传递参数。**

当需要使用多线程时，有个变量恰巧不需要共享，此时就不必使用synchronized这么麻烦的关键字来锁住，每个线程都相当于在堆内存中开辟一个空间，线程中带有对共享变量的缓冲区，通过缓冲区将堆内存中的共享变量进行读取和操作，ThreadLocal相当于线程内的内存，一个局部变量。每次可以对线程自身的数据读取和操作，并不需要通过缓冲区与主内存中的变量进行交互。并不会像synchronized那样修改主内存的数据，再将主内存的数据复制到线程内的工作内存。ThreadLocal可以让线程独占资源，存储于线程内部，避免线程堵塞造成CPU吞吐下降。

在每个Thread中包含一个ThreadLocalMap，ThreadLocalMap的key是ThreadLocal的对象，value是独享数据。

#### 14. 创建线程的三种方式的对比?

采用实现Runnable、Callable接口的方式创见多线程时，

优势是： 线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。

在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程 来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型， 较好地体现了面向对象的思想。 

劣势是： 编程稍微复杂，如果要访问当前线程，则必须使用`Thread.currentThread()`方法。 

使用继承Thread类的方式创建多线程时优势是： 编写简单，如果需要访问当前线程，则无需使用`Thread.currentThread()`方法，直接使用`this`即可获得当前线程。 

劣势是： 线程类已经继承了Thread类，所以不能再继承其他父类。

#### 15. 通过Callable和Future创建线程?

（1）创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。 

（2）创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该 FutureTask对象封装了该Callable对象的call()方法的返回值。 

（3）使用FutureTask对象作为Thread对象的target创建并启动新线程。 

（4）调用FutureTask对象的get()方法来获得子线程执行结束后的返回值，调用 get()方法会阻塞线程。

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

#### 5. 线程池原理？

从数据结构的角度来看，线程池主要使用了阻塞队列（BlockingQueue）和HashSet集合构成。 从任务提交的流程角度来看，对于使用线程池的外部来说，线程池的机制是这样的：

1、如果正在运行的线程数 < coreSize，马上创建核心线程执行该task，不排队等待；
2、如果正在运行的线程数 >= coreSize，把该task放入阻塞队列；
3、如果队列已满 && 正在运行的线程数 < maximumPoolSize，创建新的非核心线程执行该task；
4、如果队列已满 && 正在运行的线程数 >= maximumPoolSize，线程池调用handler的reject方法拒绝本次提交。

理解记忆：1-2-3-4对应（核心线程->阻塞队列->非核心线程->handler拒绝提交）。

线程池的线程复用：

这里就需要深入到源码`addWorker()`：它是创建新线程的关键，也是线程复用的关键入口。最终会执行到runWoker，它取任务有两个方式：

- `firstTask`：这是指定的第一个runnable可执行任务，它会在Woker这个工作线程中运行执行任务run。并且置空表示这个任务已经被执行。
- `getTask()`：这首先是一个死循环过程，工作线程循环直到能够取出Runnable对象或超时返回，这里的取的目标就是任务队列workQueue，对应刚才入队的操作，有入有出。

其实就是任务在并不只执行创建时指定的`firstTask`第一任务，还会从任务队列的中通过`getTask()`方法自己主动去取任务执行，而且是有/无时间限定的阻塞等待，保证线程的存活。

信号量：

semaphore 可用于进程间同步也可用于同一个进程间的线程同步。

可以用来保证两个或多个关键代码段不被并发调用。在进入一个关键代码段之前，线程必须获取一个信号量；一旦该关键代码段完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量。

#### 6. 线程池都有哪几种工作队列？

- ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。

- LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法`Executors.newFixedThreadPool()`和`Executors.newSingleThreadExecutor()`使用了这个队列。

- SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法`Executors.newCachedThreadPool()`使用了这个队列。

- PriorityBlockingQueue：一个具有优先级的无限阻塞队列。

#### 7. 怎么理解无界队列和有界队列？

有界队列

1.初始的poolSize < corePoolSize，提交的runnable任务，会直接做为new一个Thread的参数，立马执行 。 

2.当提交的任务数超过了corePoolSize，会将当前的runable提交到一个block queue中。 

3.有界队列满了之后，如果poolSize < maximumPoolsize时，会尝试new 一个Thread的进行救急处理，立马执行对应的runnable任务。 

4.如果3中也无法处理了，就会走到第四步执行reject操作。

无界队列

与有界队列相比，除非系统资源耗尽，否则无界的任务队列不存在任务入队失败的情况。当有新的任务到来，系统的线程数小于corePoolSize时，则新建线程执行任务。当达到corePoolSize后，就不会继续增加，若后续仍有新的任务加入，而没有空闲的线程资源，则任务直接进入队列等待。若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存。 当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略。




### 三、锁

#### 1. 死锁触发的四大条件？

   **概念**：多个并发进程因争夺系统资源而产生相互等待的现象。

   **原理**： 当一组进程中的每个进程都在等待某个事件发生，而只有这组进程中的其他进程才能触发该事件，这就称这组进程发生了死锁。

- 互斥锁：某种资源一次只允许一个进程访问，即该资源一旦分配给某个进程，其他进程就不能再访问，直到该进程访问结束。
- 占有与保持：一个进程本身占有资源（一种或多种），同时还有资源未得到满足，正在等待其他进程释放该资源。
- 不可剥夺：别人已经占有了某项资源，你不能因为自己也需要该资源，就去把别人的资源抢过来。
- 循环的请求与等待：存在一个进程链，使得每个进程都占有下一个进程所需的至少一种资源。

当以上四个条件均满足，必然会造成死锁，发生死锁的进程无法进行下去，它们所持有的资源也无法释放。这样会导致CPU的吞吐量下降。所以死锁情况是会浪费系统资源和影响计算机的使用性能的。

在JAVA编程中，有3种典型的死锁类型： 静态的锁顺序死锁，动态的锁顺序死锁，协作对象之间发生的死锁。

**静态的锁顺序死锁**：a和b两个方法都需要获得A锁和B锁。一个线程执行a方法且已经获得了A锁，在等待B锁；另一个线程执行了b方法且已经获得了B锁，在等待A锁。这种状态，就是 发生了静态的锁顺序死锁。

解决方法：所有需要多个锁的线程，都要以相同的顺序来获得锁。

**动态的锁顺序死锁**：动态的锁顺序死锁是指两个线程调用同一个方法时，传入的参数颠倒造成的死锁。

解决方法：使用`System.identifyHashCode`来定义锁的顺 序。确保所有的线程都以相同的顺序获得锁。

**协作对象之间发生的死锁**：在持有锁的情况下调用了外部的方法。

解决方法：需要使用开放调用，即避免在持有锁的情况下调用外部的方法。

#### 2. synchronized关键字的使用？放在不同位置区别？

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

两者虽然实现细节不同，但本质上都是对一个对象的监视器（monitor）的获取。任意一个对象都拥有自己的监视器，当同步代码块或同步方法时，执行方法的线程必须先获得该对象的监视器才能进入同步块或同步方法，没有获取到监视器的线程将会被阻塞，并进入同步队列，状态变为 BLOCKED 。当成功获取监视器的线程释放了锁后，会唤醒阻塞在同步队列的线程，使其重新尝试对监视器的获取。

![img](https://camo.githubusercontent.com/57e44b9c59d9441a163e7d4b53d241fb589b3d1e/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d633338383132643866343538313064632e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

在 Java 6 之前，Monitor 的实现完全是依靠**操作系统内部的互斥锁**，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。

现代的（Oracle）JDK 中，JVM 对此进行了大刀阔斧地改进，提供了三种不同的 Monitor 实现，也就是常说的三种不同的锁：**偏斜锁**（Biased Locking）、**轻量级锁**和**重量级锁**，大大改进了其性能。

所谓锁的升级、降级，就是 JVM 优化 synchronized 运行的机制，当 JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。

当没有竞争出现时，默认会使用偏斜锁。JVM 会利用 CAS 操作，在对象头上的 Mark Word 部分设置线程 ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。

如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM 就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖 CAS 操作 Mark Word 来试图获取锁，如果重试成功，就使用普通的轻量级锁；否则，进一步升级为重量级锁（可能会先进行自旋锁升级，如果失败再尝试重量级锁升级）。

当 JVM 进入安全点（SafePoint）的时候，会检查是否有闲置的 Monitor，然后试图进行降级。

#### 4. synchronized和Lock的区别？

主要区别：

1. `synchronized`是Java中的关键字，是Java的内置实现；`Lock`是Java中的接口。
2. `synchronized`遇到异常会释放锁；`Lock`需要在发生异常的时候调用成员方法`Lock#unlock()`方法。
3. `synchronized`是不可以中断的，`Lock`可中断。
4. `synchronized`不能去尝试获得锁，没有获得锁就会被阻塞； `Lock`可以去尝试获得锁，如果未获得可以尝试处理其他逻辑。
5. `synchronized`多线程效率不如`Lock`，不过Java在1.6以后已经对`synchronized`进行大量的优化，所以性能上来讲，其实差不了多少。

| 类别     | synchronized                                                 | Lock（底层实现主要是Volatile + CAS）                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | Java的关键字，在jvm层面上                                    | 是一个类                                                     |
| 锁的释放 | 1、已获取锁的线程执行完同步代码，释放锁2、线程执行发生异常，jvm会让线程释放锁。 | 在finally中必须释放锁，不然容易造成线程死锁。                |
| 锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待。 | 分情况而定，Lock有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待 |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可重入 不可中断 非公平                                       | 可重入 可判断 可公平（两者皆可）                             |
| 性能     | 少量同步                                                     | 大量同步                                                     |

Lock（ReentrantLock）的底层实现主要是 Volatile + CAS（乐观锁），而Synchronized是一种悲观锁，比较耗性能。但是在JDK1.6以后对Synchronized的锁机制进行了优化，加入了偏向锁、轻量级锁、自旋锁、重量级锁，在并发量不大的情况下，性能可能优于Lock机制。所以建议一般请求并发量不大的情况下使用synchronized关键字。

#### 5. 悲观锁和乐观锁的举例？相关实现？使用场景?

悲观锁和乐观锁的概念：

- 悲观锁：悲观锁会认为，修改共享数据的时候其他线程也会修改数据，因此只在不会受到其他线程干扰的情况下执行。这样会导致其他有需要锁的线程挂起，等到持有锁的线程释放锁
- 乐观锁：每次不加锁，每次直接修改共享数据假设其他线程不会修改，如果发生冲突就直接重试，直到成功为止

举例：

- `悲观锁`：典型的悲观锁是独占锁，有`synchronized`、`ReentrantLock`。
- `乐观锁`：典型的乐观锁是CAS，实现CAS的`atomic`为代表的一系列类

使用场景

乐观锁适用于写比较少的情况下（多读场景），而一般多写的场景下用悲观锁就比较合适。

#### 6. CAS是什么？底层原理？

`CAS`全称`Compare And Set`，核心的三个元素是：`内存位置`、`预期原值`和`新值`，执行CAS的时候，会将内存位置的值与预期原值进行比较，如果一致，就将原值更新为新值，否则就不更新。 底层原理：是借助CPU底层指令`cmpxchg`实现原子操作。

**Unsafe**

Unsafe是CAS的核心类。因为Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类`Unsafe`，它提供了硬件级别的原子操作。

**CAS**

CAS即比较并交换，设计并发算法时常用到的一种技术，java.util.concurrent包全完建立在CAS之上，没有CAS也就没有此包，可见CAS的重要性。当前的处理器基本都支持CAS，只不过不同的厂家的实现不一样罢了。并且CAS也是通过Unsafe实现的，由于CAS都是硬件级别的操作，因此效率会比普通加锁高一些。

**CAS的缺点**

CAS操作无法涵盖并发下的所有场景，并且CAS从语义上来说也不是完美的，存在这样一个逻辑漏洞：如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？如果在这段期间它的值曾经被改成了B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个漏洞称为CAS操作的"ABA"问题。java.util.concurrent包为了解决这个问题，提供了一个带有标记的原子引用类`AtomicStampedReference`，它可以通过控制变量值的版本来保证CAS的正确性。不过目前来说这个类比较"鸡肋"，大部分情况下ABA问题并不会影响程序并发的正确性，如果需要解决ABA问题，使用传统的互斥同步可能回避原子类更加高效。

#### 7. Synchronized是公平锁还是非公平锁，ReteranLock是公平锁吗？

Synchronized是非公平锁；ReteranLock是非公平锁还是公平锁和设置有关。

#### 8.可见性，原子性，有序性，Synchronized可以保证什么？

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

#### 11. 访问 synchronized 修饰 static 方法、synchronized (类.class) 是否会冲突受干扰？

**Synchronized修饰非静态方法**，实际上是对调用该方法的对象加锁，俗称“对象锁”。也就是锁住的是这个对象，即this。如果同一个对象在两个线程分别访问对象的两个同步方法，就会产生互斥，这就是对象锁，一个对象一次只能进入一个操作。

**Synchronized修饰静态方法**，实际上是对该类对象加锁，俗称“类锁”。也就是锁住的是这个类，即xx.class。如果一个对象在两个线程中分别调用一个静态同步方法和一个非静态同步方法，由于静态方法会收到类锁限制，但是非静态方法会收到对象限制，所以两个方法并不是同一个对象锁，因此不会排斥。

#### 12. 同一个类中的 2 个方法都加了同步锁，多个线程能同时访问同一个类中的这两个方法吗？

Synchronized修饰2个非静态方法/2个静态方法，不可以。

Synchronized修饰1个非静态方法和1个静态方法，可以。

#### 13. 什么情况下导致线程死锁，遇到线程死锁该怎么解决？

死锁是指多个线程因竞争资源而造成的一种僵局（互相等待），若无外力作用，这些进程都将无法向前推进。

死锁产生的必要条件： 

- 互斥条件：线程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个线程所占有。此时若有其他线程请求该资源，则请求线程只能等待。 

- 不剥夺条件：线程所获得的资源在未使用完毕之前，不能被其他线程强行夺走，即只能由获得该资源的线程自己来释放（只能是主动释放)。 

- 请求和保持条件：线程已经保持了至少一个资源，但又提出了新的资源请求，而该资源已被其他线程占有， 此时请求进程被阻塞，但对自己已获得的资源保持不放。

- 循环等待条件：存在一种线程资源的循环等待链，链中每一个线程已获得的资源同时被链中下一个线程所请求。

在有些情况下死锁是可以避免的。两种用于避免死锁的技术：

1）加锁顺序（线程按照一定的顺序加锁）

2）加锁时限（线程尝试获取锁的时候加上一定的时限，超过时限则放弃对该锁的请求，并释放自己占有的锁）

#### 14. Synchronized优化后的锁机制简单介绍一下，包括自旋锁、偏向锁、轻量级锁、重量级锁？

自旋锁：

线程自旋说白了就是让cpu在做无用功，比如：可以执行几次for循环，可以执行几条空的汇编指令，目的是占着CPU不放，等待获取锁的机会。如果旋的时间过长会影响整体性能，时间过短又达不到延迟阻塞的目的。

偏向锁

偏向锁就是一旦线程第一次获得了监视对象，之后让监视对象“偏向”这个线程，之后的多次调用则可以避免CAS操作，说白了就是置个变量，如果发现为true则无需再走各种加锁/解锁流程。

轻量级锁：

轻量级锁是由偏向锁升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁竞争用的时候，偏向锁就会升级为轻量级锁；

重量级锁

重量锁在JVM中又叫对象监视器（Monitor），它很像C中的Mutex，除了具备Mutex(0|1)互斥的功能，它还负责实现了Semaphore(信号量)的功能，也就是说它至少包含一个竞争锁的队列，和一个信号阻塞队列（wait队列），前者负责做互斥，后一个用于做线程同步。

#### 15. 谈谈对Synchronized关键字涉及到的类锁，方法锁，重入锁的理解？

synchronized修饰静态方法获取的是类锁(类的字节码文件对象)。

synchronized修饰普通方法或代码块获取的是对象锁。这种机制确保了同一时刻对于每一个类实例，其所有声明为 synchronized 的成员函数中至多只有一个处于可执行状态，从而有效避免了类成员变量的访问冲突。

它俩是不冲突的，也就是说：获取了类锁的线程和获取了对象锁的线程是不冲突的！

```java
public class Widget {

    // 锁住了
    public synchronized void doSomething() {
        ...
    }
}

public class LoggingWidget extends Widget {

    // 锁住了
    public synchronized void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }
}
```

因为锁的持有者是“线程”，而不是“调用”。

线程A已经是有了LoggingWidget实例对象的锁了，当再需要的时候可以继续**开锁**进去的！

这就是内置锁的可重入性。



### 四、线程间通信

#### 1. notify和notifyAll方法的区别？

`notify`随机唤醒一个线程，`notifyAll`唤醒所有等待的线程，让他们竞争锁。

当一个拥有Object锁的线程调用`wait()`方法时，就会使当前线程加入`Object.wait` 等待队列中，并且释放当前占用的Object锁，这样其他线程就有机会获取这个Object锁，获得Object锁的线程调用`notify()`方法，就能在`Object.wait` 等待队列中随机唤醒一个线程（该唤醒是随机的与加入的顺序无关，优先级高的被唤醒概率会高）

如果调用`notifyAll()`方法就唤醒全部的线程。**注意:调用`notify()`方法后并不会立即释放Object锁，会等待该线程执行完毕后释放Object锁。**

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

#### 5. wait、sleep的区别和notify运行过程？

wait、sleep的区别

最大的不同是在等待时 wait 会释放锁，而 sleep 一直持有锁。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。

- 首先，要记住这个差别，**sleep是Thread类的方法,wait是Object类中定义的方法**。尽管这两个方法都会影响线程的执行行为，但是本质上是有区别的。
- `Thread.sleep`不会导致锁行为的改变，如果当前线程是拥有锁的，那么Thread.sleep不会让线程释放锁。
- `Thread.sleep`和`Object.wait`都会暂停当前的线程，对于CPU资源来说，不管是哪种方式暂停的线程，都表示它暂时不再需要CPU的执行时间。OS会将执行时间分配给其它线程。区别是，调用wait后，需要别的线程执行`notify/notifyAll`才能够重新获得CPU执行时间。
- 线程的状态参考 `Thread.State`的定义。新创建的但是没有执行（还没有调用start())的线程处于“就绪”，或者说`Thread.State.NEW`状态。
- Thread.State.BLOCKED（阻塞）表示线程正在获取锁时，因为锁不能获取到而被迫暂停执行下面的指令，一直等到这个锁被别的线程释放。BLOCKED状态下线程，OS调度机制需要决定下一个能够获取锁的线程是哪个，这种情况下，就是产生锁的争用，无论如何这都是很耗时的操作。

notify运行过程

当线程A（消费者）调用wait()方法后，线程A让出锁，自己进入等待状态，同时加入锁对象的等待队列。 线程B（生产者）获取锁后，调用notify方法通知锁对象的等待队列，使得线程A从等待队列进入阻塞队列。 线程A进入阻塞队列后，直至线程B释放锁后，线程A竞争得到锁继续从wait()方法后执行。



### 五、volatile 

#### 1. volatile 的作用和原理

`volatile`关键字就是Java中提供的一种解决可见性有序性问题的方案。对于原子性：对volatile变量的单次读/写操作可保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作。

`volatile`也是互斥同步的一种实现，不过它非常的轻量级。

**可见性** 如果对声明了`volatile`的变量进行写操作的时候，JVM会向处理器发送一条`Lock`前缀的指令，将这个变量所在缓存行的数据写入到系统内存。多处理器的环境下，其他处理器的缓存还是旧的，为了保证各个处理器一致，会通过嗅探在总线上传播的数据来检测自己的数据是否过期，如果过期，会强制重新将系统内存的数据读取到处理器缓存。

**有序性** `Lock`前缀的指令相当于一个内存栅栏，它确保指令排序的时候，不会把后面的指令拍到内存栅栏的前面，也不会把前面的指令排到内存栅栏的后面。

#### 2. 一个 int 变量用 volatile 修饰，多线程去操作 i++，是否线程安全？如何保证 i++ 线程安全？AtomicInteger 的底层实现原理？

不安全。CAS 使用 AtomicInteger 可以使 i++ 线程安全

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

- **加volatile是为了禁止指令重排**。指令重排指的是在程序运行过程中，并不是完全按照代码顺序执行的，会考虑到性能等原因，将不影响结果的指令顺序有可能进行调换。所以初始化的顺序本来是这三步：

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

**Java7 ConcurrentHashMap**

ConcurrentHashMap作为一种线程安全且高效的哈希表的解决方案，尤其其中的"分段锁"的方案，相比HashTable的表锁在性能上的提升非常之大。HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

concurrencyLevel：并行级别、并发数、Segment 数。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。其中的每个 Segment 很像 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

初始化槽: ensureSegment

ConcurrentHashMap 初始化的时候会初始化第一个槽 segment[0]，对于其他槽来说，在插入第一个值的时候进行初始化。对于并发操作使用 CAS 进行控制。

**Java8 ConcurrentHashMap**

抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。结构上和 Java8 的 HashMap（数组+链表+红黑树） 基本上一样，不过它要保证线程安全性，所以在源码上确实要复杂一些。1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（O(logn)），甚至取消了 ReentrantLock 改为了 synchronized，这样可以看出在新版的 JDK 中对 synchronized 优化是很到位的。

#### 3. CopyOnWriteArrayList的了解

对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉。

原理：

CopyOnWriteArrayList这是一个ArrayList的线程安全的变体，CopyOnWriteArrayList 底层实现添加的原理是先copy出一个容器(可以简称副本)，再往新的容器里添加这个新的数据，最后把新的容器的引用地址赋值给了之前那个旧的的容器地址，但是在添加这个数据的期间，其他线程如果要去读取数据，仍然是读取到旧的容器里的数据。

优点和缺点:

优点:

- 数据一致性完整，因为加锁了，并发数据不会乱。

- 解决了像ArrayList、Vector这种集合多线程遍历迭代问题，记住，Vector虽然线程安全，只不过是加了synchronized关键字，迭代问题完全没有解决！

缺点:

- 内存占有问题：很明显，两个数组同时驻扎在内存中，如果实际应用中，数据比较多，而且比较大的情况下，占用内存会比较大，针对这个其实可以用ConcurrentHashMap来代替。

- 数据一致性：CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

使用场景：

- 读多写少（白名单，黑名单，商品类目的访问和更新场景），因为写的时候会复制新集合。

- 集合不大，因为写的时候会复制新集合。

- 实时性要求不高，因为有可能会读取到旧的集合数据。



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

#### 1. 源码中有哪里用到了AtomicIntger

#### 2. AQS了解吗？



## JVM
### 一、Java内存模型

#### 1. Jvm内存区域是如何划分的？

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfaf97ca63?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

内存区域划分：

- `程序计数器`：当前线程的字节码执行位置的指示器，线程私有。
- `Java虚拟机栈`：描述的Java方法执行的内存模型，每个方法在执行的同时会创建一个栈帧，存储着局部变量、操作数栈、动态链接和方法出口等，线程私有。
- `本地方法栈`：本地方法执行的内存模型，线程私有。
- `Java堆`：所有对象实例分配的区域。
- `方法区`：所有已经被虚拟机加载的**类的信息**、**常量**、**静态变量**和**即时编辑器编译后的代码数据**。

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

stack：由系统自动分配。例如，声明在函数中一个局部变量 int b; 系统自动在栈中为 b 开辟空间 

heap：需要程序员自己申请，并指明大小，在 c 中 malloc 函数，对于 Java 需要手动 new Object()的形式开辟

**申请后系统的响应** 

stack：只要栈的剩余空间大于所申请空间，系统将为程序提供内存，否则将报异常提示栈溢出。 

heap：首先应该知道操作系统有一个记录空闲内存地址的链表，当系统收到程序的申请时， 会遍历该链表，寻找第一个空间大于所申请空间的堆结点，然后将该结点从空闲结点链表中删除，并将该结点的空间分配给程序。另外，由于找到的堆结点的大小不一定正好等于申请的大小，系统会自动的将多余的那部分重新放入空闲链表中。

**申请大小的限制** 

stack：栈是向低地址扩展的数据结构，是一块连续的内存的区域。这句话的意思是栈顶的地址和栈的最大容量是系统预先规定好的，如果申请的空间超过栈的剩余空间时，将提示 overflow。因此，能从栈获得的空间较小。

heap：堆是向高地址扩展的数据结构，是不连续的内存区域。这是由于系统是用链表来存储的空闲内存地址的， 自然是不连续的，而链表的遍历方向是由低地址向高地址。堆的大小受限于计算机系统中有效的虚拟内存。由此可见， 堆获得的空间比较灵活，也比较大。

**申请效率的比较**

stack：由系统自动分配，速度较快。但程序员是无法控制的。 

heap：由 new 分配的内存，一般速度比较慢，而且容易产生内存碎片,不过用起来最方便。

**heap 和 stack 中的存储内容**

stack： 在函数调用时，第一个进栈的是主函数中后的下一条指令（函数调用语句的下一条可执行语句）的地址， 然后是函数的各个参数，在大多数的 C 编译器中，参数是由右往左入栈的，然后是函数中的局部变量。注意静态变量是不入栈的。 

当本次函数调用结束后，局部变量先出栈，然后是参数，最后栈顶指针指向最开始存的地址，也就是主函数中的 下一条指令，程序由该点继续运行。 

heap：一般是在堆的头部用一个字节存放堆的大小。堆中的具体内容有程序员安排。

**数据结构层面的区别**

还有就是数据结构方面的堆和栈，这些都是不同的概念。这里的堆实际上指的就是（满足堆性质的）优先队列的 一种数据结构，第 1 个元素有最高的优先权；

栈实际上就是满足先进后出的性质的数学或数据结构。 虽然堆栈，堆栈的说法是连起来叫，但是他们还是有很大区别的，连着叫只是由于历史的原因。

#### 5. 堆内存，栈内存理解，栈如何转换成堆？

- 在函数中定义的一些基本类型的变量和对象的引用变量都是在函数的栈内存中分配。

- 堆内存用于存放由new创建的对象和数组。JVM里的“堆”（heap）特指用于存放Java对象的内存区域。所以根据这个定义，Java对象全部都在堆上。JVM的堆被同一个JVM实例中的所有Java线程共享。它通常由某种自动内存管理机制所管理，这种机制通常叫做“垃圾回收”（garbage collection，GC）。

- 堆主要用来存放对象的，栈主要是用来执行程序的。

- 实际上，栈中的变量指向堆内存中的变量，这就是 Java 中的指针。

####  6. Java 中的引用类型分类和区别，虚引用的使用场景？

强引用：当内存不足，JVM开始垃圾回收，对于强引用的对象，就算是出现了OOM也不会对该对象进行回收。

软引用：是一种相对强引用弱化了一些的引用，需要用`java.lang.ref.SoftReference`类来实现，对于只有软引用的对象来说，当系统内存充足时它不会被回收，当系统内存不足时它会被回收。

弱引用：需要用`java.lang.ref.WeakReference`类来实现，它比软引用的生存期更短，对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。

虚引用：需要`java.lang.ref.PhantomReference`类来实现。顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，虚引用必须和引用队列 ReferenceQueue 联合使用。

虚引用的主要作用是跟踪对象被垃圾回收的状态。仅仅是提供了一种确保对象被finalize以后，做某些事情的机制。

#### 7. Java中对象的生命周期

在Java中，对象的生命周期包括以下几个阶段：

1.创建阶段(Created)

JVM 加载类的class文件，此时所有的static变量和static代码块将被执行加载完成后，对局部变量进行赋值（先父后子的顺序），再执行new方法，调用构造函数 一旦对象被创建，并被分派给某些变量赋值，这个对象的状态就切换到了应用阶段。

2.应用阶段(In Use)

对象至少被一个强引用持有着。

3.不可见阶段(Invisible)

当一个对象处于不可见阶段时，说明程序本身不再持有该对象的任何强引用，虽然该这些引用仍然是存在着的。 简单说就是程序的执行已经超出了该对象的作用域了。

4.不可达阶段(Unreachable)

对象处于不可达阶段是指该对象不再被任何强引用所持有。 与“不可见阶段”相比，“不可见阶段”是指程序不再持有该对象的任何强引用，这种情况下，该对象仍可能被JVM等系统下的某些已装载的静态变量或线程或JNI等强引用持有着，这些特殊的强引用被称为”GC root”。存在着这些GC root会导致对象的内存泄露情况，无法被回收。

5.收集阶段(Collected)

当垃圾回收器发现该对象已经处于“不可达阶段”并且垃圾回收器已经对该对象的内存空间重新分配做好准备时，则对象进入了“收集阶段”。如果该对象已经重写了finalize()方法，则会去执行该方法的终端操作。

6.终结阶段(Finalized)

当对象执行完finalize()方法后仍然处于不可达状态时，则该对象进入终结阶段。在该阶段是等待垃圾回收器对该对象空间进行回收。

7.对象空间重分配阶段(De-allocated)

垃圾回收器对该对象的所占用的内存空间进行回收或者再分配了，则该对象彻底消失了，称之为“对象空间重新分配阶段。



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

字节码文件就是以.CLASS文件结尾的文件，是通过JAVAC命令编译过生成的。因为JAVA不是编译型语言，所以它需要去解释字节码文件才能够运行。

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

4）反射（Class.forName("com.xxx.test")） 

5）初始化一个类的子类（会首先初始化子类的父类） 

6）JVM 启动时标明的启动类，即文件名和类名相同的那个类

类的初始化步骤：

1）如果这个类还没有被加载和链接，那先进行加载和链接 

2）假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一 次），那就初始化直接的父类（不适用于接口） 

3）加入类中存在初始化语句（如 static 变量和 static 块），那就依次执行这些初始化语句。

#### 11. 类的加载过程，Person person = new Person();为例进行说明。

1) 因为new用到了Person.class，所以会先找到Person.class文件，并加载到内存中;

2) 执行该类中的static代码块，如果有的话，给Person.class类进行初始化;

3) 在堆内存中开辟空间分配内存地址;

4) 在堆内存中建立对象的特有属性，并进行默认初始化;

5) 对属性进行显示初始化;

6) 对对象进行构造代码块初始化;

7) 对对象进行与之对应的构造函数进行初始化;

8) 将内存地址付给栈内存中的p变量。

#### 12. JAVA常量池

Interger中的 `-128~127`

a.当数值范围为-128~127时：如果两个new出来的Integer对象，即使值相同，通过“==”比较结果为false，但两个对直接赋值，则通过“==”比较结果为“true，这一点与String非常相似。

b.当数值不在-128~127时，无论通过哪种方式，即使两对象的值相等，通过“==”比较，其结果为false；

c.当一个Integer对象直接与一个int基本数据类型通过“==”比较，其结果与第一点相同；

d.Integer对象的hash值为数值本身；

为什么是`-128-127`?

在Integer类中有一个静态内部类IntegerCache，在IntegrCache类中有一个Integer数组，用以缓存当前数值范围为-128~127时的Integer对象。






### 三、垃圾回收

#### 1. 如何判断对象可回收？

判断一个对象可以回收通常采用的算法是引用计数法和可达性算法。由于互相引用导致的计数不好判断，Java采用的可达性算法。

> 给对象添加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用 失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能被再使用的。主流的JVM里面没有选用引用计数算法来管理内存，其中最主要的原因是它很难解决对象间的互循环引用的问题。

可达性算法的思路是：通过一些列被成为GC Roots的对象作为起始点，自上往下从这些起点往下搜索，搜索所有走过的路径称为引用链，如果一个对象没有跟任何引用链相关联的时候，则证明该对象不可用，所以这些对象就会被判定为可以回收。

可以被当作GC Roots的对象包括：

- Java虚拟机栈中的引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法中JNI引用的对象

总结就是，方法运行时，方法中引用的对象；类的静态变量引用的对象；类中常量 引用的对象；Native方法中引用的对象

Java 垃圾收集的原理：

- 自动垃圾收集的前提是清楚哪些内存可以被释放，主要有两个方面，最主要部分就是**对象实例**，存储在堆上的；另一个是**方法区中的元数据**等信息，例如类型不再使用，卸载该 Java 类比较合理；
- **对象实例收集**主要是两种基本算法，引用计数和可达性分析，Java 选择的可达性分析。JVM 会把虚拟机栈和本地方法栈中正在引用的对象、静态属性引用的对象和常量，作为 GC Roots。

#### 2. GC的常用算法？

- `标记 - 清除`：首先标记出需要回收的对象，标记完成后统一回收所有被标记的对象。容易产生碎片空间。
- `复制算法`：它将可用的内存分为两块，每次只用其中的一块，当需要内存回收的时候，将存活的对象复制到另一块内存，然后将当前已经使用的内存一次性回收掉。需要浪费一半的内存。
- `标记 - 整理`：让存活的对象向一端移动，之后清除边界外的内存。
- `分代搜集`：根据对象存活的周期，Java堆会被分为新生代和老年代，根据不同年代的特性，选择合适的GC收集算法。

JVM中分为年轻代和老年代。 

HotSpot JVM把**年轻代**分为了三部分：1个Eden区和2个Survivor区（分别叫from和 to）。默认比例为8：1。

一般情况下，新创建的对象都会被分配到Eden区(一些大对象特殊处理)，这些对象经过第一次Minor GC后，如果仍然存活，将会被移到Survivor区。对象在Survivor 区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到年老代中。 因为年轻代中的对象基本都是朝生夕死的(80%以上)，所以 在**年轻代的垃圾回收算法使用的是复制算法**，复制算法的基本思想就是将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面。复制算法不会产生内存碎片。 

在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor 区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

#### 3. Minar GC和Full GC的区别？

- `Minar GC`：频率高、针对新生代。
- `Full GC`：频率低、发生在老年代、通常会伴随一次Minar GC和速度慢。

#### 4. 说一下四种引用以及他们的区别？

- `强引用`：强引用还在，垃圾搜集器就不会回收被引用的对象。
- `软引用`：对于软引用关联的对象，在系统发生内存溢出异常之前，将会把这些对象列进回收范围进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。
- `弱引用`：被若引用关联的对象只能存活到下一次GC之前。
- `虚引用`：为对象设置虚引用的目的仅仅是为了GC之前收到一个系统通知。

![img](https://user-gold-cdn.xitu.io/2019/3/11/1696c45ababaed39?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 5. 圾回收的GCRoot是什么？

JVM 会把**虚拟机栈和本地方法栈中正在引用的对象**、**方法区中静态属性引用的对象和常量**，作为 GC Roots。

