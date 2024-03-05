---
title: 使用无线电台模拟音频信号传输文本数据
tags:
  - arduino
  - esp32
  - 无线电
cover: /img/RUN.png
categories:
  - - 折腾记录
  - - 无线电
date: 2024-03-05 19:05:13
description: 用于未来可能的一次神秘小活动
---
# 实现目标
以较低成本实现无线电台传输少量数据

未来可能放一个探空气球，计划应该让氦气球在平流层爆炸并使得负载降落，但是高空没有基站信号，如果等降落到地面之后使用4G信号返回位置数据，有可能降落到基站信号范围之外无法找回，因此使用无线电台就是最佳解决方案。、

同时由于有可能找不回负载设备，所以希望尽量降低负载成本，可以传输数据的电台又过于昂贵因此出此下策：使用模拟手台语音通联，直接esp32产生正弦波音频信号通过手台发送，然后手台接收进行解码得到位置数据。

# 空中发射
下面代码可以实现esp32使用dac外设发送500Hz的正弦信号

需要注意的是由于esp32的dac性能限制，信号频率最好不要高于1k，否则失真明显

| ![](009.BMP)  | ![](001.BMP)  | ![](005.BMP)  | ![](006.BMP)  |
| :------------: | :------------: | :------------: | :------------: |
| 2000Hz  | 1000Hz  |  500Hz |  250Hz |

```c
hw_timer_t *timer = NULL;

unsigned long angle = 0;
int value;
//设置为100时输出正弦波频率为500hz
int add_value = 100;

//中断函数
void IRAM_ATTR InterruptEvent(){
  angle = angle + add_value;
}

void setup(){
  Serial.begin(115200);
  //定时器初始化，esp32频率为80Mhz，分频80则时间单位为1Mhz
  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &InterruptEvent, true);
  //32us中断一次(实际应该31.415926us这样设置会导致实际频率偏低一丢丢)
  timerAlarmWrite(timer, 32, true);
  timerAlarmEnable(timer);
}

void loop(){
  value = sin(angle/1000.0)*63+192;
  dacWrite(25, value);
  if(angle>=31415926){angle = 0;}
}

```
显然我们只需要对这个信号通过调频方式编码即可，实测输出信号频偏在2%以内，保守起见可以一次传输4位，也就是对应16个频率，每秒传输200次，也就是100个字节每秒，完全可以满足GPS位置信息的传输速率要求。

