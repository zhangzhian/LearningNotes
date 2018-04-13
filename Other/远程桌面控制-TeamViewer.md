# 远程桌面控制-TeamViewer

---

### 一.前提

 - 远程控制PC和被控制PC都需要安装TeamViewer软件
 - 被控制PC保持开启状态，网络畅通，TeamViewer打开

Teamviewer软件[下载地址][1]

### 二.使用

#### 1.不登录TeamViewer使用

![](http://img-blog.csdn.net/20180413094558581?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
不登录TeamViewer账户使用依赖于ID和密码，密码每次打开TeamViewer会随机生成，所需以要记录被控制电脑的ID和密码。
在控制端PC输入“伙伴ID”，然后“连接到伙伴”，输入密码即可进行远程操作

> **注意：** 这种方法存在一种问题，需要提前记录密码，如果突然需要操作被控制PC，未记录密码，则无法控制。

#### 2.登录TeamViewer使用
- 注册TeamViewer账号

> **注意：** TeamViewer官网注册的时候网络很差，需要耐心点，不行的话多试几次。

- 在TeamViewer中登录

![这里写图片描述](http://img-blog.csdn.net/2018041309553157?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> **注意：** TeamViewer在PC第一次登录后需要设备授权操作，大概是发一封邮件，点击验证。

- 添加计算机

![这里写图片描述](http://img-blog.csdn.net/20180413095653263?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

有两种方案：添加本计算机和远程计算机。在被控制PC上登录后，直接添加本计算机即可。添加成功后“我的计算机”可以看到添加好的计算机

- 远程控制

在控制端PC中登录同一个TeamViewer账号，打开“我的计算机”，双击需要控制的PC即可进行控制。

  [1]: https://www.teamviewer.com/zhcn/download/windows/