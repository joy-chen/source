---
title: Android_NDK开发之Android.mk
date: 2018-01-17 18:07:59
tags:
---

Author JOY.
<!-- excerpt -->

native代码目录结构：
* project   
  * jni   
    * Android.mk   
    * Application.mk   
  * libs   
    * armeabi   
    * armeabi-v7a   
    * xxx.jar   
  * res   
  * src   
  * Android.mk   
  * AndroidManifest.xml

添加三方so库：
project/Android.mk:
  LOCAL_PREBUILT_JNI_LIBS += xxx.so

project/jni/Application.mk
  APP_ABI：设置编译哪些平台库（armeabi armeabi-v7a x86)

project/jni/Android.mk
  LOCAL_LDFLAGS 链接库
  include $(BUILD_SHARED_LIBRARY) 编译成动态库

### 技巧
打印变量的值：   
$(warning "the value of LOCAL_PATH is$(LOCAL_PATH)")  

示例1:
* project   
  * lib   
    * armeabi   
      * xxx.so   
  * Android.mk   
  * xxx.apk   

Android.mk写法：
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)M

LOCAL_MODULE := xxx
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := xxx.apk
LOCAL_MODULE_CLASS := APPS
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_PREBUILT_JNI_LIBS:= \
lib/armeabi/xxx.so

include $(BUILD_PREBUILT)
```

## Android.mk
### LOCAL_EXPORT_C_INCLUDES    
定义确保了任何依赖这个预编译库的模块会自动在自己的 LOCAL_C_INCLUDES 变量中增加到这个预编译库的include目录的路径，从而能够找到其中的头文件。
