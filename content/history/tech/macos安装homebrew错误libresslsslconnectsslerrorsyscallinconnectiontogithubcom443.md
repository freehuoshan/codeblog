---
title: macOS安装Homebrew遇到错误LibreSSL SSL_connect SSL_ERROR_SYSCALL in connection to github.com:443
id: 29
date: 2023-04-10 18:19:22
auther: FREEDOM
cover: https://codeblog.net/upload/2021/04/image-4fd97c35807f4577b35f93099b78dbf1.png
excerpt: 安装Homebrew官方提供了针对各操作系统的一键安装脚本git仓库地址：https//github.com/homebrew/install#uninstall-homebrew针对macOS,只需在terminal中运行/bin/bash -c &quot;$(curl -fsSL http
permalink: /archives/macos%E5%AE%89%E8%A3%85homebrew%E9%94%99%E8%AF%AFlibresslsslconnectsslerrorsyscallinconnectiontogithubcom443
categories:
 - code
tags: 
 - brew
---

## 安装Homebrew
官方提供了针对各操作系统的一键安装脚本git仓库地址：https://github.com/homebrew/install#uninstall-homebrew

针对macOS,只需在terminal中运行:

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
## LibreSSL SSL_connect: SSL_ERROR_SYSCALL

在执行该安装脚本是遇到错误：
```shell
fatal: unable to access 'https://github.com/Homebrew/homebrew-core/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443 
fatal: ambiguous argument 'refs/remotes/origin/master': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
```

解决：由于在中国大陆网络的问题，为git添加socks代理

```shell
macos@wangs-MacBook-Air % tee ~/.gitconfig <<HERE
heredoc> [http]
        sslBackend = openssl
        proxy = socks5://127.0.0.1:1089
heredoc> HERE
```

添加该文件后，如果再次尝试运行安装脚本，会遇到错误

## error: Not a valid ref: refs/remotes/origin/master

```shell
HEAD is now at 94ce3236a Merge pull request #11263 from Bo98/upgrade-source
error: Not a valid ref: refs/remotes/origin/master
fatal: ambiguous argument 'refs/remotes/origin/master': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
^Cfatal: index-pack failed
Failed during: /usr/local/bin/brew update --force --quiet
```

解决：这是由于多次重复安装造成冲突，需要执行卸载然后再次安装

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

还需要手动删除两个目录
```shell
sudo rm -rf /usr/local/Homebrew
sudo rm -rf /usr/local/var/homebrew
```

卸载成功。

现在再次执行安装脚本即可成功安装，如下
![image.png](https://codeblog.net/upload/2021/04/image-4fd97c35807f4577b35f93099b78dbf1.png)


