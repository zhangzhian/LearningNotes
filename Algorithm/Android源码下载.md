# Android源码下载（Mac移动硬盘）

## 创建区分大小写的磁盘映像

Mac系统默认磁盘，文件系统运行不区分大小写。Git 并不支持此类文件系统，而且此类文件系统会导致某些 Git 命令（例如 git status）的行为出现异常。因此，建议始终在区分大小写的文件系统中对 AOSP 源文件进行操作。

有两种方式可以创建磁盘映像，具体操作如下：

由于AOSP比较大，但是我们存放在移动硬盘上，更大的空间能够更好地满足未来的需求，所以预留200G+的空间。
可以通过 shell 使用以下命令创建磁盘映像：

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

### 使用每月更新的初始化包

由于首次同步需要下载约 30GB 数据，过程中任何网络故障都可能造成同步失败，强烈建议使用初始化包进行初始化。

下载 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar (可以使用三方工具下载)，下载完成后记得根据 checksum.txt 的内容校验一下。

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

### 传统初始化方法

建立工作目录:

```
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
```

初始化仓库:

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```

**如果提示无法连接到 gerrit.googlesource.com，请参照“更新 Repo”。**

如果需要某个特定的 Android 版本：

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
```

列表：https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds

同步源码树（以后只需执行这条命令来同步）：

```
repo sync
```

参考：

1.https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/

2.https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/

3.https://blog.csdn.net/YuDBL/article/details/86129195

**欢迎关注我的公众号，持续分析优质技术文章**
![欢迎关注我的公众号](https://img-blog.csdnimg.cn/20190906092641631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMyMjM3NzE5,size_16,color_FFFFFF,t_70)