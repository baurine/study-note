# 《The Rust Programming Language》Note 1

- [eBook Link](https://rustlang-cn.org/office/rust/book/)

其它资源：

- [Rust 三天简易教程](https://www.yuque.com/progfun/rust/tss53d)

重点是要理解 Rust 与其它语言所不同的思想，不要用其它语言的思想来学习 Rust，那样肯定会很困惑。

## 前言

略。

## 介绍

略。

## 1. 开始

### 1.1 安装

安装，通过 rustup 下载安装 rust，先要安装 rustup，安装 rustup 时会自动安装最新版 rust。

```shell
$ curl https://sh.rustup.rs -sSf | sh
```

更新和卸载：

```shell
$ rustup update
$ rustup self uninstall
```

查看 rust 版本：

```shell
$ rustc --version
```

### 1.2 Hello World

```rust
// main.rs
fn main() {
  println!("Hello World");
}
```

编译运行：

```shell
$ rustc main.rs
$ ./main
Hello World
```

### 1.3 Hello Cargo

Cargo 是 Rust 的构建系统和包管理器。就像是 js 的 npm，Ruby 的 gem。Rust 安装包已经带了 cargo。

检查是否安装 cargo: `$ cargo --version`。

用 cargo 创建项目：`$ cargo new hello_cargo`，将生成 Cargo.toml 配置文件及 src 目录，以及生成 git 仓库。

Cargo.toml 中是此项目的描述信息及依赖。

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
```

构建并运行 cargo 项目：`$ cargo build`，最终生成的可执行文件放在 `target/build` 目录中。执行 `./target/build/hello_cargo` 得到结果。

也可以直接执行 `$ cargo run` 进行编译并运行。

另外 cargo 还提供了 `cargo check` 命令，该命令快速检查代码确保其可以编译，但并不产生可执行文件，它比 `cargo build` 快很多。(rust 的编译速度很感人...)

发布 release 版本：`cargo build --release`。

## 2. 编写猜猜看游戏

第一步，接收输入：

```rust
// main.rs
use std::io;

fn main() {
    println!("Hello, world!");

    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("failed");
    println!("guessing: {}", guess);
}
```

`read_line()` 方法的返回值是一个枚举类型 Result，包括 Ok 和 Err 两种结果，用 `expect()` 方法处理 Result。

第二步，添加一个依赖，rand crate，生成随机数。

```toml
[dependencies]
rand = "0.3.14"
```

修改代码：

```rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);
    println!("The secret number is: {}", secret_number);
    // ...
}
```

`use rand::Rng`，首先 `use rand::` 可以让我们使用 rand crate 中的所有内容，`rand::Rng` 中一个 trait (暂不理解，后面会讲)。

所以我们使用 `rand::thread_rng().gen_range(1, 101)` 方法产生一个 1~100 之间的随机数。

第三步，将自己的输入值转换成数值，与随机值进行比较。

```rust
use std::io;
use rand::Rng;
use std::cmp::Ordering;

