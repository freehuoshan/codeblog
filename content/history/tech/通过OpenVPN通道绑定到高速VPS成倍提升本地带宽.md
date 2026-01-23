---
title: 通过OpenVPN通道绑定到高速VPS成倍提升本地带宽
id: 25
date: 2023-04-10 18:19:19
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/hFsfSGCN_400x400-e88fb1bb79f64db1bc1b17b8937a9572.jpg
excerpt: 对于生活在互联网基础设施较差的地区，即使是时至今天网络带宽仍然可能很低。但是在youtube上意外看到一个家伙提出一种解决方案，可以利用Openvpn通过多个网络接口（例如，本地电脑同时连接多个wifi,或路由器多个WAN口接入）绑定到远程高速带宽VPS中的多个openvpn实例来提升本地带宽。这里
permalink: /archives/tong-guo-openvpn-tong-dao-bang-ding-dao-gao-su-vps-cheng-bei-ti-sheng-ben-de-dai-kuan
categories:
 - code
tags: 
 - openvpn
 - 提速带宽
 - vpn
---

对于生活在互联网基础设施较差的地区，即使是时至今天网络带宽仍然可能很低。但是在youtube上意外看到一个家伙提出一种解决方案，可以利用Openvpn通过多个网络接口（例如，本地电脑同时连接多个wifi,或路由器多个WAN口接入）绑定到远程高速带宽VPS中的多个openvpn实例来提升本地带宽。

这里是他写的用于实现这一方案的 git 仓库地址:

https://github.com/onemarcfifty/openvpn-bonding

这里是他详细阐述其原理和操作步骤的youtube 视频地址:

https://www.youtube.com/watch?v=I08A4-PWawk