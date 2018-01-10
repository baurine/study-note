# Kotlin - Note 1

Note for [Kotlin offical document](https://www.kotlincn.net/docs/reference/).

(失策，不应该一开始看参考的，这个应该是写代码时不知该怎么用时才看的...)

Kotlin 基于 Java，语法和 Swift 又那么相似，所以，快速了解就行，等到用的时候自然会上手。

各种语言的语法都大同小异，重要是去了解它们与其它语言所与众不同的地方。

## 基础

### 数据类型

**整数**

毫无疑问，包括整数与浮点数。

- Double (64)
- Float (32)
- Long (64)
- Int (32)
- Short (16)
- Byte (8)

> 在 Java 平台数字是物理存储为 JVM 的原生类型，除非我们需要一个可空的引用 (如 Int?) 或泛型。后者情况下会把数字装箱。

用 `==` 判断两个对象的值是否相等，用 `===` 判断两个对象是不是同一个对象。

    val a: Int = 10000
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    print(boxedA == anotherBoxedA)  // true
    print(boxedA === anotherBoxedA) // false

用 var 声明变量，用 val 声明只读的变量。

**显式转换**

较小的类型不能隐式地转换成较大的类型，此时，要进行显式转换。

    val b: Byte = 1        // ok
    val i: Int = b         // wrong!
    val i: Int = b.toInt() // ok

类似的转换还有，toByte(), toShort() ... 详略。

**运算**

位运算居然没有特殊字符来表示，神奇，为什么不能用 Java 的 `>>` `<<` 等?

位运算：shl(bits), shr(bits), ushr(bits), and(bits), or(bits), xor(bits), inv()

**字符**

单个字符用 Char 类型表示，用单引号 '' 表示。

多个字符用 String 类型表示，用双引号 "" 表示。

Char 类型有 toInt() 方法，可以转成整数。

**布尔值**

true/false

**数组**

`Array<T>` 类型，具有 get() 和 set() 方法。

初始化：

    val arr = arrayOf(1,2,3) // [1,2,3]
    val asc = Array(3, {i -> (i*i).toString()})  // ["0", "1", "2"]

Kotlin 还有无装箱开销的数组类来表示原生类型数组：ByteArray, ShortArray, IntArray 等，它们并非继承自 `Array<T>`，但有相同的方法集。

    val x: IntArray = intArrayOf(1,2,3)
    x[0] = x[1] + x[2]

**字符串**

String 类型，不可变，元素可以用下标访问 - s[i]，可以用 for...in 循环访问。

    for (c in str) { ... }

字面值，两种表达方式，双引号，里面有转义字符，三引号，原生字符串，内部没有转义。和其它语言类似，不详述了。

字符串模板，即在字符串在插值，和 JavaScript / Ruby 类似。

    val i = 10
    val s = "i = $i"  // i = 10

    val s = "abc"
    val str = "$s.length is ${s.length}"  // abc.length is 3

### 包

    package foo.bar

声明此源文件的包名，其实就是命名空间啦。然后这个源文件中的所有内容都包含在这个包名之内。

用 import 导入其它包。有多个 Kotlin 的包会默认导入到每个 Kotlin 文件中。

### 控制流 - if / when / for / while

**if**

Kotlin 的 if 有点特别呀，吸收了 Ruby 的特性，比如表达式可以不用 {}，最后一行做为返回值，可以写在一行。if 可以替代三目表达式。

    // 传统用法
    var max = a
    if (a<b) max = b

    // with else
    var max: Int
    if (a>b) {
      max = a
    } else {
      max = b
    }

    // 作用表达式
    val max = if (a>b) a else b
    // 还是觉得三目表达式好用，那 Kotlin 还支持三目表达式吗?

if 的分支可以是代码块，最后的表达式作为该块的值。

    val max = if (a>b) {
      print("Choose a")
      a
    } else {
      print("Choose b")
      b
    }

**when**

用来取代 switch...case，很多种灵活的用法，详略。

    when (x) {
      1 -> print("x == 1")
      2 -> print("x == 2")
      else -> {
        print("x is neither 1 nor 2")
      }
    }

when 也可以用来取代 if...else if...else 链，此时 when 后面没有参数。(Go 的 switch 也可以)

    when {
      x.isOdd() -> print("x is odd")
      x.isEven() -> print("x is even")
      else -> print("x is funny")
    }

**for**

for...in 对集合进行迭代访问。

**while**

while, do...while，略。

### 返回与跳转

return / break / continue

Kotlin 还支持类似 goto 的用法，即在 return / break / continue 后面加上标签值。(为啥要加这种功能呢，好丑，先跳过吧，看到有人用再看)

## 类和对象

### 类和继承

用 class 声明类。

**构造函数**

在 Kotlin 中一个类可以有一个主构造函数和多个次构造函数 (和 Swift 一样)。

主构造函数是类头的一部分，它跟在类名后。

    class Person constructor(firstName: String) {
      ...
    }

如果主构造函数没有任何注解或可见性修饰符，constructor 这个关键词可以省略。

    class Person(firstName: String) {
      ...
    }

主构造函数不能包含任何的代码 (嗯，有点特别)，初始化的代码放到以 init 关键字开始的初始化块 (initializer block) 中，可以用多个 init block，与属性初始化器交织在一起。

    class InitOrderDemo(name: String) {
      val firstProperty = "First property: $name".also(::println)
      init {
        println("First initializer block that prints ${name}")
      }
      val secondProperty = "Second property: ${name.length}".also(::println)
      init {
        println("Second initializer block that prints ${name.length}")
      }
    }

(shit，难怪上面的用法不常见) 事实上，声明属性以及从主构造函数初始化属性，Kotlin 有简洁的语法：

    class Person(val firstName: String, val lastName: String, var age: Int) {
      ...
    }

如果构造函数有注解或可⻅性修饰符，这个 constructor 关键字是必需的，并且这些修饰符在它前面:

    class Customer public @Inject constructor(name: String) { ... }

**次构造函数**

类也可以声明前缀有 constructor 的次构造函数。

    class Person {
      constructor(parent: Person) {
        parent.children.add(this)
      }
    }

如果类有一个主构造函数，每个次构造函数需要委托给主构造函数，可以直接委托或者通过别的次构造函数间接委托。委托到同一个类的另一个构造函数用 this 关键字即可。

    class Person(val name: String) {
      constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
      }
    }

