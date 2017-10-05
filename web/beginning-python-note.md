# Python 基础教程 (第二版) 笔记

Time：2014/09/07

所用 python 为 python2。

## 第 1 章 基础知识

略。

### 1.4 数字和表达式

python 里的基本数据类型几乎就只有数值，字符串，布尔值。然后由这些组成序列、列表、元组、字典... (字符串也是序列的一种)。

### 1.7 获得用户输入

    x = input("x:")
    x = raw_input("x:")

二者的区别：前者会根据输入的值的字面值，转换成相应的格式，而后者把输入当字符串返回，不进行任何转换。比如输入 23。前者得到是一个类型为整型的值 23，后者得到的是字符串 "23"。

### 1.9 模块

    import math
    math.floor(xxx)

    import floor from math
    floor(xxx)

尽量用第一种方式。(第二种方式和 JavaScript 的 ES6 module 一样，用第二种也挺好的呀，ES6 就根本没有第一种使用方式。)

当模块名导出的对象同名时，使用 as 命名别名。

    from yyy import xxx as xxx1
    from zzz import xxx as xxx2

### 1.10 运行

    #!/usr/bin/env python
    #!/usr/bin/python2

### 1.11 字符串

1.11.1 ' 和 " 无区别

1.11.2 拼接：'' "" +

1.11.3 str/repr

    str("hello") --> "hello"
    repr("hello") --> "'hello'"

1.11.4 input 与 raw_input

1.11.5

