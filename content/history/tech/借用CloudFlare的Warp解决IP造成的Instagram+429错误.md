---
title: 借用CloudFlare的Warp解决IP造成的Instagram 429错误
id: 161
date: 2023-04-10 18:20:03
auther: FREEDOM
cover: https://codeblog.net/upload/2023/03/image.png
excerpt: 最近遇到了无法访问 instagram 的问题，检查发现几乎每条请求都返回 429 错误码，这个错误码的含义是客户端发送了太多请求，instagram 的服务器认为我们是恶意机器人，所以block掉了我们的请求。因为我是通过 v2ray 访问的，所以一开始以为是我的配置有问题，最后发现也不是配置问题
permalink: /archives/%E5%80%9F%E7%94%A8cloudflare%E7%9A%84warp%E8%A7%A3%E5%86%B3ip%E9%80%A0%E6%88%90%E7%9A%84instagram429%E9%94%99%E8%AF%AF
categories:
 - code
tags: 
 - instagram
 - iplocation
 - warp
 - cloudflare
 - 429
---

最近遇到了无法访问 instagram 的问题，检查发现几乎每条请求都返回 **429** 错误码，这个错误码的含义是客户端发送了太多请求，instagram 的服务器认为我们是恶意机器人，所以block掉了我们的请求。
![image](https://codeblog.net/upload/2023/03/image.png)

因为我是通过 v2ray 访问的，所以一开始以为是我的配置有问题，最后发现也不是配置问题造成重发请求引起的, 经过一番研究后发现应该是我的**代理 IP 是 Vultr** 的，而且像使用 **DigitalOcean** 这类**数据中心的 IP** 都会有这个问题。

解决这个问题也很容易，就是换用其他服务器提供商然后尝试，然而呢我的情况是如果只针对访问 Instagram 时使用不被它屏蔽的 IP , 其他情况照常使用我平常的 ip， 才是我最想要的方案。

经过一番研究后发现，CloudFlare的 Warp 的出口 IP 通常不会被屏蔽，而且 Warp 使用的 Wireguard 协议，我常用的 Xray 支持 Wireguard 协议，所以我这里的**解决方案**就是使用 Xray + Warp 做一个链式代理，然后添加一个路由：**访问 Instagram 时使用 Warp 出口**。

## 创建 Warp 账户

1. ssh 登录自己的代理服务器
2. 下载Warp命令行工具 [wgcf](https://github.com/ViRb3/wgcf/releases) 
我使用的Linux系统，所以: 
```
wget https://github.com/ViRb3/wgcf/releases/download/v2.2.15/wgcf_2.2.15_linux_386 -O wgcf && chmod +x wgcf
```
3. 注册账户

```
./wgcf register --accept-tos
```
4. 使用创建的账户生成Wireguard配置文件
```
./wgcf generate
```

生成结果截图：

![image-1677722011080](https://codeblog.net/upload/2023/03/image-1677722011080.png)

## 配置 Xray

1. 安装 Xray
```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
```
2. 配置 Xray
```
vim /usr/local/etc/xray/config.json
```

```

{
    "log":{
        "access":"/var/log/xray/access.log",
        "error":"/var/log/xray/error.log",
        "loglevel":"Warning"
    },
    "inbounds":[
        {
            "port":9993,
            "protocol":"vless",
            "settings":{
                "clients":[
                    {
                        "id":"77ccb553-1a24-4abf-88d6-fbd31450a29e",
                        "level":0
                    }
                ],
                "decryption":"none"
            },
            "sniffing": {
                "enabled": true,
                "destOverride": ["http", "tls"]
            },
            "streamSettings":{
                "network":"ws",
                "wsSettings":{
                    "path":"/test"
                }
            }
        }
    ],
    "outbounds":[
        {
            "tag":"direct",
            "protocol":"freedom",
            "settings":{
            }
        },
        {
            "tag":"WARP",
            "protocol":"wireguard",
            "settings":{
                "secretKey":"KB3aMYGyxRngyeV1Ckr36e/rshZhv7VnpT8kbpx1cFk=",
                "address":[
                    "172.16.0.2/32",
                    "2606:4700:110:8962:1470:1950:5c50:8a37/128"
                ],
                "peers":[
                    {
                        "publicKey":"bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=",
                        "endpoint":"engage.cloudflareclient.com:2408"
                    }
                ]
            }
        }
    ],
    "routing":{
        "domainStrategy":"AsIs",
        "rules":[
            {
                "type":"field",
                "domain":[
                    "domain:www.instagram.com",
                    "domain:instagram.com",
                    "domain:iplocation.com",
                    "domain:iplocation.net"
                ],
                "outboundTag":"WARP"
            }
        ]
    }
}
```

## 最后结果

![image-1677740570863](https://codeblog.net/upload/2023/03/image-1677740570863.png)