**创建类的实例**

    val customer = Customer("Joe")

Kotlin 不需要 new，也没有 new 关键字。

**类成员**

- 构造函数和初始化块
- 函数
- 属性
- 嵌套类和内部类
- 对象声明

**继承**

共同的超类，Any，Any 不是 java.lang.Object。

    class Any {
      fun equals()
      fun hashCode()
      fun toString()
    }

声明继承的父类：

    open class Base(p: Int)
    class Drived(p: Int) : Base(p)  // 注意 Base 中的参数

如果该类有一个主构造函数，其基类型可以 (且必须) 用基类型的主构造函数就地初始化。

如果没有主构造函数，则在次构造函数中用 super() 初始化其基类型，或用 this() 委托给另一个构造函数。

    class MyView : View {
      constructor(ctx: Context) : super(ctx)
      constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
    }

open 与 Java 中的 final 相反，表示允许其它类继承自它，Kotlin 默认所有的类都是 final，与 Java 正好相反。

**方法重载**

Kotlin 是必须用 override 显式声明重载的方法，否则编译通不过。Java 是可选的，且用注解 @override 声明。

如果一个方法用 final 声明，则这个方法禁止被重载。

**属性重载**

也必须用 override 显式声明。可以用 var 来重载父类中的 val 属性，但反之不行。因为 val 属性相当于 get() 方法，用 var 重写相当于增加了 一个 set() 方法。

**访问父类**

在子类中用 super 访问父类。

在内部类中访问外部类的超类，用 `super@Outer`。

    class Bar : Foo() {
      override fun f() { /* ...... */ }
      override val x: Int get() = 0

      inner class Baz {
        fun g() {
          super@Bar.f() // 调用 Foo 实现的 f()
          println(super@Bar.x) // 使用 Foo 实现的 x 的 getter
        }
      }
    }

