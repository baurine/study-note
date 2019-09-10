# Go Misc

- Go Modules

## Go Modules

go 官方依赖管理。`go mod init`，生成 go.mod 文件。

## Web 框架

资源：

- [7 天用 Go 从零实现 Web 框架 Gee 教程](https://github.com/geektutu/7days-golang)
- [Go-Mega Tutorial Go web](https://github.com/bonfy/go-mega)
- [gin offical doc](https://github.com/gin-gonic/gin)
- [gin full doc](https://www.jianshu.com/p/98965b3ff638)
- [Go Gin 简明教程](https://geektutu.com/post/quick-go-gin.html)
- [gin 教程](https://youngxhui.top/categories/gin/)
- [go iris](https://wxnacy.com/2019/03/01/go-iris-simple/)

主要是这两个框架：gin / iris

略微看了一下文档，路由基本是这么配置的：

gin 的例子：

```go
func main() {
    router := gin.Default()

    // 此规则能够匹配/user/john这种格式，但不能匹配/user/ 或 /user这种格式
    router.GET("/user/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.String(http.StatusOK, "Hello %s", name)
    })

    // 但是，这个规则既能匹配/user/john/格式也能匹配/user/john/send这种格式
    // 如果没有其他路由器匹配/user/john，它将重定向到/user/john/
    router.GET("/user/:name/*action", func(c *gin.Context) {
        name := c.Param("name")
        action := c.Param("action")
        message := name + " is " + action
        c.String(http.StatusOK, message)
    })

    router.POST("/form_post", func(c *gin.Context) {
        message := c.PostForm("message")
        nick := c.DefaultPostForm("nick", "anonymous") // 此方法可以设置默认值

        c.JSON(200, gin.H{
            "status":  "posted",
            "message": message,
            "nick":    nick,
        })
    })

    router.Run(":8080")
}
```

iris 也差不多。

和 Node.js 的 express，Python 的 Flask 很像，不像 rails 那样是以 controller 为核心的 (Python 的 Djanjo 也是以 controller 为核心的吧?)

web 框架原理都差不多，详略，需要时再看文档。

orm 库可以用 gorm 包。
