# Python 基础教程 (第二版) 笔记 - Note 3

(第 14 章 - 第 19 章)

## 第 14 章 网络编程

(2014/09/20)

python 是一个很强大的网络编程工具。因为 python 提供了如此丰富的网络工具，所以对于 python 的网络编程只进行简要介绍。(?? 这是什么逻辑嘛，没关系，只需要简要介绍就行了，具体需要的话就自己去深入学习喽。)

### 14.1 介绍少数几个网络设计模块

在标准库中有很多网络设计模块，这里只介绍少数几个。

14.1.1 socket 模块

最底层的模块。略。已经熟得不能再熟，你还要再看?? 省点时间好不。

14.1.2 urllib 和 urllib2 模块 (!!! 功能最强大的网络模块)

它们能通过网络访问文件，就像访问本地文件一样。

这两个模块的功能差不多，但 urllib2 更好一些。如果只使用简单的下载，用 urllib 就行了；如果需要使用 http 验证，或者 cookie 等，用 urllib2 是个更好的选择。

1. 打开远程文件 (使用 urlopen)

        from urllib import urlopen
        webpage = urlopen('http://www.python.org')

   urlopen 返回的类文件对象支持 close, read, readline, readlines 方法，也支持迭代。

1. 获取远程文件 (使用 urlretrieve)

   urlretrieve 用来下载远程文件并保存在文件中。(我想其实也可以用 urlopen + file.write 来实现吧)

14.1.3 其它模块

太多了，略。你能想到的都有，比如操作邮件，ftp, telnet 等等。

### 14.2 SocketServer 和它的朋友们

(从名字就能看出这是用来写服务器端的，而上面介绍 urllib 和 urllib2 是用来写客户端的。)

SocketServer 是对 socket 模块的封装。它是标准库中很多服务器框架 (如 BaseHTTPServer, SimpleHTTPServer, CGIHTTPServer, SimpleXMLRPCServer, DocXMLRPCServer ...) 的基础。

SocketServer 包含 4 个基本的类：TCPServer, UDPServer, UnixStreamServer, UnixDatagramServer。其中后三者不常用。

(!!! 通用的服务器处理方法，大部分框架也是类似这么处理)

为了写一个使用 SocketServer 框架的服务器，大部分代码会在一个请求处理程序 (request handler) 中。每当服务器收到一个请求 (即来自客户端的连接) 时，就会实例化一个请求处理程序 (即 request handler)，并且它的各种处理方法 (handler methods) 会在处理请求时被调用。

具体调用哪个方法取决于特定的服务器和使用的处理程序类 (handler class)，这样可以把它们子类化，使得服务器调用自定义的处理程序集。

基本的 BaseRequestHandler 类把所有的操作都放到了处理器的一个叫 handle 的方法中，这个方法会被服务器调用。这个方法就会访问属性 self.request 中的客户端套接字。

如果使用的是流 (比如 TCPServer)，那么可以使用 StreamRequestHandler 类，创建了其它两个新的属性， self.rfile (用于读取) 和 self.wfile (用于写)。

一个简单的例子：

    from SocketServer import TCPServer, StreamRequestHandler
    class Handler(StreamRequestHandler):
      def handle(self):
        addr = self.request.getpeername()
        print 'Got connection from', addr
        self.wfile.write('Thank you for connection')

    server = TCPServer(('', 1234), Handler)
    server.serve_forever()

### 14.3 多连接 (即 APUE 中的多路复用)

前面的解决方法都是同步阻塞的，必须处理完一个连接才能处理下一个连接。如何同时处理多个连接。(==，好像说的也不是一回事，前者是指服务器端，后者指客户端... anyway，知道这么个意思就行了)。

三种方法：分叉 (forking)、多线程、以及异步 IO。(分叉，即 linux 上的多进程，在 windows 下无法使用)

