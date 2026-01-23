---
title: 双启动:删除Windows系统盘,Ubuntu断电后无法重启
id: 130
date: 2023-04-10 18:19:28
auther: FREEDOM
cover: https://codeblog.net/upload/2022/09/updage_grub.png
excerpt: 之前我的电脑做过双启动，Ubuntu在一个硬盘装着，Windows在另一个硬盘装着。由于我几乎不用Windows系统，就将装有Windows系统的那个硬盘直接格式化了。刚开始也没有发现什么问题，电脑重启也可以直接进入Ubuntu系统
permalink: /archives/%E5%8F%8C%E5%90%AF%E5%8A%A8%E5%88%A0%E9%99%A4windows%E7%B3%BB%E7%BB%9F%E7%9B%98ubuntu%E6%96%AD%E7%94%B5%E5%90%8E%E6%97%A0%E6%B3%95%E9%87%8D%E5%90%AF
categories:
 - code
tags: 
 - grub
 - dual-boot
 - 双启动
 - ubuntu
---


之前我的电脑做过双启动，Ubuntu在一个硬盘装着，Windows在另一个硬盘装着。由于我几乎不用Windows系统，就将装有Windows系统的那个硬盘直接格式化了。刚开始也没有发现什么问题，电脑重启也可以直接进入Ubuntu系统。

## 问题

但是后来发现每次断电后都无法进去Ubuntu系统，而是显示Windows的引导界面。
刚开始，我是通过我的ubuntu USB启动盘进入，然后使用 boot-repair 程序解决的。

## 临时解决

通过用U盘制作的Ubuntu USB启动盘进入，然后"Try Ubuntu" 。随后在　Terminal　中输入：

```
sudo add-apt-repository ppa:yannubuntu/boot-repair && sudo apt update
sudo apt install -y boot-repair && boot-repair
```

然后点击弹出窗口中的＂Recommanded repair＂

最后重启系统，就可以直接进入Ubuntu 系统。

但是随后发现，断电后还是同样的问题，上面的操作还有再来一遍。

## 最终解决

其实问题的原因是，我之前只是将Windows系统的盘移除掉了，但是Grub 里的Windows引导文件夹还在，而且指向了那个已经被删除的硬盘。只要将这个引导文件夹删除，然后重新生成下引导启动就可以了。　

```
sudo rm -rf /boot/efi/EFI/Microsoft
sudo update-grub
```
![boot_efi](https://codeblog.net/upload/2022/09/boot_efi.png)
![updage_grub](https://codeblog.net/upload/2022/09/updage_grub.png)

