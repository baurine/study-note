# WireGuard 使用

WireGuard 用来组建自己的虚拟网络 (Virutal Private Network)。

假设别人已经有组好的网络 (即已配置好服务端)，你作为客户端加入。

在 Mac 上，首先安装相应的客户端：`brew install wireguard-tools`。

在 /usr/local/etc 目录下创建 wireguard 目录。

生成公钥私钥对：`wg genkey | tee privatekey | wg pubkey > publickey`。

将公钥交给服务端。私钥用于自己的配置文件。

在 wireguard 目录下创建 wg0.conf 配置文件。

配置文件内容分两部分，一部分是自己的配置，一部分是 peer 的配置：

```
[Interface]
Address = 10.0.0.201/24
ListenPort = 51820
PrivateKey = # 这里填入刚才生成的私钥

#SH-WAN
[Peer]
PublicKey =
Endpoint =
AllowedIPs = 10.0.0.1/32, 192.168.33.0/24, 192.168.0.0/24, 192.168.31.0/24
PersistentKeepalive =
```

Peer 中的内容由服务端提供给你，包括它的公钥，Endpoint 等信息。

最后，启动 wireguard，加入 VPN 中：`wg-quick up wg0`。

- [wireguard vpn 安装 mac 客户端](https://unpc.github.io/2019/02/18/wireguard%20vpn%E5%AE%89%E8%A3%85mac%E5%AE%A2%E6%88%B7%E7%AB%AF/)

然后就可以根据上面配置中的 AllowedIPs 中的 IP 进行访问，比如 `http://192.168.33.200/`。
