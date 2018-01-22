---
title: Android Native Tombstone/Crash 分析
date: 2017-01-14 16:12:32
tags:
---

## 前言
APK运行异常时，会闪退，然后弹出对话框“xxx应用已停止运行。。。”，造成这种现象的原因有两种：
* 发生了未捕获的异常
* Native层运行异常

Tombstone 是 Android 用来记录 native 进程崩溃的 core dump 日志

## Tombstone 文件格式
```
*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
Build fingerprint: 'LeEco/X7_CN/le_x7:6.0/KGXCNFN5961511291D/1480366802:userdebug/test-keys'   
Revision: '0'   
ABI: 'arm64'   
pid: 2741, tid: 2832, name: L_UPDATE_ACTION  >>> com.google.android.gms <<<   
    signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0
    x0   0000000000000000  x1   0000000000000000  x2   0000000000000006     
    x3   000000000000736d  x4   0000000000000003  x5   0000007f6b9c8090   

backtrace:
    #00 pc 000000000001cff4  /system/lib64/libc.so (strlen+16)
    #01 pc 0000000000012e24  /system/lib64/libutils.so (_ZN7android7String8C2EPKc+24)
    #02 pc 0000000000116518  /system/lib64/libandroid_runtime.so
    #03 pc 00000000029a5438  /data/dalvik-cache/arm64/system@framework@boot.oat (offset 0x2696000)

stack:
         0000007f78421b60  0000000000000000
```
* 编译信息
* 问题进程－问题线程－进程名－错误信号
* 寄存器的值
* 当前堆栈
* 历史堆栈

### SIGSEGV
* 越界访问
