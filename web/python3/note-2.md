# Python3 - Note 2

## 模块

一个 .py 文件就是一个模块，一个目录下放置 `__init__.py` 使这个目录成为一个模块，否则就是普通目录。

使用 import 在代码中引用模块。

详略。看 Python2 笔记。

## 面向对象编程

Python 的 class。

    class Student(object):

        def __init__(self, name, score):
            self.name = name
            self.score = score

        def print_score(self):
            print('%s: %s' % (self.name, self.score))

`__init__` 为构造函数。

Python 中 this 用 self 表示，且作为成员方法的第一个参数，显式声明。

类内部用 `_` 或 `__` 下划线开头的变量，视为私有成员变量，外部不要直接访问。

继承和多态，略。只是继承的语法比较特殊，父类放在括号中：

    class Animal(object):
        def run(self):
            print('Animal is running...')

    class Dog(Animal):
        pass

### 获取对象信息

- type() - 判断对象类型
- isinstance() - 判断对象的 class 类型
- dir() - 获得一个对象的所有属性和方法

详略。

### 实例属性和类属性

略。

## 面向对象高级编程

### 使用 `__slots__`

wow，Python 和 JavaScript 真的好像啊。

JavaScript 可以给一个对象任意绑定新的属性和方法，还可以给一个 class 扩展新的方法 (用 prototype)，Python 也一样可以。

但是，如果这个类不能其它人对它指手划脚，东加西加，它可以用 `__slots__` 来声明只允许别人添加的属性，不在这个名单中的属性不允许添加。(有啥用啊? 为什么要做这个限制呢)

而且这个声明对子类没有效果...

    class Student(object):
        __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称

    >>> s = Student() # 创建新的实例
    >>> s.name = 'Michael' # 绑定属性'name'
    >>> s.age = 25 # 绑定属性'age'
    >>> s.score = 99 # 绑定属性'score'
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Student' object has no attribute 'score'

### 使用 @property

类似 Objcet-C 中的 @property，自动生成 getter 和 setter 函数。

@property 也是一种装饰器函数。

详略，需要时再看。

### 多重继承

还真是和 C++ 一样的多重继承。但不要真的这么用。

实际用的时候，继承一个主类，其余的用类来模拟接口，这些接口又称为 MixIn。

    class Animal(object):
        pass

    # 大类:
    class Mammal(Animal):
        pass

    # 用类模拟的接口，即 MixIn
    class RunnableMixIn(object):
        def run(self):
            print('Running...')

    class FlyableMixIn(object):
        def fly(self):
            print('Flying...')

使用，对于需要 Runnable 功能的动物，就多继承一个 Runnable，例如 Dog：

    class Dog(Mammal, RunnableMixIn):
        pass

对于需要 Flyable 功能的动物，就多继承一个 Flyable，例如 Bat：

    class Bat(Mammal, FlyableMixIn):
        pass

### 定制类

形如 `__xx__` 的变量或方法，是有特殊用途的，在类内部，它一般是由系统来调用，而不是自己手动调用。

`__str__` 方法。当使用 `print(obj)` 时，实际是打印这个 obj 的 `__str__` 方法的返回值。默认对象的 `__str__` 只返回内存地址等有限信息，没什么大用，因此，我们可以重写类的 `__str__` 方法来返回一些我们想要的信息。

    >>> class Student(object):
    ...     def __init__(self, name):
    ...         self.name = name
    ...     def __str__(self):
    ...         return 'Student object (name: %s)' % self.name
    ...
    >>> print(Student('Michael'))
    Student object (name: Michael)

`__repr__` 方法，类似 `__str__` 方法，只不过它不用 print() 时的输出，而是用于在 REPL 中输入这个变量时的输出。

    >>> s = Student('Michael')
    >>> s
    <__main__.Student object at 0x109afb310>

可以重新定义 `__repr__` 方法，不过一般来说它和 `__str__` 的实现是一样的，因此我们可以用 `__repr__ = __str__` 来简化代码。

`__iter__` 和 `__next__` 配合，使对象成为可迭代对象。

    class Fib(object):
        def __init__(self):
            self.a, self.b = 0, 1 # 初始化两个计数器a，b

        def __iter__(self):
            return self # 实例本身就是迭代对象，故返回自己

        def __next__(self):
            self.a, self.b = self.b, self.a + self.b # 计算下一个值
            if self.a > 100000: # 退出循环的条件
                raise StopIteration()
            return self.a # 返回下一个值

`__getitem__`，使对象可以用于索引取值。

    class Fib(object):
        def __getitem__(self, n):
            a, b = 1, 1
            for x in range(n):
                a, b = b, a + b
            return a

    >>> f = Fib()
    >>> f[0]
    1

`__getattr__`，基本可以理解成 Ruby 的 `method_missing` 方法，当调用的属性不存在时，会进入到 `__getattr__` 方法中，默认是抛出异常，你可以重写它来实现一些逻辑。

`__call__`，对实例自身进行调用。

    class Student(object):
        def __init__(self, name):
            self.name = name

        def __call__(self):
            print('My name is %s.' % self.name)

