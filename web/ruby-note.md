# Ruby Note

## Note 1

Note for 《Programing Ruby》第二版。

2016/8/1，花了半天读完了《Programming Ruby》第2版，是以 Ruby 1.8 版为例讲解的。

因为有了 c/c++/java/swift/js/python 等语言的基础，理解起来就快多了。也从 ruby 中领悟了其它语言的一些设计，比如，从 ruby 的 block 中领悟了 js 中用 yield 实现 generator，也可以说和协程很相似。

前段时间看了 swift，可以看出，swift 和 ruby 也是很多相通之处的。

语言之间是可以融汇贯通的。

ruby 语言的特性：

1. 动态语言，和 js/python 一样，定义变量时不需要指定类型，实际变量的类型在运行是可以变化的，不是强类型。
1. 强大的 block。
1. 方法调用可以不用加括号。

我是如何理解 ruby 中的符号，symbol。其实可以把它理解为枚举值，全局的枚举值，也可以理解为类似指针。他们的特点是程序运行期间全局唯一，可以把它的值想像成是内存地址，内存地址是不会冲突的。

ruby 没有 interface，只有比 interface 更强大的 module 与 mixin 机制 (python 也没有，js 也没有，动态语言是不是都缺这个?? interface 和 多态是强类型语言的特性，而动态语言因为依靠鸭子类型而不需要 interface 和多态)。

ruby 的 module 相当于有逻辑实现的 interface，类使用 include 关键字将 module 包含进来，相当于 java 中的 implement。ruby 的类还可以 extend module，表示包含进来的 module 方法是作为该类的类方法，而不是实例方法。

ruby 的 module 的另一个作用是生成命名空间。

2017/5/31 重新阅读。不会是很仔细的阅读，已经熟悉的就跳过。这本书应该是一本案头书。

### 第一部分 - Ruby 面面观

#### 第 1 章 - 入门

ruby 安装，略。

#### 第 2 章 - Ruby.new

ruby 基本知识。

`$name` 全局变量，`@name` 实例变量，`@@name` 类变量。

方法名可以以 `?` `!` `=` 结尾。(这是与一般语言不同的地方)

`@w{ant bee cat dog elk}` -> `['ant', 'bee', 'cat', 'dog', 'elk']`，生成数组的便捷方法。

控制结构，略。

正则表达式，`=~`，`match()`，`sub()`，`gsub()`。`sub` 表示替换第一个匹配的值，`gsub` 表示替换所有匹配的值。

    line.gsub(/Perl|Python/, 'Ruby')

Block 和迭代器，略。

读/写文件，通过 `gets` 函数从标准输入流中读取下一行。

    line = gets
    puts line

#### 第 3 章 - 类、对象和变量

没有很特别的内容，大部分内容略。

类变量必须显式地初始化。

单例的写法：

    class MyLogger
      private_class_method :new
      @@logger = nil
      def self.create
        @@logger = new unless @@logger
        @@logger
      end
    end

使用 `private_class_method` 将 new 方法标记为 private。只允许通过 create 类方法创建对象。

变量保存对象引用。

#### 第 4 章 - 容器、Blocks 和迭代器

Ruby 中的容器主要两大类：数组 Array 和散列表 Hash。

数组：支持负数索引，支持 range 索引。

    a = [1, 3, 5, 7, 9]
    a[-1]      -> 9
    a[1, 3]    -> [3, 5, 7]
    a[2..3]    -> [5, 7, 9]
    a[2...3]   -> [5, 7]

散列，略。

实现一个 SongList 容器，略。

Blocks 可以作为闭包，它包括了定义 block 时的上下文。

容器、block 和迭代器是 Ruby 的核心概念。

#### 第 5 章 - 标准类型

数字、字符串、区间、正则表达式。

数字：整数 (Fixnum/Bignum)，浮点数。

字符串：单引号，双引号。

操作字符串：String 的一些方法，split，chomp，squeeze，scan ...

区间：Range，就是一种数据类型，支持这种类型的现代语言越来越多了，比如 swift。

区间作为序列：`1..10`，`'a'..'z'`，`0...3`。

区间作为条件，这个很神奇啊：

    while line = gets
      puts line if line =~ /start/ .. line =~ /end/
    end

这段代码将打印从标准输入得到的行的集合，每组的第一行包含 start 这个词，最后一行包含 end 这个词。

