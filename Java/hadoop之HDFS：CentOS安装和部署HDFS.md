# hadoop之HDFS：CentOS安装和部署HDFS

---

## 一、准备工作

 1. [下载 jdk][2]  
 2. [下载 Hadoop][1]

    ![下载好的jdk和Hadoop](http://img.blog.csdn.net/20170930144643568?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 二、安装JDK、Hadoop及配置环境变量

#### 安装JDK

进入 /usr/lib/java-1.8.0，把压缩包jdk-8u144-linux-x64.tar.gz移动到该目录，并解压
```
 cd /usr/lib/java-1.8.0
 tar zxf ./jdk-8u144-linux-x64.tar.gz 
```

#### 安装Hadoop
进入 /opt/hadoop目录，把压缩包hadoop-2.8.1.tar.gz移动到该目录，并解压 
```
 cd /opt/hadoop
 tar zxf ./hadoop-2.8.1.tar.gz
```
#### 配置环境变量
配置环境，编辑 /etc/profile 文件
```
vim /etc/profile
```
在其后添加如下信息：
```
export JAVA_HOME=/usr/lib/java-1.8.0/jdk1.8.0_144
export HADOOP_HOME=/opt/hadoop/hadoop-2.8.1
PATH=$JAVA_HOME/bin:$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
export PATH JAVA_HOME CLASSPATHo

```
JAVA_HOME jdk的路径
HADOOP_HOME hadoop路径
使配置的变量生效：
```
source /etc/profile
```
## 三、SSH无密码登录
```
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```
验证ssh，`# ssh localhost` 
不需要输入密码即可登录。

## 四、Hadoop的伪分布式环境搭建
Hadoop 伪分布式模式是在一台机器上模拟Hadoop分布式，单机上的分布式并不是真正的分布式，而是使用线程模拟的分布式。分布式和伪分布式这两种配置也很相似，唯一不同的地方是伪分布式是在一台机器上配置，也就是名字节点（namenode）和数据节点（datanode）均是同一台机器。

需要配置的文件有core-site.xml和hdfs-site.xml这两个文件他们都位于${HADOOP_HOME}/etc/hadoop/文件夹下。 
#### core-site.xml：
```
<configuration>
 <property>
     <name>hadoop.tmp.dir</name>
      <value>file:/opt/hadoop/hadoop-2.8.1/tmp</value>
      <description>Abase for other temporary directories.</description>
    </property>
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://192.168.3.39:9000</value>
        <final>true</final>
 </property>
</configuration>

```
fs.defaultFS的value修改为本机的ip地址
#### hdfs-site.xml：
```
<configuration>
    <property>
      <name>dfs.replication</name>
      <value>1</value>
    </property>
    <property>
      <name>dfs.namenode.name.dir</name>
      <value>file:/opt/hadoop/hadoop-2.8.1/tmp/dfs/name</value>
   </property>
    <property>
      <name>dfs.datanode.data.dir</name>
      <value>file:/opt/hadoop/hadoop-2.8.1/tmp/dfs/data</value>
    </property>
    <property>
      <name>dfs.namenode.http-address</name>
      <value>master:50070</value>
      <name>dfs.permissions</name>
      <value>false</value>
    </property>
</configuration>
```
#### 修改/etc/hosts
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 yodosmart.hdfs.01
192.168.3.39   master
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6 yodosmart.hdfs.01
```
在/etc/hosts中添加DHCP获取或者自己设置的IP地址 到localhost主机名的映射.这种方式是基于主机名称的访问，对于ip访问无效
查看当前主机名`# hostname`

#### 开放9000端口
```
#开放9000端口
iptables -I INPUT -p tcp -m tcp --dport 9000 -m state --state NEW,ESTABLISHED -j ACCEPT

#重启防火墙
service iptables save 

#保存iptables
service iptables restart
```
如果需要访问web管理页面，还需要开发50070端口
#### 格式化hdfs
配置完成后，执行格式化命令，使HDFS的目录进行格式化：
```
    hdfs namenode -format
```
## 五、启动HDFS

启动HDFS的脚本位于Hadoop目录下的sbin文件夹中
移动到hdfs主目录
```
sbin/start-dfs.sh # 启动HDFS脚本
```
在执行start-dfs.sh脚本启动HDFS时，可能出现类似如下的报错内容：
```
  localhost: Error: JAVA_HOME is not set and could not be found.
```
JAVA_HOME没找到，这是因为在hadoop-env.sh脚本中有个`JAVA_HOME=${JAVA_HOME}`，所以只需将`${JAVA_HOME}`替换成JDK的路径：
```
vim /etc/hadoop/hadoop-env.sh
```
```
export JAVA_HOME=/usr/lib/java-1.8.0/jdk1.8.0_144
```
查看是否启动成功
```
jps
```
成功的话如下
```
7617 Jps
8214 DataNode
8393 SecondaryNameNode
8074 NameNode
```
关闭
```
sbin/stop-dfs.sh
```
当成功启动之后，可以在浏览器通过访问网址http://192.168.3.39:50070/

![hdfs成功](http://img.blog.csdn.net/20170930154521140?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


  [1]: http://www.apache.org/dyn/closer.cgi/hadoop/common/
  [2]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
  
