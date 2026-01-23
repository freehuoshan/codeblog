---
title: nginx failed (28 No space left on device)-error-fixed
id: 6
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/download-edfa2ed2ee33496b9a90808f69cea0d1.png
excerpt: error 7086#7086 *74580 open() &quot;/var/lib/nginx/proxy/4/22/0000007224&quot; failed (28 No space left on device) while reading upstream,but the d
permalink: /archives/nginxfailed28nospaceleftondevice-error-fixed
categories:
 - code
tags: 
 - nginx
 - linux
---

error: 
```shell
 7086#7086: *74580 open() "/var/lib/nginx/proxy/4/22/0000007224" failed (28: No space left on device) while reading upstream,
```

but the disk space the linux have a lot free: df -h

```shell
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           393M   62M  331M  16% /run
/dev/sda2        20G  9.1G  9.2G  50% /
tmpfs           2.0G   32M  1.9G   2% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/sda1       487M   36M  423M   8% /boot
/dev/sda3       1.8T  1.7T   36G  98% /home
tmpfs           393M     0  393M   0% /run/user/0
```
so disk space not the problem. but let's check indoes : df -i

```shell
Filesystem        Inodes   IUsed     IFree IUse% Mounted on
udev              501260    1463    499797    1% /dev
tmpfs             502501    6338    496163    2% /run
/dev/sda2        1281120 1281120         0  100% /
tmpfs             502501      79    502422    1% /dev/shm
tmpfs             502501       4    502497    1% /run/lock
tmpfs             502501      14    502487    1% /sys/fs/cgroup
/dev/sda1         131072     299    130773    1% /boot
/dev/sda3      120750080 2014363 118735717    2% /home
tmpfs             502501       5    502496    1% /run/user/0
```
that's the reason:

 just clear the the indoes under path / , like delete file in /


