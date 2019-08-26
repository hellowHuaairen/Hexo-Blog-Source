
<!--more-->
# 1.准备工作

## 硬件准备

- 小熊派开发板

## 软件准备

- 拉取RT-Thread master分支代码
- MDK+STM32L4 pack
- 串口终端

# 2.编译小熊派BSP

在拉取的 RT-Thread 源码中，进入`bsp\stm32\stm32l431-iotclub-bearpi`文件夹，这就是小熊派开发板的 RT-Thread BSP 支持包，该 BSP 中包含 MDK5 和 IAR 的工程，这里我们打开 MDK5 的工程`project.uvprojx`。

因为启动文件中使用了 `__main` 函数，在**调试的时候**需要在工程设置中开启使用`Use MicroLIB`，否则调试时会停留在启动文件中无法继续执行：

![mark](http://mculover666.cn/image/20190825/U209JYiVW4fl.png?imageslim)

然后点击编译按钮，编译小熊派BSP：

![mark](http://mculover666.cn/image/20190825/25V5BKbCpHGB.png?imageslim)

编译完成之后，编译信息如下：

![mark](http://mculover666.cn/image/20190825/fMALpn37B2P1.png?imageslim)

# 3.进入MDK的Debug模式

确保**开发板与PC之间已经正常连接**，点击 `Debug` 按钮，进入Debug模式：

![mark](http://mculover666.cn/image/20190825/EME5NH92Fdj8.png?imageslim)

点击复位按钮，程序跳至复位后的状态，整个MDK界面如图：

![mark](http://mculover666.cn/image/20190825/juLfh4k5LJxn.png?imageslim)

好啦，万事俱备，只欠东风，接下来我们开始探索RT-Thread的启动流程~

>Tips：在MDK的Debug模式下，按`F11`键单步执行代码。

# 4.一次愉快的探索之旅

## 复位处理程序
按下 `F11`，第一步执行的代码是复位处理程序，如下：
```c
; Reset handler
Reset_Handler    PROC
    EXPORT  Reset_Handler   [WEAK]
    IMPORT  SystemInit  //导入外部函数SystemInit
    IMPORT  __main      //导入外部函数__main

    LDR     R0, =SystemInit     //加载SystemInit的地址到R0
    BLX     R0                  //跳转到R0寄存器给出的地址中执行，然后返回
    LDR     R0, =__main         //加载__main函数
    BX      R0                  //跳转到R0寄存器给出的地址中执行，不返回
    ENDP
```
芯片上电复位后首先执行启动文件中的这段程序，在这段程序中首先调用 SystemInit 函数初始化系统时钟，然后调用C库函数 `__main` ，由该库函数调用 main 函数，去往 C 的世界~

## \$Sub\$\$main(MDK特有)
执行完复位处理程序后，再次按 `F11`，因为使用的是 MDK，所以执行下面这段程序：
```c
#if defined(__CC_ARM) || defined(__CLANG_ARM)
extern int $Super$$main(void);
/* re-define main function */
int $Sub$$main(void)
{
    rtthread_startup();
    return 0;
}
```
`$Sub$$main` 函数是 MDK 中的一个扩展功能，在调用 main 函数之前先调用`$Sub$$main` 完成系统启动，然后调用给用户的 main 函数。





