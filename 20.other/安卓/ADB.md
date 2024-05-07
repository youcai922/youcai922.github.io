#### 什么是ADB

ADB 全称为 Android Debug Bridge，起到调试桥的作用，是一个**客户端-服务器端程序**。其中客户端是用来操作的电脑，服务端是 Android 设备。



#### 下载和安装

下载地址

```
Windows版本：https://dl.google.com/android/repository/platform-tools-latest-windows.zip
Mac版本：https://dl.google.com/android/repository/platform-tools-latest-windows.zip
Linux版本：https://dl.google.com/android/repository/platform-tools-latest-linux.zip
```

解压后需要添加环境变量

path添加F:\software\adb\platform-tools（目录下有adb.exe）

cmd输入adb version，输出adb版本信息



#### 常用命令

启用adb服务：	adb start-server

关闭adb服务：	adb kill-server

链接模拟器：		adb.exe connect 127.0.0.1:62001

```
adb -d：如果同时连了usb，又开了模拟器，连接当前唯一通过usb连接的安卓设备
adb -e shell：指定当前连接此电脑的唯一的一个模拟器
adb -s <设备号> shell：当电脑插多台手机或模拟器时，指定一个设备号进行连接
```

输出当前连接设备：	adb devices



截图：	adb shell screencap -p /sdcard/screenshot.png

拉取截图到指定目录：	adb pull /sdcard/screenshot.png  E:/work

adb shell pm list packages：列出当前设备/手机，所有的包名
adb shell pm list packages -f：显示包和包相关联的文件(安装路径)
adb shell pm list packages -d：显示禁用的包名
adb shell pm list packages -e：显示当前启用的包名
adb shell pm list packages -s：显示系统应用包名
adb shell pm list packages -3：显示已安装第三方的包名
adb shell pm list packages xxxx：加需要过滤的包名，如：xxx = taobao
adb install <文件路径\apk>：将本地的apk软件安装到设备(手机)上。如手机外部安装需要密码，记得手机输入密码。
adb install -r <文件路径\apk>：覆盖安装

adb install -d <文件路径\apk>：允许降级覆盖安装
adb install -g <文件路径\apk>：授权/获取权限，安装软件时把所有权限都打开
adb uninstall <包名>：卸载该软件/app。
注意：安装时安装的是apk，卸载时是包名，可以通过 adb shell pm list packages 查看需要卸载的包名。

adb get-serialno：获取设备的序列号（设备号）
adb shell wm size：获取设备屏幕分辨率
adb shell screencap -p /sdcard/mms.png：屏幕截图
adb shell screencap -p /sdcard/screenshot.png：屏幕截图
adb pull /sdcard/mms.png <存放的路径>：将截图导出到本地
adb pull /sdcard/screenshot.png <存放的路径>：将截图导出到本地
adb shell cat /proc/meminfo：获取手机内存信息
adb shell df：获取手机存储信息
adb shell screenrecord <存放路径/xxx.mp4>：录屏，命名以.mp4结尾
adb shell screenrecord --time-limit 10 <存放路径/xxx.mp4>：录屏时间为10秒