当区间作为条件的逻辑是这样的，当区间的第一部分的条件为 true 时，它们就打开，当区间的第二部分的条件为 true 时，它们就关闭。

区间作为间隔：

    (1..10) === 5  -> true

**正则表达式**

暂略，回头再用到时回来补充。

#### 第 6 章 - 关于方法的更多细节

可变长度的参数列表：在最后一个参数前加一个 `*` 号即可，内部这些参数将被 array 接收。

最后一个参数如果前缀为 `&`，那么所关联的 block 会被转换成一个 Proc 对象，然后赋值给这个参数。这样做的目的在于，proc 对象可以保存起来在将来使用。

#### 第 7 章 - 表达式

Ruby 和其它语言的一个不同之处就是任何东西都能返回一个值，几乎所有的东西都是表达式。

Ruby 中的许多运算符是由方法调用来实现的，比如 `a*b+c`，等价于 `(a.*(b)).+(c)`

用反引号包围或 %x 为前缀的字符串，会被作为底层 shell 命令执行并返回，返回值就是该命令的标准输出，命令的退出状态保存在全局变量 `$?` 中。

    2.3.0 :001 > `date`
     => "Wed May 31 16:25:02 CST 2017\n"

Ruby 的赋值有 2 种基本形式。第一种是将一个对象引用赋值给变量或常量，这种形式的赋值在 Ruby 中是直接执行的 (hardwired)。

    instrument = "piano"

第二种形式，等号左边是对象属性或者元素的引用。

    song.duration = 234
    instrument["ano"] = "ccolo"

这种形式实际是通过调用左值的方法来实现的，这意味着你可以重载它们。

    class Song
      def duration=(new_val) # 第二种形式的赋值方法
        @duration = new_val  # 第一种形式的赋值方法
      end
    end

并行赋值：

    a, b = b, a

**7.4 条件执行**

布尔表达式。Ruby 对真值的定义很简单，任何不是 nil 或 false 的值都为真，数字 0 也是真，这和一般语言不一样，因为 0 在 ruby 中也是一个真实存在的对象。

`defined?()` 方法，判断参数是否定义，如果定义，返回对参数的描述，否则返回 nil。

if / unless

**7.5 case 表达式**

case 支持各种匹配：数字，字符串，正则，区间，类的继承关系 ...

**7.6 循环**

循环，迭代器。break / redo / next，retry。

**7.7 变量作用域、循环和 Blocks**

while / until / for 内建到了 ruby 语言中，但没有引入新的作用域：前面已存在的局部变量可以在循环中使用，而循环中新创建的局部变量也可以在循环后使用。(原来如此，我之前就一直在想，为什么我在 if/else 中定义的变量，出了 if/else 之后为什么还能用。这个和 java/c/c++ 还是不一样，在 c 中，用 `{}` 包围起来的一段代码就是一个单独的作用域)

被迭代器使用的 block，在这些 block 中创建的局部变量，外部无法访问，而 block 可以访问在 block 外部创建的局部变量。

#### 第 8 章 - 异常、捕获和抛出

begin...rescue...else...ensure...end

catch...throw

raise

#### 第 9 章 - 模块 (Module)

模块是一种将方法、类和常量组织在一起的方式，它提供两个主要的好处：

1. 模块提供了命名空间来防止命名冲突
1. 模块实现了 mixin 功能

module 中的实例变量与 class 中的实例变量有冲突的风险。

包含其它文件：load / require，前者每次 load 时都重新加载，而后者多次 require，实际只加载一次。

#### 第 10 章 - 基本输入与输出

Ruby 提供了两套操作 IO 的接口：

1. Kernel 模块，相关方法：gets / open / print / printf / putc / puts / readline / readlines ...
1. IO 对象，基类 IO 类，子类有 File 类，BasicSocket 类 ...

**10.2 文件打开和关闭**

    File.open("testfile", "r") do |file|
      # ...
    end

block 退出时，file 会自动关闭。

**10.3 文件读写**

Kernel 模块中定义的方法基本都可以用于 IO 对象：

    File.open("testfile") do |file|
      while line = file.gets
        puts line
      end
    end

但 IO 对象还有一组额外的访问方法，使用起来更简单，即迭代器：

    File.open("testfile") do |file|
      file.each_line("e") { |line| puts line }
    end

    IO.foreach("testfile") { |line| puts line }

