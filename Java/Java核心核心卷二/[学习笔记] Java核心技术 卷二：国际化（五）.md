- Locale类 可以对格式化进行控制。
- locale由5个部分组成
  - 语言，由2，3个小些字母组成，en（英语）
  - 可选的一段脚本，由首字母大写的四个字母表示，Latn（拉丁文）
  - 可选的一个国家或地区，由2个大写字母或3个数字表示，US（美国）
  - 可选的一个变体，用于指定各种杂项特性，例如方言和拼写规则
  - 可选的一个扩展，扩展描述了日历和数字等内容的本地偏好

- Java提供了格式器对象的集合，对java.text包中的数字值进行格式化和解析
  - 得到Locale对象
  - 使用一个“工厂方法”得到一个格式器对象
  - 使用这个格式器对象来完成格式化和解析工作
- 工厂方法是NumberFormat类的静态方法，接受一个Locale类型的参数。getNumberInstance数字，getCurrencyInstance货币量，getPercentInstance百分比
- Currency类控制被格式器所处理的货币，可以将一个货币标识符传给静态的Currency.getInstance方法来得到一个Currency对象，然后对每一个格式器都调用setCurrency方法
- 格式化日期和时间，可以使用DataTimeFormatter，需要考虑4点：
  - 月份和星期应该用本地语言表示
  - 年月日的顺序要符合本地习惯
  - 公历可能不是本地首选的日期表示方法
  - 必须考虑本地时区
- 为了获得Locale敏感的比较烦，可以使用静态的Collator.getInstance 方法，Collator实现了Comparator接口，可以传递一个Collator对象给排序方法对一组字符串进行排序
- 字符间的差别可以被分为首要的primary（A和Z）、其次的secondary（A 和 Å）、再次的tertiary（A和a）
![不同强度下的排序](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E4%B8%8D%E5%90%8C%E5%BC%BA%E5%BA%A6%E4%B8%8B%E7%9A%84%E6%8E%92%E5%BA%8F.png?raw=true)
- Collator.INDNTICAL 不允许有任何差异，分解模式
![分解模式之间的差异](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E5%88%86%E8%A7%A3%E6%A8%A1%E5%BC%8F%E4%B9%8B%E9%97%B4%E7%9A%84%E5%B7%AE%E5%BC%82.png?raw=true)
- MessageFormat 类，支持Locale，并且会对数字和日期进行格式化，MessageFormat.format()  
- 占位符索引后面可以跟一个类型type和一个风格style，之间用逗号分隔
- 一个选择格式字符串由一个序列对构成的，每一个对包括：一个下限，一个格式字符串，下限和格式字符串由#分隔符，对与对之间由符号|分隔
- 行结束符%n
- 当本地化一个应用时，会产生很多资源包，每一个包都是一个属性文件或事一个描述了与locale相关的项的类，对于每一个包，都要为所有你想要支持的lacale提供相应的版本
- 一般来说，使用 包名_语言_国家 来命名所有和国家相关的资源，使用 包名_语言 来命名所有和语言相关的资源