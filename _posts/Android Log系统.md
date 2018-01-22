---
title: Log系统
date: 2016-09-30 15:22:04
tags:
metaAlignment: center
---

QC打开log入口：\*#\*#8888#\*#\*  
MTK打开log入口：\*#\*#3646633#\*#\*
<!-- excerpt -->
## Log分类：

* Mobile Log(也叫AP Log)

	```
涉及到应用层相关的log:ARN,FC
再细分包含：kernel,main,event,radio

	```
	+ events_log
	+ main_log
	+ sys_log
	+ radio_log
* bugreport
* dropbox
* traceview

	```
Debug.startMethodTracingSamplingEx

	```
* systrace  

	```
atrace -z gfx input view wm am sched freq idle
-b 10480 -t 15 -c --async_start  

	```
* dmesg
* last_kmsg : `/proc/last_kmsg`
* tombstones  

	```
产生异常信号的C/C++程序与debuggerd建立连接后，debuggerd将进程信息dump
到tombstone_XX文件中保存到/data/tombstone/文件夹下。可通过
查看tombstone_XX分析异常进程的堆栈信息。
在控制台中以命令debuggerd -b [<tid>]启动。如果加上-b参数，则由tid指定的进
程的信息将dump到控制台上，否则dump到tombstone文件中。控制台中运行
命令callstack/dumpstate，进程信息会写入这两个命令指定的文件中。
	```
* anr
* bootprof : /proc/bootprof
* properties
* coredump   
```
由linux支持，进程崩溃时记录存储堆栈空间，寄存器等相关内容，保留致命现场数据，便于分析查找根源。
```

#### MTK
MTK log 分类MD Log、Mobile Log、Network Log，红屏、FC、ANR时，会出现aee_exp log
* crash_log

## Log分析工具：
#### logcat
```
参数：
-c:清楚日志
-v:设置输出格式（time，process，tag）
-b:查看指定日至缓冲区(radio，events，main，all)

```

#### bugreport
记录dumpsys,dumpstat和logcat等信息，包括进程列表、内存信息、VM等，获取的内容相当丰富。

## 对Log分析，定位
* crash问题：通过搜索FATAL关键字，可以快速定位到错误点。
* anr&fc:搜索{% hl_text primary %}am_anr{% endhl_text %} in event log and {% hl_text primary %}ANR in{% endhl_text %} main log.  
	+ 定位到log后，如果cpu使用率高，表明在进行大量计算或死循环，如果cpu使用率不高，说明处于IOWait状态。
	+ 再结合data/anr/traces.txt分析

## Logs for different Feature Group
* Android AOSP  
```
App, Framework, Kernel
Wlan, BT, Camera, Audio, GPS, Sensor, TP, Fingerprints
```
* Platform BSP
```
Qualcomm: System Crash, Memory Dump  
Qualcomm + MTK: Media, Modem, Audio, TCP IP
```
## Logs for different platform - MTK and Qualcomm
* MTK:
```
	1. Generally for MTK platform, the Offline MTKLoger will be fully enough
	2. For some display issue, also need dumpsys and dumpstate
	   adb shell dumpsys > DumpSys.txt & adb shell dumpstate > DumpState.txt
```
* Qualcomm:
```
	1. The Offline log is less than needed generally, especially for Modem related feature.
	   (The Offline Log should only be used for Daily Use scenario, or AP + Kernel + TCP log capture)
	2. Modem related, GPS, Audio, and some others feature, Its’ better to take trace log via QXDM
	3. Everything MTK needed for more info (dumpstate and dumpsys) is also needed for Qualcomm
```
## Log获取方式
* 开机log  
```
adb shell dmesg > dmesg.text
```
* kernel log(kernel缓冲区的log)  
```
adb shell cat proc/kmsg > kmsg.txt
adb shell cat /proc/last_kmsg >/local/file/name
```
* bugreport

## Log保存路径
* /data/anr
{% alert info %}
Some trace files seem to get here (Dalvik writes stack traces here on ANR, i.e. "Application Not Responding" aka "Force-Close";
{% endalert %}
* /data/tombstones
{% alert info %}
It may hold several tombstone_nn files (with nn being a serial, increased with every new file). As tombstones are placed for the dead, it is done here for "processes died by accident" (i.e. crashed) -- and it is what is referred to as "core dumps" on Linux/Unix systems. However, not all apps create tombstones; this must be explicitly enabled by the developer  
{% endalert %}
* /data/system/dropbox
{% alert info %}
dropbox日志文件的命名有一定的规则，其前缀都是一个特定的tag（标签）,
tag由两部分组成，合起来是“进程类型_事件类型”。
processClass函数返回该进程的类型，包括`system_server`,`system_app`和`data_app`。  
eventType用于指定事件类型，目前也有3种类型：`crash`,`wtf`和`anr`
如SYSTEM_BOOT、SYSTEM_TOMBSTONE等，这些都是由BootReceiver在收到BOOT_COMPLETE广播后收集相关信息并传递给DBMS而生成的日志文件。
{% endalert %}

## Android Tracing – App, App FW – MTK + Qualcomm
+ Logcat, Kernel Log, Pull ANR, Tombstone, Dropbox out

+ ANR file is necessary log for “Application not responsible”
Force Close: FC logcat must Containing the keyword “fatal exception”

+ Most Feature (such as APK and display level) only need Logcat + Kernel Log

## logcat
+ logcat -v threadtime -d *:v  
+ logcat -b events -v threadtime -d *:v

## ptrace
ptrace 提供了一种父进程可以控制子进程运行，并可以检查和改变它的核心image。它主要用于实现断点调试。一个被跟踪的进程运行中，直到发生一个信号，则进程被中止，并且通知其父进程。在进程中止的状态下，进程的内存空间可以被读写。父进程还可以使子进程继续执行，并选择是否是否忽略引起中止的信号。

## Log分析
* 先确定错误发生时间，缩小log寻找范围
* 通过进程id来抽取log
{% alert info %}
Log的打印顺序并不完全对应程序的执行顺序
c++层和java层返回的线程id并不是一样的概念
{% endalert %}

#### native log分析
* 根据pid tid定位到进程id，然后根据id寻找上下文log
* 查看backtrace，用add2line -f -e path来找到调用栈

#### FC
* 常规FC，在Main LOG 关键字 "FATAL EXCEPTION"
* Native Crash，在Main LOG 关键字 "backtrace"

#### ANR
* 在Main LOG 中关键字“ANR IN”
* 在EventLog中对应的关键字是“am_anr”

```
ANR 主要原因有以下三种：
　　1. KeyDispatchTimeout(5 seconds) --主要类型按键或触摸事件在特定时间内无响应；
　　2. BroadcastTimeout(10 seconds)BroadcastReceiver在特定时间内无法处理完成；
　　3. ServiceTimeout(20 seconds) --小概率类型Service在特定的时间内无法处理完成。
```
#### REBOOT
Reboot：主要分Framwork层重启和底层重启。所有重启在Main LOG 中关键字"Entered the Android system server!"。
* Framework 层重启（快速重启，系统开机时间不变）：在Main/Sys.log 对应关键字“Watchdog killing system proces”/"system_server_crash"；
* 底层重启（系统开机时间重置）；在Main/Sys.log 对应关键字“Kernel panic”

### dumpsys
查看activity开关状况：dumpsys activity -d list

## 相关阅读
+ [Reading Bug Reports](https://source.android.com/source/read-bug-reports.html#anrs-deadlocks)
