---
layout: post
title: gcc内联汇编-howto-Volatile
date: 2020-04-22
Author: xuxeu
categories: 
tags: [内联汇编]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 5.3 受影响列表

Some instructions clobber some hardware registers. We have to list those registers in the clobber-list, ie the field after the third ’**:**’ in the asm function. This is to inform gcc that we will use and modify them ourselves. So gcc will not assume that the values it loads into these registers will be valid. We shoudn’t list the input and output registers in this list. Because, gcc knows that "asm" uses them (because they are specified explicitly as constraints). If the instructions use any other registers, implicitly or explicitly (and the registers are not present either in input or in the output constraint list), then those registers have to be specified in the clobbered list.

一些指令会影响一些硬件寄存器。我们必须在受影响列表中列出那些寄存器，即`asm`函数第三个`:`后的区域。这用于指示gcc我们将使用并修改它们。所以gcc将不会假设它加载到这些寄存器中的值是合法的。**我们不应该在受影响列表中列出输入和输出寄存器**。因为gcc知道`asm`使用它们（因为它们被明确指定为限制符(constraints)）。**如果指令使用了任何其他寄存器，显式或隐式的（并且这些寄存器没有出现在输入和输出列表上），那么那些寄存器必须在受影响列表中指定**。

If our instruction can alter the condition code register, we have to add "cc" to the list of clobbered registers.

如果我们的指令可以修改条件码寄存器（the condition code register），我们必须增加`cc`到受影响寄存器列表。

If our instruction modifies memory in an unpredictable fashion, add "memory" to the list of clobbered registers. This will cause GCC to not keep memory values cached in registers across the assembler instruction. We also have to add the **volatile** keyword if the memory affected is not listed in the inputs or outputs of the asm.

如果我们的指令用一个不可预期的方法修改了内存，添加`memory`到受影响寄存器。这会使GCC在汇编指令期间不在寄存器内保持内存值的缓存。我们也必须添加**volatile**关键字，如果内存影响（memory affected）未列在`asm`的输入和输出中。

We can read and write the clobbered registers as many times as we like. Consider the example of multiple instructions in a template; it assumes the subroutine _foo accepts arguments in registers `eax` and `ecx`.

我们可以读写受影响寄存器任意多次。注意模板中乘法指令的例子；它假设子程序`_foo` 接受`eax` 、`ecx`寄存器中的参数。

```c
asm ("movl %0,%%eax;
      movl %1,%%ecx;
      call _foo"
      : /* no outputs */
      : "g" (from), "g" (to)
      : "eax", "ecx"
      );
```

#### Volatile关键字 ...?

If you are familiar with kernel sources or some beautiful code like that, you must have seen many functions declared as `volatile` or `__volatile__` which follows an `asm` or `__asm__`. I mentioned earlier about the keywords `asm` and `__asm__`. So what is this `volatile`?

如果你熟悉内核源码或者一些类似的优美代码，你必然已见过很多函数声明为`volatile`或`__volatile__`，跟随在`__asm__`之后。我之前提到过关于关键字`asm`和`__asm__`。所以什么是`volatile`？

If our assembly statement must execute where we put it, (i.e. must not be moved out of a loop as an optimization), put the keyword `volatile` after asm and before the ()’s. So to keep it from moving, deleting and all, we declare it as

如果我们的汇编语句必须在我们放置它的地方执行，(即，必须不被作为一个优化而移出循环)，则将`volatile`放在`asm`之后。所以**防止它被移动、删除和任何改变**，我们如此声明: 

`asm volatile ( ... : ... : ... : ...);`

Use `__volatile__` when we have to be verymuch careful.

当我们必须非常小心时，使用`__volatile__`。

If our assembly is just for doing some calculations and doesn’t have any side effects, it’s better not to use the keyword `volatile`. Avoiding it helps gcc in optimizing the code and making it more beautiful.

如果我们的汇编只是做一些计算而没有任何副作用，最好不要使用`volatile`关键字。忽略它将帮助GCC优化代码使其更优美。

In the section `Some Useful Recipes`, I have provided many examples for inline asm functions. There we can see the clobber-list in detail.

在**“一些有用的代码”**小节，我已经提供了很多内联汇编函数的例子。我们可以详细了解受影响列表。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)