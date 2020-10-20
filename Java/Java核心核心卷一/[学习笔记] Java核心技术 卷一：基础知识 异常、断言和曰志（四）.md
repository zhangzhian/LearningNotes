# [学习笔记] Java核心技术 卷一：基础知识 异常、断言和曰志（四）


---

 - try 语句可以只有 finally 子句，而没有 catch 子句
 - 当 finally 子句包含 return 语句时，这个返回值将会覆盖 try 语句块中的返回值。
 - 带资源的 try 语句（try-with-resources) 的最简形式为：
    ```
    try (Resource res = . . .)
    {
        work with res
    }
    
    ```
    try块退出时 ，会自动调用 res.doseO。
    
 - 断言机制允许在测试期间向代码中插入一些检査语句。当代码发布时，这些插人的检测语句将会被自动地移走。Java 语言引人了关键字 assert。这两种形式都会对条件进行检测， 如果结果为 false, 则抛出一个 AssertionError 异常。在第二种形式中 ，表达式将被传人 AssertionError 的构造器， 并转换成一个消息字符串。
 -  assert 条件；
 -  assert 条件：表达式；

 -  在默认情况下， 断言被禁用。可以在运行程序时用 -enableassertions 或 -ea 选项启用 ：
`java -enableassertions MyApp`
 - 可以在某个类或整个包中使用断言， 例如：
`java -ea:MyClass -eaiconi.inycompany.inylib.. , MyApp`
 - 用选项 -disableassertions 或 -da 禁用某个特定类和包的断言：
`java -ea:... -da:MyClass MyApp`
 -  启用和禁用所有断言的 -ea 和 -da 开关不能应用到那些没有类加载器的“ 系统类”
上。对于这些系统类来说， 需要使用 `-enablesystemassertions/-esa` 开关启用断言。

 - 要生成简单的日志记录，可以使用全局日志记录器（global logger) 并调用其 info 方法：
`Logger.getClobal 0,info("File->Open menu item selected");`

 - 在适当的地方（如 main 开始）调用
`Logger.getClobal ().setLevel (Level.OFF);`

 - 通常， 有以下 7 个日志记录器级别：
• SEVERE
• WARNING
• INFO
• CONFIG
• FINE
• FINER
• FINEST

 - 典型的用法是：
 ```
 if (…）
    {
        IOException exception = new IOException(". . .")；
        1ogger.throwing("com•mycompany.mylib.Reader" , "read", exception) ;
        throw exception;
}
 ```
 
 还有
 ```
    try
    {… }
    catch (IOException e)
    {
        Logger.getLogger("com.mycompany.myapp") .log(Level .WARNING , "Reading image", e);
    }
    
    ```
调用 throwing 可以记录一条 FINER 级别的记录和一条以 THROW 开始的信息。

 - 1 ) 可以用下面的方法打印或记录任意变量的值：
`System.out.println("x=" + x);`
或
`Logger.getClobal0-info(nx=" + x);`
2 )在每一个类中放置一个单独的 main方法。可以对每一个类进行单元测试。
   ```
    public class MyClass
    {
        methods andfields
        public static void main(String[] args)
        {
            test code
        }
    }
   ```
  
    3 )  http://junit.org 査看 JUnit。
    JUiiit 是一个非常常见的单元测试框架，利用它可以很容易地组织测试用例套件。
    4 ) 日志代理（logging proxy) 是一个子类的对象， 它可以截获方法调用， 并进行日志记录， 然后调用超类中的方法。
    5 ) 利 用 Throwable 类提供的 printStackTmce 方法，可以从任何一个异常对象中获得堆栈情况。 
    在代码的任何位置插入下面这条语句就可以获得堆栈轨迹：`Thread.duapStack();`
    6 )—般来说 ， 堆栈轨迹显示在 System.err 上 。也可以利用 `printStackTrace(PrintWriter s)`方法将它发送到一个文件中。
    7 )可以调用静态的`Thread.setDefaultUncaughtExceptionHandler` 方法改变非捕获异常的处理器