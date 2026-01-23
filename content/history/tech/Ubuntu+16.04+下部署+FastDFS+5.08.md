---
title: Ubuntu 16.04 下部署 FastDFS 5.08
id: 24
date: 2023-04-10 18:19:19
auther: FREEDOM
cover: https://codeblog.net/upload/2021/03/image-b0410b111269420086b38ad5af17a79e.png
excerpt: FastDFS 目前更新很慢, 最新的版本可以从这里下载 https//sourceforge.net/projects/fastdfs/ ,github源码地址为 https//github.com/happyfish100/fastdfs 我下载的是最新版本 5.08 , FastDFS 是
permalink: /archives/ubuntu1604xia-bu-shu-fastdfs508
categories:
 - code
tags: 
 - ubuntu
 - fastdfs
---

FastDFS 目前更新很慢, 最新的版本可以从这里下载 https://sourceforge.net/projects/fastdfs/ ,github源码地址为 https://github.com/happyfish100/fastdfs 我下载的是最新版本 5.08 , FastDFS 是为互联网应用量身定做的一套分布式文件存储系统,非常适合用来存储用户图片、视频、文档等文件。对于互联网应用，和其他分布式文件系统相比，优势非常明显。具体情况大家可以看相关的介绍文档，包括 FastDFS 介绍 PPT 等等。

出于简洁考虑，FastDFS没有对文件做分块存储，因此不太适合分布式计算场景。

下载好后，server端分为两个部分，一个是tracker，一个是storage。顾名思义，前者调度管理，负载均衡，后者则是实际的存储节点。两个都能做成集群，以防止单点故障。以前的4.x版本依赖libevent，现在不需要了，只需要libfastcommon。安装步骤如下:

## 下载安装libfastcommon

```shell
git clone https://github.com/happyfish100/libfastcommon.git
cd libfastcommon/
./make.sh
./make.sh install
```
1. 首先 git clone 作者仓库中的 libfastcommon 库到本地, 如果你还没有安装 git 请先执行以下命令安装 git 
	-  ```shell
		apt install git
		```
2. 进入 libfastcommon文件夹
3. 编译源代码
4. 安装 libfastcommon

确认编译与安装没有错误, 64 位系统默认会复制到 /usr/lib64 路径下. 然后需要设置环境变量与创建软链接.

```shell
export LD_LIBRARY_PATH=/usr/lib64/
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
```
1. 创建环境变量
2. 创建软链接 

## 下载安装fastdfs

在命令行执行以下命令，下载 FastDFS:

```shell
wget https://sourceforge.net/projects/fastdfs/files/FastDFS%20Server%20Source%20Code/FastDFS%20Server%20with%20PHP%20Extension%20Source%20Code%20V5.08/FastDFS_v5.08.tar.gz
```
然后解压 .tar.gz --> 编译 --> 安装:

```shell
tar xzf FastDFS.tar.gz
cd FastDFS/
./make.sh
./make.sh install
```
1. 解压
2. 进入 FastDFS 文件夹
3. 编译源代码
4. 执行安装

Ubuntu 系统中, 默认会安装到 /usr/bin , 关于 fastdfs 的命令脚本都在该路径下
![image.png](https://codeblog.net/upload/2021/03/image-657758ecb19c40c5b6aa970d34893406.png)
并在 /etc/fdfs 下有三个以 .sample 结尾的示例配置文件
![image.png](https://codeblog.net/upload/2021/03/image-abdac47e69db412c89764d94a140cfa6.png)

## 修改配置文件
首先拷贝三个 .sample 文件为正式文件, 然后分别修改复制后的三个文件参数, 先让 FastDFS 跑起来, 其余参数调优的时候再考虑.

```shell
➜  ~ cd /etc/fdfs
➜  fdfs cp client.conf.sample client.conf
➜  fdfs cp tracker.conf.sample tracker.conf
➜  fdfs cp storage.conf.sample storage.conf
```
修改配置:

tracker.conf 中修改:

```shell
base_path=/opt/fastdfs #用于存放数据与日志的基路径。
```
storage.conf 中修改:

```shell
tracker_server=192.168.1.181:22122 #指定tracker服务器地址。
base_path=/opt/fastdfs #用于存放数据与日志的基路径。
store_path0=/opt/fastdfs/storage #存放数据，若不设置默认为前面那个
```
Note:

1. FastDFS 是 storage 追踪 tracker， 所以在 storage 配置文件中指定 tracker ip 地址.
2. tracker_server 配置项 ip 地址不可以为 127.0.0.1 , 否则无法启动.
3. 如果需要指定多个 tracker, 添加多行即可， 例如:

```shell
tracker_server=192.168.1.181:22122
tracker_server=192.168.1.182:22122
```
client.conf 中修改:
```shell
base_path=/opt/fastdfs #用于存放数据与日志基路径。
tracker_server=192.168.1.181:22122 #指定tracker服务器地址
```
## 启动 tracker 和 storage

```shell
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf
```

## 检查进程

```shell
ps -ef | grep fdfs
```
![image.png](https://codeblog.net/upload/2021/03/image-1c8f046ee6bc4821a36a227fe5eed03e.png)
可以看到 tracker 与 storage 都启动 OK 了. 这时我们可以看到 FastDFS 日志与数据路径结构如下:

![image.png](https://codeblog.net/upload/2021/03/image-8628001b71784b8ea954e3c227dc2d78.png)

如果启动失败, 可以到 logs 路径下查看相应日志.

## 上传/删除测试

使用自带的 fdfs_test 测试, 使用方式如下:

```shell
fdfs_test /etc/fdfs/client.conf upload /home/huoshan/图片/webwxgetmsgimg.jpg
```
![image.png](https://codeblog.net/upload/2021/03/image-b4c48b80362a4e528ab7790124601e0a.png)

使用 fdfs_delete_file 来测试删除文件, 使用方式如下:
```shell
fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/01/wKgfmFmuHkSALvZdAAEhg1sJVzs863.jpg
```
OK， 我们上传与删除测试都 OK 了。 我们的图片上传到了 /opt/fastdfs/storage/data 路径下的子目录里, 默认 FastDFS 会在该路径下生成 256 * 256 个子目录. 默认 FastDFS 会在各目录里依次存放 . 所以在子目录 00/00 应该可以看到我们刚才上传的文件。
![image.png](https://codeblog.net/upload/2021/03/image-b0410b111269420086b38ad5af17a79e.png)

在我们上边上传成功后, 会为我们返回一个 example file url , 这个 url 现在是无法在 浏览器中打开的, 需要配置 nginx 使用.

在 nginx 中添加 fastdfs-nginx-module 模块才可以打开. 如果需要可以查看我的另一篇文章 Ubuntu 16.04 安装 fastdfs-nginx-module 模块