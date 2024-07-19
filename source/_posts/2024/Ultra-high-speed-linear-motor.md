---
title: 超高速直线电机
tags:
  - 电力电子
  - PCB设计
cover: /img/RUN.png
categories:
  - 值得一提的文章
  - 折腾记录
  - 作品&项目总结
date: 2024-07-02 21:37:22
description: 高功率高压高速直线电机
---
# 概述
这个项目是我大学本科四年做的最复杂的一个，从大一下学期到大三结束一路磕磕绊绊走走停停终于到了可以写总结的时候。

{% note danger modern %}
警告：本文设计内容具有危险性，不提供任何制造文件资料，仅供科研交流使用，装置测试后已经拆除，严禁仿造用于其他用途。
{% endnote %}

设计主要分为如下几个过程：
+ 材料选型
+ 电路设计
+ 机械结构
+ 参数优化仿真
+ 调试

说一下我的设计路线和目标：
+ 高压方案(>300V)
+ 快速连续工作(<1s)
+ 速度快(>100m/s)
+ 效率有追求但不很重要(>10%)

# 制作过程
## 材料选型
### 加速体
最开始测试使用的是M6x28的A3材质圆柱定位销，后来计算得知直径越大(管壁厚度占比越小)效率越高，同时28mm长度对于送料结构设计过于麻烦，因此方案变更为M8x20。

### 管道
管道有很多可以选择的材料，包括ABS塑料304不锈钢等。我选择的是外径9.5mm厚度0.5mm的304不锈钢，使用金属材料会导致涡流损耗，但是可以在使用更薄的厚度情况下承受更大的压力(减小管壁厚度提高效率)，且不易变形(我的ABS管就曾经因为焊接线圈时导热融化)

### 线圈
线圈设计较为麻烦，使用更少的匝数更粗的线可以减少电阻提高效率，但是相同匝数下体积变大成本变高，而且绕制线圈极其麻烦。较细的线圈可以有更多匝数，更多匝数就会有更大的电感和电阻从而降低峰值电流，减小开关器件的载流压力。如果匝数太多就会导致电感和电阻过大，功率不足同时关断过慢容易导致反拉。我使用的方案是0.6mm漆包铜线，匝数逐级减少，具体匝数等参数在参数优化仿真里讲吧

### 开关管
这个不同的电路拓扑需要使用不同的开关管，这里直接说每种管子选型。可控硅选用70tps16，IGBT选用IRGPS4067d。根据测试4067的抗脉冲性能极佳。

## 电路设计
### 功率电路
最开始我做的是晶闸管直接控制自由运行，好似大概十几年前非常流行，因为直接用可控硅简单耐造，但是问题显而易见，由于可控硅的不可靠关断，除非经过复杂的参数设计(谐振关断)否则就会反拉，效率极低。因此后来做了晶闸管换成IGBT的可控关断，但是经过仿真计算，效率提高并不太多，因为磁场能量还是经过电阻发热消耗掉了。最后做了BOOST拓扑，因为半桥拓扑需要两倍的开关管，成本高得多。如果将来做行波加速方案，那就还需要开关阵列，但是看起来我已经没有时间接着做下去了。所以我就主要说晶闸管自由运行方案(简单易懂)和BOOST拓扑方案，其他只做简要原理介绍

#### SCR自由运行和IGBT可控关断

这是我最开始做的版本的原理图和3D模型，现在看来三极管放大部分的参数需要一些修改但是还算能用

| ![](微信截图_20231014171245.png) | ![](微信截图_20231014171323.png) |
|:---:|:---:|
| 原理图 | 3D |

这个电路十分简单易懂，电容充电完成后，光耦电流经过三极管放大驱动SCR，电流经过线圈直到电容能量耗尽，电感电流通过上方二极管续流防止给电解电容反充。

这个电路缺点显而易见，SCR无法控制关闭，特别是后面速度高之后不可避免的反拉(除非精确控制电容电感和触发时序)，而讲这个电路的SCR换成IGBT即可解决问题(虽然要改一下驱动电路)，但是经过仿真计算，由于IGBT关断以后电感电流完全由二极管消耗发热，电流下降率低，依然容易导致反拉，所以效率和速度提升并不明显

#### 半桥拓扑能量回收

![](fbq.png)

为了解决上面的问题，半桥拓扑十分必要。上下管IGBT同时关闭后，电流从电感充回电容，电容剩余的电压将让电感的电流下降率更高，也就是反拉下降，同时能量回收，下一次充电更快。唯一缺点就是每一级需要2个开关管提高了成本。这个我没有动手做实物，就不再详细叙述了。

#### BOOST能量回收

![](微信截图_20240704123718.png)

上图即为boost拓扑的单级电路原理图，这个电路左右串联即可。上一级触发之后IGBT关断时，电感能量经过二极管进入下一级电容(这时下一级电容的电压将会高于初始充电电压，因此初始务必不要充满)。下一级电容此时仍然为满电状态，因此电感的电流下降率最高。

