---
title: 使用全新atmega328p芯片自制arduino开发板
tags:
  - arduino
cover: /img/QQ截图20230426223142.png
categories:
- [计算机, 折腾记录]
notshow: false
date: 2023-04-26 22:33:58
excerpt: 使用全新atmega328p芯片自制arduino开发板
---
## first
最近做了一个4010无刷电机foc驱动板，pcb直接放在电机下面，在第二个版本中希望把主控芯片集成到驱动板上，减少驱动板接线数量(原来需要三个pwm信号线+两个电源线+2个编码器iic数据时钟线)，顺便做个spi总线控制，同时控制多个电机。

## atmega328p需要的外围电路
![只要有这些其实就已经可以工作了](QQ截图20230426223142.png)

除此之外使用isp烧录还需要6个引脚引出：
![isp接口](QQ截图20230426223111.png)

## 烧录步骤
首先需要一个能够正常使用的arduino开发板作为烧录器，烧录好示例程序`arduinoISP`
![arduinoISP](arduinoisp.jpg)
这里先给一个uno板子的照片:
![arduino uno](v2-fa59cd9d15becb2a3ba02817ec09395b_720w.webp)
右边2x3=6的排针就是isp烧录接口，引脚定义在示例文件也已经给出：
```
// By default, the hardware SPI pins MISO, MOSI and SCK are used to communicate
// with the target. On all Arduinos, these pins can be found
// on the ICSP/SPI header:
//
//     MISO °. . 5V (!) Avoid this pin on Due, Zero...
//     SCK   . . MOSI
//           . . GND
//
```
烧录好做个程序之后就可以与要烧录的pcb连线了，注意MISO和MOSI的顺序不能错，否则会上传失败

作为被烧录接口的剩余引脚也就是RST，接烧录器的D10引脚。

然后设置编程器为`arduino as isp`：
![arduino as isp](arduino_as_isp.jpg)

然后就可以选择使用编程器上传了：
![使用编程器上传](使用编程器上传.jpg)

附送一张我的PCB的照片
![4010电机驱动板](ff5086b587210735df80cb063875b18.jpg)