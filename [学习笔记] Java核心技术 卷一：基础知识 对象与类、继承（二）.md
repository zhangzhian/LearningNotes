#  [学习笔记] Java核心技术 卷一：基础知识 对象与类、继承（二）



---
## 对象与类
 - 表达类关系的 UML 符号

 ![表达类关系的UML符号](http://img.blog.csdn.net/20170927100459855?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
 -  Java 中方法参数的使用情况：
 -  一个方法不能修改一个基本数据类型的参数（即数值型或布尔型）
 -  一个方法可以改变一个对象参数的状态
 -  一个方法不能让对象参数引用一个新的对象
 -  初始化块（initializationblock)。在一个类的声明中，可以包含多个代码块。只要构造类的对象，这些块就会被执行

    ```
        // object initialization block
        {
            id = nextld;
            nextld++;
        }
    ```
 -  import 语句不仅可以导入类，还增加了导入静态方法和静态域的功能
## 继承

 - 一个对象变量可以指示多种实际类型的现象被称为多态（ polymorphism)。在运行时能够自动地选择调用哪个方法的现象称为动态绑定（dynamic binding。 )
 - 不允许扩展的类被称为 final 类。如果在定义类的时候使用了 final  修饰符就表明这个类是 final 类。 将方法或类声明为 final 主要目的是： 确保它们不会在子类中改变语义
 - 只能在继承层次内进行类型转换。在将超类转换成子类之前，应该使用 instanceof进行检查
 - 在 Java 中 ， 只有基本类型 （ primitive types) 不是对象， 例如，数值、 字符和布尔类型的值都不是对象。所有的数组类塱，不管是对象数组还是基本类型的数组都扩展了 Object 类
 - 在子类中定义 equals 方法时， 首先调用超类的 equals。 如果检测失败， 对象就不可能相等。 如果超类中的域都相等，就需要比较子类中的实例域
 - 散列码（ hash code ) 是由对象导出的一个整型值。 散列码是没有规律的。如果 x 和 y 是两个不同的对象， x.hashCode( ) 与 y.hashCode( ) 基本上不会相同
 - Equals 与 hashCode 的定义必须一致：如果 x.equals(y) 返回 true, 那么 x.hashCode( ) 就必
须与 y.hashCode( ) 具有相同的值

 -  trimToSize方法将存储区域的大小调整为当前元素数量所需要的存储空间数目。垃圾回收器将回收多余的存储空间。一旦整理了数组列表的大小，添加新元素就需要花时间再次移动存储块，所以应该在确认不会添加任何元素时， 再调用 trimToSize。

 - **难点：反射**