上图设计时我还没有放弃模块化，依然是每一级单独制作然后拼接而成，后面舍弃了这个方案因为互相连接如果焊接就失去了模块化的意义，而螺丝连接容易导致接触不良，因此后面就改为了连续的整体PCB设计

这一次的设计可以稳定工作，至少我测试了几十次没有出问题，可能也有电流比较保守，只给了200A以下的原因

| ![](微信截图_20240719201740.png) | ![](微信截图_20240719202048.png) | ![](f6e19e7d493377e57490c1ba6c95cbc.jpg) |
|:---:|:---:|:---:|
| 功率板原理图 | 驱动板原理图 | 照片 |
| <img width=2000/> | <img width=2000/> | <img width=2000/> |

本来设计用0.6mm的漆包线的，但是做的时候找不到了，改成手头有的0.45mm的，电阻变大了牺牲掉一些效率，不过也降低了电流提高了稳定性。线圈设计参数如下

![](微信截图_20240719202940.png)

实测速度基本稳定在45m/s，如果连续使用线圈发热增加的电阻率会降低电流导致速度降低，实测温度到40度时速度会降低到40左右，因此未来的设计将会关注热管理问题。

![](fb16ed462e5e7065995940dd605b93f.jpg)

代码在这里，时序使用循环计算，不再需要手动考虑高低电平切换顺序了，以及做了串口调参，节省一下flash的烧录寿命hhh

```c
//cxx: control[just a str], level, 0:duration/1:start time//2:end time

// start time
int c11_time = 0;
int c21_time = 1650;
int c31_time = 2400;
int c41_time = 2950;
// duration
int c10_time = 1460;
int c20_time = 840;
int c30_time = 670;
int c40_time = 580;

// trigger status
const int c1 = 26;
const int c2 = 27;
const int c3 = 14;
const int c4 = 12;
const int c5 = 13;
const int sw = 0;

unsigned long pulseStartTime = 0;
unsigned long pulseNowTime = 0;

String inData="";

void setup() {
  Serial.begin(115200);
  pinMode(c1, OUTPUT);
  pinMode(c2, OUTPUT);
  pinMode(c3, OUTPUT);
  pinMode(c4, OUTPUT);
  pinMode(c5, OUTPUT);
  digitalWrite(c1, LOW);
  digitalWrite(c2, LOW);
  digitalWrite(c3, LOW);
  digitalWrite(c4, LOW);
  digitalWrite(c5, LOW);

  pinMode(sw, INPUT);
}

void loop() {
  // serial
  while(Serial.available()>0){
    delay(5);
    char recieved = Serial.read();
    inData += recieved;
    if(recieved == '\n'){
      if (inData.length()<4){
        Serial.println("Too short");
        break;
      }
      String function = inData.substring(0, 4);

      if (function=="c11 "){
        String value_str = inData.substring(4, inData.length());
        c11_time = value_str.toInt();
        Serial.printf("Now c11 = ");
        Serial.println(c11_time);
      }else if (function=="c10 "){
        String value_str = inData.substring(4, inData.length());
        c10_time = value_str.toInt();
        Serial.printf("Now c10 = ");
        Serial.println(c10_time);
      }else if (function=="c21 "){
        String value_str = inData.substring(4, inData.length());
        c21_time = value_str.toInt();
        Serial.printf("Now c21 = ");
        Serial.println(c21_time);
      }else if (function=="c20 "){
        String value_str = inData.substring(4, inData.length());
        c20_time = value_str.toInt();
        Serial.printf("Now c20 = ");
        Serial.println(c20_time);
      }else if (function=="c31 "){
        String value_str = inData.substring(4, inData.length());
        c31_time = value_str.toInt();
        Serial.printf("Now c31 = ");
        Serial.println(c31_time);
      }else if (function=="c30 "){
        String value_str = inData.substring(4, inData.length());
        c30_time = value_str.toInt();
        Serial.printf("Now c30 = ");
        Serial.println(c30_time);
      }else if (function=="c41 "){
        String value_str = inData.substring(4, inData.length());
        c41_time = value_str.toInt();
        Serial.printf("Now c41 = ");
        Serial.println(c41_time);
      }else if (function=="c40 "){
        String value_str = inData.substring(4, inData.length());
        c40_time = value_str.toInt();
        Serial.printf("Now c40 = ");
        Serial.println(c40_time);
      }
      inData="";
    }

  }

  // switch
  if (digitalRead(sw)==LOW){
    // reset
    int c11 = 1;
    int c21 = 1;
    int c31 = 1;
    int c41 = 1;
    int c12 = 1;
    int c22 = 1;
    int c32 = 1;
    int c42 = 1;
    int c12_time = c11_time + c10_time;
    int c22_time = c21_time + c20_time;
    int c32_time = c31_time + c30_time;
    int c42_time = c41_time + c40_time;
    Serial.println("starting");
    pulseStartTime = micros();
    pulseNowTime = micros();
    while(c11||c12||c21||c22||c31||c32||c41||c42){
      pulseNowTime = micros();
      if(pulseNowTime-pulseStartTime>c11_time && c11){
        digitalWrite(c1, HIGH);
        c11 = 0;}
      if(pulseNowTime-pulseStartTime>c12_time && c12){
        digitalWrite(c1, LOW);
        c12 = 0;}

      if(pulseNowTime-pulseStartTime>c21_time && c21){
        digitalWrite(c2, HIGH);
        c21 = 0;}
      if(pulseNowTime-pulseStartTime>c22_time && c22){
        digitalWrite(c2, LOW);
        c22 = 0;}

      if(pulseNowTime-pulseStartTime>c31_time && c31){
        digitalWrite(c3, HIGH);
        c31 = 0;}
      if(pulseNowTime-pulseStartTime>c32_time && c32){
        digitalWrite(c3, LOW);
        c32 = 0;}

      if(pulseNowTime-pulseStartTime>c41_time && c41){
        digitalWrite(c4, HIGH);
        c41 = 0;}
      if(pulseNowTime-pulseStartTime>c42_time && c42){
        digitalWrite(c4, LOW);
        c42 = 0;}

      if(pulseNowTime-pulseStartTime>1000000){
        Serial.print("ERROR, time out");
        break;
      }
    }
    Serial.println("releasing");

    digitalWrite(c5, HIGH);
    delay(2000);
    digitalWrite(c5, LOW);
    Serial.println("finished");
  }
}

```

