# Clojure - Note 1

References:

- [Clojure 快餐教程 (1) - 运行在 JVM 上的 Lisp 方言](https://yq.aliyun.com/articles/229367)
- [Clojure 小教程](http://www.cnblogs.com/fcat/p/5540878.html)
- [Clojure 入门教程](https://www.gitbook.com/book/wizardforcel/clojure-fpftj/details)
- [Clojure 教程](https://www.w3cschool.cn/clojure/)

函数式编程的特点：

- 函数作为一等公民，且应该是纯函数
- 函数可以作为参数传递，也可以作为函数的返回值，高阶函数
- 数据的不可修改性

Clojure 是一个运行在 JVM 之上的函数式编程语言。它可以访问丰富已存在的 Java 类库，比如文件 IO，多线程，数据库操作，GUI 编程，Web 编程...

ClojureScript 项目可以把 Clojure 代码编译成 JavaScript 代码。

## Part 1

Note for [Clojure 快餐教程 (1) - 运行在 JVM 上的 Lisp 方言](https://yq.aliyun.com/articles/229367)

### Clojure 对 Lisp 的改进

- 可以无缝调用 Java API
- 针对 Lisp 括号多的问题，Clojure 除了 S 表达式常用的 `()` 之外，增加了 `[]` 和 `{}` 两种括号

`()` 表示列表，`[]` 表示向量 (即可随机访问的数组)，`{}` 表示 HashMap，`#{}` 表示 Set。

### 查看帮助

使用 `clojure.repl/doc` 宏。

    user=> (doc doc)
    -------------------------
    clojure.repl/doc
    ([name])
    Macro
      Prints documentation for a var or special form given its name

### 列表

在 Lisp 中，列表这种数据和代码的形式是一样的，都是 S 表达式。S 表达式可以理解成用 `()` 包围起来的，以第一个符号进行求值的表达式。

所以，我们不能直接用类似 `(1 2 3)` 这样的表达式来得到一个列表，因为这样一个表达式，会用 1 作为符号或函数，对参数 2 和 3 进行求值，很明显会失败。

所以，有两种方法来得到列表：

- 用 list 函数来生成列表
- 用 quote 特殊表 (special form) 来避免被求值，即得到一个未被求值的表

注意，list 是函数，但 quote 不是函数，它也不是宏，而是特殊表，special form，像后面会讲到的 def，fn，也是 special form，我觉得可以理解成其它语言中的关键字，由语言本身提供，不可自己定义，而函数和宏是可以自己再定义的。

quote 还有简略写法，如 `quote (1 2 3)` 可以简写为 `'(1 2 3)`，可以理解成语言本身提供的语法糖。

    user=> (list 1 2 3)
    (1 2 3)
    user=> (doc list)
    -------------------------
    clojure.core/list
    ([& items])
      Creates a new list containing the items.
    nil
    user=> (quote (2 3 4))
    (2 3 4)
    user=> (doc quote)
    -------------------------
    quote
      (quote form)
    Special Form
      Yields the unevaluated form.

      Please see http://clojure.org/special_forms#quote
    nil
    user=> '(3 4 5)
    (3 4 5)

quote 对任意 S 表达式，不仅仅是列表，都直接返回原值，使其不被求值，比如：

    user=> (quote (+ 1 (+ 2 3)))
    (+ 1 (+ 2 3))

### 列表操作

Lisp 中最常用的 car 和 cdr 在 Clojure 中不被支持。取而代之的是 first / rest / last 之类的函数。

用 first 取得表头：

    user=> (first (list 1 2 3 4))
    1

用 rest 取得除表头余下的部分：

    user=> (rest (list 1 2 3 4))
    (2 3 4)

用 last 取得列表最后一个元素：

    user=> (last (list 1 2 3 4))
    4

用 nth 取得第 n 个元素：

    user=> (nth (list 2 3 4) 2)
    4

将 first 和 rest 元素组合成一个列表：

    user=> (cons 1 '(2 3 4))
    (1 2 3 4)

### 向量

向量类似数组，对于随机访问进行了优化。

Clojure 用 `[]` 表示向量。由于 `[]` 不会和 S 表达式冲突，所以就不用像生成列表那样需要显式地用 list 函数或 quote，直接用类似 `[1 2 3]` 的表达式就可以生成一个向量。

first / rest / last / nth 函数对向量仍然适用。

但需要注意的是，rest 作用在向量上，返回值并不是一个向量，而是一个序列类型 (那是列表吗?)。

    user=> (rest [1 2 3 4])
    (4 5 6) ;看着像是列表

    user=> (class (rest '(1 2 3)))
    clojure.lang.PersistentList
    user=> (class (rest [1 2 3 4]))
    clojure.lang.PersistentVector$ChunkedSeq

### 哈希表

用 `{}` 表示哈希表，使用 key 的名字作为函数名来取得 key 对应的 value。

    {:name "hello", :age 18}

    user=> (:name {:name "hello", :age 18})
    "hello"

### 集合 (Set)

大中小括号都用完了，只好在大括号前加个 `#` 表示集合 (Set)。

    user=> #{1 2 3}
    #{1 3 2} ;为什么不是 #{1 2 3}，待验证

集合 (Set) 中的元素不能重复。可以用 sorted-set 函数构建有序的 set。

    user=> (sorted-set 1 2 2 3)
    #{1 2 3}

### 变量绑定

相当于其它语言中用 `=` 进行赋值。在 Clojure 中用 def 特殊表 (即其它语言中的关键字) 来将值绑定到一个全局变量上，也可以将匿名函数绑定到全局变量上。

    user=> (def a '(1 2 3))
    #'user/a

### 定义函数

一般用 fn 特殊表来定义一个匿名函数，然后用 def 特殊表来将这个匿名函数绑定到一个变量上。

或者直接用 defn 宏 (我觉得 defn 内部应该就是用 fn + def 来实现的) 来定义函数。

定义一个求平方的匿名函数：

    user=> (fn [i] (* i i))
    #object[user$eval17$fn__18 0x7f284218 "user$eval17$fn__18@7f284218"]

将其用 def 绑定到一个变量上：

    user=> (def sqr (fn [i] (* i i)))
    #'user/sqr
    user=> (sqr 4)
    16

直接用 defn 宏定义函数：

    user=> (defn sqr3 [i] (* i i i))
    #'user/sqr3
    user=> (sqr3 4)
    64

通过 defn 宏，还可以为函数提供说明文档，然后通过 doc 函数就可以查看这个说明文档。

    user=> (defn inc "return inc x" [x] (+ x 1))
    #'user/inc
    user=> (doc inc)
    ----------------------
    user/inc
    ([x])
      return inc x
    nil

## Part 2

Note for [Clojure 小教程](http://www.cnblogs.com/fcat/p/5540878.html)

Clojure 简易语法。

### 注释

Clojure 的注释用 `;` (有点特别，既不是 `//`，也不是 `#` ...)

    ; This is a comment
    (map #(+ % 3) [2 3 7])

### 基本类型

- 布尔值
- 整型
- 浮点
- 字符 (\a \A \u2014 ...)
- 字符串
- 符号
- 关键字 (冒号打头，比如 :red)

### 集合类型

- list --> (1 2 3)
- vector --> [1 2 3]
- map --> {"key1" 1, "key2" 2}
- set --> #{1 2 3 4}

### 函数定义

fn / def / defn，在 Part 1 里讲过的略。

函数定义支持变长参数，定义形式，示例：

    (fn mul [& x] #{x})

& 后的参数名为变长参数名，所有参数将作为一个 list 传入函数定义中。比如：

    user=> ((fn mul [& x] #{x}) 1 2 3)
    #{(1 2 3)}

fn 的语法糖简写形式 `#()`，同时，可以用 %1 %2 代替参数名。(从后面来看，貌似如果只有一个参数，可以直接用 % 来表示这个唯一的参数)

    user=> ((#(+ %1 %2) 2 3))
    5

### 块序列

使用 do 将一些操作序列组成一个块，只有最后一个元素的值返回。

    user=> (do 1 (+ 1 2) (- 9 2) 3)
    3

### 设定全局不变量

使用 `(let [] ())` 表达式。

    user=> (let [x 10 y 100] (do (println x) (println y)))

### if / when

    (if (cond) (then-clause) (else-clause))

    (when (cond) (sta1) (sta2) (sta3))

### 递归与循环

讲得不是很清楚，略。

### quote / unquote

讲得不是很清楚，略。

### 与 Java 互操作

**静态域与静态方法**

语法：`ClassName/static_method_or_field`

示例：

    user=> (println Math/PI)
    user=> (println (Math/sqrt 9))

**创建实例对象**

语法：`(ClassName.)` 或 `(new ClassName)`

示例：

    user=> (java.util.HashMap. ["x" 1 "y" 2])
    user=> (new java.util.HashMap ["x" 1 "y" 2])

**调用对象的实例方法或域**

语法：`(.method_or_filed)`

示例：

    user=> (.divide (java.math.BigDecimal. "42") 2M)
    user=> (.x (java.awt.Point. 10 0))
    10

**设置实例属性**

(如果没有提供 setter，可以用 `set!`)

(注意，操作 Java 对象不属于函数式编程，因为用 set 改变对象属性的值，明显就违返了函数式的原则 - 变量不可变)

示例：

    (def point (java.awt.Point 10 10))
    (defn setX [po xval] (set! (.x po) xval))
    (setX point 15)
    (.x point) ;15

**链式调用**

使用 `..` 宏来支持链式调用。

比如，Java 代码是这样的：`new java.util.Date().toString().endsWith("2016")`，用 Clojure 可以这样表示：

    (.. (java.util.Date) toString (endsWith "2016"))

**doto 宏，设置对象的多个实例域**

Java 代码：

    Date time = new Date();
    time.setYear(1);
    time.setMonth(2);
    time.setDate(3);

Clojure 代码：

    (doto (java.util.Date.) (.setYear 1) (.setMonth 2) (.setDate 3))

### 异常处理

抛出异常：

    (throw (Exception. "error happens"))

捕捉异常：

    (defn throw-catch [f]
      (try (f) (catch ArithmeticException e "No dividing by zero!")
               (catch Exception e (str "You are so bad " (.getMessage e)))
               (finally (println "returning... "))))
    (throw-catch #(/ 10 2))

### 命名空间

**创建命名空间**

使用 ns (函数，or 宏，or 特殊表?)

    (ns my.clojure)

**引用其它 Clojure 命名空间**

使用 :require 关键字

    (ns my.core (:require my.clojure))

**只引用其它命名空间的部分函数**

使用 :use

    (ns my.util (:use [my.core :only [myfunc1 myfunc2]]))    ; 只导入 myfunc1 与 myfunc2
    (ns my.util (:use [my.core :exclude [myfunc1 myfunc2]])) ; 只有 myfunc1 与 myfunc2 不导入

**导入 Java 类库**

使用 import

    (import java.util.*)
