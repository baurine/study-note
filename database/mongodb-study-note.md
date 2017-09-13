# MongoDB Study Note

## 安装和配置

[Offical Document](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)

1. install

       $ brew install mongodb

2. create data directory (默认在 `/data/db`)

       $ mkdir -p /data/db

3. 运行服务端 mongod

       $ mongod [--dbpath <path to data directory>]

   我在 mac 上创建的 db 目录是在 `~/MonogoData/db` (可以写在配置文件里吗? -- 可以的)，所以启动命令是:

       $ mongod --dbpath ~/MongoData/db

4. 运行客户端 mongo shell

       $ mongo

## Note 1

Note for 《MongoDB 权威指南》 2011 版。

### 第一章

面向文档，易于扩展。

### 第二章 - 入门

- 文档是 MongoDB 的基本单元，相当于传统关系型数据库的行，但比它复杂；
- 集合是文档的集合，可以看作是没有模式的表；
- 一个数据库里有多个集合；
- 一个 MongoDB 实例可以容纳多个独立的数据库；
- 每个文档都有一个特殊的键 "_id"，它在文档所在的集合中是唯一的。

子集合：db.blog，db.blog.posts

#### MongoDB shell 的基本操作 - CRUD

**创建**

    > post = {
      "title": "My Blog Post",
      "content": "blog post content",
      "date": new Date()
    }
    > db.blog.insert(post)
    > db.blog.find()

**读取**

    > db.blog.find()
    > db.blog.findOne()

**更新**

使用 `update()` 方法，至少接收两个参数，第一个是匹配文档的条件，第二个是新的文档。

    collection.update( condition, new_doc )

    > post.comments = []
    > db.blog.update({"title": "My Blog Post"}, post)
    > db.blog.find()

**删除**

    > db.blog.remove({})  // 删除集合中的所有文档，注意用 db.blog.remove() 是不起作用的
    > db.blog.remove({"title":"My Blog Post"}) // 删除符合条件的文档

#### Shell 使用技巧

使用帮助：

    > help

其它命令：

    > show dbs  // 显示数据库
    > use <db_name>  // 选择数据库
    > db  // 当前数据库
    > show collections  // 显示当前数据库的所有集合

显示一个方法的实现 (即源代码)：

    > db.blog.update  // 注意，不要加 `()`，加上了 `()` 就是让它执行了，而不是显示它的实现

#### 数据类型

MongoDB 在 JSON 的基础上，增加了对 ObjectId，Date，正则表达式 等类型的支持。

### 第三章 - 创建、更新及删除文档

#### 插入

使用 `insert` 方法：

    > db.foo.insert({"bar":"barz"})

如果要插入多个文档，使用批量插入会快一些，批量插入会传递一个文档的数组给数据库。

#### 删除

删除集合中的所有文档，但不会删除集合本身，原来的索引也会保留。

    > db.users.remove({})

只删除符合条件的文档。

    > db.users.remove({"name":"admin"})

删除整个集合，包括索引：

    > db.drop_collection("users")

#### 更新文档

使用 `update()` 方法：

- 进行单个文档的整体替换。

      > db.blog.posts.update({"title":"My Blog Post"}, new_post)

- 使用更新修改器只修改文档的部分内容。

      > db.blog.posts.update({"_id": ObjectId("xxx")}, {"$set": {"title": "BlogTitle"}})

修改器：

- "$set"
- "$unset"
- "$inc"
- "$dec"

数组修改器：

- "$push"
- "$addToSet"
- "$pop"
- "$pull"

数组定位操作符：

- "$"

      > db.blog.update({"comments.author":"John"}, {"$set": {"comments.$.author": "Jim"}})

upsert: 一种特殊的更新，如果文档存在，则正常更新，如果文档不存在，就会以查询条件和更新条件为基础新建一个文档。

    > db.math.remove({})
    > db.math.update({"count": 25}, {"$inc": 3}, true)  // 第三个参数说明是 upsert 更新
    > db.math.findOne()
    {
        "_id": ObjectId("xxx"),
        "count": 28
    }

save Shell 帮助程序：快速修改保存一个文档，内部实际使用 upsert，文档存在则更新，否则插入新文档。

    > var x = db.foo.findOne()
    > x.num = 42
    > db.foo.save(x)  // == db.foo.update({"_id": x_id}, x)

