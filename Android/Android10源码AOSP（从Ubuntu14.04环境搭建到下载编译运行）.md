# Android10源码AOSP（从Ubuntu14.04环境搭建到下载编译运行）

## Ubuntu14.04环境搭建

电脑为笔记本，CPU i7-10750H，16G内存

虚拟机使用VMware Workstation 15 Pro

操作系统为Ubuntu 64 位 14.04

AOSP比较大，安装虚拟机的时候预留250G+的空间，8G内存。具体的安装请自行进行。

根据你要编译的系统版本，选择相应的 Ubuntu版本：

- Android 6.0 (Marshmallow) – AOSP master：Ubuntu 14.04 (Trusty)
- Android 2.3.x (Gingerbread) – Android 5.x (Lollipop)：Ubuntu 12.04 (Precise)
- Android 1.5 (Cupcake) – Android 2.2.x (Froyo)：Ubuntu 10.04 (Lucid)

### Ubuntu 环境准备

- 设置root密码：sudo passwd

- 安装VMware Tools，方便与window交互
  sudo apt-get install open-vm-tools-desktop -y
  然后重启
  sudo reboot

- 替换国内源，见“镜像替换国内阿里”，apt-get intstall 会快很多
  重启，然后
  sudo apt-get update

- 安装编译源码时候的第三方库（14.04）
  sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip

  > 注意：如需使用 SELinux 工具进行政策分析cank，您还需要安装 python-networkx 软件包。
  > 注意：如果您使用的是 LDAP 并且希望运行 ART 主机测试，还需要安装 libnss-sss:i386 软件包。

- 安装adb sudo apt-get install android-tools-adb

- python 安装 (推荐采用3.6.8，使用3.7.X的版本存在import ssl的错误，理论上3.6+即可)

  参考“python3.6.8 安装”

- 分区工具，可以给虚拟机多分配些内存。现在系统安装号默认是12G的物理内存加8G虚拟内存。在编译过程中发现内存不够用（出现堆溢出，内存溢出），增加了6G虚拟内存，总共12G+14G内存。（推测和make -j8相关，如果内存不够的话j后面的数字小些）
  sudo apt-get install gparted

### 镜像替换国内阿里

用你熟悉的编辑器打开：

/etc/apt/sources.list

例如：sudo gedit /etc/apt/sources.list

替换默认的

http://archive.ubuntu.com/

为

https://mirrors.aliyun.com/

以Ubuntu 14.04.5 LTS为例，最后的效果如下：

```
deb https://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb https://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

## Not recommended
# deb https://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiv
```

### Python3.8.3 安装

安装SSL：

```
apt-get install openssl
apt-get install libssl-dev
```

安装Python3.6

```
wget https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tgz
tar -zxvf Python-3.8.3.tgz
cd Python-3.8.3
```

修改Moudles/Setup (该目录在Python-3.6.8目录下)，把以下行的注释去掉

```
SSL=/usr/local/ssl
_ssl _ssl.c \
        -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
        -L$(SSL)/lib64 -lssl -lcrypto
```

编译安装

```
./configure
make && make install
```

切换版本，使用该方式比较简单：

查看python版本的优先级（初次没有，报错的话正常）

```
sudo update-alternatives --config python
```

以通过update-alternatives来设置默认python版本, 最后的参数1，2是优先级，数字越大优先级越高，比如经过如下设置后，在终端输入python，显示的就是3.6的版本了。

```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.4 2
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
sudo update-alternatives --install /usr/bin/python python /usr/local/bin/python3.8 3
```

> 注意：自行编译安装的Python是在/usr/local/bin/目录下

## Android源码下载更新

Android 源代码树位于由 Google 托管的 Git 代码库中。Git 代码库中包含 Android 源代码的元数据，其中包括与对源代码进行的更改以及更改日期相关的元数据。下面介绍了如何下载特定 Android 代码源代码树。

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

### 使用每月更新的初始化包（推荐使用该方法）

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
> （强烈多同步几次，确保同步没有错误。在第一次同步完成后再进行一次同步，如果第一次没问题的话速度会很快的）
>
> 也可以写个脚本，自动执行
>
> ``` shell
> #!/bin/bash 
> repo sync -j4
> while [ $? = 1 ]; do 
> echo "================sync failed, re-sync again =====" 
> sleep 3 
> repo sync
> 	done
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
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-10.0.0_r10
```

默认是master，是android11版，我选择了Android10。我使用Android10

查看版本：`cd .repo/manifests && git branch -a`

## Android 编译

### 设置文件描述符限制

先用`$ ulimit `查看，如果是无限制就不需要修改

可能默认限制的同时打开的文件数量很少，不能满足编译过程中的高并发需要，因此需要在shell中运行命令：

```shell
$ ulimit -S -n 2048
```

### 避免OutOfMemoryError（内存足够的话不会出现）

Android10之前的解决方案：OutOfMemoryError: Java heap space解决方案

```shell
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"
```

Android10以后：

```shell
export _JAVA_OPTIONS="-Xmx4096m"
```

或者增加虚拟机内存或是swp空间

### 环境设置

在源码根目录下调用下面的命令：

```shell
$ source build/envsetup.sh
```

### 选择设备

在命令行输入下面的命令选择打算编译的源码类型

