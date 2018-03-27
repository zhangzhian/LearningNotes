# [学习笔记] Android群英传：Android体系与系统架构和ADB常用命令


---
### Android体系与系统架构

![此处输入图片的描述][1]
1. Dalvik：在运行时编译 ART：安装时就进行编译
2. Activity、Service、Application都是继承自Context，在创建Activity、Service、Application的时候创建Context
3. android系统目录

 - /system/app/         -------系统app
 - /system/bin/         -------Linux自带组件
 - /system/build.prop   -------系统属性信息
 - /system/fonts/       -------字体
 - /system/framework/   -------系统核心文件、框架层
 - /system/lib/         -------共享（.so）库
 - /system/media/       -------系统提示音、铃声
 - /system/usr/         -------用户配置文件
 - /data/app/           -------安装的app
 - /data/data/          -------app的数据信息、文件信息、数据库
 - /data/system/        -------手机的系统信息
 - /data/misc/          -------大部分的wifi、vpn信息

### ADB 常用命令

 - 安装apk adb install -r 应用程序.apk
 - 向手机写入文件 adb push <local> <remote>
 - 从手机获取文件 adb pull <remote> <local>
 - 列出目标设备中已安装的应用程序包 adb shell pm list packages 
 - 删除应用 adb uninstall 完整包名
 - 查看系统盘符 adb shell df
 - 模拟按键输入 adb shell input keyevent 数字 数字为keyevent对应的code
 - 模拟滑动输入 adb shell input touchscreen <x1> <y1> <x2> <y2> 
    - adb shell input touchscreen swipe 18 665 18 350
 - 重新启动系统 adb reboot
 - 录制屏幕adb shell screenrecord 目录
 - 查询已连接的设备与模拟器 adb devices
 - 启动 adb server ：adb start-server
 - 停止 adb server ：adb kill-server
 - 获取 MAC 地址 adb shell  cat /sys/class/net/wlan0/address
 - 查看设备型号 adb shell getprop ro.product.model
 - 查看 Android 系统版本 adb shell getprop ro.build.version.release
 - 查看屏幕分辨率 adb shell wm size
 - 查看屏幕密度 adb shell wm density


  [1]: http://img.my.csdn.net/uploads/201501/23/1421980198_3393.jpg