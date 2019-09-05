---
title: SPI总线外设驱动添加
date: 2019-08-11 18:48:56
tags:
    STM32CubeMX
    RT-Thread
categories:
    RT-Thread
---

<!--more-->

# 1. 添加SPI
## 1.1. 打开 STM32CubeMX 工程

打开 BSP 的 STM32CubeMX 配置文件：

![mark](http://mculover666.cn/image/20190827/PAmcyQgMgnQj.png?imageslim)

## 1.2. 按原理图配置 SPI1 的引脚，并生成代码

按图示顺序配置 SPI1，然后点击生成代码：

![mark](http://mculover666.cn/image/20190828/dqkSAh7d5lOL.png?imageslim)

## 1.3. 修改 Kconfig 文件

打开 board 文件夹下的 Konfig 文件，添加 SPI1 的配置项：

```
    menuconfig BSP_USING_SPI
        bool "Enable SPI BUS"
        default n
        select RT_USING_SPI
        if BSP_USING_SPI
            config BSP_USING_SPI1
                bool "Enable SPI1 BUS"
                default n

            config BSP_SPI1_TX_USING_DMA
                bool "Enable SPI1 TX DMA"
                depends on BSP_USING_SPI1
                default n
                
            config BSP_SPI1_RX_USING_DMA
                bool "Enable SPI1 RX DMA"
                depends on BSP_USING_SPI1
                select BSP_SPI1_TX_USING_DMA
                default n
        endif
```

## 1.4. 重新配置工程

经过上一步的修改，此时重新打开 ENV 工具，在 menuconfig 中就会出现添加的 SPI1 的配置项：

![mark](http://mculover666.cn/image/20190828/J8zi04czTD6C.png?imageslim)

## 1.5. 生成工程，检查驱动文件

使用 ENV 重新生成工程并打开，检查原有驱动文件是否支持新添加的驱动（查看是否有新驱动的配置文件，中断函数，DMA配置和中断函数等等），如不支持，需参考现有驱动添加相关的代码：

![mark](http://mculover666.cn/image/20190828/gxy2xYxdBK0w.png?imageslim)

## 1.6. 编译下载

检查完工程后，编译下载到开发板，程序会自动开始运行。输入 list_device 命令，可以看到 spi1 总线已经注册到内核，说明驱动已经添加成功：

![mark](http://mculover666.cn/image/20190828/skop3oNOoXFV.png?imageslim)

# 2. 添加 SPI3 驱动

和添加 SPI1 的方法相同，添加SPI2，具体的过程不再赘述：

![mark](http://mculover666.cn/image/20190828/xccSm3qIcTR0.png?imageslim)

![mark](http://mculover666.cn/image/20190828/yWRoPWBfzUnu.png?imageslim)




在小熊派开发板上SPI2是接在LCD屏幕的，没有外接，所以使用 STM32CubeMX 配置时需要使用如下的模式：




