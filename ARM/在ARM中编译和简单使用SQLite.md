# 在ARM中编译和简单使用SQLite

## 简介

SQLite是一种C语言库，实现了一个 小型， 快速， 自包含， 高可靠性，功能齐全的 SQL数据库引擎。SQLite是世界上最常用的数据库引擎。SQLite内置于所有手机和大多数计算机中，在人们每天使用的无数其他应用程序中都有使用。 

SQLite 文件格式稳定，跨平台且向后兼容，开发人员保证至少在2050年之前保持这种格式。SQLite数据库文件通常用作在系统之间传输丰富内容的容器，并作为数据的长期存档格式。活跃使用的SQLite数据库超过1万亿（1e12）。

SQLite 源代码开源，任何人都可以免费使用于任何目的。

官方主页：https://www.sqlite.org/index.html

## 源码下载

官方下载页面：https://www.sqlite.org/download.html

sqlite-amalgamation-3310100.zip	C源代码，合并，为3.31.1版：https://www.sqlite.org/2020/sqlite-amalgamation-3310100.zip

sqlite-autoconf-3310100.tar.gz	C源代码，合并，还包括“配置”脚本和用于TCL接口的TEA makefiles 文件：https://www.sqlite.org/2020/sqlite-autoconf-3310100.tar.gz



## 文献资料

官方文档资料：https://www.sqlite.org/docs.html

## 编译

### 程序中使用

SQL不需要额外编译，官方提供了作为预打包的合并源代码文件：**sqlite3.c**。合并后更容易处理，所有内容都包含在一个代码文件中，很容易放入较大的C或C ++程序的源代码中。

所有代码生成和转换步骤均已执行，因此无需配置和编译辅助C程序，也无需运行脚本。而且，由于整个库都包含在一个源文件中，因此编译器能够进行更高级的优化，从而将性能提高5％到10％。

直接下载：sqlite-amalgamation-3310100.zip。

### 编译命令行界面

1.将下载好的sqlite-autoconf-3310100.tar.gz存放在/root/sqlite（根据自己需要进行修改）。

2.解压

```shell
tar -zxvf sqlite-autoconf-3310100.tar.gz
```

生成sqlite-autoconf-3310100目录

命令行界面的构建需要三个源文件：

- **sqlite3.c**：SQLite合并源文件
- **sqlite3.h**：sqlite3.c附带的头文件，它定义SQLite的C语言接口。
- **shell.c**：命令行界面程序本身。这是C源代码文件，其中包含main（）例程的定义以及提示用户输入并将该输入传递到SQLite数据库引擎进行处理的循环。

3.创建“/usr/local/sqlite”目录，用于存储生成文件

4.配置

进入sqlite-autoconf-3310100目录中:

```shell
./configure --host=arm-linux --prefix=/usr/local/sqlite/ CC=arm-none-linux-gnueabi-gcc CXX=arm-none-linux-gnueabi-g++
```

生成Makefile文件

5.编译和安装

```shell
make
make install
```

6.移植

编译好后在/usr/local/sqlite/目录中会生成4个文件夹“bin 、include 、lib 、share”

将bin文件夹中的文件拷贝到开发板的/bin中，并将lib中的文件拷贝到开发板的/usr/lib中。

7.检验是否移植成功：

输入sqlite3 test.db，如下即成功。

```shell
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
```

## 简单使用

### 创建库

```c
	rc = sqlite3_open("/root/sqlite/test.db", &db); //打开指定的数据库文件,如果不存在将创建一个同名的数据库文件
	if(rc)
	{
		//fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
		printf("[SQLite]Can't open database: %s\n",sqlite3_errmsg(db));
		sqlite3_close(db);
	}	
```

### 创建表

```c
	//创建一个表,如果该表存在，则不创建，并给出提示信息，存储在 zErrMsg 中
	char *sql = "CREATE TABLE TestData(ID INTEGER PRIMARY KEY,Info VARCHAR(12),Num INTEGER,Data REAL); \
				CREATE TABLE BLOBData(ID INTEGER PRIMARY KEY,BData BLOB);";

	sqlite3_exec( db , sql , 0 , 0 , &zErrMsg);
```

### 增

```c
	for (i = 0; i < 3; i++)
	{
		char sql[1024] = {0}; 

		sprintf(sql,"INSERT INTO TestData VALUES( NULL ,'hello world', %d, %f);",i,16.0);

		sqlite3_exec(db , sql, 0, 0, &zErrMsg);
    }
```

### 删

```c
	//删除数据
	sql = "DELETE FROM TestData WHERE Num = 0;" ;
	sqlite3_exec( db , sql , 0 , 0 , &zErrMsg );
```

### 改

```c
	sql = "UPDATE TestData SET Info = 'ok' WHERE Num = 1;" ;
	sqlite3_exec( db , sql , 0 , 0 , &zErrMsg );
```

