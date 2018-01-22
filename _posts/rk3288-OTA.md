---
title: rk3288_OTA
date: 2017-12-25 19:56:32
tags:
---

Author JOY.
<!-- excerpt -->

OTA整个功能代码分布：服务端，recovery，framework层，以及apk共同组成
  APK(vendor/rockchip/common/apps/RKUpdateService)
  RK只提供apk，没有源码。反编译，apk的主要功能是从服务器下载ota包到本地，
  然后写入参数提供给recovery，然后重启进入recovery。

系统升级分为OTA升级和固件包升级
OTA方式：   
1)写如下字段到/cache/recovery/last_flag
updating$path=/mnt/internal_sd/update.zip
2)写如下字段到/cache/recovery/command
--update_package=/mnt/internal_sd/update.zip
--locale=en_US
3)在adb shell中运行reboot recovery

### 固件包方式：
1)写如下字段到/cache/recovery/last_flag
updating$path=/mnt/internal_sd/update.img
2)写如下字段到/cache/recovery/command
--update_rkimage=/mnt/internal_sd/update.img
--locale=en_US
3)在adb shell中运行reboot recovery

### OTA升级失败排查:
http://blog.csdn.net/luzhenrong45/article/details/62042400

### dd命令：
  dd备份分区：
    df查看各分区使用
    mount查看挂载路径
    ls /dev/block／platform/xxx/ 查看各分区
    cat proc/partitions 查看各分区大小
  备份system分区：
  dd if=/dev/block/platform/ff0f0000.rksdmmc/by-name/system of=/sdcard/system.img
  挂载：mount -t ext4 system.img dest_path
  http://blog.csdn.net/u014134180/article/details/78120143

### recovery相关部分的代码路径
  ${code_root}/bootable/recovery
  ${code_root}/bootable/recovery/recovery.cpp
  ${code_root}/bootable/recovery/rkimage.cpp
  ${code_root}/frameworks/base/core/java/android/os/RecoverySystem.java

### 流程分析
The recovery tool communicates with the main system through /cache files.
/cache/recovery/command - INPUT - command line for tool, one arg per line
/cache/recovery/log - OUTPUT - combined log file from recovery run(s)
/cache/recovery/intent - OUTPUT - intent that was passed in

The arguments which may be supplied in the recovery.command file:   
 *   --send_intent=anystring - write the text out to recovery.intent
 *   --update_package=path - verify install an OTA package file
 *   --wipe_data - erase user data (and cache), then reboot
 *   --wipe_cache - wipe cache (but not user data), then reboot
 *   --set_encrypted_filesystem=on|off - enables / diasables encrypted fs
 *   --just_exit - do nothing; exit and reboot         
Arguments may also be supplied in the bootloader control block (BCB).

After completing, we remove /cache/recovery/command and reboot.

#### 应用层
app执行安装/重置/清楚缓存操作调用代码文件
  frameworks/base/core/java/android/os/RecoverySystem.java
    installPackage
    rebootWipeUserData
    rebootWipeCache

上面的所有操作都是往/cache/recovery/command文件中写入不同的命令，在进入recovery后（recovery.cpp）对command的关键字进行判断，执行相应的操作


recovery.cpp
  -main()
   ->install_package()
    install.cpp
      ->install_package()
        ->try_update_binary()
