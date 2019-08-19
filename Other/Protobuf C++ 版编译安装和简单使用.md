# Protobuf C++ 版编译安装和简单使用
## Protobuf 简介
Protobuf是一种平台无关、语言无关、可扩展且轻便高效的序列化数据结构的协议，可以用于网络通信和数据存储。

[项目官方GitHub](https://github.com/protocolbuffers/protobuf)

[项目官方编译安装指导](https://github.com/protocolbuffers/protobuf/blob/master/src/README.md)

本次编译安装版本为 [protobuf-cpp-3.9.1.tar.gz](https://github.com/protocolbuffers/protobuf/releases/download/v3.9.1/protobuf-cpp-3.9.1.tar.gz)

## Protobuf 特点
![protobuf特点](https://github.com/zhangzhian/LearningNotes/blob/master/res/protobuf特点.png?raw=true)
## Protobuf 编译安装
在Linux 64位环境下进行编译

### 安装依赖的工具
```
$ sudo apt-get install autoconf automake libtool curl make g++ unzip
```

### 文件下载

#### 官方GitHub下载
[GitHub Release 页面](https://github.com/protocolbuffers/protobuf/releases)

选择protobuf-cpp-3.9.1.tar.gz

我需要编译的版本是C++版的，所以使用cpp的版本，最新是3.9.1

#### git clone

```
    git clone https://github.com/protocolbuffers/protobuf.git
    cd protobuf
    git submodule update --init --recursive
    ./autogen.sh
```
我这里使用的是官方GitHub下载，这种方式下载后未实践

### 文件解压
解压： `tar -zvf protobuf-cpp-3.9.1.tar.gz`
进入到protobuf目录： `cd protobuf-3.9.1`


### 指定安装目录
安装在/usr/local/protobuf
```
  ./configure --prefix=/usr/local/protobuf
```

### 编译安装

编译：`make`
测试： `make check`
安装 ： `make install`

### 设置环境变量
添加如下环境变量
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/protobuf/lib
export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/protobuf/lib
export PATH=$PATH:/usr/local/protobuf/bin
```
保存执行

```
source /etc/profile
```
检查版本号
```
protoc --version
```

## Protobuf 简单使用
将protobuf文件转为c++文件 

```
protoc proto文件路径 --cpp_out=C++代码文件导出目录
```
例如： `protoc ./test.proto --cpp_out=./trans`

以下为使用编译安装好的Protobuf 进行数据的写入和读取demo

[Protobuf C++ 版入门Demo](https://blog.csdn.net/baidu_32237719/article/details/99723353)
