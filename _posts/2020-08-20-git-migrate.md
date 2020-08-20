---
layout: post
title: git仓库完整迁移到其它远程平台
date: 2020-08-20
Author: xuxeu
categories: 
tags: [编程杂谈]
comments: true
typora-root-url: ..
---

> 完整迁移，就是指，不仅将所有代码移植到新的仓库，而且要保留所有的commit记录。我记得之前有过一次akx项目的代码迁移，但我没有参与。原来如此简单，git还是强啊！

#### 1. 随便找个文件夹，从原地址克隆一份裸版本库

```bash
git clone --bare 旧的git地址
```

  会在当前目录下产生一个 **xxx.git** 的文件夹

> 这个步骤，就是克隆git每一次的提交信息
>  和本地的代码没有关系，只要线上的代码是最新的，这个git版本就是完整的

#### 2. 推送裸版本库到新的地址

```bash
cd xxx.git
git push --mirror 新的git地址
```

#### 3. 删掉xxx.git文件夹

  删不删无所谓，只是说明它没有用了而已。

#### 4. 代码迁移就成功了，接下来就可以使用新的地址了

```bash
git clone 新的git地址
```

