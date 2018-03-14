# 在Windows上编译Chromium（CEF3）并加入mp3/mp4的支持

---


现在因为工作需要，为了得到支持mp3、mp4的cef32和64位版本，需要编译cef3，本次编译版本是3239（6.0.3239.132）。

**一、编译条件**
1.可用的shadowsocks，用于稳定的翻墙
2.Win7或者更新的系统，必须64位，至少8GB的RAM，我采用win10 64位，16GRAM
3.比较新的VS,最近免费的社区版（编译不同版本要求不一样，具体看[Cef官网帮助][1]，我用的是VS2017），需要安装“C++桌面组件” 和 “MFC和ATL支持”,最好安装在默认路径，VS2017还需要特殊配置
4.Win10 SDK（官方10.0.15063，VS2017中有）
5.至少100G剩余空间（官方要求），NTFS文件系统，部分文件超过4G，部分资料显示最少60G，编译结束后发现远超60G

**二、准备工作**
1.设置系统区域为英语（美国）。（控制面板-区域-管理-更改系统区域设置-英语（美国）），设置完需要重启
2.创建工作目录，路径不能包含空格及特殊字符。例如e:\cef
3.下载[编译工具][2]包，解压至工作目录。例如e:\cef\depot_tools
4.下载[编译脚本][3]至工作目录。例如e:\cef\
5.在工作目录下创建源码目录。例如e:\cef\source
6.添加系统环境变量
```
　　　　 set CEF_USE_GN=1
        set GN_DEFINES=is_official_build=true
        set GYP_DEFINES=buildtype=Official
        set GYP_MSVS_VERSION=2017
        set CEF_ARCHIVE_FORMAT=tar.bz2
```
　　　　Path添加e:\cef\depot_tools，为避免与已安装的python或git冲突，写在path靠前位置。
　若环境变量设置后任有问题，在cmd使用set设置，例如：set DEPOT_TOOLS_WIN_TOOLCHAIN=0
```
    完整目录结构：
    e:/ 
        cef/
            automate-git.py
            depot_tools/
            source/
```  
 **三、设置代理**
1.shadowsocks设置为全局模式
2.打开具有管理员权限的cmd，输入如下指令
```
    >netsh
    netsh>winhttp
    netsh winhttp>
    netsh winhttp>
    netsh winhttp>set proxy 127.0.0.1:1080
```
   其中127.0.0.1:1080为代理的IP地址和端口号。 
   设置完成后，退出该cmd就可以了。该设置使固化在系统，重新启动之后，该设置依然有效。
3.设置http代理和Git代理
  在CMD中输入：
```
>set http_proxy=http://127.0.0.1:1080
>set https_proxy=http://127.0.0.1:1080
>set socks5_proxy=socks5://127.0.0.1:1080
```
为git设置代理 
a)使用http/https代理服务器
```
>git config --global http.proxy %http_proxy%
>git config --global https.proxy %https_proxy%
```
或者：b)使用socks5代理服务器
```
>git config --global http.proxy %socks5_proxy%
>git config --global https.proxy %socks5_proxy%
```
验证git代理 
设置完后，用下面命令看是否成功：
```
>git config --get http.proxy
>git config --get https.proxy
```
4.设置Boto代理
创建.boto文件
```
[Boto]
proxy = 127.0.0.1
proxy_port = 1080

```
在cmd中`set NO_AUTH_BOTO_CONFIG=E:\cef\.boto`，.boto位置任意
用来下gs://开头的文件，**千万127.0.0.1前不要加http://！！！**网上很多教程此次加了http://，导致无法使用。这个配置不好gs://开头的文件会下载失败，网上gs://替换为https://storage.googleapis.com/ 在浏览器下载的方案我只能下载前面一部分文件，中间有一步会清除目录下的文件，重新下载。

 **四、检出代码**
 
1.切换到工作目录e:\cef
2.使用命令下载源码

