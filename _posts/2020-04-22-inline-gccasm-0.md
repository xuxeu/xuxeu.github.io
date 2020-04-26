---
layout: post
title: gcc内联汇编-howto-概述
date: 2020-04-22
Author: xuxeu
categories: 
tags: [内联汇编]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 1.介绍

##### 1.1 版权

This document is free; you can redistribute and/or modify this under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.

This document is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

#####  1.2 反馈

请将反馈和批评通过博客下方的邮件和我联系。 感谢任何在本文件中指出错误和不准确之处的人，我将纠正错误。

##### 1.3 感谢

I express my sincere appreciation to GNU people for providing such a great feature. Thanks to Mr.Pramode C E for all the helps he did. Thanks to friends at the Govt Engineering College, Trichur for their moral-support and cooperation, especially to Nisha Kurur and Sakeeb S. Thanks to my dear teachers at Govt Engineering College, Trichur for their cooperation.

Additionally, thanks to Phillip, Brennan Underwood and colin@nyx.net; Many things here are shamelessly stolen from their works.

####  2.概述

We are here to learn about GCC inline assembly. What this inline stands for?

我们是来此学习GCC内联汇编的。 那么，内联是什么？

We can instruct the compiler to insert the code of a function into the code of its callers, to the point where actually the call is to be made. Such functions are inline functions. Sounds similar to a Macro? Indeed there are similarities.

我们可以指导编译器将函数的代码直接插入调用的位置，这类函数叫做内联函数。听起来像是宏？事实上还真挺像。

What is the benefit of inline functions?

- 内联函数有什么好处？

This method of inlining reduces the function-call overhead. And if any of the actual argument values are constant, their known values may permit simplifications at compile time so that not all of the inline function’s code needs to be included. The effect on code size is less predictable, it depends on the particular case. To declare an inline function, we’ve to use the keyword `inline` in its declaration.

内联的方法降低了函数调用的问题。而且如果任何参数是常量的话，在编译器将得到明显优化，而不是所有的内联函数代码都被包含。代码量会更少，取决于具体的情况。为了定义内联函数，我们使用关键字`inline`声明。

Now we are in a position to guess what is inline assembly. Its just some assembly routines written as inline functions. They are handy, speedy and very much useful in system programming. Our main focus is to study the basic format and usage of (GCC) inline assembly functions. To declare inline assembly functions, we use the keyword `asm`.

- 什么是内联汇编？

内联汇编是写在内联函数中的汇编过程(assembly routines)。它非常方便、快速，在系统编程中非常有用。我们主要关注学习GCC内联汇编函数的基础格式和用法。要声明内联汇编函数，我们使用关键字`asm`。

Inline assembly is important primarily because of its ability to operate and make its output visible on C variables. Because of this capability, "asm" works as an interface between the assembly instructions and the "C" program that contains it.

内联汇编很重要，因为有能力操作并输出到C变量中。因为这些能力，`asm`作为了C和汇编指令间的接口。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)