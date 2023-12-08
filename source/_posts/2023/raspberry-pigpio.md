---
title: 树莓派pigpio的使用
tags:
  - 树莓派
  - python
cover: /img/RUN.png
categories:
  - null
date: 2023-12-09 02:15:41
description: raspberry-pigpio
---

```py
import pigpio
import time
import os

#初始化
os.system("sudo pigpiod")
pi = pigpio.pi()

motor_gpio = 20
pulsewidth = 1300
#参数配置
pi.set_mode(motor_gpio,pigpio.OUTPUT)
pi.set_PWM_frequency(motor_gpio,200)
pi.set_PWM_range(motor_gpio,40000)
#解锁
pi.set_PWM_dutycycle(motor_gpio,10000)
time.sleep(2)
#正转
pi.set_PWM_dutycycle(motor_gpio,11000)
time.sleep(3)

#停转
pi.wave_tx_stop() 
pi.wave_clear()
pi.stop()
os.system("sudo killall pigpiod")

```


```
pi@raspberrypi:~ $ python3 -m pip3 install opencv-python
/usr/bin/python3: No module named pip3

pi@raspberrypi:~ $ sudo apt install python3-pip
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
Suggested packages:
The following NEW packages will be installed:
The following packages will be upgraded:
6 upgraded, 25 newly installed, 0 to remove and 129 not upgraded.
Need to get 56.8 MB of archives.
After this operation, 84.4 MB of additional disk space will be used.
Do you want to continue? [Y/n] y

pi@raspberrypi:~ $ python3 -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python
```

