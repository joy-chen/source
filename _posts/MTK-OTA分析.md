---
title: MTK_OTA分析
date: 2017-12-28 20:44:24
tags:
---


Author JOY.
<!-- excerpt -->

升级一般通过运行升级包中的META-INF/com/google/android/update-script脚本来执行自定义升级，脚本中是一组recovery系统能识别的UI控制，文件系统操作命令，例如write_raw_image（写FLASH分区），copy_dir（复制目录）。

### SystemUpdate
MainEntry
  onStart()
    bindService(serviceIntent, mConnection, Context.BIND_AUTO_CREATE);
      onServiceConnected()
        queryPackagesInternal();
          mService.queryPackages();
            sQueryNewVersionThread.start();
              mHttpManager.queryNewVersion();
                checkNewVersion()


### 命令形式
```
echo
--update_package=/sdcard/update.zip
--local=en_US
to /cache/recovery/command

adb reboot recovery
```

### /cache/recovery/command：
recovery命令，由主系统写入。所有命令如下：

* --send_intent=anystring - write the text out to recovery.intent

* --update_package=root:path - verify install an OTA package file

* --wipe_data - erase user data (and cache), then reboot

* --wipe_cache - wipe cache (but not user data), then reboot

### 文件对应
update-binary -> bootable/recovery/updater   
patch -> bootable/recovery/applypatch

### 调试
#### /cache/recovery/last_log
该log记录了patch过程的信息，如果出现错误，很容易定位到具体问题点。

### 差分包
制作差分包基于：   
out/target/product/$(product_name)/obj/PACKAGING/target_files_intermediates/$(product_name)-target_files-eng.root.zip
脚本：   
build/tools/releasetools/ota_from_target_files.py
ota_from_target_files.py -i Base.zip New.zip update.zip
md5sum -b update.zip > md5sum
 zip package.zip update.zip md5sum

[参考链接](http://blog.csdn.net/luzhenrong45/article/details/60968458)
