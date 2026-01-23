---
title: openssl命令行工具的简单使用
id: 40
date: 2023-04-10 18:19:25
auther: FREEDOM
cover: https://codeblog.net/upload/2021/05/openssl-flaw-enables-https-decryption-showcase_image-10-a-8834-0e6ee1743d8e4f8ba383a77ca15dff33.jpg
excerpt: 命令行工具openssl支持丰富的加密算法，对于日常的加密学习和工作非常方便安装linuxdebian/ubuntusudo apt install opensslcentossudo yum install opensslMacosbrew install opensslMacos上通常默认已经安
permalink: /archives/openssl%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7%E7%9A%84%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8
categories:
 - code
tags: 
 - 加密
 - openssl
 - 工具
---

命令行工具openssl支持丰富的加密算法，对于日常的加密学习和工作非常方便

## 安装

### linux

debian/ubuntu

```shell
sudo apt install openssl
```
centos

```shell
sudo yum install openssl
```

### Macos

```shell
brew install openssl
```

Macos上通常默认已经安装了一个LibreSSL ，所以默认会使用这个版本，但是由于该版本功能不全，可以通过openssl version 查看，如果是的话，需要在当前用户环境变量中覆盖一下，以使用刚才安装的版本：

```shell
echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> .bash_profile
source .bash_profile
openssl version
```
![](https://codeblog.net/upload/2021/05/image-bffacac362fb4255a19cae19e7cb3647.png)

## 命令参数使用简要

```shell
openssl help
```

查看某个命令的参数详情： openssl command -help 例如,查看命令 enc 的参数细节

```shell
openssl enc -help
```
![image.png](https://codeblog.net/upload/2021/05/image-460c372d0e8d43b0b1415e9d61eeb5d0.png)

## SHA256摘要

```shell
echo 'Hello World' | openssl dgst -sha256
```

![image.png](https://codeblog.net/upload/2021/05/image-0758d01f02784315a22c514d0a3b80f8.png)

## RSA 非对称加密

### 生成密匙对

#### 生成私匙

```shell
openssl genpkey -algorithm RSA -out privatekey.pem -pkeyopt rsa_keygen_bits:1024
```
#### 生成公匙

```shell
openssl rsa -pubout -in privatekey.pem -out publickey.pem
```

![image.png](https://codeblog.net/upload/2021/05/image-95a43b23d84b48fb8521faf4d4e33069.png)

#### 查看私匙和公匙的细节

```shell
openssl rsa -text -in privatekey.pem
openssl pkey -in publickey.pem -pubin -text
```

### 加密

```shell
echo test_data > message.txt
openssl rsautl -encrypt -inkey publickey.pem -pubin -in message.txt -out message.rsa
```
或者也可以结合为一条命令

```shell
echo testdata | openssl rsautl -encrypt -inkey publickey.pem -pubin -in message.txt -out message.rsa
```
这样加密后的文件内容
![image.png](https://codeblog.net/upload/2021/05/image-b80ff327a8564928bf322114a043c5bb.png)

### 解密

```shell
openssl rsautl -decrypt -inkey privatekey.pem -in message.rsa -out message.dec
```

### 签名

#### 生成签名

```shell
openssl dgst -sha256 -sign privatekey.pem -out signature.bin message.txt 
```
#### 签名验证

```shell
openssl dgst -sha256 -verify publickey.pem -signature signature.bin message.txt
```

## AES对称加密

这里以AES-256-CBC为例

### 加密

```shell
echo abc_test > test.txt
openssl enc -aes-256-cbc -in test.txt -out test.encry -pbkdf2
```
或者使用管道符合并为一条命令

```shell
echo abc_test | openssl enc -aes-256-cbc -in test.txt -out test.encry -pbkdf2
#如果要将加密后内容base64编码加一个 -a 参数
echo abc_test | openssl enc -aes-256-cbc -in test.txt -a -out test.encry -pbkdf2
```
### 解密

```shell
openssl enc -aes-256-cbc -d -in test.encry -pbkdf2
#如果加密内容经过base64编码加一个 -a 参数
openssl enc -aes-256-cbc -d -in test.encry -a -pbkdf2
```

注意：in参数可以不指定通过标准输入传入，out可以不指定输出文件会输出到标准输出





