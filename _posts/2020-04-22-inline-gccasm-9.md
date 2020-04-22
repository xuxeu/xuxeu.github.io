---
layout: post
title: gcc内联汇编-howto-9
date: 2020-04-22
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 7. 一些有用的代码

如何设置／清除寄存器中的一个位？

```c
__asm__ __volatile__(   "btsl %1,%0"
                      : "=m" (ADDR)
                      : "Ir" (pos)
                      : "cc"
                      );
```

此处，`ADDR`（一个内存变量）中的`pos`变量对应的比特位将设为1.我们可以用`btrl`替代`btsl`来清除一个位。限制符`Ir`指出，`pos`是一个寄存器，且它的值介于0-31（x86限制符）。即我们可以设置/清除`ADDR`变量中任意0～31位值。因为条件码将被改变，我们增加`cc`到受影响列表。

现在我们看一些复杂但有用的函数。字符串复制。

```c
static inline char * strcpy(char * dest,const char *src)
{
int d0, d1, d2;
__asm__ __volatile__(  "1:\tlodsb\n\t"
                       "stosb\n\t"
                       "testb %%al,%%al\n\t"
                       "jne 1b"
                     : "=&S" (d0), "=&D" (d1), "=&a" (d2)
                     : "0" (src),"1" (dest) 
                     : "memory");
return dest;
}
```

源地址保存在`esi`中，目标地址在`edi`中，然后开始复制，当我们到达**0**时，复制结束。限制符`&S`,`&D`,`&a`说明寄存器`esi`, `edi`, `eax`是早期受影响寄存器。即，它们的内容将在函数完成前被改变。此处明显`memory`也在受影响之列。

我们看一个类似的函数，移动一块双字(double words)。注意函数声明为一个宏。

```c
#define mov_blk(src, dest, numwords) \
__asm__ __volatile__ (                                          \
                       "cld\n\t"                                \
                       "rep\n\t"                                \
                       "movsl"                                  \
                       :                                        \
                       : "S" (src), "D" (dest), "c" (numwords)  \
                       : "%ecx", "%esi", "%edi"                 \
                       )
```

这里我们没有输出，所以改变发生在寄存器`ecx`, `esi` 和 `edi`上，是块移动的副作用。所以我们将它们加在受影响列表上。

Linux中，系统调用是由GCC内联汇编实现的。让我们看一些一个系统调用是如何实现的。所有的系统调用被写作一个宏（linux/unistd.h）。如，一个有3个参数的系统调用被写作如下的宏：

```c
#define _syscall3(type,name,type1,arg1,type2,arg2,type3,arg3) \
type name(type1 arg1,type2 arg2,type3 arg3) \
{ \
long __res; \
__asm__ volatile (  "int $0x80" \
                  : "=a" (__res) \
                  : "0" (__NR_##name),"b" ((long)(arg1)),"c" ((long)(arg2)), \
                    "d" ((long)(arg3))); \
__syscall_return(type,__res); \
}
```

无论任何的3个参数的系统调用，都使用以上宏进行。syscall数字放在`eax`中，然后每个参数放在`ebx`, `ecx`, `edx`。最终`int 0x80`指令真正的执行调用。返回码可以在`eax`中获得。

每个系统调用实现方法类似。退出是一个单参数系统调用，让我们看看它的代码是什么样的，如下：

```c
{
        asm("movl $1,%%eax;         /* SYS_exit is 1 */
             xorl %%ebx,%%ebx;      /* Argument is in ebx, it is 0 */
             int  $0x80"            /* Enter kernel mode */
             );
}
```

退出数字是`1`，参数（parameter）是`0`。所以我们安排`eax`包含1，`ebx`包含`0`，通过`int $0x80`执行`exit(0)`。这就是exit的工作原理。









#### 出差必备

买火车票、高铁票、机票，订酒店都打9折的出行工具TRIP,[点击注册](https://h5.itrip.world/#/register/6tpd1Z)