**覆盖规则**

如果继承自多个父类 (包括接口)，父类中有多个同名的方法，那么子类必须显式地重写此方法，用 `super<A>` `super<B>` 来区分不同的父类。

**抽象类**

abstract 声明。

**伴生对象**

与 Java 或 C# 不同，Kotlin 没有静态方法。(What!?) 它建议简单地使用包级函数。(原来如此)

### 属性和字段

属性的简单声明：

    class Address {
      var name: String = ...
    }

复杂声明：

    var stringRepresentation: String
      get() = this.toString()
      set(value) {
        setDataFromString(value)
      }

在 set() 中用 filed 表示该属性自身：

    var counter = 0 // 此初始器值直接写入到幕后字段
      set(value) {
          if (value >= 0) field = value
      }

或者使用幕后属性 (backing property)

    private var _table: Map<String, Int>? = null
    public val table: Map<String, Int>
        get() {
            if (_table == null) {
                _table = HashMap() // 类型参数已推断出
            }
            return _table ?: throw AssertionError("Set to null by another thread")
        }

**编译期常量**

用 const 修饰。

    const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

**延迟初始化属性与变量**

用 lateinit 修饰符。先跳过。

### 接口

Kotlin 的接口与 Java 8 类似，既包含抽象方法的声明，也包含实现。与抽象类不同的是，接口无法保存状态。它可以有属性但必须声明为抽象或提供访问器实现。

    interface MyInterface {
        val prop: Int  // 抽象的

        val propertyWithImplementation: String
            get() = "foo"

        fun foo() {
            print(prop)
        }
    }

    class Child : MyInterface {
        override val prop: Int = 29
    }

### 可见性修饰符

private, protected, internal, public，默认 public。

internal，相同模块内可见。

### 扩展

为已存在的类扩展方法或属性，Kotlin 采用了 C# 的语法。

扩展函数：

    fun MutableList<Int>.swap(index1: Int, index2: Int) {
        val tmp = this[index1] // this 对应该列表
        this[index1] = this[index2]
        this[index2] = tmp
    }

`MutableList<Int>` 为接收者类型，也就是被扩展的类型。

注意，扩展是静态解析的，扩展的函数，并没有成为类的成员方法，上例中，实际它不过是声明了一个 swap() 函数，它隐含的第一个参数是一个 `MutableList<Int>` 的对象。它没有多态的功能。

(有点明白了 Go 的为结构体扩展方法的设计理念了，它实际是统一把成员方法也作为扩展方法处理了)

**可空接收者**

(Go 也可以)

    fun Any?.toString(): String {
        if (this == null) return "null"
        // 空检测之后，"this" 会自动转换为非空类型，所以下面的 toString()
        // 解析为 Any 类的成员函数
        return toString()
    }

**扩展属性**

    val <T> List<T>.lastIndex: Int
        get() = size - 1

**伴生对象的扩展**

    class MyClass {
        companion object { }  // 将被称为 "Companion"
    }

    fun MyClass.Companion.foo() {
        // ...
    }

使用：

    MyClass.foo()

**扩展声明为成员**

可以把一个类 (A) 的扩展方法声明在另一个类 (B) 中，并成为这个类 (B) 的成员方法，并且这个成员方法可以被子类继承，重写。

    class D {
        fun bar() { ... }
    }

    class C {
        fun baz() { ... }

        // D.foo() 可以被 C 的子类继承，重写
        fun D.foo() {
            bar()   // 调用 D.bar
            baz()   // 调用 C.baz
        }

        fun caller(d: D) {
            d.foo()   // 调用扩展函数
        }
    }

### 数据类

(我觉得是对应 Java 中的 JaveBean)

    data class User(val name: String, val age: Int)

详略，需要时再看。

### 密封类

没明白，先跳过。

### 泛型

`Type<T>`，通过在 T 前使用 in 或 out，替代在 Java 中使用 `<? super T>` 和 `<? extends T>`。

没有完全理解，先跳过。

