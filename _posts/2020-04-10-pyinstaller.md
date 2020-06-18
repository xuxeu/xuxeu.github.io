---
layout: post
title: python打包成exe
date: 2020-04-10
Author: xuxeu
categories: 
tags: [python]
comments: true
typora-root-url: ..
---

> 很多时候，我们编写的Python脚本需要给其它人使用！如果我们直接把py脚本给别人，而这个使用的人则需要在她的电脑上安装Python运行环境。

如果还需要安装相关库，则更加麻烦了，这无疑增加了脚本的传播难度和实用性。

为了解决这个问题，我们可以把Python脚本打包成exe，这样，只要把exe发给其他人，她就可以直接使用了，岂不快哉。

在Python3，打包exe尤为简单，难就难在少有人会去**思考打包exe**的问题。

### 主要操作如下：

1. 使用pip安装pyinstaller

   ```pip install pyinstaller```

2. 使用pyinstaller打包

   ```pyinstaller [参数] xxx.py```

3. pyinstaller参数

   ```
   -icon=your path    加一个图标

   -F 打包成一个文件

   -w 无控制台运行界面

   -D 创建一个目录，里面包含exe以及其他一些依赖性文件

   pyinstaller -h 查看参数
   ```

   注意：


*该脚本为Python3编写,想知道更多参数可查看pyinstaller -h*
