# Web Misc

- JWT (Json Web Token)

## JWT

在浏览器中，我们为了记住用户登录的情况，使用 session 或 cookie 来保存用户登录情况。

在 API 中，session / cookie 不再方便使用，我们用 access token 来作用户认证。

但是，无论是 session / cookie 还是 access token，你都至少需要在服务端的内存或数据库中保存一些额外的数据，比如 `session_id` 或 token。

但如果使用了 jwt，你不需要在服务端保存额外的数据，只需要额外的一个对称密钥或一对非对称密钥用来加密和解密。

而且相比 access token，jwt 还可以附带额外的数据 (payload)。

总结起来，jwt 的优点：

- 不需要服务端保存额外的数据
- 可以携带额外的数据，比如 `user_id`，但不能在 payload 中保存敏感信息

jwt 的数据分三部分，header，payload，signature。

jwt 的数据在服务端产生，服务端用对称密钥或非对称密钥的公钥，对 header base64 和 payload base64 后的结果用 `.` 连接后，进行加密，生成签名。签名的目的是确保 header 和 payload 的数据没有被人篡改。

服务端将生成的 jwt 返回给客户端，以后客户端在每次请求时在 http 的 header 中带上这个 jwt，服务端收到请求后，对 header, payload 进行 base64 解码，同时用对称密钥或非对称密钥的私钥，对签名进行验证。验证 OK 说明 payload 数据是可信的。

(有一个疑问是，使用非对称密钥的必要性何在? 感觉用对称密钥就行了啊...)

payload 中可以包含任意非敏感数据，比如 `user_id`，以及其它自定义的数据。
