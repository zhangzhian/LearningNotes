# 在Mac上编译FFMpeg并添加x264支持 

---
之前写过一篇[在Android Studio中使用cmake编译FFmpeg][1]，主要是在为了在android中使用FFmpeg进行视频的编解码，在使用过程中，在编码h264是发生错误，查找原因是没有添加x264支持。现在编译能够进行h264编译的FFmpeg。

##**前期准备**
1.可以在[x264官网][2]下载，也可以直接`git clone http://git.videolan.org/git/x264.git x264`，我采用第二种
2.[FFmpeg官网][3]下载3.3.6版本，我使用3.3.3和3.4.2和git最新版本均有一些问题。
3.x264编译前修改一下configure文件： 
找到所有的`libx264.so.$API`修改为`libx264-$API.so`（若不修改，生成的动态库为libx264.so.152，android无法识别），直接去掉`.$API`我这儿会报错，是因为生成个`libx264.so`链接文件，导致冲突
4.修改FFmpeg的configure，由于编译出来的动态库文件名的版本号在.so之后（例如“libavcodec.so.5.100.1”），而android不能识别，所以需要修改。在configure文件中找到下面几行代码：
```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'  
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)' 
```
替换为
```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'  
SLIB_INSTALL_LINKS='$(SLIBNAME)'  
```
![##](http://img.blog.csdn.net/20180314211440431?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
##**编写编译脚本**
1.x264的shell脚本：build_x264.sh
```
#!/bin/sh

NDK=/Users/zhangzhian/Documents/android-sdk-macosx/ndk-bundle
SYSROOT=$NDK/platforms/android-21/arch-arm
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64

function build_one
{
  ./configure \
--prefix=$PREFIX \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--sysroot=$SYSROOT \
--host=arm-linux \
--enable-pic \
--enable-shared \
--enable-static \
--disable-cli
make
make install

}
CPU=arm
PREFIX=/usr/local
build_one

```
前面的NDK，SYSROOT，TOOLCHAIN替换为自己的路径，PREFIX为输出的路径
注意\后面不能有空格
放到x264的目录下
2.ffmpeg的shell脚本：build_android.sh

```
#!/bin/sh

NDK=/Users/zhangzhian/Documents/android-sdk-macosx/ndk-bundle
SYSROOT=$NDK/platforms/android-21/arch-arm
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
function build_one
{
./configure \
--prefix=$PREFIX \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-avdevice \
--disable-doc \
--enable-gpl \
--enable-libx264 \
--enable-protocols \
--enable-muxer=mp4 \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=linux \
--arch=arm \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic -I/usr/local/include $ADDI_CFLAGS" \
--extra-ldflags="-L/usr/local/lib $ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}
CPU=arm
PREFIX=/usr/local
ADDI_CFLAGS="-marm"
build_one

```

前面的NDK，SYSROOT，TOOLCHAIN替换为自己的路径，PREFIX为输出的路径
--extra-cflags和--extra-ldflags中的-I/usr/local/include -L/usr/local/lib是x264的输出路径
放到FFmpeg的目录下
##**编译**
**1.x264**
移动到x264目录下
添加可执行权限：`sudo chmod +x build_x264.sh`
开始执行`./build_x264.sh`
最后一步可能出现权限错误，`sudo make install`，然后输入密码，即可。
编译成功后切换到 ／usr／local目录下会看到include 和lib两个文件夹，为输出对应文件夹

**2.FFmpeg**
移动到FFmpeg目录下
添加可执行权限：`sudo chmod +x build_android.sh`
开始执行`./build_android.sh`
最后一步可能出现权限错误，`sudo make install`，然后输入密码，即可。
编译成功后切换到 ／usr／local目录中的include 和lib两个文件夹里会多出文件，为输出对应文件。


下图为**成功编译**的文件：
![编译成功](http://img.blog.csdn.net/20180314210900105?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

参考文件：
http://blog.csdn.net/qq_26093363/article/details/52645397

  [1]: https://blog.csdn.net/baidu_32237719/article/details/77675784
  [2]: http://www.videolan.org/developers/x264.html
  [3]: http://ffmpeg.org/download.html#releases