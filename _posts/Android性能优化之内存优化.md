---
title: Android性能优化之内存优化
date: 2018-03-02 14:45:24
tags:
---

Author JOY.
<!-- excerpt -->

### 内存泄漏   
对于Java来说，就是new出来的Object 放在Heap上无法被GC回收（内存中存在无法被回收的对象）

### 分析工具
* Android Studio 内存分析工具
* MAT 内存分析工具
* LeakCanary 分析

###  Android OOM
Android系统的每个进程都有一个最大内存限制，如果申请的内存资源超过这个限制，系统就会抛出OOM错误。

#### 避免方法
* 对内存的状态进行监听
```
@Override
public void onTrimMemory(int level) {
    super.onTrimMemory(level);
}
```

* 避免内存抖动   
避免在循环调用的场景下，生成对象

### 代码注意点
* 节制地使用Service   
建议使用JobScheduler，而尽量避免使用持久性的Service。还有建议使用IntentService，它会在处理完交代给它的任务之后尽快结束自己。

* 使用优化过的集合   
Android API当中提供了一些优化过后的数据集合工具类，如SparseArray，SparseBooleanArray，以及LongSparseArray等，使用这些API可以让我们的程序更加高效。

* 移除消耗内存的库、缩减Apk的大小   
查看Apk的大小，包括三方库和内嵌的资源，这些都会影响应用消耗的内存。通过减少冗余、非必须或大的组件、库、图片、资源、动画等，都可以改善应用的内存消耗。
