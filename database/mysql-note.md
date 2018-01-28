# MySQL Note

## References

- [MySQL 教程](http://www.runoob.com/mysql/mysql-tutorial.html)

## Note 1

Note for [MySQL 教程](http://www.runoob.com/mysql/mysql-tutorial.html)

### 安装

在 macOS 上的安装很简单，直接用 brew 安装：

    $ brew install mysql

启动服务端：

    $ mysql.server start

停止服务端：

    $ mysql.server stop

使用客户端连接：

    $ mysql

直接用 mysql 命令，不加任何选项连接 mysql 服务端时，默认是以当前系统的用户名登录，且密码为空，比如当前系统的用户名为 boo，则 mysql 命令相当于：

    $ mysql -u foo

但是用 brew 命令安装的 MySQL 并没有自动生成一个和当前系统用户同名的 user，因此登录会失败，但 MySQL 安装后自动生成了一个名为 root 且密码为空的用户，因此可以用下面的命令登录：

    $ mysql -u root
    mysql >

可以用 mysqladmin 命令为 root 用户设置密码，比如设置密码为 bar：

    $ mysqladmin -u root password bar

为 root 设置了错误的密码后，如何修改或清除 - [How to remove MySQL root password](https://stackoverflow.com/questions/3032054/how-to-remove-mysql-root-password)

修改：

    $ mysqladmin -u root -pType_in_your_current_password_here password new_pwd

清除：

    $ mysqladmin -u root -pType_in_your_current_password_here password ''

重新以用户名和密码登录：

    $ mysql -u root -p
    Enter password: ****
    mysql >

### 管理

进入 mysql 命令行后如何退出，用 `\q` 命令：

    mysql > \q

显示所有数据库：

    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.00 sec)

注意，命令遇到 `;` 才会真正执行。

切换数据库：

    mysql> use mysql;

mysql 是 MySQL 默认自带的数据库，里面放的是 MySQL 的管理数据，比如 user 表，存的就是所以允许登录 mysql 的用户，及其密码，权限。

显示当前数据库中的所有表：

    mysql> show tables;
    +---------------------------+
    | Tables_in_mysql           |
    +---------------------------+
    | ...                       |
    | user                      |
    +---------------------------+
    31 rows in set (0.00 sec)

显示某个表的所有列：

    mysql> show columns from user;

显示某个表的所有索引：

    mysql> show index from user;

增加用户，管理用户权限，暂略。

### 创建、删除数据库

创建：

    $ mysqladmin -u root -p create RUNOOB
    Enter pasword: ****

如果没有密码，就不用 `-p` 选项。

删除：

    $ mysqladmin -u root -p drop RUNOOB

选择数据库，前面讲了，先登录，再用 use 命令。

### 数据类型

分三大类：数值、日期 / 时间、字符串。

详略。

### 其它

其它，略，看 PostgreSQL Note。

包括表的创建、删除、修改，记录的 CRUD (Create / Retrieve / Update / Delete)。

- create / drop / alter a database
- create / drop / alter a table of a database
- add / drop / alert / rename a column of a table
- insert / select / update / delete records for table

### Dump

导出：

    $ mysqldump -u root -p database_name > dump.txt

如果没有密码，则不用 `-p` 选项。

如果只需要导出单个表，则用：

    $ mysqldump -u root -p database_name table_name > dump.txt

导入：

    $ mysql -u root -p database_name < dump.txt

应该要确保当前 `database_name` 中没有数据，如果有，可以先把它 drop 掉，再重新 create，然后导入。

另一种方式：

    $ mysql -u root -p
    mysql > use database_name;
    mysql > source ~/path/dump.txt

上面只是最简单的用法，更复杂的用法需要时再看。
