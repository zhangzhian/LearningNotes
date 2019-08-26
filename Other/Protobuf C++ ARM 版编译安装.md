# Protobuf C++ ARM 版编译安装
## 前言
Protobuf C++ ARM 版依赖于linux版本，需要使用交叉编译环境进行编译，这里使用的是 arm-none-linux-gnueabi-c++

 [arm-none-linux-gnueabi-c++下载地址](https://download.csdn.net/download/baidu_32237719/11608951)

## 安装交叉编译环境
1. 下载arm-none-linux-gnueabi-c++
2. 将其移动到linux目录下
3. 解压

```c
tar -jxvf arm-2014.05-29-arm-none-linux-gnueabi-i686-pc-linux-gnu.tar.bz2
```

4. 修改环境变量

```c
 vim /etc/profile
```
添加解压后文件所在路径（... 代表路径，替换为自己的）
```c
export PATH=$PATH:.../arm/bin
```

5. 保存执行

```c
source /etc/profile
```



## 验证
```c
arm-none-linux-gnueabi-c++ -v
```
如果返回如下图所示，则成功
![-v 结果](https://img-blog.csdnimg.cn/20190824153024904.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)
### 可能存在问题

```c
arm-none-linux-gnueabi-c++ -v
```
提示找不到路径之类的错误，可能是有与32位和64位造成
需要安装兼容库

```c
sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0
```

## 安装protobuf linux版本
参考[Protobuf C++ 版编译安装和简单使用](https://blog.csdn.net/baidu_32237719/article/details/99649451)

## 安装protobuf arm版本
安装protobuf arm版本需要先安装linux版本，用到其中的生成文件

### 生成Makefile文件

```c
./configure --host=arm-linux --prefix=/usr/local/protobuf_arm --with-protoc=/usr/local/protobuf/bin/protoc CC=arm-none-linux-gnueabi-gcc CXX=arm-none-linux-gnueabi-g++
```
- `--host`编译的版本为arm-linux
- `--prefix`为生成文件的路径
- `--with-protoc`为linux版protoc的路径，即之前安装的linux版本生成的目录中的工具
- `CC`指定编译c的工具链为`arm-none-linux-gnueabi-gcc`
- `CXX`指定编译c的工具链为`arm-none-linux-gnueabi-g++`

这一步主要目的是用来生成Makefile文件


### 编译安装
编译：`make`
测试： `make check`
安装 ： `make install`


### 设置环境变量
添加如下环境变量

```c
 vim /etc/profile
```

```c
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/protobuf_arm/lib
export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/protobuf_arm/lib
export PATH=$PATH:/usr/local/protobuf_arm/bin
```
保存执行
```c
source /etc/profile
```

## 编译Demo

```c
arm-none-linux-gnueabi-c++ -std=c++11 Test.pb.cc test.cpp -o test `pkg-config --cflags --libs protobuf-lite`
```
上述链接的protobuf-lite库，有些高级用法进行了删减，主要是减小链接库烦人体积，对应的`.proto`文件添加`option optimize_for = LITE_RUNTIME;`

也可以采用protobuf库，包含完整用法，不需要加`option optimize_for`选项


## 使用生成的文件
在liunx已无法直接运行，需要移动到ARM端执行

在执行前需要先将依赖的库推入ARM设备

```c
adb shell mount -o rw,remount /
adb push protobuf-lite.so.20 /usr/lib/protobuf-lite.so.20
```
我这里根据错误提示推入的是protobuf-lite.so.20

然后即可执行

```c
./tets
```

