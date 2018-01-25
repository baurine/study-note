# Redis Note

## References

- [Redis 教程](http://www.runoob.com/redis/redis-tutorial.html)
- [Redis 入门与应用教程](http://redisbook.rails365.net/467181)
- [Redis 命令参考](http://redisdoc.com)

## Note

不是很复杂，只是简单总结。

Redis 是一个内存数据库 (也可以持久化)，用来做 cache。

和 memcached 的比较：memcached 的数据只在内存中，且只支持一种最简单的键值对数据，键和值都是字符串。Redis 的数据是可以持久化到硬盘的，且支持丰富的数据类型，包括字符串、列表 (Lists)、哈希 (其实是 Map)、集合 (Sets)、排序集合 (Sorted Sets)、位图 (Bitmaps)。

针对不同的数据类型，使用不同的命令，比如字符串用 set/get、列表用 lpush/lpop/lrange、哈希用 hmset/hget ...

### Note 1

Note for [Redis 教程](http://www.runoob.com/redis/redis-tutorial.html)

#### 安装配置

[如何在 macOS 上安装 redis](https://medium.com/@djamaldg/install-use-redis-on-macos-sierra-432ab426640e)

分别安装服务端和客户端，服务端用 redis-server 启动，客户端用 redis-cli 启动。

配置文件在 redis.conf 中，常用配置：

    bind 127.0.0.1    # 绑定本机，避免远程访问
    maxmemory 1536mb  # 限制最大内存使用

也可以在启动客户端后，在客户端使用 `config set` 动态配置，通过 `config get` 查看配置。

#### 数据类型

Redis 支持五种数据类型：string (字符串)，hash (哈希)，list (列表)，set (集合) 及 zset (sorted set，有序集合)。(位图呢?)

**string**

    redis> set name runoob  # runoob 不需要引号，默认是字符串
    redis> get name
    "runoob"

**hash**

    redis> hmset myhash field1 "Hello" field2 "World"
    "OK"
    redis> hget myhash field1
    "Hello"
    redis> hget myhash field2
    "World"

**list**

    redis> lpush runoob redis
    (integer) 1
    redis> lpush runoob mongodb
    (integer) 2
    redis> lpush runoob rabitmq
    (integer) 3
    redis> lrange runoob 0 -1
    1) "rabitmq"
    2) "mongodb"
    3) "redis"

**set**

无序集合，没有重复元素

    redis> sadd runoob redis
    (integer) 1
    redis> sadd runoob mongodb
    (integer) 1
    redis> sadd runoob rabitmq
    (integer) 1
    redis> sadd runoob rabitmq
    (integer) 0
    redis> smembers runoob

    1) "rabitmq"
    2) "mongodb"
    3) "redis"

**zset**

有序集合，添加元素可以为这个元素设置一个分数，有序集合可以这个分数来排序

语法：

    zadd key score member

示例：

    redis> zadd runoob 2 redis
    (integer) 1
    redis> zadd runoob 1 mongodb
    (integer) 1
    redis> zadd runoob 0 rabitmq
    (integer) 1
    redis> zadd runoob 0 rabitmq
    (integer) 0
    redis> zrangebyscore runoob 1 1000

    1) "mongodb"
    2) "redis"

#### 命令

针对以上数据类型的命令，详略，需要时看参考手册就行。

稍微补充几个常用的命令。

删除某个 key：`del key`

在字符串数据类型中，如果字符串可以转换成数字，即像 "100" 这样的，那么可以用 incr, incrby, decr 之类的命令对它进行数学操作，即把这个值当作数值对待。示例：

    redis> set counter 100
    redis> get counter
    "100"
    redis> incr counter
    redis> incrby counter 100
    redis> get counter
    "201"

**订阅与发布**

Redis 支持订阅与发布 (感觉都成新技术的标配了，比如 GraphQL 也支持，Meteor 也支持)，一个客户端订阅某个 channel，当有其它客户端发布消息到这个 channel 时，这个客户端能马上接收到新消息。

客户端 A

    redis> subscribe redisChat

客户端 B

    redis> publish redisChat "Hello Redis"

**事务**

