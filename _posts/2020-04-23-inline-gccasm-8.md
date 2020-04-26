---
layout: post
title: gcc内联汇编-howto-举个栗子
date: 2020-04-23
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 7. 一些有用的代码

Now we have covered the basic theory about GCC inline assembly, now we shall concentrate on some simple examples. It is always handy to write inline asm functions as MACRO’s. We can see many asm functions in the kernel code. (/usr/src/linux/include/asm/*.h).

现在我们已经基本涵盖了GCC内联汇编内容，我们应该关注一些简单的例子。使用宏来定义内联汇编总是方便的。我们可以看到很多内核（kernel）代码的`asm`函数例子。(/usr/src/linux/include/asm/*.h)

First we start with a simple example. We’ll write a program to add two numbers.

首先我们从一个简单的例子开始。我们写一个程序，将两个数字相加：

```c
int main(void)
{
        int foo = 10, bar = 15;
        __asm__ __volatile__("addl  %%ebx,%%eax"
                             :"=a"(foo)
                             :"a"(foo), "b"(bar)
                             );
        printf("foo+bar=%d\n", foo);
        return 0;
}
```

Here we insist GCC to store foo in %eax, bar in %ebx and we also want the result in %eax. The ’=’ sign shows that it is an output register. Now we can add an integer to a variable in some other way.

处我们让GCC将`foo`存入`%eax`，将`bar`存入`%ebx`，然后我们希望结果也存在`%eax`中。`=`符号表示那是一个输出寄存器。现在我们可以用另一种方式让变量加整数。

```c
__asm__ __volatile__(
                      "   lock       ;\n"
                      "   addl %1,%0 ;\n"
                      : "=m"  (my_var)
                      : "ir"  (my_int), "m" (my_var)
                      :                                 /* no clobber-list */
                      );
```

This is an atomic addition. We can remove the instruction ’lock’ to remove the atomicity. In the output field, "=m" says that my_var is an output and it is in memory. Similarly, "ir" says that, my_int is an integer and should reside in some register (recall the table we saw above). No registers are in the clobber list.

这是一个原子加法。我们可以移除`lock`指令来移除原子性。输出段`=m`意为`my_var`是一个输出操作数且在内存中。类似的`ir`说明`my_int`是一个整数并应该载入(reside)到寄存器中。没有受影响寄存器列表。

Now we’ll perform some action on some registers/variables and compare the value.

现在我们会执行一些动作在寄存器/变量上并比较它们到值。

```c
__asm__ __volatile__(  "decl %0; sete %1"
                      : "=m" (my_var), "=q" (cond)
                      : "m" (my_var) 
                      : "memory"
                      );
```

Here, the value of my_var is decremented by one and if the resulting value is `0` then, the variable cond is set. We can add atomicity by adding an instruction "lock;\n\t" as the first instruction in assembler template.

此处，`my_var`的值减1，如果结果为0则`cond`变量被设置。我们同样可以添加`lock;\n\t`在第一句来实现原子性。

In a similar way we can use "incl %0" instead of "decl %0", so as to increment my_var.

类似的，我们可以用`incl %0`代替`decl %0`来实现`my_var`的加1。

Points to note here are that：

此处需要指出：

1. my_var is a variable residing in memory. 

   my_var 是一个位于(residing in)内存中的变量

2. cond is in any of the registers eax, ebx, ecx and edx. The constraint "=q" guarantees it. 

   `cond`可以在`eax`,`ebx`,`ecx`和`edx`中的任意一个。

3. And we can see that memory is there in the clobber list. ie, the code is changing the contents of memory

   受影响列表中包含`memory`，即代码将改变内存中的值。

#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)