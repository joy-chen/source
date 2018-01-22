---
title: Android FgThread
date: 2016-09-13 11:22:02
tags: AFS
category: [framework]
description: test
clearReading: true
---

FgThread继承ServiceThread
<!-- more -->
![img](https://github.com/joy-chen/joy-chen.github.io/blob/master/pic/bg/BgThread.png?raw=true)

FgThread作为单例存在系统中：
```java
private FgThread() {
  super("android.fg",android.os.Process.THREAD_PRIORITY_DEFAULT, true);
}
```
framework中很多系统服务会利用这个Thread,做耗时工作.
由于FgThread作为单例形式存在于system_server进程里，所以FgThread不能出现阻塞等异常，由Watchdog来监听：
```java
private Watchdog() {
  super("watchdog");
  mMonitorChecker = new HandlerChecker(FgThread.getHandler(),
    "foreground thread", DEFAULT_TIMEOUT);
  mHandlerCheckers.add(mMonitorChecker);
  ...
}
```
FgThread运行异常，会导致SWT，系统重启。
