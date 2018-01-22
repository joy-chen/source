---
title: Android便捷命令
date: 2017-03-07 16:20:18
tags:
---

Author JOY.
<!-- excerpt -->
## pm
  pm list package列出安装再设备上的应用：
  * 不带任何选项：列出所有的应用的包名：
    pm list package
  * -s:列出系统应用
    pm list package -s
  * -3:列出第三方应用
    pm list package -3
  * -f:列出应用包名及对应的apk名及存放位置
    pm list package -f
  * -i:列出应用包名及其安装来源
    pm list package -i
    例如：package:com.zhihu.android installer=com.xiaomi.market  
  * 参数组合使用，例如查找三方应用中知乎的包名、apk存放位置、安装来源：
    pm list package -f -3 -i zhihu
    ```
    package:/data/app/com.zhihu.android-1.apk=com.zhihu.android  installer=com.xiaomi.market
    ```

pm list permissions -g -d   
`列出系统所有危险权限`

pm path 列出对应包名的.apk位置：
* pm path com.tencent.mobileqq
  package:/data/app/com.tencent.mobileqq-1.apk

pm dump，后跟包名，列出指定应用的dump信息，里面有各种信息：
* pm dump com.tencent.mobileqq
```
Packages:
Package [com.tencent.mobileqq] (4397f810):
userId=10091 gids=[3003, 3002, 3001, 1028, 1015]
pkg=Package{43851660 com.tencent.mobileqq}
codePath=/data/app/com.tencent.mobileqq-1.apk
resourcePath=/data/app/com.tencent.mobileqq-1.apk
nativeLibraryPath=/data/app-lib/com.tencent.mobileqq-1
versionCode=242 targetSdk=9
versionName=5.6.0
applicationInfo=ApplicationInfo{43842cc8 com.tencent.mobileqq}
flags=[ HAS_CODE ALLOW_CLEAR_USER_DATA ]
dataDir=/data/data/com.tencent.mobileqq
supportsScreens=[small, medium, large, xlarge, resizeable, anyDensity]
usesOptionalLibraries:
com.google.android.media.effects
com.motorola.hardware.frontcamera
timeStamp=2015-05-13 14:04:24
firstInstallTime=2015-04-03 20:50:07
lastUpdateTime=2015-05-13 14:05:02
installerPackageName=com.xiaomi.market
signatures=PackageSignatures{4397f8d8 [43980488]}
permissionsFixed=true haveGids=true installStatus=1
pkgFlags=[ HAS_CODE ALLOW_CLEAR_USER_DATA ]
User 0:  installed=true blocked=false stopped=false notLaunched=false enabled=0
grantedPermissions:
android.permission.CHANGE_WIFI_MULTICAST_STATE
com.tencent.qav.permission.broadcast
com.tencent.photos.permission.DATA
com.tencent.wifisdk.permission.disconnect
```
pm install，安装应用。目标apk存放与PC端，用adb install安装。目标apk存放于Android设备上，用pm install安装             

pm uninstall，卸载应用，同adb uninstall,后面跟的参数都是应用的包名

pm clear，清除应用数据

## am
am start，启动一个Activity，以启动系统相机应用为例：   
* 启动相机：
  am start -n com.android.camera/.Camera
* 先停止目标应用，再启动：
  am start -S com.android.camera/.Camera
* 等待应用完成启动：
  am start -W com.android.camera/.Camera
  ```
  Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER]                           cmp=com.android.camera/.Camera }
     Status: ok
     Activity: com.android.camera/.Camera
     ThisTime: 500
     TotalTime: 500
     Complete
  ```
* 启动默认C页：
  am start -a android.intent.action.VIEW -d http://testerhome.com
* 启动拨号器拨打10086
  am start -a android.intent.action.CALL -d tel:10086
* am monitor，监控crash与ANR
* am force-stop，后跟包名，结束应用
* am startsevice，启动一个服务
* am broadcast，发送一个广播   

## monkey
monkey -p com.jared.performancetool -v 500   
`-p表示包名，-v表示反馈级别 500就是500个伪随机事件,若在压力测试中程序崩溃或者接收到任何失控异常，就会自动停止。`
