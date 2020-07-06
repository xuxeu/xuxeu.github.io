---
layout: post
title: 潘多拉stm32l4初尝试
date: 2020-07-01
Author: xuxeu
categories: 
tags: [one]
comments: true
typora-root-url: ..
---

> 此次准备使用pandora的板子进行一次最简单的点灯镜像固化与运行~

🆗，那么今天我就开始使用正点原子得潘多拉stm32l4这块板子，开始吧！xuxu <(￣︶￣)↗[GO!]
[RT-Thread 潘多拉 STM32L475 上手指南](https://www.rt-thread.org/document/site/tutorial/quick-start/iot_board/quick-start/)

因为需要下载st-link，用于烧写固化程序，所以我使用**落落大小姐**注册了一个网站[stmcu](https://www.stmcu.com.cn/User/LoginByWeChat),虽然我也不知道这个网站有多牛还是多小众~
[下载地址](https://www.stmcu.com.cn/Designresource/design_resource_detail?file_name=STSW_LINK009_ST-LINK%2FV2-1%E7%9A%84WinXP-USB%E9%A9%B1%E5%8A%A8&lang=EN&ver=2.0.1)
下载安装好以后，就可以使用keil这个IDE进行下载固化了。

但我现在，有个疑惑，Pandora应该是典型的stm32了，那么在这上面可以测试nb模组吗~~

最后，放一张图，真的很有用：

![iot](/images/2020-07-01-pandora-start/iot.png)