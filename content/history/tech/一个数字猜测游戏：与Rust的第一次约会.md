---
title: 一个数字猜测游戏：与Rust的第一次约会
id: 134
date: 2023-04-10 18:19:30
auther: FREEDOM
cover: https://codeblog.net/upload/2022/10/image-1665746517858.png
excerpt: 创建项目$ cargo new guessing_game --bin–bin 选项指定创建一个binary项目处理并验证输入use stdio;fn main() {    println!(&quot;Guess the number!&quot;);    println!(&quot;P
permalink: /archives/yi-ge-shu-zi-cai-ce-you-xi--yu-rust-de-di-yi-ci-yue-hui
categories:
 - code
tags: 
 - rust
---

## 引子

这篇文章是我阅读 [The Rust Programming Language](https://doc.rust-lang.org/book/) 的第二章[Programming a Guessing Game](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html) 的读书笔记，主要记录了其中需要注意的关键点，以及一些我自己的理解。这个章节的主要目的是通过一个简单的游戏，对于使用Rust编写程序有一个大体印象，在这个过程中，对于已经有其他编程语言经验的人来说——大部分代码是可以理解的，但是对于一些细节会产生问题和疑惑，在这个章节里会大体稍加解惑，但是对于更深入的细节会在书中接下来的其他章节全面介绍。这种教学或者学习方式在各个领域都有用到——比如很多的书籍的结构、电影和文章以倒序的方式叙事，这种方式一开始就会在我们的大脑里创造一个破碎的全景图，然后把断裂的部分连接起来。

学习 Rust 语言，起因是从去年（2021）接触了区块链后阅读了大量关于相关的书籍，对于区块链的技术可以带给这个世界的可能性和颠覆性让我深深着迷，我最关注的polkadot项目就是使用的Rust语言，为了可以更多的参与这样的项目，学习Rust是很有必要的，而且这门语言会越来越流行。

这个章节要实现的猜数字游戏，最终要实现的效果是: 程序会生成一个处于1-100之间的数字，但是它是对玩家来说是不可见的，让玩家不断盲猜，直到猜到正确的值，程序结束。

这个游戏非常简单，但如果我们是一个初学者，就不要心浮气躁，不要时刻带着一个功利心，不要什么都瞧不起，如果真的对编程感兴趣，就只是欣赏它能够给我们带来的美和乐趣，这就足够了。🕵️‍♂️

既然是写一个游戏，即使简单那也是一个项目，当然最最最首要的是创世了——创建项目

## 创建项目

使用cargo创建项目结构，因为我们这里是创建一个可运行的游戏应用而不是创建工具库之类的crate, 所以传递 --bin

```Shell
$ cargo new guessing_game --bin
```

-  --bin 选项指定创建一个binary项目

我这里使用了[vscode](https://code.visualstudio.com/) 作为编辑器，下面是导入后的项目结构截图。

![image-1665471219157](https://codeblog.net/upload/2022/10/image-1665471219157.png)

和我们上边提到的学习方式一样，我们这里写代码的思路依然使用倒序的方式，通常人们在开发一个产品的时候也是这样，比如开发一个手机 APP 或者网站也是同样的思路，由面到线再到点——先画出网站/APP页面原型图，再构思项目结构，再根据需要不断细化，如果将这个过程反过来的话，难度就会大很多。

网站是供用户浏览使用的，所以网页的结构就是最轮廓的项目的地图。
而我们这里要做的是一个猜数字游戏，那么我们这个项目最轮廓的地图是——用户输入与我们给予反馈的这个交互过程。
所以第一步是勾勒出这个项目的大致轮廓：
1. 提示用户输入
2. 读取用户输入
3. 提示输入结果如何

好了，接下来我们开始实际操作，先画出粗略的轮廓，再逐渐添加细节。

## 处理并验证输入

下面是我们要写游戏的地图轮廓的第一个版本，非常粗躁😂

```Rust

use std::io; //如果程序中使用没有包含在 prelude 中的类型，需要使用use显式导入


fn main() {  // main 函数是每个Rust程序的入口， fn 关键字声明函数, ()用于包含函数的参数，{} 包含函数体

    println!("Guess the number!");
    // 1. 提示用户输入
    println!("Please input your guess."); //println! 是宏 macro, 不是方法
   
   // 2. 读取用户输入
    let mut guess = String::new();
    
    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
        
    // 3. 提示输入结果如何
    println!("You guessed: {}", guess); 
}

```

和其他语言一样，// 在Rust中也是注释符号

> preclude  顾名思义，就是Rust编译器默认为每个程序导入的模块，这个模块里包含了最常使用的一些类型。

在读取用户输入这一步中，需要考虑这些问题
1. 读取用户的输入，以及读取过程中如果出错如何处理
2. 读取到输入后需要存储

### 通过变量存储值

```Rust
 let mut guess = String::new();  // 通过 let 关键字创建变量
```

- String类型包含在 prelude 中，所以不需要使用 use 显式导入。
- 使用 :: 调用 new 函数创建一个String实例，String 是 UTF-8 编码的可修改的文本。
- 这里直接String:: 调用而不是创建一个实例再调用，说明 new 是String的关联函数（Associated Function），类似于Java中的静态方法。

> 和其他语言不同，在Rust中的变量默认是不可修改的（immutable）, 要想使变量可修改，需要在变量名前使用 mut 关键字

```Rust
 io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
```

- 将用户输入内容读取到变量 guess 中，但不是像Java等其他语言直接传入变量，而是传入引用，引用是在变量前使用&符号来标识
- Rust中的引用和变量一样: 默认都是不可修改。这里需要修改字符串内容，可修改引用也是在变量前使用 mut 关键字来标识。

> 可变变量（mutalbe variable）和可变引用（mutable reference）这两个两个概念是比较难理解的，这里有一个[解答](https://www.reddit.com/r/rust/comments/vvowkl/comment/iflpcpv/?utm_source=share&utm_medium=web2x&context=3)比较清晰 : 当 y 是一个引用（它的值是另一个值的地址）
> y: &i32 :你不可以修改任何东西。
> mut y: &i32 :你可将 y指向一个新的内存地址，但不可以修改当前指向地址的内容。
> y: &mut i32 : 你可以修改当前指向地址的内容，但不能指向新的内存地址。
> mut y: &mut i32 : 你可以指向新的内存地址，也可以修改当前指向地址的内容。

虽然这简单的两行代码，却这么有内涵👍，人不可貌相，代码也是。

### 通过Result类型来处理可能的失败

处理完了存储，那么在读取过程中很有可能由于各种原因出现意外情况——比如，用户输入了我们期待以外的值，比如我们这里期待输入数字，而他却输入了非数值，就会出错。

```Rust
.expect("Failed to read line");
```

.read_line() 会返回一个Result类型，这是一个枚举类型，有两个变体 

- Ok , 如果操作成功，返回得值包裹在其中。
- Err，如果操作失败，返回的错误信息包裹在其中

Result 中的 expect 方法根据 Result的变体

- 如果是Ok, 返回其中包裹的值
- 如果是Err,会引起程序崩溃，并显示传入expect的错误信息

### 通过println! 打印值

最后提示用户输入结果如何。

```Rust
println!("You guessed: {}", guess); 
```

- {} 是占位符

到这里，我们已经完成了第一个版本：描述粗糙的轮廓。
接下来做一些进一步的细化，已经有了用户的输入，但是好像没有与之比较的秘密值哎，那岂不是猜了个寂寞😅

## 生成秘密值

因为需要随机生成一个 1-100 之间的随机值，所以需要一个生成随机数的功能。

### 使用 crate 获得更多功能

> Rust 中将代码组件包叫做 crate

```Cargo.toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.3.14"
```

- 在Cargo.toml中添加需要的依赖包，运行cargo build 会自动从 https://crates.io/ 下载，并编译

虽然这个配置文件和功能非常类似于Java中的maven, 但是还是有些许区别，比如maven就没有兄弟文件 .lock 。

#### 通过Cargo.lock 文件确保可重现的编译

*什么叫做可重现的编译?*

就是当你在自己的机器上成功编译的项目，交给别人，他们在自己的机器上和你使用完全一致的依赖，这样他们的编译就是对你的编译的重现——完全一致。

*为什么会不一致,不是已经在Cargo.toml中指定了每个依赖包的版本了吗 ？*

因为我们在Cargo中使用的是 [语义版本号](https://semver.org/lang/zh-CN/)（[SemVer](https://semver.org/)）——它是版本号的一个命名标准，版本号 0.3.14 是 ^0.3.14 的缩写，含义是 ——任何与版本 0.3.14 兼容的版本。所以 Cargo 并非精确下载版本号 0.3.14 ，它会下载最新与0.3.14兼容的版本，如果Cargo只是根据 Cargo.toml下载，每个人有可能会下载不一样的版本。

*Cargo.lock 是如何工作的？*

当第一次编译项目的时候，Cargo会根据我们在Cargo.toml中指定的标准计算出符合的精确版本号，然后写入到Cargo.lock 。当我们再次编译或者其他人使用我们的源代码再次编译的时候，Cargo会直接下载Cargo.lock中指定的版本号，不会再次计算，这样就保证了每个人使用的依赖包版本号是一致的。

如果想要重新计算精确版本号，可以使用 cargo update 命令，它会将新的计算结果写到 Cargo.lock文件。

如果指定的版本号是 0.3.14 , 即使重新计算 ，Cargo 也只会寻找小于 0.4 大于 0.3 的版本号。所以如果想要使用 0.4.x 的版本号，必须手动修改 Cargo.toml 中的版本号，这样再次运行 cargo build 就会寻找大于 0.4的版本号，然后将计算结果写入到 Cargo.lock。

### 生成随机值

现在 rand 包已经下载到本地，就可以愉快地使用其中提供的方法了🤩

```Rust
use std::io; 

use rand::Rng; //导入

fn main() {

    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101); //使用

    println!("The secret number is: {}", secret_number);

    //省略展示

}

```

- 因为用到了gen_range ，它是定义在Rng trait 中的方法，所以需要使用use导入Rng
- rand::thread_rng() 返回的 ThreadRng 实现了这个trait。

关于crate中方法的详细使用，在项目路径下使用命令

```Shell
cargo doc --open
```
这个命令会提取我们项目依赖的crate中的文档注释，生成API文档，并在浏览器中打开供我们浏览查看，多么体贴，简直就是一条龙服务  💯

为了不让玩家猜测个寂寞，将用户的输入值与秘密值做比较，然后将比较结果反馈给玩家——毕竟这才是游戏的乐趣所在：有努力有反馈🤝。

## 猜测值与秘密值做比较

```Rust
use std::{io, cmp::Ordering};

use rand::Rng; //如果程序中使用不包含在prelude中的类型，需要使用use显式导入

fn main() {

    //省略显示
    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");
    
    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small"),
        Ordering::Greater => println!("Too big"),
        Ordering::Equal => println!("You win")
    }

    
}

```

- 由于通过 read_line 读取用户输入，最后总会有一个回车符号	\n,所以需要使用 trim() 去除
- 由于parse()方法可以将字符串解析为很多数值类型，为了不让Rust 迷惑，我们需要在 guess 后面明确指定要转换为的类型是 u32
- 由于parse有可能在将字符串转换为数值时出现错误，比如用户输入了非数字字符，所以这个方法的返回类型也是 Result,在这里通过expect简单处理
- 任何可比较的类型都可以调用 .cmp 方法做比较，它会返回 Ordering 枚举类型

## 通过循环允许多次猜测

到目前为止，用户只猜测一次，程序就结束了，无论他猜测对错与否，要想再次玩这个游戏就得重新运行程序 🤯 , 这用户体验没得说得——差，我们想要的是——用户可以不停的猜测，直到猜对然后游戏才可以结束。

通过 loop 这个无限循环来实现:

```Rust
use std::{io, cmp::Ordering};

use rand::Rng; 

fn main() {

    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop { //无限循环
        println!("Please input your guess.");
    
        let mut guess = String::new();
    
        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");
    
        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");
        
        println!("You guessed: {}", guess);
    
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small"),
            Ordering::Greater => println!("Too big"),
            Ordering::Equal => {
                println!("You win");
                break;  //退出循环
            } 
        }
    }
    
}

```

- 当测试数字与秘密值匹配，标明猜测对了，退出循环，loop循环后面没有代码可执行，程序结束

so far so good, 现在试玩体验已经很好了，但是美中不足的是，继续看 🤔

## 处理无效输入

目前当用户输入的内容包含非数字内容时，在执行到下面这行代码时，会发生错误导致程序崩溃

```Rust
let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");
```

这实在是太弱鸡了，一点挫折都经历不了，在这种情况下我们需要它不慌不忙，给用户一个友好提示，继续走

```Rust
let guess: u32 = match guess.trim().parse(){
            Ok(num) => num,
            Err(_) => {
                println!("Please input a number");
                continue;
            }
        };
```

我们用 match 表达式对 parse() 的解析结果做了一个匹配， 对解析成功和失败做了分别处理:

- 解析成功，直接返回解析后的数值
- 解析失败，先输出提示信息，然后使用 continue 跳过后面的代码，进入下一个循环。你的猜测没有资格进行比较 🤣 

>   _ 下划线的含义是——捕获任何值，但不会使用它们。（不管你们是些什么货色，知道是你们就行了，关于你们的具体细节，爷不care）

## 完整代码

perfect, 我们的程序首先生成一个介于 [1-100] 的秘密值，然后提示用户输入猜测值，如果输入无效值会友好提醒输入有效数字，直到用户输入有效的数字后，将输入值与秘密值进行比较，对于太大和太小进行了提醒以便用户下次的输入更靠近正确答案然后让用户再次输入，如果猜测正确，提示正确后，跳出循环结束游戏。 

虽然是一个非常简单的游戏，但已经蛮优雅的了，难道不是吗😝 。下面是完整代码

```Rust
use std::{cmp::Ordering, io};

use rand::Rng; //如果程序中使用不包含在prelude中的类型，需要使用use显式导入

fn main() {
    println!("Guess the number!");

     //生成秘密值
    let secret_number = rand::thread_rng().gen_range(1, 101);
    //猜测到正确数字之前，不退出程序，持续循环
    loop {
        println!("Please input your guess.");
        //创建保存用户输入的变量 
        let mut guess = String::new();
        //读取用户输入到变量guess
        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");
        //去除前后空格，并验证
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("Please input a number"); //友好提示
                continue; //进入下次循环，重新输入
            }
        };

        println!("You guessed: {}", guess);
        //比较
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small"),
            Ordering::Greater => println!("Too big"),
            Ordering::Equal => {
                println!("You win"); //猜测正确
                break; //跳出循环，结束游戏
            }
        }
    }
}
```

```Toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.3.14"
```

结束---------------------------------










