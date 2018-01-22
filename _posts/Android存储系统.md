---
title: Android存储系统
date: 2017-01-11 09:54:36
tags:
---

Android存储系统
<!-- excerpt -->

```c
service installd /system/bin/installd
    class main
    socket installd stream 600 system system
```

## Log 分析
关键字搜索
* Storage manager

## 查看存储空间的方法
### 使用dump命令

adb shell dumpsys devicestoragemonitor   
Current DeviceStorageMonitor state:   
mFreeMem=375 MB mTotalMemory=12.55 GB   
mFreeMemAfterLastCacheClear=877 MB   
mLastReportedFreeMem=375 MB mLastReportedFreeMemTime=-1h11m4s666ms   
mLowMemFlag=false mMemFullFlag=false   
mIsBootImageOnDisk=true mClearSucceeded=false mClearingCache=false   
mMemLowThreshold=500 MB mMemFullThreshold=1.00 MB   
mMemCacheStartTrimThreshold=375 MB mMemCacheTrimToThreshold=750 MB   
mFreeMem较小时，比如为375 MB，证明手机内剩余存储空间较少。   
