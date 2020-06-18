---
layout: post
title: gcc内联汇编-howto-汇编语法
date: 2020-04-22
Author: xuxeu
categories: 
tags: [内联汇编]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 3.gcc汇编语法

*GCC, the GNU C Compiler for Linux, uses **AT&T**/**UNIX** assembly syntax. Here we’ll be using AT&T syntax for assembly coding. Don’t worry if you are not familiar with AT&T syntax, I will teach you. This is quite different from Intel syntax. I shall give the major differences.*

GCC使用AT&T/UNIX汇编语法。这里我们将使用AT&T汇编语法来codeing。如果你对AT&T语法不熟悉也不要担心，接下来会告诉你。GCC的AT&T汇编语法与Intel语法区别较大，主要区别有：

1. *Source-Destination Ordering.*

   源-目标顺序

   *The direction of the operands in AT&T syntax is opposite to that of Intel. In Intel syntax the first operand is the destination, and the second operand is the source whereas in AT&T syntax the first operand is the source and the second operand is the destination. ie*

   AT&T语法的操作码顺序刚好和Intel相反。在Intel语法中，它的第一个操作数是目标操作数，第二个则是源操作数。而在AT&T语法中，第一个操作数是源操作数，第二个才是目标操作数。

   *"Op-code dst src" in Intel syntax *

   **"Op-code src dst" in AT&T syntax.**

2. *Register Naming.*

   寄存器命名

   *Register names are prefixed by % ie, if eax is to be used, write %eax.*

   AT&T语法中，寄存器命名以`%`为前缀，如使用`eax`，写作`%eax`

3. *Immediate Operand.*

   立即操作数

   *AT&T immediate operands are preceded by `$`. For static "C" variables also prefix a `$`. In Intel syntax, for hexadecimal constants an ’h’ is suffixed, instead of that, here we prefix ’0x’ to the constant. So, for hexadecimals, we first see a ’$’, then ’0x’ and finally the constants.*

   AT&T立即操作数以`$`开头，对static“c”变量也是前置`$`。16进制常量，Intel语法后缀h，AT&T前缀0x。所以对于16进制数，我们会先看到$,然后是0x,最后是常量。

4. *Operand Size.*

   操作数大小

   *In AT&T syntax the size of memory operands is determined from the last character of the op-code name. Op-code suffixes of ’b’, ’w’, and ’l’ specify byte(8-bit), word(16-bit), and long(32-bit) memory references. Intel syntax accomplishes this by prefixing memory operands (not the op-codes) with ’byte ptr’, ’word ptr’, and ’dword ptr’.*

   AT&T语法中操作数大小取决于操作码最后一个字符。操作码后缀`b`,`w`,`l` 对应 byte(8-bit),  word(16-bit), 和 long(32-bit)。Intel语法中，通过在操作数（非操作码）前缀 `byte ptr`, `word ptr`, 和 `dword ptr` 实现该功能。

   Thus, Intel "mov al, byte ptr foo" is "movb foo, %al" in AT&T syntax.

5. *Memory Operands.*

   内存操作数

   In Intel syntax the base register is enclosed in ’[’ and ’]’ where as in AT&T they change to ’(’ and ’)’. Additionally, in Intel syntax an indirect memory reference is like

   Intel语法中基址寄存器（The base register）内于`[`、`]`之间，而AT&T于`(`、`)` 之间。此外，间接内存引用（indirect memory reference）。

   *section:[base + index*scale + disp] in Intel

   section:disp(base, index, scale) in AT&T.

   One point to bear in mind is that, when a constant is used for disp/scale, ’$’ shouldn’t be prefixed.

   需指出，当常量使用disp/scale，`$` 无需前置。

*Now we saw some of the major differences between Intel syntax and AT&T syntax. I’ve wrote only a few of them. For a complete information, refer to GNU Assembler documentations. Now we’ll look at some examples for better understanding.*

以上是Intel于AT&T语法的主要区别，完整信息请参加GNU Assembler documentations。以下一些例子有助于我们更好的理解：

| Intel Code                   | AT&T Code                         |
| ---------------------------- | :-------------------------------- |
| mov eax,1                    | movl $1,%eax                      |
| mov ebx,0ffh                 | movl $0xff,%ebx                   |
| int 80h                      | int $0x80                         |
| mov ebx, eax                 | movl %eax, %ebx                   |
| mov     eax,[ecx]            | movl    (%ecx),%eax               |
| mov     eax,[ebx+3]          | movl    3(%ebx),%eax              |
| mov     eax,[ebx+20h]        | movl    0x20(%ebx),%eax           |
| add     eax,[ebx+ecx*2h]     | addl    (%ebx,%ecx,0x2),%eax      |
| lea     eax,[ebx+ecx]        | leal    (%ebx,%ecx),%eax          |
| sub     eax,[ebx+ecx*4h-20h] | subl    -0x20(%ebx,%ecx,0x4),%eax |