### 嵌套类和内部类

嵌套类，不可以访问外部类，相当于是 Java 中的内部静态类。

内部类，用 inner 修饰，可以访问外部类。

匿名内部类，和 Java 中一样，略。

### 枚举类

枚举不是一个单独的类型，而是用 enum 修饰的类。

    enum class Direction {
        NORTH, SOUTH, WEST, EAST
    }

    enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
    }

详略。

### 对象表达式和对象声明

对象表达式，匿名内部类的进一步使用说明。

    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            // ...
        }

        override fun mouseEntered(e: MouseEvent) {
            // ...
        }
    })

**对象声明**

直接声明一个单例对象，用 object 修饰符。

    object DataProviderManager {
        fun registerDataProvider(provider: DataProvider) {
            // ...
        }

        val allDataProviders: Collection<DataProvider>
            get() = // ...
    }

使用：

    DataProviderManager.registerDataProvider(...)

**伴生对象**

类内部的对象声明可以用 companion 关键字标记：

    class MyClass {
        companion object Factory {
            fun create(): MyClass = MyClass()
        }
    }

伴生对象的成员可以通过类名来调用：

    val instance = MyClass.create()

可以省略伴生对象的名字，此时它的名字默认为 Companion (那就只能有一个省略名字的伴生对象喽?)

    class MyClass {
        companion object {
        }
    }

    val x = MyClass.Companion

虽然看着很像是静态成员，但实际在运行时它们仍然是真实对象的实例成员。它们还可以实现接口。详略。

对象表达式和对象声明之间有一个重要的语义差别：

- 对象表达式是在使用他们的地方立即执行 (及初始化) 的；
- 对象声明是在第一次被访问到时延迟初始化的；
- 伴生对象的初始化是在相应的类被加载 (解析) 时，与 Java 静态初始化器的语义相匹配。

### 委托

(明白)

委托，用组合来替代继承，将实际功能转发给另一个对象来完成。在其它语言中实现委托会产生很多样板代码，而 Kotlin 用一个关键字 by 来消除这些样析代码。

示例： 类 Derived 可以继承一个接口 Base，并将其所有共有的方法委托给一个指定的对象。

    interface Base {
        fun print()
    }

    class BaseImpl(val x: Int) : Base {
        override fun print() { print(x) }
    }

    class Derived(b: Base) : Base by b

    fun main(args: Array<String>) {
        val b = BaseImpl(10)
        Derived(b).print() // 输出 10
    }

Derived 的超类型列表中的 by-子句表示 b 将会在 Derived 中内部存储。 并且编译器将生成转发给 b 的所有 Base 的方法。

(实际是 Kotlin 帮你自动在 Drived 类中生成了 `override fun print() { b.print() }` 方法。)

### 委托属性

呃，有一点复杂。先跳过吧。

总的思想是说，可以把一个属性的 get() 和 set() 方法委托由另一个对象来实现。并且库提供了一些默认的属性委托对象，比如 lazy，observable ... lazy 委托可以让你的初始化延迟到第一次访问，observable 可以观察一个属性旧值和新值的对比。

以及讲了如何自己实现一个类似 lazy, observable 之类的委托。

## 函数与 lambda 表达式

### 函数

函数声明：

    fun double(x: Int): Int {
        return 2*2
    }

用 fun 关键字声明，参数和类型采用 Pascal 定义法。

默认参数，命名参数，略。

单表达式函数：

    fun double(x: Int): Int = x*2

当函数返回值可以推导时，可以省略：

    fun double(x: Int) = x*2

可变数量的参数，用 vararg 修饰，只能是最后一个，用 Array 接收。

将一个数组对象展开，用展开操作符 `*`。

**中缀表示法**

    // 给 Int 定义扩展
    infix fun Int.shl(x: Int): Int {
        ...
    }

    // 用中缀表示法调用扩展函数
    1 shl 2

    // 等同于这样
    1.shl(2)

- 成员函数或扩展函数
- 只有一个参数
- 用 infix 标注

**函数作用域**

