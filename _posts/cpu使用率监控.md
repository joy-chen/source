---
title: CPU使用率监控
date: 2016-10-19 16:49:27
tags:
---

核心类：ProcessCpuTracker
<!-- more -->
{% tabbed_codeblock 获取进程cpu使用率 %}
<!-- tab java -->
  mProcessCpuTracker.update();
  int num = mProcessCpuTracker.countWorkingStats();
  for (int i = 0; i < num; i++) {
    ProcessCpuTracker.Stats st = mProcessCpuTracker.getWorkingStats(i);
    long total = (st.rel_uptime + 5) / 10;
    if (total == 0) total = 1;
    long valid = st.rel_utime +  st.rel_stime;
    long thousands = (valid * 1000) / total;
    long hundreds = thousands / 100;
  }
<!-- endtab -->
{% endtabbed_codeblock %}
