---
title: 使用 Xray 实现 Vless + Wesocket + TLS 代理
id: 162
date: 2023-04-10 18:20:03
auther: FREEDOM
cover: https://codeblog.net/upload/2023/03/Featured.jpg
excerpt: 由于目前中国大陆网络环境的严格封锁，使用普通的 VPN 协议很难拥有一个较为稳定的自由网络环境，目前最为有效的手段还是通过 TLS 将流量包装起来最为隐秘和稳定 —— 可以通过 V2ray 或者 Xray 实现，Xray 是由于 V2ray 开发团队对于协议实现的分歧分支出来的一个版本。通过 Web
permalink: /archives/%E4%BD%BF%E7%94%A8xray%E5%AE%9E%E7%8E%B0vlesswesockettls%E4%BB%A3%E7%90%86
categories:
 - code
tags: 
 - v2ray
 - 代理
 - vpn
---

由于目前中国大陆网络环境的严格封锁，使用普通的 VPN 协议很难拥有一个较为稳定的自由网络环境，目前最为有效的手段还是通过 TLS 将流量包装起来最为隐秘和稳定 —— 可以通过 V2ray 或者 Xray 实现，Xray 是由于 V2ray 开发团队对于协议实现的分歧分支出来的一个版本。

通过 Websocket + TLS 包装的流量流通路径是：

（V2ray）代理客户端 ——> VPS服务器（Nginx——>V2ray代理服务器）——> 自由互联网

开始设置之前需要准备:

1. 一个域名
2. 一个在中国大陆可以 ping 得通的海外服务器（VPS）

在开始之前将域名的 DNS 指向到你的 VPS 服务器 IP.

## 安装并配置 Nginx

首先在VPS服务器上安装并配置 Nginx.

Nginx 会接受来自客户端的 Https 请求，然后将流量转发的 Xray 代理服务器.

安装 Nginx 和 TLS 工具

```
apt install -y nginx certbot python3-certbot-nginx vim ufw curl socat cron
```

然后配置 Nginx

```
vim /etc/nginx/sites-enabled/example.com
```

```
server {
    server_name example.com;
    location / {
        proxy_pass https://www.aliexpress.com;
    }
    location /articles {
          if ($http_upgrade != "websocket") {
              return 404;
          }
          proxy_redirect off;
          proxy_pass http://127.0.0.1:9993;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_set_header Host $host;
          # Show real IP in v2ray access.log
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    listen 80;
}
```

重新加载配置

```
systemctl reload nginx
```

然后检查服务器的防火墙是否在开启状态，如果是开启状态，放行 80, 443 端口
```
ufw status
ufw allow 80
ufw allow 443
```
然后为伪装网站配置 Https 证书

```
 certbot --nginx -d example.com
```

配置过程中需要输入你的邮箱、同意条款、是否同意你的邮件地址被分享、以及是否要自动https跳转等。
如果最后显示 Congratulation 字样，说明配置成功。

然后在浏览器中访问你的域名 https://example.com

![image-1678061500715](https://codeblog.net/upload/2023/03/image-1678061500715.png)

## 安装并配置 Xray

安装 xray

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install -u root
```

配置 xray

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
                  "id":"9a86a410-2c51-4019-8ad9-23ae2ccfb43b",
                  "level":0
               }
            ],
            "decryption":"none"
         },
         "streamSettings":{
            "network":"ws",
            "wsSettings":{
               "path":"/articles"
            }
         }
      }
   ],
   "outbounds":[
      {
         "protocol":"freedom",
         "settings":{
            
         }
      }
   ]
}
```

配置时最好自己生成一个新的UUID。

然后重启 xray

```
systemctl restart xray
```

检查下是否启动成功

```
ps -ef | grep xray
```

如果在运行状态 ——在VPS服务器的配置完成。

如果没有在运行状态，说明配置有问题，检查配置：

```
xray run -c /usr/local/etc/xray/config.json -test
```

查看是否有丢失或者多余逗号，或者括号等。
或者使用 Json 校验工具　https://jsonformatter.curiousconcept.com/　校验一下。

## 配置客户端

选择支持 Vless 的客户端

然后添加 Vless 服务器:

1. 地址(address): 你的域名 (example.com)
2. 端口(port): 443 (如果被封)
3. ID:  9a86a410-2c51-4019-8ad9-23ae2ccfb43b（根据自己的配置）
4. 加密(encryption)：none
5. 网络(Network): ws
6. 路径(path): /articles (根据自己的配置)
7. TLS: tls

## 被封的解决方案

有两种被封的可能性：

1. IP
2. 端口

如果是无法 ping 通说明 IP 被封了,可以通过使用Cloudflare的CDN, 或者更换IP。
如果只是端口不通，可以通过 iptables 映射端口即可:

```
iptables -t nat -A PREROUTING -p tcp --match multiport --dports 5601:5900 -j REDIRECT --to-port 443
```
然后更换端口即可。

完毕。
