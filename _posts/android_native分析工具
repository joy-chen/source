---
title: GUI
date: 2016-09-18 19:26:37
tags: Graphics
thumbnailImage: ape_fwk_graphics.png
thumbnailImagePosition: right
metaAlignment: center
coverSize: partial
---

SurfaceFlinger
<!-- excerpt -->
{% image center clear ape_fwk_graphics.png "img" %}

### SurfaceFlinger
SurfaceFlinger accepts buffers of data from multiple sources, composites them, and sends them to the display.

WindowManager -> SurfaceFlinger -> Surface -> Layer -> BufferQueue

## Debug
How to check whether HWUI is enabled or not
* adb shell dumpsys SurfaceFlinger
  * mConnectedAPI=1 means the window is rendered by OpenGL, ie. Hwui
  * mConnectedAPI=2 means the window is rendered by CPU, ie. skia
