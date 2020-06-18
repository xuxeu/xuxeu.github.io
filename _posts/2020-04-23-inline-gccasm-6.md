---
layout: post
title: gcc内联汇编-howto-限制符
date: 2020-04-23
Author: xuxeu
categories: 
tags: [内联汇编]
comments: true
typora-root-url: ..
---

> 本文是[GCC-Inline-Assembly-HOWTO](http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)的翻译，尽量翻译正确，如有疑惑，或参看原文。

#### 6. 详解限制符

By this time, you might have understood that constraints have got a lot to do with inline assembly. But we’ve said little about constraints. Constraints can say whether an operand may be in a register, and which kinds of register; whether the operand can be a memory reference, and which kinds of address; whether the operand may be an immediate constant, and which possible values (ie range of values) it may have.... etc.

此时，你可能已经理解限制符必须要做很多的事。但关于限制符我们说的很少。限制符可以说出操作数是否可能是一个寄存器，及哪类寄存器；操作数是否可以是一个内存引用，及哪一类地址；操作数是否可能是一个立即常量，及它可以有哪些可能的值（即值的范围）...等。

##### 6.1 常用限制符

There are a number of constraints of which only a few are used frequently. We’ll have a look at those constraints.

有许多限制符，只有一部分是常用的。我们看一看这些限制符。

1. **Register operand constraint(r)**

   **寄存器操作数限制符**

   When operands are specified using this constraint, they get stored in General Purpose Registers(GPR). Take the following example:

   当操作数指定使用此限制符时，它们会存储在常规寄存器GPR中。如：

   ```
   asm ("movl %%eax, %0\n"
       :"=r"(myval)
       );
   ```

   Here the variable myval is kept in a register, the value in register `eax` is copied onto that register, and the value of `myval` is updated into the memory from this register. When the "r" constraint is specified, gcc may keep the variable in any of the available GPRs. To specify the register, you must directly specify the register names by using specific register constraints. They are:

   此处`myval`变量保存在一个寄存器中，`eax`的值会复制到那个寄存器，而`myval`的值会从这个寄存器中更新到内存。当`"r"`限制符被指定后，gcc可以在任何可用的GPR中保存这个变量。要指定该寄存器，你必须使用特定寄存器限制符指定寄存器名称。它们是：

| r    | Register(s)    |
| ---- | -------------- |
| a    | %eax, %ax, %al |
| b    | %ebx, %bx, %bl |
| c    | %ecx, %cx, %cl |
| d    | %edx, %dx, %dl |
| S    | %esi, %si      |
| D    | %edi, %di      |

2. **Memory operand constraint(m)**

   **内存操作限制符**

   When the operands are in the memory, any operations performed on them will occur directly in the memory location, as opposed to register constraints, which first store the value in a register to be modified and then write it back to the memory location. But register constraints are usually used only when they are absolutely necessary for an instruction or they significantly speed up the process. Memory constraints can be used most efficiently in cases where a C variable needs to be updated inside "asm" and you really don’t want to use a register to hold its value. For example, the value of idtr is stored in the memory location loc:

   当操作数是在内存中时，任何在它上的操作将直接在内存位置进行，而寄存器限制符，则优先存于寄存器而后修改再写回内存。但寄存器限制符通常只在指令必需或者明显提升性能时使用。当C变量需在`asm`中修改且无需寄存器保持其值时，内存限制符可最大化性能。如，将`idtr`的值存储于loc的内存位置中：

   ```c
   asm("sidt %0\n"
      :
      :"m"(loc)
      );
   ```

3. **Matching(Digit) constraints**

   匹配（数字）限制符

   In some cases, a single variable may serve as both the input and the output operand. Such cases may be specified in "asm" by using matching constraints.

   有时，一个单独变量既是输入也是输出操作符，这时可使用匹配限制符。

   ```c
   asm ("incl %0"
       :"=a"(var)
       :"0"(var)
       );
   ```

   We saw similar examples in operands subsection also. In this example for matching constraints, the register %eax is used as both the input and the output variable. var input is read to %eax and updated %eax is stored in var again after increment. "0" here specifies the same constraint as the 0th output variable. That is, it specifies that the output instance of var should be stored in %eax only. This constraint can be used:

   我们在操作数一节看到了类似的例子，在这个例子中寄存器`%eax`既是输入也是输出变量。`var`输入读入`%eax`并更新到`%eax`最后在自增后存入`var`。这里的`"0"`指定了和输出变量一样的第0个限制符。也就是说，它指定了var的输出过程应该只存于`%eax`中。这类限制符可用于：

   - In cases where input is read from a variable or the variable is modified and modification is written back to the same variable.

     输入输出是统一变量，或变量被修改并被写会同一变量时。

   - In cases where separate instances of input and output operands are not necessary.

     将输入和输出操作符分开是不必要的时候。

   The most important effect of using matching restraints is that they lead to the efficient use of available registers.

   使用匹配限制符最重要的效果是使可用寄存器的使用更有效。

Some other constraints used are:

一些其他的限制符有：

1. "m" : A memory operand is allowed, with any kind of address that the machine supports in general.

   `m`: 接受内存操作数，任意的机器支持的地址。

2. "o" : A memory operand is allowed, but only if the address is offsettable. ie, adding a small offset to the address gives a valid address.

   `o`: 接受内存操作数，只接受偏移地址（offsettable）。即对某个合法地址添加一个微小的偏移量。

3. "V" : A memory operand that is not offsettable. In other words, anything that would fit the `m’ constraint but not the `o’constraint.

   `V`: 非偏移内存操作数。换句话说，任何符合`"m"`但不符合`"o"`限制符的地址。

4. "i" : An immediate integer operand (one with constant value) is allowed. This includes symbolic constants whose values will be known only at assembly time.

   `i`: 立即整型操作数，允许在编译期(assembly-time)可知常量符号。

5. "n" : An immediate integer operand with a known numeric value is allowed. Many systems cannot support assembly-time constants for operands less than a word wide. Constraints for these operands should use ’n’ rather than ’i’.

   `n`: 立即整型操作数，允许已知数字值。许多系统不支持小于16-bit的（word wide)编译期(assembly-time)常量作为操作数。这些操作数应该使用`n`而不是`i`。

6. "g" : Any register, memory or immediate integer operand is allowed, except for registers that are not general registers.

   任何寄存器，内存或立即整型操作数都可用，要求寄存器不是常规寄存器(general registers)。

Following constraints are x86 specific.

以下限制符为x86限定：

1. "r" : Register operand constraint, look table given above.
2. "q" : Registers a, b, c or d.
3. "I" : Constant in range 0 to 31 (for 32-bit shifts).
4. "J" : Constant in range 0 to 63 (for 64-bit shifts).
5. "K" : 0xff.
6. "L" : 0xffff.
7. "M" : 0, 1, 2, or 3 (shifts for lea instruction).
8. "N" : Constant in range 0 to 255 (for out instruction).
9. "f" : Floating point register
10. "t" : First (top of stack) floating point register
11. "u" : Second floating point register
12. "A" : Specifies the `a’ or `d’ registers. This is primarily useful for 64-bit integer values intended to be returned with the `d’ register holding the most significant bits and the `a’ register holding the least significant bits.
