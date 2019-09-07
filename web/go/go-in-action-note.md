# 《Go in Action》Note

## 第 1 章 - 关于 Go 语言的介绍

编译速度快。

并发：内置，goroutine / channel，无线程

类型系统：静态类型，无继承的类型系统，使用组合设计模式，鸭子模式 (不需要显式声明某个类型实现了某个接口，只要你实现了这个接口，你自动就是这个类型)，对行为进行建模，一个接口一般只实现一个方法。

内存管理：垃圾回收。

其余略。

## 第 2 章 - 快速开始一个 Go 程序

在没有 go module 之前，所有 go 代码必须放在 GOPATH 或 GOROOT (`/Users/me/go`) 路径中，代码中没有相对路径，都使用绝对路径。

本章演示了一个小型的完整程序代码。

目录结构：

```
- sample
  - data
      data.json     -- 包含一组数据源
  - matchers
      rss.go        -- 搜索 rss 源的匹配器
  - search
      default.go    -- 搜索数据用的默认匹配器
      feed.go       -- 用于读取 json 数据文件
      match.go      -- 用于支持不同匹配器的接口
      search.go     -- 执行搜索的主控制逻辑
  main.go           -- 程序的入口
```

main.go，包括入口函数 main()，init() 函数在 main() 之前执行，入口函数所在包名必须为 main `package main`。

```
import (
  "log"
  "os"

  _ "github.com/goinaction/code/chapter2/sample/matchers"
  "github.com/goinaction/code/chapter2/sample/search"
)
```

用 `_` 声明导入但并不使用的包，不然通不过编译，虽然没有使用，但此包的 init() 函数会自动执行。

> 所有处于同一个文件夹里的代码文件，必须使用同一个包名。按照惯例，包和文件夹同名。

### 2.3 - search 包

整个程序的核心逻辑都在 search.go 中的 Run() 函数中。main.go 几乎只做了一件事，那就是调用 `search.Run('keyword')`。

```go
// Run performs the search logic.
func Run(searchTerm string) {
	// Retrieve the list of feeds to search through.
	feeds, err := RetrieveFeeds()
	if err != nil {
		log.Fatal(err)
	}

	// Create an unbuffered channel to receive match results to display.
	results := make(chan *Result)

	// Setup a wait group so we can process all the feeds.
	var waitGroup sync.WaitGroup

	// Set the number of goroutines we need to wait for while
	// they process the individual feeds.
	waitGroup.Add(len(feeds))

	// Launch a goroutine for each feed to find the results.
	for _, feed := range feeds {
		// Retrieve a matcher for the search.
		matcher, exists := matchers[feed.Type]
		if !exists {
			matcher = matchers["default"]
		}

		// Launch the goroutine to perform the search.
		go func(matcher Matcher, feed *Feed) {
			Match(matcher, feed, searchTerm, results)
			waitGroup.Done()
		}(matcher, feed)
	}

	// Launch a goroutine to monitor when all the work is done.
	go func() {
		// Wait for everything to be processed.
		waitGroup.Wait()

		// Close the channel to signal to the Display
		// function that we can exit the program.
		close(results)
	}()

	// Start displaying results as they are available and
	// return after the final result is displayed.
	Display(results)
}
```

首先调用 RetrieveFeeds() 方法，这个方法在 feed.go 中实现，主要工作是从 data/data.json 读取内容并解析 json 成 `[]Feed`。

然后对这个 `[]Feed` 切片进行遍历，对每一个 Feed，通过它的 type 去找到相应的 matcher，如果没有相应的 matcher，则用默认的 default matcher。目前程序实现了两个 matcher，一个是 default matcher，在 default.go 中实现，一个是 rss matcher，在 matcher/rss.go 中实现，matcher 需要实现 Search() 方法。它们在 init() 函数中调用 search.go 中定义的 Register() 方法注册到全局变量 `var matchers = make(map[string]Matcher)` 中。

对每一个 Feed，起一个 goroutine 对它进行搜索，调用 Match(matcher...) 方法进行搜索，Match(matcher...) 方法在 match.go 中定义，实际内部则调用 matcher.Search() 进行搜索，得到结果后，通过之前定义的通道 `results := make(chan *Result)`，将结果传输到通道中。

此方法同时使用了 waitGroup 进行并发的控制，在所有协程完成后将通道关闭。

最后 `Display(results)` 打印结果，从通道中读取值并打印，当所有协程完成，通道关闭时，此函数将退出，整个程序结束。

