---
title: Anaconda+Pycharm配置pytorch
date: 2025-05-07 14:40:18
tags:
  - Federal Learning
  - Data
updated:
categories:
  - install 
keywords:
description: If you want to start learning FederalLearning, maybe you need to use pytorch. Ah, this is your happiness!
top _img:
comments: True
cover: https://cdn.jsdelivr.net/gh/Miss3504186601/blog-assets@latest/pictures/%E6%99%9A%E9%9C%9E.JPG
---

# Anaconda
`anaconda`会创建一个“小房间”，你可以在里面装上你的项目需要的依赖与单独的解释器，这样可以实现项目分离。

这么解释还是不能说服你使用`anaconda`而不使用`python`虚拟环境，这里会进一步解释[button](https://www.cnblogs.com/LuvLetter-whx/p/18864307)

1. 安装`anaconda`.[Download](https://www.anaconda.com/download)
2. init anaconda virtual env
3. install pytorch

# Pycharm

1. create new project
2. configure project interpreter(My Pycharm version is 2024.3.1)
   1. file
   2. setting
   3. project
   4. add local interpreter
      1. 选择`现有`的（在anaconda创建好的虚拟环境）
      2. `conda`本地选择`conda.exe`路径即可（一般在`***/Anaconda3/Scripts/conda.exe`）





