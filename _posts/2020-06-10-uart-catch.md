---
layout: post
title: 串口抓包？wireshark！
date: 2020-06-10
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 网络抓包，相信大家都会使用鼎鼎大名的wireshark，但是串口抓包呢？

**portmon？ Device Monitoring Studio？ CommMonitor？**可是，当串口通信时，端口号会被占用啊，这样子，你们这些工具抓包时竟然还要指定端口号，我的天呀，⛏可能是我没有搞懂这些工具吧！

在百愁莫解之际，我发现wireshark竟然还能抓USB包。如果我们的设备连接电脑用的是USB转串口通信，那么我们就可以通过wireshark监测USB口，从而达到串口抓包的目的！

这里需要注意，wireshark在安装的时候需要指定安装USBPcap。如果没有，请重新安装wireshark，并把它勾选上。

![1](/images/2020-06-10-uart-catch/1.png)

安装好以后，需要**重启电脑**，然后就可以看到下图的USBPcap1！

![2](/images/2020-06-10-uart-catch/2.png)

双击**USBPcap1**，oh yeah，抓包成功！剩下的就是分析包了。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)