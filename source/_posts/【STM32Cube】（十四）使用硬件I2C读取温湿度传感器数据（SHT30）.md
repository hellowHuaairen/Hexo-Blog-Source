---
title: 【STM32Cube】（十四）使用硬件I2C读取温湿度传感器数据（SHT30）
date: 2019-08-08 10:48:56
tags:
    STM32CubeMX
    温湿度传感器
    SHT30
categories:
    STM32CubeMX
---

本篇详细的记录了如何使用STM32CubeMX配置STM32L431RCT6的硬件IIC外设，读取SHT30温湿度传感器的数据并通过串口发送。
<!--more-->

# 1. 准备工作
## 硬件准备
- 开发板
首先需要准备一个开发板，这里我准备的是STM32L4的开发板（BearPi）：

![mark](http://mculover666.cn/image/20190806/9uiPTi5odYSj.png?imageslim)

- SHT30温湿度传感器
SHT30温湿度传感器是一个完全校准的、现行的、带有温度补偿的**数字输出型**传感器，具有 2.4V-5.5V 的宽电压支持，使用IIC接口进行通信，最高速率可达1M并且有两个用户可选地址，除此之外，它还具有8个引脚的DFN超小封装，如图：

![mark](http://mculover666.cn/image/20190808/XthdnB8dTXFW.png?imageslim)

SHT30的原理图如下：
![mark](http://mculover666.cn/image/20190808/BSqAexcE2Pir.png?imageslim)

## 软件准备
- 需要安装好Keil - MDK及芯片对应的包，以便编译和下载生成的代码；
- 准备一个串口调试助手，这里我使用的是`Serial Port Utility`；

# 2.生成MDK工程
## 选择芯片型号
打开STM32CubeMX，打开MCU选择器：
![mark](http://mculover666.cn/image/20190806/gBP6glmUSH80.png?imageslim)

搜索并选中芯片`STM32L431RCT6`:
![mark](http://mculover666.cn/image/20190806/gnyHwdl53uVD.png?imageslim)

## 配置时钟源
- 如果选择使用外部高速时钟（HSE），则需要在System Core中配置RCC；
- 如果使用默认内部时钟（HSI），这一步可以略过；

这里我都使用外部时钟：
![mark](http://mculover666.cn/image/20190806/k593lGGb5tlW.png?imageslim)

## 配置串口
配置串口章节参考我的上一篇文章：[]()。

## 配置I2C接口
查看小熊派E53接口的原理图：
![mark](http://mculover666.cn/image/20190808/gHWofMib3ISQ.png?imageslim)

接下来开始配置I2C接口1：
![mark](http://mculover666.cn/image/20190808/GuKoTgin8iDJ.png?imageslim)

这些设置都保持默认即可。

## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：
![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

![mark](http://mculover666.cn/image/20190808/EVKCwrQNEWcl.png?imageslim)

![mark](http://mculover666.cn/image/20190808/iGkkUf1zI1HN.png?imageslim)


## 生成工程设置
![mark](http://mculover666.cn/image/20190808/vJ8NCpGew9fg.png?imageslim)

## 代码生成设置
最后设置生成独立的初始化文件：
![mark](http://mculover666.cn/image/20190806/T6WvSK6Dfpts.png?imageslim)

## 生成代码
点击`GENERATE CODE`即可生成MDK-V5工程：
![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

# 3. 在MDK中编写、编译、下载用户代码
## 重定向printf( )函数
参考：[【STM32Cube】（八）基于串口发送函数实现printf()](https://blog.csdn.net/Mculover666/article/details/95975461)。


## 修改I2C初始化代码的小BUG
![mark](http://mculover666.cn/image/20190808/zGOS6y9hjXhu.png?imageslim)

## 编写SHT30驱动程序

参考[SHT30数据手册.pdf](https://download.csdn.net/download/mculover666/11368379)进行编程。

先来编写`sht30_i2c_drv.h`头文件：

### 宏定义SHT30器件地址
SHT30的器件地址由`ADDR`端口的高低电平决定：

![mark](http://mculover666.cn/image/20190808/mYPfGe7jdXS3.png?imageslim)

注意数据手册中给出了8位数据，只有低7位用作地址，结合原理图，可以定义如下：
```c
/* ADDR Pin Conect to VSS */

#define	SHT30_ADDR_WRITE	0x44<<7         //10001000
#define	SHT30_ADDR_READ		(0x44<<7)+1	    //10001011
```
### 枚举SHT30命令列表
参考数据手册给出如下枚举定义：
```c
typedef enum
{
    /* 软件复位命令 */

    SOFT_RESET_CMD = 0x30A2,	
    /*
    单次测量模式
    命名格式：Repeatability_CS_CMD
    CS： Clock stretching
    */
    HIGH_ENABLED_CMD    = 0x2C06,
    MEDIUM_ENABLED_CMD  = 0x2C0D,
    LOW_ENABLED_CMD     = 0x2C10,
    HIGH_DISABLED_CMD   = 0x2400,
    MEDIUM_DISABLED_CMD = 0x240B,
    LOW_DISABLED_CMD    = 0x2416,

    /*
    周期测量模式
    命名格式：Repeatability_MPS_CMD
    MPS：measurement per second
    */
    HIGH_0_5_CMD   = 0x2032,
    MEDIUM_0_5_CMD = 0x2024,
    LOW_0_5_CMD    = 0x202F,
    HIGH_1_CMD     = 0x2130,
    MEDIUM_1_CMD   = 0x2126,
    LOW_1_CMD      = 0x212D,
    HIGH_2_CMD     = 0x2236,
    MEDIUM_2_CMD   = 0x2220,
    LOW_2_CMD      = 0x222B,
    HIGH_4_CMD     = 0x2334,
    MEDIUM_4_CMD   = 0x2322,
    LOW_4_CMD      = 0x2329,
    HIGH_10_CMD    = 0x2737,
    MEDIUM_10_CMD  = 0x2721,
    LOW_10_CMD     = 0x272A,
	
} SHT30_CMD;
```
### 发送命令函数
```c
/**
 * @brief	向SHT30发送一条指令(16bit)
 * @param	cmd —— SHT30指令（在SHT30_MODE中枚举定义）
 * @retval	成功返回HAL_OK
*/
static uint8_t	SHT30_Send_Cmd(SHT30_CMD cmd)
{
    uint8_t cmd_buffer[2];
    cmd_buffer[0] = cmd >> 8;
    cmd_buffer[1] = cmd;
    return HAL_I2C_Master_Transmit(&hi2c1, SHT30_ADDR_WRITE, (uint8_t* cmd_buffer, 2, 0xFFFF);
}
```
### 复位函数
```c
/**
 * @brief	复位SHT30
 * @param	none
 * @retval	none
*/
void SHT30_reset(void)
{
    SHT30_Send_Cmd(SOFT_RESET_CMD);
    HAL_Delay(20);
}
```
### SHT30工作模式初始化函数（周期测量模式）
```c
/**
 * @brief	初始化SHT30
 * @param	none
 * @retval	none
 * @note	周期测量模式
*/
void SHT30_Init(void)
{
    SHT30_Send_Cmd(MEDIUM_2_CMD);
}
```
### SHT30数据8位CRC校验
在数据手册中可知，SHTY30分别在温度数据和湿度数据之后发送了8-CRC校验码，确保了数据可靠性，接下来编写8-CRC校验代码：
```c

```



