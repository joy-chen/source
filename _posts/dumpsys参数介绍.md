---
title: dumpsys参数介绍
date: 2016-12-21 17:38:21
tags:
---

Author JOY.
<!-- excerpt -->

dumpsys -l
```
列出所有服务名
```
dumpsys gfxinfo [pid]   
dumpsys meminfo [pid]   
dumpsys input   
dumpsys netstats detail   
dumpsys procstats --hours 3   
`系统内存使用状况，统计最近3小时的 minPSS-avgPSS-maxPSS/minUSS-avgUSS-maxUSS`
dumpsys batterystats --checkin
