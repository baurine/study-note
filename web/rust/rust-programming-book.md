# 《The Rust Programming Language》Note

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

## 4. 理解所有权

> 所有权（系统）是 Rust 最独特的功能，其令 Rust 无需垃圾回收（garbage collector）即可保障内存安全。

相关功能：

- 借用
- slice
- Rust 如何在内存中布局数据

### 4.1 什么是所有权

> 跟踪哪部分代码正在使用堆上的哪些数据，最大限度的减少堆上的重复数据的数量，以及清理堆上不再使用的数据确保不会耗尽空间，这些问题正是所有权系统要处理的。一旦理解了所有权，你就不需要经常考虑栈和堆了，不过明白了所有权的存在就是为了管理堆数据，能够帮助解释为什么所有权要以这种方式工作。

**所有权规则**

1. Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。
1. 值有且只有一个所有者。
1. 当所有者（变量）离开作用域，这个值将被丢弃。

变量作用域：略。

**String 类型** (因为它是在堆上使用的)

```rust
let mut s = String::from("hello");
s.push_str(" world!");
println!("{}", s);
```

上例中 "hello" 是字符串字面值，编译时就已经被编码直接写进代码段了，因此它是不可变的。

如何回收 s 变量，因为它是在堆上分配的。c/c++ 需要显示 free 掉，其它语言有 GC。Rust 采取了不同的策略：内存在拥有它的变量离开作用域后就被自动释放。

```rust
{
  let s = String::from("hello");  // 从此处起，s 有效
  // 使用 s
} // 作用域结束，s 不再有效，内存空间自动被释放，不会有内存泄漏，实际是调用了 drop 函数，类似 c++ 中的析构函数
```

(嗯，其实 c++ 通过包装指针也能做到，只不过因为它的所有权不是唯一，难免这个地方析构了，其实地方可能还在使用，然后程序就崩溃了。而且也会有后面说的二次释放的问题。)

(哦哦，看到下文说 c++ 的这种实现叫 RAII)

**变量与数据交互的方式一 - 移动**

```rust
let x = 5;
let y = x;
```

因为 x 和 y 都是在栈上，y 拷贝了 x 的值，栈上有两个 5。

再来看堆：

```rust
{
  let s1 = String::from("hello");
  let s2 = s1;
}
```

当把 s1 赋值给 s2 时，并没有将堆上的数据也拷贝一份，而只是在栈上将指针 s1 的值进行了浅拷贝给 s2。因为 s1 和 s2 同时指向同一份堆空间。当它们都离开作用域时，问题来了，它们会尝试释放相同的内存，这是一个叫做**二次释放 (double free)**的错误。

所以，Rust 是如何解决的呢，当把 s1 的值赋值给 s2 后，它认为 s1 引用不再有有效，因此离开作用域是不需要释放空间，而且在赋值给 s2 后，s1 也不再可以访问。这个操作被称为**移动 (move)**。

```rust
{
  let s1 = String::from("hello");
  let s2 = s1;

  println!("s1 is {}", s1); // error!
}
```

**变量与数据交互的方式一 - 克隆**

如果我们确实需要深度复制 String 中堆上的数据，而不仅仅是栈上的引用，可以使用一个叫 clone 的函数。

```rust
{
  let s1 = String::from("hello");
  let s2 = s1.clone();

  println!("s1 is {}, s2 is {}", s1, s2);
}
```

栈上的数据，比如上面 `let x=5; let y=x;` 不存在移动和无效，因为它们的数据都是深度克隆。

> Rust 有一个叫做 Copy trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上（第十章详细讲解 trait）。如果一个类型拥有 Copy trait，一个旧的变量在将其赋值给其他变量后仍然可用。

**所有权与函数**

> 将值传递给函数在语义上与给变量赋值相似。向函数传递值可能会移动或者复制，就像赋值语句一样。

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

完全理解了。

**返回值与作用域**

返回值也可以转移所有权。理解。

示例略。

### 4.2 引用与借用

所有权的移动导致的问题是，写代码不方便了 (^+^)...

解决办法，使用引用，它允许你使用值但不获取其所有权 (所以当引用离开作用域时并不会执行 drop 函数去释放堆上空间，对吧...看了后文，是的)

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权，
  // 所以什么也不会发生
```

我们将获取引用作为函数参数称为**借用（borrowing）**。

正如变量默认是不可变的，引用也一样。（默认）不允许修改引用的值。

**可变引用**

使用 `&mut String` 声明可变引用，但有一个很大的限制，特定作用域中的特定数据有且只能有一个可变引用，用来避免数据竞争。

```rust
fn main() {
  let mut s = String::from("hello");
  change(&mut s);
}

fn change(s_s: &mut String) {
  s_s.push_str("world");
}
```

另外，在相同作用域内，不能同时拥有不可变引用和可变引用... (Rust 限制颇多，要习惯。)

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3);
```

**悬垂引用**

即所谓的野指针。

> 在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个悬垂指针（dangling pointer），所谓悬垂指针是其指向的内存可能已经被分配给其它持有者。相比之下，在 Rust 中编译器确保引用永远也不会变成悬垂状态：当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

