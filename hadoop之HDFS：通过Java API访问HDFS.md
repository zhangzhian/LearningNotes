# hadoop之HDFS：通过Java API访问HDFS

---
HDFS是一个分布式文件系统，可以通过Java API接口对HDFS进行操作，下面记录实现Java API的过程和出现的一些问题及解决方案
## 环境搭建
### 导入jar包
```
#common包中的jar文件导入
hadoop-2.8.1\share\hadoop\common\lib\*.jar
hadoop-2.8.1\share\hadoop\common\hadoop-common-2.8.1.jar

#客户端需要配置，里面有hdfs封装的client，不导入会报错
hadoop-2.8.1\share\hadoop\hdfs\hadoop-hdfs-client-2.8.1.jar
```
导入jar包，上面是需要导入的jar所在目录，关于common\lib\中jar，可能有部分jar包不用导入，查了很多资料没有说明，现在都导入。
### 配置 log4j日志
```
log4j.rootLogger=WARN, stdout, R
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
# Pattern to output the caller's file name and line number.
#log4j.appender.stdout.layout.ConversionPattern=%5p [%t] (%F:%L) - %m%n
# Print the date in ISO 8601 format
log4j.appender.stdout.layout.ConversionPattern=%d [%t] %-5p %c - %m%n
log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=example.log
log4j.appender.R.MaxFileSize=100KB
# Keep one backup file
log4j.appender.R.MaxBackupIndex=1
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n
# Print only messages of level WARN or above in the package com.foo.
log4j.logger.com.foo=WARN
```
放在src目录下

## 访问HDFS Java代码

```
package com.yodosmart.hadoop;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.io.UnsupportedEncodingException;
import java.io.Writer;
import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

/**
 * 利用HDFS java API操作文件
 * 
 */
public class HdfsDAO {
	// 修改成自己的HDFS主机地址
	private static final String HDFS = "hdfs://192.168.3.39:9000";
	private static final String utf8 = "UTF-8";
	private static final String gbk = "GBK";

	/**
	 * 两个构造器
	 * 
	 * @param conf
	 */
	public HdfsDAO(Configuration conf) {
		this(HDFS, conf);
	}

	public HdfsDAO(String hdfs, Configuration conf) {
		this.hdfsPath = hdfs;
		this.conf = conf;
	}

	private String hdfsPath;
	private Configuration conf;

	/**
	 * 测试方法入口
	 */
	public static void main(String[] args) throws IOException {
		Configuration conf = config();
		HdfsDAO hdfs = new HdfsDAO(conf);
		// hdfs.mkdirs("/dir1");
		// hdfs.rm("/123");
		// hdfs.rename("/dir", "/out");
		// hdfs.createFile("/test", "测试");
		// hdfs.copyFile("D:/agera.png", "/agera.png");
		// convert("D:/111.txt",gbk,"D:/123.txt",utf8);
		// hdfs.download("/agera.png", "D:/agera1.png");
		// hdfs.cat("/hello1");
		// hdfs.ls("/");
	}

	public static Configuration config() {
		Configuration conf = new Configuration();
		return conf;
	}

	/**
	 * 创建目录
	 * 
	 * @param folder
	 *            hdfs路径
	 * @throws IOException
	 */
	public void mkdirs(String folder) throws IOException {
		Path path = new Path(folder);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		if (!fs.exists(path)) {
			fs.mkdirs(path);
			System.out.println("Create: " + folder);
		}
		fs.close();
	}

	/**
	 * 删除文件或目录
	 * 
	 * @param folder
	 *            hdfs路径
	 * @throws IOException
	 */
	public void rm(String folder) throws IOException {
		Path path = new Path(folder);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		fs.deleteOnExit(path);
		System.out.println("Delete: " + folder);
		fs.close();
	}

	/**
	 * 重命名文件
	 * 
	 * @param src
	 *            hdfs路径原文件名
	 * @param dst
	 *            hdfs路径新文件名
	 * @throws IOException
	 */
	public void rename(String src, String dst) throws IOException {
		Path name1 = new Path(src);
		Path name2 = new Path(dst);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		fs.rename(name1, name2);
		System.out.println("Rename: from " + src + " to " + dst);
		fs.close();
	}

	/**
	 * 遍历文件 hdfs路径
	 * 
	 * @param folder
	 * @throws IOException
	 */
	public void ls(String folder) throws IOException {
		Path path = new Path(folder);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		FileStatus[] list = fs.listStatus(path);
		System.out.println("ls: " + folder);
		System.out
				.println("==========================================================");
		for (int i = 0; i < list.length; i++) {
			System.out.println("name:" + list[i].getPath() + ", folder: "
					+ list[i].isDir() + ", size:" + list[i].getLen() + "\n");
		}
		System.out
				.println("==========================================================");
		fs.close();
	}

	/**
	 * 创建文件
	 * 
	 * @param file
	 *            hdfs路径文件名
	 * @param content
	 *            文件内容
	 * @throws IOException
	 */
	public void createFile(String file, String content) throws IOException {
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		byte[] buff = content.getBytes();
		FSDataOutputStream os = null;
		try {
			os = fs.create(new Path(file));
			os.write(buff, 0, buff.length);
			System.out.println("Create: " + file);
		} finally {
			if (os != null)
				os.close();
		}
		fs.close();
	}

	/**
	 * 拷贝文件到HDFS
	 * 
	 * @param local
	 *            本地路径文件名
	 * @param remote
	 *            hdfs路径文件名
	 * @throws IOException
	 */
	public void copyFile(String local, String remote) throws IOException {
		// convert(local, gbk, local, utf8);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		FSDataOutputStream out = fs.create(new Path(remote));
		FileInputStream in = new FileInputStream(local);
		IOUtils.copyBytes(in, out, 2048, true);
		// fs.copyFromLocalFile(new Path(local), new Path(remote));
		System.out.println("copy from: " + local + " to " + remote);
		fs.close();
	}

	/**
	 * 转换编码
	 * 
	 * @param oldFile
	 *            原文件
	 * @param oldCharset
	 *            原文件编码
	 * @param newFlie
	 *            新文件
	 * @param newCharset
	 *            新文件编码
	 */
	public static void convert(String oldFile, String oldCharset,
			String newFlie, String newCharset) {
		BufferedReader bin;
		FileOutputStream fos;
		StringBuffer content = new StringBuffer();
		try {
			bin = new BufferedReader(new InputStreamReader(new FileInputStream(
					oldFile), oldCharset));
			String line = null;
			while ((line = bin.readLine()) != null) {
				content.append(line);
				content.append(System.getProperty("line.separator"));
			}
			bin.close();

			File dir = new File(newFlie.substring(0, newFlie.lastIndexOf("/")));
			if (!dir.exists()) {
				dir.mkdirs();
			}
			fos = new FileOutputStream(newFlie);
			Writer out = new OutputStreamWriter(fos, newCharset);
			out.write(content.toString());
			out.close();
			fos.close();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	/**
	 * 从HDFS中下载文件到本地中
	 * 
	 * @param remote
	 *            hdfs文件路径
	 * @param local
	 *            本地路径
	 * @throws IOException
	 */
	public void download(String remote, String local) throws IOException {
		Path path = new Path(remote);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		fs.copyToLocalFile(path, new Path(local));
		System.out.println("download: from" + remote + " to " + local);
		fs.close();
	}

	/**
	 * 查看文件中的内容
	 * 
	 * @param remoteFile
	 *            文件路径
	 * @return
	 * @throws IOException
	 */
	public String cat(String remoteFile) throws IOException {
		Path path = new Path(remoteFile);
		FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
		FSDataInputStream fsdis = null;
		System.out.println("cat: " + remoteFile);
		OutputStream baos = new ByteArrayOutputStream();
		String str = null;
		String trans = null;
		try {
			fsdis = fs.open(path);
			IOUtils.copyBytes(fsdis, baos, 4096, false);
			str = baos.toString();
			// 转码
			// trans = new String(str.getBytes(), 0, str.getBytes().length,
			// utf8);
		} finally {
			IOUtils.closeStream(fsdis);
			fs.close();
		}
		System.out.println(str);
		return str;
	}
}
```


