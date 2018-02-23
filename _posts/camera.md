---
title: Camera调试相关总结
date: 2017-12-21 18:49:21
tags:
---

Author JOY.
<!-- excerpt -->

1.获取设备的自然方向？
public int getDeviceDefaultOrientation() {
    WindowManager windowManager =  (WindowManager) getSystemService(Context.WINDOW_SERVICE);

    Configuration config = getResources().getConfiguration();
    int rotation = windowManager.getDefaultDisplay().getRotation();

    if ( ((rotation == Surface.ROTATION_0 || rotation == Surface.ROTATION_180) &&
            config.orientation == Configuration.ORIENTATION_LANDSCAPE)
        || ((rotation == Surface.ROTATION_90 || rotation == Surface.ROTATION_270) &&    
            config.orientation == Configuration.ORIENTATION_PORTRAIT)) {
      return Configuration.ORIENTATION_LANDSCAPE;
    } else {
      return Configuration.ORIENTATION_PORTRAIT;
    }
}

2.rk3288 摄像头显示改为前置？
cam_board.xml 里头有个front 和back配置
http://dev.t-firefly.com/forum.php?mod=viewthread&tid=13091&highlight=%C9%E3%CF%F1%CD%B7

3.rk3288 摄像头向左翻转90%显示如何修改?
http://dev.t-firefly.com/forum.php?mod=viewthread&tid=12667&highlight=%C9%E3%CF%F1%CD%B7

4.查看相机型号版本
getprop | grep sys_graphic

5.camera配置文件path
hardware/rk29/camera/Config

6.关键log点
CameraHal,
