---
title: 通过requests使用moonraker的api
tags:
  - 3D打印
cover: /img/RUN.png
categories:
- [计算机, 折腾记录]
date: 2023-08-02 22:45:47
description: 通过python向moonraker发送api请求
---
```
Python 3.8.6 (tags/v3.8.6:db45529, Sep 23 2020, 15:52:53) [MSC v.1927 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import requests
>>> r = requests.post("http://192.168.0.104/printer/gcode/script?script=M220 S150")
>>> r.text
'{"result": "ok"}'
>>> r
<Response [200]>
>>>
```
此时打印机移动速度就变成了150%