(nice!)

引用的总结：

- 在任意给定时间，要么只能有一个可变引用，要么只能有多个不可变引用。
- 引用必须总是有效。

### 4.3 Slice 类型

(字符串 slice 的类似是 `&str`，原来如此，我以前还好奇怎么会有 str 这种类型呢，str 没有单用的。)

字符串 slice 是 String 中一部分值的引用，它看起来像这样：

```rust
let s = String::from("hello world");

let hello = &s[0..5]; // 其它写法：&s[0..=4]; &[..5];
let world = &s[6..11]; // 其它写法：&s[6..=10]; &[6..];

let hello_world = &s[..]; // 表示所有
```

字符串字面值就是 slice。

```rust
let s = "hello world";
```

这里 s 的类型就是 &str (soga!)，它是一个指向二进制程序特定位置的 slice。因为 &str 是一个不可变引用，所以字符串字面值是不可变的。

字符串 slice 作为参数。

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
fn main() {
    let my_string = String::from("hello world");

    // first_word 中传入 `String` 的 slice
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word 中传入字符串字面值的 slice
    let word = first_word(&my_string_literal[..]);

    // 因为字符串字面值 **就是** 字符串 slice，
    // 这样写也可以，即不使用 slice 语法！
    let word = first_word(my_string_literal);
}
```

其它类型的 slice。

```rust
let a = [1,2,3,4,5];
let b: &[i32] = &a[1..3];
```

## 5. 使用结构体组织相关数据

### 5.1 定义并实例化结构体

Rust 中的 struct 和 Go 中的 struct 初始化方式相同，和 ES6 的 object 语法有些类似，比如用 `..` 扩展对象，key 和 value 同名时省略 key 的写法。

```rust
struct user {
    username: string,
    email: string,
    sign_in_count: u64,
    active: bool,
}

fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

//...
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1  // 注意，ES6 中这行要写在最前面，而 Rust 是写在最后面...
};
```

使用没有命名字段的元组结构体来创建不同的类型。(好像 swift 有这用法)

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

没有任何字段的类单元结构体。主要是用来实现 trait，不需要存储数据时使用。

通过派生 trait 增加实用功能。对 struct 使用注解 `#[derive(Debug)]` 让其增加 Display trait，这样就可以用 `println!("{:?}", rect_2)` 打印整个对象。

示例：

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}

fn area_2(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}

fn area_3(rect: &Rectangle) -> u32 {
    rect.width * rect.height
}

fn main() {
    let width = 30;
    let height = 50;

    println!("area is {}", area(width, height));

    let rect = (40, 60);
    println!("area is {}", area_2(rect));

    let rect_2 = Rectangle {
        width: 10,
        height: 20,
    };
    println!("area is {}", area_3(&rect_2));
    println!("rect_2 is {:?}", rect_2);
}
```

### 5.2 一个使用结构体的示例

略。

### 5.3 方法

在 Rust 中，方法是指 struct 的成员函数，而函数是全局的，不属于任何对象。(Go 也是。不过方法和函数本质是一样的。)

在 Rust 中，方法的第一个参数是 &self (或 &mut self)，表示调用该方法的结构体实例。(和 python 类似。)

定义方法：

```rust
impl Rectangle {
    fn aear(&self) -> u32 {
        self.width * self.height
    }
}
```

关联函数，可以简单地理解成其它语言中的静态方法，一般用来作为 builder，比如前面用过的 `String::from()`。

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

// 使用
let sq = Rectangle::squre(50);
```

## 6. 枚举和模式匹配

### 6.1 定义枚举

enums，目测和 swift 中的枚举类似。此文中说与 F#, OCaml, Haskell 中的代数数据类型相似。(有空看一看)

确实，Rust 中的枚举和 swift 中的枚举一样，每个枚举值都可以包裹任意类型的数据。(那后面必然需要从中把包裹的值解构出来)

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

枚举和结构体一样，也可以定义方法 (好巧，swift 也是)。

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}
```

Option，和 swift 中一样，用来解决非空值的问题。

```rust
enum Option<T> {
    Some(T),
    None,
}
```

### 6.2 match 控制流运算符

使用 match 可以把枚举包裹的值解构出来。match 还可以替代 if...else 控制流。

```rust
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

但 match 有个麻烦的地方，你不能只匹配部分情况，必须匹配所有情况，可以用 `_` 匹配所有剩余情况 (类似 switch...case 中的 default)。

如果在 match 中只关心一个情况的，使用 if let 更简单。

### 6.3 if let 简单控制流

```rust
# let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

(swift 也有这个语法)

## 7. 包、Crates 与模块

带有 Cargo.toml 的包用来描述如何构建一个或多个 crate，一个包中最多可以有一个库 crate，但可以有任意多个二进制 crate。二进制 crate 的入口是 src/main.rs，库 crate 的入口是 src/lib.rs。

使用 mod 声明模块，默认内部一切私有，使用 pub 使其公开。使用 use 导入模块。

```rust
// src/main.rs
mod sound {
    pub mod instrument {
        pub fn clarinet() {
            // 函数体
            super::breathe_in(); // 使用 super 相对路径
        }
    }
    fn breathe_in() {
        // 函数体
    }
}

