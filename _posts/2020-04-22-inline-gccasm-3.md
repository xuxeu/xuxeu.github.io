---
layout: post
title: gcc内联汇编-howto-扩展asm
date: 2020-04-22
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 5.扩展asm

In basic inline assembly, we had only instructions. In extended assembly, we can also specify the operands. It allows us to specify the input registers, output registers and a list of clobbered registers. It is not mandatory to specify the registers to use, we can leave that head ache to GCC and that probably fit into GCC’s optimization scheme better. Anyway the basic format is:

基本汇编中我们只有指令。在扩展汇编中，我们可以指定操作对象。它允许我们指定输入寄存器，输出寄存器及一列受影响的寄存器。它不是强制地指定寄存器使用，我们可以将麻烦留给GCC而GCC有可能更好的适配GCC的优化机制。总之基本形式如下：

```c
 asm ( assembler template
     : output operands                  /* optional */
     : input operands                   /* optional */
     : list of clobbered registers      /* optional */
     );
```

The assembler template consists of assembly instructions. Each operand is described by an operand-constraint string followed by the C expression in parentheses. A colon separates the assembler template from the first output operand and another separates the last output operand from the first input, if any. Commas separate the operands within each group. The total number of operands is limited to ten or to the maximum number of operands in any instruction pattern in the machine description, whichever is greater.

汇编模板（assembler template）由汇编指令构成。每个操作数由一个操作数约束字符串描述，后面是括号中的c表达式。冒号分割汇编模板、输出操作数组、输入操作数组、受影响的寄存器组。逗号分割每个组内的操作数。操作数总数限制在10个或者机器描述中任何指令模式中的最大操作数，以较大者为准。

If there are no output operands but there are input operands, you must place two consecutive colons surrounding the place where the output operands would go.

如果没有输出操作数但有输入操作数，你必须放两个连续冒号。

举个栗子：

```c
asm ("cld\n\t"
     "rep\n\t"
     "stosl"
     : /* no output registers */
     : "c" (count), "a" (fill_value), "D" (dest)
     : "%ecx", "%edi"
     );
```

Now, what does this code do? The above inline fills the `fill_value` `count` times to the location pointed to by the register `edi`. It also says to gcc that, the contents of registers `eax` and `edi` are no longer valid. Let us see one more example to make things more clearer.

以上代码是什么作用？ 上面的内联将fill_value计数时间填充到寄存器EDI指向的位置。它也同时告诉gcc, 寄存器 `eax` and `edi` 的内容不再有效. 让我们看看另一个例子来更好的理解：

```c
int a=10, b;
asm ("movl %1, %%eax; movl %%eax, %0;"
    :"=r"(b)        /* output */
    :"r"(a)         /* input */
    :"%eax"         /* clobbered register */
    ); 
```

Here what we did is we made the value of ’b’ equal to that of ’a’ using assembly instructions. Some points of interest are:

这里我们使用汇编指令让`b`的值等于`a`的值。有趣的点是：

- "b" is the output operand, referred to by %0 and "a" is the input operand, referred to by %1.

  `b` 是输出操作数 指的是 `%0` ；而 `a` 是 输入操作数, 指的是 `%1` 。

- "r" is a constraint on the operands. We’ll see constraints in detail later. For the time being, "r" says to GCC to use any register for storing the operands. output operand constraint should have a constraint modifier "=". And this modifier says that it is the output operand and is write-only.

  `r` 是操作数的限制符。此时, `r` 告诉 GCC 使用任意寄存器来储存操作数。输出操作数限制应该有一个限制修饰符`=`。这个修饰符意味着它是一个输出操作数且是只写的。

- There are two %’s prefixed to the register name. This helps GCC to distinguish between the operands and registers. operands have a single % as prefix.

  在寄存器名称前出现了两个`%`。这帮助GCC来区分操作数和寄存器。操作数有一个单独的`%`作为前缀。

- The clobbered register %eax after the third colon tells GCC that the value of %eax is to be modified inside "asm", so GCC won’t use this register to store any other value.

  受影响（clobbered）寄存器`%eax`在第三个冒号之后，告诉GCC `%eax`的值已在`asm`内被修改，所以GCC不会使用这个寄存器去保存其他的值。

When the execution of "asm" is complete, "b" will reflect the updated value, as it is specified as an output operand. In other words, the change made to "b" inside "asm" is supposed to be reflected outside the "asm".

当`asm`执行结束后，`b`将反应更新后的值，因为它被指定为一个输出操作数。另一方面，asm内部对`b`的改变应该支持在`asm`外部被识别。

Now we may look each field in detail.

现在我们详细的看一下每一个区域。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)