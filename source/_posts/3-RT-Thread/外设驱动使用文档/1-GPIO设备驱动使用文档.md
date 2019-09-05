---
title: RT-Thread 通用GPIO设备管理框架使用文档
date: 2019-08-30 09:48:56
tags:
    RT-Thread
categories:
    RT-Thread
---
RT-Thread的通用GPIO设备驱动向用户提供了**类似 Arduino 风格的操作GPIO的通用API**，非常简单易用，包括设置 GPIO 模式，输出高低电平，读取电平输入，配置GPIO外部中断等操作，本文详细的说明了**如何配置使用GPIO设备驱动框架及使用API**。

<!--more-->

# 1. RT-Thread 通用GPIO设备驱动

RT-Thread 的 I/O 设备分为三层：应用层、I/O设备管理层、硬件驱动层，如图所示：

![mark](http://mculover666.cn/image/20190830/dm6DQXNSWjgg.png?imageslim)

其中，RT-Thread 提供的 I/O 设备管理框架处于中间层：

- 向上层（应用层）提供抽象的设备操作接口
- 向下层（硬件驱动层）提供底层驱动框架

因为 GPIO 设备的驱动比较简单，所以通常情况下**直接使用 RT-Thread 提供的 通用 GPIO 设备驱动来操作具体的 GPIO 硬件**。

那么问题来了，ST 提供的 HAL 库中就有直接操作 GPIO 的函数呀，比如`HAL_GPIO_WritePin()`等等，为什么我们不直接使用 HAL 库的，而是使用 RT-Thread 提供的这一套设备驱动框架呢？

原因有二：

- 操作简单：相比 HAL 库的初始化，使用 RT-Thread 的设备驱动提供的类似Arduino的API，操作变得非常简单；
- 使用 RT-Thread 设备驱动框架提供的 API，可移植性大大的提高，如果说 HAL 库的 API 可以移植到STM32全系列上，那么 RT-Thread 的通用 API 可以移植到所有 RT-Thread BSP支持的板子上，听着就很爽哈哈。

接下来使用小熊派开发板讲述如何具体如何配置和使用。

# 2. menuconfig 配置说明

在小熊派开发板的BSP中(`rt-thread\bsp\stm32\stm32l431-iotclub-bearpi`)右键，选择`ConEmu Here`，输入 `menuconfig` 打开 menuconfig 配置工具：

![mark](http://mculover666.cn/image/20190830/W4V7PBBmsYSY.png?imageslim)


## 2.1. 开启 GPIO 硬件（硬件驱动层）
>**GPIO 硬件默认在BSP中是开启的**，这里仅仅是为了演示如何手动开启！

进入 `Hardware Drivers Config → On-chip Peripheral Drivers` 配置，然后开启GPIO硬件外设驱动：

![mark](http://mculover666.cn/image/20190830/Skv0U2ETQdVi.png?imageslim)

## 2.2. 开启 RT-Thread 通用GPIO设备驱动框架（I/O设备管理层）

>在上一步中使能 GPIO 硬件后，**配置工具会自动开启使用通用GPIO设备驱动框架**，这里仅仅是为了演示如何手动开启！

进入`RT-Thread Components → Device Drivers` 配置，可以看到 RT-Thread 通用 GPIO 设备驱动已经开启：

![mark](http://mculover666.cn/image/20190830/PYa24Nr7sB9e.png?imageslim)

## 2.3. 保存配置退出并生成工程

GPIO硬件驱动层和 RT-Thread I/O 设备管理层已经开启，接下来保存配置退出，然后使用如下命令生成工程，在 MDK 中编写用户层代码
```
scons --target=mdk5
```

# 3. 检查 GPIO 设备是否成功注册

打开生成的 MDK 工程即可看到相关的文件：

![mark](http://mculover666.cn/image/20190830/x7rdo8AUegIj.png?imageslim)

然后编译整个工程，下载到开发板中，在串口终端可以看到RT-Thread打印的信息。

在串口终端中输入 `list_device` 查看当前注册到内核的设备：

![mark](http://mculover666.cn/image/20190830/51usB8yGgblq.png?imageslim)

>GPIO 设备不是标准的字符设备或者块设备，所以设备类型是 Miscellaneous Device(杂类设备)。

# 4. 调用 API 编写应用层代码

## 4.1. GPIO 设备驱动框架提供的API

RT-Thread 通用 GPIO 设备驱动框架提供了类似 Arduino 风格的 API，这些 API 存放在`rt-thread\components\drivers\include\drivers\pin.h` 文件中，所以**要想使用这些 API， 首先要包含头文件**：
```c
#include <drivers/pin.h>
```
提供的API列表如下：

|API|功能|
|:---:|:---:|
|rt_pin_mode|设置 GPIO 引脚模式|
|rt_pin_write|设置 GPIO 引脚输出电平|
|rt_pin_read|读取输入 GPIO 引脚电平|
|rt_pin_attach_irq|绑定 GPIO 引脚中断|
|rt_pin_detach_irq|脱离 GPIO 引脚中断|
|rt_pin_irq_enable|使能/禁止 GPIO 引脚中断|

这里先讲述 `rt_pin_mode` API 的用法，剩余的 API 在后面具体使用时会讲到。

>API说明 —— rt_pin_mode

API原型：
```c
void rt_pin_mode(rt_base_t pin, rt_base_t mode);
```
参数：

- pin：引脚编号，由 GPIO 硬件驱动定义，在文件 `drv_gpio.c` 中的 `struct pin_index pins[]` 中可以找到，比如这里 `PC13` 对应的就是 `45`。
- mode：引脚模式，在文件`pin.h`中定义，有如图所示的5种模式可以用。
![mark](http://mculover666.cn/image/20190830/OgPPJeRkyMVH.png?imageslim)

返回参数：无。

## 4.2. GPIO 输出示例
小熊派开发板板载一个用户LED，连接在 PC13 引脚上，高电平点亮，如图：

![mark](http://mculover666.cn/image/20190812/5iCtQUfKbgzA.png?imageslim)

因为BSP的 `main.c` 中已经包含了点亮该LED的代码，将相关代码全部注释掉：

![mark](http://mculover666.cn/image/20190830/lGMhPmQ45bQy.png?imageslim)

然后开始编写我们自己的代码~

新建 `led_example.c` 文件，添加到 `application` 分组下。

首先包含必要的头文件：
```c
#include <rtthread.h>
#include <drivers/pin.h>
```
然后编写线程主体函数，使用 GPIO 通用驱动框架提供的 API 初始化 LED引脚，使LED闪烁：
```c
#define LED 45  //PC13 <--> 45

/**
 * @brief   LED线程入口函数
 * @param   parameter:函数被调用时传入参数
 * @retval  none
 */ 
void led_thread_entry(void* parameter)
{
    /* 设置LED的GPIO引脚模式为输出 */
    rt_pin_mode(LED, PIN_MODE_OUTPUT);
    
    /* LED 闪烁 */
    while(1)
    {
        rt_pin_write(LED, PIN_HIGH);
        rt_kprintf("led on\n");
        rt_thread_mdelay(500);
        
        rt_pin_write(LED, PIN_LOW);
        rt_kprintf("led off\n");
        rt_thread_mdelay(500);
    }
}
```
创建该线程：
```c
/**
 * @brief   动态创建LED线程
 * @param   none
 * @retval  none
 */ 
int led_thread(void)
{
    rt_thread_t tid;    //线程句柄
    
    tid = rt_thread_create("led_thread",
                            led_thread_entry,
                            RT_NULL,
                            256,
                            9,
                            10);
   if(tid != RT_NULL)
   {
        //线程创建成功，启动线程
        rt_thread_startup(tid);
   }
   
   return 0;
}
```
最后导出到msh命令列表中：
```c
/* 导出到 msh 命令列表中 */
MSH_CMD_EXPORT(led_thread, pin led sample);
```
编译运行之后，效果如下：

![mark](http://mculover666.cn/image/20190830/qpEcxdM3ArEq.png?imageslim)

## 4.3. GPIO 输入示例
小熊派开发板板载两个用户按键，KEY1 连接在 PB2 引脚上，KEY2 连接在 PB3 引脚上，松开时高电平，按下时低电平，如图：

![mark](http://mculover666.cn/image/20190813/QNbG5i6QKGk3.png?imageslim)

本实验中检测 KEY1 按键是否按下。

同样，新建一个 `key_example.c` 文件，添加到 `application` 分组下。

首先包含必要的头文件：
```c
#include <rtthread.h>
#include <drivers/pin.h>
```

然后编写线程主体函数，使用 GPIO 通用驱动框架提供的 API 初始化 KEY1 引脚，检测按键是否按下：
```c
#define KEY1 18  //PB2 <--> 18
/**
 * @brief   KEY线程入口函数
 * @param   parameter:函数被调用时传入参数
 * @retval  none
 */ 
void key_thread_entry(void* parameter)
{
    /* 设置 KEY1 的GPIO引脚模式为上拉输入 */
    rt_pin_mode(KEY1, PIN_MODE_INPUT_PULLUP);
    
    /* 检测 KEY1 按键 */
    while(1)
    {
        /* 检测按键KEY1是否按下 */
        if(rt_pin_read(KEY1) == PIN_LOW)
        {
            rt_kprintf("key1 pressed!\n");
        }
        rt_thread_mdelay(10);
    }
}
```
创建该线程：
```c
/**
 * @brief   动态创建KEY线程
 * @param   none
 * @retval  none
 */ 
int key_thread(void)
{
    rt_thread_t tid;    //线程句柄
    
    tid = rt_thread_create("key_thread",
                            key_thread_entry,
                            RT_NULL,
                            256,
                            9,
                            10);
   if(tid != RT_NULL)
   {
        //线程创建成功，启动线程
        rt_thread_startup(tid);
   }
   
   return 0;
}
```
最后导出到msh命令列表中：
```c
/* 导出到 msh 命令列表中 */
MSH_CMD_EXPORT(key_thread, pin key sample);
```
编译运行之后，效果如下：

![mark](http://mculover666.cn/image/20190830/lK98tbTOrEKR.png?imageslim)

## 4.4. GPIO 中断使用示例
小熊派开发板板载两个用户按键，KEY1 连接在 PB2 引脚上，KEY2 连接在 PB3 引脚上，松开时高电平，按下时低电平，如图：

![mark](http://mculover666.cn/image/20190813/QNbG5i6QKGk3.png?imageslim)

本实验中使用中断方式检测 KEY2 是否按下。

同样，新建一个 `key_it_example.c` 文件，添加到 `application` 分组下：

首先包含必要的头文件：
```c
#include <rtthread.h>
#include <drivers/pin.h>
```

然后编写线程主体函数，使用 GPIO 通用驱动框架提供的 API 初始化 KEY1 引脚，检测按键是否按下：
```c
#define KEY2 19  //PB3 <--> 19

/**
 * @brief   KEY2 按键中断回调函数
 * @param   args:函数被调用时传入参数
 * @retval  none
 */ 
void key2_press(void *args)
{
    rt_kprintf("key 2 press!\n");
}

/**
 * @brief   KEY线程入口函数
 * @param   parameter:函数被调用时传入参数
 * @retval  none
 */ 
void key_it_thread_entry(void* parameter)
{
    /* 设置 KEY2 的GPIO引脚模式为上拉输入 */
    rt_pin_mode(KEY2, PIN_MODE_INPUT_PULLUP);
    
    /* 绑定中断，下降沿模式，回调函数key2_press */
    rt_pin_attach_irq(KEY2, PIN_IRQ_MODE_FALLING, key2_press, RT_NULL);
    
    /* 使能中断 */
    rt_pin_irq_enable(KEY2, PIN_IRQ_ENABLE);
    
}
```
创建该线程：
```c
/**
 * @brief   动态创建key_it线程
 * @param   none
 * @retval  none
 */ 
int key_it_thread(void)
{
    rt_thread_t tid;    //线程句柄
    
    tid = rt_thread_create("key_it_thread",
                            key_it_thread_entry,
                            RT_NULL,
                            256,
                            9,
                            10);
   if(tid != RT_NULL)
   {
        //线程创建成功，启动线程
        rt_thread_startup(tid);
   }
   
   return 0;
}
```
最后导出到msh命令列表中：
```c
/* 导出到 msh 命令列表中 */
MSH_CMD_EXPORT(key_it_thread, pin key it mode sample);
```
编译运行之后，效果如下：

![mark](http://mculover666.cn/image/20190830/qeQkHDcRasJj.png?imageslim)







