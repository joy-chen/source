---
title: Android广播流程
date: 2017-01-09 15:41:38
tags:
---

介绍Android广播的处理流程
<!-- excerpt -->

```java
framework/base/core/java/android/app/ContextImpl.java
```

## 一、发送过程

### 1.1 ContextImpl#sendBroadcast
```java
// UID = system，直接调用sendBroadcast方法，会警告，应调用sendBroadcastAsUser   
warnIfCallingFromSystemProcess();

// 获取所查询URI的MIME类型，如果没有类型则返回null，这里应该是返回null   
String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());

// 调用ActivityManagerService#broadcastIntent方法
ActivityManagerNative.getDefault().broadcastIntent
```

### 1.2 ActivityManagerService#broadcastIntent
```java

```

## 二、注册过程

### 2.1 ActivityManagerService#registerReceiver
```java
 /*Keeps track of all IIntentReceivers that have been registered for    broadcasts.Hash keys are the receiver IBinder, hash value is a ReceiverList.*/
mRegisteredReceivers.put(receiver.asBinder(), rl);
BroadcastFilter bf = new BroadcastFilter(filter,  rl, callerPackage,  permission, callingUid, userId);
rl.add(bf);
mReceiverResolver.addFilter(bf);
queue.enqueueParallelBroadcastLocked(r);
queue.scheduleBroadcastsLocked();
```
