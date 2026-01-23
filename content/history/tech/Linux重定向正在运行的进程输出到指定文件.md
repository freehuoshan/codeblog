---
title: Linux重定向正在运行的进程输出到指定文件
id: 20
date: 2023-04-10 18:19:19
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/download%20(3)-aa8689bc7e7a481692d3cb1435b949be.jpeg
excerpt: 今天遇到一个问题，自己写的一个 java 程序运行着一个定时任务，而且该定时任务正在运行。由于启动时指定输出文件与另一个进程重复，日志没有被输出到该文件，现在需要检查正在运行的日志。所以需要将正在运行的java进程输出日志输出到一个文件，但是又不能终止该进程。linux 中没有现有工具可以实现这个，
permalink: /archives/linux-zhong-ding-xiang-zheng-zai-yun-xing-de-jin-cheng-shu-chu-dao-zhi-ding-wen-jian
categories:
 - code
tags: 
 - linux
---

今天遇到一个问题，自己写的一个 java 程序运行着一个定时任务，而且该定时任务正在运行。由于启动时指定输出文件与另一个进程重复，日志没有被输出到该文件，现在需要检查正在运行的日志。所以需要将正在运行的java进程输出日志输出到一个文件，但是又不能终止该进程。

  linux 中没有现有工具可以实现这个，github上找到了一个工具恰好可以实现这一点。

  地址: https://github.com/jerome-pouiller/reredirect/

  该工具可以将正在运行的进程的输出输出到指定文件，然后可以重置到之前的状态。

  安装:
```shell
 git clone git@github.com:jerome-pouiller/reredirect.git
 cd reredirect
 make
 make install
```

  使用:

  ```shell
reredirect -m filepath PID  
```
 
  进程 PID 的输出重定向到 filepath 文件

  其他用法参考文档。