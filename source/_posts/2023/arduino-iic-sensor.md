---
title: 使用arduino的同轴麦轮自平衡车[正在鸽子]
tags:
  - arduino
index_img: /img/
categories:
- [计算机, 源码分享]
- [计算机, 折腾记录]
notshow: false
date: 2023-02-25 23:28:44
excerpt: 使用arduino的同轴麦轮自平衡车
---
## 硬件部分
### 获取传感器sensor数据
#### imu:mpu6050
[`MPU6050_tockn`库](https://github.com/tockn/MPU6050_tockn)的示例代码：
```

#include <MPU6050_tockn.h>
#include <Wire.h>

MPU6050 mpu6050(Wire);

long timer = 0;

void setup() {
  Serial.begin(9600);
  Wire.begin();
  mpu6050.begin();
  mpu6050.calcGyroOffsets(true);
}

void loop() {
  mpu6050.update();

  if(millis() - timer > 1000){
    
    Serial.println("=======================================================");
    Serial.print("temp : ");Serial.println(mpu6050.getTemp());
    Serial.print("accX : ");Serial.print(mpu6050.getAccX());
    Serial.print("\taccY : ");Serial.print(mpu6050.getAccY());
    Serial.print("\taccZ : ");Serial.println(mpu6050.getAccZ());
  
    Serial.print("gyroX : ");Serial.print(mpu6050.getGyroX());
    Serial.print("\tgyroY : ");Serial.print(mpu6050.getGyroY());
    Serial.print("\tgyroZ : ");Serial.println(mpu6050.getGyroZ());
  
    Serial.print("accAngleX : ");Serial.print(mpu6050.getAccAngleX());
    Serial.print("\taccAngleY : ");Serial.println(mpu6050.getAccAngleY());
  
    Serial.print("gyroAngleX : ");Serial.print(mpu6050.getGyroAngleX());
    Serial.print("\tgyroAngleY : ");Serial.print(mpu6050.getGyroAngleY());
    Serial.print("\tgyroAngleZ : ");Serial.println(mpu6050.getGyroAngleZ());
    
    Serial.print("angleX : ");Serial.print(mpu6050.getAngleX());
    Serial.print("\tangleY : ");Serial.print(mpu6050.getAngleY());
    Serial.print("\tangleZ : ");Serial.println(mpu6050.getAngleZ());
    Serial.println("=======================================================\n");
    timer = millis();
    
  }

}

```
#### 磁编码器:as5600(实际做好像没用到)
[`AS5600`库](https://github.com/Seeed-Studio/Seeed_Arduino_AS5600)的示例代码：
```
#include "AS5600.h"
#include "Wire.h"

AS5600 as5600;   //  use default Wire


void setup()
{
  Serial.begin(115200);
  Serial.println(__FILE__);
  Serial.print("AS5600_LIB_VERSION: ");
  Serial.println(AS5600_LIB_VERSION);

  //  ESP32
  //  as5600.begin(14,15);
  //  AVR
  as5600.begin(4);  //  set direction pin.
  as5600.setDirection(AS5600_CLOCK_WISE);  // default, just be explicit.
  int b = as5600.isConnected();
  Serial.print("Connect: ");
  Serial.println(b);
  delay(1000);
}


void loop()
{
  //  Serial.print(millis());
  //  Serial.print("\t");
  Serial.print(as5600.readAngle());
  Serial.print("\t");
  Serial.println(as5600.rawAngle());
  //  Serial.println(as5600.rawAngle() * AS5600_RAW_TO_DEGREES);

  delay(100);
}

```
## 软件部分
### 同轴麦轮速度分量计算

![大半夜乱涂乱画](photo_2023-02-26_06-19-33.jpg)

其中x和y为平移速度，w为旋转角速度，d和D为轮子与中心距离。
PID平衡控制的值直接加上去就好。

### PID
![PID流程图](7528d75d2ff7922206c42b53b15c26e.png)

使用速度和角度的串级PID控制。