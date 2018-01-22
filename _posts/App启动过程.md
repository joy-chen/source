---
title: App启动过程
date: 2017-04-19 16:02:01
tags:
---

Author JOY.
<!-- excerpt -->

#### 从桌面到Activity的调用堆栈，因为这部分较简单，我们直接列出调用堆栈
->com.android.launcher3.Launcher#onClick   
&emsp;->Launcher#onClickAppShortcut   
&emsp;&emsp;->Launcher#startAppShortcutOrInfoActivity   
&emsp;&emsp;&emsp;->Launcher#startActivitySafely   
&emsp;&emsp;&emsp;&emsp;->Launcher#startActivity   
&emsp;&emsp;&emsp;&emsp;&emsp;->Activity#startActivity

#### 从Activity#startActivity，到启动进程和launch activity的调用堆栈   
`1.调用堆栈`      
`1-1.准备新的activity`  
->Activity#startActivity   
&emsp;->Activity#startActivityForResult  //此时mInstrumentation变量为源activity的，即launcher3   
&emsp;&emsp;->Instrumentation#execStartActivity   
&emsp;&emsp;&emsp;->ActivityManagerService#startActivity   
&emsp;&emsp;&emsp;&emsp;->ActivityManagerService#startActivityAsUser   
&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStackSupervisor#startActivityMayWait   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStackSupervisor#startActivityLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStackSupervisor#startActivityUncheckedLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStack#startActivityLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStackSupervisor#resumeTopActivitiesLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStack#resumeTopActivityLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStack#resumeTopActivityInnerLocked  

`1-2.暂停旧的activity`   
&emsp;->ActivityStackSupervisor#pauseBackStacks   
&emsp;&emsp;->ActivityStack#startPausingLocked     
&emsp;&emsp;&emsp;->ActivityThread#schedulePauseActivity   
&emsp;&emsp;&emsp;&emsp;->ActivityThread#handlePauseActivity   
&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityManagerService#activityPaused   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStack#activityPausedLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityStack#completePauseLocked   

`1-3.启动进程`
&emsp;->ActivityStackSupervisor#resumeTopActivitiesLocked   
&emsp;&emsp;->ActivityStack#resumeTopActivityLocked   
&emsp;&emsp;&emsp;->ActivityStack#resumeTopActivityInnerLocked   
&emsp;&emsp;&emsp;&emsp;->ActivityStackSupervisor#startSpecificActivityLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityManagerService#startProcessLocked   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->Process#start   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityThread#main   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->从ActivityThread#attach   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityManagerService#attachApplication   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityManagerService#attachApplicationLocked    
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityThread$ApplicationThread#bindApplication     
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityThread#handleBindApplication    

`1-4.启动新的activity，走ActivityStackSupervisor#attachApplicationLocked分支`
&emsp;->ActivityManagerService#attachApplicationLocked   
&emsp;&emsp;->ActivityStackSupervisor#attachApplicationLocked   
&emsp;&emsp;&emsp;->ActivityStackSupervisor#realStartActivityLocked   
&emsp;&emsp;&emsp;&emsp;->ActivityThread$ApplicationThread#scheduleLaunchActivity   
&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityThread#handleLaunchActivity   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->ActivityThread#performLaunchActivity   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->LoadedApk#makeApplication   
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;->Activity#attach   
