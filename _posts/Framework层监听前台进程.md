---
title: Framework层监听前台进程
date: 2016-09-27 16:13:50
tags:

---

ActivityStack$moveTaskToFrontLocked: 从Launcher或RecentTasks列表里(非第一次启动)面点击，会被调用。
ActivityStackSupervisor$realStartActivityLocked: Activity创建的时候会被调用。
ActivityStack$destroyActivityLocked: Activity销毁的时候会被调用(按返回键)。
ActivityStack$completeResumeLocked: Activity resume的时候被调用。
<!-- more -->

### 需要对前台进程的监听点：
1.应用第一次启动  
2.moveTaskToFrontLocked
3.completeResumeLocked