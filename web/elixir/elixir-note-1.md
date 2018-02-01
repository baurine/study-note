# Elixir - Note 1

Note for [Elixir School](https://elixirschool.com/cn/)

纯兴趣，做一下简单了解，不求全部理解。

- 基础
  1. 基础
  2. 集合
  3. Enum 模块
  4. 模式匹配
  5. 控制语句
  6. 函数
  7. 管道操作符
  8. 组合
  9. Mix
  10. 魔符(Sigil)
  11. 文档模块
  12. 测试
  13. 推导
  14. 字符串
  15. 自定义 Mix 任务
- 高级
  1. 和 Erlang 互操作
  3. 可执行文件
  4. 并发
  5. OTP 并发
  6. OTP Supervisors
  7. 元编程
  11. GenStage
- 专题
  1. Plug
  2. Ecto
  4. Erlang 项式存储 (ETS)

## 基础

Elixir 是一种动态的函数式编程语言...

(注意，Elixir 是函数式编程语言，不是面向对象的编程语言，只有函数，没有对象，所以不要问，咦，为什么没有 class 关键字啊...)

特性：

- 高并发
- 高容错性
- 函数式编程
- 可扩展

安装，略。

iex 命令进入交互模式。

基本数据类型：

- 整数
- 浮点数
- 布尔类型 - true / false
- 原子类型 - Ruby 中的符号，:foo, true 和 false 实际也是原子类型，即 :true, :false
- 字符串 - UTF-8 编码(?? 还是说 Unicode 编码)，双引号

基本操作：算术运算，布尔运算，比较，略。

字符串插值，和 Ruby 一样，字符串拼接，用 `<>` 替代常见的 `+` 号 (奇芭)。

### 集合

列表、元组、关键字列表 (keywords)、图 (maps) (?? 不是映射吗)、字典和函数组合子 (combinators)

#### 列表

    iex> [3.14, :pie, "Apple"]

内部是用链表实现的，所以获取长度是 O(n) 的操作，在头部插入最快。

列表拼接，使用 `++/2` 操作符。`++/2` 中的 2 表示元数，参数的个数。

    iex> [1, 2] ++ [3, 4, 1]
    [1, 2, 3, 4, 1]

减法用 `--/2` 操作符。

    iex> [1,2,2,3,2,3] -- [1,2,3,2]
    [2, 3]

**头/尾**

(注意，这个概念在函数式编程中非常重要)

使用列表时，经常要和头部、尾部打交道，列表的头部是第一个元素，尾部是除去第一个元素外剩下的元素。

Elixir 提供了 hd 和 tl 来获取这两个部分。

    iex> hd [3.14, :pie, "Apple"]
    3.14
    iex> tl [3.14, :pie, "Apple"]
    [:pie, "Apple"]

除了 hd 和 tl，还可以用模式匹配和 `|` 操作符来把一个列表分为头尾两部分。(模式匹配，Erlang 的核心)

    iex> [head | tail] = [3.14, :pie, "Apple"]
    [3.14, :pie, "Apple"]
    iex> head
    3.14
    iex> tail
    [:pie, "Apple"]

#### 元组

和列表类似，但在内存中连续存放，因此获取长度很快，但修改操作很昂贵，需要在内存中重新拷贝一份。(因为是函数数编程，所以你不能直接修改原来的对象)

    iex> {3.14, :pie, "Apple"}
    {3.14, :pie, "Apple"}

元组一般用来作为返回值。

    iex> File.read("path/to/existing/file")
    {:ok, "... contents ..."}
    iex> File.read("path/to/unknown/file")
    {:error, :enoent}

(上面的用法和 Go 很想似嘛)

#### 关键字列表

关键字列表，它首先是一个列表，但是是特殊的列表，它的每个元素必须是元组，而且是二元元组，元组中的第一个元素必须是原子类型。

它和列表的行为完全一致。

    iex> [{:foo, "bar"}, {:hello, "world"}]
    [foo: "bar", hello: "world"]

    # 简化写法
    iex> [foo: "bar", hello: "world"]
    [foo: "bar", hello: "world"]

关键字列表很重要，它的特性：

- key 都是原子类型
- key 是有序的
- key 并不唯一

#### 图

(其实是映射啦)

Elixir 的 maps 是键值对结构的第一选择，和关键字列表不同的是，map 允许任意类型作为 key，用 `%{}` 定义 map。

    iex> map = %{:foo => "bar", "hello" => :world}
    %{:foo => "bar", "hello" => :world}
    iex> map[:foo]
    "bar"
    iex> map["hello"]
    :world

如果 map 中的 key 都是原子类型，可以简化写法，使用关键字列表的写法。

    iex> %{foo: "bar", hello: "world"} == %{:foo => "bar", :hello => "world"}
    true

map 中的 key 是唯一的。

map 提供了自己更新和获取原子 key 的语法。

    iex> map = %{foo: "bar", hello: "world"}
    %{foo: "bar", hello: "world"}
    iex> %{map | foo: "baz"}
    %{foo: "baz", hello: "world"}
    iex> map.hello
    "world"

### Enum 模块

这个模块包含一些枚举集合元素的算法，超过 100 个函数。

看了一下，类似 Ruby 的 Array.map / Array.reduce 等，没错，就是这种用法。

**all?**

当集合中的所有元素满足条件时，返回 true，否则 false。

    iex> Enum.all?(["foo", "bar", "hello"], fn(s) -> String.length(s) == 3 end)
    false
    iex> Enum.all?(["foo", "bar", "hello"], fn(s) -> String.length(s) > 1 end)
    true

所以，函数式编程中是没有类、对象这种概念的是吧...否则，干嘛不把这个方法直接作用在集合对象上，你看，取字符串的长度，用的也是 String.length(s)，而不是 s.length 或 s.length()

any?，略。

`chunk_every/2`，把集合按每 2 个元素拆分，略。

还介绍了：`map_every`, each, map, min, max, reduce, sort, `uniq_by` ...

### 模式匹配

模式匹配是 Elixir 很强大的特性，它允许我们匹配简单值、数据结构、甚至函数。

Elixir 中，`=` 不再是赋值操作值，而是匹配操作符。

    iex> x = 1
    1
    iex> 1 = x
    1
    iex> 2 = x
    ** (MatchError) no match of right hand side value: 1

集合类型的匹配：

    # Lists
    iex> list = [1, 2, 3]
    iex> [1 | tail] = list
    [1, 2, 3]
    iex> tail
    [2, 3]
    iex> [2 | _] = list
    ** (MatchError) no match of right hand side value: [1, 2, 3]

    # Tuples
    iex> {:ok, value} = {:ok, "Successful!"}
    {:ok, "Successful!"}
    iex> value
    "Successful!"

(有点像解构)

`^` 操作符，当区配的左边包含变量时，区配操作符同时会做赋值操作，如果不想赋值，则用 `^` 禁止赋值。

    iex> x = 1
    1
    iex> ^x = 2
    ** (MatchError) no match of right hand side value: 2
    iex> {x, ^x} = {2, 1}
    {2, 1}
    iex> x
    2

模式匹配，有意思。

### 控制语句

if / unless / case / cond

if / unless 在 Elixir 中是宏定义，不是语言本身的语句。

Elixir 中唯一的假值是 nil 和 false。

    iex> if String.valid?("Hello") do
    ...>   "Valid string!"
    ...> else
    ...>   "Invalid string."
    ...> end
    "Valid string!"

如果需要匹配多个模式，可以使用 case。

    iex> case {:ok, "Hello World"} do
    ...>   {:ok, result} -> result
    ...>   {:error} -> "Uh oh!"
    ...>   _ -> "Catch all"
    ...> end
    "Hello World"

case 还支持卫兵表达式 (... 就是 guard expression 啦)，只有在条件成立的情况下，才匹配生效。

    iex> case {1, 2, 3} do
    ...>   {1, x, 3} when x > 0 ->
    ...>     "Will match"
    ...>   _ ->
    ...>     "Won't match"
    ...> end
    "Will match"

cond，当需要匹配条件而不是值的时候，用 cond，这和其它语言的 else if 类似。

    iex> cond do
    ...>   2 + 2 == 5 ->
    ...>     "This will not be true"
    ...>   2 * 2 == 3 ->
    ...>     "Nor this"
    ...>   1 + 1 == 2 ->
    ...>     "But this will"
    ...>   true -> "Catch all"
    ...> end
    "But this will"

在最后放置一个 true，防止没有任何条件匹配时导致的 crash。

### 函数

**匿名函数**

fn ... -> ... end

    iex> sum = fn (a, b) -> a + b end
    iex> sum.(2,3)

因为匿名函数在 Elixir 中太常见，用 `&` 操作符来简化。

    iex> sum = &(&1 + &2)
    iex> sum.(2, 3)
    5

参数用 `&1` `&2` `&3` 来获取。

另外，用 `.()` 的方法来调用参数，也是奇芭。(后面发现，好像只有匿名函数是这么用，命令函数和其它语言相同)

**模式匹配**

在 Elixir 中，模式匹配不仅限于变量，还可用于函数签名上，用来匹配函数参数。

    iex> handle_result = fn
    ...>   {:ok, result} -> IO.puts "Handling result..."
    ...>   {:error} -> IO.puts "An error has occurred!"
    ...> end

    iex> some_result = 1
    iex> handle_result.({:ok, some_result})
    Handling result...

    iex> handle_result.({:error})
    An error has occurred!

**命名函数**

用 `def...do...end` 定义，如果只有一行，则用 `def...do:`

    defmodule Greeter do
      def hello(name) do
        "Hello, " <> name
      end
    end

    iex> Greeter.hello("Sean")
    "Hello, Sean"

用模式匹配和命名函数实现递归：

    defmodule Length do
      def of([]), do: 0
      def of([_ | tail]), do: 1 + of(tail)
    end

    iex> Length.of []
    0
    iex> Length.of [1, 2, 3]
    3

**函数名字和元数**

函数名称方式由名字和元数组成：

    defmodule Greeter2 do
      def hello(), do: "Hello, anonymous person!"                  # hello/0
      def hello(name), do: "Hello, " <> name                       # hello/1
      def hello(name1, name2), do: "Hello, #{name1} and #{name2}"  # hello/2
    end

**私有函数**

用 defp 定义，只能在同一模块内使用。

    defmodule Greeter do
      def hello(name), do: phrase <> name
      defp phrase, do: "Hello, "
    end

**卫兵 (guard)**

在函数定义中使用 when

    defmodule Greeter do
      def hello(names) when is_list(names) do
        names
        |> Enum.join(", ")
        |> hello
      end

      def hello(name) when is_binary(name) do
        phrase() <> name
      end

      defp phrase, do: "Hello, "
    end

    iex> Greeter.hello ["Sean", "Steve"]
    "Hello, Sean, Steve"

**参数默认值**

使用 `argument \\ value` 的形式 (好奇芭啊)

    defmodule Greeter do
      def hello(name, language_code \\ "en") do
        phrase(language_code) <> name
      end

      defp phrase("en"), do: "Hello, "
      defp phrase("es"), do: "Hola, "
    end
