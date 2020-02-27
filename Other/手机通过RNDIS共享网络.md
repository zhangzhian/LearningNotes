# 手机通过RNDIS共享网络

RNDIS 是 Remote Network Driver Interface Specification（远程网络驱动程序接口规范） 的首字母缩写，实际上的作用为 TCP/IP over USB，也即把 USB 设备（如手机）作为网卡，是基于USB实现RNDIS实际上就是TCP/IP over USB，从而使 Windows 可以通过 USB 设备连接网络。

## RNDIS驱动安装
RNDIS驱动安装可以参考查看下列博文。
[windows系统RNDIS驱动手动安装](https://blog.csdn.net/baidu_32237719/article/details/78189144)


## 手机设置
笔者使用的是荣耀系列手机，荣耀20（Magic UI 2.1.0）和荣耀9（EMUI 9.0.1），Android版本9

### 1.首先连接USB线

>注意：必须先连接USB，否则设置中不会出现usb共享网络选项

### 2.设置USB共享网络

荣耀20：
无线和网络-->个人热点-->更多共享设置-->USB共享网络
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200227114952888.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)
荣耀9：
无线和网络-->移动网络共享-->USB共享网络

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200227115056141.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)
其他方式：

在开发这模式-->选择USB配置中：

写![在这里插入图片描述](https://img-blog.csdnimg.cn/20200227115705750.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)
>注意：在我的手机中，直接在通知栏的设置usb连接方式中是没有RNDIS的
### 3.电脑查看
在控制面板\所有控制面板项\网络连接里：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200227115358940.png)
出现Remote NDIS based Internet Sharing Device时PC便可以使用手机网络上网。

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)