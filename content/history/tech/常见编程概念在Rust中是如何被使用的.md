---
title: 常见编程概念在Rust中是如何被使用的
id: 135
date: 2023-04-10 18:19:31
auther: FREEDOM
cover: https://codeblog.net/upload/2022/11/image-1669624231093.png
excerpt: 许多编程语言的核心有很多共通点，这篇文章覆盖的一些编程概念几乎在每个编程语言里都有，但是它们在各种编程语言里使用方式有差别，我们这里只是讲在Rust语境里，这些概念的一些约定和使用方式。变量和可修改性在 Rust 语言里，变量默认是不可修改的，在一个数字猜测游戏：与Rust的第一次约会 中有提到过。
permalink: /archives/%E5%B8%B8%E8%A7%81%E7%9A%84%E7%BC%96%E7%A8%8B%E6%A6%82%E5%BF%B5
categories:
 - code
tags: 
 - rust
 - 新语言学习
---

许多编程语言的核心有很多共通点，这篇文章覆盖的一些编程概念几乎在每个编程语言里都有，但是它们在各种编程语言里使用方式有差别，我们这里只是讲在Rust语境里，这些概念的一些约定和使用方式。

## 变量和可修改性

在 Rust 语言里，变量默认是不可修改的，在[一个数字猜测游戏：与Rust的第一次约会](https://codeblog.net/archives/yi-ge-shu-zi-cai-ce-you-xi--yu-rust-de-di-yi-ci-yue-hui#toc-head-3) 中有提到过。
如果尝试修改:

```Rust
fn main() {
    let x = 5;
    println!("The value of x is :{}", x);
    x = 6;
    println!("The value of x is :{}", x);
}
```
会发生编译错误:
![image-1665993823122](https://codeblog.net/upload/2022/10/image-1665993823122.png)

虽然在刚开始这会让人感觉很繁琐，但这是Rust的优势所在的原因之一，变量默认是不可修改的，如果以后需要修改，需要在声明变量时明确使用 mut 关键字。这样相当于是对值变化的变量的高亮显示，不需要我们检查每一行代码去确定某个变量哪里被修改了——如果它是使用 mut关键字定义的（一定在某处值被修改了），如果没有（值一定没有改变）。通过这样的方式可以规避很多隐形Bug

使用 mut 关键字定义可修改变量:

```Rust
fn main() {
    let mut x = 5;
    println!("The value of x is :{}", x);
    x = 6;
    println!("The value of x is :{}", x);
}
```
![image-1665994793034](https://codeblog.net/upload/2022/10/image-1665994793034.png)

### 变量和常量之间的区别

说到不可修改的变量，如果有其他编程语言经验的话，我们会自然联想到常量，在Rust中也有常量，这时候我们自然会疑惑——既然我们已经有了不可修改的变量为什么还需要常量呢，它们之间有什么区别:

1. 常量不允许使用mut关键字（可以理解，毕竟常量本来就是不可修改的）
2. 定义常量使用const关键字，而不是let，而且必须明确指定值的类型（而变量只有在[特殊情况](https://codeblog.net/archives/yi-ge-shu-zi-cai-ce-you-xi--yu-rust-de-di-yi-ci-yue-hui#toc-head-10)下才需要）
3. 常量可以在任何范围（scope）内定义,包括全局范围（变量不可以，见下图）

![image-1666001224885](https://codeblog.net/upload/2022/10/image-1666001224885.png)

常量通常用于存储全局范围内使用的固定值

### Shadowing(遮蔽)

Shadowing 这个词我看每个人的翻译不同，但翻译成“遮蔽”是我目前能找到最符合原意的了，但还是感觉很别扭。当一个人的影响力或者名声盖过另一个人时就可以使用这个词，在Rust的语境里的含义是——一个新的同名变量盖过了之前的值（注意：不是替换，原来的值还存在）。
```Rust

fn main() {
   let xiaomei = "小美1";
   
   println!("xiaomei: {}。。。", xiaomei);
   {
      let xiaomei = "小美2";
      println!("xiaomei: {} 。。。", xiaomei);
   }
   
   println!("xiaomei: {} 。。。", xiaomei);
}

```

就像，小美去参加一个聚会，刚开始其他人说到小美指的就是她，但一会儿又来了一个长得更漂亮的小美，其他人都被吸引了，这时候大家说到小美就指的是这个后来者。直到小美2号离开这个聚会，大家再喊小美，小美1号的回复才有附和。这整个过程中虽然她们都叫小美，但是是两个人。
![image-1666067586929](https://codeblog.net/upload/2022/10/image-1666067586929.png)

而可变变量却不同，修改可变变量是对之前的值做修改，不是创建一个新的变量。
```
fn main() {
   let mut xiaomei = "小美1";
   
   println!("xiaomei: {}。。。", xiaomei);
   {
      xiaomei = "小美2";
      println!("xiaomei: {} 。。。", xiaomei);
   }
   
   println!("xiaomei: {} 。。。", xiaomei);
}

```
之前的小美1已经变成了小美2，整个过程中只有 一个变量。

![image-1666067959958](https://codeblog.net/upload/2022/10/image-1666067959958.png)

所以，shadowing和可变变量的区别是：
1. shadowing是用let创建一个新的变量,当新的变量失效的时候，旧变量的值会显现。
2. shadowing用于当我们想要修改一个不可修改变量的值，修改后依然是一个不可修改变量时。
3. 当我们想要创建一个同名但不同类型的变量时。

## 数据类型

Rust 是静态语言，这意味着它必须在编译期间就确定所有变量的类型。通常情况下，编译器可以根据变量值以及我们是如何使用变量的来推断出变量的类型，所以一般不需要明确指定数据类型，除非遇到有歧义的情况，比如[将字符串转换为数值](https://codeblog.net/archives/yi-ge-shu-zi-cai-ce-you-xi--yu-rust-de-di-yi-ci-yue-hui#toc-head-10)。

### 标量类型

标量类型指具有单个值的类型，Rust中主要有四种标量类型：整数、浮点、布尔、字符。

#### 整数类型

整数类型分为有符号整数和无符号整数, Rust中内置了这些整数类型。


| 占用位/比特数 | 有符号 | 无符号 |
| ------------ | ------ | ------ |
| 8位          | i8     | u8     |
| 16位         | i16    | u16    |
| 32位         | i32    | u32    |
| 64位         | i64    | u64    |
| 128位        | i128   | u128   |
| 架构         | isize  | usize  |

isize 和 usize 是根据程序运行机器的架构动态决定的，如果在64位架构size就是64，如果是32位架构就是32

每种类型可以存储的数值范围是: 

- -(2^n-1^)  到 2^n-1^ -1 
- n是该类型占用的字节长度

在Rust中支持以二进制、八进制（0）、十进制、十六进制、字节 表示整数。
除了以字节表示外，其他形式都支持带类型后缀：看代码
```

fn main() {

  let decimal = 9u8;
  let hex = 0xffu64;
  let octal = 076i32;
  let byte = b'B';

  println!("{}, {}, {}, {}", decimal, hex, octal, byte);

  let big_number = 300_000;
  
  println!("{}", big_number);
  
}
```
输出结果:
![image-1666077082985](https://codeblog.net/upload/2022/10/image-1666077082985.png)

对于大数字，可以使用下划线 _ 作为分隔符，以便于阅读。

> 如果你不知道该选择哪种整数类型，正如我们之前提到的：通常不需要显式声明类型，Rust的编译器会自动为我们选择合适的类型

#### 浮点类型

浮点类型分为两种
- f64 （默认）, 64位长度（双精度）
- f32 ，32位长度（单精度）

#### 数值操作

rust支持基本的数学操作：加减乘除以及求余等。
关于完整的Rust支持的操作符号——[参见](https://doc.rust-lang.org/book/appendix-02-operators.html)

#### 布尔类型

和所有其他编程语言一样，布尔类型只有 true 和 false, 占用一个字节长度。

![image-1666081108860](https://codeblog.net/upload/2022/10/image-1666081108860.png)

#### 字符类型

字符类型使用单引号，占用四个字节的长度，代表一个 [Unicode](https://home.unicode.org/) 值，所以它可以表示的远远多余 ASCII码。

```
fn main() {

  let d = 'd';
  let emoji = '😝'; 
  let chinese = '你';

}
```

### 复合类型

复合类型是指多个值组合成一个类型，Rust中主要有两种复合类型: tuple 和数组。

#### tuple 类型

一个tuple可以包含多个不同类型的值

```Rust
fn main() {

  let tup = (4, 3.8, "hello", b'A');

  let (i, f, s, u) = tup;

  println!("{}, {}, {}, {}", i, f, s, u);

  println!("{}, {}, {}, {}", tup.0, tup.1, tup.2, tup.3);

}
```
![image-1666147517229](https://codeblog.net/upload/2022/10/image-1666147517229.png)

可以通过Pattern的方式将tuple中的值绑定到不同的变量，也可以通过索引的方式访问。

#### 数组类型

数组也可以包含多个值，但类型必须是相同的。

```Rust
fn main() {
    // 1. 直接初始化
    let arr: [i32;3] = [1, 2, 3];
    //通过pattern绑定变量
    let [x, y, z] = arr;
    //也可通过索引访问
    println!("{}, {}", x, arr[1]);

    // 1. 元素值相同数组["你好"， "你好", "你好"]的简洁初始化
    let arr2 = ["你好"; 3];

}
```
在Rust中数组和tuple一样——一旦初始化，长度是固定的，无法增大或缩小。它们之间的不同点是
1. tuple可以包含类型不同的元素，数组不可以
2. tuple使用括号(), 数组使用中括号 [],
3. tuple使用点号“.”访问索引，数组使用中括号访问索引
4. tuple没有访问越界的问题, 数组有。

#### 访问越界

对于 tuple 通过索引访问元素是通过点号“．”——无法通过变量传入索引值，而数组可以变量传入索引值——有运行时发生越界的可能性。

```Rust
fn main() {
    //Tuple
    let tup = (4, 3.8, "hello", b'A');

    let index = 5;
    // 1. 语法错误,编译会报错
    let last = tup.index;
    // 2.编译会检查出错误
    let last = tup.5;

    //数组
    let arr = [1, 2, 3];
    // 1.编译会检查出越界
    let last = arr[4];
    let index = 4;
    // 2.编译不会检查出越界，运行时才会报错
    let last = arr[index];
    
}

```

## 函数

### 定义

在Rust中定义函数，使用关键字 fn , 函数参数是一种特殊的变量——只能在函数内部使用，而且必须显式指定类型。

```
fn main() {
    function_example(1, 'A');
}

fn function_example(x:i32 , y: char){

    println!("x: {}, y:{}", x, y);
}


```

而main 函数是最特殊的——它是Rust程序的入口。

### 函数体中的语句和表达式

函数体通常包含两种元素
1. 语句（Statement）, 没有返回值。
2. 表达式（Expression）,会返回一个结果

```
//  函数的定义是一个语句
fn main() {
    // 变量的定义是一个语句，其中 5 是一个表达式,返回 5
    let x = 5;
	// 整个let定义是一个语句
    let y = { // 这个区块是一个表达式，因为最后一个没有分号，所以有返回值。
        x + 1  // 是一个表达式，返回 6.
    };
    
    //函数的调用是一个表达式，即使没有显式返回，默认会返回()
    let result = function_example();
    //和函数的调用一样，macro的调用也是一个表达式，默认返回()
    let macro_result = println!("result: {:?}", result);
    println!("macro_result: {:?}", macro_result);
    
}

// 函数的定义是一个语句
fn function_example(){

}

```
![image-1666165399958](https://codeblog.net/upload/2022/10/image-1666165399958.png)

> 表达式不包含最后的分号 ， 如果在表达式后添加分号，就转变为语句了。

### 函数的返回值

在Rust中函数以三种方式返回结果：

1. 默认（定义中不需要显式声明返回类型）， 如果函数的定义中没有以第2,3种方式返回结果，返回默认结果()
2. 函数体末尾是一个表达式（需要声明返回类型），返回这个表达式的计算结果。
3. 在函数体内部的任何位置使用 return 语句。

```Rust
fn main() {
    println!("1: {:?}, 2: {}, 3: {}", default_return(), expression_return(), return_statement());
}

// 1.默认
fn default_return(){
    
}

// 2.表达式返回
fn expression_return() -> i32 {
    5
}

// 3.return语句返回
fn return_statement() -> i32 {
    return 8;
}
```

![image-1666166979250](https://codeblog.net/upload/2022/10/image-1666166979250.png)

## 注释

和其他编程语言一样，// 在Rust中也是注释符号

## 控制流

和其他编程语言一样，控制代码执行流主要通过：if 表达式、循环。

### if 表达式

```Rust
fn main() {

    let number = 6;

    if number % 4 == 0 {
        println!("number可以被4整除");
    }else if number % 3 == 0 {
        println!("number可以被3整除");
    }else if number % 2 == 0 {
        println!("number可以被2整除");
    }else {
        println!("number无法被4,3或2整除");
    }

}
```

和其他编程语言一样——根据 if 后的条件表达式的结果是 ture 还是 false 来决定后边的代码块是否要被执行。
和其他编程语言不一样的是，if表达式是一个表达式而不是语句，所以也会返回结果——所以可以将这个结果赋值给let语句。

```

fn main() {

    let condition = true;

    let number = if condition {
        5
    } else {
        6
    };

    println!("number的值是:{}", number);

}

```

但是如果这样将结果赋值给一个let变量，那么如果 if 和 else 返回的值类型不同，那么对于Rust 需要再编译时就需要确定每个变量的类型，是有冲突的。
![image](https://codeblog.net/upload/2022/11/image.png)

> 所以，如果将if表达式的结果赋值给let变量，需要确保其各个分支返回的值类型必须是一致的，否则会发生编译错误。

### 循环

Rust 为循环提供了三种方式，loop、while、for

#### loop

```Rust

fn main() {

    let mut i = 1;

    loop {
        i = i + 1;

        if i > 5 {
            break;
        }

        println!("i: {}", i);

        
    }

}
```
loop循环本身没有条件表达式的要求，所以默认会无限循环，除非在执行代码中通过 break 跳出。

#### while

```Rust

fn main() {

    let mut i = 1;

    while i < 5 {

        i = i + 1;

        println!("i: {}", i);

    }

}
```

while 带有一个条件表达式，当条件表达式结果为true时执行代码块。
上边使用 loop 和 while 分别实现了完全相同的功能，只是将 loop 代码块中的条件判断放到了while条件表达式中，对于上边的功能来说很明显 while 更为简洁。

#### for

```Rust
fn main() {

   for number in 1..10 {
        println!("{}", number)
   }

}
```

虽然loop 和 while 也可是实现遍历集合，但写出来的代码会繁琐臃肿很多，所以对于这样的功能使用 for 更合适。