写文件：?? (这一节哪里是在说写文件啊)

**10.4 谈谈网络**

Ruby 提供了套接字库来直接访问 tcp / udp / sock 等。在较高层次，提供了 lib/net 库来处理应用层协议。还可以通过 open-uri 库，直接使用 Kernel.open 方法打开一个 uri。

#### 第 11 章 - 线程和进程

这一章基本是把多线程的知识又温习了一遍。这是 ruby 对系统底层多线程的封装。

回头把 APUE 中这一部分内容再看一下。

创建线程：

    for page_to_fetch in pages
      threads << Thread.new(page_to_fetch) do |url|
        h = Net::HTTP::new(url, 80)
        puts "Fetching: #{url}"
        resp = h.get('/', nil)
        puts "Got #{url}: #{resp.message}"
      end
    end
    threads.each { |thr| thr.join }

使用 `Thread.new` 创建线程，后面跟一个 block 执行线程要做的事情。使用 `thread.join` 等待线程结束。

`Thread.current` 得到当前线程，`Thread.list` 得到一个所有线程的列表 ...

线程变量 (类似 windows 中的 TLS - Thread Local Storage)。线程 block 中定义的局部变量，其它线程不能共享。如果需要被访问，则需要借助线程变量。可以简单地把线程对象看作一个散列表，使用 `[]=` 写入元素，并使用 `[]` 读取。

    10.times do |i|
      threads[i] = Thread.new do
        sleep(rand(0.1))
        Thread.current["mycount"] = count
        count += 1
      end
    end
    threads.each { |t| t.join; print t["mycount"], "," }

线程和异常：如果 `abort_on_exception` 置为 false，那么未处理的异常只会杀死当前线程，否则杀死整个进程。

**11.2 控制线程调度器**

- `Thread.stop` 停止当前线程
- `thread.run` 安排运行特定线程
- `Thread.pass` 把当前线程调度出去，允许别的线程运行
- `thread.join` `thread.value` 挂起调用它们的线程，直接指定的线程结束为止

**11.3 互斥**

临界区，监视器，条件变量 ...

消费者与生产者模型，等待条件变量和对条件变量发信号。

    plays_pending = playlist.new_cond
    plays_pending.wait_while
    plays_pending.signal

**11.4 运行多个进程**

衍生新进程的几种方式：

1. system 命令：`system("tar xzf test.tgz")`
1. 反引号命令，其实和 system 命令是一样的
1. `IO.popen` 方法，可以和子进程通信

独立子进程：fork

关于多线程为什么比单线程跑得快(一般来说)，我又有了新的理解：

1. 如果是多核 CPU，多个线程将被分配到多个核上同时运行。
1. 对于单核 CPU，虽然是多线程，但同一时刻仍然只有一个线程在运行，其余线程处于空闲状态，那它跟单线程又有什么区别呢，我以前的理解是，它可以增大这个进程在整个系统所有进程中的 CPU 周期占比，比如说原来系统中有 10 个进程在跑，每个都是单线程，线程调度是平均的，那么这个进程的 CPU 周期占比是 10%，如果我把这个进程改成 5 个线程，那么它的占比就增加到了 36%。实际这根本不是主要原因，你想啊，实际在一个系统中，线程数成百上千，你增加的这几个线程跟总数一比，简直是九牛一毛。我现在的理解是，如果这些线程所做的事，仅仅是操作内存，那实际多线程跟单线程的效率是差不多的，但是如果一旦线程中涉及操作 IO，网络请求这种耗时操作，那么多线程的优势就很明显了。我们举个爬虫的例子，假设我们要爬 5 个网页，如果用单线程做，每爬一个网页，我们就要等一次网络请求的响应，总共要等 5 次，但如果我们用 5 个线程来做，当第一个线程进行到网络请求时，它阻塞等待，系统会马上切到第二个线程，当第二个线程阻塞时，系统继续切换线程，或许当第五个线程开始网络请求时，第一个线程和第二个线程的网络请求就已经回来了，这样我们等待网络请求的时间肯定是小于单线程下依次执行 5 次的时间的。当然，现在哪还有单核 CPU 啊，除非在嵌入式领域。

#### 第 12 章 - 单元测试

