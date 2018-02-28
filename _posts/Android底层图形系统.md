---
title: Android底层图形系统
date: 2018-01-19 17:46:14
tags:
---
Author JOY.
<!-- excerpt -->

## OpenGL ES
### 什么是OpenGL？
OpenGL是和编程语言、平台无关的一套interface ，主要是为了渲染 2D 和 3D图形等。一般这套接口是用来和GPU进行交互的，使用GPU进行硬件加速。

### 什么是OpenGL ES？
OpenGL ES就是专为嵌入式设备设计的，OpenGL ES和OpenGL中的函数接口有一些是不一样，因为嵌入式设备和pc等的硬件处理能力有差距的。

既然OpenGL ES只是一组函数接口，Android平台提供了两种类型的实现：软件实现，硬件实现。

a.硬件实现，前面提到这组函数接口主要是为了和GPU这个硬件进行打交道的。所以各个硬件厂商会提供相关的实现，例如高通平台的adreno解决方案；

b.软件实现，Android自身也提供了一套OpenGL ES的软件实现，不使用GPU，完全用软件实现画图的相关功能，也就是libagl

### EGL
那么什么是EGL？EGL是OpenGL ES和底层的native window system之间的接口，承上启下。

![架构](https://github.com/joy-chen/ResourceForBlog/blob/master/android/graph/egl.png?raw=true)
