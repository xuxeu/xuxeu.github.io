---
layout: post
title: gcc内联汇编-howto-操作数
date: 2020-04-22
Author: xuxeu
categories: 
tags: [内联汇编]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 5.1 扩展模板

The assembler template contains the set of assembly instructions that gets inserted inside the C program. The format is like: either each instruction should be enclosed within double quotes, or the entire group of instructions should be within double quotes. Each instruction should also end with a delimiter. The valid delimiters are newline(\n) and semicolon(;). ’\n’ may be followed by a tab(\t). We know the reason of newline/tab, right?. Operands corresponding to the C expressions are represented by %0, %1 ... etc.

汇编模板包含一组嵌入到C程序中的指令。格式类似：**或者每个指令包围在双引号中，或整组指令包含在双引号中**。每个指令也应该以一个分隔符结束。**合法的分隔符可以是`\n`和`;`。**`\n`可以跟随一个`\t`。C表达式的操作数呈现为 `%0`, `%1` ...等。

#### 5.2 操作数

C expressions serve as operands for the assembly instructions inside "asm". Each operand is written as first an operand constraint in double quotes. For output operands, there’ll be a constraint modifier also within the quotes and then follows the C expression which stands for the operand. ie,

c表达式作为内联汇编指令的操作数。每个操作数首先写一个双引号内的操作数限制符。 对于输出操作数, 引号内还有一个限制修饰符， 然后跟随操作数对应的 C 表达式 。 即，

"constraint" (C expression) is the general form. For output operands an additional modifier will be there. Constraints are primarily used to decide the addressing modes for operands. They are also used in specifying the registers to be used.

**`"constraint" (C expression)` **乃通用形式。对输出操作数会有一个额外的修饰符。限制符（constraint）主要用于决定操作数的地址模式。他们也被用于指定要使用的寄存器。

If we use more than one operand, they are separated by comma.

如我们使用超过一个操作数，以逗号`,`分隔。

In the assembler template, each operand is referenced by numbers. Numbering is done as follows. If there are a total of n operands (both input and output inclusive), then the first output operand is numbered 0, continuing in increasing order, and the last input operand is numbered n-1. The maximum number of operands is as we saw in the previous section.

**在汇编模板中，每个操作数按数字被引用。数字按如下规则排列。如果有n个操作数（包括输入、输出），那么第一个输出操作数是数字`0`，连续增加，最后一个输入操作数是数字`n-1`。最大操作数数量如上一段所述**。

Output operand expressions must be lvalues. The input operands are not restricted like this. They may be expressions. The extended asm feature is most often used for machine instructions the compiler itself does not know as existing ;-). If the output expression cannot be directly addressed (for example, it is a bit-field), our constraint must allow a register. In that case, GCC will use the register as the output of the asm, and then store that register contents into the output.

输出操作数表达式必须是`lvalues`(32-bit)。输入操作数无此限制。他们必须是表达式。扩展汇编功能是最常用于编译器自身不知晓的机器指令;-)。如果输出表达式无法被直接寻址（addressed）（比如，它是一个bit-field），我们限制符必须“允许”(allow)一个寄存器。在那种情况下，GCC将使用该寄存器为asm的输出，然后将寄存器内容存储到输出。

As stated above, ordinary output operands must be write-only; GCC will assume that the values in these operands before the instruction are dead and need not be generated. Extended asm also supports input-output or read-write operands.

如上所述，**原始输出操作数必须是只写的**；GCC将假设那个操作对象中的值在指令前已失效且无需生成。扩展汇编也支持“输入-输出”或“读-写”操作数。

So now we concentrate on some examples. We want to multiply a number by 5. For that we use the instruction `lea`.

我们现在看一些例子。我们希望将一个数乘以5。对此我们使用`lea`指令。

```c
asm ("leal (%1,%1,4), %0" 
    : "=r" (five_times_x)
    : "r" (x)
    );
```

Here our input is in ’x’. We didn’t specify the register to be used. GCC will choose some register for input, one for output and does what we desired. If we want the input and output to reside in the same register, we can instruct GCC to do so. Here we use those types of read-write operands. By specifying proper constraints, here we do it.

此处我们的输入是`x`。我们没有指定使用哪个寄存器。GCC会为输入选择一些寄存器用来输入，一个用来输出，执行我们的要求。如果我们希望输入和输出放在（reside）同一个寄存器中，我们可以让GCC来实现。这里我们使用那种"读-写"操作数，通过指定合适的限制符，这里我们来实现它：

```c
asm ("leal (%0,%0,4), %0"
    : "=r" (five_times_x)
    : "0" (x)
    );
```

Now the input and output operands are in the same register. But we don’t know which register. Now if we want to specify that also, there is a way.

现在输入和输出操作数在同一个寄存器内了。但我们不知道是哪个寄存器。现在如果我们也想要指定，有一个办法：

```c
asm ("leal (%%ecx,%%ecx,4), %%ecx"
    : "=c" (x)
    : "c"  (x)
    );
```

In all the three examples above, we didn’t put any register to the clobber list. why? In the first two examples, GCC decides the registers and it knows what changes happen. In the last one, we don’t have to put `ecx` on the c lobberlist, gcc knows it goes into x. Therefore, since it can know the value of `ecx`, it isn’t considered clobbered.

以上三个例子中，我们没有把任何一个寄存器放在受影响列表中。为什么？前两个例子中，GCC决定使用哪个寄存器，因此知道发生了什么改变。在最后一个中，我们不需要将`ecx`放在受影响列表中，gcc知道它会放入x中。因为它可以知道`ecx`的值，它不会被视为受影响的。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)