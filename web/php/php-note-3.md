# PHP Note 3

参考：

- [十分钟学会 Symfony](http://www.cnblogs.com/Amos-Turing/p/5968260.html)
- [用 Symfony 3 建立 Web 应用](https://taylorr.gitbooks.io/building-a-web-site-with-symfony/content/)
- [Symfony 中文文档](http://www.symfonychina.com/doc/current/index.html)

由于项目需要，简单学习一下 Symfony 3 的用法，不会深入学习，了解一下整体框架。一个 Web 框架，总的内容那就是那些，MVC，路由，模板，ORM 等。

另外，PHP 现在还有一个很流行的框架叫 laravel，模仿 rails，使用起来非常相似，rails 程序员真的可以半小时上手。

- [Ruby 程序员学习 laravel 框架笔记](http://rubylaravelbook.rails365.net/467152)

## Part 1

Note for [十分钟学会 Symfony](http://www.cnblogs.com/Amos-Turing/p/5968260.html)

Symfony 的工程可以由 composer 来生成。

项目的配置文件：app/config/parameters.yml, app/config/config.yml

Bundle 的概念：

> 你可以把 Symfony 的 DI 理解成为一个功能池，把程序中的所有功能都做成 Bundle，或者你把 Bundle 理解成一组 php 文件组合而成的程序就可以。 比如用户注册，登录功能做成一个 Bundle, 你也可以把一个论坛的发帖回贴功能做成一个 Bundle，自然也可以把文章管理做成一个 Bundle，然后用一个 Bundle 去调用和配置不同的 Bundle，那么你就可以把网站组装起来了，而你写的各种 Bundle，在其他的应用程序中还可以继续复用，这样写的 Bundle 越多，可复用性就越强，制作新项目的时候也越有利。

(DI，所以 Symfony 是类似 Java Spring 框架? 而且，Symfony 的代码中也有注解...)

Bundle 中包含的就是 php 代码，放在 src 目录下，src 目录和 app 目录同级。

在每个 Bundle 中，包含以下一些目录：

1. Entity - 这个目录并不是必须的，很多情况下只有在生成实体的时候才会生成，放置模型，也就是 MVC 中的 M
1. Controller - 这个目录会生成 DefaultController.php，你可以在这里建立自己的 Controller 控制器，也就是 MVC 中的 C
1. Resources - 这个目录下面还有子目录，其中 views 放置的是模板，也就是 MVC 中的 V，而 public 放置的是静态文件，比如 js, css, images 等等
1. Tests - 放置单元测试与集成测试的代码，在这个样例程序中暂时不需要
1. DependencyInjection - 与 DI 相关的目录，暂时也不需要去了解
1. SymfonySampleBundle.php - 当前这个 Bundle 的定义文件

Model，使用的 ORM 框架叫 Doctrine。

View 模板使用的是 Twig，后缀是 .html.twig。

## Part 2

Note for [用 Symfony 3 建立 Web 应用](https://taylorr.gitbooks.io/building-a-web-site-with-symfony/content/)

### Symfony 重要概念

**MVC**

略。

**Bundle**

在上面已经介绍了，再详述一下。Bundle 就是一个独立的功能体。(可以理解成 Rails 中的 Gem 包)

一个 Bundle 可能包含以下目录：

- Controller 中存放所有控制器代码。
- DataFixtures 中是样本数据填充。
- Entity 中是所有的数据实体。可以简单地理解为一张张表格。
- Form 中存放所有的表单类型，用于生成表单。
- Repository 中是对数据实体的一些自定义操作。
- Resources/config 中有一部分是以 YML 形式定义的 Doctrine 数据实体；另一部分可以存放针对本包的配置文件，如 routing.yml 文件。
- Resources/views 中会存放所有本包要用到的视图模板，以 Twig 语法写成。
- Tests 中可以存放各种测试文件，既可以是单元测试也可是是功能测试。
- Twig 中存放着专为 Twig 编写的定制过滤器。

在实际应用中，你的应用可能不会有这么多目录。一个典型的 SF2 安装，是没有 DataFixtures, Form, Repository, Twig 等目录的。

事实上，有些目录的命名也完全是任意的。

**Routing**

用 YAML 语法定义在 yml 文件中 (但没说具体定义在哪个 yml 文件中... 或者是类似 src/AppBundle/Resources/config/routing.yml)

一个例子：

    book_list:
      pattern: /books/list/{type}/{page}/{key}
      defaults:
        page: 1
        key: all
        type: title
        _controller: AppBundle:Book:list

这个路由表示，用 AppBundle 中的 BookController 中的 listAction() 方法来处理 `/books/list/{type}/{page}/{key}` url。

这个 Controller 的路径一般位于 src/AppBundle/Controller/BookController.php。

**Controller**

控制器其实就是一个 PHP 类。

路由：

    homepage:
      pattern:  /
      defaults: { _controller: AppBundle:Default:index }

对应的 Controller：

    //File: src/AppBundle/Controller/DefaultController.php

    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\Security\Core\Security;

    class DefaultController extends Controller
    {
        public function indexAction()
        {
            // Code for index action.
        }
        // More codes and more actions
    }

**Entigy**

Model

**Repository**

PHP 为 Controller 和 Model 之间增加了一层 Repository。

Model 只存放数据，Repository 负责对 Model 的操作，增删改查。

**Template**

使用 Twig

**Test**

使用 PHPUnit

### 创建应用

安装 symfony

    $ sudo curl -LsS http://symfony.com/installer -o /usr/local/bin/symfony
    $ sudo chmod a+x /usr/local/bin/symfony

用 synfony 创建新应用

    $ symfony new the_new_project_name

进入目录，用 composer 安装依赖包

    $ composer install

目录结构说明：

- app 目录 - 框架的运行核心，重要文件如 autoload.php，AppKernel
  - config 目录 - 存放应用的配置
  - Resources 目录 - 存放应用级别的资源，如模板文件 (在 views 子目录下)
- bin 目录
  - console 命令，可以用来启动 PHP 内置的 web 服务器，从而不需要在配置 nginx 的情况下测试，还包括其它功能
- src 目录 - 存放各个 Bundle
- tests 目录
- var 目录
  - cache 目录
  - logs 目录
  - sessions 目录
- vendor 目录 - 第三方包和代码
- web 目录 - web 服务器的入口
  - app.php, app_dev.php - 生产环境和开发环境的入口
  - assets - image / css / js

其它略。

## Part 3

Note for [Symfony 中文文档](http://www.symfonychina.com/doc/current/index.html)

### 设置

安装，略，同 Part 2 中所记录的。

之前以为 PHP 的开发必须是要配置 Nginx 的，不能像 Rails 中用 `bin/rails server` 启动一个内置 web 服务器来开发调试。但是现在 PHP 也内置了 web 服务器，可以通过 `php bin/console server:run` 来启动一个内置的 web 服务器。

### 创建第一个页面

创建一个 Controller。

关于路由，有四种创建方法：

- 直接在 Controller 中用 `@Route()` 注解声明
- 在 app/config/routing.yml 中用 YAML 定义
- 在 app/config/routing.xml 中用 XML 定义
- 在 app/config/routing.php 中用 PHP 代码定义

详略。

渲染模板，render() 方法，详略。

### 路由

详略。

### 控制器

详略。

### 模板

略。

### 配置

在 app/config/config.yml 中可以包含其它配置。

    # app/config/config.yml
    imports:
        - { resource: parameters.yml }
        - { resource: security.yml }

    framework:
        secret:          "%secret%"
        router:          { resource: "%kernel.root_dir%/config/routing.yml" }
        # ...

    # Twig Configuration
    twig:
        debug:            "%kernel.debug%"
        strict_variables: "%kernel.debug%"

    # ...

其余略，需要时再看。
