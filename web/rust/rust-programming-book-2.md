# 《The Rust Programming Language》Note 2

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
