---
title: Android之Sensor
date: 2018-01-29 14:05:00
tags:
---
Author JOY.
<!-- excerpt -->

### TYPE_ORIENTATION
This constant is deprecated.  use SensorManager.getOrientation() instead. ”也就是说，这种方式已经被取消，要开发者使用 SensorManager.getOrientation()来获取原来的数据。实际上，android获取方向是通过磁场感应器和加速度感应器共同获得的，至于具体的算法SDK已经封装好了。