调用方式如下：

    >>> s = Student('Michael')
    >>> s() # self参数不要传入
    My name is Michael.

还有很多类似的方法，详细 Python 的文档。

### 枚举类

Enum，详略。

### 使用元类

元类? 难道是类似 Ruby 的元编程。

type() 方法与 metaclass，确实是类似 Ruby 的元编程。

type() 方法可以说是类似 Ruby 的 Class.new()，JavaScript 的 new Function()。

暂时不会用到，先跳过。理解其思想就行了。

## 错误、调试和测试

### 错误处理

捕捉异常：try...except...else...finally

    try:
        print('try...')
        r = 10 / int('2')
        print('result:', r)
    except ValueError as e:
        print('ValueError:', e)
    except ZeroDivisionError as e:
        print('ZeroDivisionError:', e)
    else:
        print('no error!')
    finally:
        print('finally...')
    print('END')

所有异常继随自 BaseException。

用 logging 模块记录错误信息：

    try:
        bar('0')
    except Exception as e:
        logging.exception(e)

抛出异常：raise error

### 调试

- print() / logging()
- assert
- 调试器 pdb (类似 gdb)
- IDE (VS Code / PyCharm)

### 单元测试

使用 unittest 模块。测试类继承自 unittest.TestCase 类，测试方法必须以 `test_` 开头。也有 setUp 和 tearDown 方法。

详略。

### 文档测试

Python 内置的 "文档测试" (doctest) 模块可以直接提取注释中的代码并执行测试。

详略。

## IO 编程

简单复习，和 Python2 大致相同。

### 文件读写

- 打开文件 open()
- 读文件 f.read(), f.readlines()
- 写文件 f.write()
- 关闭 f.close()

### StringIO 和 BytesIO

> StringIO 和 BytesIO 是在内存中操作 str 和 bytes 的方法，使得和读写文件具有一致的接口。

和文件读写具有一样的方法，读 write(), read(), readlines()

详略。

### 操作文件和目录

> Python 的 os 模块封装了操作系统的目录和文件操作，要注意这些函数有的在 os 模块中，有的在 os.path 模块中。

详略。

### 序列化

> Python 语言特定的序列化模块是 pickle，但如果要把序列化搞得更通用、更符合 Web 标准，就可以使用 json 模块。

> json 模块的 dumps() 和 loads() 函数是定义得非常好的接口的典范。当我们使用时，只需要传入一个必须的参数。但是，当默认的序列化或反序列机制不满足我们的要求时，我们又可以传入更多的参数来定制序列化或反序列化的规则，既做到了接口简单易用，又做到了充分的扩展性和灵活性。

## 进程和多线程

### 多进程

Unix/Linux 提供了 fork() 系统调用创建子进程。

Python 的 os 模块封装了常见的系统调用，包括 fork，可以在 Python 程序中轻松创建子进程。(Windows 下不能工作)

如果要考虑到在 Windows 下运行，使用跨平台的多进程支持，multiprocessing 模块就是跨平台的多进程模块。详略。

如果要启动大量的子进程，可以用进程池的方式批量创建子进程，使用 Pool 对象。

**子进程**

如果子进程不是自身，而是一个外部进程，创建了这种子进程后，还要控制子进程的输入和输出。

使用 subprocess 模块。

进程间通信，可以使用 Queue、Pipes 等多种方式，封装在 multiprocessing 模块中。

### 多线程

Python 提供两个模块支持多线程的使用，`_thread` 和 threading 模块，前者是底层模块，后者是高层模块，是对前者的封装，一般情况下，我们只需要使用到后者。

基本使用，创建一个 threading.Thread 实例，然后调用其 start() 方法开始执行子线程。

多线程中要考虑加锁的问题。

### ThreadLocal

> 一个 ThreadLocal 变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，互不干扰。ThreadLocal 解决了参数在一个线程中各个函数之间互相传递的问题。

> ThreadLocal 最常用的地方就是为每个线程绑定一个数据库连接，HTTP 请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。(nice!)

### 进程 vs. 线程

略。

计算密集型应用，需要的是代码的执行效率，因此 Python 这类脚本语言不合适，需要 C/C++ 这种底层语言，但 IO 密集型应用，99% 的时间花在 IO 上，花在 CPU 上的时间很少，因此，用 C/C++ 来做效果提升不是很显著，反而降低了开发效率。

> 对应到 Python 语言，单线程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序。

### 分布式进程

> 在 Thread 和 Process 中，应当优选 Process，因为 Process 更稳定，而且，Process 可以分布到多台机器上，而 Thread 最多只能分布到同一台机器的多个 CPU 上。

> Python 的 multiprocessing 模块不但支持多进程，其中 managers 子模块还支持把多进程分布到多台机器上。一个服务进程可以作为调度者，将任务分布到其他多个进程中，依靠网络通信。由于 managers 模块封装很好，不必了解网络通信的细节，就可以很容易地编写分布式多进程程序。

详略。

## 正则表达式