```shell
Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_blueline-userdebug
     4. aosp_bonito-userdebug
     5. aosp_car_arm-userdebug
     6. aosp_car_arm64-userdebug
     7. aosp_car_x86-userdebug
     8. aosp_car_x86_64-userdebug
     9. aosp_cf_arm64_phone-userdebug
     10. aosp_cf_x86_64_phone-userdebug
     11. aosp_cf_x86_auto-userdebug
     12. aosp_cf_x86_phone-userdebug
     13. aosp_cf_x86_tv-userdebug
     14. aosp_crosshatch-userdebug
     15. aosp_marlin-userdebug
     16. aosp_sailfish-userdebug
     17. aosp_sargo-userdebug
     18. aosp_taimen-userdebug
     19. aosp_walleye-userdebug
     20. aosp_walleye_test-userdebug
     21. aosp_x86-eng
     22. aosp_x86_64-eng
     23. beagle_x15-userdebug
     24. fuchsia_arm64-eng
     25. fuchsia_x86_64-eng
     26. hikey-userdebug
     27. hikey64_only-userdebug
     28. hikey960-userdebug
     29. hikey960_tv-userdebug
     30. hikey_tv-userdebug
     31. m_e_arm-userdebug
     32. mini_emulator_arm64-userdebug
     33. mini_emulator_x86-userdebug
     34. mini_emulator_x86_64-userdebug
     35. poplar-eng
     36. poplar-user
     37. poplar-userdebug
     38. qemu_trusty_arm64-userdebug
     39. uml-userdebug

Which would you like? [aosp_arm-eng] 21
```

根据后缀可以判断出使用的场景如下：

|   类型    |                  用途                  |
| :-------: | :------------------------------------: |
|   user    |          权限少，用于刷机使用          |
| userdebug | 和“user”类似，但可以root，并且可以调试 |
|    eng    |   具有开发配置，并且有额外的调试工具   |

根据需要选择对应的类型，比如我选择“21”。（推荐采用x86的，模拟器速度快一些）

```shell
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=10
TARGET_PRODUCT=aosp_x86
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_ARCH=x86
TARGET_ARCH_VARIANT=x86
HOST_ARCH=x86_64
HOST_2ND_ARCH=x86
HOST_OS=linux
HOST_OS_EXTRA=Linux-4.4.0-148-generic-x86_64-Ubuntu-14.04.6-LTS
HOST_CROSS_OS=windows
HOST_CROSS_ARCH=x86
HOST_CROSS_2ND_ARCH=x86_64
HOST_BUILD_TYPE=release
BUILD_ID=QP1A.191105.003
OUT_DIR=out
============================================
```

### 开始编译

清理msk

```shell
make clobber
```

为了加快编译的速度，最好并发来编译（并发数不建议太高，可能会造成内存不足，磁盘不足等问题）

```shell
make -j8
```

编译结束以后，会显示下面的日志：

```shell
#### build completed successfully (02:00:35 (hh:mm:ss)) ####
```

错误FAILED: system-qemu.img

```
============================================
wildcard(out/target/product/generic_x86/clean_steps.mk) was changed, regenerating...
[ 99% 11311/11312] Create system-qemu.img now
FAILED: out/target/product/generic_x86/system-qemu.img
/bin/bash -c "(export SGDISK=out/host/linux-x86/bin/sgdisk SIMG2IMG=out/host/linux-x86/bin/simg2img;      device/generic/goldfish/tools/mk_combined_img.py -i out/target/product/generic_x86/system-qemu-config.txt -o out/target/product/generic_x86/system-qemu.img)"
  File "device/generic/goldfish/tools/mk_combined_img.py", line 48
    print "'%s' cannot be converted to int" % (line[2])
          ^
SyntaxError: invalid syntax

```

若出现该错误，切换python2.7：` sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 10`

### 启动模拟器

```shell
emulator
```

KVM错误

```
emulator: ERROR: x86 emulation currently requires hardware acceleration!
Please ensure KVM is properly installed and usable.

CPU acceleration status: KVM is not installed on this machine (/dev/kvm is missing).
```

```
egrep -c '(vmx|svm)' /proc/cpuinfo
```

先使用该指令查看是否支持虚拟化，如果不支持的话在虚拟机的CPU选项中打开即可。

```shell
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
sudo adduser `id -un` libvirtd
sudo adduser `id -un` kvm

#检查是否安装成功
sudo kvm-ok
```

模拟器黑屏

```
emulator -partition-size 4096 -kernel ./prebuilts/qemu-kernel/x86/4.9/kernel-qemu2
```

通过使用kernel-qemu-armv7内核 解决模拟器等待黑屏问题.而-partition-size 4096则是解决警告: system partion siez adjusted to match image file (3083 MB > 800 MB)

### 模拟器启动后崩溃

```
VMware: vmw_ioctl_command error Invalid argument.
Aborted (core dumped)
```

关闭硬件加速

```
export SVGA_VGPU10=0
```

```
echo "export SVGA_VGPU10=0" >> ~/.bashrc
source ~/.bashrc
```



## Android Studio查看源码

### 编译源码idegen模块

```undefined
mmm development/tools/idegen/
```

这行命令的意思是编译idegen这个模块项目，然后生成idegen.jar文件。

编译结束以后，会显示下面的日志：

```bash
[100% 11109/11109] Install: out/host/linux-x86/framework/idegen.jar

#### build completed successfully (01:26 (mm:ss)) ####
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
Read excludes: 16ms
Traversed tree: 61259ms
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

OK，至此我们就完成了下载AOSP并编译导入Android Studio的完整过程。



参考：

1.https://source.android.google.cn/setup/start

2.https://blog.csdn.net/baidu_32237719/article/details/105715371

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)

