---
title: STM32基础：以STM32F103C8T6核心板为例
tags:
  - stm32
cover: /img/STM32F103C8T6.png
categories:
- [计算机, 知识整理]
notshow: false
date: 2023-04-17 15:25:47
description: stm32基础学习笔记
---
# 简介
![STM32F103C8T6是一款由意法半导体公司(ST)推出的基于Cortex-M3内核的32位微控制器，采用`LQFP48`封装](STM32F103C8T6.png)

![单片机命名规则](20210115142206386.png)

[STM32F103系列中文芯片手册](https://cr.triority.cn/f/1r0uP/STM32F103%E4%B8%AD%E6%96%87%E6%95%99%E7%A8%8B%E5%8F%8A%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C.pdf)

# 下载方式
STM32单片机支持3种程序下载方式
+ ISP串口下载(使用USB-TTL接PA9、PA10)
+ SWD下载(使用ST-LINK接PA13、PA14)
+ JTAG下载(使用JLINK接PA13、PA14、PA15、PB3、PB4)

虽然有三种方式，但是我个人一般是使用SWD的，所以下面主要介绍这一个
## ISP下载(串口)
使用ISP串口下载前，将单片机上电之前需要先用跳线帽把`BOOT0`短接到`1`的位置，`BOOT1`短接到`0`的位置，即系统存储器模式，然后才能通过串口下载程序。ISP串口下载完成后断电，在单片机上电之前需要先用跳线帽把`BOOT0`短接到`0`的位置，即主闪存存储器模式。

![启动模式](启动模式选择.png)

下载器GND与单片机GND相连，下载器3.3V与单片机3.3V相连(或者下载器5V与单片机VIN相连)、下载器RXD与单片机PA9(U1TX)相连，下载器TXD与单片机PA10(U1RX)相连

## SWD下载(st-link)
使用SWD接口下载只需要连接3.3V、GND、`SWDIO(PA13)`、`SWCLK(PA14)`、`RST`(非必要)，可以从淘宝购买`ST-LINK`下载器。使用SWD接口除了可以烧录程序外，还可以实现在线仿真(debug)，仿真过程可以监视寄存器等数据，非常适合软件开发(找问题)。`ST-LINK/V2`只支持给自家的STM32和STM8烧录程序，不支持为其他公司的单片机烧录程序(即使同样搭载`Cortex-M3`内核)

## JTAG下载
这种方式很少使用，不再详细叙述

> 如果我们不需要使用JTAG下载，但GPIO资源紧张或PCB设计时已经使用了这些第一功能为JTAG的引脚，那么我们就需要关闭JTAG。比如说我要使用GPIOA15作为GPIO口，那么代码层面需要这样实现：
> ```
>   GPIO_InitTypeDef GPIO_InitStructure;
>  	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_AFIO,ENABLE);//使能PORTA时钟
> 	GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable , ENABLE);// 关闭JTAG但使能SWD
> 	GPIO_InitStructure.GPIO_Pin  = GPIO_Pin_15;//PA15
> 	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU; //设置上拉输入
> 	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz; 
>  	GPIO_Init(GPIOA, &GPIO_InitStructure);//初始化GPIO
> ```

# 使用arduino编程
突然得知stm32可以用arduino编程，喜出望外hhh

添加开发板管理器地址
```
http://dan.drown.org/stm32duino/package_STM32duino_index.json
```
然后就可以找到`STM32F103C`来给我的板子编程了。


# 使用keil编程
## 软件下载安装
`Keil uVision`:编程工具，可以在官网注册获取下载链接，可能需要自行搜索破解
`STM32 ST-LINK Utility`：配套`ST-LINK`一起使用的烧录工具，包含`ST-Link`驱动，同样可以在官网下载，我也[copy的一份在网盘上](https://cr.triority.cn/f/2L8tv/STM32-ST-LINK-Utility-setup.exe)

## 工程新建编译和烧录
首先选择开发板芯片，我的是`stm32f103c8`

然后是选择运行环境，必选`CMSIS`的`Core`还有`Device`的`Startup`，如果要连接外设必须勾选外设的时钟`RCC`，一般再勾选上`Framework`、`GPIO`、和`USART`串口，其他可以暂时不选，这个随时可以改

确定创建项目，然后添加`main.c`文件，之后就可以在main文件中写代码了，写完可以编译一下，如果输出正确就表示环境配置没问题

这里编译默认是不会创建`Hex`文件的，所以还需要进入设置里面去设置一下，如下图
![设置生成hex文件](QQ截图20230511175030.png)

将`STLink`连接到单片机上并插入电脑，其上的LED指示灯用于提示当前的工作状态，具体情况如下：
+ LED 闪烁红色：`STLink`已经连接至计算机。
+ LED 保持红色：计算机已经成功与`STLink`建立通信连接。
+ LED 交替闪烁红色和绿色：数据正在传输。
+ LED 保持绿色：最后一次通信是成功的。
+ LED 为橘黄色：最后一次通信失败。

在`STM32 ST-LINK Utility`中，首先连接芯片`(Tarage -> connect或直接点击连接快捷按钮)`，然后打开`hex`文件`(也可以直接讲hex文件拖动到FLASH区域)`，最后就可以下载程序`(Taraget -> Program，也可以直接点击下载快捷按钮)`

弹出信息确认窗口，如hex文件路径、验证方式等，确认信息无误后点击`Start`开始下载程序，出现`Verification…OK`，说明下载成功。
## ST-Link仿真
刚开始学还用不到调试功能，挖大坑，以后来填

## 编程
### GPIO操作(各种点灯)
#### 点亮
在GPIO输出之前要先对要操作的GPIO进行配置，下面这个程序可以连续将PC13这个引脚拉低拉高:

```
# include "stm32f10x.h"
void LED_Init(void){
    //A
    GPIO_InitTypeDef GPIO_InitStructure;
    //B
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
    //C
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    //D
    GPIO_Init(GPIOC, &GPIO_InitStructure);
}

int main(){
    LED_Init();
    while(1){
      //E
      GPIO_ResetBits(GPIOC,GPIO_Pin_13);
      GPIO_SetBits(GPIOC,GPIO_Pin_13);
    }
}
```
下面来解释一下这个程序:

> A:定义GPIO的初始化类型结构体

```
GPIO_InitTypeDef GPIO_InitStructure;
```
此结构体的定义是在`stm32f10x_gpio.h`文件中，其中包括3个成员：
+ `uint16_t GPIO_Pin;`来指定GPIO的哪个或哪些引脚
+ `GPIOSpeed_TypeDef GPIO_Speed;`GPIO的速度配置,对应3个速度：10MHz、2MHz、50MHz
+ `GPIOMode_TypeDef GPIO_Mode;`为GPIO的工作模式配置，即GPIO的8种工作模式。
  + 输入浮空 `GPIO_Mode_IN_FLOATING`
  + 输入上拉 `GPIO_Mode_IPU`
  + 输入下拉 `GPIO_Mode_IPD`
  + 模拟输入 `GPIO_Mode_AIN`
  + 具有上拉或下拉功能的开漏输出 `GPIO_Mode_Out_OD`
  + 具有上拉或下拉功能的推挽输出 `GPIO_Mode_Out_PP`
  + 具有上拉或下拉功能的复用功能推挽 `GPIO_Mode_AF_PP`
  + 具有上拉或下拉功能的复用功能开漏 `GPIO_Mode_AF_OD`

> B:使能GPIO时钟

```
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
```
此函数是在`stm32f10x_rcc.c`文件中定义的。其中第一个参数指要打开哪一组GPIO的时钟，取值参见`stm32f10x_rcc.h`文件中的宏定义，第二个参数为打开或关闭使能，取值参见`stm32f10x.h`文件中的定义，其中`ENABLE`代表开启使能，`DISABLE`代表关闭使能。
```
void RCC_APB2PeriphClockCmd(uint32_t RCC_APB2Periph, FunctionalState NewState);
```
> C:设置GPIO_InitTypeDef结构体三个成员的值
```
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
```
> D:初始化GPIO
```
GPIO_Init(GPIOC, &GPIO_InitStructure);
```
> E:GPIO电平输出

函数就是置位GPIO，即让相应的GPIO输出高电平
```
GPIO_ResetBits(GPIOC,GPIO_Pin_13);
GPIO_SetBits(GPIOC,GPIO_Pin_13);
```

很多网上找到的程序也会这样做，在文件开头写
```
#define LED3_OFF GPIO_SetBits(GPIOB,GPIO_Pin_5)
#define LED3_ON GPIO_ResetBits(GPIOB,GPIO_Pin_5)
```
然后在调用时候就可以直接写
```
LED3_ON;
LED3_OFF;
```
#### PWM信号输出

# 参考资料
https://blog.csdn.net/xiaoshihd/article/details/110039281
