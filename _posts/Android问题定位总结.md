---
title: Android问题定位总结
date: 2018-01-04 16:27:05
tags:
---

Author JOY
<!-- excerpt -->

### Android进程终止和重启问题
原因是：虚拟机捕获了一些unchecked异常，如空指针异常等
在ddms或logcat或bugreport的log中搜索FATAL关键字，或者在/data/system/dropbox目录中找对应生成的crash字段的文件

### 异常相关的log目录  
/data/anr   
/data/system/dropbox   
/data/dontpanic/   
/data/tombstones/
