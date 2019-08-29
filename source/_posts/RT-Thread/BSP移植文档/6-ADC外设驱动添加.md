---
title: ADC外设驱动添加
date: 2019-08-11 21:58:56
tags:
    STM32CubeMX
    RT-Thread
categories:
    RT-Thread
---

<!--more-->

# 0. 说明
小熊派开发板扩展板上的 MQ-2 气体传感器使用 ADC1 的 CH_3 通道采样。

裸机HAL库驱动可以参考我的这篇博客：

- [【STM32Cube_10】使用ADC读取气体传感器数据（MQ-2）](https://www.mculover666.cn/2019/07/31/STM32Cube/【STM32Cube-10】使用ADC读取气体传感器数据（MQ-2）/)

# 1. 添加ADC1
## 1.1. 打开 STM32CubeMX 工程

打开 BSP 的 STM32CubeMX 配置文件：

![mark](http://mculover666.cn/image/20190827/PAmcyQgMgnQj.png?imageslim)

## 1.2. 使能ADC1并选择使用的通道

![mark](http://mculover666.cn/image/20190829/aFDe6JuoVLGy.png?imageslim)

## 1.3. 修改 Kconfig 文件

打开 board 文件夹下的 Konfig 文件，添加ADC1 的配置项：

```
    menuconfig BSP_USING_ADC
        bool "Enable ADC"
        default n
        select RT_USING_ADC
        if BSP_USING_ADC
            config BSP_USING_ADC1
                bool "Enable ADC1"
                default n
        endif
```

## 1.4. 重新配置工程

经过上一步的修改，此时重新打开 ENV 工具，在 menuconfig 中就会出现添加的 ADC1 的配置项：

![mark](http://mculover666.cn/image/20190829/V5SJwG20crPK.png?imageslim)

## 1.5. 生成工程，检查驱动文件

生成工程，检查工程中是否有drv_adc.c文件。

## 1.6. 编译下载

检查完工程后，编译下载到开发板，程序会自动开始运行。输入 list_device 命令，可以看到 adc1 已经注册到内核，说明驱动已经添加成功：

![mark](http://mculover666.cn/image/20190829/QEl9s2mpQXAi.png?imageslim)