fn main() {
    // 绝对路径
    crate::sound::instrument::clarinet();

    // Relative path
    sound::instrument::clarinet();
}
```

对结构体和枚举使用 pub。对结构体使用 pub，可以使结构体公有，但字段还是私有的，可以在每一个字段的基准上选择其是否公有。而对于枚举类型，只要这个类型是公有的，其所有成员都是公有。

使用 use 关键字将名称引入作用域。略。

通过 as 关键字重命名引入作用域的类型。

```rust
use std::fmt::Result;
use std::io::Result as IoResult;
```

通过 pub use 重导出名称，略。

**使用外部包**

use 的 n 种写法，暂略。

## 8. 通用集合类型

- vector
- 字符串 String
- hash map

这些集合类型都是存放在堆上。

### 8.1 vector

新建 vector:

```rust
let v: Vec<i32> = Vec::new();
let v = vec![1,2,3,4,5]; // vec! 宏
```

更新 vector:

```rust
let mut v = Vec::new();
v.push(5);
```

vector 在离开其作用域时会自动释放内存空间。

读取 vector 的元素：

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
```

遍历 vector 元素：

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

使用枚举来存储多种类型。因为 vector 要求每个元素相同类型，而枚举的每个值是相同类型，但却可以包裹不同类型的数值。于是，一个妙招出现了：

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

(妙！)

### 8.2 字符串 String

(是个难点啊)

> Rust 的核心语言中只有一种字符串类型：str，字符串 slice，它通常以被借用的形式出现，&str。

String 类型并不在 Rust 核心语言中，它由标准库提供。&str 不可变，而 String 可变。两者皆 utf-8 编码。

很多 Vec 可用的操作同样适用于 String。

新建字符串：

```rust
let mut s = String::new();

let data = "hello";
let s = data.to_string();
let s = "hello".to_string();

let s = String::from("initial contents");
```

更新字符串：`+`, `format!`, `push_str`, `push` ...

```rust
let mut s = String::from("foo");
s.push_str("bar");

let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {}", s2); // s2 所有权没有转移

let mut s = String::from("lo");
s.push('l');  // push 单个字符
```

使用 `+` 或 `format!`。

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用，s1 的所有权转给了 s3

let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{}-{}-{}", s1, s2, s3); // s1,s2,s3 的所有权都没有转移，为什么，因为用的是宏？
```

索引字符串。感觉和 swift 一样麻烦。String 实际是 `Vec<u8>`，一个字母可能由多个 u8 组成。

相关概念：字节、标量值、字形簇。总之是挺麻烦的，先跳过吧，需要时再回来看。

可以调用 `chars()` 和 `bytes()` 方法将字符串转换成 char 类型数组或原始字节数组进行遍历。

### 8.3 HashMap

新建，插入：

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

HashMap 和所有权。对于像 i32 这种实现了 Copy trait 的类型，其值可以拷贝进 hashmap，但 String 这种拥有所有权的值，其值将被移动而 hashmap 成为它新的所有者。

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();  // 为什么不用定义 key 和 value 的类型？
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效，
// 尝试使用它们看看会出现什么编译错误！
```

访问值，使用 get() 方法，得到 `Option<T>`。

遍历，使用 for...in。

详略。

## 9. 错误处理

Rust 相比其它语言，它没有异常。将错误分成两类：可恢复错误 (`Resutl<T, E>`) 和不可恢复错误 (`panic!`)。

### 9.1 `panic!` 与不可恢复错误

手动调用这个宏会使程序崩溃退出。其它略。

### 9.2 Result 与可恢复的错误

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

配合 match 使用。

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Tried to create file but there was a problem: {:?}", e),
            },
            other_error => panic!("There was a problem opening the file: {:?}", other_error),
        },
    };
}
```

上面的写法有时显得太啰嗦，有些简写的方法，unwrap() 和 expect() 方法，后者在 guessing game 中已经用过了。

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

unwrap() 方法，如果 Result 值是 Ok，则返回 Ok 包裹的值，否则调用 `panic!`。

expect() 方法和 unwrap() 几乎相同，只不过它接收额外的字符串用于 `panic!` 的输出，而 unwrap() 的 `panic!` 输出默认的内容。

将错误冒泡/传播给上层，即函数也返回 `Result<T, E>` 给上层。

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

将错误传播给上层在 Rust 中很常见，因此有了简写语法 (所谓的语法糖吧) `?`。注意，下例中有两个 `?`。

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

还可以使用 `?` 的链式调用法。(越来越多语言可以这么干了，但 Rust 和它们又有所不同，kotlin/swift 的 `?` 并不会马上返回，而 Rust 会。)

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

## 10. 泛型、trait 和生命周期

### 10.1 泛型

详略。和其它语言的理念是一样的。不过 Rust 的泛型性能和不使用泛型是一样的，因为它像 c++ 的 template 一样，会在编译期就把泛型展开，到了运行期实际已经没有泛型了。

### 10.2 trait

可以简单地理解成其它语言中的接口 interface，用来抽象行为。

Rust 的 trait 中的方法可以有默认实现。(swift 好像也是，但 Go/Java 都没有)。为 struct 实现 trait 用 `impl trait for struct {}` 语法。

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}
impl Summary for NewsArticle {
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
}
```

