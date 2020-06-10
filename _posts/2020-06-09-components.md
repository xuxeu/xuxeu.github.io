---
layout: post
title: 配置组件的两种办法！
date: 2020-06-09
Author: xuxeu
categories: 
tags: [one]
comments: true
typora-root-url: ..
---

> 什么是组件？就是什么文件系统啦，网络协议栈啦，shell控制台啦，log日志，edr等等都是组件！

我们把组件的初始化一般是怎么做的呢？vxworks是在板级下的prjconfig.c文件里直接调用组件的init接口进行初始化。这是一种办法！

现在，我发现很多系统并不是这么做的了，而是使用了另一种办法。

她们组件的初始化并不是在系统的初始化接口调用的，而是在每个组件初始化接口实现的后面加上了**OS_CMPOENT_INIT**，把他放到了连接脚本的节信息里面。然后再系统初始化的时候直接在指定内存去读取执行就好啦。

我以**sal组件的初始化**来举个栗子：

首先，在定义sal_init初始化接口的地方，把该接口地址赋值给一个指针变量，并将该指针变量放到**.init_call.**开头的节里面。如下图：

![1](/images/2020-06-09-componet/1.png)

![2](/images/2020-06-09-componet/2.png)

其次，在初始化任务中调用os_other_auto_init来统一完成组件的初始化！

![3](/images/2020-06-09-componet/3.png)

连接脚本如下:

![4](/images/2020-06-09-componet/4.png)

看，这种方法是不是很nice！更有趣的，我们可以通过配置一些宏，来确认某些组件是否编译！如果不编译，自然该初始化接口就不会放到**.init_call.**开头的节里面，也就达到了添加组件或删除组件的作用，goooooooood！

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)