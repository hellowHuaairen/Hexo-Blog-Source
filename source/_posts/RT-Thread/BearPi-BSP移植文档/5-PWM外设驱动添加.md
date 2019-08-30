---
title: PWM外设驱动添加
date: 2019-08-11 20:58:56
tags:
    STM32CubeMX
    RT-Thread
categories:
    RT-Thread
---

<!--more-->

# 0. 说明
小熊派开发板扩展板上的蜂鸣器使用 TIM16 的 channel1 产生 PWM 驱动，默认的CH1是PA6引脚，但是开发板上是与 PB8 连接的，所以配置时需要修改 channel 1 的通道。

裸机HAL库驱动可以参考我的这篇博客：

- [使用通用定时器产生PWM驱动蜂鸣器](https://www.mculover666.cn/2019/08/02/STM32Cube/【STM32Cube-12】使用通用定时器产生PWM驱动蜂鸣器/)

# 1. 添加通用定时器TIM16
## 1.1. 打开 STM32CubeMX 工程

打开 BSP 的 STM32CubeMX 配置文件：

![mark](http://mculover666.cn/image/20190827/PAmcyQgMgnQj.png?imageslim)

## 1.2. 使能通用定时器TIM16的PWM

![mark](http://mculover666.cn/image/20190829/CNOGgPVtYy1c.png?imageslim)

## 1.3. 修改 Kconfig 文件

打开 board 文件夹下的 Konfig 文件，添加 TIM16 PWM 的配置项：

```
    menuconfig BSP_USING_PWM
        bool "Enable pwm"
        default n
        select RT_USING_PWM
        if BSP_USING_PWM
        menuconfig BSP_USING_PWM16
            bool "Enable timer16 output pwm"
            default n
            if BSP_USING_PWM16
                config BSP_USING_PWM16_CH1
                    bool "Enable PWM16 channel1"
                    default n
            endif
        endif
```

## 1.4. 重新配置工程

经过上一步的修改，此时重新打开 ENV 工具，在 menuconfig 中就会出现添加的 TIM16 的配置项：

![mark](http://mculover666.cn/image/20190829/yYjMv6UwKDWC.png?imageslim)

## 1.5. 生成工程，检查驱动文件

使用 ENV 重新生成工程并打开，工程提示 PWM16_CONFIG 未定义:

![mark](http://mculover666.cn/image/20190829/aToK1omm5t0j.png?imageslim)

可以在 `stm32/libraries/HAL_Drivers/config/l4/pwm_config.h` 中进行定义，如下图所示：

![mark](http://mculover666.cn/image/20190829/NTwYeE8mRCL7.png?imageslim)

## 1.6. 编译下载

检查完工程后，编译下载到开发板，程序会自动开始运行。输入 list_device 命令，可以看到 pwm16 已经注册到内核，说明驱动已经添加成功：

![mark](http://mculover666.cn/image/20190829/HTCjK7m8j7Vh.png?imageslim)