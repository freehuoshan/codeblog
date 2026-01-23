---
title: Rust编程语言的了解
id: 132
date: 2023-04-10 18:19:30
auther: FREEDOM
cover: https://codeblog.net/upload/2022/10/index.png
excerpt: Rust 这门语言变得越来越流行，不管是浏览Twitter还是技术博客，经常会有提及到这门语言的火热。尤其是就在昨天 2022/10/03 ， Rust继C语言进入了Linux内核，Linux内核的开发以后再也不是只能使用C,也可以选择Rust编写了，C++多少年都没有被支持，但是Rust这位年轻轻
permalink: /archives/rust%E7%BC%96%E7%A8%8B%E8%AF%AD%E8%A8%80%E7%9A%84%E4%BA%86%E8%A7%A3
categories:
 - code
tags: 
 - rust
 - 新语言学习
---

Rust 这门语言变得越来越流行，不管是浏览Twitter还是技术博客，经常会有提及到这门语言的火热。尤其是就在昨天 2022/10/03 ， Rust继C语言进入了[Linux内核](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=8aebac82933ff1a7c8eede18cab11e1115e2062b)，Linux内核的开发以后再也不是只能使用C,也可以选择Rust编写了，C++多少年都没有被支持，但是Rust这位年轻轻的妙龄女子却被以苛刻著称的Linus相中了，可见必有其过人之处。![image-1664854581128](https://codeblog.net/upload/2022/10/image-1664854581128.png)

一图胜千言, lol ：

![image-1664854714030](https://codeblog.net/upload/2022/10/image-1664854714030.png)

从上面这具有代表性的两张图可以发现 Rust 主要对标的C/C++这两个兄弟，这两个兄弟之所以几十年屹立不倒，主要在系统编程、高负载、嵌入式编程这类对于性能要求高，对于功耗要求严苛的场景下一直没有更好的语言可以选择。但是现在Rust正跃跃欲试要在这些领域取代它们。

## 相比C/C++

### C/C++的问题
对于系统层级的工作，比如处理内存管理、并发，一直都是被人认为是只有少数一些程序员，在这些领域拥有数年的经验，才能避免一些臭名昭著的陷阱，然而即使是这些经验丰富的老手非常小心的情况下，他们的代码也常常会崩溃，比如最恶心的空指针问题。

### Rust的优点
然而，Rust 消除了这些陷阱，并提供了一系列工具让你在编译阶段就发现问题。当你通过Rust处理一些低层级的控制，不仅可以避免在C/C++中的那些晦涩的陷阱，还可以写出速度更快，内存利用更有效的代码。用Rust 写多线程时，它会在编译期间就为你捕获常见的错误，让你可以真正专注于你需要处理的事情上。

Rust强调的关键词有：性能、类型安全、并发。它在性能上不逊色于C/C++,虽然它也是内存安全的语言但是它没有使用垃圾回收器，也没有使用引用计数的方式，它的编译器会在编译的时候检查所有引用的对象生命周期和变量的范围，确保所有引用都指向有效内存。

虽然它是一个非常低级的语言，但是却拥有很多高级语言才有的功能特性，像函数式编程、多态、更高级别的抽象。Rust 非常适合处理像系统编程这类低层级的工作，但却远远不局限于此，可以使用它完成写命令行工具、Web服务器等几乎任何你想要实现的任务。

> Rust既拥有低级语言的速度和性能，又拥有传统上高级语言才具有的函数式编程、面对对象的高度抽象等特性，而且规避掉了传统低级语言开发时的各种隐式陷阱。

### 原因
而Rust只所以能够将一直以来，低级语言的速度和性能与高级语言的开发效率，得冲突打破，原因是它没有使用传统高级语言使用的垃圾回收和引用计数的方式来管理引用，而是通过引用规则和权限的方式在编译时通过编译器跟踪检查引用的方式，这样既规避了C/C++中那些臭名昭著的问题，实现了可以媲美高级语言的开发效率，又使得[运行时环境](https://en.wikipedia.org/wiki/Runtime_system)保持简单，保证了其性能和速度。

## 历史发展

Rust也是吸收百家之所长，主要受到 SML、 OCaml、 C++、Cyclone、Haskell、 Erlan这些编程语言的影响。

最初是由在 Mozilla (没错，就是发布firefox的那个) 工作的程序员Graydon Hoare在2006年设计的

- 2009年受到了Mozilla官方的赞助
- 2010年官方正式公布了这个项目，在同一年将最初用OCaml写的编译器用Rust重写
- 2011年用Rust重写的编译器rustc成功编译了它自己
- 2012发布了其第一个版本编号Rust 0.1
- 2013-2014 版本0.2-0.4，这几个版本之间，类型系统做了非常多改变，有些功能添加了又删除，也正是由于这种不稳定，这个语言的采用率也非常低
- 2015年正是发布了第一个稳定版本Rust 1.0
- 2020年受到武汉肺炎影响，Mozilla大幅裁员，解散掉了 Servo（一个用Rust写的浏览器引擎）整个团队，由于这个团队的许多成员是Rust核心的主要贡献者，所以也影响到了Rust
- 2021年Rust基金会成立，由五个创始公司成员组成（亚马逊、谷歌、微软、华为、Mozilla）

## 基金会

Rust 基金会是一个位于美国的非营利会员组织，由五个（亚马逊、谷歌、微软、华为、Mozilla）创始成员建立

### 治理团队

Rust项目由负责不同子模块开发的团队组成

- 核心团队负责管理Rust整体发展方向、子团队的领导、以及任何跨团队交叉的问题
- 编译器团队负责开发和管理编译器内部和优化
- 语言团队负责设计和帮助实现新的语言特性

## 学习资源

维基百科介绍
- 英文: [https://en.wikipedia.org/wiki/Rust_(programming_language)](https://en.wikipedia.org/wiki/Rust_(programming_language))
- 中文:[https://zh.wikipedia.org/wiki/Rust](https://zh.wikipedia.org/wiki/Rust)

官方网站
- 英文: [https://www.rust-lang.org/](https://www.rust-lang.org/)
- 中文:[https://www.rust-lang.org/zh-CN/](https://www.rust-lang.org/zh-CN/)

官方提供的完整介绍系统学习Rust的一本书：The Rust Programming Language
- 英文:[https://doc.rust-lang.org/book/](https://doc.rust-lang.org/book/)
- 中文翻译: [https://kaisery.github.io/trpl-zh-cn/](https://kaisery.github.io/trpl-zh-cn/)

对于不习惯阅读几百页书籍的学习者，官方还提供了一个简化版本的[Rust by Example](https://doc.rust-lang.org/stable/rust-by-example/) ，（[中文翻译版本](https://rustwiki.org/zh-CN/rust-by-example/)） , 通过示例学习，相比于上边的那本完整版书籍，这个简化版本说明解释的文字很少，主要通过代码示例学习。

Rust官方提供了一个通过示例练习的项目 [rustlings](https://github.com/rust-lang/rustlings/), 每个提供的示例都会有编译错误，直到你解决了问题，成功编译。就像一个通关游戏，非常有趣.

对于Rust更深入的学习，比如编译器、Cargo、以及嵌入式等，[https://www.rust-lang.org/learn](https://www.rust-lang.org/learn)　这个页面有免费的书籍提供系统完整的讲解。

[Rust社区论坛](https://users.rust-lang.org/)，如果由任何问题可以在这上边提问交流,或者也可以在　[discord群组](https://discord.gg/rust-lang)　里交流



关于Rust的视频频道
- [Rust官方Youtube频道](https://www.youtube.com/channel/UCaYhcUwRBNscFNUKTjgPFiA)

关于Rust的Podcast
- [rustacean station](https://rustacean-station.org/)


















