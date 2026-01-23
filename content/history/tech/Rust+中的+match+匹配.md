---
title: Rust 中的 match 匹配
id: 167
date: 2023-04-10 18:20:06
auther: FREEDOM
cover: https://codeblog.net/upload/2023/04/0%20xrnstik-kxaOAhwS.png
excerpt: 
permalink: /archives/rust%E4%B8%AD%E7%9A%84match%E5%8C%B9%E9%85%8D
categories:
 - code
tags: 
 - 控制流
 - rust
---


除了`if...else...`控制流指令，大多数编程语言还会提供一个叫做[switch...case...](https://en.wikipedia.org/wiki/Switch_statement)的指令可以根据匹配条件控制代码段的执行流向。

在Rust中也提供了一个类似的指令叫做 `match`，不过它可以匹配的范围非常广——字面量，变量名，正则表达式...等匹配模式都支持。

## match匹配

下面是之前博客 [Rust 中的Enum和空指针问题](https://codeblog.net/archives/rust%E4%B8%AD%E7%9A%84enum%E5%92%8C%E7%A9%BA%E6%8C%87%E9%92%88%E9%97%AE%E9%A2%98)中使用的代码片段

```rust
fn main() {
    
    let man = User::Man(170);
    let woman = User::Woman{height: 160, cup: 'G'};
    let gay = User::Gay;

    man.info();
    woman.info();
    gay.info();

}

enum User {
    Man(u32),
    Woman{height: u32, cup: char},
    Gay
}

impl User {
    
    fn info(&self){
       // 使用 match 将self与下面的各模式pattern做匹配
       match self {
        // pattern 中的变量会绑定 self 中的值
        Self::Man(height) => println!("男性，身高是:{}", height), //　pattern分支以逗号,分割
        Self::Woman { height, cup } => println!("女性,身高是：{},CUP是：{}", height, cup), 
        Self::Gay => println!("该用户是一个同性恋")

       } 
        
    }

}
```
![image-1680595912901](/upload/2023/04/image-1680595912901.png)

当你查看 `match` 的源码时，它的注释中有这样一句:

> match can be used to run code conditionally. **Every pattern must be handled exhaustively either explicitly or by using wildcards like _ in the match**. Since match is an expression, values can also be returned.

### match的分支覆盖所有情况

所有情况是什么意思 ?

上边的代码中 `self` 是`User`枚举类型，有三种可能性: `Man`,`Woman`, `Gay`.

如果我们将最后一个分支去掉会如何:

```rust
fn info(&self){
       // 使用 match 将self与下面的各模式pattern做匹配
       match self {
        // pattern 中的变量会绑定 self 中的值
        Self::Man(height) => println!("男性，身高是:{}", height), //　pattern分支以逗号,分割
        Self::Woman { height, cup } => println!("女性,身高是：{},CUP是：{}", height, cup)

       } 
        
    }
```
![image](/upload/2023/04/image.png)

编译错误信息中明确告诉我们，我们的`match`分支没有覆盖到当 `self`是`Gay`时的情况。
所以Rust在编译代码的时候会检测`match`指令是否覆盖到了所有可能性，它会强制要求覆盖到所有情况——因为`match`是一个表达式，它会返回值，必须有返回值，即使我们的代码没有明确返回，它会默认返回一个空tuple

![image-1680596201142](https://codeblog.net/upload/2023/04/image-1680596201142.png)

在之前讲到[Rust如何解决空指针问题](https://codeblog.net/archives/rust%E4%B8%AD%E7%9A%84enum%E5%92%8C%E7%A9%BA%E6%8C%87%E9%92%88%E9%97%AE%E9%A2%98#toc-head-6)时说过，当取出`Option<T>`中包裹的值时，Rust会强制要求处理覆盖到空值和有效值的情况，就是借助`match`的这点做到的。

### 每种情况不需要单独处理时

既然Rust强制要求覆盖所有情况，那么这意味着不得不为每种情况都单独写一个分支吗？

不是的，类似于`switch...case...`中的 `default`分支，rust中也可以通过`_`或`other`来实现相似的效果。

比如，我们希望对在我们的相亲网站中注册的用户做一个筛选，对于是同性恋的用户提醒不为其提供服务，而对于正常的男性和女性处理方式一样，可以进入到下一个流程：

```rust
impl User {
    
    fn info(&self){
       // 使用 match 将self与下面的各模式pattern
       match self {
        
            Self::Gay => println!("对不起，我们无法为您提供服务，谢谢"), // 对同性恋用户作出提醒
            other => next_stage(other) // 其他用户进入下一步
       } 
        
    }

}
```
如果对于正常男性和女性不需要进入下一步，也是只做一个他们是我们可以服务的用户提醒，那么就没有必要使用 `other` 关键字来绑定`self`, 直接使用下划线`_`就可以了:

```rust
impl User {
    
    fn info(&self){
       // 使用 match 将self与下面的各模式pattern
       match self {
        
            Self::Gay => println!("对不起，我们无法为您提供服务，谢谢"), // 对同性恋用户作出提醒
            _ => println!("恭喜，我们可以为您提供服务") 
       } 
        
    }

}
```
如果对于正常男性和女性，提醒都不需要，直接返回一个空`tuple`就可以了，这也是`match`的默认返回值

```rust
impl User {
    
    fn info(&self){
       // 使用 match 将self与下面的各模式pattern
       match self {
        
            Self::Gay => println!("对不起，我们无法为您提供服务，谢谢"), // 对同性恋用户作出提醒
            _ => ()
       } 
        
    }

}
```

## 使用 if let 精确控制流

上边最后一段代码，我们除了通知同性恋用户信息外，最后通过`_`处理其他情况完全是为了满足`match`要求覆盖所有情况的需要，并没有实际意义。

像这种情况就可以使用`if let`实现相同的功能，但是代码更加简洁:

```rust
impl User {
    
    fn info(&self){
       
       if let User::Gay = self {
         println!("对不起，我们无法为您提供服务，谢谢");
       }
        
    }
}
```

这样就没有必要处理其他情况，因为`if let`没有覆盖所有情况的强制要求，当然也就失去了它的保护。

甚至可以这样使用:

```rust
impl User {
    
    fn info(&self){
       
       if let User::Gay = self {
         println!("对不起，我们无法为您提供服务，谢谢");
       }else if let User::Man(height) = self {
           println!("男性，身高是:{}", height);
       }else if let User::Woman{height, cup} = self {
        println!("女性,身高是：{},cup:{}", height, cup);
       }
        
    }

}
```
或者

```rust
impl User {
    
    fn info(&self){
       
       if let User::Gay = self {
         println!("对不起，我们无法为您提供服务，谢谢");
       }else if let User::Man(height) = self {
           println!("男性，身高是:{}", height);
       }else{
        println!("女性");
       }
        
    }

}
```

所以 `if let` `else if let` `else` 三者组合可以实现 `match`相同的功能，但是不会有`强制覆盖所有情况`的保护。




