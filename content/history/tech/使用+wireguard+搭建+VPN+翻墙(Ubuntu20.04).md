---
title: 使用 wireguard 搭建 VPN 翻墙(Ubuntu20.04)
id: 5
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/images-c9bb644e6d024c36adffea671bf824cc.png
excerpt: 这里在 ubuntu20.04 上搭建，非常简单。小于该版本会遇到内核不支持或其他问题。购买服务器安装wireguard配置启动和停止使用购买服务器这里我使用的是 vultr 的 5$/month ，非常便宜。 点击该链接 https//www.vultr.com/?ref=8411854-6G 
permalink: /archives/shi-yong-wireguardda-jian-vpnfan-qiang-ubuntu2004
categories:
 - code
tags: 
 - wireguard
---

这里在 ubuntu20.04 上搭建，非常简单。小于该版本会遇到内核不支持或其他问题。

- 购买服务器
- 安装wireguard
- 配置
- 启动和停止
- 使用

## 购买服务器

这里我使用的是 vultr 的 5$/month ，非常便宜。 点击该链接 https://www.vultr.com/?ref=8411854-6G 注册并购买。
![image.png](https://codeblog.net/upload/2021/03/image-5ea19ea6b29e45b19cd2dd0bf00d8288.png)
购买完成并启动服务器完成后， ssh 登录。 linux/macos 用户直接打开终端 ,windows 用户使用 xshell 或者其他终端。 输入 ssh root@ip , ip 替换为刚购买的 ip 地址，输入密码。

## 安装 wireguard

```shell
apt update
apt install wireguard
```

## 配置

分别为服务器和客户端生成 wireguard 私匙和公匙。类似于 ssh 公私匙，公匙交给对方用于加密数据，私匙用于解密数据。

```shell
cd /etc/wireguard
wg genkey | tee s_privatekey | wg pubkey > s_pubkey
wg genkey | tee c_privatekey | wg pubkey > c_pubkey
```
创建文件 /etc/wireguard/wg0.conf

```shell
[Interface]
Address = 192.168.1.1/24
DNS = 8.8.8.8
ListenPort = 37750
PrivateKey = uDL86EflkdrwvRBEjWIKV6VW81VgoyVu1UMRjgR1I1w=
 
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
 
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
 
[Peer]
PublicKey = OjGfQf0IdLhp3M1mej2+tmIbKYymfQ1bm1bZivBbux8=
AllowedIPs = 192.168.1.2/32
```

1. address 网络不应与服务器上其他网络有冲突。
2. PrivateKey 为上面生成的 s_privatekey 文件内容，用于接受客户端加密数据时解密使用。
3. PostUp/PostDown 用于将 wg0 流量转发到主网卡 ens3 (我测试的机器上主网卡地址不是常用的 eth0,所以要根具实际情况修改)
4. PublicKey 为上面生成的　c_pubkey 文件内容，用于向客户端发送数据时加密使用。

打开IP转发

```shell
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.confsysctl
```
创建文件 /etc/wireguard/client.conf

```shell
[Interface]
PrivateKey = GMayU0if9hRD8ApQkkYIqCBBWBcjuSkZD+p3irmTb3Q=
Address = 192.168.1.2/24
DNS = 8.8.8.8
BlockDNS = true
MTU = 1200
 
[Peer]
PublicKey = HoELeYIncFQ5Mled8BBCdEBRLrwZzS2h3eg0n6tsWEw=
Endpoint = 137.220.39.199:37750
AllowedIPs = 0.0.0.0/0, ::0/0
PersistentKeepalive = 25
```
1. PrivateKey 为上边生成的 c_privatekey  文件内容，用于接收服务器数据时解密使用
2. PublicKey 为上边生成的 s_publickey 文件内容，用于想服务器发送数据时加密使用
3. Endpoint 为服务器公网 IP 和 wireguard 监听端口
4. AllowsIps 接收所有网卡流量

## 启动和停止

服务器启动 wireguard

```shell
root@test:~# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 192.168.1.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] resolvconf -a wg0 -m 0 -x
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
root@test:~#
```

启动可能发生错误:/usr/bin/wg-quick: line 32: resolvconf: command not found

![image.png](https://codeblog.net/upload/2021/03/image-bd52951765024e9c9e3013fcac32ff29.png)

解决:
```shell
sudo apt install openresolv
sudo apt install resolvconf
```
查看
```shell
root@test:~# ifconfig
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
inet 137.220.39.199  netmask 255.255.254.0  broadcast 137.220.39.255
inet6 fe80::5400:2ff:fed3:e19a  prefixlen 64  scopeid 0x20<link>
ether 56:00:02:d3:e1:9a  txqueuelen 1000  (Ethernet)
RX packets 375035  bytes 268978934 (268.9 MB)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 345499  bytes 258541920 (258.5 MB)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
 
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
inet 127.0.0.1  netmask 255.0.0.0
inet6 ::1  prefixlen 128  scopeid 0x10<host>
loop  txqueuelen 1000  (Local Loopback)
RX packets 148  bytes 12586 (12.5 KB)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 148  bytes 12586 (12.5 KB)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
 
wg0: flags=209<UP,POINTOPOINT,RUNNING,NOARP>  mtu 1420
inet 192.168.1.1  netmask 255.255.255.0  destination 192.168.1.1
unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 1000  (UNSPEC)
RX packets 0  bytes 0 (0.0 B)
RX errors 0  dropped 0  overruns 0  frame 0
TX packets 0  bytes 0 (0.0 B)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
```shell
root@test:~# wg show wg0
 interface: wg0
  public key: HoELeYIncFQ5Mled8BBCdEBRLrwZzS2h3eg0n6tsWEw=
 private key: (hidden)
  listening port: 37750
 
 peer: OjGfQf0IdLhp3M1mej2+tmIbKYymfQ1bm1bZivBbux8=
  allowed ips: 192.168.1.2/32
```
停止
```shell
root@test:~# wg-quick down wg0
[#] ip link delete dev wg0
[#] resolvconf -d wg0 -f
[#] iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
root@test:~#
```
使用

我使用 tunsafe 客户端使用 wireguard ，支持各主流设备, windows/macos/linux/android/ios. https://tunsafe.com/user-guide

- windows https://tunsafe.com/download 
	- 下载安装完成后。
	- 导入上边在服务器中创建的 /etc/wireguard/client.conf.
- Android https://tunsafe.com/user-guide/android  
	- 下载APK 并安装。 
	- 导入文件 /etc/wireguard/client.conf
- IOS https://tunsafe.com/user-guide/ios 
	- 进入 apple store 安装。
	- 导入文件 /etc/wireguard/client.conf
- macos/linux https://tunsafe.com/user-guide/linux 
	- 按照步骤下载代码编译安装。
	- 下载文件 /etc/wireguard/client.conf。　
	- sudo tunsafe start -d ./client.conf 启动。
