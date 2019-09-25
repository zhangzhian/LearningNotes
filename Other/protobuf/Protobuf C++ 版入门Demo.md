# Protobuf C++ 版入门Demo

## 前言
有关其编译和安装请查看：[Protobuf C++ 版编译安装和简单使用](https://blog.csdn.net/baidu_32237719/article/details/99649451)

之前已经进行了编译安装，并且成功将已知的proto文件转化为cc和h。

本文简单探讨如何使用Protobuf进行数据写入和读取，也就是做一个小demo。

## 定义数据类型
### proto文件
```

syntax = "proto3";

enum Messagetype
{
	REQUEST_RESPONSE_NONE = 0;            
	REQUEST_HEARTBEAT_SIGNAL = 1;          
	RESPONSE_HEARTBEAT_RESULT = 2;      
}

message MsgResult
{
	bool result =1;  
	bytes error_code = 2; 
}

message TopMessage
{
	Messagetype message_type = 1; 		//message type
	MsgResult msg_result = 2;

}
```
`syntax` 指定protobuf的版本,必须是文件的第一行

`message` 代表这是一个消息,TopMessage消息中指定了2个字段,每一个字段对应于要包含在这种类型的消息中的数据,每一个字段都有一个名称和类型

`enum` 代表枚举

等号后面的不是默认值,可以认为是该字段的身份证,不可重复,这些数字用于以二进制格式标识您的字段,一旦消息类型被使用就不应该被更改
> **注意:** 需要注意的是1到15范围内的字段编号需要一个字节来编码,而16到2047范围内的字段编号需要两个字节,所以经常使用的应该为其编码为1到15之间,这些数字的范围是[1, 536870911],其中19000～19999是为协议缓冲区实现而保留的,不可以使用,否则报错

### 对照关系
![对照关系](https://img-blog.csdnimg.cn/20190819141517203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)

## 解析出.cc和.h文件
使用以下命令
```
protoc proto文件路径 --cpp_out=C++代码文件导出目录
```

## Protobuf数据写入和读取

```c

#include "test.pb.h"		//解析出来的.h文件
#include "stdio.h"

void sendHeart();
void receHeart(TopMessage* topMessage);
void receHeartResp(TopMessage* topMessage);

/*
** ===================================================================
**     Method      :  sendHeart 
**
**     Description :  数据写入
** 
** ===================================================================
*/
void sendHeart(){
	
	TopMessage message;
	message.set_message_type(REQUEST_HEARTBEAT_SIGNAL);
	printf("sendHeart %d\n",message.message_type());
	receHeart(&message);
}

/*
** ===================================================================
**     Method      :  receHeart 
**
**     Description :  数据读取然后写入
** 
** ===================================================================
*/
void receHeart(TopMessage* topMessage){

	if (topMessage->message_type() == REQUEST_HEARTBEAT_SIGNAL)
	{
		
		printf("request_heartbeat_signal\n");
		TopMessage topMessageResp;

		MsgResult mesResult;
		mesResult.set_result(true);
	
		mesResult.set_error_code("error");
		
		topMessageResp.set_message_type(RESPONSE_HEARTBEAT_RESULT);
		
		*topMessageResp.mutable_msg_result() = mesResult;

		receHeartResp(&topMessageResp);
	}
	
}

/*
** ===================================================================
**     Method      :  receHeartResp 
**
**     Description :  数据读取
** 
** ===================================================================
*/
void receHeartResp(TopMessage* topMessage){

	if (topMessage->message_type() == RESPONSE_HEARTBEAT_RESULT)
	{
		printf("response_heartbeat_result\n");
		
		printf("%s\n",topMessage->msg_result().error_code().c_str());

	}
}

int main()
{
	sendHeart();
	google::protobuf::ShutdownProtobufLibrary();
}

```

先进行数据写入，然后数据读取，根据读取到的数据，进行数据写入，最后再读取验证。

整个Demo很简单，包含了数据的读取和写入基本操作

详细的读取和写入方式还请自行查找相关文档或访问官网，我这里由于官网需要翻墙无法访问，且c++水平有限，仅通过解码后的.cc 和 .h进行推测

## 编译

```
 g++ -std=c++11 test.pb.cc test.cpp -o test `pkg-config --cflags --libs protobuf`
```

`test.pb.cc`为解析出来的.cc源文件
`test.cpp`为编写Demo的源文件
`-std=c++11` 需要ISO C++ 2011 standard 支持，必须加，否则会报下列错误，编译无法通过
```
/usr/include/c++/4.8/bits/c++0x_warning.h:32:2: error: #error This file requires compiler and library support for the ISO C++ 2011 standard. This support is currently experimental, and must be enabled with the -std=c++11 or -std=gnu++11 compiler options.
```

## 执行
执行生成的test文件

```
./test
```
输出如下：

```
sendHeart 1
request_heartbeat_signal
response_heartbeat_result
error
```

