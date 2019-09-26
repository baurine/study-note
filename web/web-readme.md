# Web Note

关于 Ruby & Rails 的所有笔记都移到 [Rails Study](https://github.com/baurine/rails-study) 项目中了。

关于 Node 的笔记在 [JS Study](https://github.com/baurine/js-study) 项目中。

关于 Go 的所有笔记移到 [Go Study](https://github.com/baurine/go-study) 项目中了。

## Web 开发框架选择

- [2019 年 Web 开发路线图发布](https://www.infoq.cn/article/DcIG3BX0DG*YrcyJCttC)
- [The 2019 Web Developer RoadMap](https://hackernoon.com/the-2019-web-developer-roadmap-ab89ac3c380e)
- [roadmap.sh](https://roadmap.sh/)

![](https://raw.githubusercontent.com/kamranahmedse/developer-roadmap/master/images/backend.png)

关于 web 开发框架选择：

- 动态语言：ruby (rails) / node.js (express | koa | nest.js) / python (django | flask) / php (laravel)
- 静态语言：go (gin) / java | kotlin (spring) / rust / c# (.net)
- 函数式语言：scala / elixir | erlang / clojure / haskell

列了表才发现其实 web 框架的选择并不多啊 (其实真正流行的语言好像也就这些，再加上 c/c++/swift)，函数式语言太小众，一般人不考虑。go (2009) / node.js (2009) / rust (2015) 近些年才有，rails 和 django 是 2005 左右才诞生，所以 2005 年之前的人们几乎只能用 php (1998) / java / c#，而这三者里 php 算是最简单，开发最快速的，难怪 php 大行其道，市场占有率高，原来它诞生得早啊，人们没得选，这就是所谓的时机啊。c# 则是一手好牌给打烂了，不能跨平台，封闭在 windows 上。也难怪 rails 在 2005 发布时能如此大受欢迎，相比 php / java / c# 简直是让人耳目一新啊。

我个人的选择，首先排除小众语言，其次排除 php，个人不是很喜欢这门语言，最后排除 python，python 相比 ruby 没有明显优势，做爬虫可以用 python，rust 则用来作系统级开发。因此剩余可能考虑的：ruby (rails) / node.js (express | nest.js) / go (gin) / java (spring)。rails 操作数据库实在是太方便了。express 和 gin 比较轻量级，spring 则比较重量级。(这些框架就差 spring 没有研究过了。)