trait 作为参数。

```rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

`impl Summary` 这种写法是 trait bounds 的语法糖，简化写法，实际是这样：

```rust
pub fn notify<T: Summary>(item: T) {
  ...
}
```

有点像泛型的写法。`impl Trait` 适用于短小的例子，trait bounds 适用于复杂的场景，比如多个相同 trait 类型的参数。

```rust
fn notify(item1: impl Summary, item2: impl Summary) {...}

fn notify<T: Summary>notify(item1: T, item2: T) {...}
```

那为啥不能像其它语言一样，直接把 trait 当类型呢，比如 `fn notify(item1: Summary, item2: Summary){...}`，也许是想用这种语法来和泛型统一。这种写法有点像 Java 中 `List<T extends Summary>` 的写法。

通过 `+` 指定多个 trait。

```rust
fn notify(item: Summary + Display) {...}

fn notify<T: Summary+Display>(item: T) {...}
```

通过 where 简化代码。(好像 swift 也有这语法？)

```rust
fn some_fn<T: Display+Clone, U: Clone+Debug>(t: T, u: U) -> i32 {...}

// where
fn some_fn<T, U>(t: T, u: U) -> i32
    where T: Display+Clone,
          U: Clone+Debug
{...}
```

函数返回 trait。

```rust
fn get_summarizable() -> impl Summary {
  NewsArticle {...}
}
```

使用 trait bound 有条件地实现方法。示例：

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

上例中，为所有 `Pair<T>` 都实现了 new 方法，但只为 T 为 Display 和 PartialOrd 的 `Pair<T>` 实现 cmp_display 方法。

Rust 还可以对任何实现了特定 trait 的类型有条件地实现 trait。下例来自标准库，只为实现了 Display trait 的类型实现 ToString trait，因为整型实现了 Display trait，这就是为什么它能调用 `to_string()` 方法，比如 `let s = 3.to_string();`。

```rust
impl<T: Display> ToString for T {
    // --snip--
    pub fn to_string(&self) {
      // --snip--
    }
}
```

### 10.3 生命周期与引用有效性

生命周期是 Rust 和其它语言最与众不同的功能。其它语言在语言层级并没有这个概念。

生命周期避免了悬垂引用 (野指针)。

借用检查器，一个有效的引用，它引用的数据的生命周期必须比引用的生命周期长，否则就是出现悬垂引用。使用类似 `'a` `'b` 这样的写法来标注生命周期。

**函数中的泛型生命周期**

下面这个函数无法通过编译：

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

因为 Rust 不知道返回的引用是指向 x 还是指向 y，提示返回值需要一个泛型生命周期参数。

**生命周期注解语法**

```rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```

**函数签名中的生命周期注解**

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

这个函数指定了签名中所有的引用必须有相同的生命周期 `'a`。注意前面还有一个泛型的 `<'a>` 声明，生命周期声明确实是泛型。

(为什么需要手动标注，因为编译器很难自己推导出来。)

当具体的引用传递给 longest() 函数时，`'a` 所替代的生命周期是 `x` 的作用域和 `y` 的作用域重叠的那一部分，即较小的那一个。

两个例子，略。

**深入理解生命周期**

如果返回值与某些参数无关，则无须为这些参数标注生命周期。

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
  x
}
```

当从函数返回一个引用，返回值的生命周期参数至少需要与一个参数的生命周期参数相匹配。否则返回的肯定是一个内部引用，这样会导致悬垂引用，不会通过编译的。

**结构体定义中的生命周期注解**

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

上例的意思是说 ImportantExcerpt 实例不能比其 part 字段中的引用存在的更久。

**生命周期省略（Lifetime Elision）**

在某些情况下，如果编译器能自动推导出生命周期，就可以省略写生命周期。比如 `fn first_word(s: &str) -> &str {...}` 能自动推断为 `fn first_word<'a>(s: &'a str) -> &'a str {...}`，就可以用前面的省略写法。

有三条规则，详略。

**方法定义中的生命周期注解**

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

**静态生命周期**

`'static`，其生命周期存活于整个程序期间。

```rust
let s: &'static str = "I have a static lifetime.";
```

**结合泛型类型参数、trait bounds 和生命周期**

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

## 11. 测试

这一章先简单了解，需要时再详细看。

使用 `cargo test` 执行测试，就是所有测试框架一样，支持对部分文件进行测试，忽略某些测试，多线程进行测试等。

测试分单元测试和集成测试，单元测试和实现代码在同一个文件中，集成测试在单独的 tests 目录中。

常见的断言宏：assert!(), assert_eq!(), assert_ne!(), panic!() 等等。

单元测试的写法：

```rust
// src/lib.rs
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

`#[cfg(test)]` 注解表示被注解的代码属于测试代码，不会在 `cargo build` 时执行，不会被打包到最终长成的二进制代码中，仅在 `cargo test` 时执行。

