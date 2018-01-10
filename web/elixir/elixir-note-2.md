# Elixir - Note 2

Note for [Elixir School](https://elixirschool.com/cn/) - 续。

### 管道操作符

管道操作符 `|>` 把前面表达式的结果传递给后面的表达式作为第一个参数。

    foo(bar(baz(new_function(other_function()))))

    other_function() |> new_function() |> baz() |> bar() |> foo()

(nice!)

一些使用示例：

    iex> "Elixir rocks" |> String.upcase() |> String.split()
    ["ELIXIR", "ROCKS"]

    iex> "elixir" |> String.ends_with?("ixir")
    true

(如果元数大于 1，一定要用括号，元数为 1 时，可以省略括号)

### 组合

用模块来给函数进行分组，用结构体这种特殊字典来组织代码。

**模块**

用 defmodule 关键字，类似 Ruby 中的 module，允许嵌套。

    defmodule Example.Greetings do
    def morning(name) do
        "Good morning #{name}."
    end

    def evening(name) do
        "Good night #{name}."
    end
    end

    iex> Example.Greetings.morning "Sean"
    "Good morning Sean."

**模块属性**

通常作为模块的常量。

    defmodule Example do
      @greeting "Hello"

      def greeting(name) do
        ~s(#{@greeting} #{name}.)
      end
    end

`~s` 的用法后面会讲到。

**结构体**

结构体是字典的特殊形式，键是预定义的，一般都是默认值。只能定义在模块内部，且一个模块只定义一个结构体。使用 defstruct 关键字。

    defmodule Example.User do
      defstruct name: "Sean", roles: []
    end

    iex> %Example.User{}
    %Example.User{name: "Sean", roles: []}

这个结构体实际就是模块的成员。

可以像 map 那样更新结构体。

    iex> steve = %Example.User{name: "Steve", roles: [:admin, :owner]}
    %Example.User{name: "Steve", roles: [:admin, :owner]}
    iex> sean = %{steve | name: "Sean"}
    %Example.User{name: "Sean", roles: [:admin, :owner]}

结构体和 map 可以匹配。

    iex> %{name: "Sean"} = sean
    %Example.User{name: "Sean", roles: [:admin, :owner]}

### mix

mix 就是 Ruby 中的 bundler。

`mix help`，查看帮助。

`mix new project_name`，创建新项目，生成脚手架文件。其它 mix.exs 就和 Ruby 中的 Gemfile 类似，用来声明依赖包。

默认的 mix.exs 示例：

    defmodule Example.Mixfile do
      use Mix.Project

      def project do
        [
          app: :example,
          version: "0.1.0",
          elixir: "~> 1.5",
          start_permanent: Mix.env() == :prod,
          deps: deps()
        ]
      end

      def application do
        [
          extra_applications: [:logger]
        ]
      end

      defp deps do
        []
      end
    end

project 声明 app 名字，版字号等 meta 信息，deps 声明依赖。

`mix compile`，编译，结果生成在 `_build` 目录中。

和整个 mix 项目交互：`iex -S mix`，类似 Ruby 中的 `bin/rails console`，或者 `irb xxx.rb`。

管理依赖，把依赖写在 mix.exs 的 deps 中，用 `mix deps.get` 来下载依赖，相当于 Ruby 中的 `bundle install`。

多个环境：`:dev`, `:test`, `:prod`，可以从 `Mix.env` 变量中获取当前的环境，可以通过 `MIX_ENV=prod mix compile` 的用法设计环境变量，和 `RAILS_ENV=staging bin/rails c` 类似。

### 魔符 (Sigil)

Elixir 提供了魔符的语法糖来表示和处理字面量，一个魔符以~开头然后接上一个字符。(类似 Ruby 中的 `%i` `%w`)

- `~C` 创建一个不处理插值和转义字符的字符列表
- `~c` 创建一个处理插值和转义字符的字符列表
- `~R` 创建一个不处理插值和转义字符的正则表达式
- `~r` 创建一个处理插值和转义字符的正则表达式
- `~S` 创建一个不处理插值和转义字符的字符列表
- `~s` 创建一个处理插值和转义字符的字符列表
- `~W` 创建一个不处理插值和转义字符的单词列表
- `~w` 创建一个处理插值和转义字符的单词列表
- `~N` 创建一个 NaiveDateTime 格式的数据结构

大写不处理插值，小写处理插值。

可用的分隔符有：`<>`, `{}`, `[]`, `()`, `||`, `//`, `""`, `''`

### 文档模块

Elixir 把文档 (即注释) 看作一等公民，提供大量函数来处理文档。

- `#` - 用于单行注释
- `@moduledoc` - 用于模块文档的注释，放在 defmodule 后的第一行
- `@doc` - 用于函数的注释，放在函数前

可以用 Markdown 来写文档，ExDoc 工具用来自动生成在线 HTML 文档。

### 测试

Elixir 自带测试框架 ExUnit。

使用 test 方法进行测试，assert 进行断言，和 Ruby 大同不异。

Elixir 不推荐 mock (为什么?)。

详略。

### 推导

(貌似和 Python 的列表表达式是一个意思)

列表推导是循环遍历枚举值的语法糖。

**基础**

使用 `for...do...` 语法，for 后面跟的是生成器表达式。

    iex> list = [1, 2, 3, 4, 5]
    iex> for x <- list, do: x*x
    [1, 4, 9, 16, 25]

可用于任何可遍历类型。

    # 关键字列表
    iex> for {_key, val} <- [one: 1, two: 2, three: 3], do: val
    [1, 2, 3]
    # 字典
    iex> for {k, v} <- %{"a" => "A", "b" => "B"}, do: {k, v}
    [{"a", "A"}, {"b", "B"}]

生成器依靠模式匹配来工作，如果匹配失败，则跳过此次区配。

    iex> for {:ok, val} <- [ok: "Hello", error: "Unknown", ok: "World"], do: val
    ["Hello", "World"]

可使用多个生成器，形成多重循环。

    iex> list = [1, 2, 3, 4, 5]
    iex> for n <- list, times <- 1..n, do: IO.puts "#{n} - #{times}"

**过滤器**

在生成器表达式中加入条件判断。(嗯，没错，和 Python 的列表表达式是类似的)，可使用多个过滤器。

    import Integer
    iex> for x <- 1..100,
    ...>   is_even(x),
    ...>   rem(x, 3) == 0, do: x
    [6, 12, 18, 24, 30, 36, 42, 48, 54, 60, 66, 72, 78, 84, 90, 96]

**使用 `:into`**

列表推导式默认生成列表，如果不想生成列表，用 `:into` 指定生成其它类型，但必须是实现了 Collectable 协议的任何结构体。

    iex> for {k, v} <- [one: 1, two: 2, three: 3], into: %{}, do: {k, v}
    %{one: 1, three: 3, two: 2}

### 字符串

Elixir 字符串是字节序列。

    iex> string=<<104,101,108,108,111>>>
    “Hello”

`<<>>` 表示包含的是字节。

**字符列表**

Elixir 内部，字符串是字节序列表示，而不是字符数组 (区别?)，Elixir 也有字符列表的类型，用单引号括起来，而字符串是双引号。

- 字符列表，单引号，列表，内部是链表
- 字符串，双引号，序列

字符串常用而字符列表不常用。

**字素和字码点**

字码点 - codepoint 就是一个或多个字节表示的 unicode 字符，字符的最小单元，无法再切分。而有些字符实际是由多个 unicode 字符拼合而成，比如带声调的拉丁字符 (á, ñ, è)，这种称为字素 - graphemes。

字素由多个字码点组成。

    iex> string = "\u0061\u0301"
    "á"

    iex> String.codepoints string
    ["a", "́"]

    iex> String.graphemes string
    ["á"]

String.codepoints / String.graphemes

**字符串函数**

String 模块

length, replace, duplicate, split ....

### 自定义 Mix 任务

就是类似 Ruby 的 rake task。

    # hello/lib/mix/tasks/hello.ex
    defmodule Mix.Tasks.Hello do
      use Mix.Task

      @shortdoc "Simply runs the Hello.say/0 command."
      def run(_) do
        # 调用我们刚才创建的　Hello.say 函数
        Hello.say()
      end
    end

使用：

    $ mix hello

详略。

## 高级

### 和 Erlang 互操作

在 Elixir 直接使用 Erlang 标准库和第三方库。

Erlang 的模块在 Elixir 中用小写的原子变量表示，如 timer 模块的 tc 方法，在 Elixir 中这样用 `:timer.tc()`

### 可执行文件

使用 escript，详略。

### 并发

Elixir 的一大卖点就是对并发的支持。并发模型的基础是 Actors，通过消息传递来交互的进程 - Eralng VM 自己管理的轻量级进程，类似 Go 的 goroutine。

**进程**

创建一个新进程最简单的方法是 spawn。接受函数作为参数，返回 PID。

    defmodule Example do
      def add(a, b) do
        IO.puts(a+b)
      end
    end

    spawn(Example, :add, [2,3])

**消息传递**

用 send() 对进程发消息，在进程中用 receive() 监听和匹配消息。

**进程链接**

`spawn_link`

**进程监控**

`spawn_mointor`

**Agents**

没明白，先跳过。

**Tasks**

在后台执行一个函数。

    task = Task.async(Example, :double, [2000])
    Task.await(task)

### OTP 并发

看了几遍，总算是明白了。

Elixir 并发的底层机制 - OTP Behavior。包括两个东西 - GenServers，GenEvents。

**GenServers**

总的来说，你可以在一个实现了 GenServer 的模块中，定义 n 个 `handle_call` 函数，这些函数不能手动调用，而是在调用 GenServer.call() 时被自动调用，根据 call() 中的参数自动匹配相应的 `handle_call()` 函数，可能匹配到多个 `handle_call()` 函数。

`handle_call()` 是同步的，异步用 `handle_cast()` 定义，同时用 GenServer.cast() 调用。

代码暂略。

**GenEvents**

总体和 GenServer 差不多，也可以定义 `handle_call()` 函数，但主要是定义 `handle_event()` 来处理事件。

外界通过 `GenEvent.add_handler()` 来添加这些 handler，使用 notify() 给这些 handler 发消息，然后这些 handler 会调用它们的 `handle_event()` 方法。也可以直接用 call() 方法指定调用某个 handler 中的 `handle_call()` 方法。

代码暂略。

(`use GenEvent`, 这是什么用法，难道类似 Ruby 的 include？)

### OTP Supervisors

supervisors 是一种特殊的进程，专门用来监控其它的进程，它能够自动重启崩溃的子进程，从而编写容错性高的程序。

详略，需要时再看。

### 元编程

quote， unquote，defmacro，详略。先跳过。

### GenStage

有意思，我把这个理解成和 RxJava 思想是一样的。

生产者，生产者-消费者，消费者。

生产者，就是数据源。

生产者-消费者，就是中间的各种转换函数，map transformer。

消费者，就是最后的 subscriber。

## 专题

### Plug

Plug 类似 Ruby 中的 Rack，再加上一点 Sinatra，它提供了编写 Web 应用组件的一组规范。

Plug 就是用来写 Web 应用的，是一个 http server。

详略，回头练习一下。

### Ecto

Ecto，类似 Rails 中的 ActiveRecord，ORM，详略。需要时再看。

### ETS

Erlang Term Storage，OTP 中内置的一个功能强大的存储引擎。可以理解成内置的 Redis。

可以建表、插入数据、获取数据、删除数据。

DONE!
