# iOS Boxue - Note 2

## Swift 2.2

Time: 2016/06/29

### 第一课：Swift 初上手编程

#### 第一小节：变量和常量

定义变量用 var，常量用 let

不需要指定类型，swift 会自动推断类型。但如果变量没有初始值 (即没有字面值)，swift 将无法自动推断类型，那么需要手动指定类型。

    var name: Type

基本数据类型： Integer, Double, Bool, String

Tuple：

    var me = ("Mars", 11, "11@boxue.io")

可以在注释中使用 markdown 语法

#### 第二小节：使用 Markdown 编写代码注释

略。大致了解即可。

做为伴随 Swift 3 发布的 API 设计指南中的要求，使用 Swift Markdown 为代码编写注释已经进一步从一个道义提升为了一种行为准则。一方面，Markdown 对开发者来说，足够熟悉，学习难度并不高；另一方面，Xcode 可以在 Playground 和代码提示中，对 Markdown 注释进行漂亮的渲染，让代码在开发者之间交流起来更加容易。

#### 第三小节：整数和浮点数

略。

aVar.dynamicType 可以查看一个变量的类型

类型转化：

    aInt = 3
    Double(int)

#### 第四节：字符、Unicode 和字符串 (重要!)

可以看到，同一个汉字 "泊"，在不同的字符集里的编码值是不一样的。这使得计算机在多语言环境里，处理文本变得非常复杂。因此，国际上开始推行一种叫做 Unicode 的标准，它使用统一的编码规则对几乎我们使用的所有文字、字符和符号进行了统一编码。而 Swift 中的 String 对象，就是完全基于 Unicode 构建的。

    // String init
    let emptyString = ""
    let emptyString = String()

    var swift = "Swift is fun."

和 Objective-C 不同，Swift 中的 String 是一个值类型，当我们把一个 String 对象赋值给其它变量的时候，Swift 会完整复制字符串的内容，而不仅仅是复制一个指向原有字符串的引用。

当然，Swift 对字符串的拷贝操作进行了优化，我们只有在后面的代码真正修改字符串的时候，拷贝操作才真的发生。

String 中的每一个字符都是一个 Character 对象。

我们可以通过在字符串中插入 `\(expression)`，把 expression 的值直接插入字符串。在 Swift 里，这就叫做 string interpolation。

**Unicode Scalar**

Unicode 使用一个 21-bit 的整数，几乎对我们使用的所有字符采用统一的格式进行了编码。

事实上，每个 unicode 字符都会有一个对应的 name 和 number，我们管这个 number，就叫做 unicode scalar。

    // Unicode Scalar
    let blingHeart = "\u{1F496}"
    blingHeart.dynamicType // String.Type

> 并不是 21-bit 所有整数空间都被分配了字符，unicode scalar 并不包含 U+D800-U+DFFF 之间的值，它们叫做 unicode surrogate pair code points。

**Extended Grapheme Clusters**

在 Swift 里，每一个 Character 对象都是一个 Extended Grapheme Cluster。这是什么呢？我们用字符 é 来举个例子。同一个字符，可以有多种不同的表达方式：

- 一种是字符 é 的 unicode scalar：U+00E9
- 一种是字符 e (U+0065) +  ́ (U+0301)

这种多个 unicode scalar 的组合，就叫做 extended grapheme cluster。在 Swift 里，一个 Character 就是一个 extended grapheme cluster。了解了这两个基本概念之后，我们就可以继续学习 String 的用法了。

(所以，我的理解是，有些 unicode 字符具有聚合效应，当它和某些字符放一起时，会聚合成一个新的字符)

> 这就意味同样的字符串，有可能是由不同的字符组合构成的，也因此会占用不同的内存空间。

**String index**

由于 Swift 中的一个 "字符" 有可能由多个 unicode scalar 构成，因此，我们不能够像 C 语言一样使用类似 cafe[0], cafe[1], cafe[2], cafe[3] 这样的形式索引字符串。Swift 提供了一个专门用于索引字符串的类型：String.Index，我们可以通过 String 对象的方法，来获取这个位置。