用 `#[test]` 注解执行测试的函数。

另外，Rust 允许测试私有函数，而其它大部分语言不可以。

其实暂略。

## 12. 一个 I/O 项目：构建一个命令行程序

本章属于实战环节。相当于是期中复习，然后接下来是进阶内容，最后又是一个实战环节，期末考试。

实现一个 grep，将会用到以下上面学习的内容：

- 代码组织
- vector 和字符串
- 错误处理
- 合理的使用 trait 和生命周期
- 测试

另外还会用到闭包、迭代器和 trait 对象，后面会详略讲解。

### 12.1 接受命令行参数

运行 `cargo new minigrep` 命令新建可执行文件项目。

```rust
// src/main.rs
use std::env;

fn main() {
    // 1. receive args
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("args: {:?}", args);
    println!("query: {}", query);
    println!("filename: {}", filename);
}
```

`env::args()` 得到一个迭代器，`collect()` 获取迭代器的所有值并转换成集合。

执行 `cargo run test sample.txt"，得到输出：

```shell
args: ["target/debug/minigrep", "test", "sample.txt"]
query: test
filename: sample.txt
```

### 12.2 读取文件

在项目根目录下创建 poem.txt，填充一些内容。然后我们读取它的内容并打印。

```rust
use std::env;
use std::fs;

fn main() {
    // 1. receive args
    // ...
    // 2. read file
    let contents = fs::read_to_string(filename).expect("failed to read file");
    println!("contents: {}", contents);
}
```

执行 `cargo run test poem.txt` 在控制栏得到文件内容。

### 12.3 重构改进模块性和错误处理

详细过程暂略。主要目的是让 main() 函数简洁，其它逻辑挪到 crate 中 (即 src/lib.rs)，方便后面加测试。以及加上了各种错误处理。

重构后的代码：

```rust
// src/lib.rs
use std::error::Error;
use std::fs;

pub struct Config {
  pub query: String,
  pub filename: String
}

impl Config {
  pub fn new(args: &[String]) -> Result<Config, &'static str> {
    if args.len() < 3 {
      return Err("not enough argument");
    }

    let query = args[1].clone();
    let filename = args[2].clone();

    Ok(Config{query, filename})
  }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
  let contents = fs::read_to_string(config.filename)?;
  println!("contents:\n {}", contents);
  Ok(())
}
```

```rust
// src/main.rs
use std::env;
use std::process;

use minigrep;
use minigrep::Config;

fn main() {
    // 1. receive args
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem: {}", err);
        process::exit(1);
    });

    println!("query: {}", config.query);
    println!("filename: {}", config.filename);

    // 2. read file
    if let Err(e) = minigrep::run(config) {
        println!("error: {}", e);
        process::exit(1);
    }
}
```

`Ok(())` 中的 `()` 是 unit 类型。

### 12.4 采用测试驱动开发完善库的功能

先写测试，再写实现。

```rust
// src/lib.rs
#[cfg(test)]
mod tests {
  use super::*;

  #[test]
  fn one_result() {
    let query = "duct";
    let contents = "\
Rust:
safe, fast, productive.
Pick three.";
    assert_eq!(search(query, contents), vec!["safe, fast, productive."]);
  }
}
```

实现：

```rust
// src/lib.rs
fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
  let mut results = Vec::new();
  for line in contents.lines() {
    if line.contains(query) {
      results.push(line);
    }
  }
  results
}
```

使用：

```rust
// src/lib.rs
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
  let contents = fs::read_to_string(config.filename)?;

  for line in search(&config.query, &contents) {
    println!("{}", line);
  }

  Ok(())
}
```

### 12.5 处理环境变量

先写测试。增加 `search_case_insensitive()` 方法。在 Config 中新增 `case_sensitive: bool` 属性。在 Config 的 new() 方法中，从环境变量中读取 `CASE_SENSITIVE`，赋值给 `Config.case_sensitive`。然后在 run() 方法中，根据 `Config.case_sensitive` 的值来执行 search() 方法还是 `search_case_insensitive()` 方法。

详略。

### 12.6 将错误信息输出到标准错误而不是标准输出

使用 `eprintln!` 宏。

## 13. Rust 中的函数式语言功能：迭代器与闭包

闭包，迭代器。

### 13.1 闭包：可以捕获环境的匿名函数

> Rust 的闭包（closures）是可以保存进变量或作为参数传递给其他函数的匿名函数。可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。不同于函数，闭包允许捕获调用者作用域中的值。我们将展示闭包的这些功能如何复用代码和自定义行为。

(一定要是匿名函数吗？js 中好像不是必须的呀。js 中闭包和匿名函数是独立的。不过 js 毕竟是个不完善的语言。)

**使用闭包创建行为的抽象**

讨论闭包的语法、类型推断和 trait。

闭包的定义：

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

闭包不要求像函数那样注明参数和返回值的类型，可以自动推导。

> 闭包通常很短并只与对应相对任意的场景较小的上下文中。在这些有限制的上下文中，编译器能可靠的推断参数和返回值的类型，类似于它是如何能够推断大部分变量的类型一样。

**使用带有泛型和 Fn trait 的闭包**

缓存计算结果：memorization 或 lazy evaluation 模式。

创建一个存放闭包和调用闭包结果的结构体。为了让结构体存放闭包，需要指定闭包的类型，此时需要使用泛型和 trait bound。

所有的闭包都实现了 `Fn`, `FnMut` 或 `FnOnce` 中的一个。示例：

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32 // T 是我们想要的闭包类型
{
  calculation: T, // 闭包
  value: Option<u32>,
}
```

