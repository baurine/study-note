# 《The Rust Programming Language》Note 4

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