fn main() {
    //...

    let guess: u32 = guess.trim().parse().expect("failed");
    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small"),
        Ordering::Greater => println!("Too big"),
        Ordering::Equal => println!("You win")
    }
}
```

上例中 guess 一开始绑定的是可变的 String，后者绑定的是不可变的 u32，rust 允许同名变量被多次绑定。

match 用来匹配，可以替代 switch...case / if...else 等。

`=>` 箭头符号的使用。

第四步，加入无限循环，使用 loop。

```rust
fn main() {
  //...
  loop {
    let mut guess = String::new();
    //...
    let guess: u32 = guess.trim().parse().expect("failed");
    match guess.cmp(&secret_number) {
      //...
    }
  }
}
```

第五步，猜正确后退出循环。

```rust
fn main() {
  //...
  match guess.cmp(&secret_number) {
    //...
    Ordering::Equal => {
      println!("You win!");
      break;
    }
  }
}
```

第六步，正确处理解析失败，失败时不是崩溃退出，而是请求下一次输入。注意同样使用了 match 用法。

```rust
fn main() {
  //...
  loop {
    //...
    let guess: u32 = match guess.trim().parse() {
      Ok(num) => num,
      Err(_) => {
        println!("invalid input");
        continue;
      }
    }
    //...
  }
}
```

Result 的用法用 swift 相似。

最终实现：

```rust
fn main() {
    println!("let's start play game!");

    let secret_number = rand::thread_rng().gen_range(1, 101);
    println!("Please input a number:");
    loop {
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("failed");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("invalid input");
                continue;
            }
        };

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("too small"),
            Ordering::Greater => println!("too big"),
            Ordering::Equal => {
                println!("you win!");
                break;
            }
        }
    }
}
```

## 3. Rust 通用编程概念

### 3.1 变量与可变性

在 rust 中，变量默认是不可变的 (似乎和变量这个名称相悖...)，除非显示地加上 mut 进行声明。变量的值在初始时赋值，之后不可以再第二次赋值，和函数式编程一样。

变量和常量的区别：常量不光默认不能变，而且总是不能变，不能加 mut 使之可变；常量用 const 声明，而不是 let。另外，常量必须是编译期就确定的值，不能是运行期确认的值。(和 js 不一样)。

隐藏：rust 允许在同一个函数中声明同名的变量，后面的变量会覆盖前面的变量，可以减少命名的困难。(哈哈)

```rust
fn main() {
  let x = 5;
  x = x + 1; // error!

  let y = 5;
  let y = y + 1; // ok，且生成了新的变量

  let mut z = 5;
  z = z + 1;  // ok，没有生成新的变量

  let spaces = "   ";
  let spaces = spaces.len(); // ok，可以同名且不同类型
}
```

### 3.2 数据类型

两种数据类型子集：标量 (scalar) 和复合 (compound)。

Rust 是静态类型，必须在编译期就知道所有变量的类型。编译器通常可以推断出我们想要的类型，但当类型有多种时，必须增加类型注解，如 guessing game 中将 string 转换成 number 时：

```rust
let guess: u32 = "42".parse().expect("not a number");
```

标量类型，Rust 有四种基本的标量类型：整型、浮点型、布尔型 (bool) 和字符型 ('a')。和其它语言没太大差别，详略。

复合类型：元组 (tuple) 和数组 (array)。

元组，和 python/swift 中概念相似。

```rust
fn main() {
  let tup: (i32, f64, u8) = (500, 6.4, 1);

  let (x, y, z) = tup; // 使用模式匹配进行解构

  let five_hundred = tup.0;
  let six_point_four = tup.1;  // 使用点号和索引访问
}
```

数组，长度固定且类型相同。类型声明很特别，使用 `[type; len]`。

```rust
let a = [1, 2, 3, 4, 5]; // 自动推导类型为 [i32; 5]
let b: [u32; 5] = [1, 2, 3, 4, 5];
```

vector 是长度可变的数组，应该是在标准库里的数据结构，后面会讲吧。

### 3.3 函数

Rust 使用 fn 声明函数，函数名和变量名使用 snake case 规范。

> 因为 Rust 是一门基于表达式（expression-based）的语言，这是一个需要理解的（不同于其他语言）重要区别。... 语句（Statements）是执行一些操作但不返回值的指令。表达式（Expressions）计算并产生一个值。... 语句不返回值，因此不能把整个语句赋值给另一个变量。

```rust
fn main() {
  let x = (let y = 6); // error!
}
```

这和其它语言不一样，c 和 ruby 的赋值语句会返回所赋的值，甚至有些语言还可以这样写：`x = y = 6`，但 Rust 不行。

(重点就是学习 Rust 和其它语言不一样的地方。)

表达式会计算一些值，比如 `5+6`，这是一个表达式并计算得到值 11。表达工可以是语句的一部分，在 `let y = 6;` 中 `6` 是一个表达式，它计算出值 6。函数调用是一个表达式，宏调用是一个表达式。创建新作用域的 `{}` 也是一个表达式 (soga!! 开始说到重点了)。

```rust
fn main() {
  let x = 5;
  let y = {
    let x = 3;
    x + 1  // !! 注意，这里没有分号。如果加上分号，它就是一个语句，语句没有返回值。soga!
  }
  println!("y is: {}", y); // 4
}
```

上面 y 的赋值，整个 `{}` 包围的代码块是一个表达式。

> 表达式的结尾没有分号。如果在表达式的结尾加上分号，它就变成了语句，而语句不会返回值。

具有返回值的函数，使用 `-> type` 声明返回类型。在 Rust 中，函数的返回值等于函数最后一个表达式的值 (和 ruby 相同，和其它大部分语言不同)，但也可以用 return 提前返回。(但是注意，表达式不能加 `;` 哦，不然就变成语句了。)

```rust
fn five() -> i32 {
  5  // 注意没有分号
}

fn plus_one(i: i32) {
  i + 1 // 注意没有分号
}

fn main() {
  let x = five();
  let y = plus_one(8);
  // ...
}
```

### 3.4 注释

略。

### 3.5 控制流

if ... else if ... else，如果 else if 太多了，可以用 match 替代。

在 let 语句中使用 if，和 swift 类似。

```rust
let number = if condition {
  5
} else {
  6
};
```

因为 if 是表达式，所以可以将返回值绑定到 number 上。

循环：loop / while / for，没什么特别。break 有点特别，它可以带回返回值，比如 `break 10` (要不要加分号呢? 待试验)。
