[学习笔记] Java核心技术 卷二：输入与输出（二）

- 抽象类InputStream和OutputStream构成了I/O类层次结构的基础
- read和write方法在执行时都将阻塞，直至字节确实被读入或写出
- 输入输出流的层次结构

![输入输出流的层次结构](https://github.com/zhangzhian/LearningNotes/blob/master/res/%E8%BE%93%E5%85%A5%E8%BE%93%E5%87%BA%E6%B5%81%E7%9A%84%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.png?raw=true)

- Reader和Writer的层次结构


![Reader和Writer的层次结构](https://github.com/zhangzhian/LearningNotes/blob/master/res/Reader%E5%92%8CWriter%E7%9A%84%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.png?raw=true)

- Closeable、Flushable、Readable和Appendable接口

![Closeable、Flushable、Readable和Appendable接口](https://github.com/zhangzhian/LearningNotes/blob/master/res/Closeable%E3%80%81Flushable%E3%80%81Readable%E5%92%8CAppendable%E6%8E%A5%E5%8F%A3.png?raw=true)

- FileInputStream和FileOutputStream可以提供附着在一个磁盘上的输入流和输出流，需要向构造器提供文件名或文件的完整路径名。
- OutputStreamWriter类使用选定的字符编码方式，把Unicode码元转化为字节流
- InputStreamReader类将包含字节的输入流转化为可以产生Unicode码元的读入器
- DataInoutStream实现了DataInput接口，读入二进制数据
- DataOutputStream实现DataOutput接口，写二进制数据
- RandomAccessFile类可以在文件中的任何位置查找或写入数据
- ZipInputStream从给定的InputStream向其中填充数据
- ZipOutStream写出到制定的OutputStream的ZipOutStream
- ObjectOutputStream 将对象写出到指定的OutputStream
- ObjectInputStram 从制定的InputString中读取对象信息
- 调用FileChannel类的lock或tryLock方法可以给文件加锁
- 文件加锁机制是依赖操作系统的
