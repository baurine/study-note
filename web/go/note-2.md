# Go - Note 2

Note for [Go 指南](https://tour.go-zh.org/welcome/1)

作为 Note 1 的补充。

目录：

- 基础
  - 包、函数和变量 (居然函数在变量前...)
  - 流程控制 - for, if, else, switch 和 defer
  - 复杂类型 - struct, slice, map
- 方法和接口
- 并发

其实参考：

- [Go Web 编程](https://github.com/astaxie/build-web-application-with-golang) [GitBook](https://wizardforcel.gitbooks.io/build-web-application-with-golang/content/)
- [Go 入门指南](https://github.com/Unknwon/the-way-to-go_ZH_CN) [极客学院](http://wiki.jikexueyuan.com/project/the-way-to-go/)

## 基础

### 包、函数和变量

包，略。

导入，导出多个包时，可以用 () 用法：

    import (
      "fmt"
      "math"
    )

导出名，Go 不用像 JavaScript 使用显式的 export 进行导出，首字母大写的方法或变量是自动导出的。(所以，导出的方法名是双驼峰写法，不导出的是单驼峰，有点奇芭啊...)

函数。函数的定义，为什么要把类型放在函数名名后面 (定义变量时也是把类型入在变量名后面)，是为了解决 C 中把类型放在变量前导致的混乱情况 (深有体会)。

当两个或多个连续的函数命名参数是同一类型，则除了最后一个类型之外，其他都可以省略。比如 `x int, y int` 可以简写为 `x, y int`。

(Go 尽量不让你写多余无用的代码。)

函数的返回值。Go 的返回值可以被命名，没有参数的 return 语句返回各个返回变量的当前值，这种用法称为 "裸" 返回，应当仅用在短函数中，在长函数中影响代码可读性。

    func split(sum int) (x, y int) {
      x = sum * 4 / 9
      y = sum - x
      return  // 等同于 return x,y
    }

变量，变量声明。

短声明变量，用 `:=`，如 `k := 3`。函数外的每个语句必须以关键字开始 (如 var, const, func ...)，因此 `:=` 不能用于函数外。(原来如此!)

基本类型，略。

零值。变量在定义时没有明确的初始化时会赋值为零值。

- 数值类型零值为 0
- 布尔型为 false
- 字符串为 ""
- (对象型应该为 nil 吧)

类型转换，略。与 C 不同的是，Go 没有隐式类型转换，所以转换必须显式声明。

类型推导，略。

常量，略。常量不能用 `:=` 声明，`:=` 只用于声明变量。

### 流程控制

**for**

Go 只有一种循环结构 - for 循环。

不像 C，Java，或者 Javascript 等其他语言，for 语句的三个组成部分并不需要用括号括起来，但循环体必须用 {} 括起来。(Ruby/Python 也不用，新的语言也不用了吧，比如 Swift)

循环初始化语句和后置语句都是可选的。

    sum := 1
    for ; sum < 1000; {
      sum += sum
    }

把前后的分号都省略，就变成了 while 循环。

    sum := 1
    for sum < 1000 {
      sum += sum
    }

省略循环条件，变成死循环 (的确是最简洁的写法了)

    for {
    }

**if**

if 语句也不需要用 () 将条件括起来。

if 语句还可以在条件之前执行一个简单语句，变量作用域仅在 if 范围内。

    func pow(x, n, lim float64) float64 {
      if v := math.Pow(x, n); v < lim {
        return v
      } else {
        fmt.Printf("%g >= %g\n", v, lim)
      }
      // 这里开始就不能使用 v 了
      return lim
    }

**switch**

switch，不像 C 需要 break，也可以像 if 一样在条件之前执行一件简单语句。

    func main() {
      fmt.Print("Go runs on ")
      switch os := runtime.GOOS; os {
      case "darwin":
        fmt.Println("OS X.")
      case "linux":
        fmt.Println("Linux.")
      default:
        // freebsd, openbsd,
        // plan9, windows...
        fmt.Printf("%s.", os)
      }
    }

(Go 的 switch 可以把常量放在 switch 后，变量放在 case 里，神奇)

    func main() {
      fmt.Println("When's Saturday?")
      today := time.Now().Weekday()
      switch time.Saturday {
      case today + 0:
        fmt.Println("Today.")
      case today + 1:
        fmt.Println("Tomorrow.")
      case today + 2:
        fmt.Println("In two days.")
      default:
        fmt.Println("Too far away.")
      }
    }

没有条件的 switch，等同于 switch true，可以用来替代 if...else if...else 链。(cool!)

    func main() {
      t := time.Now()
      switch {
      case t.Hour() < 12:
        fmt.Println("Good morning!")
      case t.Hour() < 17:
        fmt.Println("Good afternoon.")
      default:
        fmt.Println("Good evening.")
      }
    }

**defer**

(新内容)

> defer 语句会延迟函数的执行直到上层函数返回。

> 延迟调用的参数会立刻生成，但是在上层函数返回前函数都不会被调用。

(明白)

    func main() {
      defer fmt.Println("world")
      fmt.Println("hello")
    }
    // 运行结果
    hello
    world

延迟的函数调用被压入一个栈中。当函数返回时， 会按照后进先出的顺序调用被延迟的函数调用。(所以和 JavaScript 上使用 setTimeout 的原理是完全不一样的)

    func main() {
      fmt.Println("counting")

      for i := 0; i < 10; i++ {
        defer fmt.Println(i)
      }

      fmt.Println("done")
    }
    // 运行结果
    counting
    done
    9
    8
    ...

### 复杂类型

**指针**

Go 是除 C/C++ 外，少数还能直接使用指针的语言。

类型 `*T` 是指向 T 类型值的指针，其零值为 nil。定义时，`*` 号写在类型前面，避免了 C 中混乱的写法。

    var p *int

& 操作符取得值的内存地址，并赋值给指针类型变量。

    i := 42
    p = &i

但和 C 不一样的是，Go 没有指针运算，不能对指针进行加或减。

**结构体 (struct)**

一个结构体就是一个字段的集合。使用点号 `.` 访问成员，即使是结构体指针，也用 `.`，Go 中没有 `->` 操作符。

> 如果我们有一个指向结构体的指针 p ，那么可以通过 (*p).X 来访问其字段 X。不过这么写太啰嗦了，所以语言也允许我们使用隐式间接引用，直接写 p.X 就可以。

结构体的初始化：

    type Vertex struct {
      X, Y int
    }
    var (
      v1 = Vertex{1, 2}  // has type Vertex
      v2 = Vertex{X: 1}  // Y:0 is implicit
      v3 = Vertex{}      // X:0 and Y:0
      p  = &Vertex{1, 2} // has type *Vertex
    )

**数组**

> 类型 `[n]T` 表示拥有 n 个 T 类型的值的数组。

> 数组的长度是其类型的一部分，因此数组不能改变大小。

**切片**

> 每个数组的大小都是固定的。 而切片则为数组元素提供动态大小的、灵活的视角。在实践中，切片比数组更常用。

> 类型 `[]T` 表示一个元素类型为 T 的切片。

> 切片通过两个下标来界定，即一个上界和一个下界，二者以冒号分隔：

>     a[low : high]

> 它会选择一个半开区间，包括第一个元素，但排除最后一个元素。

> 切片就像数组的引用：切片并不存储任何数据， 它只是描述了底层数组中的一段。更改切片的元素会修改其底层数组中对应的元素。与它共享底层数组的切片都会观测到这些修改。

切片的文法类似于没有长度的数组文法。

    // 数组文法
    [3]bool{true, true, false}

    // 创建相同的数组，并构建一个引用它的切片
    []bool{true, true, false}

切片的默认行为，下界默认值为 0，上界默认值为该切片的长度。

切片的长度和容量，略。

切片的零值为 nil，表示长度和容量为 0 且没有底层数组。

使用 make() 创建切片。`make([]type, len, cap)`

使用 append() 向切片追加元素。

**Range**

for 循环的 range 形式可遍历切片或映射。略。

**Map**

使用双赋值检测某个键是否存在：

    elem, ok = m[key]

**函数值**

函数也是值。它们可以像其它值一样传递。函数值可以用作函数的参数或返回值。

闭包，略。

## 方法和接口

### 方法

Go 没有类，不过可以为结构体类型定义方法。

方法的本质：就是一类带特殊**接收者**参数的函数。

方法接收者在它自己的参数列表内，位于 func 关键字和方法名之间。

    type Vertex struct {
      X, Y float64
    }

    func (v Vertex) Abs() float64 {
      return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func main() {
      v := Vertex{3, 4}
      fmt.Println(v.Abs())
    }

记住：方法只是个带接收者参数的函数，上面的写法只不过把接收者参数挪了个位置，实际它和下面这种写法没有功能上的区别：

    func Abs(v Vertext) float64 {
      return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    // 用法的区别，实际就是位置挪了一下
    v.Abs()
    Abs(v)

(貌似 Go 里面，称结构体的函数为方法 - method，一般的函数就叫函数 - function)

可以为内置类型设置别名，并为别名类型添加方法。(cool! 相当于为内置类型扩展方法)

    func (f MyFloat) Abs() float64 {
      if f < 0 {
        return float64(-f)
      }
      return float64(f)
    }

**指针接收者**

(这种使用方式更常用，并从这可以看出，这种方法，还真不是其它语言中的成员方法，它就是个普通函数差不多的东西)

可以为指针接收者声明方法，意味着对于类型 T，接收者的类型可以用 `*T`。

如果方法的接收者类型为 T，那么调用此方法时，T 会拷贝一个副本。(天啊，为什么要这种设计，这个确实和成员方法差太远了)

但如果方法的接收者类型为 `*T`，那么调用此方法时，T 指向原来的对象 (但其实 Go 没有对象这种说法吧，顶多是结构体对象)。

    func (v *Vertex) Scale(f float64) {
      v.X = v.X * f
      v.Y = v.Y * f
    }

    func main() {
      v := Vertex{3, 4}
      v.Scale(10)
      p := &v
      p.Scale(10)
      fmt.Println(v.Abs())
    }

从上面可以看出，当接收者为指针类型的方法被调用时，接收者既能为值，又可以为指针。实际调用 `v.Scale(10)` 时，内部是解释成 `(&v).Scale(10)`。

以值为接收者的方法被调用时，接收者也一样，可以为值，或指针。

    var v Vertex
    fmt.Println(v.Abs()) // OK
    p := &v
    fmt.Println(p.Abs()) // OK

实际 `p.Abs()` 调用时，内部会解释成 `(*p).Abs()`。

使用指针接收者的原因有二：

- 首先，方法能够修改其接收者指向的值。
- 其次，这样可以避免在每次调用方法时复制该值。若值的类型为大型结构体时，这样做会更加高效。

### 接口

> 接口类型是由一组方法签名定义的集合。

> 接口类型的值可以保存任何实现了这些方法的值。

> 类型通过实现一个接口的所有方法来实现该接口。 既然无需专门显式声明，也就没有 "implements" 关键字。

在内部，接口值可以看做包含值和具体类型的元组：`(value, type)` (不是很明白)

即便接口内的具体值为 nil，方法仍然会被 nil 接收者调用。

在一些语言中，这会触发一个空指针异常，但在 Go 中通常会写一些方法来优雅地处理它 (如本例中的 M 方法)。

    func (t *T) M() {
      if t == nil {
        fmt.Println("<nil>")
        return
      }
      fmt.Println(t.S)
    }

(这倒更符合 Objective-C 和 Ruby 所谓的消息发送的理念了)

**空接口**

指定了零个方法的接口值称为空接口：`interface{}`。空接口可保存任何类型的值 (因为每个类型都至少实现了零方法。)

空接口被用来处理未知类型的值。 例如，fmt.Print 可接受类型为 interface{} 的任意数量的参数。

**类型断言**

    t := i.(T)

为了避免类型断言失败导致 panic 异常，可以用另一个用法：

    t, ok := i.(T)

如果 i 是 T 类型，那么 t 将得到 i 为 T 类型的值，ok 为 true，否则 ok 为 false。

    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s)

    s, ok := i.(string)
    fmt.Println(s, ok)

**类型选择**

    switch v := i.(type) {
    case T:
        // v 的类型为 T
    case S:
        // v 的类型为 S
    default:
        // 没有匹配，v 与 i 的类型相同
    }

注意，`i.(type)`，括号中不再是 T，而是关键字 type。

**Stringer**

fmt 包中定义的 Stringer 是最普遍的接口之一。

    type Stringer interface {
        String() string
    }

Stringer 是一个可以用字符串描述自己的类型。fmt 包 (还有很多包) 都通过此接口来打印值。

**错误**

略，看 Note 1。

**Reader**

略。

其余略。

## 并发

goroutine 与 channel。

(JavaScript 利用 generator 也能实现类似的功能，Redux-Saga 就是这样一个库)

goroutine 是 Go 运行时管理的轻量级线程。

`go f(x,y,z)` 会启动一个新的 goroutine 并执行 `f(x,y,z)`，但记住，x,y,z 的求值是发现在原来线程中 (理解)。

    func say(s string) {
      for i := 0; i < 5; i++ {
        time.Sleep(100 * time.Millisecond)
        fmt.Println(s)
      }
    }

    func main() {
      go say("world")
      say("hello")
    }

**信道 (channel)**

线程之间通过 channel 来通信。

信道是带有类型的管道，可以用 `<-` 管道操作符来发送或接收值。

    ch <- v     // 接 v 发送至信道 ch
    v := <- ch  // 从 ch 接收值并赋给 v

(箭头是数据流的方向，注意，并没有 `->` 符号)

信道在使用前必须创建：

    ch := make(chan int)

默认情况下，发送和接收操作在另一端准备好之前都会阻塞。这使得 Go 程可以在没有显式的锁或竞态变量的情况下进行同步。

    func sum(s []int, c chan int) {
      sum := 0
      for _, v := range s {
        sum += v
      }
      c <- sum // 将和送入 c
    }

    func main() {
      s := []int{7, 2, 8, -9, 4, 0}

      c := make(chan int)
      go sum(s[:len(s)/2], c)
      go sum(s[len(s)/2:], c)
      x, y := <-c, <-c // 从 c 中接收

      fmt.Println(x, y, x+y)
    }

理解了这个示例，当程序执行到 `x, y := <-c, <-c` 时，会阻塞，当第一个 goroutine 执行完时，x 取到值，但 y 还没有取到值，继续阻塞，当第二个 goroutine 执行完时，y 取到值，阻塞结束，打印结果。

**带缓冲的信道**

默认信道没有缓冲，长度为 1，可以设置其长度为大于 1 的值，使用有缓冲。

    ch := make(chan int, 100)

仅当信道的缓冲区填满后，向其发送数据时才会阻塞。当缓冲区为空时，接受方会阻塞。

**range 和 close**

发送者没有数据可发时，可以通过调用 close(ch) 显式地关闭这个信道，注意，接收者不能关闭。

(咦，这个信道是单向还是双向?)

接收者可以通过为接收表达式分配第二个参数来测试信道是否已关闭：`v, ok := <- ch`，如果关闭，则 ok 为 false。

`for i:= range c` 会不断从信道接收值，直到它被关闭。

    func fibonacci(n int, c chan int) {
      x, y := 0, 1
      for i := 0; i < n; i++ {
        c <- x
        x, y = y, x+y
      }
      close(c)
    }

    func main() {
      c := make(chan int, 10)
      go fibonacci(cap(c), c)
      for i := range c {
        fmt.Println(i)
      }
    }

**select 语句**

(模拟多路复用)

select 语句使一个 Go 程可以等待多个通信操作。

select 会阻塞到某个分支可以继续执行为止，这时就会执行该分支。当多个分支都准备好时会随机选择一个执行。

当 select 中的其它分支都没有准备好时，default 分支就会执行。

例子略。

**sync.Mutex**

互斥锁，两个方法 Lock() 和 Unlokc()。详略。