## 常见错误

### java.net.MalformedURLException: unknown protocol: hdfs
默认的情况下，URL只支持http协议的，所以需要设定开启HDFS协议
```
//设定开启HDFS协议
        URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
```
### Unable to load native-hadoop library for your platform
win系统下调用的hadoop客户端，导致不能使用本地的hadoop类，需要导入hadoop-hdfs-client-2.8.1.jar,这样就可以解决问题了

### ConnectException: 拒绝连接

修改 core-site.xml文件和etc/hosts文件，修改后，stop-all.sh、start-all.sh重启生效 
1、修改core-site.xml文件，将fs.defaultFS 修改为ip地址就可以了(基于IP访问) 
2、将/etc/hosts中添加 DHCP获取或者自己设置的IP地址 到localhost主机名的映射（基于域名，主机名称访问）

參考 [hadoop之HDFS：CentOS安装和部署HDFS][1]中的Hadoop的伪分布式环境搭建部分

> **注意：**无论做哪一个步，都需要开启9000端口，或者关闭防火墙 

### HDFS客户端的权限错误：Permission denied
1、在hdfs的配置文件中，将dfs.permissions修改为False
2、执行这样的操作 hadoop fs -chmod 777 /user/hadoop
我是用了第一种hdfs-site.xml中添加：
```
 <property>
      <name>dfs.permissions</name>
      <value>false</value>
 </property>
```
### Hadoop bin directory does not exist和Did not find winutils.exe
Windows下eclipse开发hadoop程序会报错，原因是因为hadoop没有发布winutils.exe造成的，需要编译发布出来；在环境变量中配置 HADOOP_HOME。

1. 下载 hadoop-common-2.8.1-bin 地址：[hadoop-common-2.8.1-bin下载地址][2]
2. 用户变量新建：
![用户变量](http://img.blog.csdn.net/20170930164318721?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

    > 注意：图片是2.2.0版的，上面的资源是2.8.1，需要改变量值，我起      初用的2.2.0版的，后来直接把2.2.0的bin覆盖了，所以没改

3. 环境变量添加：

![环境变量](http://img.blog.csdn.net/20170930164541860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

参考资料：
1 http://blog.csdn.net/xiaoshunzi111/article/details/52062640
2 http://blog.csdn.net/yelllowcong/article/details/77409604
  


  [1]: https://github.com/zhangzhian/LearningNotes/blob/master/hadoop%E4%B9%8BHDFS%EF%BC%9ACentOS%E5%AE%89%E8%A3%85%E5%92%8C%E9%83%A8%E7%BD%B2HDFS.md
  [2]: https://github.com/zhangzhian/LearningNotes/tree/master/res