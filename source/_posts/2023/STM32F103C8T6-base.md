---
title: STM32基础：以STM32F103C8T6核心板为例
tags:
  - stm32
index_img: /img/STM32F103C8T6.png
categories:
- [计算机, 知识整理]
notshow: false
date: 2023-04-17 15:25:47
excerpt: stm32基础学习笔记
---
# 简介
![STM32F103C8T6是一款由意法半导体公司(ST)推出的基于Cortex-M3内核的32位微控制器，采用`LQFP48`封装](STM32F103C8T6.png)

![单片机命名规则](20210115142206386.png)

[STM32F103系列中文芯片手册](https://cr.triority.cn/f/1r0uP/STM32F103%E4%B8%AD%E6%96%87%E6%95%99%E7%A8%8B%E5%8F%8A%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C.pdf)

# 使用arduino编程
突然得知stm32可以用arduino编程，喜出望外hhh

添加开发板管理器地址
```
http://dan.drown.org/stm32duino/package_STM32duino_index.json
```
然后就可以找到`STM32F103C`来给我的板子编程了。

但是usb接口不能串口下载就很烦了，不如直接用st-link，不是很想用emmmm

# 使用keil的程序编写和下载
## 下载方式
STM32单片机支持3种程序下载方式
+ ISP串口下载(使用USB-TTL接PA9、PA10)
+ SWD下载(使用ST-LINK接PA13、PA14)
+ JTAG下载(使用JLINK接PA13、PA14、PA15、PB3、PB4)

虽然有三种方式，但是我个人一般是使用SWD的，有时候也会选择ISP，所以下面介绍一下这两种
### ISP下载
使用ISP串口下载前，将单片机上电之前需要先用跳线帽把`BOOT0`短接到`1`的位置，`BOOT1`短接到`0`的位置，即系统存储器模式，然后才能通过串口下载程序。ISP串口下载完成后断电，在单片机上电之前需要先用跳线帽把`BOOT0`短接到`0`的位置，即主闪存存储器模式。

![启动模式](启动模式选择.png)

下载器GND与单片机GND相连，下载器3.3V与单片机3.3V相连(或者下载器5V与单片机VIN相连)、下载器RXD与单片机PA9(U1TX)相连，下载器TXD与单片机PA10(U1RX)相连

### SWD下载
使用SWD接口下载只需要连接3.3V、GND、`SWDIO(PA13)`、`SWCLK(PA14)`、`RST`(非必要)，可以从淘宝购买`ST-LINK`下载器。使用SWD接口除了可以烧录程序外，还可以实现在线仿真(debug)，仿真过程可以监视寄存器等数据，非常适合软件开发(找问题)。`ST-LINK/V2`只支持给自家的STM32和STM8烧录程序，不支持为其他公司的单片机烧录程序(即使同样搭载`Cortex-M3`内核)

### JTAG下载
这种方式很少使用，不再详细叙述

如果我们不需要使用JTAG下载，但GPIO资源紧张或PCB设计时已经使用了这些第一功能为JTAG的引脚，那么我们就需要关闭JTAG。比如说我要使用GPIOA15作为GPIO口，那么代码层面需要这样实现：
```
  GPIO_InitTypeDef GPIO_InitStructure;
 	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA|RCC_APB2Periph_AFIO,ENABLE);//使能PORTA时钟
	GPIO_PinRemapConfig(GPIO_Remap_SWJ_JTAGDisable , ENABLE);// 关闭JTAG但使能SWD
	GPIO_InitStructure.GPIO_Pin  = GPIO_Pin_15;//PA15
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU; //设置成上拉输入
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz; 
 	GPIO_Init(GPIOA, &GPIO_InitStructure);//初始化GPIO
```
## 编程方法
