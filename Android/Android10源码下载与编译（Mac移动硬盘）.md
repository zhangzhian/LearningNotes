# Android10源码下载与编译（Mac移动硬盘）

## 创建区分大小写的磁盘映像

Mac系统默认磁盘，文件系统运行不区分大小写。Git 并不支持此类文件系统，而且此类文件系统会导致某些 Git 命令（例如 git status）的行为出现异常。因此，建议始终在区分大小写的文件系统中对 AOSP 源文件进行操作。

有两种方式可以创建磁盘映像，具体操作如下：

由于AOSP比较大，但是我们存放在移动硬盘上，更大的空间能够更好地满足未来的需求，所以预留200G+的空间。
可以通过 shell 使用以下命令创建磁盘映像：

```shell
hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 220g /Volumes/zza/aosp/android.dmg
```

在双击这个镜像，将其挂载。这样在Mac Finder中就可以看到我们刚刚的创建的镜像了。 

## Android源码下载更新

Android 源代码树位于由 Google 托管的 Git 代码库中。Git 代码库中包含 Android 源代码的元数据，其中包括与对源代码进行的更改以及更改日期相关的元数据。下面介绍了如何下载特定 Android 代码流水线的源代码树。

### 安装 Repo

Repo 是一款工具，可让您在 Android 环境中更轻松地使用 Git。要安装 Repo，请执行以下操作：

```shell
mkdir ~/bin # 创建文件夹
PATH=~/bin:$PATH #设置环境变量
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > ~/bin/repo #下载repro 到/bin/repo文件里
chmod a+x ~/bin/repo # 给repo 文件权限
```

### 更新 Repo

repo的运行过程中会尝试访问官方的git源更新自己，如果想使用tuna的镜像源进行更新，可以将如下内容复制到你的`~/.bashrc`里

```
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

并 `source ~/.bashrc`。

### 替换国内清华的镜像

用第三方工具打开repo文件，替换国内清华的镜像

将 `https://android.googlesource.com/` 全部使用 `https://aosp.tuna.tsinghua.edu.cn/` 代替即可。

由于使用 HTTPS 协议更安全，并且更便于灵活处理，所以强烈推荐使用 HTTPS 协议同步 AOSP 镜像。

**由于 AOSP 镜像造成CPU/内存负载过重，限制了并发数量，因此建议：**

1. sync的时候并发数不宜太高，否则会出现 503 错误，即`-j`后面的数字不能太大，建议选择4。
2. 请尽量选择流量较小时错峰同步。

### 使用每月更新的初始化包（使用该方法）

由于首次同步需要下载约 30GB 数据，过程中任何网络故障都可能造成同步失败，强烈建议使用初始化包进行初始化。

下载 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar (可以使用三方工具下载，支持断点续传)，下载完成后记得根据 checksum.txt 的内容校验一下。

由于所有代码都是从隐藏的 `.repo` 目录中 checkout 出来的，所以我们只保留了 `.repo` 目录，下载后解压 再 `repo sync` 一遍即可得到完整的目录。

使用方法如下:

```shell
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录 ，可以使用ls -al
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```

此后，每次只需运行 `repo sync` 即可保持同步。 

可以选择该命令同时发起四个并发请求，之所以选择4是因为清华的镜像的并发请求的限制的上限就是4个。

```shell
repo sync -j4
```

> 注意：出现奇奇怪怪的bug，可以重复执行一下，很多bug是由网络原因造成的。
>
> **一定要确保文件都下载成功，否则编译时会出现一些问题。**
>
> 也可以写个脚本，自动执行
>
> ``` shell
> #!/bin/bash 
> repo sync -j4
> while [ $? = 1 ]; do 
>    echo "================sync failed, re-sync again =====" 
>    sleep 3 
>    repo sync
>    	done
> ```

### 传统初始化方法（不推荐）

建立工作目录:

```shell
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
```

初始化仓库:

```shell
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```

**如果提示无法连接到 gerrit.googlesource.com，请参照“更新 Repo”。**

同步源码树（以后只需执行这条命令来同步）：

```shell
repo sync
```

### 切换版本

如果需要某个特定的 Android 版本：

```shell
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-10.0.0_r30
```

repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1

默认是master，是android11版，我选择了Android10。我使用Android11

## Android 编译

### 设置文件描述符限制

在macOS中，默认限制的同时打开的文件数量很少，不能满足编译过程中的高并发需要，因此需要在shell中运行命令：

```shell
$ ulimit -S -n 2048
```

### 环境设置

在源码根目录下调用下面的命令：

```shell
$ source build/envsetup.sh
```