//TODO: 自己实现一遍

> 命名接口的时候，也需要遵守 Go 语言的命名惯例。如果接口类型只包含一个方法，那么这个类型的名字以 er 结尾。

另外，在 Go，以小写字母开头的变量或类型，包外不可访问，以大写字母开头的变量或类型，包外可访问。Go 中并没有 public/private 这些关键字。

剩余详略。

## 第 3 章 - 打包和工具链

### 3.1 - 包

main 包会被 go 编译器编译成二进制执行文件，其它包不会。

### 3.2 - import

导入包：本地导入，远程导入，命名导入 (`import (myfmt "mylib/fmt")`)

### 3.3 - init 函数

> 每个包可以包含任意多个 init 函数，这些函数都会在程序执行开始的时候被调用。

### 3.4 - Go 工具

```shell
$ go
$ go build ...
$ go run ...
$ go clean ...
```

### 3.5 - Go 工具 2

- `go vet` - 帮助开发人员检测代码的常见错误，有点类似 lint
- `go fmt` - 格式化代码
- `go doc package` - 查看包的文档
- `godoc -http=:6060` - 启动文档服务器

### 3.7 - 依赖管理

略。现在据说官方出了 go modules，这本书出版时还没有。

## 第 4 章 - 数组、切片和映射

### 4.1 - 数组

内存结构是一段连续可随机访问的空间，不是链表。

声明：`var array [5]int`

声明并初始化：

```
array := [5]int{10, 20, 30, 40, 50}
array := [...]int{10, 20} // 长度为 2
array := [5]int{1: 20, 2: 30} // 其余为 0

array := [2]*int{new(int), new(int)} // 存储数组
array := [5]*int{0: new(int), 1: new(int)} // 存储数组
```

使用：略。

### 4.2 - 切片

切片是 Go 的一个特性，需要重点学习和掌握。

> 切片是一个很小的对象，对底层数组进行了抽象，并提供相关的操作方法。

创建和初始化，可以使用 make 和切片字面量。

```go
slice := make([]string, 5)  // 长度和容量都是 5 的字符串切片
slice := make([]string, 3, 5) // 长度是 3，容量是 5 的字符串切片

slice := []string{"Red", "Blue", "Green", "Yellow", "Pink"} // 长度和容量为 5 的字符串切片
slice := []int{10, 20, 30}

slice := []string{99: ""} // 长度为 100
```

nil 切片：`var slice []int`

空切片：

```go
slice := make([]int, 0)
slice := []int{}
```

使用切片：

```go
slice := []int{10, 20, 30, 40, 50} // 长度和容量为 5 的数值切片
newSlice := slice[1:3]  // 长度为 (3-1) = 2，容量为 (5-1) = 4 的数值切片
newSlice[1] = 35 // 会同时修改 slice[2]
```

切片增长：使用 append() 函数。如果 append 后长度超过来原来切片的容量，底层会新建一个切片，并将原来切片的内容全部拷贝过来。

```go
newSlice = append(newSlice, 60)
```

创建切片还可以用第二个参数，用来指定容量。

```go
source := []string{"Apple", "Orange", "Plum", "Banana", "Grape"}
slice := source[2:3:4] // 该切片从底层数组引用了 1 (3-2) 个元素，容量为 2 (4-2)
```

但最好是设置长度和容量一样，这样往切片 append 新元素时，不会覆盖底层数组原来的值，而是会产生一个新的切片。如上例这样写更好：`slice := source[2:3:3]`。

append() 函数接受可变参数，一次调用可以追加多个值，如 `append(slice, 1, 2, 3)`，还可以使用类似 ES6 中扩展运行符 `...`，只不过 go 是放在变量后面，比如将 slice2 所有值追加到 slice1 中 `append(slice1, slice2...)`。

迭代切片，使用 for...range，注意，range 将会创建迭代对象的副本，而不是引用。

```go
slice := []int{10, 20, 30, 40}
for index, value := range slice {
  fmt.Printf("Index: %d, Value: %d\n", index, value)
}
```

len() 和 cap() 函数可用于切片，数组，通道，用于获取其长度和容量。

在函数间传递切片。由于切片对象自身的体积很小 (本身其实已经算是引用了)，所以可以直接在函数间传递副本，而不用传递引用。

### 4.3 - 映射

创建和初始化，使用 make 或字面量，后者更常用。

