---
title: Rust中使用struct来组合构建相关联的数据
id: 165
date: 2023-04-10 18:20:06
auther: FREEDOM
cover: https://codeblog.net/upload/2023/03/structure.jpg
excerpt: 在面向对象的编程语言中都会提供一种方式让我们创建自定义的类型，以便将一个抽象对象，其相关联的属性和方法组合起来，或者构建到一起，在Java中叫做class ，而在Rust中是叫做 struct 。编写一个程序或者软件应用就像是盖一个房子或者是摩天办公大楼，它们的基础组成元素都是钢筋，水泥和沙子构成的
permalink: /archives/rust%E4%B8%AD%E4%BD%BF%E7%94%A8struct%E6%9D%A5%E6%9E%84%E5%BB%BA%E7%9B%B8%E5%85%B3%E8%81%94%E7%9A%84%E6%95%B0%E6%8D%AE
categories:
 - code
tags: 
 - rust
---

在面向对象的编程语言中都会提供一种方式让我们创建自定义的类型，以便将一个抽象对象，其相关联的属性和方法组合起来，或者构建到一起，在Java中叫做`class` ，而在Rust中是叫做 `struct` 。

编写一个程序或者软件应用就像是盖一个房子或者是摩天办公大楼，它们的基础组成元素都是钢筋，水泥和沙子构成的，那么它们是直接混合到一起就盖成了一座座房子吗？那也效率太低太原始了，如果不知道[看看这个视频](https://youtu.be/mbwuj58UEPg) : 首先根据设计图纸，然后用钢筋、水泥、沙子浇注出各式各样的混泥土构建块，然后将每个构建块就像搭积木一样拼凑到一块，最终形成想要的房子。

编写一个程序也是如此，就像钢筋水泥沙子，每种面向对象高级编程语言都默认提供了基础的数据类型——整型，浮点型，布尔型，字符型等，但是就像建筑高楼一样，我们需要使用这些最基本的数据类型来根据每个程序的需求创建出更大的构建块来更高效的完成工作，而 `struct` 就是这样的构建块。

接下来的内容是:

1. 定义和实例化 `struct` 
2. 打印显示自定义`struct`
3. 为`struct`添加关联函数（associated function）和方法（method）

## 定义和实例化 `struct`

要想创建出一个个的构建块，首先我们需要创建构建块的模板（定义），然后根据模板来生成（实例化）构建块。

### 定义并实例化

```rust
fn main() {

    // 创建一个 User 的实例，赋值给变量 user1.
    let user1 = User {
        active: true,
        username: String::from("jack"),
        email: String::from("1234@gmail.com"),
        password: String::from("123456")
    };

  
}

// 定义一个 struct,名为 User
struct User { // struct 包含用户的一些基本属性，这些属性叫做字段（Field）
    active: bool,
    username: String,
    email: String,
    password: String
}
```

需要注意的是： struct 中的字段是使用逗号“,”分割，而不是分号“;”

### 读取和修改实例的字段值

实例的字段值可以被修改和读取

```rust
fn main() {

    // 创建一个 User 的实例，赋值给变量 user1.
    let mut user1 = User {
        active: true,
        username: String::from("jack"),
        email: String::from("1234@gmail.com"),
        password: String::from("123456")
    };
    // 读取字段的值
    println!("用户 user1 的邮箱地址是：{}", user1.email);

    // 修改字段值 -> 实例必须是可修改变量
    user1.email = String::from("changed@gmail.com");

    println!("用户 user1 的邮箱地址是：{}", user1.email);

}

// 定义一个 struct,名为 User
struct User { // struct 包含用户的一些基本属性，这些属性叫做字段（Field）
    active: bool,
    username: String,
    email: String,
    password: String
}
```

需要注意的是:

1. 访问和修改 `struct` 实例的字段值时使用点“.”记号
2. 要想修改字段值，其所在实例必须是可修改变量

### 字段初始化时缩写

上边当我们创建实例 user1 的时候，初始化其字段时是为每个值传入的固定值

```rust
let user1 = User {
        active: true,
        username: String::from("jack"),
        email: String::from("1234@gmail.com"),
        password: String::from("123456")
};
```

这样的初始化方式在实际场景中是非常少见的，因为用户的字段值通常是存储在数据库中的，不可能硬编码在代码中。

实际场景中是通过传入变量的方式来初始化字段值：

```rust
fn build_user(username: String, email: String, password: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        password: password
    }
}
```

是的，变量名和字段名可以相同。

但是为了避免重复书写，当在变量名和字段名相同的情况下，Rust 允许像这样省略的写法：

```rust
fn build_user(username: String, email: String, password: String) -> User {
    User {
        active: true,
        username,
        email,
        password
    }
}
```

### 共享其他实例的字段值

当我们在创建一个新实例时，如果某些字段的值和其他实例的值相同时，通常的做法是:

```rust
   let mut user1 = User {
        active: true,
        username: String::from("jack"),
        email: String::from("1234@gmail.com"),
        password: String::from("123456")
    };

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("user2@gmail.com"),
        password: user1.password
    };
```

可以看到，新创建的 user2 和 user1 只有字段 email 值不同，所以将其他字段都一一从 user1中获取。
而 rust 提供了更简洁的方式: 

```rust
  let user2 = User {
        email: String::from("user2@gmail.com"),
        ..user1
  };
```
只需要指定值不同的字段，其他字段会自动从 user1 中获取。

同样，这样的赋值操作，也遵循 [Copy trait类型会复制，其他类型会Move的原则](https://codeblog.net/archives/%E7%90%86%E8%A7%A3%E6%9D%83%E9%99%90#toc-head-6):

#### Copy trait类型

```rust
fn main() {

    // 创建一个 User 的实例，赋值给变量 user1.
    let mut user1 = User {
        active: true,
        username: String::from("jack"),
        email: String::from("1234@gmail.com"),
        password: String::from("123456")
    };

    let user2 = User {
        email: String::from("user2@gmail.com"),
        ..user1
    };

    // 读取字段的值
    println!("用户 user1 状态是：{}", user1.active);


}

// 定义一个 struct,名为 User
#[derive(Debug)]
struct User { // struct 包含用户的一些基本属性，这些属性叫做字段（Field）
    active: bool,
    username: String,
    email: String,
    password: String
}

```
![image-1680082103999](https://codeblog.net/upload/2023/03/image-1680082103999.png)

对于实现了Copy trait 的数据类型，发生了复制，所以没有所有权的转移，旧变量依然可以使用其相应字段

#### 非Copy trait类型

```rust
fn main() {

    // 创建一个 User 的实例，赋值给变量 user1.
    let mut user1 = User {
        active: true,
        username: String::from("jack"),
        email: String::from("1234@gmail.com"),
        password: String::from("123456")
    };

    let user2 = User {
        email: String::from("user2@gmail.com"),
        ..user1
    };

    // 读取字段的值
    println!("用户 user1 的是用户名：{}", user1.username);

    println!("用户 user: {:?}", user1);

}

// 定义一个 struct,名为 User
#[derive(Debug)]
struct User { // struct 包含用户的一些基本属性，这些属性叫做字段（Field）
    active: bool,
    username: String,
    email: String,
    password: String
}

```
![image-1680081380211](https://codeblog.net/upload/2023/03/image-1680081380211.png)

从错误信息中可以看到，没有实现了 Copy trait 的数据类型的字段从其他变量中获取值时，会发生所有权的Move —— 该字段的数据内容的所有权会被从旧变量的字段转移到新变量中的相应字段 —— 所以在转移后，无法再通过旧变量使用该字段 —— 由于该字段是旧变量的一部分，所以旧变量也无法作为一个整体再被使用，因为其部分数据的所有权已经不属于它；但是没有被转移所有权的字段依然可以被单独使用。

### 没有字段名的Tuple Struct

Rust 除了支持像上边那样使用 `struct` 将一组相关的变量组合为自定义的 `struct`类型外，还可以将字段名去掉 —— 只是一组值，类似于 tuple 的形式：

```rust
fn main() {

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);

    println!("{}", black.0);

}

struct Color(u32, u32, u32);
struct Point(u32, u32, u32);
```
![image-1680089948173](https://codeblog.net/upload/2023/03/image-1680089948173.png)

除了拥有一个 `struct` 类型名外，其他几乎和 `tuple `一样，而且也可以通过索引的方式访问单个值，但是它并不是 tuple 类型，而是自定义struct类型，所以虽然 `Color`和`Point`都是拥有三个 `u32`值，但是他俩是完全不同的数据类型。

### 没有值的 Unit-Like Struct

在 Tuple struct 再进一步把值也去掉，就是 Unit-Like struct. 

```struct
fn main() {

    let subject = AlwaysEqual;   
    subject.show(); 
}

struct AlwaysEqual;

impl AlwaysEqual {
    
    fn show(&self){
        println!("hello wolrd");
    }
}
```

这个 Unit-Like 类似于 Java 语言中的空类——虽然不存储任何数据，但是它可以实现接口，以及拥有方法，Unit-Like也一样可以实现 `Trait`，以及拥有关联函数。

## 打印显示自定义`struct`

直到现在，我们遇到的所有数据类型都是直接通过 `println` 打印显示的，但是对于自定义`struct`是否行的通呢:

```struct
fn main() {

    let user1 = User{
        username: String::from("Jack"),
        email: String::from("1234@gmail.com")
    };

    println!("user1: {}", user1);

}

struct User {
    username: String,
    email: String
}
```
![image-1680093431147](https://codeblog.net/upload/2023/03/image-1680093431147.png)

编译错误信息：我们的 `User` struct 没有实现 trait `Display`
在搞清楚这条错误信息的背后含义，首先我们需要了解`println`这个打印语句做了什么事情:

```rust
println!("user1: {}", user1);
```

1. println 支持多种打印格式,  默认格式是 {}, 还有其他格式，例如 {:?}、{:#?} 等
2. 不同的打印格式对应不同的 `trait`, 格式 {} 对应 `Display`
3. 当 println 打印时会调用实现的trait 的方法。

之所以我们之前的打印都没有问题，就是因为之前打印的都是基本数据类型 —— 它们都实现了 `Display` trait.

### 实现 Display

所以，我们可以为 User 实现 `Display` trait, 来使用默认打印格式 `{}`

```rust
use std::fmt::Display;

fn main() {

    let user1 = User{
        username: String::from("Jack"),
        email: String::from("1234@gmail.com")
    };

    println!("user1: {}", user1);

}

struct User {
    username: String,
    email: String
}

impl Display for User {
    
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "{{username: {}, email: {}}}", self.username, self.email)
    }
}

```
![image-1680095446262](https://codeblog.net/upload/2023/03/image-1680095446262.png)

`Display` trait 要求我们自定类型必须要实现的函数 `fmt`中，我们可以按照自己的需要设计打印的格式。如果想要使用其他答应格式，例如 `{:?}` Debug 格式 ——>

### 使用Debug 打印格式

```rust
fn main() {

    let user1 = User{
        username: String::from("Jack"),
        email: String::from("1234@gmail.com")
    };

    println!("user1: {:?}", user1);

}

#[derive(Debug)]
struct User {
    username: String,
    email: String
}

```

![image-1680095961964](https://codeblog.net/upload/2023/03/image-1680095961964.png)

使用 `{:?}`debug打印模式，如果没有特殊的打印需求，可以直接使用 Rust 提供的 `#[derive(Debug)] `属性来为我们的自定义`struct`添加 Debug trait，而没有必要自己去手动实现 `Debug` trait.

### 使用宏dbg!

除了在 println 中使用 `{:?}` 来使用debug打印格式外，也可以使用宏 dbg! 来实现更细致的 debug 信息打印:

```rust
fn main() {

    let increase = 2;

    let user1 = User{
        username: String::from("Jack"),
        email: String::from("1234@gmail.com"),
        salary: dbg!(1000 * increase)
    };

    dbg!(&user1);
}

#[derive(Debug)]
struct User {
    username: String,
    email: String,
    salary: u32
}

```

![image-1680096784584](https://codeblog.net/upload/2023/03/image-1680096784584.png)

宏dbg!会获取输入值的所有权，然后把计算结果的所有权返回 —— 就像它不存在一样。
在打印信息中会显示，具体的位置以及详细信息。

## 关联函数和方法

上边的代码中我们定义的三种`struct` ，有的既有字段名又有值，有的仅有值，有的连字段值都没有。但是他们都可以拥有属于自己的方法:

```rust
fn main() {

    let user1 = User{
        username: String::from("Jack"),
        email: String::from("1234@gmail.com"),
        salary: 1000
    };

    let black = Color(0, 0, 0);
    let always_equal = AlwaysEqual;

    user1.demo_show();
    black.demo_show();
    always_equal.demo_show();

}

#[derive(Debug)]
struct User {
    username: String,
    email: String,
    salary: u32
}

// tuple struct
struct Color(u32, u32, u32);
// Unit-Like struct
struct AlwaysEqual;

//　在 impl 区块中定义方法
impl User {

    fn demo_show(&self){
        println!("username: {}, email: {}, salary: {}", self.username, self.email, self.salary)
    }
    
}

impl Color {
    
    fn demo_show(&self){
        println!("({},{},{})", self.0, self.1, self.2);
    }

}

impl AlwaysEqual {
    
    fn demo_show(&self){
        println!("do nothing");
    }
}
```
![image-1680139696336](https://codeblog.net/upload/2023/03/image-1680139696336.png)

到这里，会自然的产生一个疑问——> 这里的方法(Method) 和之前介绍的函数(Function)有什么区别 ? :

++方法++

1. 方法是包含在`struct`中的
2. 方法的第一个参数是 `&self` ——>是 `self: &Self`的缩写，Self 是所在自定义类的别名
3. 方法的调用者总是所在`struct`的实例（instance）
4. 实例使用点记号"."来调用

++函数++

1. 函数不局限在某个`struct`下，也不归属于任何实例
2. 可以被公共调用

> 方法（Method）是`struct`是用于操作自身实例数据的行为，函数（Function）是用于操作不局限于任何`struct`的工具行为

### 调用方法的自适应

直接看代码——>

1. 方法签名是 `(&self)`
    ```
    fn main() {
       //...省略
        user1.demo_show();  // = (&user1).demo_show() = User::demo_show(&user1)
    }

    //...省略

    //　在 impl 区块中定义方法
    impl User {

        fn demo_show(&self){
            println!("username: {}, email: {}, salary: {}", self.username, self.email, self.salary)
        }

    }
    ```
2. 方法签名是 `(self)`
    ```
    fn main() {
       //...省略
        user1.demo_show();  // = user1.demo_show() = User::demo_show(user1)
    }

    //...省略

    //　在 impl 区块中定义方法
    impl User {

        fn demo_show(self){
            println!("username: {}, email: {}, salary: {}", self.username, self.email, self.salary)
        }

    }
    ```
3. 方法签名是 `(&mut self)`
    ```
    fn main() {
       //...省略
        user1.demo_show(); // = (&mut user1).demo_show() = User::demo_show(&mut user1)
    }

    //...省略

    //　在 impl 区块中定义方法
    impl User {

        fn demo_show(&mut self){
            println!("username: {}, email: {}, salary: {}", self.username, self.email, self.salary)
        }

    }
    ```
    ![image-1680142208504](https://codeblog.net/upload/2023/03/image-1680142208504.png)
    
上面分别有三个不同签名的 `demo_show`方法, 但是我们的调用语句是一模一样的：`user1.demo_show();` 
而且没有任何问题！！！ why ?

正如上面各自 `uesr1.demo_show()` 后面的注释解释的那样

> 当调用方法时，方法签名中如果是`self` ,rust会让`uer1.demo_show()`保持原样; 如果是`&self`,rust会自动引用,实际调用为`(&user1).demo_show()`;如果是`&mut self`,rust会自动以可修改引用调用`(&mut user1).demo_show()`

所以当调用方法时，Rust会根据方法签名自动为调用者添加 `&`或`&mut`以适应或匹配签名的要求 ——注意，这只对于用方法有效，因为: 方法中的调用者总是`self` ，而函数并非如此。

### 方法包含多个参数

上边的方法都只有一个 `self` 参数，如果拥有其他参数

```rust
fn main() {

    let user1 = User{
        username: String::from("Jack"),
        email: String::from("1234@gmail.com"),
        salary: 1000
    };

    let uesr2 = User{
        username: String::from("Bob"),
        email: String::from("test@gmail.com"),
        ..user1
    };
    
    println!("user1和user2的名字是否相同：{}", user1.same_username(&uesr2));
}

#[derive(Debug)]
struct User {
    username: String,
    email: String,
    salary: u32
}

//　在 impl 区块中定义方法
impl User {

    fn demo_show(&self){
        println!("username: {}, email: {}, salary: {}", self.username, self.email, self.salary)
    }

    // 多个参数，在self后以逗号分割即可
    fn same_username(&self, other: &User) -> bool {
        self.username == other.username
    }
    
}
```
和函数一样，以逗号分割开来即可。

那么，在 `struct`中的方法是否可以没有 `self`参数呢?

### 关联函数

函数我们已经非常熟悉了，但是关联函数是个什么鸟，还是让人有点摸不着头脑。
重点在于关联二字，它和谁关联呢? 和 `struct`关联，定义在 `struct`中的函数就叫做关联函数
那它和方法（Method）的区别是啥呢？上边我们已经说了方法和函数的重要区别就是第一个参数是`self`,所以关联函数没有`self`参数。

```rust
fn main() {

    let user1 = User::default_user();
    
}

#[derive(Debug)]
struct User {
    username: String,
    email: String,
    salary: u32
}

impl User {

    // 关联函数
    fn default_user() -> Self{
        Self { 
            username: String::from("用户"),
            email: String::from("example@gmail.com"),
            salary: 0 
        }
    }
    
}
```
关联函数和方法的区别是

1. 方法通常用于操作实例，所以调用者是实例，第一个参数是 `self`, 通常使用点记号"."调用
2. 而关联函数的目的是为`struct`提供些共用工具函数，所以类似于其他语言中的`静态方法（Static method）`，使用"::调用"

> 函数（Function）用于全局范围的工具组件，关联函数(Associated Function)是`struct`内部针对于自身的工具组件, 方法（Method）是`struct`内部针对于`其实例`的工具组件

### 多个 Impl 区块

> 每个`struct`可以拥有多个`impl`区块

```rust
impl User {

    // 关联函数
    fn default_user() -> Self{
        Self { 
            username: String::from("用户"),
            email: String::from("example@gmail.com"),
            salary: 0 
        }
    }
    
}

impl User {

    fn show(&self){
        dbg!(self);
    }

}
```




