---
title: MTK之USB-OTG
date: 2018-03-02 14:12:10
tags:
---

Author JOY.
<!-- excerpt -->

### 调试准备
1.查看电路原理图，USB device功能正常的话，说明USB HUB部分没问题   
2.查看供电是否正常，是否需要配置GPIO口

### USB OTG 配置
kernel-3.10/arch/arm64/configs/tx6735_65c_xz_l1_defconfig
```
CONFIG_USB_MTK_OTG=y
CONFIG_USB_MTK_HDRC_HCD=y
```
kernel-3.10/tools/dct/
在dws中设定OTG VBUS对应的输出控制pin配置为GPIO模式，var name为GPIO_OTG_DRVVBUS_PIN。再设定USB ID默认模式为IDDIG，var name为GPIO_OTG_IDDIG_EINT_PIN(具体哪个引脚，以及是否需要配置其他引脚，需要参考电路原理图)

建立挂载目录
device/tangxun/tx6735_65c_xz_l1/init.project.rc
```
on init
    mkdir /mnt/media_rw/usbotg 0700 media_rw media_rw
    mkdir /storage/usbotg 0700 root root

    service fuse_usbotg /system/bin/sdcard -u 1023 -g 1023 -w 1023 -d /mnt/media_rw/usbotg /storage/usbotg
        class late_start
        disabled
```

device/mediatek/mt6735/fstab.mt6735
```
/devices/platform/mt_usb                auto      vfat      defaults        voldmanaged=usbotg:auto
```

device/tangxun/tx6735_65c_xz_l1/overlay/frameworks/base/core/res/res/xml/storage_list.xml
```
<storage android:mountPoint="/storage/usbotg"
     android:storageDescription="@string/storage_external_usb"
     android:removable="true" />  
```
