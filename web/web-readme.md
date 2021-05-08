# Web Note

独立的 repo：

- [JS Study](https://github.com/baurine/js-study)
- [Go Study](https://github.com/baurine/go-study)
- [Rust Study](https://github.com/baurine/rust-study)
- [Python Study](https://github.com/baurine/python-study)
- [Rails Study](https://github.com/baurine/rails-study)

## Web 开发框架选择

- [2019 年 Web 开发路线图发布](https://www.infoq.cn/article/DcIG3BX0DG*YrcyJCttC)
- [The 2019 Web Developer RoadMap](https://hackernoon.com/the-2019-web-developer-roadmap-ab89ac3c380e)
- [roadmap.sh](https://roadmap.sh/)

![](https://raw.githubusercontent.com/kamranahmedse/developer-roadmap/master/images/backend.png)

关于 web 开发框架选择：

- 动态语言：ruby (rails) / node.js (express | koa | nest.js) / python (django | flask) / php
- 静态语言：go (gin) / java | kotlin (spring) / rust / c# (.net)
- 函数式语言：scala / elixir | erlang / clojure / haskell

列了表才发现其实 web 框架的选择并不多啊 (其实真正流行的语言好像也就这些，再加上 c/c++/swift)，函数式语言太小众，一般人不考虑。go (2009) / node.js (2009) / rust (2015) 近些年才有，虽然 ruby 和 python 诞生得也很早，上世纪九十年代初就有了，但它们不是为 web 而生的语言，不像 php 生来就是用来做 web 开发的，它们相应的 web 框架 rails 和 django 直到 2005 左右才诞生，所以 2005 年之前的人们几乎只能用 php (1998) / java / c#，而这三者里 php 算是最简单，开发最快速的，难怪 php 大行其道，市场占有率高，原来它诞生得早啊，人们没得选，这就是所谓的时机啊。c# 则是一手好牌给打烂了，不能跨平台，封闭在 windows 上。也难怪 rails 在 2005 发布时能如此大受欢迎，相比 php / java / c# 简直是让人耳目一新啊。

我个人的选择，首先排除小众语言，其次排除 php，个人不是很喜欢这门语言，最后排除 python，python 相比 ruby 没有明显优势，做爬虫可以用 python，rust 则用来作系统级开发。因此剩余可能考虑的：ruby (rails) / node.js (express | nest.js) / go (gin) / java (spring)。rails 操作数据库实在是太方便了。express 和 gin 比较轻量级，spring 则比较重量级。(这些框架就差 spring 没有研究过了。)

## 编程语言总结

通过上面一小节发现，虽然印象中会觉得现在各种语言百花齐放，种类多得不行，但实际掐指一算，主流的也就那几种而已，其实算得上少得可怜。

- 静态语言 (需要编译)：c / c++ / java | kotlin | (scala) / c# / go / rust / oc | swift
- 动态语言 (脚本语言，无需编译)：php / python / ruby / javascript
- 函数式语言：scala / clojure / erlang | elixir / haskell (只有 scala 勉强算主流)

按作用分：

- 桌面开发：c++ / c# / javascript
- 移动开发：java | kotlin / oc | swift / javascript / dart
- 后端：java|kotlin / c# / go / rust / php / python / ruby / node.js (几科所有语言都能写后端)
- 底层系统级开发：c / c++ / go / rust (排除 c / c++ 后，只剩 go 和 rust，如果又不能 gc 的话，rust 就是唯一的选择了)

先排除函数式语言，c 太底层，c++ 太复杂难掌握，oc 和 swift 仅用于 iOS 编程，c# 主要用于 windows 开发，php 不喜欢，剩下也就只有 java|kotlin / go / rust / python / ruby / javascript 了，妈呀，这也太少了，而我已经把它们几乎都用过了。由此看，ruby 不会衰落，go 和 rust 会成为未来，因为人们的选择太少了。未来继续投资这几门语言。

不过，虽然语言就这几种，但架不住框架和第三库多啊。

每门语言其实大部分都很相似，但每门语言又有它自己的一些特色，没有特色它就没有产生的必要了，也是它们脱颖而出的原因。

- java：严格 OOP，一切不能脱离 class 存在，有 GC，相比 c/c++ 不用自己管理内存，方便了很多
- kotlin：基于 jvm，OOP 和函数式的混合泛式，非空机制 ...
- go：和 c 一样简洁的语法，上手快，有 GC，内置并发机制，最终编译生成一个无依赖的二进制文件 ...
- rust：混合泛式，所有权，内存安全 ...
- python：动态语言，混合泛式，语法简单，上手快，胶水语言 ...
- ruby：动态语言，混合泛式，语法优雅 ...
- javascript：完全异步编程，前端的唯一选择 ...

这么想想，为什么大公司大多都选择 java 作为主要开发语言，因为几乎都没得选啊。kotlin / go / rust 是近些年才有的，生态还跟不上。当年 twitter 从 ruby 转用 scala 也能理解了，可能是因为不想用动态语言了，所以就从静态语言里选，估计觉得 java 不够 fashion，kotlin 当时又还没有，那就只能选 scala 了啊。

一门语言的学习主要包括三部分：基本语法，库，框架。

- 基本语法
  - 基本数据类型：数值，布尔值，字符/字符串
  - 集合：数组，map (还有一些语言有列表，元组，set，切片...)
  - 复杂类型：类，接口，结构体，枚举
  - 方法 / 函数 / 闭包
  - 逻辑控制：顺序，条件，循环 (复杂的条件逐渐被模式匹配替代，循环在一些语言中被 for 统一)
- 库 (包含标准库)
  - 接受输入，格式化输出
  - 日志
  - 网络
  - 操作文件系统
  - 多线程/并发
  - UI (用于客户端开发)
    - 布局
    - 绘图
    - 动画
    - 处理触摸事件
  - 测试
- 框架
  - web 框架：mvc (主要是路由，操作数据库，处理表单等)
- 其它
  - 设计模式
  - 包管理
  - ...

对于客户端开发，一大部分工作是处理 UI，主要又包括布局，绘图，动画，处理触摸事件。

学习一门新的语言，不要纠缠于语法细节，使用的时候不对自然会报错，而是要从宏观上理解它的原理，新特性，接受它的思想。

总结出它和其它语方不一样的地方，侧重学习这些不一样的地方，不常用的就跳过，日后遇到再说。

学习语言在于理解并接受别人的思想，而不是去排斥它。总结出相似的东西，触类旁通，举一反三。

一些相关文章：

- [计算机更新这么快，怎么编程语言还是二十多年前的？](https://www.techug.com/post/why-programming-languages-are-more-than-20-years-old.html)
- [我们为什么要选择小众语言 Rust 来开发软件？](https://www.techug.com/post/why-we-choose-rust-to-dev.html)
- [Rust 语言恰巧是一门解决了 Go 语言所有问题的语言](https://www.techug.com/post/the-success-of-go-heralds-that-of-rust.html)
- [为什么是 Go 而不是 Rust](https://www.techug.com/post/why-go-and-not-rust.html)
- [前 10 大编程语言你会几种？](https://www.techug.com/post/most-popular-programming-languages-best-for-developers.html)

[计算机更新这么快，怎么编程语言还是二十多年前的？](https://www.techug.com/post/why-programming-languages-are-more-than-20-years-old.html) 这篇文章很有意思啊，有编程语言的诞生划分成 4 个时代：

- 第一波是 50~70 年代，数学驱动，理念是翻译公式。比如 Fortran (Formula Translating System, 1957)，Lisp (1958) ... 特点是学术范，很多超越时代的思想和特性...
- 第二波是 70~90 年代，系统驱动，1970 PASCAL，1972 C，1983 C++，这些语言注重性能
- 第三波是 90~2012 年左右，互联网驱动 (?)，1991 python，1993 ruby，1994 php，1995 java，1996 javascript (python / ruby / php 居然都比 java 早)，没有互联网提供的 web 服务器端开发机会，大约前 3 个都火不起来...
- 第四波，现在，厂商驱动，Apple -> Swift，Google -> Go，Microsoft -> TypeScript ...

从 [我们为什么要选择小众语言 Rust 来开发软件？](https://www.techug.com/post/why-we-choose-rust-to-dev.html) 这篇文章可以看出，对性能要求高的系统底层开发，如果排除掉 c/c++ 后，几乎就真的只有 rust 可以选择了。

对于这篇文章 - [为什么是 Go 而不是 Rust](https://www.techug.com/post/why-go-and-not-rust.html)，我想说，不同场景使用不同语言。对于系统级开发，如果可以容忍 GC 的地方，优先用 Go，否则考虑 Rust。确实，Go 能做的 Rust 都能做，但 Rust 能做的，Go 并不一定能做，看似 Rust 可以完全替代 Go，但 Rust 相比 Go 存在的缺点：编译慢，编码效率低，上手难度大，Go 内置 goroutine 处理并发简单，Rust 处理并发复杂... 而 web 开发肯定优先考虑 rails / java 之类的，爬虫那毫无疑问首选 python。
