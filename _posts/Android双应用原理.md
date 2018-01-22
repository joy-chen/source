---
title: Android双应用原理
date: 2017-01-10 11:42:27
tags:
---

Android中的多用户双应用原理
<!-- excerpt -->

## 托管配置
```
Android 5.0 提供了用于在企业环境内运行应用的新功能。如果用户已有个人帐户，则设备管理员可启动托管配置进程，向设备添加共存但独立的托管配置文件。与托管配置文件关联的应用与非托管应用一并出现在用户的启动器、最近使用的应用屏幕和通知中。

要启动托管配置进程，请通过 Intent 发送 ACTION_PROVISION_MANAGED_PROFILE。如果调用成功，系统会触发 onProfileProvisioningComplete() 回调。然后您就可以调用 setProfileEnabled() 来启用此托管配置文件。

默认情况下，托管配置文件中只启用了一小部分应用。您可以通过调用 enableSystemApp() 在托管配置文件中安装更多应用。

如果您要开发启动器应用，可以使用新增的 LauncherApps 类获取可为当前用户启动的 Activity 以及任何关联托管配置文件的列表。您的启动器可通过向可绘制图标追加工作徽章，以醒目方式显示托管应用。要检索带徽章的图标，请调用 getUserBadgedIcon()。

要查看如何使用新功能，请参阅此版本中的 BasicManagedProfile 实现示例。
```
