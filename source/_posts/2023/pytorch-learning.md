---
title: 使用pytorch设计神经网络
tags:
  - 神经网络
cover: /img/QQ截图20230920192636.png
categories:
  - null
date: 2023-09-20 18:08:22
description: pytorch？蟒蛇火把！
---
# START
有一个[名不符实但是不错的教程文章](https://pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html)，首先是跟着这个教程摸索一下

主要目的还是用神经网络做图像处理，比如基础的分类，然后实验室其他同学正在做智能车5G组的比赛，我想能不能实现在树莓派上识别摄像头拍到的操场跑道线，帧率最好不低于30Fps，之后也许会在做点别的？以后再补充

# NEURAL NETWORKS
定义一个神经网络(中文注释是我自己的理解可能有错误)
```py
import torch
import torch.nn as nn
import torch.nn.functional as F

# 定义网络
class Net(nn.Module):

    def __init__(self):
        # 继承的父类初始化
        super(Net, self).__init__()
        # 1 input image channel, 6 output channels, 5x5 square convolution
        # 一层输入，6层输出，卷积核5x5
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        # an affine operation: y = Wx + b
        # 三个全连接层，从卷积之后的16*5*5到输出的10个类别
        self.fc1 = nn.Linear(16 * 5 * 5, 120)  # 5*5 from image dimension
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        # Max pooling over a (2, 2) window
        # 对输入进行之前定义的卷积，然后使用RELU激活，然后使用2*2的窗口最大池化
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        # If the size is a square, you can specify with a single number
        # 上面的(2, 2)也可以简写为2
        x = F.max_pool2d(F.relu(self.conv2(x)), 2)
        # 形状改为一维进入全连接层进行两层连接计算和激活
        x = torch.flatten(x, 1) # flatten all dimensions except the batch dimension
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

net = Net()
print(net)

# 输入一个随机数据看看输出
input = torch.randn(1, 1, 32, 32)
out = net(input)
print(out)
```
