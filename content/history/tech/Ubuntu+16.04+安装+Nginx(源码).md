---
title: Ubuntu 16.04 安装 Nginx(源码)
id: 22
date: 2023-04-10 18:19:19
auther: FREEDOM
cover: https://codeblog.net/upload/2021/03/image-18537a93dc1b4fb59a699750ab025d46.png
excerpt: 注意首先提醒 我这里使用的 Ubuntu 16.04 ,Nginx 版本是 1.13.0如果你使用的系统是 Ubuntu 18.04 请别使用 1.13.0 版本的 nginx 换其他版本例如 1.15.2 只需将下面 1.13.0替换为 1.15.2 或者 其他版本号即可，可选版本可在这里找到
permalink: /archives/ubuntu1604an-zhuang-nginx-yuan-ma
categories:
 - code
tags: 
 - nginx
 - linux
---

注意:

首先提醒: 我这里使用的 Ubuntu 16.04 ,Nginx 版本是 1.13.0

如果你使用的系统是 Ubuntu 18.04 请别使用 1.13.0 版本的 nginx 换其他版本例如 1.15.2 只需将下面 1.13.0替换为 1.15.2 或者 其他版本号即可，可选版本可在这里找到 http://nginx.org/download/ 否则编译时会报错: 
![image.png](https://codeblog.net/upload/2021/03/image-18537a93dc1b4fb59a699750ab025d46.png)
接下来正式安装

因为我们要编译 nginx 源码安装, 所以先安装编译器
## 安装编译器

```shell
sudo apt-get install build-essential libtool
```
nginx 需要依赖一些软件包， 先安装这些依赖.

## 安装依赖
```shell
sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev  openssl
```
1. pcre (一个正则表达式库，nginx伪静态可以用到)
2. openssl (https连接需要)
3. zlib (开启gzip需要，一个提供数据压缩用的函式库)

## 编译安装 Nginx

我这里安装的是 1.13.0 , 你如果需要安装其他版本, 修改为对应版本号即可.

```shell
 cd /home/huoshan/software
 wget http://nginx.org/download/nginx-1.13.0.tar.gz
 tar -xvf nginx-1.13.0.tar.gz
 cd nginx-1.13.0
 ./configure
 make
 make install
```
1. 进入我的存放软件源码目录
2. 下载 nginx-1.13.0.tar.gz 源码包
3. 解压源码包
4. 进入解压后目录
5. 配置
6. 编译
7. 安装

好了如果操作过程没有报错，就安装成功了, ubuntu 中 nginx 源码安装默认安装到了 /usr/local/nginx 路径下, nginx 程序在 sbin 路径下. 配置文件在 conf 路径下, 日志在 logs 路径下

## 常用命令

经过上面步骤已经安装成功了，介绍一些 启动，重启 等常用命令.

```shell
sudo /usr/local/nginx #启动
或
sudo /usr/local/nginx -c /usr/local/nginx.conf
sudo /usr/local/nginx -t #检测配置文件是否正确 
sudo /usr/local/nginx -s stop #停止 
sudo /usr/local/nginx -s reload #重载配置文件
```

## 添加新模块

建议你安装完，别把源码包删掉, 如果以后需要添加新的模块需要重新编译 nginx 然后替换 nginx 时候需要用到.



