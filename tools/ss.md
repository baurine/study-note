# SS 使用

服务端略。

客户端。

首先安装图形化界面的客户端或纯命令行工具。

这里介绍命令行工具，使用 `pip3 install git+https://github.com/shadowsocks/shadowsocks.git@master` 安装，安装后的命令是 sslocal。

创建 ss.json 配置文件，使用 `sslocal -c ss.json` 在本地启动 ss。

启动 ss 后，它并不会自动接管所有网络请求。每个其它客户端需要额外的配置来将自己的请求转发到 ss 上。

对于浏览器来说，我们可以用一个插件 - Proxy SwitchyOmega 来方便地进行配置。安装插件后，在插件的配置界面的 Proxy Servers 中，添加一项 protocl 为 SOCKS5，地址和端口分别为 localhost 和 1090 的 server。

(使用图形化工具的 ss 客户端会帮我们自动把上面的所有步骤都做了。你只要给它填好 ss 服务端参数后，它会自动在本地启动 sslocal 监听网络，为流览器配置好代理。)

对于命令行来说，我们可以安装 proxychains：`brew install proxychains-ng`。安装后，修改其配置 `/usr/local/etc/proxychains.conf`，将最后一行修改为 `socks5 127.0.0.1:1090`。然后如何使用呢，在执行需要进行网络请求的命令时，在最前面加上 proxychains4，比如 `proxychains4 git clone git@github.com:webp-sh/webp_server_go.git`。

使用 ip.cn 这个网站可以方便地查看当前 IP 及相关信息。
