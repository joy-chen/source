---
title: MTK_OTA分析
date: 2017-12-28 20:44:24
tags:
---


Author JOY.
<!-- excerpt -->

### SystemUpdate
MainEntry
  onStart()
    bindService(serviceIntent, mConnection, Context.BIND_AUTO_CREATE);
      onServiceConnected()
        queryPackagesInternal();
          mService.queryPackages();
            sQueryNewVersionThread.start();
              mHttpManager.queryNewVersion();
                checkNewVersion()
