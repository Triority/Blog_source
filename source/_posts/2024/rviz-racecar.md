---
title: ros-rviz赛车
tags:
  - null
cover: /img/RUN.png
categories:
  - 整活&游戏
  - 折腾记录
date: 2024-03-27 21:31:29
description: 在rviz里运行的赛车游戏
---
# 突发奇想
可能是作业留少了，突然就想写这么个东西：一个在ros中运行的赛车游戏，最好可以多人游戏的那种，使用rviz进行显示

# 阶段任务
## 地图
首先作为一个赛车游戏，地图是必不可少的，这个地图需要一个程序来自动生成，同时自动记录路线检查点便于计算成绩。地图的加载等可以直接用`map_server`，生成地图打算写个python程序来完成。


## 游戏视角
如果只是让AI去跑地图那只需要一个俯视视角，但是如果需要玩家第一人称视角参与，需要程序自动调节rviz的视角。先上个链接：[rviz_animated_view_controller](http://wiki.ros.org/rviz_animated_view_controller)


## 物理逻辑
赛车运行逻辑模拟现实的阿克曼结构，控制上有6个选项，无操作(减速)/加速/左转(减速)/右转(减速)/左转并加速/右转并加速，同时地图的白色区域为可以行进的区域，黑色区域为地图外，离开白色区域直接死亡。

## 自动驾驶

