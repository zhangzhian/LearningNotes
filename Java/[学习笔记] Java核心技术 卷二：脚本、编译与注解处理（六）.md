- 脚本引擎是一个可以执行用某种特定语言编写的脚本的类库。构造一个ScriptEngineManager，调用getEngineFactories
- 脚本引擎工厂的属性
![脚本引擎工厂的属性](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E8%84%9A%E6%9C%AC%E5%BC%95%E6%93%8E%E5%B7%A5%E5%8E%82%E7%9A%84%E5%B1%9E%E6%80%A7.png?raw=true)
- 可以通过调用脚本上下文的setReader和setWriter方法来重定向脚本的标准输入和输出
- 脚本引擎实现Invocable接口，可以提供调用脚本语言函数的功能，需要函数名来调用invokeFunction方法，或者通过实现接口的方式
-  某些脚本引擎出于对执行效率的考虑，可以将脚本代码编译为某种中间格式，这些引擎实现了Compilable接口
- 调用Java编译器，简易方法通过ToolProvider.getSystemJavaCompiler 获取编译器，run方法进行编译，还可以通过CompilationTask对象来对编译过程进行更对的控制
  - 控制程序代码的来源
  - 控制类文件的放置位置  
  - 监听在编译过程中产生的错误和警告信息
  - 在后台运行编译器
- 注解是那些插入到源代码中使用其他工具可以对其进行处理的标签
- 每个注解都必须通过一个注解接口进行定义，这些接口中的方法与注解中的元素相对应
- 注解是有注解接口定义的

```java
modifiers @interface AnnotationName
{
	elementDecaration1
	elementDecaration2
	....
}
```
- 每个元素的声明都具有`type elementName()`形式或者`type elementName() default value`
- 所有注解接口都隐式地扩展字java.lang.annotation.Annotation接口。这个接口是一个常规接口。无法扩展注解接口。
- 注解元素类型：
  - 基本类型
  - String
  - Class
  - enum类型
  - 注解类型
  - 由前面所述类型组成的数组
- 每个注解都有以下形式

```java
@AnnotationName(elementName1=value1,elementName2=value2)
```

- 标准注解
![标准注解1](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E6%A0%87%E5%87%86%E6%B3%A8%E8%A7%A3.png?raw=true)
![标准注解2](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E6%A0%87%E5%87%86%E6%B3%A8%E8%A7%A3-%E7%BB%AD.png?raw=true)
 
- @Target元注解可以应用于一个注解，以限制该注解可以应用到哪些项上

![注解的元素类型](https://github.com/zhangzhian/LearningNotes/blob/master/res/@Target%20%E6%B3%A8%E8%A7%A3%E7%9A%84%E5%85%83%E7%B4%A0%E7%B1%BB%E5%9E%8B.png?raw=true)
- @Retention元注解用于指定一条注解应该保留多长时间，默认为.CLASS

![@Retention注解的保留策略](https://github.com/zhangzhian/LearningNotes/blob/master/res/@Retention%E6%B3%A8%E8%A7%A3%E7%9A%84%E4%BF%9D%E7%95%99%E7%AD%96%E7%95%A5.png?raw=true)

- 注释处理器通常通过扩展AbstractiProcessor类而实现Processor接口
- 注释处理器只能产生新的源文件，不能修改已有的源文件