---
title: Dell XPS13 ERROR HD Drive not installed 解决
id: 387bbdc2-40cf-44ad-ac76-78b15622a83c
date: 2023-11-14 12:27:52
auther: FREEDOM
cover: 
excerpt: Dell XPS13断电导致无法正常启动，报错"HD Drive not installed"的解决流程 平常我的笔记本电脑XPS13都是供电状态，但是前几天由于晚上工作时没有插充电器，然后电池电量剩余非常少，工作完也没有充电，第二天起来打开电脑发现关机了，尝试启动直接报错"HD Drive not
permalink: /archives/1699936062549
categories:
 - code
tags: 
 - arch
 - linux
---

**Dell XPS13断电导致无法正常启动，报错"HD Drive not installed"的解决流程**

平常我的笔记本电脑XPS13都是供电状态，但是前几天由于晚上工作时没有插充电器，然后电池电量剩余非常少，工作完也没有充电，第二天起来打开电脑发现关机了，尝试启动直接报错"HD Drive not installed",然后就是一脸闷逼状。

网络上搜索到的解决方式:

1. 将后背壳打开，电池与主板的接口断开，然后按住电源30秒，最后尝试重启
2. 用Diagnostic检测，但是在我的情况检测依然报错"HD Drive not installed"
3. 如果以上还是无法解决说明硬盘已经损坏，只能替换新的硬盘

经过上边尝试，就在我以为我的硬盘已经物理损坏，已经订购了新硬盘的情况下。无聊之中尝试用我的USB安装器检测安装Windows时是否可以检测到我的硬盘呢 —— 靠，它竟然可以检测到🤯

然后进入我的USB启动器中的Arch，然后运行`lsblk` 发现还是不显示我的硬盘。
最后发现了这个 https://wiki.archlinux.org/title/Partitioning#Drives_are_not_visible_when_firmware_RAID_is_enabled

## 将SATA Operation 切换为 AHCI

开机时，连续按`F12` 进入BIOS Setup,在 System Configuration 下的SATA Operation切换为 AHCI 模式。

这时再使用`lsblk`命令就会显示硬盘了。

但是现在BIOS的启动列表中还是没有看到我的Arch系统。所以我怀疑是Grub启动损坏了，然后我进入了我的USB Live Arch系统。

## 修复EFI GRUB 启动

进入Live 系统后，首先使用iwctl连接网络，因为需要安装`grub` `efibootmgr`
现在使用 `lsblk` 可以看到我的系统的分区了。我的系统有两个分区

/boot -> /dev/nvmen1p1
/ -> /dev/nvmen1p2

1. 然后将这两个分区分别挂载到 /mnt 和 /mnt/boot下:
    ```shell
    mount /dev/nvmen1p2 /mnt
    mount --mkdir /dev/nvmen1p1 /mnt/boot
    ```

2. 然后进入我们硬盘中的系统:
    ```shell
    sudo arch-chroot /mnt
    ```
3. 重新安装 GRUB:
   
    ```shell
    pacman -S grub efibootmgr
    grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=grub
    ```
4. 生成GRUB配置
   
    ```shell
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
5. 验证EFI Boot 条目
    
    ```shell
    efibootmgr --v
    ```
6. 退出chroot
   
    ```shell
    exit
    reboot
    ```
 
现在重启还是无法在 BIOS 启动项中看到我们的系统。我们需要在BIOS Setup  中添加一个 Boot Sequence

## 添加 Boot Sequence

进入BIOS Setup中，然后在 Settings -> General -> Boot Sequence 中，添加 Boot Option.

![photo_2023-11-14_12-45-40.jpg](/upload/photo_2023-11-14_12-45-40.jpg)

完成后重启，就可以正常进入硬盘中的系统了。