### 查

方法1：回调方式

```c
	static int callback(void *data, int argc, char **argv, char **azColName){
       int i;
       printf("[SQLite]%s: ", (const char*)data);
       for(i=0; i<argc; i++){
          printf("%s = %s  ", azColName[i], argv[i] ? argv[i] : "NULL");
       }
       printf("\n");
       return 0;
    }
	
	...

	sql = "SELECT * FROM TestData;";
	const char* data = "Callback";
	rc = sqlite3_exec(db, sql, callback, (void *)data, &zErrMsg);
	if( rc != SQLITE_OK ){
      printf("[SQLite]SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
	}else{
      printf("[SQLite]Operation done successfully\n");
	}
```

方法2：

```c
	int nrow = 0, ncolumn = 0;
	int i,j;
	char *zErrMsg = 0;
	char **azResult; //二维数组存放结果
	sqlite3_get_table( db , sql , &azResult , &nrow , &ncolumn , &zErrMsg );

	printf( "[SQLite]row:%d column=%d \n" , nrow , ncolumn );
	for( i=0 ; i< nrow + 1; i++ ){
		for( j=0 ; j< ncolumn ; j++ ){
			printf( "%s ",azResult[i * ncolumn + j]);
		}
		printf( "\n");
	}
	//释放掉 azResult 的内存空间
	sqlite3_free_table( azResult );
```

### 二进制数据：

#### 增

```c
	sqlite3_stmt* stat;//重复使用 sqlite3_prepare 解析好的 sqlite3_stmt 结构，需要用函数： sqlite3_reset。
	sqlite3_prepare( db, "INSERT INTO BLOBData(ID,BData) VALUES( NULL, ?)", -1, &stat, 0 );
	// buff为数据缓冲区，sizeof(buff)为数据大小，以字节为单位
	sqlite3_bind_blob(stat, 1, (const void *)buff, sizeof(buff), NULL);
	
	int result = sqlite3_step(stat);

	printf("[SQLite]SQL: %d %s\n", result,sqlite3_errmsg(db));

	sqlite3_finalize( stat ); //把刚才分配的内容析构掉
	
```

#### 查

```c
	sqlite3_stmt * stat_q;
	sqlite3_prepare( db,"select * from BLOBData", -1, &stat_q, 0 );
	int result1 = sqlite3_step( stat_q );
	printf("[SQLite]SQL: %d %s\n", result1,sqlite3_errmsg(db));

	int id = sqlite3_column_int( stat_q, 0 ); //第2个参数表示获取第几个字段内容，从0开始计算，因为我的表的ID字段是第一个字段，因此这里我填0
	UINT8* pContent = (UINT8*)sqlite3_column_blob( stat_q, 1 );
	int len = sqlite3_column_bytes( stat_q, 1 );
	printf("[SQLite]SQL len: %d\n", len);
	for ( i = 0; i < len; i++)
	{
		printf("[%02x]",pContent[i]);
	}
	printf("\n");

	sqlite3_finalize( stat_q ); //把刚才分配的内容析构掉
```

### 完整Demo

编写c程序testDB.c：

