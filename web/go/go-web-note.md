# 《Go Web 编程》Note

[eBook Link](https://github.com/astaxie/build-web-application-with-golang)

## 1 - Go 环境配置

Go 的所有代码都必须放在 GOROOT 或 GOPATH 中。

设置 GOPATH 环境变量，三个子目录：

- src - 存放源代码
- pkg - 编译后生成的文件，比如 .a
- bin - 编译后生成的可执行文件

main 包生成可执行文件，其它包生成 .a

Go 命令：

- go build
- go run
- go clean
- go fmt
- go get
- go install - 这个命令在内部实际上分成了两步操作: 第一步是生成结果文件 (可执行文件或者 .a 包)，第二步会把编译好的结果移到 $GOPATH/pkg 或者 $GOPATH/bin。
- go test
- go doc
- go fix
- go env
- go list
- ...

## 2 - Go 语言基础

make 用于内建类型 (map、slice 和 channel) 的内存分配。new 用于各种类型的内存分配。

import 的三种操作：

```go
import (
  . "fmt"  // 点操作，调用时可以省略包名，比如 fmt.Printf() --> Printf()
  f "fmt"  // 别名操作，fmt.Printf() --> f.Printf()
  _ "fmt"  // _ 操作其实是引入该包，而不直接使用包里面的函数，而是调用了该包里面的 init 函数
)
```

嵌入类型，又称嵌入字段，匿名字段...

接口也可以嵌入... 用来实现其它语言中的接口继承，看来 Go 是将组合替代继承的思想贯彻到底！

```go
// io.ReadWriter
type ReadWriter interface {
  Reader
  Writer
}
```

Go 实现了反射。

其它略。

## 3 - Web 基础

### 3.1 - Web 工作方式

略。

### 3.2 - Go 搭建 Web

一个最简单的例子 (堪比 node.js)：

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()       // 解析参数，默认是不会解析的
	fmt.Println(r.Form) // 这些信息是输出到服务器端的打印信息
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val:", strings.Join(v, ""))
	}
	fmt.Fprintf(w, "Hello astaxie!") // 这个写入到 w 的是输出到客户端的
}
func main() {
	http.HandleFunc("/", sayhelloName)       // 设置访问的路由
	err := http.ListenAndServe(":9090", nil) // 设置监听的端口，第二个参数为 nil，则默认使用 DefaultServeMux
	if err != nil {
		log.Fatal("ListenAndServe:", err)
	}
}
```

(看到没，好多大写字母开头的方法和变量)

Go 和 node.js 类似，可以不需要 nginx/apache 这些 web 服务器，因为它们自带 http web 服务，直接监听 socket。

### 3.3 - Go web 工作方式

这小节解释得还蛮详细的。详略，贴两张文章中的图吧。

![](../../art/go_web_3.3.http.png)

![](../../art/go_web_3.3.illustrator.png)

4 个概念：

- Request: 用户请求的信息，用来解析用户的请求信息，包括 post、get、cookie、url 等信息
- Response: 服务器需要反馈给客户端的信息
- Conn: 用户的每次请求链接
- Handler: 处理请求和生成返回信息的处理逻辑

http 包为每一个连接创建一个协程进行处理。

### 3.4 - Go http 包详解

http 包两个核心功能：conn 和 ServeMux。

我们主要来理解 ServeMux，它的作用是路由。它的结构如下：

```go
type ServeMux struct {
  mu sync.RWMutex        // 锁
  m map[string]muxEntry  // 路由规则
}

type muxEntry struct {
  explicit bool    // 是否精确匹配
  h        Handler // 路由表达式对应的 handler
}

