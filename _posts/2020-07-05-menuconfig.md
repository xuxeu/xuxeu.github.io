---
layout: post
title: menuconfig的使用
date: 2020-07-05
Author: xuxeu
categories: 
tags: [one]
comments: true
typora-root-url: ..
---

> 以前使用的都是IDE，现在要使用menuconfig，对比他们的使用，我竟发现更喜欢这种简单的，基于命令行实现的menuconfig呢~
> 

### 什么是menuconfig？

menuconfig，从名字上，就可以看出，是基于menu的config，即，菜单式的配置。

想一想，再命令行中，可以使用图形式的菜单配置，而不是简单的字符交互(例如config)，这真的是awesome~

### menuconfig的使用

如何使用menuconfig？

这里肯定不会写:

- 简单的上下键移动选项
- 空格键选择
- 左右键用于在Select/Exit/Help之前切换
- 回车键一般是根据左右键所在位置，进行当前选项的Select/Exit/Help

这里主要写两点快捷方式：

- `/`，该快捷键，启动搜索功能，可以搜索某个宏对应的选项，good！
- `?`，该快捷键，启动邦族功能，可以查看该选项的说明，特别是依赖（即打开该宏，该宏所依赖的宏会自动打开；以及，打开其它宏时，其它宏所依赖的该宏会自动打开），非常重要！

选择好配置项之后按 ESC 键退出，选择保存修改即可自动生成 rtconfig.h 文件。此时再次使用 scons 命令就会根据新的 rtconfig.h 文件重新编译工程了。

通过对比，发现改变了两个文件：

- **.config**，以前的变成了.config.old作为备份
- **rtconfig.h**

我很喜欢使用menuconfig，因为它非常轻量，但达到的效果却非常好。真的是非常好用了~