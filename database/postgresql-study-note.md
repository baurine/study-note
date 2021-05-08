# PostgreSQL Study Note

PostgreSQL 管理很多数据库，每个数据库中有很多个表，每个表中有很多条记录，每条记录由多列组成。

可以进行的操作：

- create / drop / alter a database
- create / drop / alter a table of a database
- add / drop / alert / rename a column of a table
- insert / select / update / delete records for table

## References

1. [Postgres 101](https://github.com/brettshollenberger/postgres_101)
1. [Getting Started with PostgreSQL on Mac OSX](https://www.codementor.io/devops/tutorial/getting-started-postgresql-server-mac-osx)
1. [Get Started With PostgreSQL Video Tutorial](https://egghead.io/courses/get-started-with-postgresql)
1. [PostgreSQL 在线教程](http://gitbook.net/postgresql/index.html)

在线 Playground：

1. [SQLFiddle](http://sqlfiddle.com)

## Note 1

Note for [Getting Started with PostgreSQL on Mac OSX](https://www.codementor.io/devops/tutorial/getting-started-postgresql-server-mac-osx).

### 安装

1. 服务端和命令行客户端：按照 [Postgres.app](http://postgresapp.com/) 的步骤来就行。
1. GUI 客户端：[GUI Client Apps](http://postgresapp.com/documentation/gui-tools.html)，Postico 比较好。

在上面的步骤中，将 Postgres 的各种命令加入到 $PATH，是这样做的：

    $ sudo mkdir -p /etc/paths.d && echo /Applications/Postgres.app/Contents/Versions/latest/bin | sudo tee /etc/paths.d/postgresapp

执行这句命令后，/etc/paths.d 目录下会有一个 postgresapp 的文本文件，里面的内容只有一行，是 `/Applications/Postgres.app/Contents/Versions/latest/bin`，然后执行 `echo $PATH` 后，这个路径已经在 $PATH 中了，然后就可以在任意地方执行这个路径下的命令，包括 psql, createdb, createuser, `pg_dump` ...

这说明，shell 在启动的时候，会读取 /etc/paths.d 目录下所有文件中的内容，这些内容应该都是路径，然后把这些路径加到 $PATH 环境变量中。

安装 Postgres.app 后，会在 menu bar 生成图标，可以控制 postgresql server 的启动和停止。(命令行怎么启动和停止? 直接用 postgres 命令。)

### 配置

初始化 Postgres.app 后，会自动生成 2 个超级用户和 2 个数据库，一个用户和数据库是 postgres，另一个用户和数据库，和当前登录的系统用户名同名。(对比 MySQL，MySQL 自动生成了 root 用户，以及和当前登录的系统用户名同名的数据库。)

登录 Postgres server，用 Postgres 提供的 psql 命令：

    > psql [database_name]
    database_name=#

`database_name` 可选，如果为空，则默认登录并使用和当前系统用户名同名的数据库。登录后，会在命令提示符左侧显示当前选择的数据库名。

登录并使用 postgres 数据库：

    > psql postgres
    postgres=#

退出登录，使用 `\q` 命令 (和 MySQL 一样)：

    postgres=# \q

列出所有数据库，用 `\l` 或 `\list` 命令 (MySQL 的好理解一些，用 `show databases;` 命令)：

    postgres=# \l
                                             List of databases
              Name           |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
    -------------------------+----------+----------+-------------+-------------+-----------------------
     ...
     postgres                | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     postgres_101            | baurine  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     template0               | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                             |          |          |             |             | postgres=CTc/postgres
     template1               | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                             |          |          |             |             | postgres=CTc/postgres
    (15 rows)

切换数据库，用 `\c` 或 `\connect` 命令 (MySQL 用 `use database_name` 命令)：

    postgres=# \c postgres_101
    postgres_101=#

显示当前数据库中的所有表，用 `\dt` (display tables?) 命令 (MySQL 用 `show tables from database_name` 命令)：

    postgres_101=# \dt
             List of relations
     Schema |  Name  | Type  |  Owner
    --------+--------+-------+---------
     public | movies | table | user_name
     public | orders | table | user_name
     public | users  | table | user_name
    (3 rows)

列出所有用户，使用 `\du` (display users?) 命令 (MySQL 对应的是什么命令?)：

    postgres=# \du
                                      List of roles
    Role name  |                         Attributes                         | Member of
    ------------+------------------------------------------------------------+-----------
    login_name | Superuser, Create role, Create DB                          | {}
    postgres   | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

默认登录的用户和当前系统用户同名，若想以其它用户身份登录，用 `-U user_name` 选项：

    $ psql postgres_101 -U postgres
    postgres_101=#

#### 1. 创建用户

默认生成的 2 个用户都是超级用户，能做任何事情，有点危险，因此我们还需要创建一些普通用户。

两种方法创建新用户：

- 登录 server，用 SQL 的 `CREATE ROLE` 语法
- 不需要登录 server，直接用 PostgreSQL 提供的工具命令 createuser (实际就是把 `CREATE ROLE` 包装了一下)

**`CREATE ROLE`**:

语法：

    CREATE ROLE username WITH LOGIN PASSWORD 'quoted password' [OPTIONS];

示例：

    postgres=# CREATE ROLE patrick WITH LOGIN PASSWORD 'Getting started';
    postgres=# \du

新增的用户默认没有任何权限，用 `ALTER ROLE` 为他修改权限：

    postgres=# ALTER ROLE patrick CREATEDB;
    postgres=# \du

**createuser 工具命令**

PostgreSQL 内置的一些工具命令：

- postgres: execute the SQL server itself
- psql: connect to the SQL server
- createuser: creates a user
- createdb: creates a database
- dropuser: deletes a user
- dropdb: deletes a database
- `pg_dump`: dumps the contents of a single database to a file
- `pg_dumpall`: dumps all database to a file

这些命令直接在控制台运行，不需要先连接到 server，它们不是 SQL 的一部分，只是 PostgreSQL 为了方便我们使用额外提供的一些命令，因此不能在 MySQL 中使用 (MySQL 有自己的工具命令，比如 mysqladmin)。

createuser 的使用：

    > createuser patrick
    > createuser patrick --createdb
    > psql postgres
    postgres=# \du

疑问：如何为用户设置密码? 以及用密码登录? (貌似 PostgreSQL 的策略是登录无须密码，使用系统认证。)

#### 2. 创建数据库

和创建新用户一样，有两种方法：

- SQL 命令：`CREATE DATABASE`
- 工具命令：createdb

**`CREATE DATABASE` with `psql`**

    // login psql database 'postgres' by user 'patrick'
    > psql postgres -U patrick

    // notice, the '#' has changed to '>', it means you're no longer using a Super User account
    postgres=> CREATE DATABASE awesome_psql;

创建数据库后，至少需要授权一个用户可以访问这个数据库，用 `GRANT` SQL 命令。(疑问：上例中，由 patrick 创建了 `awesome_psql` 数据库，那不应该至少 patrick 可以访问这个数据库吗?)

    postgres=> GRANT ALL PRIVILEGES ON DATABASE awesome_psql TO patrick;
    postgres=> \l
    postgres=> \c awesome_psql
    awesome_psql=> \dt
    awesome_psql=> \q

修改数据库名，用 `ALTER DATABASE old_name RENAME TO new_name` 命令：

    postgres=> ALTER DATABASE awesome_psql RENAME TO more_awesome_psql;

总结一下当前学到的 psql 命令：

- `\list` or `\l` : list all database
- `\du` - list all users
- `\cononect` or `\c database_name` - connect to a database
- `\d` or `\dt` or `\d+` : list all tables for current database
- `\d+ table_name` - list a table's schema, index, constraints
- `\q` : quit

一些其它有用的：

- `\h` - see explanation for a sql command, likes `\h select`
- `\?` - see list for psql commands
- `\e` - open an editor to write complex sql query string
- `\conninfo` - list current connection info, likes current databasse, user

**createdb 工具命令**

    > createdb awesome_psql [-U patrick]

### GUI Client Apps

Omit.

---

## Note 2

Note for [Get Started With PostgreSQL Video Tutorial](https://egghead.io/courses/get-started-with-postgresql).

### 1. 创建表

    CREATE TABLE directors (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) UNIQUE NOT NULL
    );

    CREATE TABLE movies (
      id SERIAL PRIMARY KEY,
      title VARCHAR(100) NOT NULL,
      release_date DATE,
      count_stars INTEGER,
      director_id INTEGER
    );

别记了在每条命令后加上 `;`。

删除表：`DROP TABLE table_name`

修改表：`ALTER TABLE table_name`

### 2. 往表中插入新记录

往表中增加一条或多条记录：

    INSERT INTO directors (name) VALUES ('Quentin Tarantino');
    INSERT INTO directors (name) VALUES ('Judd Apatow'), ('Mel Brooks');

    INSERT INTO movies (title, release_date, count_stars, director_id)
    VALUES ( 'Kill Bill', '10-10-2003', 3, 1),
           ( 'Funny People', '07-20-2009', 5, 2);

### 3. 用查询语句过滤数据

`SELECT ... FROM ... WHERE ...`

聚合：`COUNT / SUM / AVG`

    SELECT title,
    release_date AS release
    FROM movies
    WHERE release_date > '01-01-1975';

    SELECT AVG(count_stars)
    FROM movies
    WHERE release_date > '01-01-1975';

    SELECT title,
    count_stars,
    ((count_stars::float / 5) * 100) AS rotten_tomatoes_score
    FROM movies
    WHERE release_date > '01-01-1975';

### 4. 更新记录

`UPDATE ... SET ... WHERE ...`

    UPDATE movies
    SET count_stars=1
    WHERE release_date < '01-01-1975';

### 5. 删除记录

`DELETE FROM ... WHERE ...`

    DELETE FROM movies
    WHERE count_stars > 90;

    DELETE FROM movies;

### 6. 分组和聚合

初始化数据库数据：

    > dropdb postgres_101
    > createdb postgres_101
    > psql -d postgres_101 -f insert.sql
    // the insert.sql includes sql command about creating movies table and copy data from movies.csv
    // https://github.com/brettshollenberger/postgres_101/blob/master/bin/create_tables

分组和聚合：

`GROUP BY ...`

    SELECT ROUND(rating),
    count(*)
    FROM movies
    GROUP BY ROUND(rating)
    ORDER BY 1;
    // the '1' means the result first column, so here means the 'ROUND(rating)'

group 之后进行 select 时需要特别注意，如果 group by 的例是 primary key，那么就可以 `select *`，选择所有列，否则，只能对 group by 的那一列和它的聚合结果进行 select。

    select * from reviews group by id; // work!

    select * from reviews group by rating;  // wrong
    ERROR:  column "reviews.id" must appear in the GROUP BY clause or be used in an aggregate function
    LINE 1: select * from reviews group by rating;
                   ^

`WITH ... AS` (table view?)

    SELECT
    CASE
    WHEN action=true THEN 'action'
    WHEN animation=true THEN 'animation'
    WHEN comedy=true THEN 'comedy'
    WHEN drama=true THEN 'drama'
    WHEN documentary=true THEN 'documentary'
    WHEN romance=true THEN 'romance'
    WHEN short=true THEN 'short'
    ELSE 'other'
    END AS genre,
    title
    FROM movies
    LIMIT 100;

    WITH genres AS (
    SELECT
    CASE
    WHEN action=true THEN 'action'
    WHEN animation=true THEN 'animation'
    WHEN comedy=true THEN 'comedy'
    WHEN drama=true THEN 'drama'
    WHEN documentary=true THEN 'documentary'
    WHEN romance=true THEN 'romance'
    WHEN short=true THEN 'short'
    ELSE 'other'
    END AS genre,
    title
    FROM movies
    )
    SELECT genre,
    COUNT(*)
    FROM genres
    GROUP BY genre;

### 7. 排序

`ORDER BY ... [DESC]`

默认顺序是 ASC，正如前面所说，排序时可以以列名排序，也可以以列的序号排序。

    SELECT *
    FROM friends
    ORDER BY friend_count, name desc;

### 8. 确保唯一性

在创建表时使用 `UNIQUE`

    DROP TABLE directors;
    CREATE TABLE directors (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) UNIQUE NOT NULL
    )

在创建表之后增加唯一性的限制，使用 `ALTER TABLE table_name ADD CONTRAINT contraint UNIQUE(column_name)` 命令。

    ALTER TABLE directors ADD CONSTRAINT directors_name_unique UNIQUE(name);
    ALTER TABLE movies ADD CONSTRAINT movies_title_release_unique UNIQUE(title, release_date);

疑问：

1. 如何查看一个表的 schema?
1. 如何查看一个表的 constraints?
1. 如何删除一个表的 constraints?

前两个疑问的答案是使用 `\d+ table_name` 命令，`\d+` 命令，如果后面没有参数为空，则和 `\dt` 的效果是一样的，显示当前数据库中的所有表名，但加了表名作为参数后，是显示一个表的 schma，constraints，index。

    postgres_101=# \d+ movies

    ...
    Indexes:
        "movies_pkey" PRIMARY KEY, btree (id)
        "title_release_uniq" UNIQUE CONSTRAINT, btree (title, release_date)
        "index_movies_on_title" btree (title)
    Check constraints:
        "count_stars_less_than_6" CHECK (count_stars < 6)
        "movies_count_stars_check" CHECK (count_stars > 0)

删除一个表的 constraint，用 `alert table ... drop constraint` 命令：

    postgres_101=# ALTER TABLE movies DROP CONSTRAINT IF EXISTS "movies_count_stars_check1";

[更多 psql 命令](https://www.postgresql.org/docs/current/static/app-psql.html#APP-PSQL-META-COMMANDS).

### 9. 用外键保证数据完整性

`REFERENCES`

    CREATE TABLE directors (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) UNIQUE NOT NULL
    );

    CREATE TABLE movies (
      id SERIAL PRIMARY KEY,
      title VARCHAR(100) NOT NULL,
      release_date DATE,
      count_stars INTEGER,
      director_id INTEGER REFERENCES directors(id)
    );

使用外键后，插入新记录时，如果关联的外键记录不存在，插入将失败。

    INSERT INTO movies (title, release_date, count_stars, director_id)
                values ('Transformer', '10-10-2011', 3, 4);
    // if current the director_id 4 doesn't exist in directors table,
    // it will be failed to insert into movies table

### 10. 创建多列的外键

`PRIMARY KEY (...)`, `FOREIGN KEY (...) REFERENCES talbe_name(...)`

    CREATE TABLE rentable_movies (
      movie_id INTEGER NOT NULL REFERENCES movies(id),
      store_id INTEGER NOT NULL REFERENCES stores(id),
      copy_number INTEGER NOT NULL,
      PRIMARY KEY (movie_id, store_id, copy_number)
    );

    CREATE TABLE rentings (
      guest_id INTEGER REFERENCES guests(id),
      movie_id INTEGER NOT NULL,
      store_id INTEGER NOT NULL,
      copy_number INTEGER NOT NULL,
      due_back DATE,
      returned BOOLEAN,
      FOREIGN KEY (movie_id, store_id, copy_number) REFERENCES rentable_movies(movie_id, store_id, copy_number)
    );

### 11. 使用 CHECK Constraints

创建表时：

    CREATE TABLE movies (
      id SERIAL PRIMARY KEY,
      title VARCHAR(100) NOT NULL,
      release_date DATE,
      count_stars INTEGER,
      CHECK (count_stars > 0),
      CHECK (count_stars < 6)
    );

创建表之后增加：

    ALTER TABLE movies ADD CONSTRAINT count_stars_greater_than_0 CHECK (count_star > 0);
    ALTER TABLE movies ADD CONSTRAINT count_stars_less_than_6 CHECK (count_star < 6);

### 12. 用索引加速查询

用 EXPLAIN 命令查看一条 SQL 命令是如何解析执行的。

在没有索引时：

    EXPLAIN SELECT COUNT(*) FROM movies WHERE title='America';
    ->  Seq Scan on movies

在增加索引之后：

    CREATE INDEX CONCURRENTLY index_movies_on_title ON movies (title);
    CREATE INDEX CONCURRENTLY index_movies_on_year ON movies (year);
    CREATE INDEX CONCURRENTLY index_movies_on_title_year ON movies (year, title);

    EXPLAIN SELECT COUNT(*) FROM movies WHERE title='America';
    ->  Index Only Scan using index_movies_on_title on movies

### 13. 用 Inner Join 进行表的联合

`INNER JOIN table_name ON condition`

    SELECT movies.title,
    stores.location,
    rentings.copy_number,
    guests.name AS renter_name,
    rentings.returned,
    rentings.due_back
    FROM rentings
    INNER JOIN rentable_movies
    ON (
    rentable_movies.movie_id=rentings.movie_id AND
    rentable_movies.store_id=rentings.store_id AND
    rentable_movies.copy_number=rentings.copy_number
    )
    INNER JOIN movies
    ON movies.id=rentable_movies.movie_id // Notice! later join table can join with former joined table
    INNER JOIN stores
    ON stores.id=rentable_movies.store_id
    INNER JOIN guests
    ON guests.id=rentings.guest_id;

除了 inner join，还有 left join，它们的区别，举个例子，两个表，用户表 users 和评论表 reviews，有些用户可以多个评论，但也有一些用户没有评论。假设我现在想知道每个用户有多少个评论。此时，如果你用 inner join，那些没有评论的用户，就不会出现在结果。这也许是你想要的，但也许不是，你想要的是，即使没有评论的用户，也要出现在结果中，比如我想用评论数为所有用户排序，不能说因为它的评论数为 0，就不在这个用户排序结果中了。那么，这时候，我们就该用 left join 了。下面是示例代码：

    select users.*, count(reviews.id) from users inner join reviews on reviews.user_id = users.id group by users.id order by count(reviews.id) desc;
    // inner join，只会得到评论数不为 0 的用户
    // 此时，因为是 group by users.id，所以 select 时，只能 select users.* 或聚合结果，并不能 select *，因为 * 里包括了 reviews.*
    // select 中的 count(reviews.id)，结果和 count(*) 是一样的。但如果用 left join，结果是不一样

    select users.*, count(reviews.id) from users left join reviews on reviews.user_id = users.id group by users.id order by count(reviews.id) desc;
    // left join，也可以得到评论数为 0 的用户
    // select 中的 count(reviews.id)，为什么要用 count(reviews.id)，或者 count(reviews.*) 也行，但就是不能用 count(*)，因为对于评论数为 0 的用户，count(reviews.id) 可以得到正确的结果 0，而 count(*) 则会得到错误的结果 1

Join 的分类：

- Inner Join
- Outer Join
  - Left [Outer] Join
  - Right [Outer] Join
  - Full [Outer] Join

一些形象的解释：

1. [What is the difference between "INNER JOIN" and "OUTER JOIN"?](https://stackoverflow.com/questions/38549/what-is-the-difference-between-inner-join-and-outer-join)
1. [sql 之 left join、right join、inner join 的区别](http://www.cnblogs.com/pcjim/articles/799302.html)
   - left join (左联接) - 返回包括左表中的所有记录和右表中联结字段相等的记录
   - right join (右联接) - 返回包括右表中的所有记录和左表中联结字段相等的记录
   - inner join (等值连接) - 只返回两个表中联结字段相等的行

### 14. 查询唯一结果

    SELECT distinct title
    FROM movies;

---

## Note 3

一些其它的语句：

    # Rename a table
    ALTER TABLE users RENAME TO backup_tbl;

    # Add a new column
    ALTER TABLE users ADD email VARCHAR(40);

    # Alter a column
    ALTER TABLE users ALTER COLUMN signup_date SET NOT NULL;

    # Rename a column
    ALTER TABLE users RENAME COLUMN signup_date TO signup;

    # Drop a column
    ALTER TABLE users DROP COLUMN email;

命令中的动词：

- For database: CREATE, DROP, ALTER
- For table: CREATE, DROP, ALTER
- For columns in table: ADD, DROP, ALTER, RENAME
- For table rows: INSERT, SELECT, UPDATE, DELETE

### Dump and restore

    > pg_dump dbname > dumpfile

    > psql dbname < dumpfile

### 全文搜索 - Full Text Search

[*PostgreSQL Full Text Search*](https://www.postgresql.org/docs/current/static/textsearch.html)

    SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery;

    SELECT to_tsvector('fat cats ate fat rats') @@ to_tsquery('fat & rat');

`to_tsvector()` 用来将目标字符串生成 vector，生成的 vector 中有各个 word 出现的位置，出现的次数等信息，而 `to_tsquery` 用来将搜索关键字生成真正用于搜索的 word 词组，它会去掉 '&' 'and' 'the' 'a' 等太常见的字符。

全文搜索如果不建索引，会比 'ILIKE' 的模糊查找还慢，可以用 GIN 建索引。

相关资料：

- [Fast Full-Text Search in PostgreSQL](https://austingwalters.com/fast-full-text-search-in-postgresql/)

### Having and where

[*Postgres contains in array function*](<https://coderwall.com/p/lkgvdq/postgres-contains-in-array-function>)

    select users.email, array_agg(tags.name)
    from users
      join user_tags on users.email = user_tags.email
      join tags on tags.id = user_tags.tag_id
    group by users.email
    having '{1, 2}'::int[] <@ array_agg(tags.id);

**having 和 where 的区别：**

- where 执行在 group 之前，分组前过滤；
- having 执行在 group 之后，先得到分组结果，再从分组中过滤结果；
- having 必须用和 group 配套使用，且在 group 之后，而 where 无此限制
- where 条件中不能使用聚合函数，而 having 可以。这也是 having 使用的最大场景。因为如果你在 having 中不使用聚合结果，那完全可以直接用 where 替代嘛。比如上例中，我们在 having 中使用了 array_agg 聚合函数，而在 where 中就无法使用它。

导致这些区别的根本原因在于，where 配合 group 用来产生聚合结果，它是聚合结果的因，所以当然不能在它的语句中使用聚合结果，而 having 是在有了聚合结果后进一步过滤，所以它可以使用聚合结果。

只有 where 不能使用聚合函数，其它的除了 having，select 和 order 也可以使用聚合函数，因为只有 where 是用来产生产生数据的，而 select / order / having 是用来操作结果的。

### Extension

想使用 psql 的 [intarray](https://www.postgresql.org/docs/9.5/static/intarray.html) 功能来比较 2 个事物的相关性。(比如有 2 个用户，都被多个 tag 所标记，通过计算这 2 个 tag 数组的交集以及交集的多少来得到他们的相关性。)

这个功能属于 psql 的扩展 (extension)，使用之前要先下载安装，所幸的是这个扩展是 psql 自带的，已经在安装包中了，只是默认处于未使用状态，我们用 `create extension` 命令来使用它，使用 `\dx` 命令查看已使用的扩展。

    postgres=# create extension intarray;
    postgres=# \dx

    postgres=# select '{1,2,3}'::int[] & '{2,3,4}'::int[];
    ?column?
    --------
    {2,3}

    postgres=# select icount('{1,2,3}'::int[] & '{2,3,4}'::int[]) as count;
    count
    -----
    2

修改 extension 用 `alter extension` 命令，删除用 `drop extension`，官方文档如下：

1. [CREATE EXTENSION](https://www.postgresql.org/docs/9.1/static/sql-createextension.html)
1. [Additional Supplied Modules](https://www.postgresql.org/docs/9.1/static/contrib.html)

### Show database table size

用 `pg_total_relation_size(table_name)` 获取所占磁盘空间大小，`pg_size_pretty(size)` 用来转换 size。

    # \c test_db;
    # select pg_size_pretty(pg_total_relation_size("movies"));
    ----------------
    1285 MB
    (1 row)

还有一些其它方法：

    SELECT pg_size_pretty(pg_database_size('db_employee'));
    SELECT pg_size_pretty(pg_relation_size('Employee_table'));  // table size without indexes
    SELECT pg_size_pretty(pg_indexes_size('index_empid'));

参考：

- [How to find size of Database and Table in PostgreSQL](https://www.dbrnd.com/2015/05/how-to-find-size-of-database-and-table-in-postgresql/)
- [PostgreSQL: How to show table sizes](https://makandracards.com/makandra/52141-postgresql-how-to-show-table-sizes)