String 类型提供了两个默认的索引：

- startIndex - 字符串的起始位置
- endIndex - 字符串最后一个字符的下一个位置

以及向前、向后移动 String.Index 的三个方法：

- successor() - 返回当前 Index 的后一个位置
- predecessor() - 返回当前 Index 的前一个位置
- advancedBy(n) - 任意移动 Index n 个位置 (正值向后，负值向前)

基于特定的 StringIndex，我们使用：

- insert: 在特定的位置插入字符
- insertContentsOf: 在特定的位置插入字符串 (注意使用 characters 属性)
- removeAtIndex: 在特定位置删除字符
- removeRange: 删除特定 String.Index 范围指定的字符

**Compare String**

Swift 有两种比较字符串的方式，一种是比较相等，一种是比较是否包含前缀或后缀，先来看第一种。

由于 Extended Grapheme Clusters 的关系，"字面值" 相同的字符串，计算机的表现形式并不一定相同。因此，Swift 采取的比较策略是："如果两个字符串有同样的意义，并且有同样的外观，那么它们是相等的"。

因此，在之前我们定义过的 cafe 和 cafe1，虽然他们由不同的 Extended Grapheme Clusters 构成，但是它们是两个相等的字符串。

**Unicode and UTF-n encoding**

当我们需要把 Unicode 字符保存在存储介质的时候，由于有些字符是可以单字节存储的 (例如：A 0x41)，有些字符是不能够通过单字节存储的 (例如：💖 0x1F496)，因此，我们需要对 unicode scalar 按照一定的长度进行编码，这种编码方式，就是我们常听到的 UTF-8，UTF-16 和 UTF-32。

(所以，UTF-8, UTF-16，UTF-32 是用于传输的编码，是用于持久化的编码，它们不是在内存里表示的编码，它们在内存中的表示都是统一的 unicode 编码)

由于 Swift 中的字符是完全基于 unicode scalar 构建的，因此，我们可以通过字符串对象的 utf8 / utf16 / unicodeScalars 属性访问它的各种编码方式。

#### 第五节：使用 Tuple 打包数据

(和 es6 中的语法很相似)

    // 定义
    let success = (200, "HTTP OK")
    let me = (name: "mars", no: 11, email: "11@boxue.io")
    var redirect: (Int, String) = (302, "temporary redirect")

    // 使用
    success.0
    success.1

    me.name
    me.no
    me.email

    // tuple decompositon
    var (statusCode, statusMsg) = success
    let (_, _, email) = me

#### 第六节：常用操作符

略。

和其它语言不一样的地方：

Nil Coalescing Operator - `??`

    // opt != nil ? opt! : b
    // 如果 opt 是一个 optional，当其不为 nil 时，自动解引用，否则，使用 "默认值" b。

    var userInput: String? = "A user input"
    let value = userInput ?? "A default input”


闭区间和半开半闭区间

    for index in 1...5 {
      print(index)
    }

    for index in 1..<5 {
      print(index)
    }

### 第二课：集合类型打包数据

Collection 是一种存放数据的 "容器"。Swift 默认提供了三种 Collection - Array、Set、Dictionary，它们有着类似的接口，却有着各自不同的用处。

#### 第一小节：有序集合

Array

定义，略。使用，略。

遍历

    for number in fiveInts {
      print(number)
    }

    // 这个和 python 中的语法很相似
    fiveInts.enumerate()
    for (index, value) in fiveInts.enumerate() {
      print("Index: \(index) Value: \(value)")
    }

(学习一门新的语言，初步学的时候，并不需要完全掌握每一个细节，只需要大致知道如何使用及其原理即可，等到真正使用时，再回来看细节)

#### 第二小节：无序集合

- Set
- Dictionary

**Set**

定义，初始化，遍历... 集合之间的运算

Swift 里要求，存放在 Set 中的元素，一定是 hashable 的。我们可以通过读取一个对象的 hashValue 属性来得到它的 hash value。

(我好像突然明白了 java 中的 hashCode 的作用了，是不是一样的作用呢)

