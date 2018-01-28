# Function Programming Note

(鉴于函数式编程语言集中在 Web 开发，比如 Lisp，Erlang, Elixir, Closure，故暂且把它放在 web 目录下。)

Note for [函数式编程圣经](https://mp.weixin.qq.com/s/0gErQ3tjDLZuD1bYOhi0mQ)

刘欣公众号上的一篇文章，写得很有趣，对于理解函数式编程很有帮助。

函数式编程最大的特点就是不变量 (Immutable)，一个变量 (实际都是常量)，一旦声明且赋值后，它的值就不能再改变了。

    define result = 0
    result = 1 // Wrong!

    define myList = [1, 2, 3]
    myList[0] = 10 // Wrong!

变量不能被修改，那这逻辑还怎么写啊，解决办法是，如果要修改一个变量的值，把新值赋值给一个新的变量，然后旧的变量就抛弃了，用新的变量。

第二个特点，函数应该是纯粹的，不能用副作用，因此它：

- 不能修改传递给函数的变量！
- 不能修改全局变量！
- 对于同样的输入参数，返回值总是相同的！

由于以上的特点，在非函数式编程语言中常见的的循环，在函数式编程语言中居然无法使用。

    // wrong! i 的值不能被修改
    for (i = 0; i < 100; i++) {
      print(i)
    }

    function sum(list) {
      define result = 0
      for (item in list) {
        // wrong! result 的值不能被修改
        result += item
      }
      return result
    }

因此，在函数式编程语言中，要用尾递归来替代循环。为了方便操作，对于 list，一般都会有一个 head 函数，来取得 list 中的第一个值，还有一个 tail 函数，来取得 list 余下的值。

以 Elixir 为例，用 hd 函数来取得第一个元素，用 tl 函数取得余下的值。

    iex(1)> hd [1, 2, 3]
    1
    iex(2)> tl [1, 2, 3]
    [2, 3]

另外，在 Elixir 中，可以方便地使用匹配来取得 head 和 tail：

    iex(1)> [h | t] = [1, 2, 3, 4]
    [1, 2, 3, 4]
    iex(2)> h
    1
    iex(3)> t
    [2, 3, 4]

尾递归实现 list 求和的伪代码：

    define myList = [1, 2, 3, 4]
    head(myList) // 1
    tail(myList) // [2, 3, 4]

    define sumList(list, result) {
      if ([] == list) {
        return result
      } else {
        return sumList(tail(list), result + first(list))
      }
    }
    sumList(myList, 0) // 10

第三个特点，高阶函数，函数是一等公民，可以作为参数传递。

常见的高阶函数，map / reduce / filter。

    define a = [1, 2, 3]
    map(function(x) {return x*2}, a) // [2, 4, 6]

变量不可变的特点，使用函数式编程语言天然的不需要共享变量，没有多线程竞争加锁的情况，天生为并发而生。

其它特点：柯里化 (Currying)，惰性求值 (比如说参数，如果函数体中没有用到这个参数，那么这个参数可以永远不用运算求值)，宏 (Macro) ...

一般函数式编程语言还有一个特点，运算符视同函数，因此放在所有参数之前。

    // 一般语言
    1 + 2

    // 函数式语言
    (+ 1 2)
