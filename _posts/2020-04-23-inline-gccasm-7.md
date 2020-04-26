---
layout: post
title: gcc内联汇编-howto-限制符的修饰符
date: 2020-04-23
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 6.2 限制符修饰符

While using constraints, for more precise control over the effects of constraints, GCC provides us with constraint modifiers. Mostly used constraint modifiers are

当使用限制符时，若要精确控制其效果，GCC提供了修饰符。常用当有：

- "=" : Means that this operand is write-only for this instruction; the previous value is discarded and replaced by output data.

  `=` : 意味着操作数对该指令是只写的；前一个值将被忽略并替换为输出数据。

- "&" : Means that this operand is an earlyclobber operand, which is modified before the instruction is finished using the input operands. Therefore, this operand may not lie in a register that is used as an input operand or as part of any memory address. An input operand can be tied to an earlyclobber operand if its only use as an input occurs before the early result is written.

  `&`: 意味着操作数是一个早期受影响的操作数，也就是在指令结束前已被修改。因此，该操作数不可停留在输入寄存器中或任何内存中。在被写入前仅用于输入的输入操作数可设为一个早期受影响操作数。

The list and explanation of constraints is by no means complete. Examples can give a better understanding of the use and usage of inline asm. In the next section we’ll see some examples, there we’ll find more about clobber-lists and constraints.

关于限制符的描述并不意味结束。例子可以帮助我们更好地理解内联汇编。下一节我们会看一些例子，我们会发现更多关于受影响列表和限制符的使用。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)