---
title: Android中的异常处理机制
date: 2017-01-06 16:05:02
tags: Debug
---

Author JOY.
<!-- excerpt -->

Apk第一次启动时,AMS通过socket发出请求，zygote接收消息，处理进程创建请求。

```java

public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
        throws ZygoteInit.MethodAndArgsCaller {
    commonInit();
    nativeZygoteInit();
    applicationInit(targetSdkVersion, argv, classLoader);
}    

private static final void commonInit() {
    ......
    /* 线程默认异常处理handler */
    Thread.setDefaultUncaughtExceptionHandler(new UncaughtHandler());
    ......
}
```
zygote会对新的进程进行一些基础的初始化，包括了默认异常处理。

```java
/* 当应用内部抛出未被处理的异常而退出时，将会抓取当前log */
private static class UncaughtHandler implements Thread.UncaughtExceptionHandler {
    public void uncaughtException(Thread t, Throwable e) {
      // 弹出错误对话框，等待用户选择需要的操作
      ActivityManagerNative.getDefault().handleApplicationCrash(
              mApplicationObject, new ApplicationErrorReport.CrashInfo(e));

      // 确保进程完全退出
      Process.killProcess(Process.myPid());
      System.exit(10);
    }
}
```
当异常发生后，如果应用没有捕获，则会调用默认UncaughtHandler来处理：   
APK ---> Binder ---> AMS    
* handleApplicationCrash()   
* Error Dialog //弹出 error dialog
* wait         //等待用户选择
* exit         //程序完全退出