```go
dict := make(map[string]int)
dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}
```

nil 映射和空映射，前者不能用来存储键值。

```go
// nil map
var colors map[string]string
colors["Red"] = "#da1337"  // error

// 空 map
var colors = map[string]string{}
colors["Red"] = "#da1337"  // ok
```

判断键值是否存在：

```go
value, exists := colors["Blue"]
if exists {
  fmt.Println(value)
}
```

range：

```go
for key, value := range colors {
  fmt.Printf("Key: %s, Value: %s\n", key, value)
}
```

删除某个键值对，使用 delete() 函数：`delete(colors, "Blue")`

在函数间传递，和切片一样，map 本身就是一种引用了，所以直接在函数间传递副本即可。

## 第 5 章 - Go 的类型系统

### 5.1 - 用户定义的类型

有两种，一种是用 `type myType struct {}` 定义一个结构体，一种是用 `type myType existedType` 如 `type Duration int64`，用已有的类型创建一个新的类型，但注意不是当别名用 (和其它语言不一样)。

```go
type user struct {
  name  string
  email string
  ext   int
  privileged bool
}

// 声明，初始化
var bill user  // 值默认为零值

lisa := user{
  name: "Lisa",
  email: "lisa@email.com",
  ext: 123,
  privileged: true
}

// or
lisa := user{"Lisa", "lisa@email.com", 123, true}
```

### 5.2 - 方法

成员方法，但不严格是，这是 Go 和其它语言很不一样的地方，和 ruby/object-c 发送消息的理念相似。

定义成员方法在 Go 中是这样的：

```go
type user struct {
  name string
  email string
}

// 值接收者
func (u user) notify() {
  fmt.Printf("%s<%s>\n", u.name, u.email)
}

// 指针接收者
func (u *user) changeEmail(email string) {
  u.email = email
}
```

注意，使用值作为接收者时，实际是生成的值的副本来调用  此方法，所以如果在这个方法中修改接收者的属性，实际并不会影响接收者原始对象，而使用指针作为接收者，是直接在接收者自身上调用此方法。

### 5.3 - 类型的本质

使用值还是指针作为接收者，其原则是什么？

> 如果是要创建一个新值，该类型的方法就使用值接收者。如果是要修改当前值，就使用指针接收者。

对于引用类型，比如切片，可以直接使用值接收者。所以上面这句话表述并不严谨，自己理解就行。

内置类型：数值，字符串 (严格来说也是引用)，布尔型，对这些值进行修改时，会创建一个新值。

引用类型：切片，映射，通道，接口和函数。

struct，它不是引用类型，当它使用值接收者时，会产生整体拷贝。

```go
func Now() Time {
  sec, nsec := now()
  return Time{sec + unixToInternal, nsec, Local} 784
}

func (t Time) Add(d Duration) Time {
	t.sec += int64(d / 1e9)
	nsec := int32(t.nsec) + int32(d%1e9)
	if nsec >= 1e9 {
		t.sec++
		nsec -= 1e9
	} else if nsec < 0 {
		t.sec--
		nsec += 1e9
	}
	t.nsec = nsec
	return t
}
```

防止整体被拷贝的一个方法，再用一个 struct 包一层，上层这个 struct 里只放被包裹 struct 的指针，如下所示：

```go
type File struct {
  *file
}

type file struct {
  fd int
  name string
  dirinfo *dirinfo
  nepipe int32
}
```

像 file 这样不能被拷贝的对象，应该对它一直使用指针。

### 5.4 - 接口

定义接口，实现接口：

```go
// notifier 是一个定义了通知类行为的接口
type notifier interface {
  notify()
}

// user 在程序里定义一个用户类型
type user struct {
  name string
  email string
}

// notify 是使用指针接收者实现的方法
func (u *user) notify() {
  fmt.Printf(...)
}
```

使用：

```go
func sendNotification(n notifier) {
  n.notify()
}

func main() {
  u := user{"Bill", "bill@email.com"}
  sendNotification(u) // error!
  sendNotificaiton(&u) // ok
}
```

当使用 `sendNotification(u)` 时，提示 user 类型并没有实现 notifier 接口，因为实现此接口的是 `*user`。

方法集：

| Values | Methods Receivers |
| ------ | ----------------- |
| T      | (t T)             |
| \*T    | (t T) and (t \*T) |

当传递值作为参数时，方法接收者只能是值；当传递指针作为参数时，方法接收者既可以是值，也可以是指针。

