---
title: RT-Thread 通用GPIO设备管理框架使用文档
date: 2019-08-30 10:48:56
tags:
    RT-Thread
categories:
    RT-Thread
---
本文详细的说明了**如何配置使用 RT-Thread UART 设备驱动框架及使用API**。

<!--more-->

# 1. RT-Thread 通用 I/O 设备驱动

RT-Thread 的 通用设备框架分为三层：应用层、I/O设备管理层、硬件驱动层，其中串口设备相关的文件如图所示：

![mark](http://mculover666.cn/image/20190904/pjlW97znonEH.png?imageslim)

其中，RT-Thread 提供的 I/O 设备管理框架处于中间层：

- 向上层（应用层）提供抽象的设备操作接口
- 向下层（硬件驱动层）提供底层驱动框架

接下来使用小熊派开发板讲述如何具体如何配置和使用 I/O 设备管理框架来操作 UART 设备。

# 2. menuconfig 配置说明

在小熊派开发板的BSP中(`rt-thread\bsp\stm32\stm32l431-iotclub-bearpi`)右键，选择`ConEmu Here`，输入 `menuconfig` 打开 menuconfig 配置工具：

![mark](http://mculover666.cn/image/20190830/W4V7PBBmsYSY.png?imageslim)

## 2.1. 开启 UART 硬件（硬件驱动层）

>UART1 硬件默认在BSP中是开启的，本实验中保持**默认将 UART1 作为 Shell 输出，实验使用 UART2 进行收发测试**。

进入 `Hardware Drivers Config → On-chip Peripheral Drivers` 配置，然后开启 UART2 硬件外设驱动：

![mark](http://mculover666.cn/image/20190904/oSx7r6mlJRNo.png?imageslim)

## 2.2. 开启 RT-Thread 通用UART设备驱动框架（I/O设备管理层）

>在上一步中使能 UART 硬件后，**配置工具会自动开启使用通用UART2设备驱动框架**，这里仅仅是为了演示如何手动开启！

进入`RT-Thread Components → Device Drivers` 配置，可以看到 RT-Thread 通用 UART 设备驱动已经开启：

![mark](http://mculover666.cn/image/20190904/bUSU1lTOYUPP.png?imageslim)

## 2.3. 保存配置退出并生成工程

UART硬件驱动层和 RT-Thread UART 设备管理层已经开启，接下来保存配置退出，然后使用如下命令生成工程，在 MDK 中编写用户层代码
```
scons --target=mdk5
```
# 3. 检查 UART 设备是否成功注册

打开生成的 MDK 工程即可看到相关的文件：

![mark](http://mculover666.cn/image/20190904/Qm8NnHPToPxE.png?imageslim)

然后编译整个工程，下载到开发板中，在串口终端可以看到RT-Thread打印的信息。

在串口终端中输入 `list_device` 查看当前注册到内核的设备：

![mark](http://mculover666.cn/image/20190904/BbslSzA6DFhg.png?imageslim)

# 4. 调用 API 编写应用层代码

## 4.1. I/O 设备驱动框架提供的API

应用程序通过 RT-Thread提供的 I/O 设备管理框架提供的 API 来访问串口硬件，这些 API 存放在`rt-thread\components\drivers\include\drivers\serial.h` 文件中，所以**要想使用这些 API， 首先要包含头文件**：
```c
#include <drivers/serial.h>
```
提供的API列表如下：

|API|功能|
|:---:|:---:|
|rt_device_find|查找设备|
|rt_device_open|打开设备|
|rt_device_read|读取数据|
|rt_device_write|写入数据|
|rt_device_control|控制设备|
|rt_device_close()|关闭设备|
|rt_device_set_rx_indicate()|设置接收回调函数|
|rt_device_set_tx_complete()|设置发送完成回调函数|

## 4.2. 中断接收及轮询发送


