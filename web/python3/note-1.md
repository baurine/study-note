# Python3 - Note 1

Note for [廖雪峰的 Python 教程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

## 简介

略。

安装，略。

**输入与输出**

输入用 input() 方法，输出用 print() 方法。

    >>> name = input('please enter your name: ')
    >>> print('hello,', name)

input() 得到的值是 str 类型，如果我输入的是 "123"，且想得到数值类型的 123，如需要用 int() 方法进行转换。

    >>> age = input('your age: ')
    your age: 123
    >>> age
    '123'
    >>> int(age)
    123

## 基础

### 数据类型

基本数据类型 (各个语言都大同小异)

- 整数
- 浮点数
- 字符串：支持单引号 ''，双号引 ""，以及三个单引号 '''，三个单引号用来显示多行，以前简化转义字符写法的 r''
- 布尔值：True / False，逻辑运算 and, or, not
- 空值：None (类似 void 或 nil)

变量与常量，略。

除法：/ 和 //，后者称为地板除 (floor divide)，前者结果为浮点数，后者为整数。

(验证了一下，还真是，即使是 `6/3`，双方都是整数，也能整除，结果也是浮点数 2.0，这和其它语言不同一样，其它语言一般是得到 2，如果想得到 2.0，那么需要把除数或被除数变成浮点数，比如 `6*1.0/3`)

    >>> 6/3
    2.0
    >>> 6//3
    2

### 字符串和编码

(重要，但我觉得在这一点上 Python 还是比 Ruby 好理解，我以前写过一篇 blog 来理解这个)

核心思想就是字符虽然在文件中存储中有 UTF-8, UTF-16 等各种编码，但在内存中统一加载为 Unicode 编码。Unicode 编码是内存中的形式，UTF-8, UTF-16 等是存储或传输时的编码形式，各种编码格式是不完全相同的，可以相互转换。转换时先统一在内存中转成 Unicode，再由 Unicode 编码成其它形式存储或传输。

Unicode 一般来说都是两字节 (除非非常生僻的，需要用到 4 字节)，UTF-8，则是用哈夫曼编码，使用频率高的编码越短，使用频率低的编码越长，所以有的是 1 字节，有的是 2 字节，多的达 4 字节，或 6 字节 (有这么长的?)

> 新的问题又出现了：如果统一成 Unicode 编码，乱码问题从此消失了。但是，如果你写的文本基本上全部是英文的话，用 Unicode 编码比 ASCII 编码需要多一倍的存储空间，在存储和传输上就十分不划算。

> 所以，本着节约的精神，又出现了把 Unicode 编码转化为 "可变长编码" 的 UTF-8 编码。UTF-8 编码把一个 Unicode 字符根据不同的数字大小编码成 1-6 个字节，常用的英文字母被编码成 1 个字节，汉字通常是 3 个字节，只有很生僻的字符才会被编码成 4-6 个字节。如果你要传输的文本包含大量英文字符，用 UTF-8 编码就能节省空间：

字符 | ASCII    | Unicode           | UTF-8
----|----------|-------------------|-------
A   | 01000001 | 00000000 01000001 | 01000001
中  |   x      | 01001110 00101101 | 11100100 10111000 10101101

(插一句，UTF-8 是怎么来解析的，它怎么知道当前这个字符该用几个字节来解析，它是根据每个节字的前缀来解析的，比如上例中，01000001，如果遇到一个字节，最高比特是 0，说明是 ASCII 码，那它就把这一个字节当作一个独立的字符，而 11100100，最高比特不是 0，那就看它有多少个连续的 1，有多个少 1，就表示这个字符该用多少个字节来解析，这里有三个 1，所以接下来，包括自己在内的 3 个字节来表示这个字符，详细看这里 [字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html))

Python3 中，在内存中字符串统一用 Unicode 编码。

Python3 提供了 ord() 函数获取字符的 Unicode 编码的整数形式，chr() 则将 Unicode 编码的整数形式转换成字符。

    >>> ord('A')
    65
    >>> ord('中')
    20013
    >>> chr(66)
    'B'
    >>> chr(25991)
    '文'

还可以直接用 Unciode 编码表示字符串。

    >>> '\u4e2d\u6587'
    '中文'

由于 Python 的字符串类型是 str，在内存中以 Unicode 表示，一个字符对应若干个字节。如果要在网络上传输，或者保存到磁盘上，就需要把 str 变为以字节为单位的 bytes。(即 bytes 是用来表示 UTF-8 等编码的字符的)

Python3 提供了 encode() 方法，将 Unicode 编码的 str 类型，转换成指定编码 (如 utf-8) 的 bytes 类型。

    >>> 'ABC'.encode('ascii')
    b'ABC'
    >>> '中文'.encode('utf-8')
    b'\xe4\xb8\xad\xe6\x96\x87'
    >>> '中文'.encode('ascii')
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)

转换时不一定能成功，比如中文就不能转成 ascii 编码。

在 bytes 中，无法显示为 ASCII 字符的字节，用 `\x##` 显示。

反过来，如果我们从网络或磁盘上读取了字节流，那么读到的数据就是 bytes。要把 bytes 变为 str，就需要用 decode() 方法。

    >>> b'ABC'.decode('ascii')
    'ABC'
    >>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
    '中文'

如果 bytes 中包含无法解码的字节，decode() 方法会报错。

len() 函数，如果参数是 str 类型，则计算字符数，如 len('中文') 结果为 2，如果参数为 bytes 类型，则计算字节数。

    >>> len(b'ABC')
    3
    >>> len(b'\xe4\xb8\xad\xe6\x96\x87')
    6
    >>> len('中文'.encode('utf-8'))
    6

Python 源码中如果包含中文时，在文件头部声明用 utf-8 编码读取此文件。

    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-

字符串的格式化，有两种方式：

    >>> 'Hi, %s, you have $%d.' % ('Michael', 1000000)
    'Hi, Michael, you have $1000000.'

    >>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
    'Hello, 小明, 成绩提升了 17.1%'

### list 和 tuple

略。和 Python2 没有什么不同。

### 条件判断

略。

    if xxx:
      ...
    elif xxx:
      ...
    else:
      ...

### 循环

略。

两种形式

- for x in y:
- while x:

用 range(n) 来生成一个系列，用 list() 可以把 range 转换成一个 list。

    >>> range(5)
    range(0, 5)
    >>> list(range(5))
    [0, 1, 2, 3, 4]

break / continue。

### dict 和 set

略，和 Python2 差不多。

字典用 {}，相关方法 get(), pop()，判断 key 是否存在可以用 in 关键字。

    >>> d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
    >>> d['Michael']
    95
    >>> 'Thomas' in d
    False

set，没有简化符号可用 (那为什么下面的结果用 {} 来表示 set，奇怪)。

    >>> set([1,1,2,3])
    {1, 2, 3}

set 可以做数学意义上的交集，并集。

    >>> s1 = set([1, 2, 3])
    >>> s2 = set([2, 3, 4])
    >>> s1 & s2
    {2, 3}
    >>> s1 | s2
    {1, 2, 3, 4}

## 函数

定义参数：

    def foo(bar):
      pass

判断参数类型：isinstance(x, (type1, type2))

    def my_abs(x):
        if not isinstance(x, (int, float)):
            raise TypeError('bad operand type')
        if x >= 0:
            return x
        else:
            return -x

### 函数的参数

Python 的函参数设计得有点复杂，种类有点多，需要时再细看。

- 位置参数
- 默认参数
- 可变参数：*args，args 接收到的是一个 tuple
- 关键字参数：**kw，kw 接收到的是一个 dict
- 命名关键字参数：限制了 **kw 中 kw 的 key

定义默认参数要牢记一点：默认参数必须指向不变对象! 否则有大坑。

### 递归与尾递归

递归就不解释了。

尾递归，函数在返回时，只调用自身，不包含表达式 (即调用自身，只改变参数)。这样，编译器或解释器可以对尾递归进行优化，始之无论进行多少次调用，都只占用一个栈桢，不会出现栈溢出的情况。

    def fact(n):
        return fact_iter(n, 1)

    def fact_iter(num, product):
        if num == 1:
            return product
        return fact_iter(num - 1, num * product)

不过，遗憾的是，Python 并没有对尾递归做优化，所以...然并卵。

## 高级特性

### 切片

略。用法太多，记不住，需要时看文档。

### 迭代

    for ... in ...

### 列表生成式

Python 的独门绝技。

    [ x for ... in ... if ]

    >>> [x * x for x in range(1, 11) if x % 2 == 0]
    [4, 16, 36, 64, 100]

还可以使用两层循环，可以生成全排列：

    >>> [m + n for m in 'ABC' for n in 'XYZ']
    ['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']

### 生成器

应该是 JavaScript 的 generator 是一个作用和一个原理。

列表生成式，一次性把整个列表生成，容易造成空间浪费。生成器，需要时再生成。

果然是一模一样的，使用 next() 方法去索取下一个需要的值，用 yield 来产生此次需要的这个值。

当然，不需要用 next() 显示调用，用 for ... in ... 来迭代。

如果创建生成器，很简单，把列表生成式的 [] 替换成 () 即可。

    >>> L = [x * x for x in range(10)]
    >>> L
    [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
    >>> g = (x * x for x in range(10))
    >>> g
    <generator object <genexpr> at 0x1022ef630>

    >>> for n in g:
    ...     print(n)

定义 generator 的另一种方式，在函数中使用 yield 关键字，如果一个函数中包含 yield 关键字，那么这个函数就不再是一个普通函数，而是一个 generator。

(JavaScript 是需要显式地在函数前加 `*` 星号)

    def fib(max):
        n, a, b = 0, 0, 1
        while n < max:
            yield b
            a, b = b, a + b
            n = n + 1
        return 'done'

    >>> f = fib(6)
    >>> f
    <generator object fib at 0x104feaaa0>

明白了!

### 迭代器

- 凡是可作用于 for 循环的对象都是 Iterable 类型；
- 凡是可作用于 next() 函数的对象都是 Iterator 类型，它们表示一个惰性计算的序列；
- 集合数据类型如 list、dict、str 等是 Iterable 但不是 Iterator，不过可以通过 iter() 函数获得一个 Iterator 对象。

Python 的 for 循环本质上就是通过不断调用 next() 函数实现的，例如：

    for x in [1, 2, 3, 4, 5]:
        pass

实际上完全等价于：

    # 首先获得 Iterator 对象:
    it = iter([1, 2, 3, 4, 5])
    # 循环:
    while True:
        try:
            # 获得下一个值:
            x = next(it)
        except StopIteration:
            # 遇到 StopIteration 就退出循环
            break

## 函数式编程

> 函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

> 函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

> Python 对函数式编程提供部分支持。由于 Python 允许使用变量，因此，Python 不是纯函数式编程语言。

### 高阶函数

High-order Function。

函数名也是变量，它是指向函数的变量。

> 既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函

最简单的高阶函数：

    def add(x, y, f):
        return f(x) + f(y)

使用示例：

    >>> add(-5, 6, abs)
    11

几个内置的高阶函数：

- map / reduce
- filter
- sorted

与其它语言有差别的，map() 函数得到的结果是一个 map object，还需要手动自己再转换一下。

    >>> def f(x):
    ...   return x*x
    ...
    >>> r = map(f, [1,2,3])
    >>> r
    <map object at 0x104b45f28>
    >>> list(r)
    [1, 4, 9]

上例中 f 函数可以用 lambda 来简化，后面会讲。

filter 和 map 一样，得到的结果需要再手动转一下。

sorted() 接收一个 key 参数，key 参数为函数名。

    >>> sorted([36, 5, -12, 9, -21], key=abs)
    [5, 9, -12, -21, 36]

### 返回函数

就是闭包啦，和 JavaScript 一样。在函数中返回函数。

    def lazy_sum(*args):
        def sum():
            ax = 0
            for n in args:
                ax = ax + n
            return ax
        return sum

    >>> f = lazy_sum(1, 3, 5, 7, 9)
    >>> f
    <function lazy_sum.<locals>.sum at 0x101c6ed90>

    >>> f()
    25

在闭包中使用循环时，Python 也有和 JavaScript 一样的问题，JavaScript 中使用箭头函数解决，Python 可以用 lambda 解决。

### 匿名函数

就是 lambda 啦。

    >>> list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
    [1, 4, 9, 16, 25, 36, 49, 64, 81]

把匿名函数赋值给一个变量。

    >>> f = lambda x: x * x
    >>> f
    <function <lambda> at 0x101c6ef28>
    >>> f(5)
    25

作为返回值。

    def build(x, y):
        return lambda: x * x + y * y

匿名函数有个限制，就是只能有一个表达式，不用写return，返回值就是该表达式的结果。(嗯，没有 JavaScript 强大啊)

### 装饰器

(JavaScript 将即支持此功能... 这样看来，JavaScript 是在向 Python 看齐啊)

(感觉和 Java 的 AOP 编程模式是一个意思，而 Ruby 通过 alias 也可以支持类似的功能)

假设我们要增强一个函数函数的功能，比如，在函数调用前后自动打印日志，但又不希望修改这个函数的定义，这种在代码运行期间动态增加功能的方式，称之为 "装饰器" (Decorator)。

本质上，decorator 就是一个返回函数的高阶函数。所以，我们要定义一个能打印日志的 decorator，可以定义如下：

    def log(func):
        def wrapper(*args, **kw):
            print('call %s():' % func.__name__)
            return func(*args, **kw)
        return wrapper

哎，就和 Ruby 的 alias 相似啦。对原来的函数进行包装，返回包装后的函数对象。

    @log
    def now():
        print('2015-3-25')

相当于

    now = log(now)

如果 decorator 本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，可以自定义 log 文本。

    def log(text):
        def decorator(func):
            def wrapper(*args, **kw):
                print('%s %s():' % (text, func.__name__))
                return func(*args, **kw)
            return wrapper
        return decorator

(有点像函数柯里化啊...)

使用

    @log('execute')
    def now():
        print('2015-3-25')

当相于

    now = log('execute')(now)

(这不就是函数柯里化的用法嘛...)

使用 @functools.wraps() 方法来使返回的 wrap 函数的名字和被包装的函数名字一样。即 `wrapper.__name__ == func.__name__`

    import functools

    def log(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print('call %s():' % func.__name__)
            return func(*args, **kw)
        return wrapper

### 偏函数

(有意思，其实也就只是对函数的一层封装而已，没啥特别的)

使用 functools.partial() 方法。

> 所以，简单总结 functools.partial 的作用就是，把一个函数的某些参数给固定住 (也就是设置默认值)，返回一个新的函数，调用这个新函数会更简单。

手动实现，固定住 `base=2` 参数：

    def int2(x, base=2):
        return int(x, base)

用 functools.partial() 实现：

    >>> int2 = functools.partial(int, base=2)
    >>> int2('1000000')
    64
