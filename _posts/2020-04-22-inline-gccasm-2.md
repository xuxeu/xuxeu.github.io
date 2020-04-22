---
layout: post
title: gcc内联汇编-howto-2
date: 2020-04-22
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 4.内联基础

*The format of basic inline assembly is very much straight forward. Its basic form is*

内联汇编的基本形式非常简单直接

`asm("assembly code");`

举个例子：

```c
asm("movl %ecx %eax");  /* moves the contents of ecx to eax */
asm("movb %bh (%eax)"); /* moves the byte from bh to the memory pointed by eax */
```

*You might have noticed that here I’ve used `asm` and `__asm__`. Both are valid. We can use `__asm__` if the keyword `asm` conflicts with something in our program. If we have more than one instructions, we write one per line in double quotes, and also suffix a ’\n’ and ’\t’ to the instruction. This is because gcc sends each instruction as a string to **as**(GAS) and by using the newline/tab we send correctly formatted lines to the assembler.*

你可能已经注意到我们使用了`asm`和`__asm__`。它们都是合法的，如果`asm`与你的程序冲突，你可以使用`__asm__`。如果有多行代码，我们将每一个使用`"`包含，并后缀`\n\t`。因gcc将每行作为一个`string`到**as**(GAS)，通过换行/tab我们可以发送正确的格式给汇编器(assembler)。

举个例子：

```c
__asm__ ("movl %eax, %ebx\n\t"          
     "movl 56, %esi\n\t"          
     "movl %ecx, label(%edx,%ebx,$4)\n\t"          
     "movb %ah, (%ebx)");
```

*If in our code we touch (ie, change the contents) some registers and return from asm without fixing those changes, something bad is going to happen. This is because GCC have no idea about the changes in the register contents and this leads us to trouble, especially when compiler makes some optimizations. It will suppose that some register contains the value of some variable that we might have changed without informing GCC, and it continues like nothing happened. What we can do is either use those instructions having no side effects or fix things when we quit or wait for something to crash. This is where we want some extended functionality. Extended asm provides us with that functionality.*

如果我们的代码触及（touch）（如，改变内容）一些寄存器，而后不修复这些改变直接从asm返回的话，一些不好的事就会发生。这是因为GCC不知道对寄存器内容的改变，而这将我们带向问题，特别是当编译器进行了某些优化的时候。它将假设一些寄存器包含了一些变量的值，而我们已经改变了没有告知GCC， 然后它继续执行就像什么也没发生一样。我们可以做的是使用一些没有副作用的指令，或者在我们退出前修复问题，或者等待崩溃。这就是我们想要一些扩展功能性（functionality）的地方。**扩展asm（Extended asm）提供了我们这种功能性**。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)