在Android环境下编译 bsdiff/bspatch 增量更新库
----

## 一、 概述
什么增量更新呢？比如：应用市场省流量更新软件，一个100M的apk可能只需要下载一个20M的增量包就能完成更新，不需要下载整个Apk。增量更新不仅限于apk。

本篇博客主要记录bsdiff/bspatch增量更新编译为so库的过程。分为2个部分提取增量文件和合并增量文件
[CSDN](https://blog.csdn.net/baidu_32237719/article/details/88318760)
### 准备工作
1. [bsdiff/bspatch下载路径](https://github.com/mendsley/bsdiff)
2. [bzip2下载路径](https://github.com/zhangzhian/Bsdiff_Bspatch/tree/master/app/src/main/cpp/bzip2)
很多资料提到[bzip2官方网站](http://www.bzip.org/downloads.html)，现在未发现下载链接。不是本文重点，不再关注。
3. Android Studio 含NDK

## 二、创建项目
### 1.bsdiff/bspatch

![bsdiff:bspatch.jpeg](https://github.com/zhangzhian/Bsdiff_Bspatch/blob/master/art/bsdiff:bspatch.jpeg?raw=true)

解压bsdiff/bspatch下载好的文件，如上图所示。

重点关注1和2两个文件

- bsdiff.c 提取增量文件

- bspatch.c 合并增量文件

### 2. 创建Android 项目 
首先是用Android Studio创建一个“include c++ support”带有c++支持的项目。需要用到NDK，NDK部分的内容若有疑问请自行搜索。

![android.png](https://github.com/zhangzhian/Bsdiff_Bspatch/blob/master/art/android.png?raw=true)

将我们下载的bzip2的源文件和bsdiff/bspatch的源文件，放到Android项目中，如上图。

diff-patch-lib.cpp是项目自己生成的cpp，改了个名字。

### 3.修改CMakeList.txt

```txt
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

cmake_minimum_required(VERSION 3.4.1)

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

add_library( # Sets the name of the library.
             diff-patch

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/diff-patch-lib.cpp
             src/main/cpp/bsdiff.c
             src/main/cpp/bspatch.c
             src/main/cpp/bzip2/blocksort.c
             src/main/cpp/bzip2/bzip2.c
             src/main/cpp/bzip2/bzip2recover.c
             src/main/cpp/bzip2/bzlib.c
             src/main/cpp/bzip2/compress.c
             src/main/cpp/bzip2/crctable.c
             src/main/cpp/bzip2/decompress.c
             src/main/cpp/bzip2/huffman.c
             src/main/cpp/bzip2/randtable.c
             )

# Searches for a specified prebuilt library and stores the path as a
# variable. Because CMake includes system libraries in the search path by
# default, you only need to specify the name of the public NDK library
# you want to add. CMake verifies that the library exists before
# completing its build.

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

# Specifies libraries CMake should link to your target library. You
# can link multiple libraries, such as libraries you define in this
# build script, prebuilt third-party libraries, or system libraries.

target_link_libraries( # Specifies the target library.
                       diff-patch

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```
我把要生成的库的名字改为diff-patch

add_library中添加下载的c源文件

### 4. 添加JNI接口

设计到jni的知识，不是本文重点，如果不了解请自行学习。

- 加载diff-patch库

- 提供了diff和patch两个方法

```java
package com.yodosmart.bsdiffbspatch;
/**
 * @Author: 张志安
 * @Mail: zhangzhian2016@gmail.com
 * @Date: 2019/3/6 15:44
 */
public class DiffPatchUtil {

    static {
        System.loadLibrary("diff-patch");
    }

    /**
     * native方法 比较路径为oldPath的文件与newPath的文件之间差异，并生成patch包，存储于patchPath
     *
     * 返回：0，说明操作成功
     *
     * @param oldPath 示例:/sdcard/old.apk
     * @param newPath 示例:/sdcard/new.apk
     * @param patchPath  示例:/sdcard/xx.patch
     * @return
     */
    public static native int diff(String oldPath, String newPath, String patchPath);


    /**
     * native方法 使用路径为oldPath的文件与路径为patchPath的补丁包，合成新的文件，并存储于newPath
     *
     * 返回：0，说明操作成功
     *
     * @param oldPath 示例:/sdcard/old.apk
     * @param newPath 示例:/sdcard/new.apk
     * @param patchPath  示例:/sdcard/xx.patch
     * @return
     */
    public static native int patch(String oldPath, String newPath,
                                   String patchPath);

}

```
### 5、注意
后续会遇到的一些问题

- 添加权限`<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>`
- diff-patch-lib.cpp include头文件的时候需要添加extern "C"，如下
```c
extern "C"
{
#include "bsdiff.h"
#include "bspatch.h"
}
```
- bsdiff.c和bspatch.c中的`#if defined(BSDIFF_EXECUTABLE) #endif`去掉
- bsdiff.c `#include "bzip2/bzlib.h"`
- bspatch.c `#include "bspatch.h"  #include "bzip2/bzlib.h"`


## 三、提取增量文件
首先我们来看如何提取增量文件

```c
extern "C"
JNIEXPORT jint JNICALL
Java_com_yodosmart_bsdiffbspatch_DiffPatchUtil_diff(JNIEnv *env, jclass type, jstring oldPath_,
                                                    jstring newPath_, jstring patchPath_) {

    const char *oldPath = env->GetStringUTFChars(oldPath_, 0);
    const char *newPath = env->GetStringUTFChars(newPath_, 0);
    const char *patchPath = env->GetStringUTFChars(patchPath_, 0);

    int argc = 4;
     char * argv[argc];
    argv[0] = (char *)"bsdiff";
    argv[1] = (char *)oldPath;
    argv[2] = (char *)newPath;
    argv[3] = (char *)patchPath;

    int ret = bsdiff_main(argc,argv);

    env->ReleaseStringUTFChars(oldPath_, oldPath);
    env->ReleaseStringUTFChars(newPath_, newPath);
    env->ReleaseStringUTFChars(patchPath_, patchPath);

    return ret;
}
```

主要是用bsdiff_main(argc,argv)调用bsdiff库

源文件中含有main函数，我这里主要是把main函数改了名称，添加了注释

```java
int bsdiff_main(int argc, char *argv[]) {
    //文件句柄
    int fd;
    //bz2错误
    int bz2err;
    //老版本文件 新版本文件
    uint8_t *old, *new;
    //旧版本文件和新版本文件的大小
    off_t oldsize, newsize;
    //大小为8的buff
    uint8_t buf[8];
    //增量文件
    FILE *pf;
    //差分结构体
    struct bsdiff_stream stream;
    //bz2文件
    BZFILE *bz2;

    memset(&bz2, 0, sizeof(bz2));
    stream.malloc = malloc;
    stream.free = free;
    stream.write = bz2_write;

    if (argc != 4) errx(1, "usage: %s oldfile newfile patchfile\n", argv[0]);

    /*
     *  Allocate oldsize+1 bytes instead of oldsize bytes to ensure
     *  that we never try to malloc(0) and get a NULL pointer
     *
     *  旧版本文件分配内存，读出文件内容
     *
     *
     *  int open(const char * pathname, int flags, mode_t mode);
     *
     *  参数 pathname 指向欲打开的文件路径字符串
     *  参数 flags 为文件的打开方式 O_RDONLY 以只读方式打开文件
     *  参数 mode 只有在建立新版本文件时才会生效, 真正建文件时的权限会受到umask值所影响, 因此该文件权限应该为 (mode-umaks).
     *
     *  返回值：成功则返回文件句柄，否则返回-1
     *
     *
     *  off_t lseek(int filedes, off_t offset, int whence);
     *
     *  参数 offset 的含义取决于参数 whence：
     *    1. 如果 whence 是 SEEK_SET，文件偏移量将被设置为 offset。
     *    2. 如果 whence 是 SEEK_CUR，文件偏移量将被设置为 cfo 加上 offset，
     *       offset 可以为正也可以为负。
     *    3. 如果 whence 是 SEEK_END，文件偏移量将被设置为文件长度加上 offset，
     *       offset 可以为正也可以为负。
     *
     *  返回值：新的偏移量（成功），-1（失败）
     *
     *
     *  ssize_t read(int fd, void * buf, size_t count);
     *
     *  参数 void *buf 读上来的数据保存在缓冲区buf中，同时文件的当前读写位置向后移
     *  参数 size_t count 是请求读取的字节数。若参数count 为0, 则read()不会有作用并返回0.
     *
     *  返回值：为实际读取到的字节数
     *
     */
    if (((fd = open(argv[1], O_RDONLY, 0)) < 0) ||
        ((oldsize = lseek(fd, 0, SEEK_END)) == -1) ||
        ((old = malloc(oldsize + 1)) == NULL) ||
        (lseek(fd, 0, SEEK_SET) != 0) ||
        (read(fd, old, oldsize) != oldsize) ||
        (close(fd) == -1))
        err(1, "%s", argv[1]);

    /*
     * Allocate newsize+1 bytes instead of newsize bytes to ensure
     * that we never try to malloc(0) and get a NULL pointer
     *
     * 新版本文件分配内存，读取文件内存
     *
     */
    if (((fd = open(argv[2], O_RDONLY, 0)) < 0) ||
        ((newsize = lseek(fd, 0, SEEK_END)) == -1) ||
        ((new = malloc(newsize + 1)) == NULL) ||
        (lseek(fd, 0, SEEK_SET) != 0) ||
        (read(fd, new, newsize) != newsize) ||
        (close(fd) == -1))
        err(1, "%s", argv[2]);

    /*
     * Create the patch file
     *
     * 创建一个patch文件
     */
    if ((pf = fopen(argv[3], "w")) == NULL)
        err(1, "%s", argv[3]);

    /**
     * Write header (signature+newsize)
     *
     * 写头部
     *
     * size_t fwrite(const void* buffer, size_t size, size_t count, FILE* stream);
     *
     * 返回值：返回实际写入的数据块数目
     *（1）buffer：是一个指针，对fwrite来说，是要获取数据的地址；
     *（2）size：要写入内容的单字节数；
     *（3）count:要进行写入size字节的数据项的个数；
     *（4）stream:目标文件指针；
     *
     */
    offtout(newsize, buf);
    if (fwrite("ENDSLEY/BSDIFF43", 16, 1, pf) != 1 ||
        fwrite(buf, sizeof(buf), 1, pf) != 1)
        err(1, "Failed to write header");

    /**
     * 以bz2的方式打开增量文件
     */
    if (NULL == (bz2 = BZ2_bzWriteOpen(&bz2err, pf, 9, 0, 0)))
        errx(1, "BZ2_bzWriteOpen, bz2err=%d", bz2err);

    /**
     * 差分算法
     */
    stream.opaque = bz2;
    if (bsdiff(old, oldsize, new, newsize, &stream))
        err(1, "bsdiff");

    /**
     * 关闭文件
     */
    BZ2_bzWriteClose(&bz2err, bz2, 0, NULL, NULL);

    if (bz2err != BZ_OK)
        err(1, "BZ2_bzWriteClose, bz2err=%d", bz2err);

    if (fclose(pf))
        err(1, "fclose");

    /*
     * Free the memory we used
     *
     * 释放使用的内存
     */
    free(old);
    free(new);

    return 0;
}
```

调用提取增量文件


```java
int result = DiffPatchUtil.patch(oldpath,patchNewPath,patchPath);
```

## 四、合并增量文件

接下来我们看如何合并增量文件

```c

extern "C"
JNIEXPORT jint JNICALL
Java_com_yodosmart_bsdiffbspatch_DiffPatchUtil_patch(JNIEnv *env, jclass type, jstring oldPath_,
                                                     jstring newPath_, jstring patchPath_) {
    const char *oldPath = env->GetStringUTFChars(oldPath_, 0);
    const char *newPath = env->GetStringUTFChars(newPath_, 0);
    const char *patchPath = env->GetStringUTFChars(patchPath_, 0);

    int argc = 4;
    char * argv[argc];
    argv[0] = (char *)"bspatch";
    argv[1] = (char *)oldPath;
    argv[2] = (char *)newPath;
    argv[3] = (char *)patchPath;


    int ret = bspatch_main(argc,argv);

    env->ReleaseStringUTFChars(oldPath_, oldPath);
    env->ReleaseStringUTFChars(newPath_, newPath);
    env->ReleaseStringUTFChars(patchPath_, patchPath);

    return ret;


```


主要是调用bspatch库中的`bspatch_main(argc,argv);`合并增量文件


```c

int bspatch_main(int argc,  char * argv[])
{
    //增量文件
	FILE * f;
    // 老版本文件
	int fd;
	//bz2错误
    int bz2err;
    //16+8
	uint8_t header[24];
	uint8_t *old, *new;
	int64_t oldsize, newsize;
	BZFILE* bz2;
	struct bspatch_stream stream;
	struct stat sb;

	if(argc!=4) errx(1,"usage: %s oldfile newfile patchfile\n",argv[0]);

	/**
	 * Open patch file
	 * 打开增量文件
	 */
	if ((f = fopen(argv[3], "r")) == NULL)
		err(1, "fopen(%s)", argv[3]);

	/**
	 * Read header
	 * 读头部
	 */
	if (fread(header, 1, 24, f) != 24) {
		if (feof(f))
			errx(1, "Corrupt patch\n");
		err(1, "fread(%s)", argv[3]);
	}

	/**
	 * Check for appropriate magic
	 *
	 * int memcmp(const void *buf1, const void *buf2, unsigned int count);
	 * 比较内存区域buf1和buf2的前count个字节。
	 */
	if (memcmp(header, "ENDSLEY/BSDIFF43", 16) != 0)
		errx(1, "Corrupt patch\n");

	/**
	 * Read lengths from header
	 * 与差分offtout对应
	 * 读新版本文件的长度
	 */
	newsize=offtin(header+16);
	if(newsize<0)
		errx(1,"Corrupt patch\n");

	/**
	 * 打开老版本文件
	 */
	if(((fd=open(argv[1],O_RDONLY,0))<0) ||
		((oldsize=lseek(fd,0,SEEK_END))==-1) ||
		((old=malloc(oldsize+1))==NULL) ||
		(lseek(fd,0,SEEK_SET)!=0) ||
		(read(fd,old,oldsize)!=oldsize) ||
		(fstat(fd, &sb)) ||
		(close(fd)==-1)) err(1,"%s",argv[1]);
    /**
     * 给新版本文件分配内存
     */
	if((new=malloc(newsize+1))==NULL) err(1,NULL);
    /**
	 * Close patch file and re-open it via libbzip2 at the right places
	 * 关闭增量文件并且通过libbzip2重新打开
	 */
	if (NULL == (bz2 = BZ2_bzReadOpen(&bz2err, f, 0, 0, NULL, 0)))
		errx(1, "BZ2_bzReadOpen, bz2err=%d", bz2err);

	stream.read = bz2_read;
	stream.opaque = bz2;
    /**
     * 增量算法
     */
	if (bspatch(old, oldsize, new, newsize, &stream))
		errx(1, "bspatch");

	/**
	 * Clean up the bzip2 reads
	 * 关闭bzip2的读
	 */
	BZ2_bzReadClose(&bz2err, bz2);
	fclose(f);

	/**
	 * Write the new file
	 * 写新文件
	 */
	if(((fd=open(argv[2],O_CREAT|O_TRUNC|O_WRONLY,sb.st_mode))<0) ||
		(write(fd,new,newsize)!=newsize) || (close(fd)==-1))
		err(1,"%s",argv[2]);

	free(new);
	free(old);

	return 0;
}
```

同样是修改源码中的mian函数。

调用增量文件


```java
int result = DiffPatchUtil.patch(oldpath,patchNewPath,patchPath);
```

## 五、测试验证

我们使用了ipc021501.apk和ipc030601.apk两个安装包进行测试


```java
		String path = Environment.getExternalStorageDirectory().getPath();
        final String oldpath = path + "/新文件夹/" + "ipc021501.apk" ;
        final String newpath = path + "/新文件夹/" + "ipc030601.apk";
        final String patchPath = path + "/新文件夹/" + "ipc.patch";
        final String patchNewPath = path + "/新文件夹/" + "ipc030602.apk";
```

依次调用

DiffPatchUtil.diff和DiffPatchUtil.patch

```java
  Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                int result = DiffPatchUtil.diff(oldpath,newpath,patchPath);
                Log.e("zza",result+"");
            }
        });
        thread.start();
```

```java
  Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                int result = DiffPatchUtil.patch(oldpath,patchNewPath,patchPath);
                Log.e("zza",result+"");
            }
        });
        thread.start();
```

生成文件ipc.patch和ipc030602.apk

如何对比ipc030601.apk和ipc030602.apk是否一致呢，我们可以使用MD5

![apk差分md5.png](https://github.com/zhangzhian/Bsdiff_Bspatch/blob/master/art/apk%E5%B7%AE%E5%88%86md5.png?raw=true)

可以看到两个文件的md5至一致。我们可以认为两个apk包完全一样。

## 六、总结

如果只是单纯的要使用该功能，可以直接将生成的so文件拷入，直接loadLibrary使用即可。

[文本源码](https://github.com/zhangzhian/Bsdiff_Bspatch)

[生成的so库](https://github.com/zhangzhian/Bsdiff_Bspatch/tree/master/%E7%94%9F%E6%88%90%E7%9A%84so%E5%BA%93)
