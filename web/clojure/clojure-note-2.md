# Clojure - Note 2

## Part 3 - 1

Note for [Clojure 入门教程](https://www.gitbook.com/book/wizardforcel/clojure-fpftj/details)

### 概述

Clojure 里的每个操作被实现成以下三种形式中的一种：

- 函数 (function)
- 宏 (macro)
- 特殊表 (special form)

几乎所有的函数和宏都是用 Clojure 实现的，special form 不是用 Clojure 实现的，它是 Clojure 的类似其它语言中的关键字，个数不多，比如 catch, def, do, finally, fn, if, let, loop ...

传统语言如 Java 的方法调用：

    methodName(arg1, arg2, arg3);

Clojure 的方法调用：

    (function-name arg1 arg2 arg3)

左括号被移到了最前面，逗号和分号不需要了，我们称这种语法叫 form (表?)，这种风格简单而又美丽...

对 Clojure 代码的处理分为三个阶段：读入期，编译期以及运行期。在读入期，读入期会读取 Clojure 源代码并且把代码转变成数据结构，基本上来说就是一个包含列表的列表的列表...(所以人们说，Lisp，代码即数据) 在编译期，这些数据结构被转化成 Java 的 bytecode。在运行期这些 Java bytecode 被执行。函数只有在运行期才会执行。而宏在编译期就被展开成实际对应的代码了。

### 开始吧

如果安装 Clojure，暂略。

### Clojure 语法

Lisp 方言有一个非常简法的语法，数据和代码的表达形式是一样的 (S 表达式?)，一个列表的列表，很自然地在内存中表达成一个 tree。`(a b c)` 表示对函数 a 的调用，参数是 b 和 c，如果要表示一个列表 `(a b c)`，则需要使用 `'(a b c)` 或 `(quote (a b c))`。

语法糖汇总，暂略。

数据类型。

符号 (symbols)，是用来给东西命名的，这些名字被限制在名字空间里 ... 没太明白，后面还会再讲到吧。

后面果然又有讲到，其实符号就是类似变量名啦，用 def 来给它绑定值。

    user=> (def n 3)
    #'user/n
    user=> (* n 2)
    6

上面的 n 就是一个符号，它被限定在 user 这个名字空间中。

关键字以冒号打头，被用来当作唯一标示符，通常用在 map 中 (如 :red, :green ...)

### REPL

REPL (read-eval-print loop)

查看一个函数的帮助，用 `(doc _name_)`。

查看一个函数的源代码，用 `(source _name_)`。

加载并执行一个 clojure 文件，用 `(load-file "_file-path_")`。

### 变量

实际是指不可变的变量。def / let / binding。

def 定义的是全局的 binding，而且可以用 def 改为同一个变量的值。(咦，不是说不可变的吗? 这怎么又可变啦，奇怪?)

let 定义的 binding 只在当前表达式中有效，具有局部作用域，可以覆盖由 def 定义的全局 binding。

binding 类似 let，只在当前表达式中有效，不同的是，let 的 binding 不具有传递性，即在 let 作用域中调用另一个函数，这个函数无法使用 let 中的 binding，而 binding 定义的变量却可以。

可以看教程提供的例子，这里就略去。

### 集合

list / vector / set / map。

当然，Clojure 还可以使用 Java 中提供的集合类型，但不推荐这么做，因为 Clojure 自带的集合类型更适合函数式编程。

所有的 Clojure 集合都是不可修改的、异源的以及持久的。

不可修改意味着一旦一个集合产生后，不能从中删除一个元素，也不能添加一个元素。(函数式编程的特征)

异源，意味着在一个集合中放置任何类型的元素，不需要所有元素都具有相同类型。

持久，意味着一个集合新的版本产生后，旧的版本还存在。Clojure 以一种高效的，共享内存的方式来实现这个的。(现代语言好像都是这么做的...)

集合的各种操作函数，如

- count / reverse
- map / apply / filter
- first / rest / last / nth
- every? / not-every? / some / not-any?

详略。

### 列表 - list

相当于 Java 中的 LinkedList，链表。

创建 list 的方式：

    (def stooges (list "Moe" "Larry" "Curly"))
    (def stooges (quote ("Moe" "Larry" "Curly")))
    (def stooges '("Moe" "Larry" "Curly"))

相关的函数：

- some
- contains?
- conj / cons
- remove
- into
- peek / pop

详略。

### 向量 - vector

vector，相当于 Java 中的 ArrayList，数组，可以随机访问。定义函数时的参数列表就是 vector。

创建 vector 的两种方式：

    (def stooges (vector "Moe" "Larry" "Curly"))
    (def stooges ["Moe" "Larry" "Curly"])

一般情况下，推荐使用 vector 而不是 list。(原因是因为 `[]` 的语法不容易和 S 表达式混淆? 这原因有点勉强哦)

对 list 操作的函数同样适用于向量，除此之外，还有一些专门用于操作向量的函数：

- get
- assoc (相当于 replace 或 push)
- subvec

详略。

### Set

不包含重复元素的集合，有两种 set：排序的和不排序的。

创建 set 的方式：

    (def stooges (has-set "Moe" "Larry" "Curly"))    ; not sorted
    (def stooges #{"Moe" "Larry" "Curly"})           ; not sorted, same as above
    (def stooges (sorted-set "Moe" "Larry" "Curly")) ; sorted

contains? 方法：

    (contains? stooges "Moe")  ; -> true
    (contains? stooges "Mark") ; -> false

set 集合可以作为函数来使用 (这一点相比传统语言有点神奇)：

    (stooges "Moe")  ; -> "Moe"
    (stooges "Mark") ; -> nil
    (println (if (stooges person) "stooge" "regular person"))

conj / into 对 set 同样适用。disj 函数通过去掉 set 中给定的元素来创建新的 set：

    (def more-stooges (conj stooges "Shemp"))      ; -> #{"Moe" "Larry" "Curly" "Shemp"}
    (def less-stooges (disj more-stooges "Curly")) ; -> #{"Moe" "Larry" "Shemp"}

其它函数，略。

### 映射 - map

key-value 对，key 和 value 可以是任意对象。

创建 map 的几种方式 (注意，逗号不是必须的，这里只是为了增加可读性)：

    (def popsicle-map (hash-map :red :cherry, :green :apple, :purple :grape))
    (def popsicle-map {:red :cherry, :green :apple, :purple :grape}) ; same as above
    (def popsicle-map (sorted-map :red :cherry, :green :apple, :purple :grape))

map 可以作为函数，如果 key 是关键字 (即以冒号开头的值)，那么 key 也可以作为函数。(很神奇吧)

三种取得 :green 对应的值的方法：

    (get popsicle-map :green)
    (popsicle-map :green)
    (:green popsicle-map)

contains? 作用于 map 时，它返回 map 是否包含给定的 key。

    (contains popsiclie-map :green) ; -> true

keys 得到 map 所有 key 的集合，vals 得到 map 所有 value 的集合。

    (keys popsicle-map) ; -> (:red :green :purple)
    (vals popsicle-map) ; -> (:cherry :apple :grape)

assoc，在原来 map 的基础上添加新的 key-value 对，创建新的 map。

dissoc，在原来 map 的基础上，忽略一些 key-value 对，创建新的 map。

遍历 map，用 doseq 函数。

    (doseq [[color flavor] popsicle-map]
      (println (str "The flavor of" (name color) " popsicles is " (name flavor) ".")))

name 函数的作用是将关键字 (keyword) 转换成字符串。

select-keys，选择 map 中的子集。

conj，将两个 map 合集，得到一个新的 map。

#### 嵌套的 map

    (def person {
      :name "Mark Volkmann"
      :address {
        :street "644 Glen Summit"
        :city "St. Charles"
        :state "Missouri"
        :zip 63304}
      :employer {
        :name "Object Computing, Inc."
        :address {
          :street "12140 Woodcrest Executive Drive, Suite 250"
          :city "Creve Coeur"
          :state "Missouri"
          :zip 63141}}})

如何取得内嵌的 key 对应的 value，比如取得 person 的 employer 的 adderss 的 city 的值。用 Java 表示就是 person.employer.address.city。

Clojure 有以下几种方法实现：

    (get-in person [:employer :address :city])
    (-> person :employer :address :city)
    (reduce get person [:employer :address :city])

get-in 和 reduce 是函数，`->` 是宏。`->` 类似 Elixir 中的 `|>`，前一个函数的返回值作为后一个函数的参数。

    (f1 (f2 (f3 x))
    (-> x f3 f2 f1)

(前面说过，如果 map 的 key 是关键字，那么 key 可以作为函数)

clojure.contrib.core 还有一个 `-?>` 宏，如果链上有任何一个值遇到 nil，它就直接返回，能避免崩溃。

reduce 的解释，暂略。

assoc-in 可以修改一个内嵌的 key 的值。

    (assoc-in person [:employer :address :city] "Clayton")

update-in 也是用来修改一个内嵌的 key 的值，但它是通过一个给定的函数来计算得到。

    (update-in person [:employer :address :zip] str "-1234") ; -> new zip: 63141-1234

### StructMap

类似 map，模拟 Java 中的 JavaBean。它比普通的 map 的优点就是，它把一些常用的字段抽象到一个 map 里面去，这样你就不用一遍一遍的重复了。并且和 Java 类似，他会帮你生成合适的 equals 和 hashCode 方法。并且它还提供方式让你可以创建比普通 map 里面的 hash 查找要快的字段访问方法 (JavaBean 里面的 getXXX 方法)。

(Elixir 中也有结构体的概念。)

create-struct 函数和 defstruct 宏都可以用来定义 StructMap，key 一般是 keyword。

    (def vehicle-struct (create-struct :make :model :year :color))
    (defstruct vehicle-struct :make :model :year :color)

用 struct 实例化一个 StructMap 对象 (所以，函数式也还是有对象的嘛...)，相当于 Java 中的 new。

    (def vehicle (struct vehicle-struct "Toyota" "Prius" 2009))

accessor 函数可以创建一个类似 Java 中的 getXXX 方法，它的好处是避免 hash 查找，它比普通的 hash 查找要快。

    (def make (accessor vehicle-struct :make))
    (make vehicle)  ; -> "Toyota"
    (vehicle :make) ; same but slower
    (:make vehicle) ; same but slower

### 函数定义

def / fn / defn。

函数必须先定义，再使用。有时候做不到，比如两个方法相互调用，Clojure 采用了和 C 语言类似的做法，先声明，用 declare。

    (declare function-name)

Part 1 和 Part 2 里讲过的了就不再赘述。

函数定义可以包含多个参数列表以及对应的方法体，每个参数列表必须包含不同个数的参数，因为 Clojure 是根据参数个数来进行重载的 (Elixir 也是这样的)。

    (defn parting
      ([] (parting "World"))
      ([name] (parting name "en"))
      ([name language]
        (condp = language
          "en" (str "Goodbye," name)
          "es" (str "Adios," name)
          (throw (IllegalArgumentException.
            (str "unsupported language" language))))))

    (println (parting)) ; -> Goodbye, World
    (println (parting "Mark")) ; -> Goodbye, Mark
    (println (parting "Mark" "es")) ; -> Adios, Mark
    (println (parting "Mark" "xy")) ; -> java.lang.IllegalArgumentException: unsupported language xy

匿名函数，fn / #()，前者可以包含多个表达式，后者只能有一个表达式。如果有多个参数，可以用 %1 %2 表示，如果只有一个，可以用 % 表示。

    (def years [1940 1944 1961 1985 1987])
    (filter (fn [year] (even? year)) years) ; -> 1940 1944
    (filter #(even? %) years) ; -> same as above

defmulti / defmethod 经常被用在一起来定义 multi-method。

宏 defmulti 的参数有两个，第一个是方法名，第二个是 dispatch 函数，它会作用是方法的参数上，其返回值会被用来选择到底调用哪个被重载的函数。

宏 defmethod 的参数，第一个是方法名，第二个是上面的 dispatch 函数的返回值，剩下的参数是方法的参数以及方法体。

看例子就明白了。

    (defmulti what-am-i class)
    (defmethod what-am-i Number [arg] (println arg "is a Number"))
    (defmethod what-am-i String [arg] (println arg "is a String))
    (defmethod what-am-i :default [arg] (println arg "is something else"))

    (what-am-i 19)      ; -> 19 is a Number
    (what-am-i "Hello") ; Hello is a String
    (what-am-i true)    ; true is something else

dispatch 就是一个普通的函数，可以自己定义。

complement 函数，接受一个函数作为参数，如果这个函数的返回值为 true，那它返回 false，反之亦然，相录于取反操作。

comp 把任意多个函数组合成一个，前面一个函数的返回值作为后一个函数的参数。类似 Elixr 的 `|>`。

partial 函数，创建一个新函数，通过给旧的函数制定一个初始参数。Python 有类似用法，偏函数，详略。

### Java 互操作

导入 Java 类库，用 import，java.lang 默认已导入。

    (import
      '(java.util Calendar GregorianCalendar)
      '(javax.swing JFrame JLabel))

访问类中的常量。

    (. java.util.Calendar APRIL)
    (. Calendar APRIL)
    java.util.Calendar/APRIL
    Calendar/APRIL

调用类中的静态方法。

    (. Math pow 2 4)
    (Math/pow 2 4)

创建 Java 对象，两种方式。

    (import '(java.util Calendar GregorianCalendar))
    (def calendar (new GregorianCalendar 2008 Calendar/APRIL 16)) ; April 16, 2008
    (def calendar (GregorianCalendar. 2008 Calendar/APRIL 16))

访问对象方法，两种方式。

    (. calendar add Calendar/MONTH 2)
    (. calendar get Calendar/MONTH) ; -> 5
    (.add calendar Calendar/MONTH 2)
    (.get calendar Calendar/MONTH)  ; -> 7

推荐使用 `.add` `.get` 的方式。前面用点号的方式在定义宏中用得比较多。

可以用 `..` 宏实现链式调用。

    (. (. calendar getTimeZone) getDisplayName)
    (.. calendar getTimeZone getDisplayName)

还有一个类似的宏 `.?.`，它的区别在于，如果遇到 nil，则立即返回，不再调用，避免出现 NullPointException。

doto 函数可以用来调用一个对象上的多个方法，返回第一个参数，即对象。

    (doto calendar
      (.set Calendar/YEAR 1981)
      (.set Calendar/MONTH Calendar/AUGUST)
      (.set Calendar/DATE 1))
    (def formatter (java.text.DateFormat/getDateInstance))
    (.format formatter (.getTime calendar)) ; "Aug 1, 1981"

memfn 宏，没太明白，先跳过。

#### 代理

说得很简单，先略过。

#### 线程

所有的 Clojure 方法都实现了 java.lang.Runnable 和 java.util.concurrent.Callable 接口，这使得非常容易把 Clojure 函数和 Java 线程一起使用。

    (defn delayed-print [ms text]
      (Thread/sleep ms)
      (println text))

    (.start (Thread. #(delayed-print 1000 ", World!")))
    (print "Hello")

#### 异常处理

try / catch / finally / throw，和 Java 类似，详略。

### 条件处理

- if (special form)
- when / when-not (宏)
- if-let / when-let (宏)
- condp (宏)
- cond (宏)

if 接受三个参数，第一个是条件，第二个是条件为 true 时要执行的表达式，第三个为 false 时执行的表达式，如果表达式有多个，需要用 do 包围。

when / when-not 类似 if，但 when 只支持条件为 true 时要执行的表达式，when-not 只支持条件为 false 时要执行的表达式，如果表达式有多个，不需要用 do 包围。

if-let 把一个值 bind 到一个变量，然后根据这个 binding 的值来决定执行哪个表达式。

    (defn process-next [waiting-line]
      (if-let [name (first waiting-line)]
        (println name "is next")
        (println "no waiting")))

    (process-next '("Jeremy", "Amanda" "Tami")) ; -> Jeremy is next
    (process-next '()) ; -> no waiting

when-let 类似 if-let，但是没有 else 分支，且支持任意多表达式。

condp 相当于 switch / case。condp 接受两个参数，一个是谓语参数，一般是 `=`，或 instance?。

    (condp = value
      1 "one"
      2 "two"
      3 "three"
      (str "unexpected value, \"" value "\""))

    (condp instance? value
      Number (* value 2)
      String (* (count value) 2))

cond 宏接受任意多个 谓语/结果 表达式的组合，一旦有匹配上的，则执行相应的表达式。(Elixir 中有相同的用法)

    (cond
      (instance? String temperature) "invalid temperature"
      (<= temperature 0) "freezing"
      (>= temperature 100) "boiling"
      true "neither")

相当于 if...else if...else if...else
