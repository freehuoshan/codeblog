---
title: Linux上设置蓝牙耳机的工作模式
id: a172462f-24b7-47e3-95bb-1605dead344b
date: 2025-03-25 11:34:07
auther: FREEDOM
cover: 
excerpt: Linux 上蓝牙耳机只能在 Output Devices 中被识别，而在 Input Devices 中不被显示的问题 蓝牙耳机有两种模式 A2DP 和 HFP 模式。 默认情况下会使用 A2DP 模式，所以只能作为输出声音的设备，如果想要同时作为麦克风输入设备，需要以 HFP 模式工作。需要切换
permalink: /archives/1742873646440
categories:
 - code
tags: 
 - lan-ya-er-ji
 - linux
---

Linux 上蓝牙耳机只能在 Output Devices 中被识别，而在 Input Devices 中不被显示的问题

蓝牙耳机有两种模式 A2DP 和 HFP 模式。
默认情况下会使用 A2DP 模式，所以只能作为输出声音的设备，如果想要同时作为麦克风输入设备，需要以 HFP 模式工作。需要切换为其为 HFP 模式

`pactl set-card-profile bluez_card.1C_52_16_DE_66_70 handsfree_head_unit`

bluez_card.1C_52_16_DE_66_70 handsfree_head_unit 是蓝牙设备名称，可以通过命令

`pactl list cards`

找到