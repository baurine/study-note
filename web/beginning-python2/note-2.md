# Python 基础教程 (第二版) 笔记 - Note 2

(第 10 章 - 第 13 章)

## 第 10 章 充电时刻

(2014/09/09)

python 的模块及标准库。

### 10.1 模块

使用 import 从外部导入模块。

10.1.1 模块是程序

任何 python 程序 (我觉得是指任何一个 python 程序文件) 都可以作为模块导入。

文件保存的位置很重要。

    import sys
    sys.path.append('c:/python')  # 这里加入你保存的模块所在的路径，必须是绝对路径

模块导入多次，也只执行一次。

10.1.2 模块用于定义

1. 在模块中定义函数
1. 在模块中增加测试代码。使用 `__name__` 这个内置变量。

         if __name__ == '__main__': xxx

   如果模块作为主程序执行，内置变量 `__name__` 值为 `__main__`，如果作为模块被导入，值为模块的名字。

10.1.3 让你的模块可用

1. 将模块放置在正确位置。

   那正确的位置从何得知呢。

        import sys, pprint
        pprint.pprint(sys.path)

   最佳选择是 site-packages 目录。将你自己写的模块放到此目录，就能随时 import 了。

1. 告诉编译器去哪里找

   一是 sys.path.append('xxx')，二是设置环境变量 PYTHONPATH。

1. 命名模块

   包含模块代码的文件名字要和模块名一样。就是说，你想给这个模块取什么名字，文件名就得是什么名字。

10.1.4 包

简单理解就是，包是模块的集合，包里有多个模块。包其实就是一个文件夹而已，包含一个特殊的 python 文件 `__init__.py`，以及其它模块文件。文件夹的名字就是包名。仅此而已。没什么复杂。

(就像 JavaScript 包下的 index.js)

### 10.2 探究模块

10.2.1 模块中有什么

1. 使用 dir 函数

        >>> import copy
        >>> dir(copy)

1. `__all__` 变量

        >>> copy.__all__

   `__all__` 变量定义了模块的公有接口，如果使用 `from copy import *`，只能使用 `__all__` 变量中的 4 个函数。起了一种过滤作用。

10.2.2 用 help 获取帮助

    >>> help(copy.copy)

10.2.3 文档

    >>> print copy.copy.__doc__

10.2.4 使用源代码

去哪找模块的源代码文件。

    >>> print copy.__file__

### 10.3 标准库：一些最爱

(2014/09/12)

一些常用的标准库模块

10.3.1 sys

重要函数和变量：argv, exit(), modules, path, platform, stdin, stdout, stderr.

10.3.2 os

重要函数和变量：environ, system(command), sep, pathsep, linesep, urandom(n)

system(command) 函数用于运行外部程序，与之类似功能的函数还有 popen()。但 subprocess 模块把它们都包含了。

os 有个子模块 path。os.path 用于检查，创建，删除目录和文件，以及一些处理路径的函数。即操作文件系统。(os.path.split, os.path.join 可以替代 os.pathsep)。

10.3.3 fileinput

fileinput 模块让你能够轻松地遍历文本文件的所有行。

重要函数：input(), filename(), lineno(), filelineno(), isfirstline(), isstdin(), nextfile(), close().

具体应用略。

10.3.4 集合、堆和双端队列

1. 集合

   因为符号被列表，元组，字典 ([], (), {}) 用完了，所以集合的创建就只能通过类型显式创建了。用 set() 来创建集合。

        >>> set([1,2,3,1])
        set([1,2,3])
        >>> set((1,2,3,1))
        set([1,2,3])

   可以求集合的交集，并集，子集等。

   frozenset，表示不可变的集合。

1. 堆

1. 双端队列以及其它集合类型

   略。暂时用不到。

10.3.5 time

重要函数：time(), localtime(), gmtime(), asctime(), mktime(), strptime(), sleep().

与 unix 的 几个时间函数相对应。

另外两个与时间密切相关的模块：datetime (支持日期和时间的算法)，timeit (帮助开发人员统计代码性能)。

10.3.6 random

