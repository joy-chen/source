---
title: 'SEAndroid:为APK添加权限'
date: 2016-09-28 21:17:20
tags: SEAndroid
category: framework

---

背景：Apk为手机ROM内置apk。
需求：Apk需要读取/proc/pid/stat文件。
问题：Android4.4后引入了SeLinux模块，对资源的访问多了一层检查，apk默认情况下是读取不到/proc/pid/stat内容的。
<!-- more -->
解决方案：根据SeLinux规则，为Apk创建权限te文件。te默认文件路径：/source_root/external/sepolicy。一般厂家可以自己定制一套，目录：/source_root/device/factory/sepolicy/。我们选择厂家目录，新建apk.te文件：

```
type xxx_app, domain, mlstrustedsubject;app_domain(monitor_app)allow monitor_app xxx_service:service_manager xxx;r_dir_file(monitor_app, domain)
```
然后在device/factory/sepolicy/common/seapp_contexts下添加：

```
user=system seinfo=platform name=com.letv.android.monitor domain=monitor_app type=app_data_file

```
一般这时候打开apk是会报错：{% hl_text danger %}avc : denied{% endhl_text %}
不用慌，根据提示，缺什么补什么，按下面格式添加即可:

```
allow monitor_app xxx_service:service_manager xxx;

```

另外请注意，很多时候se权限不够时，并不会输出log，这时候：
1.关闭selinux，看守否是se的问题  
2.如果是，找到所需se权限的context:
(file_contexts  property_contexts  seapp_contexts service_contexts)  
3.在对应te文件里添加找到的权限

{% hl_text danger %}如何查看selog呢？{% endhl_text %}
```
logcat |grep avc
dmsg |grep xxx
```
