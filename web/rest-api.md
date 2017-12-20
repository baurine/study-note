# REST API

对 REST API 平时使用过程中的一些总结。

设计得比较好的参考示范：

- [GitHub REST API v3](https://developer.github.com/v3/)

一些参考文章：

- [我所认为的 RESTful API 最佳实践](http://www.scienjus.com/my-restful-api-best-practices/)

**冪等**

REST API 中有一个 "幂等" 的概念，表示多次请求的效果相同。在 REST API 中，除了 POST 请求，其它请求都是冪等的。

- 每一次 POST 请求都会在服务端产生新的资源，但其它请求不会。
- 对于相同的 DELETE 请求来说，删一次和删多次的效果是一样的，服务端的目标资源被销毁了。
- 对于相同的 PUT 请求来说，PUT 一次和多次的效果也是一样的。
- 对于相同的 GET 请求来说，虽然每次的结果可能会有略微差别，但它不会修改服务端的资源。

**点赞 / 收藏 / 加星等类似操作的 API 设计**

在 [GitHub REST API](https://developer.github.com/v3/activity/starring/#check-if-you-are-starring-a-repository) 中，对一个 Repo 的点赞行为，它是这样设计的：

- 获取是否 star 状态：`GET /user/starred/:owner/:repo`
- star repo：`PUT /user/starred/:owner/:repo`
- unstart repo： `DELETE /user/starred/:owner/:repo`

成功的响应：`Status: 204 No Content`，获取失败的响应：`Status: 404 Not Found`

在上面的最佳实践文章中举了点赞的例子，是这样设计的：

- `GET /articles/id/like`：查看文章是否被点赞
- `PUT /articles/id/like`：点赞文章
- `DELETE /articles/id/like`：取消点赞

和 GitHub 是一样的理念。为什么点赞是 PUT 操作，而不是 POST 操作，因为点赞这种行为应该是幂等的，多次操作结果应该相同。

但这篇文章认为，可以直接在响应中返回点赞的状态，而不是返回 204。人个觉得效果差不多，在响应中返回点赞状态，可以让客户端的逻辑简单一些，少一些保持状态的逻辑。

**PUT / PATCH 的区别**

> 在 RESTful 的标准中，PUT 和 PATCH 都可以用于修改操作，它们的区别是 PUT 需要提交整个对象，而 PATCH 只需要提交修改的信息。但是在我看来实际应用中不需要这么麻烦，所以我一律使用 PUT，并且只提交修改的信息。

**POST 的对象**

> 另一个问题是在 POST 创建对象时，究竟该用表单提交更好些还是用 JSON 提交更好些。其实两者都可以，在我看来它们唯一的区别是 JSON 可以比较方便的表示更为复杂的结构 (有嵌套对象)。另外无论使用哪种，请保持统一，不要两者混用。
