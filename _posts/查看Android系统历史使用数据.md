---
title: 查看Android系统历史使用数据
date: 2017-02-03 20:31:20
tags:
---

Joy
<!-- excerpt -->

## 应用使用情况统计信息
```
现在可以利用新增的 android.app.usage API 访问 Android 设备上的应用使用历史记录。此 API 提供比已弃用的 getRecentTasks() 方法更为详细的使用信息。要使用此 API，您必须先在清单中声明 "android.permission.PACKAGE_USAGE_STATS" 权限。用户还必须通过 Settings > Security > Apps 为该应用启用访问使用情况的权限。

系统以应用为单位收集使用数据，按天、周、月和年汇总数据。系统保留这些数据的最长持续时间如下：

每日数据：7 天
每周数据：4 周
每月数据：6 个月
每年数据：2 年
系统会为每个应用记录以下数据：

最后一次使用应用的时间
在该时间间隔（以天、周、月或年为单位）内应用位于前台的总时长
一天之中当组件（以软件包和 Activity 名称标识）转入前台或后台时记录的时间戳
设备配置发生变化（如设备屏幕方向因旋转而发生变化）时记录的时间戳
```