- 使用 Test::Unit 框架，使用时通过 `require 'test/unit'` 加载此框架。
- 测试用例 (test case) 必须继承自 Test::Unit::TestCase。
- 测试方法必须以 `test_` 开头。
- `setup` 和 `teardown` 方法将在每一个测试方法之前和之后执行。
- 使用 `assert_` 系列断言方法。

示例：

    require 'test/unit'
    require 'playlist_builder'
    require 'dbi'

    class TestPlaylistBuilder < Test::Unit::TestCase
      def setup
        @db = DBI.connect('DBI:mysql:playlist')
        @pb = PlaylistBuilder.new(@db)
      end

      def teardown
        @db.disconnect
      end

      def test_empty_playlist
        assert_equal([], @pb.playlist)
      end

      #...
    end

运行测试：

    $ ruby test_playlist_builder.rb

只执行其中某一个测试方法：

    $ ruby test_playlist_builder --name test_empty_playlist

#### 第 13 章 - 遇到问题时 (TroubleShooting)

ruby 提供了调试器，benchmark，profiler 的支持。

### 第二部分 - Ruby 与其环境

#### 第 14 章 - Ruby 和 Ruby 世界

ruby 的命令行参数：略，太多了，记住几个常用的就行，需要时再翻书。

命令行参数在代码中通过 ARGV 读取。

在代码中通过 ENV 读取环境变量，也可以在代码中往 ENV 中写入值，但只影响当前进程。

其它略。

#### 第 15 章 - 交互式 Ruby Shell (irb)

    $ irb
    2.3.2 :001 > ...

可以在 irb 中 load 一个文件：`load 'code/fib_up_to.rb'`

irb 就是像是 bash shell 一样，在 irb 中再运行 irb，将进入子会话，irb 支持多个、并发的会话，但当前会话只有一个，其余处理休眠状态，使用 `jobs` 命令列出所有 irb 会话，输入 fg 则激活一个特定的会话。

    > irb
    2.3.2 :001 > irb 'haha'
    2.3.2 :001 > jobs
    => #0->irb on main (#<Thread:0x007fa75c87f428>: stop)
    #1->irb#1 on haha (#<Thread:0x007fa75c980110>: running)
    2.3.2 :005 > fg 0
    => #<IRB::Irb: @context=#<IRB::Context:0x007fa75ca0f040>, @signal_status=:IN_EVAL, @scanner=#<RubyLex:0x007fa75ca05658>>
    2.3.2 :002 > jobs
    => #0->irb on main (#<Thread:0x007fa75c87f428>: running)
    #1->irb#1 on haha (#<Thread:0x007fa75c980110>: stop)

子会话与绑定。如果创建子会话时指定了一个对象，如上例中 `irb 'haha'`，它会成为绑定中的 self 值，没有指明接收者的方法会被该对象执行。

    2.3.2 :006 > irb 'test'
    2.3.2 :001 > upcase
    => "TEST"

就像是 bash shell 有相应的配置文件 `.bashrc`，irb 也有，它按以下顺序查找：`~/.irbrc`、`.irbrc`、`irb.rc`、`$irbrc`。此配置文件中放的是 ruby 代码，可以配置 irb shell 的样式等。

扩展 irb。在 irbrc 中定义的方法，可以在 irb 中执行。

irb 的配置选项，略，太多，需要时再看。

irb 的命令，常见的有 exit / conf / jobs / fg / kill ...

配置提示符，用来修改 shell 样式，略。

其它略。

#### 第 16 章 - 文档化 Ruby

RDoc，暂略，等自己写库的时候再看。

#### 第 17 章 - 用 RubyGems 进行包的管理

已了解，暂略，需要发布 Gem 时再回头仔细看。

#### 第 18 章 - Ruby 与 Web

**18.1 编写 CGI 脚本**

了解即可，毕竟太底层了，除非自己写框架，否则你永远都不会用到这部分内容的。

CGI 类提供了编写 cgi 脚本的支持，使用它，你可以更方便的操作表单、cookie、session 和环境等。

erb，在 html 中嵌入 ruby 是非常强大的概念，它基本上提供了同 asp、jsp 或 php 对等的工具。

操作 cookie / session，略。

**18.3 提升性能**

默认的 cgi 脚本，每来一个请求，就会启动一个新的解释器进程，很浪费内存，也使得访问很慢。