异步 IO 的基础是 select 模块的 select 函数。标准库中用于处理异步 IO 的模块有 asyncore 和 asynchat (24 章讨论)。Twisted 是一个非溃强大的异步网络编程框架。(!!! 使用 select 和 epoll 的前提就是需要把 socket io 设置成 异步 模式 !!!)

14.3.1 使用 SocketServer 进行分叉和线程处理

略。很简单，和上面的例子相比，将自己定义的 Server 类同时继承自 (ForkingMixIn, TCPServer) 或 (ThreadingMixIn, TCPServer)。

14.3.2 带有 select 和 poll 的异步 IO (实践!)

select() 和 poll() 函数都属于 select 模块，但后者只能在 *unix 上使用。

select() 和 poll() 的使用示例：略。看书上。(实践!)

### 14.4 Twisted

Twisted 是一个事件驱动的 Python 网络框架，原来是为网络游戏开发的，现在被所有类型的网络软件使用。

在 Twisted 中，需要实现事件处理程序，这很像 GUI 工具包中做的那样。(嗯，其实 libjinle 也是需要实现事件处理程序!)

14.4.1 下载安装 Twisted

略。

14.4.2 编写 Twisted 服务器

嗯，看完了，确实跟 libjingle 的思路是一样的。集中精力编写事件处理程序，由框架在某发事件触发时自动调用这些事件处理程序。比如得到一个连接时，事件处理程序 connectionMade 被调用；丢失一个连接时，connectionLost 被调用。接收时，dataReceived 被调用。

但主动发数据时，不能使用事件处理程序。(即**事件处理程序**是用于被动触发的，不能用于主动的行为!)

Twisted 中主动发送数据使用对象 self.transport，这个对象有一个 write 方法。

示例：略。(不过都看明白了，libjingle 库真的让我明白了很多东西，看别的东西可以触类旁通，举一反三。)

关于网络的介绍完了吗，当然没有。下一章介绍网络世界中非常具体并更为人们熟悉的实体：万维网 (其实就是 web 啦)。

## 第 15 章 Python 和 Web

(2014/09/20)

这里讲三个主题：屏幕抓取 (哎，这名字取得，还以为是截屏呢，其实这里是指爬虫，但只爬 web 页面)、CGI、mod_python。

### 15.1 抓取 Web 页面

最简单的方法是自己使用 urllib 抓取页面，使用正则模块 re 进行分析提取。

存在以下问题：

1. 正则表达式不可读，难维护。
1. 程序对于 CDATA 部分和字符实体 (如 &amp;) 之类 html 特性无法处理。
1. 正则表达式被 html 源码约束，而不是取决于更抽象的结构。这意味着网页结构很小的改动就会导致程序中断。

更好的解决办法 (不自己使用正则)：

1. 使用 Tidy 和 XHTML 解析
1. 使用 Beautiful Soup 库
1. 其它库：scrape.py

15.1.1 Tidy 和 XHTML 解析

1. Tidy 库是什么

   Tidy 库用来修复不规范且随意的 HTML 的工具，以方便后面的解析。

   Tidy 不能修复 html 文件的所有问题，但是它会确保文件的格式是正确的 (所有的元素都正确嵌套)。

1. 获取 tidy 库，略。

1. 在 python 中使用命令行 tidy

   注意：tidy 并不是一个 python 模块，它是一个第三方可执行的工具。在 python 中可以使用 subprocess 来启动它，通过标准输出输入给它传数据。

1. 但为什么使用 XHTML

   因为 XHTML 对格式有很严格的要求。解析这类从 tidy 中获得的表现良好的 xhtml 的方法是使用标准库模块 HTMLParser。

1. 使用 HTMLParser

   HTMLParser 是基于事件的。

   使用 HTMLParser 的意思就是继承它，并且对 `handle_starttage` 或者 `handle_data` 等事件处理方法进行覆盖。

(!!! 把浏览器内核的 html 解析器拿出来用不行吗?? 浏览器内核的 html 解析器应该什么样的 html 都能解析吧)