```
python automate-git.py --download-dir=e:\cef\source --branch=3239 --no-build --no-distrib --force-clean
```
其中--branch=3239是指定要下载的Cef版本； 
--no-build --no-distrib是只下载代码而不编译； 
--force-clean这个参数用于清理Chromium和Cef的一些检出信息，如果没有一次性下载成功而再次执行下载命令时，需要带上这个参数来清理一些信息，否则检出会失败（第一次下载时直接带上这个参数也可以）。
下载和编译只需要这个一个脚本就可以，脚本会自动下载depot_tools 、Chromium、Cef等源码。如果下载过程中出现错误，就再次执行这个命令直到下载完成。
网络调通以后不会有太多问题，下载时间和网络速度有关，我用了5个小时多些全部下载完成。

 **五、编译代码**

1.添加MP3、MP4支持 

source\chromium\src\third_party\ffmpeg\chromium\scripts\build_ffmpeg.py  
```
      configure_flags['Chrome'].extend([  
          '--enable-decoder=aac,h264,mp3',  
          '--enable-demuxer=aac,mp3,mov',  
          '--enable-parser=aac,h264,mpegaudio',  
      ]) 
```
改为
```
     configure_flags['Chrome'].extend([  
          '--enable-decoder=aac,h264,mp3,mpeg4,amrnb,amrwb,flv',  
          '--enable-demuxer=aac,mp3,mov,avi,amr,flv',  
          '--enable-parser=aac,h264,mpegaudio,mpeg4video,h263',  
      ])  
```
分别打开`e:\cef\source\chromium\src\third_party\ffmpeg\chromium\config\Chrome\win\ia32\config.h` 
和`e:\cef\source\chromium\src\third_party\ffmpeg\chromium\config\Chrome\win\x64\config.h`，在原有配置宏FFMPEG_CONFIGURATION里增加以下： 
```
    –enable-decoder=’rv10,rv20,rv30,rv40,cook,h263,h263i,mpeg4,msmpeg4v1,msmpeg4v2,msmpeg4v3,amrnb,amrwb,ac3,flv’ –enable-demuxer=’rm,mpegvideo,avi,avisynth,h263,aac,amr,ac3,flv,mpegts,mpegtsraw’ –enable-parser=’mpegvideo,rv30,rv40,h263,mpeg4video,ac3’
```
上面是通过查资料修改的，修改后编译能通过，没有报错，但是没有MP3、MP4支持。下面指令设置后再次编译便有了MP3、MP4支持，所以**上面的这些有没有用不敢确定**。
**下面的指令，很重要！！！**
    set GN_DEFINES=is_official_build=true proprietary_codecs=true ffmpeg_branding=Chrome

