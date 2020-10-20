- 可以使用c++实现本地方法，必须将实现本地方法的函数声明为extenrn“C”
- 将一个方法链接到Java程序中：
  - 在Java类中声明一个本地方法
  - 运行javah以获得包含该方法的C声明的头文件
  - 用C实现该本地方法
  - 将代码置于共享类库中
  - 在Java程序中加载该类库

![处理本地代码](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E5%A4%84%E7%90%86%E6%9C%AC%E5%9C%B0%E4%BB%A3%E7%A0%81.png?raw=true)

- Java数据类型和C数据类型的对应关系

![Java数据类型和C数据类型的对应关系](https://github.com/zhangzhian/LearningNotes/blob/master/res/Java%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%92%8CC%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png?raw=true)

- 本地代码调用Java，实例方法
  - 获取隐式参数的类
  - 获取方法ID
  - 进行调用
- Java编程语言的所有数组类型都有对应的C语言类型

![Java和C数组类型之间的对应关系](https://github.com/zhangzhian/LearningNotes/blob/master/res/Java%E5%92%8CC%E6%95%B0%E7%BB%84%E7%B1%BB%E5%9E%8B%E4%B9%8B%E9%97%B4%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.png?raw=true)

- C语言抛出异常，需要调用Throw 或ThrownNew函数来创建一个新的异常对象，当本地方法退出时，Java虚拟机抛出该异常