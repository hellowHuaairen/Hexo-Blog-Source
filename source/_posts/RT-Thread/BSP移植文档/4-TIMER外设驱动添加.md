---
title: TIMER外设驱动添加
date: 2019-08-11 19:58:56
tags:
    STM32CubeMX
    RT-Thread
categories:
    RT-Thread
---

<!--more-->

# 1. 添加通用定时器TIM2
## 1.1. 打开 STM32CubeMX 工程

打开 BSP 的 STM32CubeMX 配置文件：

![mark](http://mculover666.cn/image/20190827/PAmcyQgMgnQj.png?imageslim)

## 1.2. 使能通用定时器定时器 internal Clock

![mark](http://mculover666.cn/image/20190829/I1U2qPftAITm.png?imageslim)

## 1.3. 修改 Kconfig 文件

打开 board 文件夹下的 Konfig 文件，添加 TIM2 的配置项：

```
    menuconfig BSP_USING_TIM
        bool "Enable timer"
        default n
        select RT_USING_HWTIMER
        if BSP_USING_TIM
            config BSP_USING_TIM2
                bool "Enable TIM2"
                default n
        endif
```

## 1.4. 重新配置工程

经过上一步的修改，此时重新打开 ENV 工具，在 menuconfig 中就会出现添加的 TIM2 的配置项：

![mark](http://mculover666.cn/image/20190829/vRF37gaSBAqh.png?imageslim)

## 1.5. 生成工程，检查驱动文件

使用 ENV 重新生成工程并打开，工程提示 TIM2_CONFIG 未定义:

![mark](http://mculover666.cn/image/20190829/3Vs5unVHRliI.png?imageslim)

可以在 stm32/libraries/HAL_Drivers/config/l4/tim_config.h 中进行定义，如下图所示：

![mark](http://mculover666.cn/image/20190829/jl5AbNtDqvb0.png?imageslim)

其中定时器中断可以在 `stm32l431.h`中查看：

![mark](http://mculover666.cn/image/20190829/0zyVcpggA2V7.png?imageslim)

## 1.6. 编译下载

检查完工程后，编译下载到开发板，程序会自动开始运行。输入 list_device 命令，可以看到 timer2 已经注册到内核，说明驱动已经添加成功：

![mark](http://mculover666.cn/image/20190829/tViOphp1rG1w.png?imageslim)

# 2. 添加通用定时器TIM15

按照上面的步骤，添加TIM15，具体的过程不再赘述，结果如下：

![mark](http://mculover666.cn/image/20190829/6bPnqtru1t94.png?imageslim)

![mark](http://mculover666.cn/image/20190829/nJtLYN6k4L9f.png?imageslim)