(javascipt 也能将 html 解析为 DOM)

示例：略。看书上。需要实际使用时再回来看。

15.1.2 Beautiful Soup

有点类似 js 或 css 选择器，将 html 解析为 dom，然后直接用 key 拿到 value。

### 15.2 使用 CGI 创建动态网页

看完了，大致原理是知道的。以前在 Linux 的课堂上实践的，不过那时的 CGI 程序是用 C 写的。

这里再深入学习一下。目前为止已经彻底明白 CGI 的工作方式了。

CGI 必须依赖 web 服务器工作。(CGI 是上层网络框架比如 Djnago, Tornato, Flask 的基础... 吧?)

CGI 是网络服务器可以将查询 (一般来说是通过 web 表单) 传递到专门的程序 (比如 python, perl, c ...) 中并且在网页上显示结果的标准机制。(通过 url 去启动服务器上的一个可执行文件)

15.2.1 第一步：准备网络服务器

cgi 程序必须显式地标记为 cgi 脚本，这样服务器就不会把它们当作网页处理。两种方法：

1. 把脚本放到服务器的 cgi-bin 目录中
1. 把脚本文件扩展名改为 .cgi

15.2.2 第二步：加入 Pound Bang 行

    #!/usr/bin/env python

在作为 cgi 程序时这行必不可少，因为后缀可能已经改为 .cgi，那么只能通过这行内容来决定使用什么解释器了。

15.2.3 第三步：设置文件许可

在 Linux 上 用 chmod 为脚本加上可执行权限。

15.2.4 CGI 安全风险 !!!

cgi 脚本最好不要操作服务器上的本地文件，否则可能造成数据损毁。如果将用户提供的数据看做 python 代码 (比如 exec 或者 eval，可能存在的使用场景，写了一个在线版的计算器，用 eval 计算用户提供的数学表达式。又像是 SQL 语句查询一样，服务器执行的就是用户提供的数据)，则要小心注入攻击。或者 shell 命令运行，也就承担了运行任意代码的风险。

15.2.5 简单的 cgi 脚本

    #!/usr/bin/env python
    print 'Content-type: text/plain' #注意这行，重要!
    print #空行

    print 'Hello, world!'

15.2.6 使用 cgitb 调试

    import cgitb; cgitb.enable()

这样，网页调用 cgi 出错时，显示的出错信息是真正定位到程序的。

15.2.7 使用 cgi 模块

前面的例子，只产生了输出，而不能接收任何形式的输入。

输入是通过 html 表单提供给 cgi 脚本的键-值对，或称字段。

可以使用 cgi 模块的 FieldStorage 类从 cgi 脚本中获取这些字段。

    #!/usr/bin/env python
    import cgi
    form = cgi.FieldStorage()
    name = form.getvalue('name', 'world')
    #...

