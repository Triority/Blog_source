---
title: klipper做上位机diy的3D打印机
date: 2022-08-31 22:49:36
tags:
- 3D打印
excerpt: 一切开始于fluidd页面真的帅啊hhhhhhhc
index_img: /img/klipper.png
categories: 
- [计算机, 折腾记录]
- [计算机, CV工程师]
---
## 前言
这里主要讲上位机klipper的配置，硬件使用大鱼的设计，主板mks genl2.1，klipper安装于上位机。
mks的说明书已经详细说明了klipper的搭建配置过程：[MKS GEN_L V2.1 Klipper固件使用说明书](https://blog.csdn.net/gjy_skyblue/article/details/121103193 "MKS GEN_L V2.1 Klipper固件使用说明书")
![MKS GEN_L V2.1](mksgenl21.png)
## 对mks配置文件的修改
### lcd
我没有使用lcd屏，但是mks提供的配置文件`printer.cfg`对lcd进行了配置，需要删除，这一点说明书没有提及，lcd配置include了其他文件，文件在mks的github仓库可以找到。
### pin
我为打印机加装了LED灯带，但是不希望灯一直打开，需要开关功能。由于我没有lcd屏幕，所以可以使用lcd的引脚控制L298N驱动LED灯带。
klipper使用芯片的引脚名称，与marlin不同，mks的github仓库虽然给出来对应关系但是并不完整，具体引脚编号可以查看主板原理图上mega2560芯片的内容。
原理图：
![SCH](SCH.png)
引脚图：
![PIN](PIN.png)

### 我的配置文件
printer.cfg
```
# MKS Gen l V2.1

[stepper_x]
step_pin: PF0
dir_pin: PF1
enable_pin: !PD7
microsteps: 16
rotation_distance: 40   ##rotation_distance = ((360°/1.8°) * microsteps) / 80 # # 旋转距离 = （圆周360°/步距角）*细分/每MM脉冲值
endstop_pin:^!PE5 #X-Min, PE4:X-Max
position_endstop: 0
position_max: 295
homing_speed: 30

[stepper_y]
step_pin: PF6
dir_pin: PF7
enable_pin: !PF2
microsteps: 16
rotation_distance: 40
endstop_pin:^!PJ1  #Y-Min, PJ0:Y-Max
position_endstop: 0
position_max: 205
homing_speed: 30

[stepper_z]
step_pin: PL3
dir_pin: !PL1
enable_pin: !PK0
microsteps: 16
rotation_distance: 8
endstop_pin: probe:z_virtual_endstop  #Z-Min, PD2:Z-Max
position_max: 280
position_min: -3

[extruder]
step_pin: PA4
dir_pin: PA6
enable_pin: !PA2
microsteps: 16
rotation_distance: 7
nozzle_diameter: 0.4
filament_diameter: 1.750
heater_pin: PB4
sensor_type: ATC Semitec 104GT-2
sensor_pin: PK5
min_temp: 0
max_temp: 270
control: pid
pid_Kp: 21.438
pid_Ki: 0.888
pid_Kd: 129.435
max_extrude_only_distance: 500.0

[verify_heater extruder]
max_error: 1200
hysteresis: 20
check_gain_time: 20
heating_gain: 1

#[extruder_stepper extra_stepper]
#step_pin: PC1
#dir_pin: !PC3
#enable_pin: !PC7
#microsteps: 16
#rotation_distance: 8
#endstop_pin: ^!PE4
#position_endstop: 0
#position_max: 250
#position_min: -3


[heater_bed]
heater_pin: PH5
sensor_type: ATC Semitec 104GT-2
sensor_pin: PK6
min_temp: 0
max_temp: 130
control: pid
pid_kp = 74.551
pid_ki = 1.053
pid_kd = 1319.559

[verify_heater heater_bed]
max_error: 12000
hysteresis: 30
check_gain_time: 300
heating_gain: 1

[fan]
pin: PH6


[mcu]
serial:/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0


[printer]
kinematics: corexy
max_velocity: 1000
max_accel: 3000
max_z_velocity: 10
max_z_accel: 500
square_corner_velocity: 10.0


[fan_generic LED]
pin: PL0
shutdown_speed: 1.0

[fan_generic X舵机]
pin: PG5
max_power: 0.125
cycle_time: 0.02

[fan_generic Y舵机]
pin: PE3
max_power: 0.125
cycle_time: 0.02

[virtual_sdcard]
path: ~/gcode_files

[pause_resume]

[gcode_macro CANCEL_PRINT]
description: Cancel the actual running print
rename_existing: CANCEL_PRINT_BASE
gcode:
  TURN_OFF_HEATERS
  CANCEL_PRINT_BASE


[display_status]

[bltouch]
sensor_pin: ^PD2
control_pin: PB5
x_offset: 36
y_offset: 0
z_offset: 3.550
speed: 5.0
samples: 2
samples_result: median
sample_retract_dist: 3.0
samples_tolerance: 0.1
samples_tolerance_retries: 1

[safe_z_home]
home_xy_position: 150,100 # Change coordinates to the center of your print bed
speed: 50
z_hop: 10                 # Move up 10mm
z_hop_speed: 5


[gcode_macro G29]
gcode:
    BED_MESH_CLEAR
    G28
    BED_MESH_CALIBRATE
    BED_MESH_PROFILE SAVE=name
    SAVE_CONFIG
    BED_MESH_PROFILE LOAD=name

[bed_mesh]
speed: 100
horizontal_move_z: 5
mesh_min:60,40
mesh_max:260,160
probe_count: 3,3

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	  -0.001250, -0.028750, 0.018750
#*# 	  0.003750, 0.007500, 0.065000
#*# 	  -0.077500, -0.108750, -0.005000
#*# tension = 0.2
#*# min_x = 60.0
#*# algo = lagrange
#*# y_count = 3
#*# mesh_y_pps = 2
#*# min_y = 40.0
#*# x_count = 3
#*# max_y = 160.0
#*# mesh_x_pps = 2
#*# max_x = 260.0

```

## 打印参数优化
