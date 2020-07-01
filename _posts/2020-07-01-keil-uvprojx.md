---
layout: post
title: oneOS下使用scons生成工程项目
date: 2020-07-01
Author: xuxeu
categories: 
tags: [one]
comments: true
typora-root-url: ..
---

> 第一次开始使用Cube-One工具来编译构建项目，忐忑~不同的体验。当然，既然做，就得了解原理，不可茫然~

上篇文章有写过，如何在bsp下搭积木式配置，以及编译构建~，此次主要说说如果要生成keil工程或者iar工程，我们的逻辑在哪里。

**如何生成工程文件？**

```
scons --target=iar
scons --target=mdk4
scons --target=mdk5
```

**生成工程文件的原理？**

python脚本通过`template.uvoptx`、`template.uvprojx`作为输入，`project.uvprojx`等作为输出。其实也很好理解，`template`嘛，模板的意思呀！

![1](/images/2020-07-01-keil-uvprojx/1.png)

**工程文件的意义？**

Keil MDK本质上实现交叉编译链的功能，只是界面上采用eclipse IDE，从这个角度理解，MDK工程文件类型可以分为两类，工程本身的文件和源码编译文件。

工程文件主要包括.uvprojx、.uvoptx、.uvguix及.crf、.dep等。

1. **uvprojx文件**

   uvprojx文件就是我们平时双击打开的工程文件，它记录了整个工程的结构，如芯片类型、工程包含了哪些源文件等内容；

2. **uvoptx文件**

   uvoptx文件记录了工程的配置选项，如下载器的类型、变量跟踪配置、断点位置以及当前已打开的文件等等；

3. **uvguix文件**

   uvguix文件记录了MDK软件的GUI布局，如代码编辑区窗口的大小、编译输出提示窗口的位置等等。

4. **.crf交叉引用文件**

   .crf是交叉引用文件(Cross-Reference file)，它主要包含了浏览信息(browse information)，即源代码中的宏定义、变量及函数的定义和声明的位置。我们在代码编辑器中点击“Go To Definition Of ‘xxxx’”可实现浏览跳转，跳转的时候，MDK就是通过.crf文件查找出跳转位置的。

5. **.dep和.d依赖文件**

   .dep和.d文件(Dependency file)记录的是工程或其它文件的依赖，主要记录了引用的头文件路径，其中.dep是整个工程的依赖，它以工程名命名，而.d是单个源文件的依赖，它们以对应的源文件名命名。这两个文件类似于makefile文件。这些记录使用文本格式存储，我们可直接使用记事本打开。
   

**总结一下：**

- uvprojx、uvoptx及uvguix都是使用XML格式记录的文件，若使用记事本打开可以看到XML代码
- MDK软件打开时，它根据这些文件的XML记录加载工程的各种参数，使得我们每次重新打开工程时，都能恢复上一次的工作环境
- uvprojx文件是最重要的，删掉它我们就无法再正常打开工程了
- uvoptx及uvguix文件并不是必须的，可以删除，重新使用MDK打开uvprojx工程文件后，会以默认参数重新创建uvoptx及uvguix文件
- 所以当使用Git/SVN等代码管理的时候，往往只保留uvprojx文件



参考链接：https://blog.csdn.net/u011354506/article/details/54767205