type Handler interface {
  ServeHTTP(ResponseWriter, *Request) // 路由的实现器
}
```

Handler 是一个接口，但前一小节的 sayHello 并没有实现 ServerHTTP() 方法，为什么能添加到路由器里呢，原来 http 包里还定义了一个 HandlerFunc 类型，这个类型默认实现了 ServeHTTP() 方法，因此 HandlerFunc 就是 Hanlder 接口的实现。看代码：

```go
type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
  f(w, r)
}
```

有点精妙，原来 type 还可以这么用，需要好好理解一下。实际就是说因为 HandlerFunc 实现了 ServeHTTP() 方法，所以它就可以当 Handler 接口用，而 HandlerFunc 就是任意参数为 ResponseWriter 和 \*Request 的函数，而 sayHello 符合这个特征，所以它就可以当 Handler 接口用。这里的精妙在于 type 可以基于一个函数签名生成一个类型，而这个函数类型居然还可以实现接口方法，奇特，这和其它语言相当地不同。

路由器由到请求后调用 `mux.handler(r).ServeHTTP(w, r)`，也就是调用相应 handler 的 ServeHTTP() 接口方法。`mux.handler(r)` 的实现：

```go
func (mux *ServeMux) handler(r *Request) Handler {
  mux.mu.RLock()
  defer mux.mu.RUnlock()

  h := mux.match(r.Host + r.URL.Path)
  if h == nil {
    h = mux.match(r.URL.Path)
  }
  if h == nil {
    h = NotFoundHanlder()
  }
  return h
}
```

主要就是根据用户的请求去匹配相应的路由，返回相应的 handler。

Go 支持外部自定义的路由器，通过 ListenAndServe() 的第二个参数使用，它是一个 Handler 接口，因此自定义的路由器只要实现 ServeHTTP() 方法即可。可以在自定义的路由器里加一些中间件之类的。

```go
type MyMux struct {}

func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  if r.URL.Path  == "/" {
    sayHello(w, r)
    return
  }
  http.NotFound(w, r)
  return
}
// ...
```

## 4 - 表单

- 4.1 - 处理表单的输入
- 4.2 - 验证表单的输入
- 4.3 - 预防跨站脚本
- 4.4 - 防止多次提交表单
- 4.5 - 处理文件上传

### 4.1 - 处理表单的输入

`request.ParseForm()` 得到 `request.Form` 对象。

其余略。

### 4.2 - 验证表单的输入

> 开发 Web 的一个原则就是，不能信任用户输入的任何信息，所以验证和过滤用户的输入信息就变得非常重要。

通过长度，正则进行验证，详略。

### 4.3 - 预防跨站脚本

> 对 XSS 最佳的防护应该结合以下两种方法: 一是验证所有输入数据，有效检测攻击 (这个我们前面小节已经有过介绍); 另一个是对所有输出数据进行适当的处理，以防止任何已成功注入的脚本在浏览器端运行。

> 那么 Go 里面是怎么做这个有效防护的呢? Go 的 html/template 里面带有下面几个函数可以帮你转义：

> - `func HTMLEscape(w io.Writer, b []byte)` - 把 b 进行转义之后写到 w
> - `func HTMLEscapeString(s string) string` - 转义 s 之后返回结果字符串
> - `func HTMLEscaper(args ...interface{}) string` - 支持多个参数一起转义，返回结果字符串

除了 html/template 外还是 text/template，详略，需要时再了解。

### 4.4 - 防止多次提交表单

客户端可以使用 debounce 和 throttle 来限制。

服务端可以在表单中添加一个带有唯一值的隐藏字段。

```html
<input type="hidden" name="token" value="{{.}}" />
```

> 我们在模版里面增加了一个隐藏字段 token，这个值我们通过 MD5 (时间戳) 来获取惟一值，然后我们把这个值存储到服务器端 (session 来控制，我们将在第六章讲解如何保存)，以方便表单提交时比对判定。

### 4.5 - 处理文件上传

处理文件上传，第一步就是要给 form 添加 enctype 属性， enctype 属性有如下三种情况：

- `application/x-www-form-urlencoded` - 表示在发送前编码所有字符 (默认)。
- `multipart/form-data` - 不对字符编码。在使用包含文件上传控件的表单时，必须使用该值。
- `text/plain` - 空格转换为 "+" 加号，但不对特殊字符编码。

(只有这三种??)

html:

```html
<html>
  <head>
    <title>上传文件</title>
  </head>
  <body>
    <form enctype="multipart/form-data" action="/upload" method="post">
      <input type="file" name="uploadfile" />
      <input type="hidden" name="token" value="{{.}}" />
      <input type="submit" value="upload" />
    </form>
  </body>