从接收者的角度来看：

| Methods Receivers | Values    |
| ----------------- | --------- |
| (t T)             | T and \*T |
| (t \*T)           | \*T       |

当方法接收者为值时，调用者的传递参数既可以是值，又可以是指针；但如果方法接收者为指针时，调用者的传递参数只能是指针。

多态：略。

### 5.5 - 嵌入类型

> Go 语言允许用户扩展或者修改已有类型的行为。这个功能对代码复用很重要，在修改已有类型以符合新类型的时候也很重要。这个功能是通过嵌入类型 (type embedding) 完成的。嵌入类型是将已有的类型直接声明在新的结构类型里。被嵌入的类型被称为新的外部类型的内部类型。

> 通过嵌入类型，与内部类型相关的标识符会提升到外部类型上。这些被提升的标识符就像直接声明在外部类型里的标识符一样，也是外部类型的一部分。这样外部类型就组合了内部类型包含的所有属性，并且可以添加新的字段和方法。外部类型也可以通过声明与内部类型标识符同名的标识符来覆盖内部标识符的字段或者方法。这就是扩展或者修改已有类型的方法。

(这样的设计会带来什么便利吗？是出于什么工程上的考虑呢？)

示例代码：

```go
type user struct {
  name string
  email string
}

func (u *user) notify() {
  fmt.Printf(...)
}

type admin struct {
  user // 嵌入 user 类型，注意，并不是 ES6 中当 key value 相同时可以省略 value 的语法，这里相当于使用组合替代继承 (个人理解)
  level string
}

func main() {
  ad := admin{
    user: user{"Bill", "bill@email.com"},
    level: "super"
  }

  // 我们可以直接访问内部类型的方法
  ad.user.notify()
  // 内部类型的方法也被提升到外部类型
  ad.notify()
}
```

相比其它语言很特别啊。

> 要嵌入一个类型，只需要声明这个类型的名字就可以了。

将 user 类型嵌入 admin 类型后，我们称 user 类型是外部类型 admin 的内部类型。

> 通过内部类型的名字可以访问内部类型...这就意味着，虽然没有指定内部类型对应的字段名，还是可以使用内部类型的类型名，来访问到内部类型的值。

外部类型嵌入内部类型后，可以当内部类型使用：

```go
type notifier interface {
  notify()
}

func sendNotification(n notifier) {
  n.notify()
}

func main() {
  ...
  sendNotification(&ad)
}
```

个人理解，嵌入类型是使用组合代替继承的一种机制。

当外部类型实现了和内部类型一样的方法后，内部类型的方法不会被提升。(显而易见)

### 5.6 - 公开或未公开的标识符

小写字母开头的变量、属性或类型是非公开的，外部不可访问，大写字母开头则外部可以访问。

一些技巧，可以在公开的方法中返回非公开的变量。

```go
// counters/counter.go
package counters

type alertCounter int

func New(value int) alertCounter {
  return alertCounter(value)
}
```

使用：

```go
func main() {
  counter := counters.New(10)
  ...
}
```

> 将工厂函数命名为 New 是 Go 语言的一个习惯。

## 第 6 章 - 并发

> Go 语言的语法和运行时直接内置了对并发的支持。

(这是和其它语言最大的区别了)

> Go 语言的并发同步模型来自一个叫作通信顺序进程 (Communicating Sequential Processes，CSP) 的范型 (paradigm)。CSP 是一种消息传递模型，通过在 goroutine 之间传递数据来传递消息，而不是对数据进行加锁来实现同步访问。用于在 goroutine 之间同步和传递数据的关键数据类型叫作通道 (channel)。

### 6.1 - 并发与并行

并发 (concurrency)，并行 (parallelism)。

当有多个 goroutine 逻辑处理器时，多个逻辑处理器之间是并行，每个逻辑处理器内部的多个 goroutine 是并发。

### 6.2 - goroutine

通过设置逻辑处理器的个数观察多个 goroutine 的运行效果：

```go
runtime.GOMAXPROCS(1) // 设置一个逻辑处理器
runtime.GOMAXPROCS(runtime.NumCPU()) // 分配和 cpu 相同核数的逻辑处理器个数
```

使用 sync.WaitGroup 来等待 goroutine 的完成。WaitGroup 是计数信号量。

其余略。

### 6.3 - 竞争状态

多个 goroutine 同时修改相同的变量时，产生竞争。解决办法是加锁，细节见下一小节。

