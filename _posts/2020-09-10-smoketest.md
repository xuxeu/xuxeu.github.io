---
layout: post
title: 冒烟测试
date: 2020-09-10
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 知之为知之，不知为不知，是智也。
> 

第一次听见**冒烟测试**，我在想是什么呢？难道我们做的设备要看能不能冒烟吗？哈，想啥呢？bing了一下下，原来冒烟测试这个名称的来历，最初是从电路板测试得来的。因为当电路板做好以后，首先会加电测试，如果板子没有冒烟再进行其它测试，否则就必须重新来过。

所以本质是测试，而冒烟则是代表着简单的测试。，冒烟测试其实是微软首先提出来的一个概念，和微软一直提倡的每日build（构建版本）有很密切的联系。具体说，冒烟测试就是在每日build（构建版本）建立后，对系统的基本功能进行简单的测试。这种测试强调程序的主要功能进行的验证，而不会对具体功能进行更深入的测试。

因此，我们可以得出：

- 冒烟测试是最简单，基础功能的测试
- 冒烟测试是在详细测试之前的测试
- 冒烟测试是研发过程中的测试，由研发人员简单测试验证，而后交予测试人员