</html>
```

go:

```go
http.HandleFunc("/upload", upload)

// 处理 "/upload" 逻辑
func upload(w http.ResponseWriter, r *http.Request) {
	fmt.Println("method:", r.Method) //获取请求的方法
	if r.Method == "GET" {
		crutime := time.Now().Unix()
		h := md5.New()
		io.WriteString(h, strconv.FormatInt(crutime, 10))
		token := fmt.Sprintf("%x", h.Sum(nil))

		t, _ := template.ParseFiles("upload.gtpl")
		t.Execute(w, token)
	} else {
		r.ParseMultipartForm(32 << 20)
		file, handler, err := r.FormFile("uploadfile")
		if err != nil {
			fmt.Println(err)
			return
		}
		defer file.Close()
		fmt.Fprintf(w, "%v", handler.Header)
		f, err := os.OpenFile("./test/" + handler.Filename, os.O_WRONLY | os.O_CREATE, 0666)  // 此处假设当前目录下已存在 test 目录
		if err != nil {
			fmt.Println(err)
			return
		}
		defer f.Close()
		io.Copy(f, file)
	}
}
```

通过 `r.ParseMultipartForm(maxMemory int)` 解板 form，通过 `r.FormFile()` 获取上面的文件句柄，然后使用 `io.Copy()` 来存储文件。

上例文件 handler 是 multipart.FileHandler 结构：

```go
type FileHeader struct {
  Filename string
  Header textproto.MIMEHeader
  // contains filtered or unexported fields
}
```

用 Go 实现上传的客户端：略，需要时再看。

## 5 - 访问数据库

> Go 没有内置的驱动支持任何的数据库，但是 Go 定义了 database/sql 接口，用户可以基于驱动接口开发相应数据库的驱动。

- 5.1 - database/sql 接口
- 5.2 - 使用 MySQL 数据库
- 5.3 - 使用 SQLite 数据库
- 5.4 - 使用 PostgreSQL 数据库
- 5.5 - 使用 beedb 库进行 ORM 开发
- 5.6 - NoSQL 数据库操作

### 5.1 - database/sql 接口

Go 定义了一系列用于操作数据的接口，但没有实现，第三方可以通过实现这些接口来定义自己的驱动。

Go 定义了一个全局的驱动 map，任何第三方数据库驱动都需要在其 `init()` 函数中调用 `sql.Register()` 方法注册到全局驱动 map 中。

```go
// 全局的驱动 map
var drivers = make(map[string]driver.Driver)

//https://github.com/mattn/go-sqlite3驱动
func init() {
  sql.Register("sqlite3", &SQLiteDriver{})
}

