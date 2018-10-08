
# Mac OS编译V8引擎源码

## 准备工作

### 1.设置代理

这里使用的shadowsocks，用于稳定的翻墙
 为命令行设置http代理
 
```shell
export http_proxy=http://127.0.0.1:1087;
export https_proxy=http://127.0.0.1:1087;
```

### 2.安装Xcode 和 Xcode Command Line Tools

可以在Mac App Store 安装

>**注意：** Xcode Command Line Tools可能存在大坑
>详见下面可能存在问题部分

### 3.安装 depot_tools

 1. git depot_tools 库

 
	```shell
	git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
	```
2. 添加到环境变量中
使用vim或其他工具打开文件


	```shell
	sudo vim ~/.bash_profile
	```
	
	添加depot_tools  的路径
	
	
	```shell
	export PATH=/<替换为你的depot_tools存放路径>/depot_tools:"$PATH" 
	```

	 保存环境变量
	 
	```shell
	source ~/.bash_profile
	```


## 下载V8源码


### 1.安装 depot_tools 构建系统的所有依赖
运行：

```shell
	gclient sync
```
### 2.获取 V8 源码（包含了所有分支和依赖）

```shell
	fetch v8
	cd v8
```

## 编译V8源码

### 1. v8gen 生成 ninja 构建文件：
```shell
	tools/dev/v8gen.py x64.release
```
### 2. 编译源码，生成可执行文件，目标系统 x64：
```shell
	ninja -C out.gn/x64.release
```
## 测试
使用py脚本测试编译后的文件是否有问题，测试可以不进行

```shell
	tools/run-tests.py --gn
```


## 可能存在问题
v8gen 生成 ninja 构建文件时出现如下错误：

```shell
$ tools/dev/v8gen.py x64.release -vv
################################################################################
/usr/bin/python -u tools/mb/mb.py gen -f infra/mb/mb_config.pyl -m developer_default -b x64.release out.gn/x64.release

  Writing """\
  is_debug = false
  target_cpu = "x64"
  """ to ~/projects/v8/out.gn/x64.release/args.gn.

  ~/projects/v8/buildtools/mac/gn gen out.gn/x64.release --check
    -> returned 1
  ERROR at //build/config/mac/mac_sdk.gni:61:5: Script returned non-zero exit code.
      exec_script("//build/mac/find_sdk.py", find_sdk_args, "list lines")
      ^----------
  Current dir: ~/projects/v8/out.gn/x64.release/
  Command: python -- ~/projects/v8/build/mac/find_sdk.py --print_sdk_path 10.10
  Returned 1.
  stderr:

  Traceback (most recent call last):
    File "~/projects/v8/build/mac/find_sdk.py", line 89, in <module>
      print main()
    File "~/projects/v8/build/mac/find_sdk.py", line 57, in main
      sdks = [re.findall('^MacOSX(10\.\d+)\.sdk$', s) for s in os.listdir(sdk_dir)]
  OSError: [Errno 2] No such file or directory: '/Library/Developer/CommandLineTools/Platforms/MacOSX.platform/Developer/SDKs'

  See //build/toolchain/mac/BUILD.gn:14:1: whence it was imported.
  import("//build/config/mac/mac_sdk.gni")
  ^--------------------------------------
  See //BUILD.gn:521:1: which caused the file to be included.
  action("js2c") {
  ^---------------
  GN gen failed: 1
Traceback (most recent call last):
  File "tools/dev/v8gen.py", line 304, in <module>
    sys.exit(gen.main())
  File "tools/dev/v8gen.py", line 298, in main
    return self._options.func()
  File "tools/dev/v8gen.py", line 166, in cmd_gen
    gn_outdir,
  File "tools/dev/v8gen.py", line 208, in _call_cmd
    stderr=subprocess.STDOUT,
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/subprocess.py", line 573, in check_output
    raise CalledProcessError(retcode, cmd, output=output)
subprocess.CalledProcessError: Command '['/usr/bin/python', '-u', 'tools/mb/mb.py', 'gen', '-f', 'infra/mb/mb_config.pyl', '-m', 'developer_default', '-b', 'x64.release', 'out.gn/x64.release']' returned non-zero exit status 1

```

解决方案：

1.查看打印结果
```shell
	xcode-select --print-path
```
如果打印输出是：`/Library/Developer/CommandLineTools`  
而不是`/Applications/Xcode.app/Contents/Developer`
则会报这个错误

2.修改Xcode Command Line Tools版本

修改Command Line Tools版本
我这儿是`sudo rm -rf /Library/Developer/CommandLineTools`删了原先版本，然后直接选择了Xcode 9.2这个版本

![在这里插入图片描述](https://github.com/zhangzhian/LearningNotes/blob/master/res/v8引擎编译.png?raw=true)