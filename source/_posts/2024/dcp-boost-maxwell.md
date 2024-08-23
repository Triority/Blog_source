---
title: 使用maxwell进行boost拓扑的DCP仿真
tags:
  - 电磁炉
cover: /img/微信截图_20240216120300.png
categories:
- 折腾记录
date: 2024-08-23 11:49:35
description: 使用外电路作为激励源的DCP仿真模型
---
# 视频演示
之前写的教程比较粗略，这里出一个详细步骤的视频

{% raw %}
<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;">
<iframe src="//player.bilibili.com/player.html?isOutside=true&aid=113007065564118&bvid=BV168WEeBEmV&cid=500001658823200&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; Left: 0; top: 0;" ></iframe></div>
{% endraw %}

# 一些陈旧的晦涩难懂的内容
## 建立模型
首先在`maxwell 2D`中选择求解模式，DCP模型应为绕z轴旋转的瞬态模型
![](solution_type.png)

然后绘制2D模型并设置材料
![](model.png)
> egg实际使用的是A3碳钢，线圈就直接选用铜了。实际上在初级阶段只有不到一两百安这种小电流情况下可以使用铝线来减轻重量。

创建分析域(creat region)，这里就选择z方向200其他100

## 添加外电路(激励源)
为线圈添加激励，首先定义`coil`先设置为300匝，然后定义他的`Winding`为外部激励的`stranded`，就可以将coil `add to winding`了
![](assign_excitation.png)
> 在稳态仿真中可以直接使用电流做激励源，这里使用外电路因此需要定义线圈绕组

然后为绕组生成外部电路
![](creat_circuit.png)

画好电路之后导出并导入到激励
![](make_sph.png)

## 准备仿真
添加边界：先将选择模式改为`edge`边缘，然后选择三个边缘边之后右键`assign boundary/Ballon`

然后定义网格：选中两个物体之后右键`Assign Mesh Operation/Inside Selection/Length Based...`，这里我就直接设置为3mm了

设置分析步长：右键`project manager`中的`Analysis`，选择`add solution setup`。设置好停止时间和步长之后，如果还需要查看每一步过程，选择页面中的`save fields`按照需要保存。

到这里设置就基本结束了，进行一次检查，在`maxwell 2D`打开`validation check`进行检查。

最后，`Analyze ALL`启动！

## 直线运动仿真

## 附赠仿真视频

{% dplayer "url=98i7c-nntsv.mp4" %}
