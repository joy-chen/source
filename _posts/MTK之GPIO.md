---
title: MTK之GPIO
date: 2018-02-26 19:40:24
tags:
---

Author JOY.
<!-- excerpt -->

tools/dct/DrvGen.exe
drivers/misc/mediatek/mach/mt6735/tx6735_65c_xz_l1/dct/dct/codegen.dws
tools/dct/GPIO_YuSu.cmp

mt_set_gpio_mode(GPIO_LCM_PWR_EN, GPIO_MODE_00);
mt_set_gpio_dir(GPIO_LCM_PWR_EN, GPIO_DIR_OUT);
mt_set_gpio_out(GPIO_LCM_PWR_EN, GPIO_OUT_ONE);
