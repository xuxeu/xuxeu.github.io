---
layout: post
title: 函数别名alias
date: 2020-04-07
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

>  如果不更改函数调用位置的函数名，可以在函数实现的后面使用函数别名的方式，让目标文件*.o能同时存在两个函数名。

**是不是非常有趣？**

秘密就在于关键字**alias**。

### 举个栗子

```c
int vx_test(int y)
{
    int x=2+y;
 return x;
}
```

如果向上面这样，我们可以直接调用`vx_test`来执行该函数体。

```c
int delta_test(int y)
{
    int x=2;
 return x;
}
int vx_test(int y) __attribute__((alias("delta_test")));
```

而如果像这样，我们则可以调用`delta_test`或者`vx_test`来执行同一个函数体。

*想知道更多的alias解释，请自行百度。*
