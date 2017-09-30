#[学习笔记] Java核心技术 卷一：基础知识 Java 的基本程序设计结构（一）

---

 - 根据 Java 语言规范， main 方法必须声明为 public
 - Java 提供了 4 种整型
| 类型        | 存储需求   |  取值范围  |
| :--------:   | :------:  | :----:  |
| int     | 4字节 |   －2 147 483 648 ～ 2 147 483 647（超过20亿）     |
| short        |  2字节   |   －32 768 ～ 32 767   |
| long       |    8字节    |  －9 223 372 036 854 775 808 ～ 9 223 372 036 854 775 807  |
| byte       |    1字节    |  －128 ～ 127  |

 
 - 长整型数值有一个后缀L。十六进制数值有一个前缀0x。八进制有一个前缀0
 - Java7开始，加上前缀 0b 就可以写二进制数, 还可以为数字字面量加下划线,只是为了让人更易读
 - Java 没有任何无符号类型（ unsigned ）
 - 浮点类型

| 类型        | 存储需求   |  取值范围  |
| :--------:   | :------:  | :----:  |
| float     | 4字节 |   大约 ±3.402 823 47E ＋ 38F（有效位数为 6 ～ 7 位）     |
| double        |  8字节   |   大约 ±3.402 823 47E ＋ 38F（有效位数为 6 ～ 7 位）   |


 - float 类型的数值有一个后缀 F。 没有后缀 F 的浮点数值默认为double 类型
 - 量 Double.POSITIVE_INFINITY、 Double.NEGATIVE_INFINITY 和 Double.NaN
（与相应的 Float 类型的常量一样） 分别表示这三个特殊的值

    	if (x == Double.NaN) //永远为假
		if (Double.isNaN(x)) //检测x是否不是一个数字

 - 转义序列符 \u 表示 Unicode 代码单元的编码
    
    | 转义序列        | 名称   |  Unicode 值  |
| :--------:   | :------:  | :----:  |
| \b     | 退格 |  \u0008     |
| \t        |  制表   |   \u0009   |
| \n     | 换行 |  \u000a     |
| \r        |  回车   |   \u000d   |
| \"     | 双引号 |  \u0022     |
| \'        |  单引号   |   \u0027   |
| \\     | 反斜杠 |  \u005c     |

 - $ 是一个合法的 Java 字符， 但不要在自己的代码中使用这个字符。它只用在 Java 编译器或其他工具生成的名字中。
 - 位运算符包括：&（ " 与 "）、 |（ " 或 "）、 ^（ " 异或 "）、 ~（ " 非 "）

 - “>>” 和“<<”运算符将二进制位进行右移或左移操作。>>> 运算符将用 0 填充高位； >> 运算符用符号位填充高位。 没有 <<< 运算符。
 
 - 运算符优先级
![运算符优先级](http://img.blog.csdn.net/20170926114336237?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
 - 用于 printf 的转换符
 
![用于printf的转换符](http://img.blog.csdn.net/20170926144336226?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 日期和时间的转换符

![日期和时间的转换符1](http://img.blog.csdn.net/20170926145041674?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![日期和时间的转换符2](http://img.blog.csdn.net/20170926145051424?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 - case 标签可以是：类型为 char、 byte、 short 或 int（或其包装器类 Character、 Byte、 Short 和 Intege），从 Java SE 7 开始， case 标签还可以是字符串字面量。

 - BigInteger 和 BigDecimal。 这两个类可以处理包含任意长度数字序列的数值。
BigInteger 类实现了任意精度的整数运算， BigDecimal 实现了任意精度的浮点数运算

 - 列表项

