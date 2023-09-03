---
title: 通过b站直播传输加密数据
tags:
  - 整活
cover: /img/RUN.png
categories:
- [计算机, 折腾记录]
date: 2023-08-23 11:45:30
description: 一种很新的彩六直播方法
---
# b站直播推流发送数据
这个简单说一下就好，用obs截取python的窗口推流给b站，当然也可以自己推，就是懒得写了所以obs解决
# 爬取b站直播接收数据
解析b站使用的链接参考了这篇[文章](https://blog.csdn.net/Enderman_xiaohei/article/details/102626855)，我就直接拿来结论用了，首先拿到直播间号，比如我刚刚随便打开一个`21332276`，用这个链接就可以获得视频流地址，我写了个程序实现自动化：




