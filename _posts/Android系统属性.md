---
title: Android系统属性
date: 2016-10-24 21:06:21
tags:
---

Author JOY.
<!-- excerpt -->

前缀必须用system／core／init／property_service.c中定义的前缀，
进行系统属性设置的程序也必须有system或root权限,因为selinux会检测属性权限。
PS:不同前缀名权限不同。
如果apk不属于system或root用户，则需要在te文件里加入：
allow xxx system_prop:property_service { set };

ctl.start xxx 可以开启rc配置的服务
启动带参数的服务：ctl.start xxx:parms

获取服务的运行状态：
init.svc.<name>
   State of a named service ("stopped", "running", "restarting")
SystemProperties.get("init.svc.<name>");