定义

    // An empty Character set
    let emptySet = Set<Character>()
    let vowel: Set<Character> = ["a", "e", "i", "o", "u"]
    var evenSet: Set = [2, 4, 6, 8, 10]

这里要说明的是，当我们明确指定 Set "字面值" 的时候，由于形式上，它和 Array 是类似的，因此我们不能够依靠编译器推导出完整的 Set<Character> 类型，而必须使用 Type annotation 明确指定变量的类型 (例如上面 vowel 的定义)，但是，我们可以退一步，使用 Type inference 推导出 Set 中元素的类型 (例如：evenSet)。

**Dictionary**

顾名思义，Dictionary 是一个帮助我们用某种类型的值 (Key) 查找另外一种类型的值 (value) 的数据结构。同样，用作 key 的类型，也是要求 hashable 的。

初始化

    var int2Str = [Int: String]()

keys 属性，得到所有 key 组成的 array。

### 第三课：为代码的执行做个决定 (控制流程，顺序，循环，条件判断)

我们经常需要程序针对不同的情况执行不同的代码，在编程语言里，这叫做 Control Flow。

Swift 提供了和 Objective-C 类似的 Control Flow 语句，例如: if, while, for, switch 等。除了 "传统" 用法之外，Swift 还对它们进行了扩展和改进，并加入了 guard, for-in 等新的 Control Flow 语句。

(所以，重点看看 guard, for-in 就行了)

#### 第一小节：几乎所有语言都有的条件判断和循环

(html 和 css 没有，xml 也没有，json 也没有，所以它们不能叫语言)

for

    for _ in 1...100 {}

> 不要在 Swift 中使用 C 语言风格的 for 循环，它将在未来某个 Swift 版本中被替换掉。

while

    var i = 0
    while i < 10 {
      print(i)
      i += 1
    }

    // do ... while
    var n = 0
    repeat {
      print(vowel[n])
      n += 1
    } while n < 5

if...else...

#### 第二小节：经过改良的 switch

Swift 的 switch...case 与其它语言差别较大，而且更方便强大。

1. 不需要 break
2. 区配多个值时，使用 `case A,B,C: xxx;` 而不是 `case A: case B: xxx;` 这种语法
3. 可以匹配区间
4. 使用 tuple 同时匹配多个值
5. value binding (let where)

示例：

    switch point {
      case (let x, 0):
        print("with the X value of: \(x)")
      case (0, let y):
        print("with the Y value of: \(y)")
      case (let x, let y):
        print("X: \(x), Y: \(y)")
    }

    switch point {
      case let (x, y) where x == y:
        print("(\(point.0, point.1) is on y = x)")
      case let (x, y) where x == -y:
        print("(\(point.0, point.1) is on y = -x)")
      case let (x, y):
        print("(\(point.0, point.1))");
    }

### 第四课：使用 func 和 closure 加工数据

Function 和 Closure 都是一种自包含的代码块，用于封装特定的功能。在 Swift 里，它们作为 first-class type，都具有灵活的表现形式和一致的语法结构。

其中，最简单的 Closure 可以只是一个 "<"，而最复杂的函数可以包含：local & external 参数名、in-out 参数、默认参数值以及返回值，你甚至还可以在一个 Function 内嵌的作用域里，定义其它的 Function。

#### 第一小节：多种多样的函数参数

和其它语言相同的就省略不写了，只写特别之处。

使用关键字 func 定义函数 (和 js 相似，js 需要用 function 关键字，而 java 和大部分语言都不需要关键字)

swift 的函数调用时，除了第一个参数外，其余参数要指定参数的 key (好奇怪，是从 Objective-C 沿袭来的啦)，比如：

    func multipleOf(multiplier: Int, andValue: Int) {
      print("\(multiplier) * \(andValue) = \(multiplier * andValue)")
    }

    multipleOf(5, andValue: 10)

按照 Apple 的说法，这样可以提高代码的可读性，我们可以管上面的调用读作：print multipleOf 5 andValue 10。

关于参数的名字：`OuterName InnerName: Type`

    func createTable(rowNumber row: Int, colNumer column: Int) {
      print("Table: \(row) x \(column)")
    }