(可是，为什么不简单地用 `|u32| -> u32` 来表示呢？可能是这样弄编译器不好写吧...)

Fn 让我想起了 RxJava 中抽象出来的 Func 接口：

```java
public interface Func<T, R> {
    R call(T t);
}
```

最终的实现：

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
```

**闭包会捕获环境**

示例：

```rust
fn main() {
    let x = 4;
    let equal_to_x = |z| z == x;
    let y = 4;
    assert!(equal_to_x(y));
}
```

x 并不是 `equal_to_x` 的参数，却使用了 x，因为 x 和这个闭包函数处于相同作用域。

但在 Rust 中函数却不可以访问函数外作用域的变量。

```rust
fn main() {
    let x = 4;
    fn equal_to_x(z: i32) -> bool { z == x }
    let y = 4;
    assert!(equal_to_x(y));
}
```

上例编译出错。(看来 Rust 严格区分是普通函数和闭包函数，而 JavaScript 却并没有。)

> 闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：获取所有权，可变借用和不可变借用。这三种捕获值的方式被编码为如下三个 Fn trait：

> - FnOnce 消费从周围作用域捕获的变量，闭包周围的作用域被称为其环境，environment。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 Once 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
> - FnMut 获取可变的借用值所以可以改变其环境
> - Fn 从其环境获取不可变的借用值

如果要把环境的所有权移到闭包内，使用 move 关键字。

```rust
fn main() {
    let x = vec![1, 2, 3];
    let equal_to_x = move |z| z == x;  // x 的所有权移到闭包内
    println!("can't use x here: {:?}", x);
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```

### 13.2 用迭代器处理元素序列

在 Rust 中，迭代器是惰性的。(大部分语言中都是吧)

迭代器都实现了一个叫 Iterator 的 trait，这个 trait 最重要的方法就是 next()。

```rust
trait Iterator {
  type Item;
  fn next(&mut self) -> Option<Self::Item>;
}
```

`type Item` 和 `Self::Item` 是 trait 的关联类型 (associated type)，后面会讲。

使用：

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

迭代器的所有权看着像是个复杂的问题。

**消费迭代器的方法**

调用 next() 方法的方法被称为消费迭代器 (consuming adaptors)，因为调用它们会消耗迭代器。一个例子是 sum() 方法，它获取迭代器的所有权并反复调用 next() 来遍历迭代器，因而会消费迭代器。

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
    let total: i32 = v1_iter.sum(); // 调用 sum() 后不可以再访问 v1_iter，因为所有权被转移了
    assert_eq!(total, 6);
}
```

**产生其它迭代器的方法**

> Iterator trait 中定义了另一类方法，被称为迭代器适配器（iterator adaptors），他们允许我们将当前迭代器变为不同类型的迭代器。可以链式调用多个迭代器适配器。不过因为所有的迭代器都是惰性的，必须调用一个消费适配器方法以便获取迭代器适配器调用的结果。

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

使用 map() 创建新的迭代器后还需要调用 collect() 来消费它，否则编译不通过。map() 里是闭包 (不是匿名函数吗？感觉 Rust 里闭包和匿名函数已经是一体了)。

**使用闭包获取环境**

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}
```

`filter(|s| s.size == shoe_size)` 这个匿名函数捕获了 `shoe_size` 参数。

**实现 Iterator trait 来创建自定义迭代器**

实现 next() 方法即可。

```rust
# struct Counter {
#     count: u32,
# }
#
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

**使用自定义迭代器中其他 Iterator trait 方法**

标准库中还为 Iterator trait 实现了很多除 next() 方法外的其它方法的默认实现，这些实现都调用了 next() 方法，比如 zip(), map(), filter() 等等。

所以只要实现了 next() 方法就能使用 Iterator trait 中的其它方法。

### 13.3 改进 I/O 项目

**使有迭代器并去掉 clone**