//https://github.com/mikespook/mymysql驱动
// Driver automatically registered in database/sql
var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
func init() {
  Register("SET NAMES utf8")
  sql.Register("mymysql", &d)
}
```

database/sql 的详细定义，略。

### 5.2 - 使用 MySQL 数据库

因为没有官方自带的驱动，所以第三方实现的驱动比较多，本节案例使用 https://github.com/Go-SQL-Driver/MySQL。

详略。

### 5.3 - 使用 SQLite 数据库

略。

### 5.4 - 使用 PostgreSQL 数据库

使用 https://github.com/bmizerany/pq 库，其余略。

### 5.5 - 使用 beedb 库进行 ORM 开发

略。

### 5.6 - NoSQL 数据库操作

使用 mgo 库，跟 orm 的使用方式类似，详略。

## 6 - session 和数据存储

> Web 里面经典的解决方案是 cookie 和 session，cookie 机制是一种客户端机制，把用户数据保存在客户端，而 session 机制是一种服务器端的机制，服务器使用一种类似于散列表的结构来保存信息，每一个网站访客都会被分配给一个唯一的标志符, 即 sessionID, 它的存放形式无非两种: 要么经过 url 传递, 要么保存在客户端的 cookies 里. 当然, 你也可以将 session 保存到数据库里, 这样会更安全, 但效率方面会有所下降。

- 6.1 - session 和 cookie
- 6.2 - Go 如何使用 session
- 6.3 - session 存储
- 6.4 - 预防 session 存储

### 6.1 - session 和 cookie

> cookie 是有时间限制的，根据生命期不同分成两种: 会话 cookie 和持久 cookie。

> 如果不设置过期时间，则表示这个 cookie 生命周期为从创建到浏览器关闭止，只要关闭浏览器窗口，cookie 就消失了。这种生命期为浏览会话期的 cookie 被称为会话 cookie。会话 cookie 一般不保存在硬盘上而是保存在内存里。

> 如果设置了过期时间 (setMaxAge(606024))，浏览器就会把 cookie 保存到硬盘上，关闭后再次打开浏览器，这些 cookie 依然有效直到超过设定的过期时间。存储在硬盘上的 cookie 可以在不同的浏览器进程间共享，比如两个 IE 窗口。而对于保存在内存的 cookie，不同的浏览器有不同的处理方式。

可以使用 net/http 包的 `SetCookie(w ResponseWriter, c *Cookie)` 方法来设置 cookie：

```go
expiration := *time.LocalTime()
expiration.Year += 1
cookie := http.Cookie{Name: "username", Value: "astaxie", Expires: expiration}
http.SetCookie(w, &cookie)
```

读取 cookie：

```go
cookie, _ := r.Cookie("username")
fmt.Fprint(w, cookie)

// or
for _, cookie := range r.Cookies() {
  fmt.Fprint(w, cookie.Name)
}
```

session：

> 程序需要为某个客户端的请求创建一个 session 的时候，服务器首先检查这个客户端的请求里是否包含了一个 session 标识 - 称为 session id，如果已经包含一个 session id 则说明以前已经为此客户创建过 session，服务器就按照 session id 把这个 session 检索出来使用 (如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的 session 对象，但用户人为地在请求的 URL 后面附加上一个 JSESSION 的参数)。如果客户请求不包含 session id，则为此客户创建一个 session 并且同时生成一个与此 session 相关联的 session id，这个 session id 将在本次响应中返回给客户端保存。

### 6.2 - Go 如何使用 session

> 目前 Go 标准包没有为 session 提供任何支持...

session 创建过程：

1. 生成全局唯一标识符作为 session id
1. 将 session id 存储在服务器内存或数据库中
1. 将 session id 发送给客户端

如何将 session id 发送给客户端，有两种方式，一种是使用 cookie 作为载体，一种方法是 url 重写。

1. Cookie 服务端通过设置 Set-cookie 头就可以将 session 的标识符传送到客户端，而客户端此后的每一次请求都会带上这个标识符，另外一般包含 session 信息的 cookie 会将失效时间设置为 0 (会话 cookie)，即浏览器进程有效时间。至于浏览器怎么处理这个 0，每个浏览器都有自己的方案，但差别都不会太大 (一般体现在新建浏览器窗口的时候)。
1. URL 重写，所谓 URL 重写，就是在返回给用户的页面里的所有的 URL 后面追加 session 标识符，这样用户在收到响应之后，无论点击响应页面里的哪个链接或提交表单，都会自动带上 session 标识符，从而就实现了会话的保持。虽然这种做法比较麻烦，但是如果客户端禁用了 cookie 的话，此种方案将会是首选。

Go 实现 session 管理：写得很好，思想已理解，需要时再回头看。Session 在服务端也是一个可以存储任意 key-value 值的结构体，但返回给客户端的只包括 session id。服务端从请求的 cookie 中得到 session id，然后通过 session id 从服务端的内存或数据库中取回完整的 session 整个结构体的内容，对这个内容进行读取或更新。详略。

### 6.3 - session 存储

实现了一个内存的 session store。

### 6.4 - 预防 session 劫持

设置 cookieonly，增加隐藏的 token，设置 session 过期时间... (其实并没有完全理解这些策略可以防止 session 被劫持)

## 7 - 文本处理

- 7.1 - XML 处理
- 7.2 - JONS 处理
- 7.3 - 正则处理
- 7.4 - 模板处理
- 7.5 - 文件操作
- 7.6 - 字符串处理

### 7.1 - XML 处理

解析 xml，使用 xml 包的 Unmarshal() 方法：

```go
func Unmarshal(data []byte, v interface{}) error
```

第一个参数为 xml 流，第二个参数是转换成的类型，支持 strcut，slice 和 string。

```go
type Recurlyservers struct {
	XMLName     xml.Name `xml:"servers"`
	Version     string   `xml:"version,attr"`
	Svs         []server `xml:"server"`
	Description string   `xml:",innerxml"`
}