前面说过 os.urandom() 函数也可以提供随机数，这个具有真的随机性。

重要函数：random(), getrandbits(), uniform(), randrange(), choice(), shuffle(), sample()

10.3.7 shelve

以前一直没明白这个模块是干嘛用的。从这里来看，应该算是一种将文件作为数据库存储内存中的数据的模块。二进制序列化与反序列化？

嗯，就是一种简单的 NoSQL 的实现！

10.3.8 re !!! (重点学习模块)

python 的正则表达式模块。

**什么是正则表达式**

关键概念！

通配符 (wildcard)：只有一个，那就是点号 `.`，匹配除换行符外的任何单个字符！

(注意：在非正则中，`*` 号和 `?` 号是通配符，前者通配所有字符，后者通配单个字符，但在正则中，是完全不一样的，正则中只有一个通配符，而 `*` 和 `?` 是其它含义)

对特殊字符进行转义：使用反斜杠 `\`，如 `r'pyhon\.org'` 或者 `'python\\.org'` (`\` 自身也需要转义)

字符集：使用中括号 `[]` 括住的字符串。如 `'[pj]ython'`, `'[a-zA-Z0-9]'`，用来匹配括号中任意的单个字符。

反转字符集，在中括号中的字符最前面加上 `^`，如 `[^abc]`，用来匹配除 a、b、c 外的所有字符。

(在字符集中对 `.`, `*`, `?` 这些特殊字符做转义是没有必要的。但如果 `^` 放在开头且不是用来表示反转，那么要转义)。

选择符和子模式： | 和 ()

字符集只能匹配单个字符，如果要匹配多个长度不一的字符串，那么就需要使用用于选择项的特殊字符：| (可以理解为 或 符)。比如想匹配 'python' 和 'perl'，那么匹配模式可以写成 'python|perl'。

有时候不需要对整个模式使用选择运算符，这时可以使用圆括号 '()' 括起需要的部分，称为 子模式 (subparttern)。上例可写在 'p(ython|erl)'。

子模式一般用于长度超过 1 的字符串，但也可以用于单个字符。

可选项和重复子模式：?, +, *, {m,n}

- ?：可选项。也可理解为将前面的子模式 (包括单个字符) 重复 0 或 1 次。
- +：将前面的子模式重复 1 次或多次。
- *：将前面的子模式重复 0 次或多次。
- {m, n}：将前面的子模式重复 m~n 次。{m,} 表示 重复 m 及 m 次以上。

所以：

    (pattern)? = (pattern){0,1}
    (pattern)+ = (pattern){1,}
    (pattern)* = (pattern){0,}

字符串的开始和结尾：^ 和 $

前面的模式是匹配整个字符串的，如果只想在开头而不是其它位置匹配，可以使用 ^。

比如 '^ht+p' 会匹配 `http://python.org`，但不会匹配 'www.http.org'。

类似的，字符串结尾用 $ 标识。

!!! 这里所说的正则只是简单原始的正则，实际上还有正则的扩展形式，即用 \d, \w, \b 等转义字符来表示各种字符集。这个自己查资料补充吧。

**re 模块的内容**

重要的函数：

- compile(pattern, [flags])：根据包含正则表达式的字符串创建模式对象，用于需要多次匹配的地方，可提高匹配效率。
- search(pattern, string, [flags])：在字符串中寻找模式
- match(pattern, string, [flags])：在字符串的开始处匹配模式
- split(pattern, string, [maxsplit = 0])
- findall(pattern, string)
- sub(pat, repl, string, [count = 0])
- escape(string)：将特殊字符转义

re.search() 会在给定字符串中寻找第一个匹配给定正则表达式的子字符串。一旦找到子字符串，函数会返回 MatchObject (值为 True)，否则返回 None (值为 None)。

re.match() 仅在字符串开始处匹配正则表达式。

**匹配对象和组 (!!! 难点)**

嘿，这书简单描述后发现也不是难点。Got it!