```rust
// src/main.rs
fn main() {
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });
    // --snip--
}

// src/lib.rs
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

**使用迭代器适配器来使代码更简明**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

(要是 js 我第一时间就会想到用这种办法。)

### 13.4 性能对比：循环 vs 迭代器

迭代器是 Rust 的零成本抽象之一，不会造成性能下载，甚至比循环的性能还好一丢丢...

## 14. Cargo 和 Crates.io

这一章先跳过。需要时再回来看。

## 15. 智能指针

智能指针是一类数据结构，表现类似指针，但也拥有额外的元数据和功能。Rust 标准库中不同的智能指针提供了多于引用的额外功能。一个例子是引用计数智能型指针，其允许数据有多个所有者。引用计数智能指针记录总共有多少个所有者，并当没有任何所有者时负责清理数据。

引用只是一类只借用数据的指针，但大部分情况，智能指针拥有他们指向的数据。

前面用的 String 和 `Vec<T>` 实际也是智能指针。

> 智能指针通常使用结构体实现。智能指针区别于常规结构体的显著特性在于其实现了 Deref 和 Drop trait。Deref trait 允许智能指针结构体实例表现的像引用一样，这样就可以编写既用于引用又用于智能指针的代码。Drop trait 允许我们自定义当智能指针离开作用域时运行的代码。

Rust 标准库中最常用的智能指针：

- `Box<T>`，用于在堆上分配值
- `Rc<T>`，一个引用计数类型，其数据可以有多个所有者
- `Ref<T>` 和 `RefMut<T>`，通过 `RefCell<T>` 访问，一个在运行时而不是在编译时执行借用规则的类型

还会讨论内部可变性模式及引用循环。

### 15.1 `Box<T>` 在堆上存储数据，并且可确定大小

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

Box 允许创建递归类型。cons list，一个函数式编程中常见的类型。

```rust
enum List {
    Cons(i32, List),
    Nil,
}
```

上例并不能编译通过，因为无法知道确定的大小，因而不知道要分配多少内存空间。

解决办法：

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

嗯，这和 c/c++ 中定义二叉树是一样的，left/right 必须是指针。

```cpp
typedef int NodeType;

struct Tree {
  NodeType value;
  Tree *left;
  Tree *right;
}
```

> `Box<T>` 类型是一个智能指针，因为它实现了 Deref trait，它允许 `Box<T>` 值被当作引用对待。当 `Box<T>` 值离开作用域时，由于 `Box<T>` 类型 Drop trait 的实现，box 所指向的堆数据也会被清除。

### 15.2 通过 Deref trait 将智能指针当作常规引用处理

> 实现 Deref trait 允许我们重载解引用运算符（dereference operator）`*`（与乘法运算符或 glob 运算符相区别）。通过这种方式实现 Deref trait 的智能指针可以被当作常规引用来对待，可以编写操作引用的代码并用于智能指针。

核心示例代码：

```rust
use std::ops::Deref;

# struct MyBox<T>(T);
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

暂时没看懂，先跳过。

### 15.3 使用 Drop Trait 运行清理代码

类似 c++ 的析构函数。实现 drop 方法。先跳过。

c++ 中，当变量离开作用域时，只会自动析构栈上的内存空间，而不会析构堆上的内存空间，堆上的内存空间需要手动显式释放。而 Rust 会自动析构堆上的内存空间。

### 15.4 `Rc<T>` 引用计数智能指针

`Rc<T>` 允许允许变量拥有多个所有权。略复杂，先跳过。

### 15.5 `RefCell<T>` 和内部可变性模式

先跳过。

### 15.6 引用循环与内存泄漏

Rust 并不保证完全地避免内存泄漏。(Rust 只是能尽最大可能避免野指针造成的崩溃。)

解决办法：`Weak<T>` 弱引用。(swift 中也有弱引用)。

先跳过，回头再细看。整个 15 章都要重新再看。

## 16. 无畏并发

Rust 能在编译期发现并发错误，从而避免运行期的错误。

本章内容：

- 创建线程同时运行多段代码
- 消息传递 (message passing) 并发，使用通道 (channel) 在线程间传递消息 (goroutine?)
- 共享状态 (shared state) 并发
- Sync 和 Send trait

### 16.1 使用线程同时运行代码

编程语言提供的线程被称为绿色（green）线程... (实现细节先跳过...)

- 使用 thread::spawn 创建线程
- 使用 thread::join 等待线程结束

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

**线程与 move 闭包**

使用 move 将变量的所有权从一个线程转换到另一个线程。(很好理解)

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

### 16.2 使用消息传递在线程间传送数据

> 一个人气正在上升的确保安全并发的方式是消息传递（message passing），这里线程或 actor 通过发送包含数据的消息来相互沟通。这个思想来源于 Go 编程语言文档中的口号："不要共享内存来通讯；而是要通讯来共享内存。"

Rust 也有通道，通道两头一端是发送者，一端是接收者。当发送者或接收者任一被丢弃时可以认为通道被关闭。

示例：

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

rx 的 recv() 方法会阻塞线程，还有一个方法 `try_recv()` 不会。recv() 的返回值是 Result，所以要用 unwrap() 处理。

**通道与所有权转移**

```rust
let val = String::from("hi");
tx.send(val).unwrap();      // val 的所有权转移到 send() 方法中了
println!("val is {}", val); // 错误，val 不再可以被访问
```

**发送多个值并观察接收者的等待**

