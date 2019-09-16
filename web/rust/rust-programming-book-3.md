# 《The Rust Programming Language》Note 3

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
