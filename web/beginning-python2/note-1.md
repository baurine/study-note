# Python 基础教程 (第二版) 笔记 - Note 1

Time：2014/09/07

所用 python 为 python2。

(第 1 章 - 第 9 章)

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