type server struct {
	XMLName    xml.Name `xml:"server"`
	ServerName string   `xml:"serverName"`
	ServerIP   string   `xml:"serverIP"`
}
```

注意 struct 中每个属性中跟在类型后的内容，如 `xml:"serverName"`，这个部分称为之 struct tag，这是 struct 的特性，用来辅助反射的。

XML 包内部采用了反射来进行数据的映射，所以 v 里面的字段必须是导出的 (即字段是大写字母开头)。解析时优先读取 struct tag，如果不存在，则读取字段名。

一些解析规则：

1. 如果 struct 的一个字段是 string 或者 []byte 类型且它的 tag 含有 ",innerxml"，Unmarshal 将会将此字段所对应的元素内所有内嵌的原始 xml 累加到此字段上
1. 如果 struct 中有一个叫做 XMLName，且类型为 xml.Name 字段，那么在解析的时候就会保存这个 element 的名字到该字段
1. 如果某个 struct 字段的 tag 定义了中含有 ",attr"，那么解析的时候就会将该结构所对应的 element 的与字段同名的属性的值赋值给该字段
1. 如果某个 struct 字段的 tag 定义型如 "a>b>c", 则解析的时候，会将 xml 结构 a 下面的 b 下面的 c 元素的值赋值给该字段
1. ... 更多略，需要时再看

输出 xml，使用以下两个函数：

```go
func Marshal(v interface{}) ([]byte, error)
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
```

一些输出规则，详略。

### 7.2 - JSON 处理

解析 json，使用 json 包中的 Unmarshal() 方法：

```go
func Unmarshal(data []byte, v interface{}) error
```

例子：

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Server struct {
	ServerName string
	ServerIP   string
}

type Serverslice struct {
	Servers []Server
}

func main() {
	var s Serverslice
	str := `{"servers":[{"serverName":"Shanghai_VPN","serverIP":"127.0.0.1"},{"serverName":"Beijing_VPN","serverIP":"127.0.0.2"}]}`
	json.Unmarshal([]byte(str), &s)
	fmt.Println(s)
}
```

注意，此例中 struct 并没有 struct tag，而且字段名和 json 中的字段名并不对应，那是怎么匹配上的呢？

1. 匹配时优先使用 struct tag
1. 没有 struct tag 时使用 struct 的字段名
1. 当字段名跟 json 字段名不完全匹配时 (因为前者必须大写开头，后者可以小写开头)，而匹配 json 中其它非大写字母开头的字段

如果上面的例子使用 struct tag，则是这样：

```go
type Server struct {
	ServerName string `json:"serverName"`
	ServerIP   string `json:"serverIP"`
}

type Serverslice struct {
	Servers []Server `json:"servers"`
}
```

解析到 interface：如果不知晓被解析的数据的格式，可以解析到 interface{} 类型 (即其它语言的 any 类型)。详略，需要时再看。

生成 JSON，使用 marshal() 方法，详略。

### 7.3 - 正则处理

使用 regexp 包，详略。

### 7.4 - 模板处理

和其它语言一样的处理方法，先加载模板文件，获取对象，然后将对象渲染到模板中的占位符上。

```go
func handler(w http.ResponseWriter, r *http.Request) {
	t := template.New("some template") // 创建一个模板
	t, _ = t.ParseFiles("tmpl/welcome.html")  // 解析模板文件
	user := GetUser() // 获取当前用户信息
	t.Execute(w, user)  // 执行模板的 merger 操作
}
```

