---
title: R2S 路由器上 V2ray 配置
id: d1541739-b543-455d-bf1d-39426456c2e7
date: 2025-03-12 16:57:03
auther: FREEDOM
cover: 
excerpt: R2S 路由器上使用 V2ray 配置 /etc/config/firewall config redirect	option name 'V2ray'	option proto 'tcp udp'	option src 'lan'	option src_dport '1-65535'	o
permalink: /archives/1741769823055
categories:
 - code
tags: 
 - v2ray
 - r2s
---

R2S 路由器上使用 V2ray 配置

`/etc/config/firewall `

```js
config redirect
	option name 'V2ray'
	option proto 'tcp udp'
	option src 'lan'
	option src_dport '1-65535'
	option dest_ip '192.168.2.1'
	option dest_port '1081'
	option target 'DNAT'
```

`/etc/init.d/firewall restart`