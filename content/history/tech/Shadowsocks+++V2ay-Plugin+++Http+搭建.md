---
title: Shadowsocks + V2ay-Plugin + Http 搭建
id: 4
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/images-5341b06756ac41f899bdc5f20d865813.jpeg
excerpt: 将一个域名解析到自己的服务器下载V2ray-Plugin插件安装shadowsocks-libev为shadowsocks集成v2ray-plugin插件启动安装客户端以及客户端集成V2ray-Plugin插件将一个域名解析到自己的服务器如果没有域名先需要购买一个域名，如果已有一个域名使用DNS解析
permalink: /archives/shadowsocksv2ay-pluginhttpda-jian
categories:
 - code
tags: 
 - v2ray
 - shadowsocks
---

## 内容列表

- 将一个域名解析到自己的服务器
- 下载V2ray-Plugin插件
- 安装shadowsocks-libev
- 为shadowsocks集成v2ray-plugin插件
- 启动
- 安装客户端以及客户端集成V2ray-Plugin插件

## 将一个域名解析到自己的服务器


如果没有域名先需要购买一个域名，如果已有一个域名使用DNS解析服务将域名解析到自己的服务器


## 下载V2ray-Plugin插件


这里[https://github.com/shadowsocks/v2ray-plugin/releases](https://github.com/shadowsocks/v2ray-plugin/releases) 是下载列表，找到与自己服务器匹配的下载包，由于我是在 Centos7.0上安装，是64位架构，所以下载 v2ray-plugin-linux-amd64-v1.3.1.tar.gz

```shell
wget https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.3.1/v2ray-plugin-linux-amd64-v1.3.1.tar.gz && tar -xvf v2ray-plugin-linux-amd64-v1.3.1.tar.gz
```
下载解压完成后，将解压出来的插件移动到 /usr/bin下

```shell
mv v2ray-plugin_linux_amd64 /usr/bin/v2ray-plugin
```

## 安装shadowsocks-libev

注意:安装期间不要安装 sample-obfs插件

```shell
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh
```
使用该脚本安装过程中，我遇到很多问题，如下：

1. raw.githubusercontent.com 域名国内被封锁无法下载脚本
	- 解决: /etc/hosts 添加 "199.232.68.133 raw.githubusercontent.com"
2. failed to install curl-devel(系统环境 centos7.6)  
	- 解决:将系统切换到 centos7.0 
3. 安装过程中无法下载libsodium-1.0.18.tar.gz导致安装失败
	- 手动下载该文件 wget https://github.com/jedisct1/libsodium/releases/download/1.0.18-RELEASE/libsodium-1.0.18.tar.gz
	- 手动编辑安装脚本 shadowsocks-all.sh 将第851行注释掉![image.png](https://codeblog.net/upload/2021/03/image-9283035371f1462a9d58b24a714d5edb.png)
4. 重新执行安装脚本

## 为shadowsocks集成V2ray-Plugin插件

```shell
vim /etc/shadowsocks-libev/config.json
```
```shell
{
"server":"0.0.0.0",
"server_port":8888,
"password":"vpntest",
"timeout":300,
"user":"nobody",
"method":"aes-128-gcm",
"fast_open":true,
"reuse_port":true,
"workers":1,
"nameserver":"114.114.114.114",
"plugin":"v2ray-plugin",
"plugin_opts":"server;path=/admin;mode=websocket;host=example.net",
"mode":"tcp_only"
}
```

关于配置我遇到的问题:

1. 当加密方式设置为 aes-256-gcm，日志报错: error: failed to handshake with 127.0.0.1: authentication error
	- 解决: 将其修改为 aes-128-gcm 后成功运行

## 启动

```shell
ss-server -c /etc/shadowsocks-libev/config.json > ss.log 2>&1 &
```

## 安装客户端以及客户端集成V2ray-Plugin


1. 下载windows客户端: https://github.com/shadowsocks/shadowsocks-windows/releases/download/4.4.0.0/Shadowsocks-4.4.0.185.zip
2. 下载v2ray-plugin插件的windows版本: https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.3.1/v2ray-plugin-windows-amd64-v1.3.1.tar.gz
3. 解压windows客户端
4. 解压v2ray-plugin插件包
5. 将v2ray-plugin插件拷贝到windwos客户端目录下
6. 然后启动shadowsocks客户端
7. 配置如下:
![image.png](https://codeblog.net/upload/2021/03/image-d323f74dffc54a538c3416e3093f6a91.png)


