---
title: MTK_CAMERA
date: 2018-02-09 10:54:09
tags:
---

Author JOY.
<!-- excerpt -->

### Camera驱动的调试过程与方法总结
1、首先对照电路图，检查Camera的电路连接是否正确，接口是否一致；

2、用万用表量Camera的电源管脚，查看Camera的供电是否正常，确定是否需要我们在程序中进行电源控制；

3、查看Camera的Spec文档，检查PWDN和RESET的管脚触发是否正常，是否需要在程序中进行控制；

4、用示波器量Camera的MCLK管脚，看是否正确，如果MCLK正常，通常情况下PCLK也应该有波形；

5、在Camera的Datasheet中找出该设备的I2C地址，检查I2C地址配置是否正确；

6、查看I2C通信是否正常，是否能正常进行读写，用示波器量出I2C的SCL和SDA的波形是否正常，未通信时都为高电平，通信时SCL为I2C时钟信号，SDA为I2C数据通路信号；

7、检查Camera的初始化寄存器列表的配置是否正确。

### 驱动配置
CONFIG_CUSTOM_KERNEL_LCM

### isp驱动
vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/

### 摄像头驱动
kernel-3.10/drivers/misc/mediatek/imgsensor/src/mt6735/

### 驱动结构添加
kernel-3.10/drivers/misc/mediatek/imgsensor/src/mt6735/kd_sensorlist.c
驱动的注册：
```
module_init(CAMERA_HW_i2C_init);
static int __init CAMERA_HW_i2C_init(void)
{
  platform_device_register(&camerahw_platform_device);
  platform_driver_register(&g_stCAMERA_HW_Driver);
}

static int CAMERA_HW_probe(struct platform_device *pdev)
{
    return i2c_add_driver(&CAMERA_HW_i2c_driver);
}

static int CAMERA_HW_i2c_probe(struct i2c_client *client, const struct i2c_device_id *id)
{
    RegisterCAMERA_HWCharDrv();
    register_camera_sysfs();
}

inline static int RegisterCAMERA_HWCharDrv(void)
{
  cdev_init(g_pCAMERA_HW_CharDrv, &g_stCAMERA_HW_fops);
}

static const struct file_operations g_stCAMERA_HW_fops =
{
    .owner = THIS_MODULE,
    .open = CAMERA_HW_Open,
    .release = CAMERA_HW_Release,
    .unlocked_ioctl = CAMERA_HW_Ioctl,
#ifdef CONFIG_COMPAT
    .compat_ioctl = CAMERA_HW_Ioctl_Compat,
#endif
};
```

### 定义id和drv name
kernel-3.10/drivers/miscmediatek/custom/common/kernel/imgsensor/inc/kd_imgsensor.h

### hal层ＩＤ和driver　name的衔接
./vendor/mediatek/proprietary/custom/mt6735/hal/D1/imgsensor_src/sensorlist.cpp

### 时序配置
./drivers/misc/mediatek/mach/mt6735/tx6735_65c_xz_l1/camera/camera/kd_camera_hw.c
