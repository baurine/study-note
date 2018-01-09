# Go - Note 1

Note for [Go 语言教程](http://www.runoob.com/go/go-tutorial.html)

(这个教程太初级了...)

## 简介

> Go 语言被设计成一门应用于搭载 Web 服务器，存储集群或类似用途的巨型中央服务器的系统编程语言。

> 对于高性能分布式系统领域而言，Go 语言无疑比大多数其它语言有着更高的开发效率。它提供了海量并行的支持，这对于游戏服务端的开发而言是再好不过了。

Hello World Demo:

    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, World!")
    }

运行：

    $ go run hello.go
    Hello, World!

## 安装

略。下载安装包双击安装就行。

## 结构

- 包声明
- 引入包
- 函数
- 变量
- 语句 & 表达式
- 注释

第一行代码 `package main` 定义了包名，`package main` 表示一个可独立执行的程序，生个 Go 应用程序都包含一个名为 main 的包。

`import "fmt"` 表示导入 fmt 包，fmt 包实现了格式化输入输出的函数。

`func main()`，相当于 c/c++ 的 `int main()`，程序的主入口。

当标识符 (包括常量、变量、类型、函数名、结构字段等) 以一个大写字母开头时，这个对象可以被外部包所使用，称为导出，否则，只能在包内使用，对外不可见。

## 基础语法

每一行代码结尾无须 `;` (但你写上也没问题)

注释和 c/c++ 是一样的

## 数据类型

- 布尔值：true, false
- 数字类型：整型 int，浮点型 float32, float64 ...
- 字符串
- 派生类型
  - 指针类型 (Pointer)
  - 数组类型
  - 结构化类型 (Struct)
  - 函数类型
  - 切片类型
  - 接口类型 (Interface)
  - Map 类型

## 变量

一般形式，用 var 声明变量，类型放在最后面：

    var var_name var_type
    // 示例
    var age int = 10

(又是一种新的形式... 为什么非得创这种新，传统语言类型放在变量名前，Swift/TypeScript 把类型放在变量名后，并用 `:` 分开，这里也是把类型放在变量名后，但不用 `:`)

Go 支持根据字面量自动推导，从而省略类型 (和 Swift 一样)：

    var var_name = value
    // 示例
    var age = 10  // age 为 int 类型

第三种，省略 var，用 `:=` 赋值声明，变量名必须是首次出现 (但只能用于函数体中)：

    var_name := value
    // 示例
    c := 10

支持多变量声明：

    var vname1, vname2, vname3 type
    vname1, vname2, vname3 = v1, v2, v3

    var vname1, vname2, vname3 = v1, v2, v3

    vname1, vname2, vname3 := v1, v2, v3

    // 这种因式分解关键字的写法一般用于声明全局变量
    var (
        vname1 v_type1
        vname2 v_type2
    )

### 值类型与引用类型

基本数据类型，如 int / float / string，属于值类型，通过 & 操作符取得变量在内存中的地址。

其它类型一般使用引用类型。

### 注意事项

`:=` 只能在函数中使用。

在函数中声明的变量必须使用，否则编译错误 (这么严，和其它语言都不一样)，和 Swift 一样，可以用 `_` 来声明不使用的变量。

Go 支持一个函数同时返回多个值，也很常用。

## 常量

声明常量和变量一样，只不过把 var 换成 const 就行了。

### iota

iota，特殊常量，可以认为是一个可以被编译器修改的常量。

在每一个 const 关键字出现时，被重置为 0，然后再下一个 const 出现之前，每出现一次 iota，其所代表的数字会自动增加 1。

一般用来枚举值。

    func main() {
        const (
                a = iota   //0
                b          //1
                c          //2
                d = "ha"   // 独立值，iota += 1
                e          //"ha"   iota += 1
                f = 100    //iota +=1
                g          //100  iota +=1
                h = iota   //7, 恢复计数
                i          //8
        )
        fmt.Println(a,b,c,d,e,f,g,h,i)
    }

## 语言运算符

略。和 c/c++ 一样。

## 条件语句

if...else if...else / switch 和 c/c++ 都大体相同，只不过条件可以不用 `()` 括起来。

多了一个 select 语句。

> select 语句类似于 switch 语句，但是 select 会随机执行一个可运行的 case。如果没有 case 可运行，它将阻塞，直到有 case 可运行。

是不是用来实现 Go 的所谓协程作用的?

(不行，这里说的太晴蜓点水了，选跳过，换一篇教程来理解这部分内容，这可以算是 Go 的精化了呀)

## 循环

Go 的循环统一用 for，没有 while。

第一种，类似 c/c++ 的 for，只不过不用 `()`：

    for init; condition; post {}

第二种，替代 while：

    for condition {}

第三种，替代 for(;;) 或 while(true)：

    for {}

for 循环的 range 格式可以对 slice, map, array, string 等进行迭代循环：

    for key, value := range iterator_obj {}

示例：

    func main() {
        var b int = 15
        var a int

        numbers := [6]int{1, 2, 3, 5}

        /* for 循环 */
        for a := 0; a < 10; a++ {
            fmt.Printf("a 的值为: %d\n", a)
        }

        for a < b {
            a++
            fmt.Printf("a 的值为: %d\n", a)
            }

        for i,x := range numbers {
            fmt.Printf("第 %d 位 x 的值 = %d\n", i,x)
        }
    }

