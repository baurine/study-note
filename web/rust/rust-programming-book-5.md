# 《The Rust Programming Language》Note 5

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
