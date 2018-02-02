# PHP Note 2

Note for [PHP 教程](http://www.runoob.com/php/php-tutorial.html)

由于项目需要，简单学习一下 PHP 的语法。

    <?php
    // php 代码
    ?>

看了一遍下来，PHP 基本就是结合了 C 和 Java 语法的 Web 脚本语言，语法简洁明了，没有 Ruby 那么多魔法。一开始的思想是想像 C 一样面向结构编程，后来加入了面向对象的语法后，也可以面向对象编程了。

## 基础语法

### 变量

变量，弱类型，像 ruby，不用 var / let / const 声明，但是奇芭的是必须用 $ 标志。

- 局部 - 在函数内定义
- 全局 - 在函数外定义
- 静态 - 在函数中用 static 声明

如果想在函数内访问全局变量，两种方法。

- 提前用 global 声明一下需要访问的全局全量
- 直接用 $GLOBAL[index] 访问

### 常量

常量用 `define(name, value, case_insensitive)` 定义，奇芭...

    <?php
    // 区分大小写的常量名
    define("GREETING", "欢迎访问 Runoob.com");
    echo GREETING;    // 输出 "欢迎访问 Runoob.com"
    ?>

定义的常量可以全局访问，实际是把它放到了一个全局数组中，类似全局变量。

### 超级全局变量

- $GLOBALS
- $_SERVER
- $_REQUEST
- $_POST
- $_GET
- $_FILES
- $_ENV
- $_COOKIE
- $_SESSION

(果然，PHP 的目标很专一，就是为 Web 为生，而不是成为一门通用语言)

### 魔术变量

- `__LINE__`
- `__FILE__`
- `__DIR__`
- `__FUNCTION__`
- `__CLASS__`
- `__TRAIT__`
- `__METHOD__`
- `__NAMESPACE__`

trait 的用法，类似 Ruby 中的 include 一个 module。

### 输出

输出 echo / print，支持插值。

EOF

    <?php
    echo <<<EOF
        <h1>我的第一个标题</h1>
        <p>我的第一个段落。</p>
    EOF;
    // 结束需要独立一行且前后不能空格
    ?>

### 数据类型

- 基本 - 字符串，整型，浮点，布尔，null 值，数组 (array)
- 对象 - class

(注意，class 中的成员变量要用 var 声明)

    <?php
    class Car
    {
      var $color;
      function Car($color="green") {
        $this->color = $color;
      }
      function what_color() {
        return $this->color;
      }
    }
    ?>

- var_dump($var)
- print_vars($obj)

### 字符串

用 `.` 号连接...

### 数组

数组，两种形式

- 普通数组，通过索引访问

        $cars=array("Volvo","BMW","Toyota");
        $cars[0]

- Key-value 对，通过 key 访问

        $ages = array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
        $ages[“Peter”]

数组的排序方法：sort($arr)，还有 rsort / asort / ksort / arsort / krsort 等方法。

数组的长度：count($arr)

### 运算符

略

PHP7+ 支持太空船运算符 `<=>`

    $c = $a <=> $b;
    // $c = $a > $b ? 1 : ($a < $b ? -1 : 0)

### 流程控制

If / else / switch

略

### 循环

while / for / foreach

略

### 函数

用 function 关键字定义，和 JavaScript 一样，略。

### 命令空间

用 namespace 定义，可以定义多级命名空间，用 `\` 分开 (用 `\` 也是够奇芭，好丑...)

    namespace MyProject;

    namespace MyProject\Sub\Level;

PHP 中用 `::` 来访问命令空间或类中的静态方法。

    A\B::foo();

### 面向对象

基本是把 Java 那一套抄过来了... 就只有调用成员那一点是用了 C++ 的语法...

- class 定义类
- new 生成对象
- var 定义成员变量
- const 定义成员常量
- 用 `->` 访问对象的成员
- `__construct` 构造函数
- extends 继承
- final 类
- interface / implements 接口
- abstract 抽象类
- public / protected / private 访问控制

## 表单

### 表单和用户输入

PHP 用 `$_GET` 和 `$_POST` 全局变量处理表单中的信息，前者存放 get 请求的值，后者存放 post 请求的值。

(表单的 method 可以是 get，也可以是 post。)

举了下拉单选、下拉多选、按钮单选、按钮多选四个例子。详略。

处理表单上，PHP 和 Rails 是完全不一样的，Rails 中所有表单，不管是 get 请求，还是 post 请求，都通过 params 方法取得。

从后面得知，PHP 还有一个全局变量叫 `$_REQUEST`，get 请求和 post 请求的值都在里面，因此，也可以统一用 `$_REQUEST` 来取得表单中的值。

### 表单验证

在服务端对提交的表单进行验证。这里面提及了一些注意点，来避免被黑客利用进行入侵，最重要的是，一定要对输入进行检查，过滤，将标签转换成转义代码。

## 高级教程

### 多维数组

    <?php
    // 二维数组:
    $cars = array
    (
        array("Volvo",100,96),
        array("BMW",60,59),
        array("Toyota",110,100)
    );
    ?>

    <?php
    $sites = array
    (
        "runoob"=>array
        (
            "菜鸟教程",
            "http://www.runoob.com"
        ),
        "google"=>array
        (
            "Google 搜索",
            "http://www.google.com"
        ),
        "taobao"=>array
        (
            "淘宝",
            "http://www.taobao.com"
        )
    );
    print("<pre>"); // 格式化输出数组
    print_r($sites);
    print("</pre>");
    ?>

PHP 中的全局变量 `$_FILES` 就是一个二维数组。

### 日期

date() 函数，略。

### 包含文件

模块化。使用 include 或 require 导入其它 php 文件。

区别：

- include - include 时发生错误，忽略并继续
- require - require 时发生错误，抛出导常并停止

### 文件处理

即操作文件系统，使用了 C 的语法。

- fopen() - 打开文件
- fclose() - 关闭文件
- feof() - 判断文件是否到结尾
- fgets() - 逐行读取
- fgetc() - 逐字符读取

### 处理文件上传

PHP 用全局变量 `$_FILES` 来接收上传的文件信息，它是一个二维数组。

第一个下标是表单的 input name，第二个下标可以是 "name"、"type"、"size"、"tmp_name" 或 "error"。如下所示：

- $_FILES["file"]["name"] - 上传文件的名称
- $_FILES["file"]["type"] - 上传文件的类型
- $_FILES["file"]["size"] - 上传文件的大小，以字节计
- $_FILES["file"]["tmp_name"] - 存储在服务器的文件的临时副本的名称
- $_FILES["file"]["error"] - 由文件上传导致的错误代码

### Cookie

PHP 用全局变量 `$_COOKIE` 存储来自用户请求中的 cookie (在 http header 中)。

在响应请求时，为客户端设置 cookie 或删除 cookie，用 setcookie() 方法，且必须放在 `<html>` 标签前。

设置 cookie

    <?php
    $expire=time()+60*60*24*30;
    setcookie("user", "runoob", $expire);
    ?>

    <html>
    ...

删除 cookie

    <?php
    // 设置 cookie 过期时间为过去 1 小时
    setcookie("user", "", time()-3600);
    ?>

    <html>
    ...

响应的 cookie 放在 reponse 的 header 中。

在服务端查看请求中的 cookie

    <?php
    // 输出 cookie 值
    echo $_COOKIE["user"];

    // 查看所有 cookie
    print_r($_COOKIE);
    ?>

### Session

session 和 cookie 的区别，不再赘述，我在 rails note 中做过笔记了 (在《Agile Web Development Rails 5》笔记中)，也发表到 blog 上了。

开始 PHP Session，使用 `session_start()`：

    <?php session_start(); ?>

    <html>

存储 Session 变量，使用 `$_SESSION` 全局变量：

    <?php
    session_start();
    // 存储 session 数据
    $_SESSION['views']=1;
    ?>

    <html>

销毁 Session 变量，使用 unset() 或 `session_destroy()` 方法。

    <?php
    session_start();
    if(isset($_SESSION['views']))
    {
        unset($_SESSION['views']);
    }
    ?>

    <?php
    session_destroy();
    ?>

### 邮件处理。

用 PHP 发邮件，mail() 函数，详略。

### 错误处理

打印错误消息并终止脚本 - `die(err_msg)`

自定义错误处理 handler - `set_error_handler()`

主动触发错误 - `trigger_error()`

    <?php
    // 错误处理函数
    function customError($errno, $errstr)
    {
        echo "<b>Error:</b> [$errno] $errstr";
    }

    // 设置错误处理函数
    set_error_handler("customError");

    // 主动触发错误
    $test=2;
    if ($test>1)
    {
        trigger_error("变量值必须小于等于 1",E_USER_WARNING);
    }
    // 或被动触发
    // $test2 并不存在
    echo $test2
    ?>

将错误输出到 log 文件或邮件中 - `error_log()`

### 异常处理

和 Java 的 try...throw...catch 几乎是一样的，详略。

### 过滤器

过滤用户的输入

- `filter_var()`
- `filter_var_array()`
- `filter_input()`
- `filter_input_array()`

### JSON

- `json_encode()`
- `json_decode()`

## PHP7 新特性

- [PHP7 新特性](http://php.net/manual/zh/migration70.new-features.php)

继续借鉴 Java，支持匿名类等新特性，详略。

## PHP 与 MySQL

MySQL 是跟 PHP 配套使用的最流行的开源数据库系统。(与之对比的是在 Rails 生态中，一般选择 PostgreSQL。)

这里介绍的都是很底层的 MySQL 操作，手写 sql query 语句，现在一般框架都提供了 ORM 功能，不用再写这么底层的代码。

详略。

## PHP 与 AJAX

略。
