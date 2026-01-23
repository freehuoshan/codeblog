---
title: Vagrant在Macos中错误VBoxManage error Failed to create the host-only adapter解决
id: 30
date: 2023-04-10 18:19:22
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/Screen%20Shot%202021-04-28%20at%204.33.59%20PM-80a46a822bc7436abc5bcace6fd31c88.png
excerpt: 环境 Macos11.4 Beta 上使用vagrant问题 安装vagrant后运行 vagrant up ，遇到错误There was an error while executing `VBoxManage`, a CLI used by Vagrantfor controlling Vi
permalink: /archives/vagrant%E5%9C%A8macos%E4%B8%AD%E9%94%99%E8%AF%AFvboxmanageerrorfailedtocreatethehost-onlyadapter%E8%A7%A3%E5%86%B3
categories:
 - code
tags: 
 - macos
 - vagrant
---

**环境**: Macos11.4 Beta 上使用vagrant
**问题**: 安装vagrant后运行 vagrant up ，遇到错误

```shell
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["hostonlyif", "create"]

Stderr: 0%...
Progress state: NS_ERROR_FAILURE
VBoxManage: error: Failed to create the host-only adapter
VBoxManage: error: VBoxNetAdpCtl: Error while adding new interface: failed to open /dev/vboxnetctl: No such file or directory
VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component HostNetworkInterfaceWrap, interface IHostNetworkInterface
VBoxManage: error: Context: "RTEXITCODE handleCreate(HandlerArg *)" at line 95 of file VBoxManageHostonly.cpp
```
![image.png](https://codeblog.net/upload/2021/04/image-7960d6684bc14fad875a664b0d028e06.png)

**解决**：

1.进入System Preferences -> 点击左下角🔐图标使其处于开启状态 -> 点击Allow

![Screen Shot 20210428 at 4.33.59 PM.png](https://codeblog.net/upload/2021/04/Screen%20Shot%202021-04-28%20at%204.33.59%20PM-80a46a822bc7436abc5bcace6fd31c88.png)

2.然后弹出要求重新启动对话框，重启后，即可正常使用vagrant

![Screen Shot 20210428 at 4.34.33 PM.png](https://codeblog.net/upload/2021/04/Screen%20Shot%202021-04-28%20at%204.34.33%20PM-84701341a05f41f1afcbf5bde51b604a.png)