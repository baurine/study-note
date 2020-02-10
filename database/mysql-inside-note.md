# 《MySQL 是怎样运行的：从根儿上理解 MySQL》Note

- [MySQL 是怎样运行的：从根儿上理解 MySQL](https://juejin.im/book/5bffcbc9f265da614b11b731/section/5bffcbc9f265da61553a8bc9)

## 1. 序

略。

## 2. 装作自己是个小白 —— 重新认识 MySQL

MySQL 是 c/s 架构。

启动客户端访问服务端：`mysql -h主机名 -P端口号 -u用户名 -p密码`，除了 `-p` 和后面的密码之间没有空间，其它选项可以有空格，密码也可以不在 `-p` 后明文输入，而是回车后隐藏式输入。如果在同一台机器上 `-h主机名` 可以省略。如果端口号是默认值，没有修改过，则 `-P端口号` 可以省略。如果用户名和当前系统的用户名一样，则 `-u用户名` 也可以省略。

客户端和服务端通信的几种方式：

- TCP/IP
- 命名管道和共享内存
- Unix 域套接字文件

服务端处理客户端请求：

- part 1 - 处理连接，每一个分配一个线程/线程池
- part 2 - 解析与优化，生成执行计划，可以用 explain 查看生成结果
  - 查询缓存 (在 MySQL 8.0 中删除)
  - 语法解析 (AST)
  - 查询优化
- part 3 - 存储引擎 (最常用 InnoDB 和 MyISAM)

查看当前存储引擎的命令：`SHOW ENGINES;`

创建表时指定存储引擎 / 修改表的存储引擎，略。

## 3. MySQL 的调控按钮 —— 启动选项和系统变量

MySQL 的参数，配置文件。略。

系统变量：

> MySQL 服务器程序运行过程中会用到许多影响程序行为的变量，它们被称为 MySQL 系统变量，比如允许同时连入的客户端数量用系统变量 `max_connections` 表示，表的默认存储引擎用系统变量 `default_storage_engine` 表示，查询缓存的大小用系统变量 `query_cache_size` 表示，MySQL 服务器程序的系统变量有好几百条，我们就不一一列举了。每个系统变量都有一个默认值，我们可以使用命令行或者配置文件中的选项在启动服务器时改变一些系统变量的值。大多数的系统变量的值也可以在程序运行过程中修改，而无需停止并重新启动它。

查看这些变量：

```
mysql> SHOW [GLOBAL|SESSION] VARIABLES [LIKE pattern];
```

运行时修改系统变量。为了将用户相互间的修改的影响降低最低，将系统变量划分不同的作用范围：

- GLOBAL - 全局变量，影响服务器的整体操作
- SESSION - 会话变量，影响某个客户端连接的操作

修改全局变量：

```
语句一：SET GLOBAL default_storage_engine = MyISAM;
语句二：SET @@GLOBAL.default_storage_engine = MyISAM;
```

修改会话变量：

```
语句一：SET SESSION default_storage_engine = MyISAM;
语句二：SET @@SESSION.default_storage_engine = MyISAM;
语句三：SET default_storage_engine = MyISAM;
```

状态变量：

> 为了让我们更好的了解服务器程序的运行情况，MySQL 服务器程序中维护了好多关于程序运行状态的变量，它们被称为状态变量。比方说 `Threads_connected` 表示当前有多少客户端与服务器建立了连接，`Handler_update` 表示已经更新了多少行记录。

查看状态变量：

```
mysql> SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];
```

## 4. 乱码的前世今生 —— 字符集和比较规则

MySQL 中的 utf8 (实际是 utfbm3) 和 utfbm4

- utf8mb3：阉割过的 utf8 字符集，只使用 1 ～ 3 个字节表示字符。
- utf8mb4：正宗的 utf8 字符集，使用 1 ～ 4 个字节表示字符。

存储 emoji 要用 utfbm4。

查看 MySQL 支持的字符集：

```
SHOW (CHARACTER SET|CHARSET) [LIKE pattern];
```

MySQL 支持 41 种字符集 (TiDB 支持 5 种)

每一种字符集都支持数种比较规则，查看比较规则：

```sql
SHOW COLLATION [LIKE 匹配的模式];
```

比如查看 utf8 的比较规则有哪些：`SHOW COLLATION LIKE 'utf8\_%';`，结果有 27 种。常见的 `utf8_general_ci`, `utf8_bin`, `utf8_spanish_ci`。后缀 `_ci` 表示 case insensitive，如果要区分大小写，用 `_cs` (case sensitive)，`_bin` 表示直接比较它们的二进制大小。

### 字符集和比较规则的应用

MySQL 有 4 个级别的字符集和比较规则，分别是：

- 服务器级别 (相应的系统变量：`character_set_server`, `collation_server`)
- 数据库级别 (系统变量：`character_set_database`, `collation_database`)
- 表级别
- 列级别

单方面修改字符集或比较规则，都会引起另一方的改变。

详略。

#### MySQL 中字符集的转换

涉及的三个系统变量：

- `character_set_client` - 服务器解码请求时使用的字符集
- `character_set_connection` - 服务器处理请求时会把请求字符串从 `character_set_client` 转为 `character_set_connection`
- `character_set_results` - 服务器向客户端返回数据时使用的字符集

字符集转来转去让人头晕，最好的办法是让上面三者的字符符相同。`SET NAMES 字符集名;` 可以把上面三者设置成相同的字符集。

详略。