Java / C# 中所有函数必须定义在类中，而 Kotlin 可以把函数定义在文件顶层，类之外。(所以，这就是为什么 Kotlin 不需要静态成员方法)

局部函数，闭包，略。成员函数，略。泛型函数，略。内联函数，略。扩展函数，略。

高阶函数和 lambda 表达式，后面会讲。

尾递归函数，用 tailrec 声明，JVM 会做优化。

### 高阶函数和 lambda 表达式

高阶函数是将函数用作参数或返回值的函数。

map 的例子：

    fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
        val result = arrayListOf<R>()
        for (item in this)
            result.add(transform(item))
        return result
    }

调用：

    val doubled = ints.map { value -> value * 2 }

如果 lambda 只有一个参数，则可以省略 `->` 和前面的内容，变量的隐式名称为 it。

    val doubled = ints.map { it*2 }
    strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }

(Swift 中隐式使用 $0, $1 ...)

未使用的变量用 `_` 声明 (在种语言中已形成共识)

**匿名函数**

?? 呃，lambda 不是匿名函数呀... 有点矇了。

所以，从这里的意思是说，不能在 lambda 中表达复杂的逻辑，只能有一个表达式，不能用 return，如果要在其中使用 return，那么其实就变成了匿名函数... (和 Python 一样)

先跳过吧，需要时再回来看。

### 内联函数

> 使用高阶函数会带来一些运行时的效率损失：每一个函数都是一个对象，并且会捕获一个闭包。即那些在函数体内会访问到的变量。内存分配 (对于函数对象和类) 和虚拟调用会引入运行时间开销。

> 但是在许多情况下通过内联化 lambda 表达式可以消除这类的开销。

理解，就跟 C++ 的内联一个道理。通过内联后，不再把 lambda 生成对象，而是直接把它的实现插入到方法中。增大了代码体积，提高了性能...

用 inline 声明一个高阶函数，使参数中的函数内联。

详略。

### 协程 (coroutine)

跟其它语方 (JavaScript, Go, Python) 的协程理念差不多，先略过吧。

## 其它

### 解构声明

    val (name, age) = person

略。ES6 有相同的用法。

### 集合 - List / Set / Map

略。有可变与不可变集合。

### 区间

`..` 与 `...`

一些函数：rangeTo(), downTo(), reversed(), step()

判断是否在区间：in, !in

区间的迭代：for...in

### 类型检查与转换

- is, !is
- as, as?

### This 表达式

略。在不同的作用域表示不同的对象

### 相等性

- 引用相等，两个引用指向同一对象，用 `===`
- 结构相等，用 equals() 判断，用 `==`

### 操作符重载

略。

### 空安全

和 Swift 差不多，略过了。Kotlin 用 `?:` 替代 Swift 中的 `??`

### 异常

try...catch...finally, try...catch 可以作为表达式返回值。

throw

Nothing 表示永远不会返回的函数。

其余略。

### 注解

略，Java 的注解还没完全搞懂呢。

### 反射

用 `::` 符号，取得类引用、函数引用、属性引用。

    fun isOdd(x: Int) = x % 2 != 0
    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd)) // 输出 [1, 3]

### 类型安全的构建器

    import com.example.html.* // 参见下文声明

    fun result(args: Array<String>) =
        html {
            head {
                title {+"XML encoding with Kotlin"}
            }
            body {
                h1 {+"XML encoding with Kotlin"}
                p  {+"this format can be used as an alternative markup to XML"}

                // 一个具有属性和文本内容的元素
                a(href = "http://kotlinlang.org") {+"Kotlin"}

                // 混合的内容
                p {
                    +"This is some"
                    b {+"mixed"}
                    +"text. For more see the"
                    a(href = "http://kotlinlang.org") {+"Kotlin"}
                    +"project"
                }
                p {+"some text"}

                // 以下代码生成的内容
                p {
                    for (arg in args)
                        +arg
                }
            }
        }

html / head / body 其实就是函数。(和 React JSX 的写法差不多，实际在 JSX 中，每个标签就是一个函数)

### 类型别名

typealias

### 多平台项目

略。

## Java 互操作

略。
