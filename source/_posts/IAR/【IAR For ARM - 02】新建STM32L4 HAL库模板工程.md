---
title: 【IAR For ARM - 02】新建STM32L4 HAL库模板工程
date: 2019-01-01 12:06:52
tags:
    IAR
    HAL库
    STM32
categories:
    IAR
---
本篇文章详细的讲述了如何基于HAL库使用 IAR For ARM 创建STM32L4的模板工程。

# 1. 新建工程文件夹并拷贝文件

## 新建工程文件夹
新建一个文件夹，用于存放IAR的工程，其结构如下：

![mark](http://mculover666.cn/image/20190823/1UqVrIEgk8WG.png?imageslim)

然后在`Drivers`文件夹中新建如下两个文件夹，和HAL库对应：

![mark](http://mculover666.cn/image/20190823/yBuCScXlLcP2.png?imageslim)

## 从HAL库拷贝CMSIS相关文件

这里直接从 STM32CubeMX 中的 HAL 库拷贝文件，HAL库路径如图所示：

![mark](http://mculover666.cn/image/20190823/lXwaAGshPUWW.png?imageslim)

### 拷贝CMSIS内核头文件

在路径`STM32Cube_FW_L4_V1.14.0\Drivers\CMSIS\Device\ST\STM32L4xx\`中拷贝头文件文件夹`include`：

![mark](http://mculover666.cn/image/20190823/2n7BVeaCJfUO.png?imageslim)

### 拷贝ST芯片对应头文件
在路径`STM32Cube_FW_L4_V1.14.0\Drivers\CMSIS\Device\ST\STM32L4xx\Include`中拷贝头文件：

![mark](http://mculover666.cn/image/20190823/xMrxEAR0JuOo.png?imageslim)

### 拷贝HAL驱动库

在路径`STM32Cube_FW_L4_V1.14.0\Drivers\STM32L4xx_HAL_Driver`中拷贝`Inc`和`Src`文件夹：

![mark](http://mculover666.cn/image/20190823/6DvetzcV8tby.png?imageslim)

分别将这两个文件夹中带有`ll`字样的文件全部删除，只保留HAL库文件。



