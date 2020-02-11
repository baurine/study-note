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

## 5. 从一条记录说起 —— InnoDB 记录结构

我们前边唠叨请求处理过程的时候提到过，MySQL 服务器上负责对表中数据的读取和写入工作的部分是存储引擎，而服务器又支持不同类型的存储引擎。这里我们只重点探讨 InnoDB 存储引擎。

### InnoDB 页

> 而我们知道读写磁盘的速度非常慢，和内存读写差了几个数量级，所以当我们想从表中获取某些记录时，InnoDB 存储引擎需要一条一条的把记录从磁盘上读出来么？不，那样会慢死，InnoDB 采取的方式是：将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB 中页的大小一般为 16 KB。也就是在一般情况下，一次最少从磁盘中读取 16KB 的内容到内存中，一次最少把内存中的 16KB 内容刷新到磁盘中。

(完全理解)

### InnoDB 行格式

(相当于是在定义协议，和定义 TCP/IP 协义类似)

MySQL 是按行来存储记录的 (也有按列存的存储引擎，比如 ClickHouse，这种一般是用来做分析用的)。

MySQL 设计了 4 种行格式：Compact、Redundant、Dynamic 和 Compressed。

指定使用何种行格式：

```
CREATE TABLE 表名 (列的信息) ROW_FORMAT=行格式名称;

ALTER TABLE 表名 ROW_FORMAT=行格式名称;
```

#### Compact 行格式

```
|    记录的额外信息                        ||    记录的真实数据           ｜
| 变长字段长度列表 | NULL 值列表 | 记录头信息 || 列 1 的值 | 列 2 的值 | ... |
```

一行中第一部分为记录的额外信息，分三类：变长字段长度列表，NULL 值列表，记录头信息。第二部分存储所有的非 NULL 值。(因为这是 compact 类型的行，尽可能减少存储空间)

NULL 值列表和记录头信息都用位图表示。记录头信息由固定的 5 个字节组成。5 个字节也就是 40 个二进制位，不同的位代表不同的意思。

细节先暂时跳过。

记录的真实数据：除了自己定义的列的数据外，MySQL 还会为每个记录增加一些默认的列，也称为隐藏列。

- `DB_ROW_ID`，非必须，当这个表没有定义主键或 unique 列时，MySQL 会添加此列作为行 ID，唯一标识一条记录
- `DB_TRX_ID`，必须，事物 ID
- `DB_ROLL_PTR`，必须，回滚指针

`CHAR(M)` 列的存储格式：对于 `CHAR(M)` 类型的列来说，当列采用的是定长字符集时，该列占用的字节数不会被加到变长字段长度列表，而如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表。

#### Redundant 行格式

比较老的一种格式了，先跳过。

#### 行溢出

MySQL 按页存储，一页 16KB，一页至少存 2 行。如果一行的某列存储的长度过大，比如 varchar 最大支持 65535 byte，即 64KB，则这一行就溢出了。那 MySQL 如何处理呢，如果出现这种情况，则这一列只存储全部内容的一小部分，剩余内容存储到其它页中，并指页的地址存储在该列中。(就跟文件系统一样的道理)。

具体的细节略过。

#### Dynamic 和 Compressed 行格式

Dynamic 行格式是目前 MySQL 5.7 开始的默认行格式。这两种格式和 Compact 格式相似，只不过在处理行溢出时不同，在 Dynamic 行格式中发生行溢出时，所有内容都会存储到其它页中，只将指针存储在原始列中，原始列中不会存储一小部分真实的内容。

Compressed 行格式会采用压缩算法对页面进行压缩，以节省空间。