更新多个文档：默认 `update()` 只更新匹配的第一个文档，将它的第四个参数置为 true 则更新匹配的所有文档。

### 第四章 - 查询

- 使用 find 或 findOne
- 使用 "$" 条件查询实现范围、集合包含、不等式和其它查询
- 使用 "$where" 实现复杂查询
- 查询将会返回一个数据库游标，只有真正需要时才会惰性地批量返回文档
- 针对游标的元操作，包括排序，限定返回数量

#### find

第一个参数为查询条件：

    > find()
    > find({"name": "joe", "age": 27})

第二个参数指定返回的键和不希望返回的键：

    > find({}, {"username": 1, "email": 1})
    > find({}, {"fatal_weakness": 0, "_id", 0})

#### 查询条件

- "$lt" (<)
- "$lte" (<=)
- "$gt" (>)
- "$gte" (>=)
- "$ne" (!=)

例子：

    > db.users.find({"age": {"$gte": 18, "$lte": 30}})
    > db.users.find({"username": {"$ne": "joe"}})

- "$in"
- "$nin"
- "$or"

例子：

    > db.raffle.find({"$or": [
        {"ticket_no": {"$in": [100, 200, 300]}},
        {"winner": true}
      ]})

- "$not"

例子：

    > db.users.find({"id_num": {"$not": {"$mod": [5, 1]}}})

条件句是内层文档的键，修改器是外层文档的键。

#### 特定类型的查询

null

正则表达式

    > db.user.find({"name": /joey?/i})

查询数组

- "$all"

      > db.food.find({"fruit": {"$all": ["apple", "banana"]}})

- key.index 查询数组指定位置元素

      > db.food.find({"fruit.2": "peach"})

- "$size"

      > db.food.find({"fruit": {"$size": 3}})

- "$slice" 返回数组的部分结果

      > db.blog.posts.findOne({}, {"comments": {"$slice": 10}})  // 返回前 10 条 comments

查询内嵌文档

- 内嵌查询法

      > db.people.find({"name": {"first": "joe", "last": "Schmoe"}})

  这种方法结果必须和条件完全匹配，顺序也必须一样，即必须指定所有键，且键的顺序也一致

- 点表示法

      > db.people.find({"name.first": "Joe", "name.last": "Schmoe"})

- "$elemMatch"

      > db.blog.find({"comments": {
          "$elemMatch": {"author": "joe",
                         "score": {"$gte": 5}}
      }})

#### "$where" 查询

当上面的键值对的查询方法都不管用时，可以使用 "$where" 查询，强大但性能低，非到必要是不要使用。

    > db.foo.find("$where": function() {
        return this.x == this.y
    })

当 "$where" 的函数返回值为 true 时，该文档才被返回。和 array 的 filter 方法功能类似。也让我想起了 GraphQL 服务端查询数据的实现。

#### 游标

    var cursor = db.blog.find()
    while (cursor.hasNext()) {
        var obj = cursor.next();
        // do stuff
    }

游标类还实现了迭代器接口，可以使用 forEach():

    cursor.forEach(c => print(c.name))

当调用 find() 时，shell 并不立即开始查询数据库，而是等待真正需要结果时才发送查询。(懒加载，或者说延迟加载)

    var cursor = db.foo.find().sort({"x": 1}).limit(1).skip(10);
    var cursor = db.foo.find().limit(1).sort({"x": 1}).skip(10);
    var cursor = db.foo.find().skip(10).limit(1).sort({"x": 1});

以上三种表述等价，此时查询都还未执行，所有这些函数都只是在构造查询，只有当执行 `hasNext()` 或 `next()` 时，查询才真正执行。

limit, skip, sort

    db.stock.find({"description": "mp3"}).limit(50).skip(50).sort({"price": -1})

避免使用 skip 略过大量数据，性能会很低，所有数据库都存在这个问题，一种方法是利用上次查询的结果作为条件：

    var page2 = db.foo.find({"date": {"$gt": last.date}})
    page2.sort({"date": -1}).limit(100)

高级查询选项

查询分为普通的和包装的两类，上面讲到的都是普通的，实际上，普通的查询在内部会转换成包装的查询：

    > var cursor = db.foo.find({"foo": "bar"}).sort({"x": 1})

在内部包装成：

    > var cursor = db.foo.find({"$query": {"foo": "bar"}, "$orderBy": {"x": 1}})