对于 re 模块中那些能够对字符串进行模式匹配的函数而言 (比如 re.search, re.match)，当能够找到匹配项时，它们都会返回 MatchObject 对象 (即 匹配对象)。匹配对象中包括了匹配模式的子字符串的信息，还包含了哪个模式匹配了子字符串哪部分的信息，这些 "部分" 叫 组 (group)。

!!! 简而言之，组就是放置在圆括内内的子模式。组的序列取决于它左侧的括号数，组 0 就是整个模式。

MatchObject 的重要方法：

- group([group num])：获取给定子模式(组)的匹配项
- start([group num]), end(), span()：返回给定组的匹配项的起始，结束位置。

例子：

    >>> m = re.match(r'www\.(.*)\.(.{3})', 'www.python.org')
    >>> m.group(1)
    'python'
    >>> m.group(2)
    'com'
    >>> m.group(0)  //或者 m.group()
    'www.python.org'
    >>> m.span(1)
    (4,10)  // 'python' 的起始，结束位置

**作为替换的组号和函数 (??) (got it!)**

见证 re.sub 强大功能的最简单方法就是在替换字符串中使用组号。在替换内容中以 '\\n' 形式出现的任何转义序列都会被模式中与组 n 匹配的字符串替换掉。

例子，要把 `*something*` 替换成 `<em>something</em>`

    >>> emphasis_pattern = r'\*([^\*]+)\*'
    >>> re.sub(emphasis_pattern, r'<em>\1</em>', 'hello, *world*')
    'hello, <em>world</em>'

贪婪模式与非贪婪模式

贪婪模式就是尽量匹配多的内容，而非贪婪模式是尽可以少的匹配。(例子略，看书吧)

重复运算符 (+*) 默认是贪婪模式。

所有的重复运算符都可以通过在其后面加上一个问号变成非贪婪模式。(这里的问号就应该不是指可选项符了吧)

10.3.9 其它有趣的模块

- functools：包括一些函数式编程的东西，比如 filter, reduce 函数。
- difflib：计算两个序列的相似程度
- hashlib：计算签名，同时用于加密和安全。类似模块，md5, sha。
- csv：处理 csv 格式文件。
- timeit, profile, trace：用于代码分析，性能分析。
- datetime：提供比 time 模块更高级功能。
- itertools：与迭代器相关。
- logging：日志模块。
- getopt, optparse：解析命令参数，前者简单一点，后者更新更强大更易使用。
- cmd：用这个模块可以编写命令行解释器。

到目前为止，所介绍的内容都是和解释器自带的数据结构打交道，与外部的交互只有 input, raw_input 和 print 函数，与外部的交互很少。

接下来，将会介绍如何用 python 与外部世界--文件和网络--进行交互。(!!! 好书啊，逻辑层次分明)

## 第 11 章 文件和素材

(!!! 重要的一章)

(2014/09/16)

文件和流。

### 11.1 打开文件

open 函数，返回文件对象。

    open(name[, mode[, buffer]])

(新版本 python 中，open 函数是 file 函数的别外)

11.1.1 文件模式

r, w, a, b, +。

11.1.2 缓冲

第三个参数 buffer 控制是否缓冲，为 0 不缓冲，为 1 缓冲。

### 11.2 基本文件方法

11.2.1 读和写

- 读：f.read()
- 写：f.write()

最后要把文件进行 f.close()

11.2.2 管式输出

管道符号 (|) 将一个命令的标准输出和下一个命令的标准输入连在一起。

python 使用 sys.stdin, sys.stdout, sys.stderr 来操作标准输入，标准输出，标准错误。例子：

    # somescript.py
    import sys
    text = sys.stdin.read()
    words = text.split()
    wordcount = len(words)
    print "wordcount:", wordcount
    >>> cat somefile.txt | python somescript.py | sort

随机访问：f.seek(), f.tell() ...

11.2.3 读写行

读取单行：f.readline()

读取所有行并作为列表返回：f.readlines()

写序列：f.writelines()，没有 f.writeline()

11.2.4 关闭文件

f.close()

如果想确保文件被关闭，应该使用 try/finally 语句，并在 finally 语句中调用 close()。

    # open file
    try:
      # read or write
    finally:
      f.close()

