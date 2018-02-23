---
title: MTK_CAMERA
date: 2018-02-09 10:54:09
tags:
---

Author JOY.
<!-- excerpt -->

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