### 第五章 - 索引

创建正确的索引很重要，不正确的索引可能反而降低查询效率。创建索引会降低插入速度。

创建索引时思考以下问题：

1. 会做什么样的查询? 其中哪些键需要索引?
1. 每个键的索引方向是怎么样的?
1. 如何应对扩展? 有没有不同的键的排列可以使常用的数据更多地保留在内存中?

使用 `ensureIndex({}, {})` 创建索引，第一个参数为索引字段，1 表示升序，-1 表示降序，参数二为选项：

    > db.foo.ensureIndex({"a": 1, "b": -1}, {"name": "xxx", "unique": true})

消除重复：创建唯一索引前如果已存在重复文档，创建失败，可以加上 dropDups 选项自动删除重复文档，在确信没有问题的情况下可以这么做，否则还是自己写个脚本处理比较好。

    > db.foo.ensureIndex({"a": 1, "b": -1}, {"unique": true, "dropDups": true})

**explain 和  hint**

使用 `explain()` 可以分析查询内部的具体方式从而优化查询，如果查询内部没有使用预期的索引，可以用 hint 强制使用指定的索引 (最好还是通过优化查询来实现)。

    > db.foo.find({"age": {"$gt": 10}, "name": "sally"}).explain()

    > db.foo.find({"age": {"$gt": 10}, "name": "sally"}).hint({"name": 1, "age": 1})

**索引管理**

索引的元信息存储在每个数据库的 system.indexes 集合中，不能对其直接插入或删除，只能通过 ensureIndex 和 dropIndex 操作。

system.namespaces 存储所有索引的名字。

创建索引时加上 `{"background": true}` 选项可以让索引在后台创建，以免阻塞正常请求。

    > db.foo.ensureIndex({"a": 1}, {"background": true})

删除索引

    > db.runCommand({"dropIndexes": "foo", "index": "name_age"})

**地理空间索引**

随着 LBS 的发展，地理位置的查询变得越来越流行，找到离当前位置最近的几个场所。MongoDB 为坐标平面查询提供了专门的索引，称作地理空间索引。(PostgreSQL 有吗?)

使用 `2d` 来声明索引类型，查询时使用 `$near` 作为条件。

    > db.map.ensureIndex({"gps": "2d"})

这里的键 `gps` 必须是某种形式的一对值，比如：

    {"gps": [100, 200]}
    {"gps": {"x": 100, "y": 200}}
    {"gps": {"latitude": 100, "longitude": 200}}

默认情况下，地理空间索引值范围是 -180~180，可以通过选项重新指定：

    > db.map.ensureIndex({"gps": "2d"}, {"min": -1000, "max": 1000})

查询：普通查询和数据库命令查询。

    > db.map.find({"gps": {"$near": [20, 30]}}).limit(10)

    > db.runCommand({"geoNear": "map", "near": [40, 20], "num": 10})

还可以用 `$within` 查询指定形状内的文档，可以是矩形或圆。

    > db.map.find({"gps": {"$within": {"$box": [10, 20], [30, 40]}}})
    > db.map.find({"gps": {"$within": {"$center": [10, 20], 18}}})

复合地理空间索引：往往要找的并不只是一个地点，而是一种类型的地点，比如周围的咖啡店，将地理空间索引和普通索引组合起来就可以满足这种需求。

    > db.map.ensureIndex({"location": "2d", "description": 1})

    > db.map.find({"location": {"$near": [10, 20]}, "description": "coffeeshop"}).limit(1)
    {
        "_id": ObjectId("xx"),
        "name": "Mud",
        "location": [x, y],
        "description": ["coffeeshop", "espresso"]
    }

### 第六章 - 聚合

- count
- distinct
- group
- MapReduce

**count**

    > db.foo.count()

**distinct**

    > db.runCommand({"distinct": "people", "key": "age"})
    {"value": [20, 35, 60], "ok": 1}

**group**

    db.runCommand({"group": {
        "ns": "stocks",
        "key": "day",
        "initial": {"time": 0},
        "$reduce": function(doc, prev) {
            if (doc.time > prev.tim) {
                prev.price = doc.price
                prev.time = doc.time
            }
        },
        "condition": {"day": {"$gt": "2010/09/30"}}
    }})

使用完成器：定义 "finalizer" 函数，对 ”$reduce" 返回结果做进一步处理。

