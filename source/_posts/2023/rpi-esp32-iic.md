---
title: 树莓派和esp32通过iic通讯的底盘控制
tags:
  - null
cover: /img/
categories:
  - [计算机, 源码分享]
date: 2023-07-10 14:02:29
description: 树莓派通过iic与esp32通讯来控制电机转速，并据此计算小车底盘的位置信息
---
在[这篇文章](https://triority.cn/2023/esp32-all-in-one/#%E5%BA%94%E7%AD%94%E4%BA%A4%E4%BA%92%E9%80%9A%E8%AE%AF)我已经写过arduino uno和esp32通讯并控制电机速度，但是树莓派并没有封装好的Wire库，或者说我没找到，所以如果使用树莓派作为上位机这段程序需要重写。

树莓派速度发送程序（还没有进行位置计算）:
```
import time
import numpy as np
from smbus2 import SMBus
i2c = SMBus(1)
i2c.open(1)
while 1:
  inf = "0010!"
  adr = 0x08
  str_list = np.fromstring(inf,dtype=np.uint8)

  str_list=np.append(str_list,10)
  for i in str_list:
      #print(i)
      i = int(i)
      i2c.write_byte(adr,i,force=None)
  time.sleep(0.1)
```

esp32电机控制程序：
```
#include <SimpleFOC.h>
#include <Wire.h>
#include <dummy.h>

MagneticSensorI2C sensor = MagneticSensorI2C(AS5600_I2C);

BLDCMotor motor = BLDCMotor(11);
BLDCDriver3PWM driver = BLDCDriver3PWM(25, 26, 27, 14);

float target_velocity = 0;

TwoWire Wire_foc = TwoWire(0);
TwoWire Wire_rec = TwoWire(1);

String inString="";

void setup() {
  Serial.begin(115200);

  Wire_rec.setPins(16,17);
  Wire_rec.begin(0x08);
  Wire_rec.onReceive(receiveEvent);
  Wire_rec.onRequest(requestEvent);

  Wire_foc.setPins(33,32);
  Wire_foc.begin();
  sensor.init(&Wire_foc);

  motor.linkSensor(&sensor);

  driver.voltage_power_supply = 12;
  driver.init();

  motor.linkDriver(&driver);

  motor.controller = MotionControlType::velocity;

  motor.PID_velocity.P = 0.2f;
  motor.PID_velocity.I = 20;
  motor.PID_velocity.D = 0;
  motor.voltage_limit = 6;
  motor.PID_velocity.output_ramp = 1000;
  motor.LPF_velocity.Tf = 0.01f;

  motor.init();
  motor.initFOC();

  _delay(1000);
}

void loop() {
  motor.loopFOC();

  motor.move(target_velocity);
}

void receiveEvent(int howMany) {
    target_velocity = inString.toFloat()/100;
    char ch = Wire_rec.read();
    inString += ch;
    if (ch=='!'){
      inString[4]=0;
      target_velocity = inString.toFloat()/100;
      Serial.println(target_velocity);
      inString="";
    }
}

void requestEvent() {
  float get_ang = sensor.getAngle();
  int ang = int(get_ang*100);
  Serial.println(ang);
  char cstr[8];
  itoa(ang, cstr, 10);
  Wire_rec.write(cstr);
}

```