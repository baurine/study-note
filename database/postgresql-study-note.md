# PostgreSQL Study Note

PostgreSQL manages many databases, and one database has many tables, one table has many columns.

So you can:

- create / drop / alter a database
- create / drop / alter a table of a database
- add / delete / update / select data for tables

## References

1. [Postgres 101](https://github.com/brettshollenberger/postgres_101)
1. [Getting Started with PostgreSQL on Mac OSX](https://www.codementor.io/devops/tutorial/getting-started-postgresql-server-mac-osx)
1. [Get Started With PostgreSQL Video Tutorial](https://egghead.io/courses/get-started-with-postgresql)
1. [PostgreSQL 在线教程](http://gitbook.net/postgresql/index.html)

Online playground:

1. [SQLFiddle](http://sqlfiddle.com/#!15)

## Note 1

Note for [Getting Started with PostgreSQL on Mac OSX](https://www.codementor.io/devops/tutorial/getting-started-postgresql-server-mac-osx).

### Installation

1. Install it by graphical installer: [Postgres.app](http://postgresapp.com/)

1. Use it by [GUI Client Apps](http://postgresapp.com/documentation/gui-tools.html)

### Configuring PostgreSQL

    > psql [postgres]   // login psql database 'postgres', if it is empty, then login default database, it is same with your sytem logined username
    postgres=# \du      // list users in postgresql

                                      List of roles
    Role name |                         Attributes                         | Member of
    -----------+------------------------------------------------------------+-----------
    baurine   | Superuser, Create role, Create DB                          | {}
    postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

    postgres=# \l       // list databases in postgresql

After install PostgreSQL, it will automatically created 2 users and 2 databases, one is 'postgres', another one is same with your system logined user name, in my case, it is 'baurine'.

#### 1. Creating Users

Different users should be granted different permissions. super user can do anything, including delete database, it is dangerous.

There are two main ways to do this:

- Directly execute the `CREATE ROLE` SQL query on the database
- Use the `createuser` utility that comes installed with Postgres (which is just a wrapper for executing `CREATE ROLE`).

**`CREATE ROLE`**:

    CREATE ROLE username WITH LOGIN PASSWORD 'quoted password' [OPTIONS];
    
    postgres=# CREATE ROLE patrick WITH LOGIN PASSWORD 'Getting started';
    postgres=# \du  // to see changes

The new created user has nothing permissions, so we should add some permissions for it by `ALTER ROLE` command.

    postgres=# ALTER ROLE patrick CREATEDB;
    postgres=# \du  // to see changes

**The `createuser` utility**

- createuser: creates a user
- createdb: creates a database
- dropuser: deletes a user
- dropdb: deletes a database
- postgres: execute the SQL server itself
- pg_dump: dumps the contents of a single database to a file
- pg_dumpall: dumps all database to a file
- psql: connect to the SQL server

Notice! these commands run outside psql, not inside psql; and it supplied by PostgreSQL, it is not a part of SQL itself, so it means if you use MySQL, you can't use these utilities.

    > createuser patrick
    > createuser patrick --createdb
    > psql postgres
    postgres=# \du

#### 2. Creating a Database

Just like creating a user, there are two ways to create a database:

- Executing SQL commands directly with psql
- The `createdb` command line utility

**`CREATE DATABASE` with `psql`**

    > psql postgres -U patrick  // login psql database 'postgres' by user 'patrick'
    postgres=> CREATE DATABASE awesome_psql;  // notice, the '#' has changed to '>', it means you're no longer using a Super User account

After create a database, you need to add at least one user who has permission to access the database, we use `GRANT` command.
( I have a question here, the database `awesome_psql` is created by user 'patrick', should not 'patrick' has the default permission to access `awesome_psql`)

    postgres=> GRANT ALL PRIVILEGES ON DATABASE awesome_psql TO patrick;
    postgres=> \list  // or \l
    postgres=> \connect awesome_psql  // or \c awesome_psql
    postgres=> \dt
    postgres=> \q

Modify database by `ALTER DATABASE [old_name] RENAME TO [new_name]`.

    postgres=> ALTER DATABASE awesome_psql RENAME TO more_awesome_psql;

Until now, we have learnt following psql commands:

- \du : list all users
- \list or \l : list all database
- \cononect or \c [database_name]: connect to a database
- \d or \dt or \d+ : list all tables for current database
- \d+ table_name: list a table's schema, index, constraints
- \q : quit

Some others:

- \h: see explanation for a sql command, likes `\h select`
- \?: see list for psql commands
- \e: open an editor to write complex sql query string
- \conninfo: list current connection info, likes current databasse, user

**The `createdb` Utility

    > createdb awesome_psql [-U patrick]

### GUI Client Apps

Omit.

----

## Note 2

Note for [Get Started With PostgreSQL Video Tutorial](https://egghead.io/courses/get-started-with-postgresql).

### 1. Create a Postgres Table

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

**Don't forget to** add the `;` in the end of the query string.

Delete a table: `DROP TABLE table_name`

Modify a table: `ALTER TABLE table_name`

### 2. Insert Data into Postgres Tables

Add one or multiple rows records at one query string.

    INSERT INTO directors (name) VALUES ('Quentin Tarantino');
    INSERT INTO directors (name) VALUES ('Judd Apatow'), ('Mel Brooks');

    INSERT INTO movies (title, release_date, count_stars, director_id)
    VALUES ( 'Kill Bill', '10-10-2003', 3, 1),
           ( 'Funny People', '07-20-2009', 5, 2);

### 3. Filter Data in a Postgres Table with Query Statements

`SELECT ... FROM ... WHERE ...`

`COUNT / SUM / AVG` for aggerating

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

### 4. Update Data in Postgres

`UPDATE ... SET ... WHERE ...`

    UPDATE movies
    SET count_stars=1
    WHERE release_date < '01-01-1975';

### 5. Delete Postgres Records

`DELETE FROM ... WHERE ...`

    DELETE FROM movies
    WHERE count_stars > 90;

    DELETE FROM movies;

### 6. Group and Aggregate Data in Postgres

**Init movies table**

    > dropdb postgres_101
    > createdb postgres_101
    > psql -d postgres_101 -f insert.sql
    // the insert.sql includes sql command about creating movies table and copy data from movies.csv
    // https://github.com/brettshollenberger/postgres_101/blob/master/bin/create_tables

**Group and Aggregate**

`GROUP BY ...`

    SELECT ROUND(rating),
    count(*)
    FROM movies
    GROUP BY ROUND(rating)
    ORDER BY 1;
    // the '1' means the result first column, so here means the 'ROUND(rating)'

**Notice!** After group, you must and only aggregate for result, you can't just use `select *` to see all column, you must use `count(*)` or some other aggregation methods to aggregate the result, or select the only column you grouped by, in this case, you can `select rating`, I think it implicitly means `select DISTINCT(rating)`, grouped column will automatically be distincted.

~~(上面这句话不对，深深地误导了我。单表的时候是这样，但是如果多表进行 join，比如 users 表和 comments 表进行 join，用 users.id 进行 group，用 count(comments.id) 进行 order，是可以 select users.* 的)~~

(No No No! 上面的结论也是不对的，并不是因为 join 了才可以 `select *`，而是因为用的是 users.id 进行 group by，这才是最根本的原因，无论 join 与否。总结就是，如果你 group by 的列是 primary key，那么就可以 `select *`，否则，只能 select group by 的那一列和聚合结果，以下是测试结果：)

    select * from reviews group by id; // work!

    select * from reviews group by rating;  // wrong
    ERROR:  column "reviews.id" must appear in the GROUP BY clause or be used in an aggregate function
    LINE 1: select * from reviews group by rating;
                   ^

`WITH ... AS` means table view?

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

### 7. Sort Postgres Tables

`ORDER BY ... [DESC]`, the default order is `ASC`, and remember last section talked, you can order by a column name or column ordinal.

    SELECT *
    FROM friends
    ORDER BY friend_count, name desc;

### 8. Ensure Uniqueness in Postgres

`UNIQUE`

When create table:

    DROP TABLE directors;
    CREATE TABLE directors (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) UNIQUE NOT NULL
    )

After create table then want to add constraint for it:

    ALTER TABLE directors ADD CONSTRAINT directors_name_unique UNIQUE(name);
    ALTER TABLE movies ADD CONSTRAINT movies_title_release_unique UNIQUE(title, release_date);

Question: 

1. how to see a table's schema?
1. how to see a table's constraints?
1. how to remove a table's constraint? `remove constraint constraint_name`?

Use `\d+ table_name` to list a table's schema, constraints, index:

    postgres_101=# \d+ movies

    ...
    Indexes:
        "movies_pkey" PRIMARY KEY, btree (id)
        "title_release_uniq" UNIQUE CONSTRAINT, btree (title, release_date)
        "index_movies_on_title" btree (title)
    Check constraints:
        "count_stars_less_than_6" CHECK (count_stars < 6)
        "movies_count_stars_check" CHECK (count_stars > 0)

Use `alter table ... drop constraint ...` to drop a constraint:

    postgres_101=# ALTER TABLE movies DROP CONSTRAINT IF EXISTS "movies_count_stars_check1";
    ALTER TABLE

[More psql command](https://www.postgresql.org/docs/current/static/app-psql.html#APP-PSQL-META-COMMANDS).

### 9. Use Foreign Keys to Ensure Data Integrity in Postgres

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

Here after you add references for `director_id` in movies table to `id` in directors table, then if you want to insert a record to movies but the `director_id` not exists in directors table, it will fail.

    INSERT INTO movies (title, release_date, count_stars, director_id)
                values ('Transformer', '10-10-2011', 3, 4);
    // if current the director_id 4 doesn't exist in directors table,
    // it will be failed to insert into movies table

### 10. Create Foreign Keys Across Multiple Fields in Postgres

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

### 11. Enforce Custom Logic with Check Constraints in Postgres

When create table:

    CREATE TABLE movies (
      id SERIAL PRIMARY KEY,
      title VARCHAR(100) NOT NULL,
      release_date DATE,
      count_stars INTEGER,
      CHECK (count_stars > 0),
      CHECK (count_stars < 6)
    );

After create table:

    ALTER TABLE movies ADD CONSTRAINT count_stars_greater_than_0 CHECK (count_star > 0);
    ALTER TABLE movies ADD CONSTRAINT count_stars_less_than_6 CHECK (count_star < 6);

### 12. Speed Up Postgres Queries with Indexes

Before add index:

    EXPLAIN SELECT COUNT(*) FROM movies WHERE title='America';
    ->  Seq Scan on movies

After add index:

    CREATE INDEX CONCURRENTLY index_movies_on_title ON movies (title);
    CREATE INDEX CONCURRENTLY index_movies_on_year ON movies (year);
    CREATE INDEX CONCURRENTLY index_movies_on_title_year ON movies (year, title);

    EXPLAIN SELECT COUNT(*) FROM movies WHERE title='America';
    ->  Index Only Scan using index_movies_on_title on movies

### 13. Find Intersecting Data with Postgres’ Inner Join

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
    ON movies.id=rentable_movies.movie_id      // Notice! later join table can join with former joined table
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

Some visual explanations:

1. [What is the difference between "INNER JOIN" and "OUTER JOIN"?](https://stackoverflow.com/questions/38549/what-is-the-difference-between-inner-join-and-outer-join)
1. [sql 之 left join、right join、inner join 的区别](http://www.cnblogs.com/pcjim/articles/799302.html)
   - left join (左联接) - 返回包括左表中的所有记录和右表中联结字段相等的记录
   - right join (右联接) - 返回包括右表中的所有记录和左表中联结字段相等的记录
   - inner join (等值连接) - 只返回两个表中联结字段相等的行

### 14. Select Distinct Data in Postgres

    SELECT distinct title
    FROM movies;

----

## Note 3

Some others statement:

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

Verbs: 
- For databse / table: CREATE, DROP, ALTER
- For table rows: SELECT, UPDATE, DELETE
- For columns in table: ADD, DROP, ALTER, RENAME

### Dump and restore

    > pg_dump dbname > dumpfile

    > psql dbname < dumpfile

### Full Text Search

[*PostgreSQL Full Text Search*](https://www.postgresql.org/docs/current/static/textsearch.html)

    SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery;

    SELECT to_tsvector('fat cats ate fat rats') @@ to_tsquery('fat & rat');

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

(只有 where 不能使用聚合函数，而不光 having，select 和 order 也可以使用聚合函数，因为只有 where 是用来产生产生数据的，而 select/order/having 是用来操作结果的)

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