对于 apache web 服务器来说，它通过支持加载的模块解决这个问题。一般来说，这些模块是动态加载的，并成为 web 服务器运行进程的一部分，你不必一次又一次地衍生解释器进程来响应请求，web 服务器便是解释器。(大悟！原来如此，以前看 python 的时候，说要借助一个 `mod_python` 的东西，当时是不理解的。) 

`mod_ruby`，它是 apache 的一个模块，将一个完整的 ruby 解释器链接到 apache web 服务器本身。

另外，使用 FastCGI 协议也可以解决部分问题，对所有 cgi 类型的程序都适用，它使用了一个非常简单的代理程序，通常作为 apache 的一个模块，当请求到达时，这个请求会将它们转发到一个特定的、始终运行的进程，它像普通的 CGI 脚本那样进行响应，结果会返回给代理，然后发送回浏览器。(soga，明白了，这就涉及了进程间通信，另外这是不是就是所谓的反向代理，ngnix 是不是可以取代这个东西?)

**18.4 Web 服务器的选择**

除了 apache web 服务器，ruby 1.8 之后捆绑了 WEBrick，一个灵活的、纯 Ruby 的 http 服务器工具。(node 也自带 http 服务器，但 php 印象中一直是需要借助 apache 的)。

**18.5 SOAP 及 Web Services**

SOAP 是一种 RPC，数据传输一般用 XML 格式。它最终实现的效果就是，就像是调用本地的一个方法一样来调用远程服务器上的一个方法。

(终于明白了，但是一直有的疑惑是，为什么不用 rest api 来实现呢? 当问出这个问题，心中又有所明白，rest api 并不适合所有场景，它适合用来表现资源，且多是数据库存储的资源，且 url 的格式有约束，总之局限性比较大，而 SOAP 则没有约束，且它调用的方法并不一定是用来操作数据库的，举个例子，比如把一段文字送到服务器加密再返回来，这个纯粹是内存操作，而且也与资源无关，这个用 rest api 怎么来表达，不好表达嘛，而用 SOAP 表达起来就觉得很顺其自然)。

(其实，这么说起来，SOAP 和 GraphQL 某种形式上也有点相似呢，比如应该都是用 post 请求，传输的数据全部放在 body 里，即没有多个 key-value 对，只是前者是用 xml 来存放数据，后者用类 json 的 graph 语法，在服务端自己解析数据。)

#### 第 19 章 - Ruby Tk

略。这年头还用这玩意来写 UI，那是疯了吧。

#### 第 20 章 - Ruby 和 Windows

略。windows?? bye-bye!

#### 第 21 章 - Ruby 扩展

略，用 C 来扩展 Ruby，高级内容，暂时用不上。

### 第三部分 - Ruby 的核心

#### 第 22 章 - Ruby 语言

这一章是前面内容的温习，查缺补漏。

具体内容略，可作为参考手册查阅。这一章有讲到 block / proc / lambda 的内容，但担心内容已过时，毕竟是 ruby 1.8，现在是 ruby 2.4 了，这部分内容在《Ruby 元编程》看吧。

#### 第 23 章 - Duck Typing

(怎么说呢，这一章在强调动态类型相比静态类型语言的好处，从语言层面上来说，确实没有太多劣势，但抛开语言层面，说到重构，动态语言的重构基本只能靠全文搜索，替换了，全得是非常小心翼翼，这就是一个极大的劣势了，这就是一个最大的非语言因素。所以人们常说，"动态一时爽，重构火葬场"。)

在 Ruby 中，类不是类型，相反，对象类型更多是根据对象能够做什么决定的。在 Ruby 中，它被称为 duck typing。如果对象能够像鸭子那样行走，像鸭子那样呱呱叫的话，那么解释器会很高兴地把它当成鸭子来对待的。

如果你想使用 duck typing 哲学来编写代码，只需要记住一件事情：对象的类型是根据它能够做什么而不是根据它的类决定的。

一个例子，当需要对参数进行检查了，如果你用静态语言的思维习惯来写，你可能会对参数进行类型判断，而如果用 duck typing 思维来写，你会判断参数是否能够响应某些方法，比如：

    def append_song(result, song)
      unelss result.respond_to?(:<<)
        fail TypeError.new("'result' needs '<<' capcability")
      end
      unless song.respond_to?(:artist) && song.respond_to?(:title)
        fail TypeError.new("'song' needs 'artist' and 'title'")
      end

      result << song.title << " " << song.artist
    end

