# PHP Note 1

对 PHP 不感兴趣，只是因为工作需要，测试一个 PHP 项目，走了一遍 PHP 的环境搭建，故此记录一下，以防以后还需要。

- 环境配置

## 环境配置

参考：

- [Mac 下用 brew 搭建 LNMP 和 LAMP 开发环境](http://yansu.org/2013/12/11/lamp-in-mac.html)

PHP 的开发环境俗称 LAMP (Linux + Apache + MySQL + PHP)，现在一般用 Nginx 替换 Apache，因此又称 LNMP。

PHP 的开发和 Rails / Node 不太一样，Rails 和 Node 在开发阶段可以不需要 Apache 或 Nginx，Rails 通过 `rails s` 启动服务端，但 PHP 不行，必须依赖 Apache 或 Nginx，没有也不需要一个显式的命令来启动 PHP 程序。在本地开发时，从浏览器的本地访问请求也是必须经过 Apache 或 Nginx，然后由 Apache 或 Nginx 调用相应的 PHP 脚本。

(更正，简单看了 Symfony 的教程后，得知最新的 PHP 也内置了 web 服务器，在 Symfony 项目中通过 `php bin/console server:run` 启动。)

### Nginx

Nginx 的安装，可以看 Nginx Note，这里稍微复习一下。

安装：

    $ brew install nginx

启动：

    $ sudo nginx [-c /usr/local/etc/nginx/nginx.conf]

重新加载配置 / 停止 / 重启：

    $ sudo nginx -s reload|stop|restart

然后把自己网站的 nginx 配置文件放到 /usr/local/etc/nginx/servers/ 下，比如 mysite.conf。

    # /usr/local/etc/nginx/servers/mysite.conf
    server {
        listen       3000;
        server_name  localhost;
        root /Users/username/workspace/mysite;

        charset UTF-8;

        location / {
            index index.php;
            autoindex on;
        }

        # proxy the php scripts to php-fpm
        location ~ \.php$ {
            include /usr/local/etc/nginx/fastcgi.conf;
            fastcgi_intercept_errors on;
            fastcgi_pass   127.0.0.1:9000;
        }
    }

127.0.0.1:9000 是 php-fpm 服务的地址。上面这个配置的意思是说，Nginx 监听 localhost:3000 端口的请求，默认首页请求是访问 index.php，然后如果访问的是 .php 文件，那么把这个文件转给在 127.0.0.1:9000 端口监听的 php-fpm 服务来解析并返回结果。

在安装并启动 php-fpm 后，在 ~/workspace/mysite 下新建 index.php，随便写入点 php 代码，比如

    <!DOCTYPE html>
    <html>
    <body>

    <?php
    echo "Hello World!";
    ?>

    </body>
    </html>

然后在浏览器中输入 localhost:3000，如果能看到 `Hello World!` 的页面，则配置成功。

之后对 index.php 的修改都是即时生效的，并不需要重启 nginx 或 php-fpm 服务。

### MySQL

安装：

    $ brew install mysql

启动：

    $ mysql.server start

默认 root 密码为空。

### PHP & php-fpm & Composer

然后就是安装 PHP 了，macOS High Serra 已经都内置了 PHP 7.1.7 了。PHP 是以 homebrew 的插件形式存在的 (网上是这么说的，还不太明白)，所以要先用 `brew tap` 命令，再用 `brew install` 命令。

    $ brew tap homebrew/dupes
    $ brew tap homebrew/versions
    $ brew tap homebrew/homebrew-php
    $ brew install php71 php71-imagick php71-intl php71-opcache

PHP 中包括了一个叫 php-fpm (PHP-FPM - A simple and robust FastCGI Process Manager for PHP) 的服务，这个服务是用来和 Apache / Nginx 通信用的。本地开发时，从浏览器发出的本地请求到达 Nginx 时，Nginx 通过 php-fpm 来调用解析 PHP 脚本 (有待进一步确认)。

php-fpm 的启动：

    sudo /usr/local/opt/php71/sbin/php71-fpm start

php-fpm 的停止：

    sudo /usr/local/opt/php71/sbin/php71-fpm stop

另外一般还需要安装一个叫 [Composer](https://getcomposer.org/) 的程序，类似 Rails 中的 bundler，Node 中的 NPM，用来管理包的依赖，安装包。安装后在 PHP 项目下执行 `composer install` 来安装所有依赖。

Composer 的安装 (其实就是下载 installer 脚本，用 php 执行这个脚本，然后删除这个脚本)：

    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"

执行上面的脚本后会在当前目录下生成 composer.phar 的可执行文件，执行 `mv composer.phar /usr/local/bin/composer`，相当于实现了全局安装。

### 其它

- 怎么调试 PHP 代码
- 怎么打 log，怎么查看 log (`error_log()` ?)