在一个事务中完成多命令。以 multi 开始一个事务， 然后将多个命令入队到事务中， 最后由 exec 命令触发事务， 一并执行事务中的所有命令。

详略。

**脚本**

略。没太明白，我以为是把批量命令写在脚本文件中，然后用 eval 命令来加这个脚本执行呢。

其余命令略，需要时再查阅。

**管道技术**

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

其余略。

### Note 2

Note for [Redis 入门与应用教程](http://redisbook.rails365.net/467181)

此文补充了 Redis 在 Rails 中的应用。

#### 数据类型与应用场景

对于集合，可以取集合的交集与并集。

> 这种可以用于社交领域，比如两个用户取相同的好友，那就是他们各自好友的集合的交集。在微博应用中，经常要计算你和你的朋友共同关注了哪些人，如果用传统的关系型数据库来做，可能一般的做法是用 joins 语句，假如表很大，这个可能很费性能，但是用 Redis，就可以事先把他们各自关注的人的 id 存到 set 里，再用这两个 set 取交集，就能取到共同关注的人的 id 列表。

> 排序集合可用于排行榜应用，因为排行版的内容肯定是唯一的，且有顺序，可以先按照条件排好顺序，比如文章可按阅读量来排。还能用于自动完成，当你在一个搜索框输入时，会提示有哪些可以选择的，提示的那些数据就可以事先存到 Redis 中，先排好顺序且是唯一的。

#### Ruby 客户端

封装了 Redis API 的 gem。

#### 图形化工具

介绍了几个关于 Redis 的图形化的监控工具和管理工具，比如 redis-stat。

#### Redis 实现 cache 原理

> 先来说整个网站的请求响应的过程：用户从浏览器发出一个请求，比如点击一个按钮或在浏览器输入网址回车，然后经过层层的网络最终到达网站的服务器，假设我们用 nginx 来作为静态文件服务器，而用 unicorn 作为运行 ruby 代码的容器，先经过 nginx 处理，nginx 一般是处理 html，css，js 的，它会默认处理带有 htlm，css，js 后缀的请求，比如 /articles.html，如果发现有它处理不了的，比如 /articles/1 这样的请求地址，因为 nginx 没有匹配这样的地址，就会反向代理到 unicorn，unicorn 是运行 ruby 代码的，它会连接数据库，它可能从数据库取到一些数据，把数据处理完，再返回给 nginx，最 nginx 返回给用户。

将页面中不常变化的部分内容放到 redis 中来。

#### Redis 实现 cache 系统实践

rails 中就自带有 cache 功能，不过它默认是用文件来存储数据的。我们要改为使用 redis 来存储。而且我们也需要把 sessions 也存放到 redis 中。

第一步，开启 cache，cache 在 production 下默认开启，在 development 默认关闭，修改 config/environment/development.rb。

    config.action_controller.perform_caching = true

安装 redis 相关的 gem，修改 Gemfile。

    gem 'redis-namespace'
    gem 'redis-rails'

修改 config/application.rb，修改 cache 方式为 redis。

    config.cache_store = :redis_store, {host: '127.0.0.1', port: 6379, namespace: "rails365"}

优先从缓存中获取 @articles。

    class HomeController < ApplicationController
      def index
        @articles = Rails.cache.fetch "articles" do
          Article.except_body_with_default.order("id DESC").limit(10).to_a
        end
      end
    end

有任何修改 article 的地方，清除缓存，比如新增 article。

    class Admin::ArticlesController < Admin::BaseController
      ...
      def create
        @article = Article.new(article_params)
        Rails.cache.delete "articles"
        ...
      end
    end

对页面片断使用 cache。

    .row
      - cache @articles do
        .col-md-6
          .panel.panel-default
            .panel-heading
              div 最近的文章
            .panel-body
              - @articles.each do |article|
                p.clearfix
                  span.pull-right = datetime article.created_at
                  = link_to article.title, article_path(article)

#### Redis 实现消息队列

使用了 Redis 的 list 数据类型，并结合一个叫 ost 的 gem 实现。

#### Redis 实现自动完成

使用了 Redis 的 zset 数据类型，并结合一个叫 soulmate 的 gem，相应的 soulmate.js 前端库实现。

#### Redis 客户端 medis

略。

其余略。