略。在 Python2 中做了详细笔记。使用 re 模块。

## 常用内建模块

- datetime
- collections - 提供了很多有用的集合类
- base64
- struct - bytes 和其他二进制数据类型的转换
- hashlib - 提供常见的摘要算法 (就是哈希算法啦)，如 MD5, SHA1
- hmac - 对哈希进行加盐? (还没完全理解，回头再看)
- itertools - 提供了非常有用的用于操作迭代对象的函数，如 count(), cycle(), repeat(), chain(), groupby() ...
- contextlib - 帮助你简化实现上下文管理，以便可以使用 with 语句
- urllib - http 客户端，实现 http 的 get/post
- XML - 操作 XML
- HTMLParser - 解析 HTML

详略。

## 常用第三方模块

- Pillow - PIL 的 python 3.x 版本，操作图像
- requests - urllib 的替代品
- chardet - 检测编码
- psutil - process and system utilities，获取机器的系统信息

## virtualenv

Python 的多版本管理工具，类似 Ruby 的 rvm。

详略，实践时再看。

## 图形界面

Tkinter。略。

## 网络编程

(这里具体指的是写服务端，而不是客户端，客户端前面说了，用 urllib 或 requests 模块)

实现服务端是 Python 的主要用途之一。

这里介绍了用底层的 socket 模块实现 TCP Server/ UDP Server，了解就行，我们不般不会写这么底层的东西，都是用别人写好的框架。

## 电子邮件

SMTP 发送邮件，POP3 收取邮件，略。

## 访问数据库

SQLite，MySQL，SQLAlchemy ... 略。

现在没需要，不看，看了也记不住。

## Web 开发

HTTP 协议简介，略。HTML 简介，略。

### WSGI 接口

WSGI：Web Server Gateway Interface

> 无论多么复杂的 Web 应用程序，入口都是一个 WSGI 处理函数。HTTP 请求的所有输入信息都可以通过 environ 获得，HTTP 响应的输出都可以通过 `start_response()` 加上函数返回值作为 Body。

> 复杂的 Web 应用程序，光靠一个 WSGI 函数来处理还是太底层了，我们需要在 WSGI 之上再抽象出 Web 框架，进一步简化 Web 开发。

### 使用 Web 框架

介绍了 Flask 的简单使用，略。

其它框架：Diango, web.py, Bottle, Tornado ...

### 使用模板

Flask 默认的 HTML 模板是 jinja2 ... 详略。

## 异步 IO

(这下是向 JavaScript 看齐啦)

异步 IO 模型需要一个消息循环，在消息循环中，主线程不断地重复 “读取消息 - 处理消息” 这一过程：

    loop = get_event_loop()
    while True:
        event = loop.get_event()
        process_event(event)

消息模型其实早在应用在桌面应用程序中了。一个 GUI 程序的主线程就负责不停地读取消息并处理消息。所有的键盘、鼠标等消息都被发送到 GUI 程序的消息队列中，然后由 GUI 程序的主线程处理。(就是说嘛，Node.js 所谓的 EventLoop，不也一样嘛，有啥神秘的)

### 协程

(就是 generator 啦)

> 协程，又称微线程，纤程。英文名 Coroutine。

> 子程序 (就是函数啦) 调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。

> 协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。

协程的优势：

1. 执行效率高，不用切上下文
1. 不用锁

> 因为协程是一个线程执行，那怎么利用多核 CPU 呢？最简单的方法是多进程 + 协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

> Python 对协程的支持是通过 generator 实现的。

这里举了一个用协程实现的生产者-消费者模型，有意思。

> 整个流程无锁，由一个线程执行，produce 和 consumer 协作完成任务，所以称为 "协程"，而非线程的抢占式多任务。

(可以列到面试题里，尝试自己实现一下，用各种语言)

### asyncio

asyncio 是 Python 3.4 版本引入的标准库，直接内置了对异步 IO 的支持。

- asyncio 提供了完善的异步 IO 支持；
- 异步操作需要在 coroutine 中通过 yield from 完成；
- 多个 coroutine 可以封装成一组 Task 然后并发执行。

详略，因为从 Python 3.5 起提供了 async/await 的简略写法。(和 JavaScript 一模一样了)

### async/await

新语法。

Python 3.4 写法：

    @asyncio.coroutine
    def hello():
        print("Hello world!")
        r = yield from asyncio.sleep(1)
        print("Hello again!")

Python 3.5 写法：

    async def hello():
        print("Hello world!")
        r = await asyncio.sleep(1)
        print("Hello again!")

### aiohttp

> asyncio 可以实现单线程并发 IO 操作。如果仅用在客户端，发挥的威力不大。如果把 asyncio 用在服务器端，例如 Web 服务器，由于 HTTP 连接就是 IO 操作，因此可以用单线程 + coroutine 实现多用户的高并发支持。

> asyncio 实现了 TCP、UDP、SSL 等协议，aiohttp 则是基于 asyncio 实现的 HTTP 框架。

详略。

## 实践

列为 Todo 吧。

Done!
