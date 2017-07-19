# Rails Misc Note

会记录一些 rails 项目中遇到的比较常见的问题及解决方法。

## rvm 和 gemset

rvm 用来管理 ruby 版本，gemset 是 rvm 的功能之一，用来管理 gem 包版本。rvm 官网 [rvm.io](https://rvm.io/)

rvm 其实跟 rails 没有什么关系，但 rvm 是 rails 开发中常用的工具之一。

新 clone 一个 rails 工程，需要指定 ruby 版本，且安装的 gem 不要影响现有的其它 rails 工程，该怎么做呢。

首先，进入当前工程目录，用 `rvm list` 列举当前系统安装的 ruby 版本：

    > rvm list

    rvm rubies

       ruby-1.9.3-p484 [ x86_64 ]
       ruby-2.2.0 [ x86_64 ]
    =* ruby-2.3.0 [ x86_64 ]
       ruby-2.3.2 [ x86_64 ]
       ruby-2.3.3 [ x86_64 ]

    # => - current
    # =* - current && default
    #  * - default

如果没有所需要的 ruby 版本，则用 `rvm install` 安装，安装之前可以用 `rvm list known` 查看一下当前都有哪些版本，然后从中选一个。

    > rvm install 2.4

如果已经安装了所需版本，但不是当前使用的版本，则用 `rvm use` 切换版本：

    > rvm use 2.4

用 `rvm current` 查看当前使用的 ruby 版本，当然用上面的 `rvm list` 也是可以的。

我们可以把目标版本号写到当前目录下的 `.ruby-version` 配置文件中，这样，每次进入这个目录时，就会自动切换到这个 ruby 版本，不需要我们再手动运行 `rvm use` 来切换。

    > echo '2.4' > .ruby-version

然后，我们创建一个 gemset 来存放此工程用到的所有 gem，一般 gemset 的名称为项目名称，或加上环境，比如 `myweb`，`myweb_staging`，`myweb_production`：

    > rvm gemset create myweb

然后，用 rvm gemset use 命令来切换到使用此 gemset：

    > rvm gemset use myweb

之后，运行 `bundle` 后所安装的 gem 就会放在此 gemset 所对应的目录中。

我们可以把 gemset 写到 `.ruby-gemset` 中，这样，每次进入这个目录时，就会自动切换到这个 gemset，不用再手动运行 `rvm gemset use` 命令。

    > echo 'myweb' > .ruby-gemset

需要注意的是，gemset 是与 ruby 版本号相关的，如果在 ruby 2.4 下创建，那么就只有在 ruby 2.4 下生效，如果切换了另一个 ruby 版本，这个 gemset 就不存在。所以，实际这个 gemset 的全名是 `2.4@myweb`，可以用 `rvm use` 命令一步到位，同时指定使用的 ruby 版本和 gemset：

    > rvm use 2.4@myweb

可以把 `.ruby-version` 和 `.ruby-gemset` 的内容写到一个配置文件 `.versions.conf` 中，格式如下：

    ruby=ruby-2.4
    ruby-gemset=myweb

## sort by associate count

根据关联记录的数量来排序。比如有一个用户表 users 和评论表 reviews，有些用户可能有多个评论，有些用户没有评论。现在有一个显示所有用户及它的评论数的页面，要求可以按照评论数来对用户进行排序。

此时，如果用默认的 joins 即 inner join，会发现，那些评论数为 0 的用户不见了。怎么办呢，用 `left_joins`，如下所示：

    User.left_joins(:reviews)
        .group(:id)
        .order('count(reviews.id) desc')

注意，order 中的必须是 `count(reviews.id)` 或 `count(reviews.*)`，而不能是 `count(*)`，因为对于评论数为 0 的用户来说，`count(*)` 的结果是 1，而 `count(reviews.*)` 的结果才是正确的 0。

我也写了一部分解释在 [Postgresql Study Note](../database/postgresql-study-note.md) 中。

## 多态 (Polymorphic)

一般的文章讲多态只讲到 model 怎么写，忽略了 route 和 controller 该怎么写，找到了一篇讲解多态并包含 route 和 controller 的文章，可以认为是最佳实践：[Polymorphic Routes in Rails](http://thelazylog.com/polymorphic-routes-in-rails/)。(但这篇文章没有讲到 migration 该怎么写，我应该再写一篇文章，把二者结合起来)。

使用：
<http://caok1231.com/rails/2014/10/13/many-to-many-and-polymorphic.html>

剩下的待补充。
