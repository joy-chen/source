---
title: MTK-之音频驱动
date: 2018-03-10 15:33:04
tags:
---

Author JOY.
<!-- excerpt -->

### 配置
kernel-3.10/arch/arm64/configs/tx6735_65c_xz_l1_defconfig
```
CONFIG_MT_SND_SOC_V3=y
CONFIG_MTK_SPEAKER=y
```
### 驱动源码
kernel-3.10/sound/soc/mediatek/mt_soc_audio_v3/