**23.3 转换**

- `to_i` `to_s`
- `to_int` `to_str`
- `to_arr` `to_hash` `to_io` `to_proc` `to_sym`

#### 第 24 章 - 类和对象

这一章算是核心内容之一了，但翻译得并不好。这应该也是《Ruby 元编程》的核心内容。

通篇看下来，Ruby 的类/对象模型和 JavaScript 的原型链有点相似。

与静态语言的不同之一，静态语言，类定义是在编译期处理的：编译期创建符号表，计算出分配多少空间，构造分发表 (dispatch table)，以及其它。而 Ruby，**类和模块的定义是可执行的代码**，虽然是在编译期进行解析，但当遇到定义时，类和模块是在运行时创建的，方法同样也是 (你可以在运行时根据不同条件，创建出不同的方法来)，这可以让你可以比传统语言更动态地构架你的程序。

    module Tracing
      # ...
    end

    class MediaPlayer
      include Tracing if $DEBUG

      # 通过这段代码，体会类、模块、方法的定义是可执行的代码
      if ::EXPORT_VERSION
        def decrypt(stream)
          raise 'Decryption not availabel'
        end
      else
        def decrpty(stream)
          # ...
        end
      end
    end

与静态语言的不同之二，类，比如 String 类，它被对象，比如 `s = "haha"` 的 s 所引用，而它自己也是对象，它是 Class 这个类 (shit，这就是英文原文也会把人绕晕啊) 的对象。

所以可以这么理解，有一个特殊的类叫 Class，其它一切的类 (包括 module) 都是它的对象，比如 String，Object，自定义的类如 `class Guitar` 等。当你在使用 `class Guitar` 这样的语句定义一个类时，正如上面所说的，这部分代码是运行时执行，它实际是在生成一个叫 Guitar 的 Class 对象。

所以，在 Ruby 中，有两种对象，一种是用 `class ClassName` 或 `module ModuleName` 定义的对象，它们是 Class 的对象，所以这些对象都包括以下属性：

- flags：
- super：指向父类对象，比如 String 对象的 super，是指向 Object 对象
- iv_tbl：指向 include 的 module ??
- klass：指向 Class ?? (不对，指向虚拟类，虚拟类里存放了类方法)
- methods：定义的方法，应该是个数组

另一种是通过 Class 对象 new 出来，或者通过字面值赋值的对象，比如：

    lucille = Guitar.new
    s = "hello" # 实际等同于 s = String.new("hello")

这种对象包括以下属性：

- flags：
- iv_tbl：
- klass：被谁 new 出来的，就指向谁，比如上例中，lucille 被 Guitar 对象 new 出来，那 klass 就指向 Guitar 对象

相比 Class 对象，它没有 super 和 methods 属性，也在情理之中。

所以在 Ruby 中，首先通过 `class ClassName` 创建出各种 Class 对象，再通过 Class 对象的 new 方法创建出各种 Class 对象的对象，有种自举的感觉，在 Ruby 中，Class 是火种，是上帝，是起源，不像在静态语言中，一般最基类是 Object，在 Ruby 中，Object 也是 Class 对象。

现在对动态语言的理解有一种耳目一新的感觉，这部分内容也有助于理解 JavaScript 的原型链。

**24.1 类和对象是如何交互的**

部分内容已经总结在上面的。

Ruby 允许创建一个和特定对象绑定的匿名类，有两种写法。

一种是 `class <<obj` 写法：

    a = "hello"
    b = a.dup

    class <<a
      def to_s
        "The value is '#{self}'"
      end
      def two_times
        self + self
      end
    end

    a.to_s       # "The value is 'hello'"
    a.two_times  # "hellohello"
    b.to_s       # hello

 另一种写法，效果和上面是一样的 (这种写法不好，但还是要理解，只是为了看到这种代码时不会懞逼)：

    a = "hello"
    def a.to_s
      "The value is '#{self}'"
    end
    def a.two_times
      self + self
    end

这两种写法的效果都是，生成了一个虚拟类对象，插在了对象 a 和对象 String 之间，原来对象 a 的 klass 指向 String 对象，现在，a 对象的 klass 指向这个虚拟类对象，而这个虚拟类对象的 super 指向 String 对象。(没有图，自己体会一下，真想把书上的图复制粘贴过来，这样一看图就理解了)。