Go 提供了工具来检测代码中是否存在竞争冲突：`go build -race`。

### 6.4 - 锁住共享资源

Go 的 atomic 包提供了原子函数，sync 提供了互斥锁 mutex。

原子函数：

- atomic.AddInt64
- atomic.LoadInt64
- atomic.StoreInt64
- ...

互斥锁：

- mutex.Lock()
- mutex.Unlock()

### 6.5 - 通道

> 在 Go 语言里，你不仅可以使用原子函数和互斥锁来保证对共享资源的安全访问以及消除竞争状态，还可以使用通道，通过发送和接收需要共享的资源，在 goroutine 之间做同步。

只能使用 make 创建通道：

```go
// 无缓冲的通道
unbuffered := make(chan int)
// 有缓冲的通道
buffered := make(chan string, 10)
```

#### 6.5.1 - 无缓冲的通道

> 无缓冲的通道 (unbuffered channel) 是指在接收前没有能力保存任何值的通道。这种类型的通道要求发送 goroutine 和接收 goroutine 同时准备好，才能完成发送和接收操作。如果两个 goroutine 没有同时准备好，通道会导致先执行发送或接收操作的 goroutine 阻塞等待。这种对通道进行发送和接收的交互行为本身就是同步的。其中任意一个操作都无法离开另一个操作单独存在。

两个案例，详略。

#### 6.5.2 - 有缓冲的通道

> 有缓冲的通道 (buffered channel) 是一种在被接收前能存储一个或者多个值的通道。这种类型的通道并不强制要求 goroutine 之间必须同时完成发送和接收。通道会阻塞发送和接收动作的条件也会不同。只有在通道中没有要接收的值时，接收动作才会阻塞。只有在通道没有可用缓冲区容纳被发送的值时，发送动作才会阻塞。这导致有缓冲的通道和无缓冲的通道之间的一个很大的不同: 无缓冲的通道保证进行发送和接收的 goroutine 会在同一时间进行数据交换; 有缓冲的通道没有这种保证。

案例，略。

// TODO，回头有空把这几个例子写一遍。

## 第 7 章 - 并发模式

三个用来简化编写并发程序的包。

### 7.1 - runner

> runner 包用于展示如何使用通道来监视程序的执行时间，如果程序运行时间太长，也可以用 runner 包来终止程序。

详略，需要时再看。

### 7.2 - pool

> 这个包用于展示如何使用有缓冲的通道实现资源池，来管理可以在任意数量的 goroutine 之间共享及独立使用的资源。这种模式在需要共享一组静态资源的情况 (如共享数据库连接或者内存缓冲区) 下非常有用。

详略，需要时再看。

### 7.3 - work

> work 包的目的是展示如何使用无缓冲的通道来创建一个 goroutine 池，这些 goroutine 执行并控制一组工作，让其并发执行。

详略，需要时再看。

## 第 8 章 - 标准库

主要探讨 3 个非常有用的包：log，json 和 io。

### 8.2 - 记录日志

log 包的使用，详略，需要时再看。

### 8.3 - 编码/解码

json 包的使用，详略，需要时再看。

```go
type gResult struct {
  GsearchResultClass string `json:"GsearchResultClass"`
  UnescapeURL string `json:"unescapedUrl"`
  ...
}
```

> 你会注意到每个字段最后使用单引号声明了一个字符串。这些字符串被称作标签 (tag)，是提供每个字段的元信息的一种机制，将 JSON 文档和结构类型里的字段一一映射起来。如果不存在标签，编码和解码过程会试图以大小写无关的方式，直接使用字段的名字进行匹配。如果无法匹配，对应的结构类型里的字段就包含其零值。

### 8.4 - 输入和输出

io 包。这个包可以以流的方式高效处理数据，而不用考虑数据是什么，数据来自哪里，以及数据要发送到哪里的问题。

这个包含有 io.Writer 和 io.Reader 两个接口。

> io 包是围绕着实现了 io.Writer 和 io.Reader 接口类型的值而构建的。

详略，需要时细看。

## 第 9 章 - 测试和性能

### 9.1 - 单元测试

使用 testing 包，测试方法以 Test 开头，文件以 `_test.go` 结尾。

详略，需要时再看。

### 9.2 - 示例

Go 提供了工具允许你为代码编写示例并显示在文档中。

详略，需要时再看。

### 9.3 - 基准测试

进行 benchmark。

详略，需要时再看。
