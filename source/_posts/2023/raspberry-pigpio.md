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

```
import pigpio
import time
pi = pigpio.pi()

user_gpio = 20
pulsewidth = 1300
pi.set_mode(20,pigpio.OUTPUT)
pi.set_PWM_frequency(user_gpio,200)
pi.set_PWM_range(user_gpio,40000)
#解锁
pi.set_PWM_dutycycle(user_gpio,10000)
time.sleep(2)
#正转
pi.set_PWM_dutycycle(user_gpio,11000)
time.sleep(3)            # 延迟10秒
pi.wave_tx_stop() 
pi.wave_clear()
pi.stop()

```