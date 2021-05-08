# Clojure - Note 3

## Part 3 - 2

Note for [Clojure 入门教程](https://www.gitbook.com/book/wizardforcel/clojure-fpftj/details)

### 迭代

- dotimes (宏)
- while (宏)

dotimes 执行给定的表达式指定的次数。

while，当条件为 true 时一直执行相应的表达式。

详略。

#### 列表推导式

(Python 中也有)

- for
- doseq

可以用 :when 和 :while 进行过滤。

for 只接受一个表达式，它返回一个惰性集合作为结果，用 dorun 强制提取 for 所返回的惰性集合 (不懂...)

doseq 接受任意数据表达式，以有副作用的方式执行它们，并且返回 nil。

示例，暂略。

### 递归

递归发生在一个函数直接或者间接调用自己的时候。一般来说递归的退出条件有检查一个集合是否为空，或者一个状态变量是否变成了某个特定的值 (比如 0)。这一种情况一般利用连续调用集合里面的 next 函数来实现。后一种情况一般是利用 dec 函数来递减某一个变量来实现。

尾递归，Java 和 Clojure 并不支持 (啊，真的吗...)

在 Clojure 中避免递归导致栈溢出的解决方案：

- 使用 loop 和 recur 特殊表
- 使用 trampoline 函数

loop / recur 组合把一个看似递归的调用变成一个迭代。loop 和 let 类似，会建立一个本地的 binding，但它同时也建立了一个递归点，loop 给这些 binding 一个初始值，对 recur 的调用使用程序的控制权返回给 loop 并且给那些 binding 赋给了新值。给 recur 传递的参数一定要和 loop 所创建的 binding 个数一样，且 recur 只能出现在 loop 的最后一行。(类似尾递归，但确实不是尾递归，因为至少它没有显式地调用自身)

recur 不支持那种一个函数调用另外一个函数，然后那个函数再回调这个函数的这种递归。

示例：

    (defn factorial-1 [number]
      (loop [n number factorial 1]
        (if (zero? n)
          factorial
          (recur (dec n) (* factorial n))))

    (println (time (factorial-1 5))) ; -> "Elapsed time: 0.071 msecs"\n120

另外的实现方法，reduce / apply，略。比 loop / recur 慢很多。

trampoline 方法的实现，这里没有讲。

### 谓词

Clojure 提供很多函数来充当谓词 - 测试条件是否成立，它们的返回值是 true 或 false。在 Clojure 中，除了 false 和 nil 被解释成 false，其余都解释成 true。谓词函数的名字一般以问号 `?` 结束。

反射是一种获取一个对象的特性，而不是它的值的过程。比如说对象的类型。有很多谓词函数进行反射。 测试一个对象的类型的谓词包括 class? , coll? , decimal? , delay? , float? , fn? , instance? , integer? , isa? , keyword? , list? , macro? , map? , number? , seq? , set? , string? 以及 vector? 。 一些非谓词函数也进行反射操作，包括 ancestors , bases , class , ns-publics 以及 parents。

测试两个值之间关系的谓词有：< , <= , = , not= , == , > , >= , compare , distinct? 以及 identical?。

测试逻辑关系的谓词有：and , or , not , true? , false? 和 nil?。

测试集合的一些谓词在前面已经讨论过了，包括：empty? , not-empty , every? , not-every? , some? 以及 not-any?。

测试数字的谓词有 even? , neg? , odd? , pos? 以及 zero?。

### 序列

序列可以看成是集合的一个逻辑视图。

LazySeq，很多函数返回的是一个 lazy 序列，这种序列里面的元素不是实际的数据，而是一些方法，它们直到用户真正需要时才被调用。

filter / for / map / range 这些常用的函数返回的都是 lazy 序列。

示例：

    (map #(println %) [1 2 3])

当这句表达式在 REPL 中执行时，会马上三行，分别是 1 2 3，然后再输出三个 nil，因为 println 的返回值是 nil。REPL 总是解析执行所有表达式，但在脚本中执行是，这句表达式不会输出任何东西，因为 map 得到的是一个 LazySeq (还是没太理解...)

有很多方法可以强制对 LazySeq 里的方法进行调用，比如 first / second / nth / last 等。

(其余的还不太理解。)

dorun / doall 迫使一个 LazySeq 里面的函数被调用。

doseq 类似 dorun / doall，但它可以同时操作多个 LazySeq。

doall 会缓存结果，dorun / doseq 不会。

一般推荐使用 doseq...

(其余的还不太理解。我理解 doseq 差不多就是用来遍历的吧)

LazySeq 使得创建无限序列成为可能。因为只有需要使用的数据才会在用到的时候被调用创建。

### 输入和输出

三个特殊的符号：

- `*in*` - 对应 stdin
- `*out*` - stdout
- `*err*` - stderr

可以手动对 `*out*` 重定向，比如重定向到文件：

    (binding [*out* (java.io.FileWriter. "my.log")])
      ...
      (println "This goes to the file my.log")
      ...
      (flush))

打印：

- print
- println
- newline - pirnt + newline = println

其余略，需要时再看文档。

### 解构

- `_`
- `&`
- :as

详略。

### 命名空间

Java 用 class 组织方法，用包来组织 class。Clojure 用名字空间来组织事物，事物包括：

- Vars
- Refs
- Atoms
- Agents
- 函数
- 宏
- 名字空间本身

(名字空间? 命名空间?)

符号 (symbols) 是用来给函数、宏以及 binding 分配名字的，符号属于名字空间。任何时候总有一个默认的名字空间，初始化为 "user"，保存在 `*ns*` 特殊符号中。用 in-ns 函数和 ns 宏可以改变默认名字空间。

require 函数加载 Clojure 类库。

    (require 'clojure.string) ; 注意前面的单引号

使用此类库中的 join 方法。

    (clojure.string/join "$" [1 2 3]) ; -> 1$2$3

alias 函数为名字空间取别名。

    (alias 'cs 'clojure.string)
    (cs/join "$" [1 2 3])

refer 函数使得访问类库中的方法时不用全称。

    (require 'clojure.string)
    (refer 'clojure.string)
    (join "$" [1 2 3])

require + refer = use

    (use 'clojure.string)
    (join "$" [1 2 3])

ns 宏，改变当前默认的名字空间，通常在源码第一行指定，支持 :require, :use, :import 等指令 (其实是上面函数的另一种形式，但推荐使用指令，而不是函数，为什么?)

    (ns com.ociweb.demo
      (:require ...)
      (:use ...)
      (:import (java.text NumberFormat) (javax.swing JFrame JLabel)))
    ...

其余略。

### 元数据

(元编程? 好像并不是)

- :private - 表示一个 Var 能否被包外的函数访问
- :doc - 一个 Var 的文档字符串
- :test - 此函数是否是测试函数
- :tag - Var 在 Java 中对应的类型?
- :file - 定义这个 Var 的文件的文件名
- :line - 定义这个 Var 的行数
- :name - Var 名字的 Symbol 对象
- :ns - 所在命名空间
- :macro - 标志是不是宏

函数及宏，都是有一个 Var 对象来表示的，它们都有关联的元数据。用 var 来获得此对象，用 meta 来查看 var 对象。

    (meta (var reverse))
    ; or
    ^#'reverse

    {
      :ns #<Namespace clojure.core>,
      :name reverse,
      :file "core.clj",
      :line 630,
      :arglists ([coll]),
      :doc "Returns a seq of the items in coll in reverse order. Not lazy."
    }

source 函数查看函数源码。


    (source reverse)

    (def reverse
      ...)

### 宏

宏是用来给语言添加新的结构，新的元素的。它们是一些在读入期 (而不是编译期) 就会实际代码替换的一个机制。

(读入期，就是预编译期吧)

函数的所有参数都会被求值，即使它没有用到，但宏不一样，如果没有用到的参数，不会被求值。

用 defmacro 定义宏。宏定义最开始的地方是一个反引号 "`"，它用来防止宏体中的表达式被求值，宏体中如果想让表达式求值，在表达式前面加上波浪线 "~"。

    (defmacro around-zero [number negative-expr zero-expr positive-expr]
      `(let [number# ~number] ; so number is only evaluated once
        (cond
          (< (Math/abs number#) 1e-15) ~zero-expr
          (pos? number#) ~positive-expr
          true ~negative-expr)))

其余的不是很明白，先跳过。

宏不能作为参数传递给函数，但可以用匿名函数对它进行包装再作为参数传递给函数。

### 并发

Clojure 对于并发是非常友好的，因为它的状态的不可变。

为了更好地进行并发编程是很多开发人员选择 Clojure 的原因。Clojure 的所有的数据都是只读的，除非你显示的用 Var, Ref, Atom 和 Agent 来标明它们是可以修改的。

future 宏可以把它 body 里面的表达式在另外一个线程执行。

pmap，map 的并行版本。

详略。

### 引用类型

引用类型是一种可变引用指向不可变数据的一种机制。Clojure 有 4 种引用类型：

- Vars
- Refs
- Atoms
- Agents

共同特征：

- 都可以指向任意类型的对象
- 都可以利用函数 deref 以及宏 @ 来读取它所指向的对象
- 都支持验证函数，这些函数在它们把指向的值发生变化时自动调用，如果新值合法，验证函数返回 true，如果不合法，则返回 false 或抛出异常
- Agents，支持 watchers。如果监听的引用的值发生变化，Agent 将得到通知 (类似验证函数?)

#### Vars

Vars 是一种可以有一个被所有线程共享的 root binding 并且每个线程线程还能有自己线程本地 (thread-local) 的值的一种引用类型。

Vars 的创建：

    (def name value)

创建和修改本地线程的 Vars：

    (binding [name expression] body)
    (set! name expression)

不鼓励使用 Vars，因为这和函数式不可变的原则违背。

#### Refs

Refs 是用来协调对于一个或者多个 binding 的并发修改的。这个协调机制是利用 Software Transactional Memory (STM) 来实现的。 Refs 指定在一个事务里面修改。

对 Refs 的修改要在事务中进行，用 dosync 宏执行一个事务。

创建 Ref 变量，用 ref 函数：

    (def name (ref value))

dosync 宏用来包裹一个事务，在事务中中 ref-set 来改变一个 Refs 的值并返回新值。

    (dosync
      ...
      (ref-set name new-value)
      ...)

如果要赋的新值提基本旧值，需要三个步骤：

1. deref 这个 Ref，取得它的旧值
1. 计算新值
1. 设置新值

alter 和 commute 函数在一个操作里面完成这三个步骤。示例：

    (dosync
      ...
      (alter counter inc)
      ; or as
      (commute counter inc)
      ...)

其余略。

#### Validation 函数

举了个 validation 函数的例子。如下例所示，my-ref 只能是整数值。

    (def my-ref (ref 0 :validator integer?))

    (try
      (dosync
        (ref-set my-ref 1)      ; works
        (ref-set my-ref "foo")) ; doesn't work
      (catch IllegalStateException e
        ; do nothing
        ))

    (println "my-ref =" @my-ref) ; due to validation failure -> 0

#### Atoms

Atoms 提供了一种比使用 Refs & STM 更简单的更新当前值的方法。它不受事务的影响，不需要在事务中修改值。

创建 Atom 变量：

    (def my-atom (atom 1))

有三个函数可以修改一个 Atom 的值：`reset!` `compare-and-set!` `swap!`

    (def my-atom (atom 1))
    (reset! my-atom 2)
    (println @my-atom) ; -> 2

`compare-and-set!` 接受三个参数，第一个为 Atom 变量，第二个为期望的 Atom 变量的旧值，第三个为新值。set 时，会先读取 Atom 变量此时的值，看是否与期望的旧值相等，如果相等，就赋新值，否则赋新值失败。

`swap!` 实际是 `compare-and-set!` 的封装，但不同的是，`swap!` 如果失败了，会自动重试，直到成功为止。

#### Agents

Agents 是用把一些事情放到另外一个线程来做 -- 一般来说不需要事务控制的。它们对于修改一个单个对象的值 (也就是 Agent 的值) 来说很方便。这个值是通过在另外的一个 thread 上面运行一个 "action" 来修改的。一个 action 是一个函数， 这个函数接受 Agent 的当前值以及一些其它参数。

创建 Agent 变量：

    (def my-agent (agent value))

使用，说得很模糊，没有例子，先跳过。

#### Watchers

Agents 可以用作其它几种引用类型的监视器。当一个被监视的引用的值发生了改变之后，Clojure 会通过给 Agent 发送一个 action 的形式通知它。

两个常用方法：add-watch, remove-watch。

例子没太看懂，先跳过。

### 编译

当 Clojure 的源码作为脚本执行时，它们是在运行时被编译成 Java 的 ByteCode 的。但同时我们也可以提前编译 (AOT - ahead-of-time) 它们成 Java ByteCode，这会缩短程序启动时间，而且产生的 .class 文件还可以供 Java 使用。

编译步骤详略。

#### 在 Java 中调用 Clojure

先跳过，需要时再看。

### 自动化测试

先跳过，需要时再看。

### 编辑器和 IDE

略。

### 桌面应用

略。

### Web 应用

Compojure 框架，详略。

### 数据库

jdbc 库。

clj-record 提供了一个类似 Ruby on Rails 中 ActiveRecord 的数据库访问包。

详略。

### 库

常用的第三方库，详略。

### 总结

略。

### 参考

略。
