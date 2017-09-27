#  [学习笔记] Java核心技术 卷一：基础知识 对象与类、继承（一）



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