## 函数

函数定义 (和 Swift 类似，返回类型放在声明最后)：

    func function_name( [parameter list] ) [return_types] {}

示例：

    func swap(x, y string) (string, string) {
        return y, x
    }

    func main() {
    a, b := swap("Mahesh", "Kumar")
        fmt.Println(a, b)
    }

参数传递：值传递和引用传递，和 c/c++ 一样。

在 Go 中，可以像其它动态语言一样，把函数赋值给一个变量。

    /* 声明函数变量 */
    getSquareRoot := func(x float64) float64 {
        return math.Sqrt(x)
    }

    /* 使用函数 */
    fmt.Println(getSquareRoot(9))

Go 支持在函数中定义函数，并返回函数，支持闭包。

    func getSequence() func() int {
        i:=0
        return func() int {
            i+=1
            return i
        }
    }

    /* nextNumber 为一个函数，函数 i 为 0 */
    nextNumber := getSequence()

### 成员方法

    type Circle struct {
        radius float64
    }

    // 该 method 属于 Circle 类型对象中的方法
    func (c Circle) getArea() float64 {
        // c.radius 即为 Circle 类型对象中的属性
        return 3.14 * c.radius * c.radius
    }

## 变量作用域

函数内定义的为局部变量，函数外定义的为全局变量。

## 数组

声明

    var variable_name [SIZE] variable_type
    // 示例
    var balance [10] float32

声明并初始化

    var balance = [5]int{1, 2, 3, 4, 5}
    var balance = [...]int{1, 2, 3}

(奇怪，为什么要采用这种奇怪的语法，balance = [1, 2, 3] 这种不好吗?)

二维数组，略。

数组作为函数参数，略，和 c/c++ 一样。

## 指针

略。和 c/c++ 一样。

定义的时候更明确了，`*` 号必须和类型放在一起。

    var var_name *var_type

空指针的值为 nil。

## 结构体 (Struct)

定义

    type struct_variable_type struct {
        member definition;
        member definition;
        ...
        member definition;
    }

使用并初始化

    variable_name := structure_variable_type {value1, value2...valuen}

成员，用 `.` 号，不论是值类型还是指针类型。

## 切片 (slice)

切片是 Go 语言对数组的抽象。

看下来以后，可以理解成数据结构中的可随机访问的数组，C++ STL 中的 Vector。

切片的声明，其实是一个不需要指定大小的数组：

    var identifier []type

或使用 make() 来创建切片：

    var slice1 []type = make([]type, len)
    // or
    slice1 := make([]type, len)

直接初始化：

    s := []int{1, 2, 3}
    // 注意，数组的初始化，[] 中是有值的，要么为数值，要么为 "..."

通过别的切片来创建新的切片：

    s := arr[:]
    s := arr[startIndex:endIndex]

切片的使用貌似和 Python 相似。

len() 函数获取切片实际长度，cap() 函数获取切片的容量。

append() 追加元素，copy() 拷贝切片。

## 范围 (range)

> Go 语言中 range 关键字用于 for 循环中迭代数组 (array)、切片 (slice)、通道 (channel) 或集合 (map) 的元素。在数组和切片中它返回元素的索引值，在集合中返回 key-value 对的 key 值。

    nums := []int{2, 3, 4}

    for i, num := range nums {
        if num == 3 {
            fmt.Println("index:", i)
        }
    }

## 集合 (map)

无序的 key-value 对的集合。

声明 (语法好奇芭)

    /* 声明变量，默认 map 是 nil */
    var map_variable map[key_data_type]value_data_type

    /* 使用 make 函数 */
    map_variable := make(map[key_data_type]value_data_type)

使用 range 对集合进行迭代时，只返回一个值 - key。

    /* 使用 key 输出 map 值 */
    for country := range countryCapitalMap {
        fmt.Println("Capital of",country,"is",countryCapitalMap[country])
    }

delete() 函数用于删除 key。

## 递归

略。

## 类型转换

和 c 一样

    type_name(expression)

## 接口 (interface)

和 TypeScript 的理论一样，不用显式地实现此接口，只要对象中有些方法，那就是这个接口类型，鸭子模型。

示例：

    type Phone interface {
        call()
    }

    type NokiaPhone struct {
    }

    func (nokiaPhone NokiaPhone) call() {
        fmt.Println("I am Nokia, I can call you!")
    }

    func main() {
        var phone Phone

        phone = new(NokiaPhone)
        phone.call()
    }

使用 new() 方法来生成一个对象。

## 错误处理

Go 语言通过内置的错误接口提供了非常简单的错误处理机制。

error 类型是一个接口类型，这是它的定义：

    type error interface {
        Error() string
    }

我们可以在编码中通过实现 error 接口类型来生成错误信息。

函数通常在最后的返回值中返回错误信息。使用 errors.New() 可返回一个错误信息。

    func Sqrt(f float64) (float64, error) {
        if f < 0 {
            return 0, errors.New("math: square root of negative number")
        }
        // 实现
    }

判断是否出错：

    result, err := Sqrt(-1)

    if err != nil {
        fmt.Println(err)
    }

(确实烦啊)

## 开发工具

略。

Done!

但是，Go 最重要的 Goroutine，居然完全没讲到...

而且，看到 Go 没有 class，只有 interface 和 struct。这里也没有讲到继承，所以应该是没有喽，因为 c 也没有嘛。