#### 开关阵列等其他
随着后续发展，出现了行波加速方式，即线圈宽度减少到直径以下，并且相邻多个线圈一起通电(起止时间不同)达到磁场近似均匀行进的效果，以平滑加速度使效率进一步提升，同时大幅提高了平均加速度。这种方式需要更更更多的级数(不到1cm的连续几十级)，因此开关管的数量变得巨大，于是显而易见应该尝试使用开关管组成矩阵多次触发减少使用量，目前这种拓扑还在实践中没有看到成熟的设计，现有行波加速依然是在使用boost拓扑

电路拓扑并不局限于上述几种，还有很多其他的类型，比如SCR的谐振关断，可以自行了解

### 电源
电源也一样有多种方案，我最开始使用了最简单的ZVS谐振电路逆变升压，这种方案十分简单但是不够可控可靠，无法控制工作状态，而且在负载过流情况下一旦停止谐振就会直接在电源上短路，如果使用电池供电十分危险，不过只要重新接入电源重新进入谐振状态就可以继续正常工作。另一种可控的方案就是使用开关电路逆变升压，较为复杂但是可控。当然我们的需求是给电容充电，使用恒流源更合适，所以后来参考了《一种谐振型推挽式直流变换器》

#### ZVS升压
不知道为什么这个电路被称为ZVS(零电压开关)电路，这只是一个恰好工作在零电压开关状态的谐振电路，不过这个名称和电路十分经典广为流传和使用，甚至淘宝所售套件的电源全都是这个方案，而且也足够简单，非常容易做，最开始大家应该都用的这个方案吧

| ![](微信图片_20231022221839_1.jpg) | ![](微信图片_20231022221839.jpg) | ![](微信图片_20231022221839_2.jpg) | ![](微信图片_20231022221839_3.jpg) |
| :---: | :---: | :---: | :---: |
| 洞洞板试验 | 输入 | 波形 | 输出 |
| <img width=2000/> | <img width=2000/> | <img width=2000/> | <img width=2000/> |

|![](微信截图_20240704130314.png)| ![](微信截图_20240704130544.png) | ![](微信图片_20231113160735.jpg) |
|:---:|:---:|:---:|
| 原理图 | 3D模型 | 照片 |

缺失模型的元件是电感和变压器。电感一定要电流足够大，分享一下我的电感和变压器选型吧

| ![](80744d6641f6b1eeb3cd457683ebc8e.jpg) | ![](055435e79723270fdaf48b5150b58e2.jpg) | ![](11fc579bfb3f5c95989dac205c9dcf4.jpg) |
| :---: | :---: | :---: |
| 电感 | 现在用的电容 | 打算以后换的电容 |
| <img width=2000/> | <img width=2000/> | <img width=2000/> |


这个电路的问题我也说过了，就是一旦过载停振就会短路，所以我在前面加了一个PMos控制电源开关，短路可以及时关闭和重启。


#### 开关电源
这种方案使用芯片产生的指定频率的互补的PWM波形控制MOS推挽升压，至少不会停振短路了。我用的是SG3525产生信号然后驱动MOS。



#### 全桥恒流谐振
这种方案我还没来得及实验，直接上论文吧

{% pdf 一种谐振型推挽式直流变换器_袁义生.pdf %}

## 参数优化仿真
除非使用光电检测位置触发的方案，否则时序控制是必然的，时序控制更加可控可靠。

时序的计算流程一般是先由RLC工具计算得到最佳的触发位置，然后在Maxwell中根据位置触发仿真计算时间，然后再进行一些调整调试。

### RLC参数仿真

### Maxwell时序仿真

## 调试

# 特别感谢

[科创网](https://www.kechuang.org/f/367)
[JLC](https://www.jlc.com/)
**QQX**
