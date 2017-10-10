# windows系统RNDIS驱动手动安装

windows系统中RNDIS自动当成串口，按网上的更新驱动的方式无法更新为RNDIS，所以采用手动更新的方式，亲测win7和win10可用

 1. 在设备管理器选择需要更新RNDIS驱动的设备
> **注意：**一定要选择正确的设备，选择错误的话无法成功，不确定是否正确可插拔设备进行
验证

 2. 右键“更新驱动程序”
 3. 选择第二项“浏览我的计算机以查找驱动程序软件”
![这里写图片描述](http://img.blog.csdn.net/20171010092230683?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 4. 选择“让我从计算机上的可用驱动程序列表中选取”
![这里写图片描述](http://img.blog.csdn.net/20171010092646978?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 5. 点击“从磁盘安装”
![这里写图片描述](http://img.blog.csdn.net/20171010092818988?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 6. 点击“浏览”选择RNDIS驱动的位置
![这里写图片描述](http://img.blog.csdn.net/20171010092902322?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 7. 选择RNDIS.inf
![这里写图片描述](http://img.blog.csdn.net/20171010092954794?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
RNDIS下载地址[RNDIS.7z](https://github.com/zhangzhian/LearningNotes/blob/master/res/RNDIS.7z)