```c
// testDB.c

#include <stdio.h>
#include <string.h>
#include <sqlite/sqlite3.h>

typedef	unsigned char	UINT8;
typedef	unsigned short	UINT16;


int query(sqlite3 *db,char *sql){
	int nrow = 0, ncolumn = 0;
	int i,j;
	char *zErrMsg = 0;
	char **azResult; //二维数组存放结果
	sqlite3_get_table( db , sql , &azResult , &nrow , &ncolumn , &zErrMsg );

	printf( "[SQLite]row:%d column=%d \n" , nrow , ncolumn );
	for( i=0 ; i< nrow + 1; i++ ){
		for( j=0 ; j< ncolumn ; j++ ){
			printf( "%s ",azResult[i * ncolumn + j]);
		}
		printf( "\n");
	}
	//释放掉 azResult 的内存空间
	sqlite3_free_table( azResult );
}

static int callback(void *data, int argc, char **argv, char **azColName){
   int i;
   printf("[SQLite]%s: ", (const char*)data);
   for(i=0; i<argc; i++){
      printf("%s = %s  ", azColName[i], argv[i] ? argv[i] : "NULL");
   }
   printf("\n");
   return 0;
}


int main(int argc, char* argv[])
{
	sqlite3 *db=NULL;
	int rc;
	char *zErrMsg = 0;
	int i,j;

	printf("[SQLite]===================open=================\n");

	rc = sqlite3_open("/root/sqlite/test.db", &db); //打开指定的数据库文件,如果不存在将创建一个同名的数据库文件
	if(rc)
	{
		//fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
		printf("[SQLite]Can't open database: %s\n",sqlite3_errmsg(db));
		sqlite3_close(db);
	}
	printf("[SQLite]===================create=================\n");
	//创建一个表,如果该表存在，则不创建，并给出提示信息，存储在 zErrMsg 中
	char *sql = "CREATE TABLE TestData(ID INTEGER PRIMARY KEY,Info VARCHAR(12),Num INTEGER,Data REAL); \
				CREATE TABLE BLOBData(ID INTEGER PRIMARY KEY,BData BLOB);";

	sqlite3_exec( db , sql , 0 , 0 , &zErrMsg);

	printf("[SQLite]===================insert=================\n");

	for (i = 0; i < 3; i++)
	{
		char sql[1024] = {0}; 

		sprintf(sql,"INSERT INTO TestData VALUES( NULL ,'hello world', %d, %f);",i,16.0);

		sqlite3_exec(db , sql, 0, 0, &zErrMsg);
    }
	printf("[SQLite]===================query=================\n");
	
	sql = "SELECT * FROM TestData;";

	query(db,sql);
	printf("[SQLite]===================delete=================\n");
	
	//删除数据
	sql = "DELETE FROM TestData WHERE Num = 0;" ;
	sqlite3_exec( db , sql , 0 , 0 , &zErrMsg );

	printf("[SQLite]===================update=================\n");

	sql = "UPDATE TestData SET Info = 'ok' WHERE Num = 1;" ;
	sqlite3_exec( db , sql , 0 , 0 , &zErrMsg );

	printf("[SQLite]===================query=================\n");

	sql = "SELECT * FROM TestData;";
	const char* data = "Callback";
	rc = sqlite3_exec(db, sql, callback, (void *)data, &zErrMsg);
	if( rc != SQLITE_OK ){
      printf("[SQLite]SQL error: %s\n", zErrMsg);
      sqlite3_free(zErrMsg);
	}else{
      printf("[SQLite]Operation done successfully\n");
	}

	printf("[SQLite]===================insert blob=================\n");
	sqlite3_stmt* stat;//重复使用 sqlite3_prepare 解析好的 sqlite3_stmt 结构，需要用函数： sqlite3_reset。
	sqlite3_prepare( db, "INSERT INTO BLOBData(ID,BData) VALUES( NULL, ?)", -1, &stat, 0 );
	


	//char* buf = "abc#START* hello#END*#START* hello#END*"; 
	UINT8 buff[41] = {0};
	UINT16 u16Data = 5;
	memcpy(buff,"abc#START*",strlen("abc#START*"));
	buff[10] = (UINT8)(u16Data >> 8);
	buff[11] = (UINT8)u16Data;

	//memcpy(buff + 10,&u16Data,2);
	memcpy(buff + 12,"hello#END*#START*",strlen("hello#END*#START*"));
	buff[29] = (UINT8)(u16Data >> 8);
	buff[30] = (UINT8)u16Data;
	memcpy(buff + 31,"hello#END*",strlen("hello#END*"));
	//printf("[client]buff %s\n",buff);


	// pdata为数据缓冲区，length_of_data_in_bytes为数据大小，以字节为单位
	sqlite3_bind_blob(stat, 1, (const void *)buff, sizeof(buff), NULL);
	
	int result = sqlite3_step(stat);

	printf("[SQLite]SQL: %d %s\n", result,sqlite3_errmsg(db));

	sqlite3_finalize( stat ); //把刚才分配的内容析构掉

	printf("[SQLite]===================query blob=================\n");

	sqlite3_stmt * stat_q;
	sqlite3_prepare( db,"select * from BLOBData", -1, &stat_q, 0 );
	int result1 = sqlite3_step( stat_q );
	printf("[SQLite]SQL: %d %s\n", result1,sqlite3_errmsg(db));

	int id = sqlite3_column_int( stat_q, 0 ); //第2个参数表示获取第几个字段内容，从0开始计算，因为我的表的ID字段是第一个字段，因此这里我填0
	UINT8* pContent = (UINT8*)sqlite3_column_blob( stat_q, 1 );
	int len = sqlite3_column_bytes( stat_q, 1 );
	printf("[SQLite]SQL len: %d\n", len);
	for ( i = 0; i < 41; i++)
	{
		printf("[%02x]",pContent[i]);
	}
	printf("\n");

	sqlite3_finalize( stat_q ); //把刚才分配的内容析构掉
	printf("[SQLite]===================close=================\n");

	sqlite3_close(db); //关闭数据库
	

	return 0;
}
```

交叉编译：

```powershell
arm-none-linux-gnueabi-gcc testDB.c -o testDB -lsqlite3 -L/usr/local
/sqlite/lib/ -I/usr/local/sqlite/include
```

将生成的bin文件移植到ARM中执行：

```
./testDB
```

可以看到在当前目录下生成了test.db文件。



后续如果还有时间和项目应用SQLite的话，希望将其封装为类似Android SQLite API 接口的形式，不需要使用SQL语句即可进行增删改查。



**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)