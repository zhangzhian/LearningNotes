- 注册驱动器：
  - 在Java程序中加载驱动器类 `Class.forName("com.mysql.jdbc.Driver");`
  - 设置jdbc.drivers属性
- 连接到数据库：调用DriverManager.getConnection
- 执行SQL语句`Statement state=conn.createStatement(); ` ，增删改`state.executeUpdate(sql);`，查询`ResultSet rs = state.executeQuery(sql);`
- ResultSet接口，迭代器初始化时被设定在第一行之前的位置，必须调用next方法将其移动到第一行，没有hasNext方法，需要不断调用next，直到返回false
- SQL异常类型
![SQL异常类型](https://github.com/zhangzhian/LearningNotes/blob/master/res/SQL%E5%BC%82%E5%B8%B8%E7%B1%BB%E5%9E%8B.png?raw=true)
- 预备语句，每个宿主变量都用“ ？”表示，`PreparedStatement stat = conn.prepareStatement(sql)`，在执行预备语句前，使用set方法将变量绑定到实际的值上`stat.setString(pos, value)`,pos=1代表第一个位置
- 在SQL中，二进制大对象称为BLOB，字符型大对象称为CLOB，读取LOB，需要执行SELECT语句，然后在ResultSet上调用getBlob或getClob方法，可以获得Blob或clob类型对象，要从Blob中获取二进制数据，可以调用getBytes或getBinaryStream，要从Clob中获取数据，可以调用etSubString或getCharacterStream方法来获取其中的字符数据
- SQL转义应用于：
  - 日期和时间字面常量: {d '2019-01-01'},{s '00:00:00'},{ts '2019-01-01 00:00:00'}
  - 调用标量函数: 指仅返回单个值的函数 {fn ...}
  - 调用存储过程: 在数据库中执行的用数据库相关的语言编写的过程 {call ...}，用=捕获返回值
  - 外连接：{oj ...}
  - 在LIKE子句中的转义字符 _和%在LINK子句中具有特殊含义，匹配一个字符或一个字符序列 {escape '!'} 定义！为转义字符
- 多结果集
  - 使用execute方法执行SQL语句
  - 获取第一个结果集或更新计数
  - 重复调用getMoreResults方法以移动到下一个结果集
  - 当不存在更多结果集或更新计数时，完成操作
- 可滚动的结果集，需要得到一个不同的Statement对象：`Statement state=conn.createStatement(type,con);`,`Statement state=conn.prepareStatement(command,type,con);`
- ResultSet类的type值 

值     |  解释
-------- | -----
TYPE_FORWARD_ONLY  | 结构集不能滚动
TYPE_SCROLL_INSENSITIVE  | 结果集可以滚动，但对数据库变化不敏感
TYPE_SCROLL_SENSITIVE  | 结果可以滚动，且对数据库变化敏感

- Result类的Concurrency值

值     |  解释
-------- | -----
CONCUR_READ_ONLY  | 结构集不能滚动
CONCUR_UPDATABLE  | 结果集可以滚动，但对数据库变化不敏感
- previous 向后滚动， relative(n) n为正，前移，负数后移，absolute(n) 游标移到指定行，getRow返回当前行号
- 并非所有的查询都会返回可更新的结构集
- updateXXX方法 更新结果中当前行上的某个字段值 

- CachedRowSet 允许在断开连接的状态下执行相关操作，被缓存的行集
- WebRowSet对象代表了一个被缓冲的行集，该行集可以保存为XML文件
- FileredRowSet和JoinRowSet接口支持对行集的轻量级操作，等同与SQL中的SELECT喝JOIN操作
- JdbcRowSet是ResultSet接口的一个瘦包装器
- 在SQL中，描述数据库或其组成部分的数据称为元数据。我们可以获得三类元数据：关于数据库的元数据、关于结果集的元数据以及关于预备语句参数的元数据
- 将一组语句构成建成一个事务，所有语句都顺利被执行后，事务回被提交，否则，事务将被回滚
- 默认情况，数据库连接处于自动提交模式。关闭默认值`conn.setAutoCommit(false)`
- 在使用批量更新是，一个语句序列作为一批操作将同时被收集和提交。addBatch方法添加需要批量执行的语句，executeBatch方法将为所有已提交的语句返回一个记录数的数组
- SQL数据类型及其对应的Java类型
![SQL数据类型及其对应的Java类型](https://github.com/zhangzhian/LearningNotes/blob/master/res/SQL%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%8F%8A%E5%85%B6%E5%AF%B9%E5%BA%94%E7%9A%84Java%E7%B1%BB%E5%9E%8B.png?raw=true)