- 长字符串："""
- 原始字符串：r''，用于带转义字符的字符串
- unicode：u''

## 第 2 章 列表和元组

列表和元组都属于序列。

### 2.1 序列概述

python 有 6 种序列：

- 列表
- 元组
- 字符串
- unicode 字符串
- buffer 对象
- xrange 对象

### 2.2 通用序列操作

2.2.1 索引。从 0 开始，-1 最后。

2.2.2 分片。两个索引作为边界，第 1 个索引包含在分片内，第 2 个不包含在内。

优雅的捷径，访问最后三个 numbers[-3:]。

2.2.3 相加。

2.2.4 乘法。

2.2.5 成员资格。in。

2.2.6 长度，最小值，最大值。len, min, max。

### 2.3 列表：Python 的苦力 []

2.3.1 list 函数与 join。

其实 list 不是函数，而是类型，即列表类型。

2.3.2 列表的基本操作 (序列的基本操作适用于列表)

- 赋值
- 删除， del
- 分片赋值，插入

2.3.3 列表方法

- append
- count
- extend，append 和 extend 都是原位操作，修改原变量值，返回什么呢? 没有返回值
- index
- insert
- pop
- remove
- reverse，原位操作，改变了列表，但不返回值
- sort，原位操作。但 sorted 函数返回一个列表副表
- 高级排序，sort 有三个参数。sort(cmp, key=, reverse=True/False)

### 2.4 元组，不可变序列 ()

2.4.1 tuple 函数，将序列转换成元组。(其实 tuple 也是类型。)

2.4.2 基本操作，和序列一样，没有列表的方法，因为是不可变的嘛。

2.4.3 意义。字典的 key 是元组，不可变。

## 第 3 章 使用字符串

字符串也是不可修改的序列。

### 3.2 字符串格式化：精简版

format % value (format 和 c 语言一样，比如 %s，%d)

模板字符串，类似 unix shell，变量替换。模板的概念。

### 3.3 字符串格式化：完整版

大致了解，需要时重看，基本和 c 一样。包括宽度，精度，符号，对齐，0 填充。

### 3.4 字符串方法

- find。和 c++ 的 string.find 差不多。
- startswith, endswith。
- join, split。
- replace。
- strip (trim)，去除两侧特定字符。
- lower, translate ...

## 第 4 章 字典 {}

### 4.2 创建和使用字典

{}

4.2.1 dict 函数 (其实 dict 是类型)

如何理解 list, tuple, str, dict 是类型而不是函数。我是这么理解的，这些相当于 c++ 中的类，这些函数相当于构造函数。

4.2.2 字典基本操作

len(d), d[k], d[k] = v, del d[k], k in d。

键必须为不可变类型 (基本就只有数值和字符串了)。

4.2.3 字典的格式化字符串 (!!! cool) (fuck! 现在再看完全不记得了)

4.2.4 字典方法

clear, copy, fromkeys, get (安全)

浅拷贝，深拷贝。

items, iteritems，前者返回列表，后者返回迭代器。同理 keys, iterkeys; values, itervalues。

其它方法，大致了解，需要时再回头看。

pop, popitem, setdefault, update ...

## 第 5 章 条件、循环、其它

### 5.1 print, import

5.1.1 print

和 c++ 的 std::cout 一样，可以连续打印变量。

5.1.2 import

    from module1 import open as open1
    from module2 import open as open2

### 5.2 赋值魔法 cool

5.2.1 序列解包

    x,y,z = 1,2,3

5.2.2 链式赋值

5.2.3 增量赋值

### 5.3 缩排

### 5.4 条件语句

bool：True/False。有东西为 True，没有东西为 False。

    if XXX:  # 注意冒号
    elif:
    else:

5.4.6 更复杂的条件

== 比较值，is 比较是否同一个对象

逻辑：and, or, not (不是 &&, ||, !)

5.4.7 断言

assert

### 5.5 循环

(有比 c/c++ 多更多内容)

while, for。

for xx in yy:

range/xrange：都是产生序列，但前者一次性产生一个大列表，后者一次只产生一个数。

5.5.3 循环遍历字典，迭代器

    d = {}
    for key in d:
    for key, value in d.items():

5.5.4 一些迭代工具

1. zip 函数，并行迭代
1. 编号迭代。enumerate() 函数，返回索引-值对。
1. 翻转，排序。sorted, reversed。

### 5.6 列表推导式 []

(!!! 相比 c/c++ 是新内容)

如何理解，可以把它等价地扩展为 for...for...if 语句。所以就是 for...for...if 的缩写形式。got!

### 5.7 三人行，pass, del, exec

`del x` 删除对象和名字。(和 c++ 的 del 不一样，这里还删除变量的名字，以后都不可以再引用它了。这点 c/c++ 做不到，因为在 c++ 中变量名字是静态编译的)。

exec/eval，黑暗魔法。动态地创造并执行 python 代码。eval 返回值，exec 不返回值。

使用 `in <scope>` 增加命名空间。

需要时再回头具体学习。

## 第 6 章 抽象

(其实讲的是函数啦)

6.3.1 记录函数 (即文档字符串)

    def square(x):
      'Calculates the square of the number x.'
      return x*x

文档字符串可以按如下方式访问：

    >>> square.__doc__
    >>> help(square)

### 6.4 参数魔法

6.4.2 我能修改参数吗

参数可变的会修改原值，不可变的不会。

6.4.3 关键参数和默认值

之前的参数叫位置参数。

    def hello(greeting='hello', name='bao'):
      ....

    hello(name='baurine', greeting='good morning')
    hello()

6.4.4 收集参数 (很 cool)

*param：参数转为元组。

**param：参数转成字典。

6.4.5 反转过程

*param：元组转成独立的位置参数

**param：字典转成独立的关键参数

    def add(x,y): return x+y
    param = (1,2)
    add(*param)

### 6.5 作用域

python 中作用域是一个字典，用内建 vars() 函数可以得到这个字典。(和 Ruby 中的 `local_variables` 类似)

    >>> x = 1
    >>> scope = vars()
    >>> scope['x']
    1

globals() 函数类似 vars()，获得全局变量值的字典。

locals() 获得局部作用域字典。

!!! 在函数中使用全局变量。默认是只读的，不能被函数修改。如果强行修改，那么它会自动变成局部变量。如果非要在函数中修改全局变量，那么在使用它前要用 global 修饰。got it! 为了安全。

python 可以函数嵌套，可以用函数嵌套实现闭包。

### 6.6 递归

迭代是循环。for, while。

递归是 if...else...

函数式编程：map, filter, reduce。不过在 python2 中不是特别有用。前二者可以用列表推导式替代。

lambda，源自希腊字母，在数学中表示匿名函数。

## 第 7 章 更加抽象

(其实讲的是类，面向对象编程)

### 7.1 对象的魔力

多态，封装，继承。没啥太多可说的。

### 7.2 类和类型

(我的理解是，类型是语言内置的基本数据类型，而类是自定义。)

在旧版本的 python 中，类和类型之间有很明显的区别。内建的对象是基于类型的，自定义的对象是基于类的，可以创建类但是不能创建类型。新版本的 python，两者开始模糊。(其实相当于前者与 c/c++ 的行为接近，后者开始学习 java，一切皆对象，包括基本数据类型也变成了类。)

7.2.2 创建自己的类

    __metaclass__ = type # 确定使用新式类 ??
    class Person:
      def setName(self, name):
        self.name = name
    ...

7.2.3 特性，函数，方法。

方法是类的成员函数，第一个参数为 self。

私有化。

python 不像 c++ 那样有明确的 private 属性，所有都是公开的。但可以使用一些技巧把成员变成类似 private。

1. 使用 `__` 修饰。因为 python 解释器会把带 `__` 的函数翻译成其它名字，所以直接用 `__xxx` 是无法访问到原来的函数的。
1. 使用 `_` 修饰。因为 `ipmort *` 时不会导入带 `_` 的名字。

7.2.4 类的命名空间

类的定义中，没有用 `self.` 修饰的成员变量，可以被所有实例同时访问到，其实相当于是 c/c++ 中的 static 变量啦。但又不一样。(具体看书上说的吧，这里我也说不清，类似前面说到的在函数中使用全局变量。)

7.2.5 指定超类 (哎，就是父类啦，这里就说是继承嘛)

    class Filter:
      ...
    class SPAMFilter(Filter):
      ...

7.2.6 调查继承

    issubclass
    isinstance
    s.__bases__
    s.__class__
    type(s)

7.2.7 多重继承

注意继承顺序。

## 第 8 章 异常

用 raise 主动触发异常。

    raise Exception("xxx")

异常的捕捉。

    try:
      1/0
    except ZeroDivisionError, e:
      print e
    else:
      print "ok"
    finally:
      print "clean up"

### 8.11 异常之禅

在 python 中，使用 try/except 语句比使用 if/else 会更自然一些 (更 "Python化")，应该尽可能使用 try/except 语句的习惯。

在做一件事时去处理可能出现的错误，而不是在开始做事前就进行大量的检查。

## 第 9 章 魔法方法、属性和迭代器

(2014/09/08)

从第 9 章开始的内容，每天看一章。

生成器部分没有完全明白，不过没关系，用到时再回来细看，不要卡在这里，继续往下看。

前后带 `__` 的方法，特殊情况下被 python 主动调用，而几乎没有直接调用它们的必要。这些方法称为魔法方法。(不就是一些勾子嘛，还什么魔法方法...)

### 9.1 准备工作

使用新式类。继承自 object，或者用 `__metaclass__ = type` 声明。用前者。

    class NewStyle(object):
      more_code_here

### 9.2 构造方法

    __init__(self)

    class FooBar(object):
      def __init__(self):
        self.somevar = 12

析构方法 `__del__`，尽量避免使用。

9.2.1 重写一般方法和特殊的构造方法

!!! 子类的构造方法必须调用其父类的构造方法来确保基本的初始化。(和 c++ 不一样 。c++ 如果不显式调用父类的构造方法，那么其值是未知的，但至少可以拿来用。但 python 如果不显式调用父类的构造方法，那么在父类中的变量相当于没有，连用都不可以用。)

两种方法：调用超类构造方法的未绑定版本，或者使用 super 函数。后者只能用于新式类。(按理说，应该用后者就行了，但作者推荐前者。不知为何。)

9.2.2 调用未绑定的超类构造方法

    class SongBird(Bird):
      def __init__(self):
        Bird.__init__(self)
        self.sound = 'Squawk!'

      def sing(self):
        print self.sound

9.2.3 使用 super 函数

    class SongBird(Bird):
      def __init__(self):
        super(SongBird, self).__init__()
        self.sound = 'Squawk!'

### 9.3 成员访问

(这一节的标题起得莫名其妙，感觉与内容不符。其实这一节讲的是如何实现 `__len__()`, `__getitem__(self, key)` 这些魔法方法，以便可以通过 len(), a[] 这些内建方法得到正确的值。其实就是相当于实现这些虚方法。)

9.3.1 基本的序列和映射规则

实现 `__len__(self)`, `__getitem__(self, key)`, `__setitem__(self, key, value)`, `__delitem__(self, key)`...

9.3.2 子类化列表，字典和字符串

在新版本的 python 中可以子类化内建类型。如果不想自己手动实现 `__len__`, `__iter__` 等等方法，可以继承自 list, dict, str 等。只需要再实现与它们不一样的函数就行。

### 9.4 更多魔力

没讲。

### 9.5 属性

访问器。

9.5.1 property 函数

    class Rectangle(object):
      def __init__(self):
        self.width = 0
        self.height = 0
      def setSize(self, size):
        self.width, self.height = size
      def getSize(self):
        return self.width, self.height
      size = property(getSize, setSize)

9.5.2 静态方法和类成员方法

(没太明白类成员方法是做什么用的?)

    class MyClass:
      @staticmethod
      def smeth():  # 没有 self 参数
        print 'smeth'

      @classmethod
      def cmeth(cls):  # 没有 self 参数，但有 cls 参数
        print 'cmeth'

    >>> MyClass.smeth()
    >>> MyClass.cmeth()

9.5.3 `__getattr__`, `__setattr__` 和它的朋友们。

(相当于 hook 一样)，拦截对象的所有特性访问。

略。

### 9.6 迭代器

特殊方法 `__iter__`。实现了 `__iter__` 方法的对象都可以被迭代。

`__iter__` 方法返回一个迭代器 (iterator)，所谓的迭代器就是具有 next 方法的对象。在调用 next 方法时，迭代器返回它的下一个值，如果没有值可返回，引发一个 StopIteration 异常。

正确的说法：一个实现了 `__iter__` 方法的对象是可迭代的，一个实现了 next 方法的对象则是迭代器。

9.6.2 从迭代器得到序列

使用 list(iter_obj)。

### 9.7 生成器

没有特别明白，以后用到时再说吧。

生成器函数是包含了关键字 yield 的函数。当被调用时，生成器函数返回一个生成器 (一种特殊的迭代器)。

(2017/10/5，哦，现在明白了，和 ES6 中的生成器概念是一样的。)

### 9.8 八皇后问题

用生成器解决。略。

python 的基础知识到此为止。

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