1. 默认情况下，outer name 和 inner name 相同
2. 默认情况下，第一个参数没有 outer name，所以传参数时候，第一个参数不需要指定参数名，但也可以强行指定，指定以后传参时就要加上了。

参数的默认值：和 c++ 一样。

可变长参数列表

    func arraySum(number: Double...) {
      // number: [Double]
      var sum: Double = 0.0

      for i in number {
        sum += i
      }

      print("sum: \(sum)")
    }

在函数内部，可变长参数列表是一个数组，可以通过遍历获取所有值。一个函数只有一个可变长度参数。

常量和变量参数

如果我们需要在函数内部修改参数，Swift 默认不会允许我们这么干。如果要想修改参数，显式给参数加上 inout 关键字。(和 C# 类似)

但使用了 inout 传参后，实参必须加上 & 表示传递一个变量的引用。(...Fuck，和 C++ 又一样了，这一点上和 java 完全不一样，C# 是不是要用一个 ref 关键字?)

#### 第二小节：函数的返回值和类型

    func funcName(param list) -> RetType {}

前方要高能了，终于到了 swift 与其它静态语言与众不同的地方了，函数作为一级公民的使用！

swift 与 js 一样，函数是一级公民，因此可以方便地将函数作为参数进行传递。

函数类型，函数类型变量：

在了解了函数的参数和返回值之后，我们就可以理解函数类型本身了。在 Swift 里，函数是 first class type。它的类型由函数的参数和返回值共同决定。例如，multipleOf 的类型，就是 "一个接受两个 Int 参数并返回一个 Int 的函数"。

    multipleOf (Int, Int) -> Int
    tableInfo () -> (Int, Int)
    string2Int (String) -> Int?

我们可以直接把函数名做为值，赋值给变量，type inference 就会自动推到变量的类型为函数的类型，当然，我们也可以使用 type annotation 来明确指定变量的类型。

    var f1: (Int, Int) -> Int = multipleOf
    var f2 = tableInfo
    var f3 = string2Int

Function type as parameter and return value

除了定义变量之外，函数类型还可以用于定义函数参数和返回值。(说起到将函数作为参数使用这一点， c 比 c++/java 都方便啊)

Nested function

对于之前的 whichOne 例子，increment 和 decrement 都只被 whichOne 调用，属于 whichOne 的 "内部功能"，因此我们可以把它们写在 whichOne 的内部，我们还使用了 typealias。我们可以为复杂的类型描述定义一个符合代码语义的别名，提高代码的可读性。

    typealias op = (Int) -> Int

    func whichOne(n: Bool) -> op {
      func increment(n: Int) -> Int {
        return n + 1
      }

      func decrement(n: Int) -> Int {
        return n - 1
      }

      return n ? increment : decrement
    }

#### 第三小节：Closure - 会 "抓" 变量的匿名函数

而 Function 和 Closure 的关系，也是如此，Closure 就是一个没有名字的函数。当然，Closure 也有自己的特性，它可以捕获变量 (Capture variable)，我们在后面会看到这个用法。

定义

    { ( param list ) -> RetType in
      /* Closure body */
    }

closure 的各种简化写法

    addClosure = { a, b in return a + b }
    addClosure = { a, b in a + b }
    addClosure = { $0 + $1 }

事实上，把 closure 赋值给变量，之后直接调用它并不是 closure 真正的用武之地，通过 closure 简化函数类型参数，才是大部分时候我们要面对的场景。(哦，看了例子后明白，closure 主要在传参的时候使用)

    func execute(a: Int,
      _ b: Int,
      operation: (Int, Int) -> Int) -> Int {
      return operation(a, b)
    }

    execute(1, 2, operation: { (a: Int, _ b: Int) -> Int in 
      return a + b
    })

    // By type inference
    execute(1, 2, operation: { a, b in return a + b })

    // No return
    execute(1, 2, operation: { a, b in a + b })

    // Param placeholder
    execute(1, 2, operation: { $0 + $1 })

这里要特别说明的是，如果函数类型参数是函数的最后一个参数，我们可以把 closure 写在函数调用的外面：这样的形式在 Swift 里，叫做 Trailing Closures。

    // By type inference
    execute(1, 2) { $0 + $1 }

Capturing Values

相比 Function，closure 当然不仅仅是写起来更简单而已。前面我们也提到过，closure 可以捕获变量 (capturing values)。为了搞清楚这个问题，我们要从作用域 (Scope) 这个话题说起。(这个是闭包的特性，和 js 一样)

### 第五课：Optional

对各种值为 "空" 的情况处理不当，几乎是所有 Buuuuuuug 的来源。在其它编程语言里，空值的表达方式多种多样，"" / nil / NULL / 0 / nullptr 都是我们似曾相识的表达空值的方法。而当我们访问一个变量时，我们有太多情况无法意识到一个变量有可能为空，进而最终在程序中埋藏了一个个闪退的隐患。因此，Swift 里，明确区分了 "变量" 和 "值有可能为空的变量" (Optionals) 这两种情况，以时刻警告你："哦，它的值有可能为空，我应该谨慎处理它。" 而对于后者，谨慎不仅仅是精神层面的，Swift 还从语法层面上，帮助你在处理空值时，游刃有余。

#### 第一小节：强迫你谨慎处理的 "空" 值

Swift 不仅统一了所有空值的表达方式，并且，当一个变量值有可能为空时，你必须明确指出来。为了避免犯错，你甚至不能只是简单的读取 optional 变量的值。总之，对于这类 "危险分子"，Swift 处处都会强迫你小心处理。

定义：

    var name: Type?

在特定的类型后面加一个问号，表示这个变量的值有可能为空，也就是定义了一个 optional。我们只能给 optional 类型的变量赋值 nil，普通的变量是不可以为 nil 的。

访问一个 Opitional 变量：加 ! 取值，但前提是必须不为 nil

    if convertResult != nil {
      // Force unwrapping
      print(convertResult!)
      let sum = convertResult! + 1
    }

当我们确定一个 convertResult 不为空的时候，我们就在 convertResult 后面加一个叹号 (!) 来读取 optional 包含的值，这个过程叫做 unwrap optional。如果你不加判断直接去 unwrap 一个 optional，当 optional 赶巧为 nil 时，会引发运行时错误，你的 App 就会没有悬念的闪退了。

Optional binding

在前面的例子里，判断了 convertResult 不为空之后，在 if 语句内部，我们明知道它不为空，每次还要使用叹号 unwrap optional 很麻烦，为了解决这个问题，Swift 提供了一个叫做 optional binding 的语法：

    // Optional binding
    if var number = convertResult {
      let sum = number + 1
      number += 1
      print(number)
      print(convertResult)
    } else {
      print("Convert result is nil")
    }

Implicitly Unwrapped Optional

除了我们在上面介绍的 optional 之外，Swift 还提供了一种叫做 implicity unwrapped optional 的变量，它既有 optional 的特性 (可以为 nil)，用起来又像是一个普通的变量 (无需使用叹号进行 unwrap)。

    // Implicitly Unwrapped Optionals
    var possibleString: String! = "A dangerous way!"
    print(possibleString)

定义这种 optional，直接在变量类型的末尾使用一个叹号就可以了。在后面的 print 语句里，我们可以看到，访问 possibleString 时，我们没有使用叹号进行 unwrap。

这看上去方便，实则隐患重重，当你访问了一个值为 nil 的 implicitly unwrapped optional 时，你的 App 就闪退了。

通常，这种类型的 optional 会被用在某个 "一定会被初始化" 的机制里，用来隐藏某个变量是一个 optional 的语义。我们在处理对象 reference cycle 的三种方式中，会看到它的一个正确的用法。

#### 第二小节：Optional chaining 关键技术模拟

    nilPerson?.card?.type
    noCardPerson?.card?.type
    creditCardPerson?.card?.type

在这里，我们使用 `?.` 操作符把多个 optional 串联了起来，在 Playground 里我们可以看到，当 optional 为 nil 的时候，串联的结果会自动变成 nil，而当 optional 为某个对象时，optional 就像不存在一样。