python 2.5 以后，有专门为这种情况设计的语句，即 with 语句。

    with open("somefile.txt") as somefile:
      do_something(somefile)

with 语句执行完了会自动把文件关闭。

with 语句是一种上下文管理器 (!!! 新概念)。上下文管理器是一种支持 `__enter__` 和 `__exit__` 方法的对象。 `__enter__` 不带参数，它在进入 with 语句块时被调用，返回值绑定到 as 关键字后的变量上。`__exit__` 方法有 3 个参数：异常类型、异常对象、异常回溯。语句离开时，这个方法被调用。

11.2.5 使用基本文件方法

演示了上述所讲到的方法的使用。略。

### 11.3 对文件内容进行迭代

11.3.1 按字节处理

f.read(1)

11.3.2 按行处理

f.readline()

11.3.3 读取所有内容

如果文件不太大的话，可以用 f.read() 或 f.readlines() 一次性读取所有文件内容到内存中。

11.3.4 使用 fileinput 实现懒惰行迭代

对大文件进行迭代时，readlines() 会占用太多内存。

    import fileinput
    for line in fileinput.input(filename):
      process(line)

11.3.5 文件迭代器 (!!! 压轴)

从 python 2.2 开始，文件对象是可迭代的! 意味着可直接在 for 循环中使用。

    f = open(filename)
    for line in f:
      process(line)
    f.close()

sys.stdin 也是可迭代的。

可以对文件迭代器执行和普通迭代器相同的操作，比如转换成字符串列表，用 list(open(filename))，达到 readlines() 的效果。

至此，已经知道了如何通过文件与环境交互，但怎么和用户交互呢? (图形界面。再一次体现了章节之前如何衔接，如何循序渐进，cool~~!!)

## 第 12 章 图形用户界面

(2014/09/16)

略。这一章不是重点，稍做了解即可。

## 第 13 章 数据库支持

(2014/09/18)

大致看完一遍。再看一遍，略微做一下简单的笔记。这一章讲得也很简单，只是一个入门，和 12 章差不多，需要时再自己深入学习。

### 13.1 Python 数据库 API

现在有一个标准和 DB API，版本 2.0。

13.1.1 全局变量

apilevel, threadsafety, paramstyle

13.1.2 异常

各种异常异，最顶层的基类是 StandardError 类。

13.1.3 连接和游标

为了使用数据库，首先必须连接到它。使用 connect() 函数。

    connnect(dsn[, user, passwd, host, database])

connect() 函数返回连接对象。(就跟 open() 函数返回文件对象一样。)

连接对象具有以下重要方法：

- close()：关闭连接和游标
- commit()：提交事务
- rollback()：回滚 (需要使用的数据库支持事务)
- cursor()：返回连接的游标对象

游标对象用于实际对数据库进行 sql 查询等操作。游标对象具有以下重要方法：

- callproc()：我理解是调用存储过程或是 sql 子程序
- close()：关闭游标对象
- execute()：执行 sql 语句
- executemany()：对序列中的每个参数执行 sql 操作
- fetchnone(), fecthmany(), fetchall(), nextset()：取结果
- setinputsize(), setoutputsize()：设置缓冲区大小

游标对象的特性：description, rowcount, arrayszie ...

13.1.4 类型

用于定义数据库字段的类型。

### 13.2 SQLite 和 PySQLite

PySQLite 是 连接 python 和 SQLite 的模块。SQLite 和 PySQLite 都需要单独安装，但最新的 python 安装包里可能已经包含了两者，所以可以不必再单独安装。

13.2.1 入门

    import sqlite3
    conn = sqlite3.connect('sample.db')
    cursor = conn.cursor()
    # 执行各种 sql 语句，选择表，增删改查，但此时结果都是存在内存里的
    conn.commit()  # 提交结果
    conn.close()

13.2.2 示例

看懂了。没有实践，用到的时候再回来看，不用的话实践了之后也会忘的。

坚持不懈地进行数据库处理是绝大多数程序 (如果不是大多数，那就是大型程序系统) 的重要部分。

接着介绍另外一个大型程序系统都会用到的组件，即网络。
