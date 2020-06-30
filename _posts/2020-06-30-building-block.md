---
layout: post
title: rt-thread之如何积木式编译？
date: 2020-06-30
Author: xuxeu
categories: 
tags: [one]
comments: true
typora-root-url: ..
---

> 查看rt-thread的资料时，对积木式的选择并构建物联网操作系统很感兴趣。

其实也很好理解，为了适应各种不同资源的设备，有的资源少，那么就代码尽可能少，提供它最需要的功能就好~；有的资源多，那么可能需要实现的功能就得更多，更好，更炫！

这也就说明了，我们得保证系统是可裁剪的，现在大部分嵌入式操作系统应该都是可裁剪的，但如果这种裁剪或增添特别方便，就像搭积木一样好使，那就再好不过了。那么是怎么做到呢？

**我认为有两点：**

一是menuconfig生成:

- **.config**
- **rtconfig.h**

如果配置了某些功能，那么会在文件中，以宏的方式体现出来。见上篇文章~

二是scons的脚本文件：

- **menuconfig.py**

  menuconfig.py是通过.config文件来生成rtconfig.h

- **building.py**

  building.py则是通过解析rtconfig.h来获取编译时是否编译某些组件或功能的~



可见**rtconfig.h**真的，非常的重要，不仅源代码会用到里面的宏，scons也会用到！





