不用表单调用 cgi 脚本。(哎，其实就是把表单的内容直接拼接在 url 上啦，如 http://www.someserver.com/simple2.cgi?name=Gumby&age=42)

15.2.8 简单的表单

示例，略。表单和表单的处理都在一个简单的页面上。

### 15.3 更进一步：mod_python

(看完了，got it。 其实 mod_python 就相当于把 python 变成像 php 一样工作在 apache 上。)

(mod_python 不是 python 程序，而是 c/c++ 程序，编成了 .so 动态库，用来衔接 python 和 apache)

mod_python 是 apache 网络服务器的扩展，它可以让 python 解释器直接成为 apache 的一部分，就像 php 一样。

使用 mod_python 处理程序框架可以访问丰富的 api，深入 apache 内核等。

三个标准和处理程序：

1. cgi 处理程序，允许 mod_python 解释器运行 cgi 脚本，执行速度有很大提高。
1. PSP 处理程序，允许用 html 和 python 代码混合编程创建可执行网页，或者 Python 服务器页面 (Python Server Page, PSP)。(就是和 PHP, JSP, ASP 差不多类似啦。)
1. 发布处理程序 (Publish Handler)，允许使用 URL 调用 Python 函数。(路由??)

15.3.1 下载安装 mod_python, 并配置 apache

略。配置 apache，告诉它上哪去找 mod_python。和 php 的配置类似。

15.3.2 CGI 处理程序

略。比普通的 cgi 性能有一个数量级的提升。

15.3.3 PSP

略。和 php, jsp, asp 类似。将 python 代码写在 html 的 <%  %> 包含之中。一些网络框架的模板与此类似。

15.3.4 发布

(?? 名字好奇怪，不能反映其意义) (允许 url 调用 python 函数)

(越来越接近上层网络框架的工作方式了)

!!! mod_python 真正得到认可的原因是：它可以让程序员在比 cgi 脚本更有趣的环境中进行 python 开发。

它会把函数当作文档暴露给网络。比如 http://example.com/script.py/func 会让发布首先运行 script.py 中的 func 函数。

这个函数使用特殊的请求对象 (req) 作为唯一参数，将返回值作为显示给用户的文档。

    def func(req):
      return "hello world!"

另外可以使用 `__auth__` 和 `__access__` 函数进行访问控制。

OK，至此，大致了解即可，实际项目中不会直接自己使用 mod_python，而是使用上层封装好的各种网络框架。但了解一下内部原理也是有益的。

### 15.4 网络应用框架

Zope, Django, Pylons, web.py ...

### 15.5 Web 服务：正确分析 (??)

web 服务一般应用于很高层次的抽象。它们使用 http (web 协议) 作为底层协议，上面则是更多基于内容的协议。(那么我理解，XMMP 也是一种 web 服务吧)

15.5.1 RSS 和相关内容

略。

15.5.2 使用 XML-RPC 进行远程调用

略。RPC 和 REST。 ... REST 式的程序设计中使用得最多、最简单、最优雅的格式就是 JSON。它允许使用普通文本格工来表示复杂的对象。

15.5.3 SOAP

SOAP 是一种使用 HTTP 和 XML 作为底层技术的信息交换协议，就像 XML-RPC 一样，支持远程调用。异步。简单了解即可。

下一章介绍如何真正的测试程序。

## 第 16 章 测试

(2014/09/20)

(!!! 看这本书，并不是只在学习 python，而是在学习关于编程的一切! 各种语言之间的思想是相通的，只不过是换了一些语法罢了)

看完了，理解了。cool~~!!! 以后在实际项目中使用。

### 16.1 先测试，后编码

16.1.3 测试的 4 步

1. 指出需要的新特性。为其编写一个测试。
1. 编写特性的概要代码，这样程序就可以运行而没有任何语法方面的错误，但是测试会失败。
1. 为特性的概要编写虚设代码 (dummy code)，能满足测试要求就行。不用准确地实现功能，只要保证测试可以通过。
1. 现在重写代码，这样它就会做自己应该做的事，从而保证测试一直成功。

### 16.2 测试工具

doctest, unittest。

16.2.1 doctest

cool~~! 将测试内容 (交互式运行) 写在函数的 `__doc__` 中。具体使用略。需要时再看。

16.2.2 unittest

和 junit, gtest 类似。使用略。需要时再回头看。

### 16.3 单元测试以外的内容

源代码检查和分析。

黄金法则：使用工作 (单元测试)、使用其好 (源代码检查)、使用更快 (性能分析)。

16.3.1 使用 PyChecker 和 PyLint 检查源代码。

在命令行使用。(可以使用 subprocess 模块调用它们进行检查。)

如何把它们与单元测试结合起来。略。看书上例子。

16.3.2 分析

profile 和 pstats 模块。使用略。

如果已经到了需要使用性能分析工具的时候，那么是时候拿起重武器了。

## 第 17 章 扩展 Python

(2014/09/20)

粗略看完了，大致了解即可。

python 的灵活性是以运行效率作为沉重代价的。对于一般程序来说，它足够快了，但如果真的很在意速度的话，c/c++/java 比 python 快几个数量级。

### 17.1 考虑哪个更重要

推荐方法：

1. 在 python 中开发一个原型程序
1. 分析程序并且找出瓶颈
1. 用 c (或者 c++，c#,  java ...) 作为扩展来重写出现瓶颈的代码。

封装可能的瓶颈。

### 17.2 非常简单的途径：Jython 和 IronPython

Jython 可以直接使用 java 的类，java 也可以使用 Jython 的类。将 java 类编译成 .class，然后就可以直接在 Jython 中 import。

IronPython 可以使用 c# 中定义的类。但使用起来稍微比 Jython 麻烦一点。c# 代码需要使用专门的编译器编译成 .dll，然后在 IronPython 中加载这个 .dll 进行使用。

### 17.3 编写 C 语言扩展

略。稍麻烦。

一种方法是使用 SWIG 工具 (或者说是一种框架)。或者直接使用 Python C API。

(我的理解是，这就像是为浏览器写插件一般，需要遵循插件的规范。以在 Linux 为浏览器写插件为例，前者即 SWIG 相当于使用 FireBreath 框架，后者相当于直接使用 npapi 接口)

(又如写 jni 接口)

(所以，写插件，思想是一样一样的，遵循别人定义的接口规范，实现自己的功能，接口由主程序在一定时间调用)

(从广义上说，Android 的 app 也算是插件，因为它的 onCreate, onDestroy... 都由 android 系统控制调用。而 windows 上的桌面程序，算得上是最轻量级的插件，是属于操作系统的插件，因为它几乎只有 main() 函数由系统控制调用。)

(可不可以这么说，除了操作系统，一切应用都是某种程序上的插件)

与别人分享你的程序

## 第 18 章 程序打包

(2014/09/20)

略。大致看完了。需要时再回来细看。

使用 Distutils 模块，打包，生成 windows 安装程序或 rpm 包。还可以用来编译 c 写的扩展 (免去了上一章自行编译的麻烦)。

使用 py2exe 创建可执行程序。(但还是要配合 Inno Setup 或 nsis 使用)。

至此，技术相关的内容就介绍到这里了。(没有介绍模块管理方面的知识，自己看吧)。

接下来介绍具有方法论和哲学意味的编程方式。

## 第 19 章 好玩的编程

(2014/09/20)

### 19.1 为什么要好玩

### 19.2 程序设计的柔术

柔，就是灵活性。其中两个：原型设计、配置。第三点，自动化测试 (前面讲过了)。

### 19.3 原型设计

### 19.4 配置

重温抽象的重要原则。

19.4.1 提取常量

19.4.2 配置文件

- 一种方法是将常量写在 python 脚本中，作为模块 import 到主程序中。
- 一种方法是将常量写在 配置文件 中，然后在程序中使用 ConfigParser 模块解析配置文件。

使用方法略。(以前我用过)。

配置的级别。

可配置性是 UNIX 编程传统中的一部分。一般遵循以下顺序：

1. 配置文件
1. 环境变量
1. 命令行中传递的选项和参数

### 19.5 日志记录

一种方法是使用 >> ，print 时将内容写到文件中。

    log = open('logfile.txt', 'w')
    print >> log, ('test...')

另一种方法是使用 logging 模块

    import logging
    logging.basicConfig(level = logging.INFO, filename='mylog.log')
    logging.info('Start program')
    #...

Done! 接下来的是项目实践。

看完以上内容，重新翻了《Python 核心编程 (第二版)》，章节都差不多啊，好像就多了多线程一章的内容，每个知识点的内容也多一些。但这本书我怎么就是没有耐心看完呢。可能还是语言风格的区别吧。

以后这两本书结合着来看。(比如正则这本书就讲得更详细得多)。

接下来实践！

后面的实践主要是写代码，就不做笔记了。