扩展对象，通过 `obj.extend(Module)` 方法来将 module 中的方法添加到对象中，它基本等价于：

    class <<obj
      include Module
    end

但是，如果你在一个类中使用 `extend Module`，那么模块的方法会变成类方法。

Mixin 模块，在类中 include 一个 module 时，模块的实例方法就变成类的实例方法，就好像模块变成了类的超类，这正是它的工作方式。当你包含一个模块时，Ruby 会创建一个指向该模块的匿名代理类，并将这个代理插入到实施包含的类中作为其直接超类，代理类包含有指向模块实例变量和实例方法的引用。(看书上的图，非常形象。疑问，如果一个类 include 了多个 module 呢，这个图会变成怎样? 如果 include 了多个 module，它们会按顺序插入到继承链中)

**24.2 类和模块的定义**

剩余内容不是太明白，暂略。《Ruby 元编程》应该还会讲到。

#### 第 25 章 - Ruby 安全

所有外部数据都是有危险的，不要让它们靠近那些可能改动你的系统的接口。

Ruby 为减少这种危险提供了支持，所有来自外部的数据都可以被标记为**被污染的 (tainted)**，当运行在安全模式下时，传递被污染的对象给一个具有潜在威胁的方法会引发 SecurityError。

其余略。

#### 第 26 章 - 反射，ObjectSpace 和分布式 Ruby

能够内省 (introspect) 是 Ruby 等动态语言的诸多优点之一，那就是在程序内部自己检验程序的方方面面，Java 把这个特性称之为反射 (reflection)，而 Ruby 的内省比 Java 的反射强大很多。

内省可以做到：

- 包括哪些对象
- 类的层次结构
- 对象的属性和方法
- 有关方法的信息

**26.1 看看对象**

遍历当前进程中所有现存对象，使用 `ObjectSpace.each_object` 方法：

    2.3.0 :008 > ObjectSpace.each_object(Numeric) { |x| p x }
    (0+1i)
    9223372036854775807
    279535655170462513955024678816840110169
    NaN
    Infinity
    1.7976931348623157e+308
    2.2250738585072014e-308

查看对象的方法：`obj.methods`

查看对象是否支持某个方法：

    obj.respond_to?("method_name")
    obj.respond_to?(:method_name)

**26.2 考察类**

`superclass` 查看父类，`ancestors` 查看父类和 mixin 的模块。

类似的方法还是 `private_instance_methods`、`class_variables`、`instance_variables`、`constants`，不一而足，用于查看类、模块或对象的方法，变量或常量等。

**26.3 动态地调用方法**

方法一，使用 send 方法

    "Ruby".send(:length) # -> 4
    "Java".send("sub", /Java/, "Ruby") # -> "Ruby"

方法二，使用 method 的 call 方法，类似 c 中的函数指针

    trane = "Ruby".method(:length)
    trane.call # -> 4

方法三，使用 eval 方法

    trane - %q{"Ruby".length}
    eval trane # -> 4

性能，eval 最慢，send 和 method.call 差不多。

**26.4 系统钩子**

即一种 hook 技术。使用 `alias_method` 为原来的方法创建别名，然后重新定义此方法，在此方法中调用原来的方法。如下例所示，为创建的对象增加时间戳。

    class Object
      attr_accessor :timestamp
    end

    class Class  # !! 看，Class 现身了
      alias_method :old_new, :new
      def new(*args)
        result = old_new(*args)
        result.timestamp = Time.now
        result
      end
    end

但是要小心陷入无限循环中。

另外，Ruby 还提供了一些回调方法来跟踪某些事件 (我怎么觉得 Ruby 有点过度设计了呀，有必要搞得这么无所不能吗?)，比如，在添加 module 时会调用 module 的 `method_added` 回调方法。

- `method_added`：添加 module 的实例方法时调用
- `method_removed`：删除 module 的实例方法时调用
- ... 其余略

**26.5 跟踪程序的执行**

- `set_trace_func`
- caller：当前堆栈
- `__FILE__`：当前源文件名

其余略。

**26.6 列集和分布式 Ruby**

