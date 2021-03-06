# Register Domain

关于注册购买域名。

## References

- [NameSilo 注册使用全攻略](http://flamepeak.com/2016/09/02/NameSilo-gong-lue-20160902/)

## Note

首先，可供选择的，较知名的域名注册商 (个人使用，暂不考虑国内) 有 Godaddy，Name，NameCheap，NameSilo。综合了网上的评价，选择了比较便宜的 [NameSilo](https://www.namesilo.com/index.php)。

取得域名之后，我们要给这个域名增加 A 记录和 CNAME。

这些记录在哪里添加，是要在域名解析服务商的网站上设置。

> 当你购买域名之后，这个域名的 DNS 记录都是在域名使用的 NS 服务器上面设置的，而 NS 服务器是可以根据需要改动的，比如我在 NameSilo 注册了域名，但是我的网站是面向国内浏览者的，为了提高访问速度我把 NS 服务器放在了 DNSPOD 上面。那么涉及到网站的 A 记录、CNAME 记录和 MX 记录等添加、修改、删除都需要在 DNSPOD 上面完成，也就是说 NS 服务器在哪里，就去哪里设置 DNS 记录。

(MX 记录与邮件服务器相关，暂时不关心。)

我直接使用 NameSilo 提供的域名解析服务，所以我们就直接在 NameSilo 上增加 A 记录和 CNAME。

所谓 A 记录，其实是 Address Record 的简称，再说白一点，它就是指你的服务器的 IP 地址，得到域名之后，你要把你的服务器的 IP 地址和这个域名关联起来，这样，别人访问这个域名之后，域名服务器就知道要指向这个 IP 地址。

IP 地址从你的服务提供商那里得知，比如我是购买的 AWS EC2 云主机，从 EC2 的控制面板处可以得到这个值。把它填到 A 记录的 IP 地址栏里。

CNAME，指向的另一个域名。比如，我之前已经有一个解析正常的域名，假设为 a.com，IP 为 1.2.3.4，现在我又新注册了一个域名 b.com，我暂时没有什么用处，就想也让它解析到 1.2.3.4 上，我既可以给 b.com 直接指定一个 A 记录 1.2.3.4，也可以给 b.com 指定一个 CNAME 为 a.com，这样 b.com 就会解析到 a.com 所指向的 IP 上。

如果一个域名既有 A 记录，又有 CNAME，那么 A 记录的优先级更高。

注册域名过程中，有个 `auto-renew` 的选项，那是说是否自动续费的意思。

---

2021/02/19 Update

我在 NameSilo 上注册了域名 (比如 `baurine.com`)，但我的人个网站是布署在 Netlify 上的，这时我需要在 Netlify 给个人网站项目添加自己的域名，然后 Netlify 会告诉你它自己的域名服务器 (NameServers)，我们复制下来，回到 NameSilo 中，将默认的 NameServers 改成 Netlify 的 NameServers。

Netlify NameServers:

![Netlify NameServers](../art/netlify-nameservers.jpg)

NameSilo NameServers:

![NameSilo NameServers](../art/namesilo-nameservers.jpg)