2.windows 构建指令设置
```
//为保险再设置一次
set CEF_USE_GN=1
set GN_DEFINES=is_official_build=true	
//set GN_DEFINES=is_official_build=true proprietary_codecs=true ffmpeg_branding=Chrome 添加MP3、MP4支持 使用此条指令
set GYP_DEFINES=buildtype=Official    
//set GYP_DEFINES=proprietary_codecs=1 ffmpeg_branding=Chrome 此条指令可能是以前版本用来添加MP3、MP4支持
set GYP_MSVS_VERSION=2017
set CEF_ARCHIVE_FORMAT=tar.bz2

set GYP_GENERATORS=ninja,msvs-ninja
set GN_ARGUMENTS=--ide=vs2017 --sln=cef --filters=//cef/*
//VS2017安装在默认目录，但任然需要下面设置，可能是由于VS2015和VS2017同时安装，路径根据自己的安装目录和版本确定
set WIN_CUSTOM_TOOLCHAIN=1
set CEF_VCVARS=none
set GYP_MSVS_OVERRIDE_PATH=C:\Program Files (x86)\Microsoft Visual Studio\2017\Community
set SDK_ROOT=C:\Program Files (x86)\Windows Kits\10
set INCLUDE=C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\ucrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\shared;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\include;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\atlmfc\include;%INCLUDE%
set PATH=C:\Program Files (x86)\Windows Kits\10\bin\10.0.15063.0\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\bin\HostX64\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\bin\HostX64\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Redist\MSVC\14.13.26020\x64\Microsoft.VC141.CRT;%PATH%
set LIB=C:\Program Files (x86)\Windows Kits\10\Lib\10.0.15063.0\um\x86;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.15063.0\ucrt\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\lib\x86;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\atlmfc\lib\x86;%LIB%
set VS_CRT_ROOT=C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\crt\src\vcruntim
```
[此处参考资料][4]
3.Bug修改
可以跳过，直接编译，等bug出现再查找相应解决方案。
编译过程中如果出现bug导致编译过程结束，一方面看cmd的输出，可能提供解决方案，还可以查看src\build-3239-release.log文件，搜索关键字FAILED来查找发生错误的文件
以下是我出现的问题：
错误
```
FAILED: obj/cef/chrome_elf_set/content_switches.obj
FAILED: obj/cef/chrome_elf_set/crash_keys.obj
```
解决方案
在cef/BUILD.gn文件中，查找 "chrome_elf_set"，在其子节点deps下添加"//media:media_features" 。
[参考资料][5]
4.编译代码
打开cmd切换到工作目录，然后输入命令来编译
```
python automate-git.py --download-dir=e:\cef\source --branch=3239 --no-update --no-debug-build --build-log-file --verbose-build --force-distrib --force-build
```
其中--no-update是让脚本不再更新代码，因为已经下载完毕了； 
--no-debug-build是只编译release版本，这样编译速度会快很多，--no-release-build可以只编译debug版本； 
--force-distrib --force-build用于强制编译cef代码； 
--build-log-file --verbose-build用于输出编译日志到e:\cef\source目录，名字为build-3239-release.log，编译发生错误，可以打开这个日志文件并通过搜索关键字FAILED来查找发生错误的文件； 
如果需要**64位版本**，则添加**--x64-build**参数且设置下列环境变量
```
set PATH=C:\Program Files (x86)\Windows Kits\10\bin\10.0.15063.0\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\bin\HostX64\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Redist\MSVC\14.13.26020\x64\Microsoft.VC141.CRT;%PATH%
set LIB=C:\Program Files (x86)\Windows Kits\10\Lib\10.0.15063.0\um\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.15063.0\ucrt\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\lib\x64;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\atlmfc\lib\x64;%LIB%
set INCLUDE=C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\ucrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.15063.0\shared;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\include;C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Tools\MSVC\14.13.26128\atlmfc\include;%INCLUDE%
```

编译第一次大概有6、7个小时，第二次以后2、3个小时

**六.编译完成**

 - 输出目录为source\chromium\src\out\Release_GN_x86 和 source\chromium\src\cef\binary_distrib，Release_GN_x86下有cefclient.exe可以测试，binary_distrib下有cef_binary_3.3239.1723.g071d1c1_windows32.tar.bz2
 - http://html5test.com可以测试结果
![http://html5test.com](http://img.blog.csdn.net/2018031016431035?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - chrome://version查看版本

![chrome://version](http://img.blog.csdn.net/20180310164322981?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


部分参考资料：
https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart
https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md#markdown-header-windows-configuration
https://github.com/cefsharp/cef-binary/wiki/Building-Cef-from-source
http://blog.csdn.net/spark_fountain/article/details/73867813?locationNum=9&fps=1
http://www.cnblogs.com/hezhixiong/p/5935143.html
http://blog.csdn.net/zhuhongshu/article/details/54193842
https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md
http://blog.csdn.net/cromma/article/details/51141573
https://mfweb.top/820.html
   
  [1]: https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md
  [2]: https://storage.googleapis.com/chrome-infra/depot_tools.zip
  [3]: https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py
  [4]: https://bitbucket.org/chromiumembedded/cef/wiki/AutomatedBuildSetup.md#markdown-header-windows-configuration
  [5]: https://bitbucket.org/chromiumembedded/cef/issues/2352