---
title: Android之输入系统
date: 2018-02-05 20:31:47
tags:
---

Author JOY.
<!-- excerpt -->

### android 子系统主要由以下几个模块组成：   
EventHub：事件的传入是从EventHub开始的，EventHub是事件的抽象结构，维护着系统设备的运行情况，并维护一个所有输入设备的文件描述符   
Input Reader: 负责从硬件获取输入，转换成事件（Event), 并分发给Input Dispatcher.      
Input Dispatcher: 将Input Reader传送过来的Events 分发给合适的窗口，并监控ANR。   
Input Manager Service：负责Input Reader 和 Input Dispatchor的创建，并提供Policy 用于Events的预处理。   
Window Manager Service：管理Input Manager 与 View（Window) 以及 ActivityManager 之间的通信。
View and Activity：接收按键并处理。   
ActivityManager Service：ANR 处理

### getevent
输入设备插拔或者使用时，数据会实时显示出来

### 查看外接输入设备
cat /proc/bus/input/devices
