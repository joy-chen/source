---
title: MTK之lcm架构
date: 2018-02-22 17:24:36
tags:
---

Author JOY.
<!-- excerpt -->

Linux kernel有一些总线，比如USB、I2C等。对于每一个总线都会有一些设备和驱动挂在上面。驱动服务于匹配的设备，使Linux正确的操作硬件设备。当一个设备或者驱动注册到特定的总线上的时候就会触发总线匹配函数，比如一个设备注册到了总线，所有的该总线的驱动都会被枚举，判断是不是可以服务于新添加的设备（一般通过name来匹配），反之亦然。
如果总线匹配成功，就会调用驱动的probe函数，检查指定的硬件确实存在，然后确定是否所需的资源都能够从系统申请。
事实上，设备或者驱动能够正确的合作，在probe之后，模块初始化顺序决定于probe的执行顺序，可以由BSP函数中注册设备的顺序控制。MT6572平台，L版本的BSP文件放在kernel/arch/arm/mach-mt6572/mt_devs.c,mt_board_init()函数控制着probe的顺序。
platform虚拟总线，关联在该总线的设备和驱动通过name来匹配。

### 源码路径
drivers/misc/mediatek/lcm/mt65xx_lcm_list.c
arch/arm64/configs/tx6735_65c_xz_l1_defconfig


arch/arm64/configs/tx6735_65c_xz_l1_defconfig
CUSTOM_KERNEL_LCM = xxx  //对应lcm目录驱动的子目录名
配置驱动

### drivers/misc/mediatek/mach/mt6735/mt_devs.c
```
__init int mt_board_init(void) {

}
```

### ./drivers/misc/mediatek/videox/mt6735/mtkfb.c
```
static struct fb_ops mtkfb_ops = {
    .owner          = THIS_MODULE,
    .fb_open        = mtkfb_open,
    .fb_release     = mtkfb_release,
    .fb_setcolreg   = mtkfb_setcolreg,
    .fb_pan_display = mtkfb_pan_display_proxy,
    .fb_fillrect    = cfb_fillrect,
    .fb_copyarea    = cfb_copyarea,
    .fb_imageblit   = cfb_imageblit,
    .fb_cursor      = mtkfb_soft_cursor,
    .fb_check_var   = mtkfb_check_var,
    .fb_set_par     = mtkfb_set_par,
    .fb_ioctl       = mtkfb_ioctl,
#ifdef CONFIG_COMPAT
	.fb_compat_ioctl = mtkfb_compat_ioctl,
#endif    
#ifdef CONFIG_DMA_SHARED_BUFFER
    .fb_dmabuf_export = mtkfb_dmabuf_export,
#endif
};

static struct platform_driver mtkfb_driver =
{
    .driver = {
        .name    = MTKFB_DRIVER,
#ifdef CONFIG_PM
        .pm     = &mtkfb_pm_ops,
#endif
        .bus     = &platform_bus_type,
        .probe   = mtkfb_probe,
        .remove  = mtkfb_remove,
        .suspend = mtkfb_suspend,
        .resume  = mtkfb_resume,
	.shutdown = mtkfb_shutdown,
	.of_match_table = mtkfb_of_ids,
    },
};
static int mtkfb_probe(struct device *dev) ｛
  primary_display_init(mtkfb_find_lcm_driver(), lcd_fps);
  mtkfb_fbinfo_init(fbi);
  mtkfb_register_sysfs(fbdev);
  register_framebuffer(fbi);
｝
mtk_fb_probe函数主要做的工作如下：
Called by LDM binding to probe andattach a new device.
 * Initialization sequence:
 *   1.allocate system fb_info structure
 *     select panel type according to machine type
 *   2.init LCD panel
 *   3.init LCD controller and LCD DMA
 *   4.init system fb_info structure
 *   5.init gfx DMA
 *   6.enable LCD panel
       start LCD frame transfer
 *   7.register system fb_info structure
```

### ./drivers/misc/mediatek/videox/mt6735/primary_display.c
```
int primary_display_init(char *lcm_name, unsigned int lcm_fps) {
  disp_lcm_probe( lcm_name, LCM_INTERFACE_NOTDEFINED);
  disp_lcm_init(pgc->plcm, 0);
}
```

### ./drivers/misc/mediatek/videox/mt6735/disp_lcm.c
```
disp_lcm_handle* disp_lcm_probe(char* plcm_name, LCM_INTERFACE_ID lcm_id)
{
  lcm_drv = lcm_driver_list[0];
}
```

### drivers/misc/mediatek/lcm/mt65xx_lcm_list.c
```
驱动列表
LCM_DRIVER* lcm_driver_list[] = {
  ...
}
```

### 驱动调试
* 白屏无显示   
检查电压是否正常，确定硬件是否有问题

* 黑色横条，颜色失真   
初始化时序问题

* 闪屏   
时序，pclk配置不对
