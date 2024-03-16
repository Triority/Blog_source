---
title: sdruno使用说明
tags:
  - 无线电
cover: /img/8df36049471386e523f41dbefb70d8d.jpg
categories:
  - 文档&笔记
  - 无线电
date: 2024-03-16 12:51:14
description: sdruno简明教程
---
# sdr(Software Defined Radio)设备
前段时间买了RSP1接收机，搭配天线是直径大概在40cm的小环有源天线(用于短波段)挂在我宿舍的衣柜上，然后从`BI1PTK`那里白嫖了一根大概45cm的中心频率在u段和v段玻璃钢天线，架设在宿舍阳台上。
# sdr软件
sdr常用软件一般是`sdr#`(sdrsharp)和`sdruno`，sdr#有汉化版本，但是由于某些未知原因我的电脑使用sdr#无法读取到sdr设备，因此只能使用sdruno。

`sdruno`是`sdrplay`官方支持的接收软件，但是没有中文版本，而且功能按钮众多难以记忆，广泛流传的中文说明明显是机器翻译难以使用，因此写一篇常用功能的使用说明。

`sdruno`的GUI：

![](8ee1a3e43724e841907bb7aff58b8b2.png)

# 操作面板
## main(主控面板)
![](main.png)

最上面的`PLUGINS`可以打开[插件面板](#plugins插件已加载rec插件)

左上角的`OPT`可以修改操作面板设置，例如将窗口恢复默认排布。当你调整好了窗口之后可以按`ctrl+w`保存你当前的窗口设置。`SCAN``SCHEDULER``SP1``SP2``RX`分别可以打开其他几个窗口，分别是扫描器大小瀑布和接收控制面板

右侧`ADD VRX`和`DEL VRX`可以在同频段内设置多个收听频率。
`LO LOCK`可以锁定瀑布中的频率选择线。开关可以选择是移动频率选择线还是水平频率坐标轴。

`IF MODE`设置为`ZIF`时可以在右侧调节总频宽(图中为10M宽度)

`PLAY`和`STOP`可以打开必应翻译查看按键功能。

## rx control(接收控制)
![](rx.png)

上面最显眼的就是频率，可以鼠标滚轮调节每一位数字。下面可以选择各种接收模式，`AM`调幅，`SAM`，`FM`调频，`CW`短幅报，`DSB`，`LSB`，`USB`，`DIGTIAL`

如果选择`FM`下面需要选择`NFM`还是`WFM`等

如果选择`CW`下面的`CW OP`可以设置

右侧`FILTER`可以设置

`NB`可以选择

最右侧`NORCH`陷波器可以添加最多四个陷波器

左下侧`MUTE`可以一键静音

`SQLC`可以设置静噪，`VOLUME`设置音量

右下侧`AGC`

面板最右侧可以快捷调整接收波段

## rx ex(其他接收控制选项)
![](rx_ex.png)


`PDBPF`带通滤波器可以设置音频输出频率范围到`LC`和`HC`，可以用于去除模拟哑音信号

## main sp(大瀑布)
![](main_sp.png)

## aux sp(小瀑布)
![](aux.png)

## plugins(插件,已加载rec插件)
![](plugins_rec.png)