模板中的占位符：`{{.}}` 表示当前对象，`{{.FieldName}}` 表示当前对象的 FieldName 字段。

输出嵌套字段：`{{with ...}} ... {end}`，`{{range ...}} ... {{end}}`

条件处理： if-else

pipelines：`{{. | html}}`

模板变量：`$variable := ...`

```go
{{with $x := "output" | printf "%q"}}{{$x}}{{end}}
{{with $x := "output"}}{{printf "%q" $x}}{{end}}
{{with $x := "output"}}{{$x | printf "%q"}}{{end}}
```

模板函数：略。

Must 操作：检测模板是否正确。

嵌套模板。定义：

```go
{{define "子模板名称"}}内容{{end}}
```

使用：

```go
{{template "子模板名称"}}
```

### 7.5 - 文件操作

os 包，包括文件的读，写，删掉，文件/文件夹的创建... 详略。

### 7.6 - 字符串处理

strings 包。

常用方法：Contains() / Join() / Index() / Repeat() / Replace() / Split() / Trim() / Fields()

字符串转换：AppendXxx() / FormatXxx() / ParseXxx() 系列函数

详略。

## 8 - Web 服务

- 8.1 - Socket 编程
- 8.2 - WebSocket
- 8.3 - REST
- 8.4 - RPC

### 8.1 - Socket 编程

net 包，直接使用 tcp/udp socket，详略。

### 8.2 - WebSocket

详略，需要时再看。

### 8.3 - REST

老生常谈了，略。

一个示例：

```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/julienschmidt/httprouter"
)

func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
	fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func getuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are get user %s", uid)
}

func modifyuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are modify user %s", uid)
}

func deleteuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are delete user %s", uid)
}

func adduser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	// uid := r.FormValue("uid")
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are add user %s", uid)
}

func main() {
	router := httprouter.New()
	router.GET("/", Index)
	router.GET("/hello/:name", Hello)

	router.GET("/user/:uid", getuser)
	router.POST("/adduser/:uid", adduser)
	router.DELETE("/deluser/:uid", deleteuser)
	router.PUT("/moduser/:uid", modifyuser)

	log.Fatal(http.ListenAndServe(":8080", router))
}
```

### 8.4 - RPC

> Go 标准包中已经提供了对 RPC 的支持，而且支持三个级别的 RPC：TCP、HTTP、JSONRPC。但 Go 的 RPC 包是独一无二的 RPC，它和传统的 RPC 系统不同，它只支持 Go 开发的服务器与客户端之间的交互，因为在内部，它们采用了 Gob 来编码。

详略，需要时再看。

## 9 - 安全与加密

- 9.1 - 预防 CSRF 攻击
- 9.2 - 确保输入过滤
- 9.3 - 避免 XSS 攻击
- 9.4 - 避免 SQL 注入
- 9.5 - 存储密码
- 9.6 - 加密和解密数据

详略，很多以前在 rails 开发中都了解的。

## 10 - 国际化和本地化

go-i18n 包。

## 11 - 错误处理，调试和测试

错误处理：error

调试：gdb

测试：单元测试和压力测试 (benchmark)

详略。

## 12 - 部署与维护

- 12.1 - 应用日志：使用 log 包或第三方库记录 log
- 12.2 - 网站错误处理
- 12.3 - 应用部署：使用 Supervisord 将 Go 程序 daemon 化
- 12.4 - 备份和恢复：rsync / MySQL 备份和恢复 / redis 备份和恢复

## 13 - 如何设计一个 Web 框架

路由设计，MVC (model / view / controller)，日志和配置设计... 详略。

## 14 - 扩展 Web 框架

- 14.1 - 静态文件支持：Go 的 net/http 包中提供了静态文件的服务，ServeFile 和 FileServer 等函数
- 14.2 - Session 支持
- 14.3 - 表单支持：表单验证
- 14.4 - 用户认证：auth
- 14.5 - 多语言支持：i18n
- 14.6 - pprof 支持：性能监控

详略。

Done! 这本电子书写得真不错，归纳总结得很完善，可以作为一本工具书时常翻阅参考。
