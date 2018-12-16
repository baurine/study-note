# Web Security

记录一些关于 Web 安全的知识。

参考：

- [浏览器：一个家族的奋斗](https://mp.weixin.qq.com/s/5mOPs9Vcl0VNq4NAjdRPAg)
- [浏览器家族的安全反击战](https://mp.weixin.qq.com/s/Wpy04kDKOWgpz0qEEWC2EQ)
- [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
- [HTTP 访问控制 (CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)
- [黑客三兄弟](https://mp.weixin.qq.com/s/YvYvL0siJT1UhO0tXnYVNA)
- [黑客三兄弟 (续)](https://mp.weixin.qq.com/s/_-7C_ZfFfvNKhBQzSB6j4Q)
- [别用 raw 和 html_safe](https://ruby-china.org/topics/16633#reply-337949)
- [如何让前端更安全? - XSS 攻击和防御详解](https://juejin.im/entry/58a598dc570c35006b5cd6b4)
- [XSS 攻击及防御](http://blog.csdn.net/ghsau/article/details/17027893)
- [Web 安全：CSRF 与 XSS](https://blog.csdn.net/victor_ll/article/details/79477655)
- [Everything you should know about certificates and PKI but are too afraid to ask](https://smallstep.com/blog/everything-pki.html)

## cookie，同源策略，CORS

cookie，把 cookie 理解成浏览器为每个网站提供的在客户端中的一小块数据私有地址，你可以往这里存任意数据。就像 Android APP 的 SharePreference。

一开始，任何网站都可以访问浏览器中所有的 cookie，这样 A 网站就可以窃取 B 网站在浏览器中的 cookie。

后来浏览器做了限制，每个网站只能访问本域名下 cookie (第一次重大的限制)。

同源策略，要求 protocol / host / port 相同。

这个同源策略确实严格， 不同源的网页无法访问另外一个网页的 DOM，Cookie，恶意网站想偷走别的网站的 cookie 就不容易了。

但 script img link 标签除外，因此可以用 script 实现 JSONP，跨域请求。

CORS (Cross Origin Resource Sharing) - 一个服务器可以设置一个白名单，里边列出它允许哪些其它的 domain 的 AJAX 请求。

当进行非简单请求的跨域请求时 (比如在 a.com 页面中以 PUT 方式访问 b.com 的 API)，浏览器做了点手脚，它会首先在后台悄悄地把当前的源 a.com 发到 b.com，询问 b.com 是否允许 a.com 访问，如果允许，则继续访问，否则报错。详细过程看这篇文章 - [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

CORS 需要浏览器和服务器同时支持。

引起这一切的根源就是 cookie，而且是 javascript 可以主动访问 cookie，最终解决方案，不用 cookie 行了。在 Android / iOS 的客户端，敏感数据放在各 APP 私有的数据空间，其它 APP 无法访问，其实，除了自己的代码，没有其它代码都可以访问其中的数据。

## 注入 - XSS

浏览器最大的问题是，javascript 太灵活了，你无法控制它的来源，它甚至可以来自用户的输入。通过用户输入进行攻击，一般称为注入。

不能信赖用户的输入，注入攻击来自外部输入。

针对同源策略的反制手段，在目标域名下网页中，注入我们的 javascript 代码，使之在目标域名运行。

怎么注入，如果对方网站有输入框，直接在输入框中输入 js 代码。

注入一段 js 代码，窃取 cookie，这就是 XSS。

执行的细节，在注入的 js 代码中生成一个 img 标签，设置 img 的 src 为跨域的 get 请求，这个 get 请求发送至我们自己的服务器，并在请求中带上 cookie 中的内容作为参数。妙！最近 ryf 说用 img 跟踪用户请求，用的是同一种思想。

解决这个漏洞的一些方法：

1. 在 cookie 中加上 HttpOnly，禁止 javascript 操作 cookie，只允许浏览器根据 http header 来修改 cookie
1. 在后端对用户的输入进行过滤，去掉 `<script>` 标签，比如把 `<script>` 变成 'script'，或完全删除
1. 在前端显示时对输入进行转义，比如把 `<script>` 变成 `&lt;script&gt;`

哦：

1. 把 `<script>` 变成 `script` 或删除，这叫过滤
1. 把 `<script>` 变成 `&lt;script&gt;`，这叫转义

> 一方面他们有人会对输入进行过滤，发现不符合他们要求的输入例如 `<`，`>` 等就会过滤掉，我们的 `<script>` 可能会变成 `script` 被存到数据库里。

> 另一方面有人还会对输出进行编码/转义操作，例如会把 `<` 变成 `&lt;`，把 `>` 变成 `&gt;`，然后再输出，这样一来我们的 `<script>` 就会变成 `&lt;script&gt;`，浏览器收到以后，就会认为是数据，把 `<script>` 作为字符串给显示出来，而不是执行后面的代码！

通过转义的，还可以反转义回来，但过滤的，就变不回来了...

另外，即使 `<script>` 作为原始文本，存入了数据库中，也不用太担心，还是有办法的。可以在渲染这些数据的时候，对这些数据进行转义或过滤，不要直接显示。

在 Rails 的 View 中，所有输出默认都是转义的：

    <%= danger_string %>

如果想输出原始内容 (即取消默认的转义，一般需要显示富文本时，用 raw 方法)，则用：

    <%= raw danger_string %>

但直接用 raw 时，可以带来风险，因为 danger_string 中可能含有可执行的 javascript 代码，我们要把这个危险代码过滤掉，可以使用 sanitize 方法来过滤危险代码：

    <%= sanitize danger_string %>

转义和过滤的区别，参看我在这篇帖子中的评论：[别用 raw 和 html_safe](https://ruby-china.org/topics/16633#reply-337949)

在 React 中，默认输出也都是转义的，比如：

    <p>{danger_string}<p>

如果想输出原始内容，则需要显式声明：

    <p dangerouslySetInnerHTML={{__html: danger_string}}>

(疑惑，在 React 中有没有类似 Rails 中提供的 sanitize 过滤方法?)

## CSRF

XSS 的升级攻击：CSRF - 跨站请求伪造 Cross Site Request Forgery (CSRF)

只要打开一个页面 (这个页面是攻击者的域名，但里面隐藏了 img 标签，img 标签的 src 指向攻击的目标域名，比如银行网站)，什么都不做，攻击就完了。(妈呀，原来真的是这样的，新闻中老听说，一打开页面，什么都没做，钱就转走了...)

把请求隐藏在 img，太牛了。

具体细节见 [黑客三兄弟](https://mp.weixin.qq.com/s/YvYvL0siJT1UhO0tXnYVNA)

解决办法，要求每个请求带上一个服务端生成的 token。

总结一下：

- XSS - 在目标域名网页中进行注入攻击
- CSRF - 在攻击者自己的页面中进行攻击 (非注入)

## SQL 注入

SQL 注入

    SELECT id, name, age from users WHERE id=1 or 1=1
    SELECT xxx from ep_users WHERE user='admin' AND password='password' OR '1'='1'

最重要的一个原则：不要信赖用户的输入，不要直接使用用户的输入

## 转义和过滤

再举例详细说一下转义和过滤。

以 Rails 为例，假设用户在某个页面中输入评论：`<h1>Test XSS</h1><script>alert(document.cookie)</script>`，这段输入中含有危险的 script 代码，我们需要对它进行处理。

当这段输入传到服务器时，我们可以在后端对这段输入进行处理，转义或过滤，然后存入数据库，这样前端显示时就不用再转义或过滤；或者在后端不处理，直接将原始输入存入数据库中，这样，前端显示时，就要进行转义或过滤。

我们采取在后端存储原始文本，在前端对文本进行转义或过滤。Rails (4.0 版本以上) 在渲染 View 时的默认行为是转义。所以假设 `danger_str = "<h1>Test XSS</h1><script>alert(document.cookie)</script>"`，那么：

    <%= danger_str %>

实际得到的输出是：

    &lt;h1&gt;Test XSS&lt;/h1&gt;script&gt;alert(document.cookie)&lt;/script&gt;

这样的 HTML 代码，在浏览器中渲染时，`&lt;` 会显示成 `<`，`&gt;` 会显示成 `>`，因此，我们可以在浏览器中看到和用户的输入一样的文本：

    <h1>Test XSS</h1><script>alert(document.cookie)</script>

(你会想，如果我就是想在浏览器中显示 `&lt;` 该怎么办，那么把 `&` 转义成 `&amp;`，即 `&amp;lt;` 在浏览器中就可以显示成 `&lt;`)

但是，如果我们的网站想支持用户的富文本输入，比如 `<h1>Test XSS</h1>`，我们就是想输出 h1 的 Test XSS，即将用户的输入作为原始的 HTML 代码输出，不进行转义，那么可以用 raw 方法来支持富文本输出：

    <%= raw danger_str %>

这样，得到的输出就是原始文本：

    <h1>Test XSS</h1><script>alert(document.cookie)</script>

这样的 HTML 代码，在浏览器中渲染时，`<h1>Test XSS</h1>` 显示成大号加粗的 "Test XSS"，这是期望值，但后面的 `<script>alert(document.cookie)</script>` 会作为 script 代码进行执行，弹出一个 alert 窗，显示你的 cookie，这是不行的。

那怎么办呢，那就把它过滤掉，可以用 sanitize 方法把整个 script 标签的内容删除掉：

    <%= sanitize danger_str %>

得到的输出：

    <h1>Test XSS</h1>

这样，我们显示了大号加粗的 "Test XSS"，又不会执行危险代码，完美。

总结一下：

1. 如果不需要支持富文本输出，那就用 Rails 默认的转义就行，用户输出什么，就给它显示什么。
1. 如果要支持富文本输出，又要避免 XSS 风险，就用 sanitize 方法，直接把危险的标签及其内容删除。

当然，上面说是只是很简单的例子，需求是多变的，比如说，爬虫，我们去爬取别人网站上的一段 HTML 文本，里面有各种标签，比如 `<h1>`, `<p>`，但是我们只想要其中的 text，不想要这些标签，那就需要更强大的 sanitizer，过滤所有标签，保留标签中的内容，有第三方库实现的更强大的 sanitizer，Rails 也提供了 `strip_tags` 及 `full_sanitizer` 等方法和对象来实现更灵活的过滤。

## [Everything you should know about certificates and PKI but are too afraid to ask](https://smallstep.com/blog/everything-pki.html)

// TODO
