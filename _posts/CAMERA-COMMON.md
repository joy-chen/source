---
title: CAMERA_COMMON
date: 2018-02-09 13:40:45
tags:
---

Author JOY.
<!-- excerpt -->

## Image Sensor类型

### YUV Sensor
YUV Sensor输出的Data格式为YUV，图像的效果处理使用Sensor内部的ISP，BB端接收YUV格式的data后只进行格式的转换，效果方面不进行处理，由于Sensor内部的ISP处理能力有限，且YUV Sensor的数据量比较大（YUV422的格式1个pixel2个byte），一般Size都比较小，常见的YUV sensor都是5M以下。

### Raw Sensor
Raw Sensor输出的Data格式为Raw，图像的效果处理使用BB端的ISP，BB端接收Raw data后进行一系列的图像处理（OB，Shading，AWB，Gamma，EE，ANR等），效果方面由BB端控制，需要针对不同的模组进行效果调试，Raw sensor是目前的主流，数据量比YUV Sensor小（RAW10 格式的sensor 1个pixel 10个bit）使用平台ISP处理，能支持较大的size。