使用函数作为分组的 key：

    db.posts.group({
        "ns": "posts",
        "$keyf": (x)=>x.category.toLowerCase(),
        ...
    })

(这种方法相当于 PostgreSQL 中的 `select LOWER(category) as low_category ... group by low_category`)

**MapReduce**

大致明白其原理。MapReduce 是很慢的，不用于实时处理，一般作为后台任务处理。

### 第七章 - 进阶管理

#### 数据库命令

前面的章节已涵盖了大部分命令，很多命令都可以用一个统一的方法 `db.runCommand()` 来调用。比如：

    > db.foo.drop()
    > db.runCommand({"dorp": "test"})

而 `db.runCommand()` 实际上又是作为一种特殊类型的查询来实现的。(呵呵，又绕回去了)：

    > db.$cmd.findOne({"drop": "test"})

查看上一次命令的影响 `getLastError`：

    > db.runCommand({"getLastError": 1})

查看所有命令：

    > db.listCommands()

#### 固定集合

暂时用不上，一般用于存储日志，略。

#### GridFS：存储文件

GridFS 是一种在 MongoDB 中存储大二进制文件的机制。

使用 mongofiles 程序，往 GridFS 中上传、下载、查看、删除文件。

    $ echo "hello world" > foo.txt
    $ mongofiles put foo.txt
    $ mongofiles list
    $ rm foo.txt
    $ mongofiles get foo.txt
    $ cat foo.txt

    $ mongofiles --help

    > db.fs.files.distince("filename")

#### 服务器脚本

在服务器端可以通过 `db.eval` 函数来执行 js 脚本，可以把 js 代码存储在数据库中。

#### 数据库引用

DBRef，看下来，其实和关系型数据库中的 reference 类似。先略过。

### 第八章 - 管理

#### 启动和停止服务端

直接从命令行启动：

    > mongod --help
    > mongod --port 5586 --fork --logpath mongodb.log

从配置文件读取选项参数启动：

    > mongod --config ~/.mongodb.conf
    > cat ~/.mongodb.conf
    port = 5586
    fork = true
    logpath = mongodb.log
    dbpath = ~/MongoData/db

停止，稳妥的方式是在 mongo shell 中运行 shutdownServer 命令：

    > use admin
    > db.shutdownServer()

#### 监控

- 使用 web 管理接口
- serverStatus 命令
- mongostat 程序
- 第三方插件

#### 安全和认证

MongoDB 在数据库层面有安全和认证功能，使用 `mongod --auth` 启动服务器，在 shell 中使用 `db.addUser()` 和 `db.foo.auth()` 来管理用户和认证用户。

添加的用户放在存储在 `db.system.users` 集合中，如果要删除某个用户，则执行 `db.system.users.remove({"user": "test_user"})`。

但一般来说，数据库是布署在内网，不应该被外界所访问，不推荐使用数据库层面的认证功能，而是应该在应用层进行安全和认证。

#### 备份和恢复

- 直接复制数据库文件，可能记录不完整
- mongodump / mongorestore
- fsync 和 锁
- 修复：`mongod --repair`

### 第九章 - 复制

一主多从，主从复制。

暂略。属于运维范畴。简单了解。

### 第十章 - 分片

分片是指将数据拆分，将其分散在不同的机器上的过程。分片是 MongoDB 的扩展方式，通过分片能够增加更多的机器来应对不断增加的负载和数据，还不影响应用。

暂略，简单了解。

### 第十一章 - 应用举例

- Java
- PHP
- Ruby
- Python

从上面的例子来看，Java 真的是好啰嗦，MongoDB 比较适合动态语言使用。特别是传递复杂的查询条件的时候，在 Java 里要用多个 Map 构造。私以为，MongoDB 应该像 SQL 一样，在任意客户端，查询语句都应该直接用字符串构造，然后统一由 MongoDB 自己解析。(GraphQL 也是和 SQL 一样的的做法)

### 附录

#### 深入 MongoDB 内部

BSON: 类似 ProtoBuffer??

Mongo 传输协议：自定义协议，在 TCP/IP 协议层之上封装的应用层协议，用来和 MongoDB 交互。(在 MeShare 的时候我们也干过同样的事情，封装了流媒体的私有协议，在 H264 的流媒体数据中加入了私有控制指令)
