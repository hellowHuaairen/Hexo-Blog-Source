---
title: 移植Coremark跑分工具到STM32
date: 2019-08-15 17:00:00
tags:
    STM32CubeMX STM32 Coremark
categories:
    Coremark移植
---
本篇文章详细的讲述了如何移植Coremark跑分工具到STM32 系列 MCU 上。

<!--more-->
 # 1. Coremark跑分工具

 Coremark是一个测试处理器性能的工具，一般只关心它最后给出的分数，所以江湖上称之为：“单片机跑分工具”。

 Coremark全部是用C语言写的，其中主要包括了一些测试性能的运算，如下：

 - 列举
 - 数学矩阵运算
 - CRC运算
 - ……

Coremark通过测量CPU进行这些运算的时间，从而测出CPU的性能，给出Coremark得分，分数越高，性能越强。

# 2. 下载Coremark代码

Coremark的[官方网站](https://www.eembc.org/coremark/index.php)上可以看到更多介绍，并且在官网上可以看到由众多网友提交的处理器跑分，本文中我使用的是 `STM32L431RCT6` 这个芯片，在官网上先搜索一下看看别人的跑分：

![mark](http://mculover666.cn/image/20190815/wpvNF1HyztfY.png?imageslim)

Coremark的代码托管在[Github](https://github.com/eembc/coremark)上，使用Apache许可证，可以免费使用，使用 Git 下载代码到本地：
```bash
git clone https://github.com/eembc/coremark.git
```
下载之后第一眼看到的C文件就是Coremark的测试代码文件了，如下：

- core_list_join.c
- core_main.c
- core_matrix.c
- core_state.c
- core_util.c
- coremark.h
- simple/core_portme.c
- simple/core_portme.h

# 3. 使用STM32CubeMX快速生成带USART的工程

测试之后的结果**需要使用串口输出**，所以先使用STM32CubeMX快速生成一个初始化USART的工程：

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
小熊派开发板板载ST-Link并且虚拟了一个串口，原理图如下：

![mark](http://mculover666.cn/image/20190814/IwyXONVefPx9.png?imageslim)

这里我将开关拨到`AT-MCU`模式，使PC的串口与USART1之间连接。

接下来开始配置`USART1`：

![mark](http://mculover666.cn/image/20190814/nLMRMYtmzghl.png?imageslim)


## 配置时钟树
STM32L4的最高主频到80M，所以配置PLL，最后使`HCLK = 80Mhz`即可：

![mark](http://mculover666.cn/image/20190806/1TQg7frjRpVr.png?imageslim)

## 生成工程设置

![mark](http://mculover666.cn/image/20190815/Dpl3BcXAQQjB.png?imageslim)

## 代码生成设置

最后设置生成独立的初始化文件：

![mark](http://mculover666.cn/image/20190812/PwTCS6QzHiyG.png?imageslim)

## 生成代码

点击`GENERATE CODE`即可生成MDK-V5工程：

![mark](http://mculover666.cn/image/20190806/s0jGhLBWW6Cm.png?imageslim)

## 测试串口是否可以正常输出
在 `main` 函数中的 `while(1)` 之前，添加下面这行测试代码：
```c
    /* USER CODE BEGIN 1 */   
    char hello[12] = "Hello world\n";
    /* USER CODE END 1 */

    //这里省略一些自动生成的函数

    /* USER CODE BEGIN 2 */
    HAL_UART_Transmit(&huart1, (uint8_t*)hello, 12, 0xFFFF);
    /* USER CODE END 2 */
```
设置下载器，编译运行，可以看到串口正常：

![mark](http://mculover666.cn/image/20190815/72uuvLOxChqP.png?imageslim)

# 4. 添加Coremark代码

## 拷贝代码

![mark](http://mculover666.cn/image/20190815/BJuXXACV97sw.png?imageslim)

![mark](http://mculover666.cn/image/20190815/79vH0DzPbiEj.png?imageslim)

![mark](http://mculover666.cn/image/20190815/IQpTw8BRBNVr.png?imageslim)

## 向工程中添加代码

![mark](http://mculover666.cn/image/20190815/YOsUW9iSm4fL.png?imageslim)

因为core_main.c中已经有一个main函数了，所以将原来的 `main.c` 文件从工程中去除（**是去除，不是删除源文件，后面还会用到**）：

![mark](http://mculover666.cn/image/20190815/kXCKuvCmdqJN.png?imageslim)

## 添加头文件路径

在src文件夹中新建了 `Coremark` 文件夹，将其添加到头文件搜索路径中：

![mark](http://mculover666.cn/image/20190815/LuPnjCoUdtUt.png?imageslim)

# 5. 配置Coremark文件

添加完了Coremark所有的相关文件后，就要开始最精彩的环节了 —— 修改移植文件，适配自己的开发板。

## 添加初始化代码

### 修改portable_init函数
`Core_portme.c` 中的 `portable_init` 函数在 `Core_main.c` 中首先调用，将原来的main函数中的硬件初始化代码合并到该函数中，合并后的代码如下：

```c
void portable_init(core_portable *p, int *argc, char *argv[])
{
    /* MCU Configuration--------------------------------------------------------*/

    /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
    HAL_Init();

    /* USER CODE BEGIN Init */

    /* USER CODE END Init */

    /* Configure the system clock */
    SystemClock_Config();

    /* USER CODE BEGIN SysInit */

    /* USER CODE END SysInit */

    /* Initialize all configured peripherals */
    MX_GPIO_Init();
    MX_USART1_UART_Init();
	

    if (sizeof(ee_ptr_int) != sizeof(ee_u8 *)) {
        ee_printf("ERROR! Please define ee_ptr_int to a type that holds a pointer!\n");
    }
    if (sizeof(ee_u32) != 4) {
        ee_printf("ERROR! Please define ee_u32 to a 32b unsigned type!\n");
    }
    p->portable_id=1;
}
```

### 拷贝SystemClock_Config函数和Error_Handler函数的声明和实现
将原来main函数中的 `SystemClock_Config` 函数和 `Error_Handler`函数 拷贝到移植文件 `Core_portme.c` 中：

![mark](http://mculover666.cn/image/20190815/mPBUKe0Lz5BO.png?imageslim)

![mark](http://mculover666.cn/image/20190815/fhpwctvlajdx.png?imageslim)

![mark](http://mculover666.cn/image/20190815/ECnI9Hh4VmL4.png?imageslim)

### 拷贝包含的头文件
将原来main函数中包含的头文件拷贝到移植文件 `Core_portme.c` 中：

![mark](http://mculover666.cn/image/20190815/3WoegSU0AKbl.png?imageslim)

## 实现printf函数
在 `usart.c` 文件的最后添加如下代码：
```c
#if 1
#pragma import(__use_no_semihosting)
#include <stdio.h>
//标准库需要的支持函数
struct __FILE
{
    int handle;
};

FILE __stdout;
/**
 * @brief	定义_sys_exit()以避免使用半主机模式
 * @param	void
 * @return  void
 */
void _sys_exit(int x)
{
    x = x;
}
/**
 * @brief		重定义fputc函数
 * @param		ch	输出字符量
 * @param		f		文件指针
 * @retval	none
 */
int fputc(int ch, FILE *f)
{
    while((USART1->ISR & 0X40) == 0); //循环发送,直到发送完毕

    USART1->TDR = (uint8_t) ch;
    return ch;
}
#endif
```

## 修改计时相关代码

`Core_portme.c` 中的`start_time() stop_time() get_time()`这几个函数是Coremark程序运行时测量时间使用的。

HAL库中默认开启了Systick给RTOS作心跳用，所以本文中我们直接使用system_tick进行计时：

- 将Systick配置为1ms的中断间隔；
- 在Systick的中断处理函数中增加tick的值；

接下来我们逐个进行修改：

### 添加宏定义和全局变量
在 `Core_portme.c` 添加如下代码：
```c
#define SysTick_Counter_Disable ((uint32_t)0xFFFFFFFE)
#define SysTick_Counter_Enable ((uint32_t)0x00000001)
#define SysTick_Counter_Clear ((uint32_t)0x00000000)

__IO uint32_t Tick;
```
### 修改Systick中断处理函数
SysTick的中断处理函数在 `stm32l4xx_it.c` 中，如图：

![mark](http://mculover666.cn/image/20190815/Csn7Jsw0UWpW.png?imageslim)

该函数转而调用 `HAL_IncTick()` 函数处理中断，该函数使用 `__weak`弱定义，所以在 `Core_portme.c` 中重新实现该函数：
```c
void HAL_IncTick(void)
{
  Tick++;
}
```
### 修改coremark API函数
- `start_time()`
```c
void start_time(void) {
	Tick = 0;
	SysTick_Config(SystemCoreClock/1000);
}
```
- `stop_time()`
```c
void stop_time(void) {
	SysTick->CTRL &= SysTick_Counter_Disable;
	SysTick->VAL = SysTick_Counter_Clear;
}
```
- `get_time()`
```c
CORE_TICKS get_time(void) {
	CORE_TICKS elapsed = (CORE_TICKS)Tick;
	return elapsed;
}
```
### 修改文件中自带的宏定义

注释掉不用的定义，修改宏定义 `EE_TICKS_PER_SEC`：

![mark](http://mculover666.cn/image/20190815/p7qmK3khOlqX.png?imageslim)

# 6. CoreMark运行配置

## 设置迭代次数
CoreMark 要求程序运行的最短时间至少是 10s, 根据使用的系统时钟等情况，可以在 Core_portme.h 中修改迭代次数：
```c
#define ITERATIONS 12000
```
## 设置打印信息

在 `Core_portme.c` 文件中，根据具体所用的编译器版本，优化配置进行修改：
```c
#ifndef COMPILER_VERSION 
 #ifdef __GNUC__
 #define COMPILER_VERSION "GCC"__VERSION__
 #else
 #define COMPILER_VERSION "Using Compiler V5.06 update 6 (build 750)"
 #endif
#endif
#ifndef COMPILER_FLAGS 
 #define COMPILER_FLAGS "-o3" /* "Please put compiler flags here (e.g. -o3)" */
#endif
#ifndef MEM_LOCATION 
 #define MEM_LOCATION "STACK"
#endif
```

## 设置优化等级

![mark](http://mculover666.cn/image/20190815/13tSGATkoFYG.png?imageslim)

# 7. 运行结果
编译下载运行，结果如下：


