---
title: 设置Rust开发环境（安装、配置、编译、运行）
id: 133
date: 2023-04-10 18:19:30
auther: FREEDOM
cover: https://codeblog.net/upload/2022/10/image-1665392459618.png
excerpt: 安装Rust官方网站 提供了完整详细的安装教程官方默认推荐的方式是使用Rust的工具链管理器 rustup 来安装和管理rust版本及其各种工具。类Unix系统在类Unix系统,例如 Linux、MacOS上安装rustup非常简单，只需在终端运行以下命令(需要网络连接)：curl --proto 
permalink: /archives/%E8%AE%BE%E7%BD%AErust%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE
categories:
 - code
tags: 
 - rust
 - 安装
---



## 安装

[Rust官方网站](https://www.rust-lang.org) 提供了完整详细的[安装教程](https://www.rust-lang.org/tools/install)

官方默认推荐的方式是使用Rust的工具链管理器 [rustup](https://github.com/rust-lang/rustup) 来安装和管理rust版本及其各种工具。

### 类Unix系统

在类Unix系统,例如 Linux、MacOS上安装rustup非常简单，只需在终端运行以下命令(**需要网络连接**)：
```Shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

如果最后输出信息中显示

> Rust is installed now. Great!

说明rustup 已经安装成功。
除此之外你还需要安装一个链接器（linker）,rust需要使用它将编译输出组织到一个文件。通常系统中已经有安装，如果没有的话你可以安装一个C语言编译器，它通常包含一个链接器，C编译器也会用到，因为一些常用Rust包依赖于C语言代码，需要使用C编译器。Linux系统可以安装C语言编译器gcc或Clang,如果是Ubuntu系统可以安装build-essential包。
在MacOS系统上，可以通过以下命令安装C语言编译器:

```Shell
xcode-select --install
```

### 验证安装成功

在Rust 开发环境中，所有工具都安装在路径 ~/.cargo/bin 路径下，包括 rustc、cargo、rustup 等。
在安装rustup的过程中，会自动将上边的路径添加到环境遍历PATH中，要使环境遍历生效需要开启一个新的终端。

运行以下命令查看环境变量是否已经配置:

```Shell
echo $PATH
```

运行以下命令查看版本:
```Shell
rustc --version
```
![image-1665384495847](https://codeblog.net/upload/2022/10/image-1665384495847.png)

如果运行结果如图，说明安装成功。

### 更新和卸载

通过rustup安装后, 一旦由新的rust版本发布，只需通过以下命令即可升级:

```Shell
rustup update
```

如果要卸载 rust 和 rustup , 运行命令:

```Shell
rustup self uninstall
```

### 配置VScode

为了在 [Visual Studio Code](https://code.visualstudio.com/) 中写 Rust 代码，需要安装Rust官方开发的一个插件 [rust-analyzer](https://rust-analyzer.github.io/) , 它是Rust语言的 [LSP（ Language Server Protocol）](https://microsoft.github.io/language-server-protocol/) 实现，提供包括代码补全等很多特性，支持的代码编辑器包括 VScode、Emacs、Vim 等。

以前每个编程语言对于代码补全、定义跳转、悬浮等特性，每个IDE工具都要自己实现一遍，而这些实现又费事费力，LSP是针对这种杂乱不统一，定义的标准协议。

安装VScode后在插件市场，搜索 rust-analyzer ，然后安装即可

![image-1665386831851](https://codeblog.net/upload/2022/10/image-1665386831851.png)

## 编译运行

rust 语言是类似于 C/C++ 或者 Java 的编译型语言，所以编译和运行是分开的。编译是将源代码编译为二进制可执行文件。

### rustc

#### 编译
编译源代码文件 hello.rs ：
```Shell
rustc hello.rs
```

#### 运行
编译后生成 hello 二进制可执行文件，该文件可以直接运行
```Shell
./hello
```

关于编译器更详尽内容可以查看[这本书](https://doc.rust-lang.org/rustc/index.html)

### Cargo

cargo 是 rust 的编译系统和包管理器，作用和 Java 中的 maven 或 gradle 一样。它会自动下载项目依赖的包，并编译。如果是通过rustup安装的，cargo也已经安装了，如果通过其他方式的，运行以下命令查看是否有安装
```Shell
cargo --version
```
#### 创建项目

和 maven 一样可以通过命令创建想要的项目结构：

```Shell
cargo new hello_world
```
![image-1665390953094](https://codeblog.net/upload/2022/10/image-1665390953094.png)

cargo 也有配置文件 Cargo.toml ,配置依赖等。关于更详尽用法可以查看[这本书](https://doc.rust-lang.org/cargo/index.html)　

#### 编译

```Shell
cargo build
```
默认会将编译文件生成在目录　target/debug　下

#### 运行

```Shell
cargo run
```

这个命令会完成编译和运行的任务，如果要直接运行，可以直接运行无需提前只需编译命令，和maven类似。


#### 编译检查

```Shell
cargo check
```
这个命令会只需编译检查，如果编译有错误会显示，但不会生成编译文件，所以速度比cargo build 块。

#### 发布

```Shell
cargo build --release
```

与cargo build 不同的是，这个命令会将编译文件生成在 target/release目录下，并且在编译过程中会做更多优化让代码的运行速度更快，但是编译速度会更慢。