### 选择设备

在命令行输入下面的命令选择打算编译的源码类型

```shell
$ lunch
	
	You're building on Darwin
	
	Lunch menu... pick a combo:
	     1. aosp_arm-eng
	     2. aosp_arm64-eng
	     3. aosp_mips-eng
	     4. aosp_mips64-eng
	     5. aosp_x86-eng
	     6. aosp_x86_64-eng
	     7. full_fugu-userdebug
	     8. aosp_fugu-userdebug
	     9. mini_emulator_arm64-userdebug
	     10. m_e_arm-userdebug
	     11. m_e_mips-userdebug
	     12. m_e_mips64-eng
	     13. mini_emulator_x86-userdebug
	     14. mini_emulator_x86_64-userdebug
	     15. aosp_dragon-userdebug
	     16. aosp_dragon-eng
	     17. aosp_marlin-userdebug
	     18. aosp_sailfish-userdebug
	     19. aosp_flounder-userdebug
	     20. aosp_angler-userdebug
	     21. aosp_bullhead-userdebug
	     22. hikey-userdebug
	     23. aosp_shamu-userdebug
	
	Which would you like? [aosp_arm-eng]
```

根据后缀可以判断出使用的场景如下：

|   类型    |                  用途                  |
| :-------: | :------------------------------------: |
|   user    |          权限少，用于刷机使用          |
| userdebug | 和“user”类似，但可以root，并且可以调试 |
|    eng    |   具有开发配置，并且有额外的调试工具   |

根据需要选择对应的类型，比如我选择arm “1”。（推荐采用x86的，模拟器速度快一些）

```shell
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=10
TARGET_PRODUCT=aosp_arm
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a-neon
TARGET_CPU_VARIANT=generic
HOST_ARCH=x86_64
HOST_OS=darwin
HOST_OS_EXTRA=Darwin-17.7.0-x86_64-10.13.6
HOST_BUILD_TYPE=release
BUILD_ID=QQ2A.200305.002
OUT_DIR=out
============================================
```

### 开始编译

为了加快编译的速度，最好并发来编译

```shell
$ make -j4
```

我的机器比较老，2线程，所以采用了-j4。

编译结束以后，会显示下面的日志：

```shell
#### build completed successfully (11:16:35 (hh:mm:ss)) ####
```

### 启动模拟器

```shell
emulator
```

## Android Studio查看源码

### 编译源码idegen模块

```undefined
mmm development/tools/idegen/
```

这行命令的意思是编译idegen这个模块项目，然后生成idegen.jar文件。

编译结束以后，会显示下面的日志：

```bash
#### build completed successfully (27:28 (mm:ss)) ####
```

mmm指令就是用来编译指定目录。通常来说，每个目录只包含一个模块。

```shell
  - croot: Changes directory to the top of the tree.
  - m: Makes from the top of the tree.
  - mm: Builds all of the modules in the current directory.
  - mmm: Builds all of the modules in the supplied directories.
  - cgrep: Greps on all local C/C++ files.
  - jgrep: Greps on all local Java files.
  - resgrep: Greps on all local res/*.xml files.
  - godir: Go to the directory containing a file.
```

### 生成AS配置文件

接着执行如下脚本：

```shell
development/tools/idegen/idegen.sh
```

这行命令的意思是在根目录生成对应的android.ipr、android.iml IEDA工程配置文件。

等待片刻得到类似如下信息说明OK：

```shell
Read excludes: 217ms
Traversed tree: 3605277ms
```

警告1：

```
emulator: WARNING: Couldn't find crash service executable /Volumes/android/aosp/prebuilts/android-emulator/darwin-x86_64/emulator64-crash-service
```

这个警告一般没影响，是模拟器崩溃时进行处理的程序，对使用者意义不是很大。

警告2：

```
emulator: WARNING: system partition size adjusted to match image file (3083 MB > 800 MB)
```

调整系统分区大小以匹配图像文件，执行命令：`emulator -partition-size 4096`。（可以不管）

推荐采用x86的，模拟器速度快一些。我这个机器启动arm架构的模拟器花费很长时间。

### 导入源码

启动Android Studio，然后选择打开一个已存在的Android Studio工程，选择源码根目录的`android.ipr`，经过的加载过程以后，Android 源码就已经成功的加载到了Android Studio中。

OK，至此我们就完成了在macOS上下载AOSP并编译导入Android Studio的完整过程。



参考：

1.https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/

2.https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/

3.https://blog.csdn.net/YuDBL/article/details/86129195

4.https://blog.csdn.net/YuDBL/article/details/86496890

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)