```rust
fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

详略，看代码就明白了。

**通过克隆发送者来创建多个生产者**

mpsc: multiple producer, single consumer。

```rust
let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx);
// --snip--
```

### 16.3 共享状态并发

使用互斥器 `Mutex<T>`。在其它语言，上锁和解锁可能出错，但 Rust 得益于类型系统和所有权，则不会出错。

所有权的问题，有点复杂，先跳过。

### 16.4 使用 Sync 和 Send trait 的可扩展并发

- 通过 Send 允许在线程间转移所有权
- Sync 允许多线程访问

详略，先跳过。

## 17. Rust 的面向对象特性

Rust 传统 OOP 语言 (c++/java/c#) 中的 class 概念。Rust 是混合范式。

### 17.1 面向对象语言的特征

传统课本上常说的 OOP 三大特性：封装，继承，多态。

- 对象包含数据和行为
- 封装隐藏了实现细节
- 继承，作为类型系统与代码共享

Rust 满足封装，多态的特性，但没有实现继承。传统 OOP 语言的多态是通过继承来实现的，但 Rust 是通过 trait 来实现的，而 Go 是通过 interface 来实现的。

> 近来继承作为一种语言设计的解决方案在很多语言中失宠了，因为其时常带有共享多于所需的代码的风险。子类不应总是共享其父类的多有特征，但是继承却始终如此...

### 17.2 为使用不同类型的值而设计的 trait 对象

勿需赘言，多态的实现，不同的类型对象实现相同的接口方法。

多态是在运行期进行动态分发。

**Trait 对象要求对象安全**

- 返回值类型不为 Self
- 方法没有任何泛型类型参数

Self 关键字是我们要实现 trait 或方法的类型的别名。

### 17.3 面向对象设计模式的实现

用了一个示例演示 Rust 也是可以实现 OOP 的设计模式。详略。

## 18. 模式用来匹配值的结构

(是讲 match 吧，match 是 elixir 的核心。)

> 模式是 Rust 中特殊的语法，它用来匹配类型中的结构，无论类型是简单还是复杂。结合使用模式和 match 表达式以及其他结构可以提供更多对程序控制流的支配权。

模式由以下内容组合而成：

- 字面值
- 解构的数组、枚举、结构体或者元组
- 变量
- 通配符
- 占位符

(好像不仅仅是 match，match 只是其中之一。)

### 18.1 所有可能会用到模式的位置

match / if let 前面讲过了。还有 while let / for / let / 函数参数。好像，最常见的 let 也可以被理解成是模式匹配，嗯，好像 elixir 也是这样的，elixir 中 `=` 不是赋值，而是匹配，一切皆匹配。

while let:

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

for:

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

let:

```rust
// let PATTERN = EXPRESSION;
let x = 5;
let (x, y, z) = (1, 2, 3);
```

函数参数：

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

### 18.2 Refutability（可反驳性）: 模式是否会匹配失效

> 模式有两种形式：refutable（可反驳的）和 irrefutable（不可反驳的）。能匹配任何传递的可能值的模式被称为是不可反驳的（irrefutable）。一个例子就是 `let x = 5;` 语句中的 x，因为 x 可以匹配任何值所以不可能会失败。对某些可能的值进行匹配会失败的模式被称为是可反驳的（refutable）。一个这样的例子便是 `if let Some(x) = a_value` 表达式中的 Some(x)；如果变量 `a_value` 中的值是 None 而不是 Some，那么 Some(x) 模式不能匹配。

> let 语句、 函数参数和 for 循环只能接受不可反驳的模式，因为通过不匹配的值程序无法进行有意义的工作。if let 和 while let 表达式被限制为只能接受可反驳的模式，因为根据定义他们意在处理可能的失败：条件表达式的功能就是根据成功或失败执行不同的操作。

嗯，显而易见的。理解。

### 18.3 所有的模式语法

总结了所有的模式语法，有些前面已经很常见了，这里再列举一些新的：

- 使用 `|` 同时匹配多个值
- 使用 `...` 匹配范围
- 使用 `_` 忽略
- 使用 `..` 匹配剩余
- 使用 `@` 将匹配的值绑定到另一个变量上
- 解构
- ...

需要时再回来看。

## 19. 高级特性

- 不安全 Rust：unsafe
- 高级生命周期
- 高级 trait
- 高级类型
- 高级函数和闭包：函数指针和返回闭包
- 宏

先简单了解吧。还是先跳过吧。

### 19.1 不安全 Rust

unsafe Rust 可以做的事：

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变静态变量
- 实现不安全 trait

详略。

## 20. 最后的项目: 构建多线程 web server

1. 学习一些 TCP 与 HTTP 知识
1. 在套接字（socket）上监听 TCP 请求
1. 解析少量的 HTTP 请求
1. 创建一个合适的 HTTP 响应
1. 通过线程池改善 server 的吞吐量

### 20.1 构建单线程 web server

(经测试发现，有时在浏览器必须访问 `localhost:7878` 才能得到预期结果，而不是 `127.0.0.1:7878`。)

```rust
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::fs;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    let (status_line, file_name) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 Not Found\r\n\r\n", "404.html")
    };

    let contents = fs::read_to_string(file_name).unwrap();
    let response = format!("{}{}", status_line, contents);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

### 20.2 将单线程 server 变为多线程 server

使用线程池创建有限数量个线程处来连接。有点小复杂，先跳过。

### 20.3 优雅停机与清理

实现 Drop trait 和向线程发送信号使其停止接收任务，暂时先跳过。