列集 (marshaling，第一次听见这种翻译)，其余就相当于 Java 中的序列化和反序列化。用 `Marshal.dump` 进行序列化，用 `Marshal.load` 进行反序列化。如果希望一个类可以被序列化和反序列化，那么需要实现 `marshal_dump` 和 `marshal_load` 方法。

默认 marshal 使用二进制存储，也可以选用其它方式，比如 JSON 和 YAML。YAML 库提供了 `YAML.dump(obj)` 和 `YAML.load(data)` 方法来将对象存储为 YAML 格式和从 YAML 格式中加载对象。`YAML.dump(obj)` 中的这个 obj 必须实现 `to_yaml_properties` 方法。

分布式 Ruby，嗯，就是实现远程调用啦，web 语言都能做，没什么特别。

使用 Ruby 要记住一件重要事情是：Ruby 的 "编译时" 和 "运行时" 几乎没什么区别。

### 第四部分 - Ruby 库

略。

----

## Note 2

Note for 《Ruby 元编程》第二版。

----

## Note 3

### `attr_reader / attr_writer / attr_accessor`

Ruby 的类的成员变量的定义方式是 `@variable`，但是在类外部，我们无法通过 `instance.@variable` 的方式访问它。这种机制决定了 ruby 的类成员变量只能是私有的，如果想从外部访问它，就必须为它定义公开的方法，或者使用属性访问器。

属性访问器的原理是自动为这些成员变量生成公开方法。所以可见，对于 ruby 来说，对外公开的只有方法，没有属性。这个特性好啊，相比 Java 这种语言，在实现 Model 时，就会很犯难，到底是直接把属性都 public 了，还是 private + getter / setter，后者实在是啰嗦。

#### 1. 手动定义方法

    class Foo
      def initialize
          @valid = false
      end

      def valid?
          @valid
      end
    end

#### 2. 使用属性访问器

    class Foo
      attr_reader :name
      attr_writer :age
      attr_accessor :gender

      attr_reader :valid
      alias valid? valid

      def initialize(name)
        @name = name
        @valid = false
      end
    end

`attr_reader` 定义的成员变量，可以在外部被访问，但不能在外部被修改，只能在内部被修改。`attr_reader :name` 等价于：

    def name
      @name
    end
 
`attr_reader` 定义的成员变量在内部的访问方法：

    @name = 'bar'

如果在内部访问时不加 `@`，那么实际访问的是局部变量

    # 局部变量
    name = 'bar'

在外部访问的方式：

    f = Foo.new
    puts f.name

    f.name = 'foo' # ERROR! 不能被修改，不存在 f.name= 方法

`attr_writer` 定义的成员变量，可以在外部被修改，但不能在外部被访问。`attr_writer :age` 等价于：

    def age=(val)
      @age = val
    end

在外部被修改：

    f = Foo.new
    f.age = 20

    puts f.age # ERROR! 不能被访问，不在存 f.age 方法

使用 `attr_writer` 定义的成员变量，在内部被修改时，有 2 种使用方式：

    @age = 40
    # or
    self.age = 40

    # 局部变量
    age = 40

`attr_accessor` 定义的成员变量，既可以在外部被访问，也可以被修改。`attr_accessor :gender` 等价于：

    def gender
      @gender
    end
    
    def gender=(val)
      @gender = val
    end

对于 boolean 型的成员变量，我们习惯于在它的访问方法名中加上 `?`，所以我们可以用 alias 关键字来定义一个别名。

    attr_reader :valid
    alias valid? valid

使用：

    f = Foo.new
    puts f.valid?

### method / block / proc / lambda

ruby-china 上有一篇帖子写得很详细：[聊聊 Ruby 中的 block, proc 和 lambda](https://ruby-china.org/topics/10414)，而且据说在《Ruby 元编程》一书对此也有详细解释，那之后就去看这本书。

简单地说来，block 就像是匿名的 proc，它会被自动转换成 proc 对象，而 lambda 就像是匿名的 method。在 block/proc 中 return，会导致调用它的方法也直接 return，因此，它们就像是一个块代码，inline 代码，直接内联到调用它的方法中，它们的作用域是和调用它们的方法一致的。而 lambda/method 则不同，它们有自己的作用域，在 lambda/method 中 return，只是结束了当前 lambda/method，而不会影响调用它们的方法。

另外一些微小的区别是，block 是依存于方法而存在的，它是方法的一部分，而不能单独存在，它不是对象，而 proc 可以。
