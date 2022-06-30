## HTTP常用状态码
+ 200 OK,请求成功
+ 204 No Content，没有资源返回
+ 206 Partial Content，当从客户端发送Range范围标头以只请求资源的一部分时，将使用此响应代码
+ 301 Moved Permanently，请求资源的 URL 已永久更改。在响应中给出了新的 URL。
+ 302 Found，请求资源的 URI 已 暂时 更改
+ 303 See Other，指示客户端通过一个 GET 请求在另一个 URI 中获取所请求的资源
+ 304 Not Modified，它告诉客户端响应还没有被修改，因此客户端可以继续使用相同的缓存版本的响应
+ 307 Temporary Redirect，这与 302 Found HTTP 响应代码具有相同的语义，但用户代理 不能 更改所使用的 HTTP 方法：如果在第一个请求中使用了 POST，则在第二个请求中必须使用 POST
+ 400 Bad Request，由于被认为是客户端错误（例如，错误的请求语法、无效的请求消息帧或欺骗性的请求路由），服务器无法或不会处理请求。
+ 401 Unauthorized，客户端必须对自身进行身份验证才能获得请求的响应
+ 403 Forbidden，客户端没有访问内容的权限；与 401 Unauthorized 不同，服务器知道客户端的身份。
+ 404 Not Found，服务器找不到请求的资源。
+ 405 Method Not Allowed，服务器知道请求方法，但目标资源不支持该方法。
+ 500 Internal Server Error，服务器遇到了不知道如何处理的情况。
+ 502 Bad Gateway，该状态码表明扮演网关或代理角色的服务器，从上游服务器中接收到的响应是无效的
+ 503 Service Unavailable，服务器没有准备好处理请求。
## HTTP/0.9
仅支持 Get仅能访问 HTML 格式资源  
## HTTP/1.0
+ 默认短连接（一次请求建议一次TCP连接，请求完就断开）,新增POST，DELETE，PUT，HEADER等方式
+ 增加请求头和响应头概念，指定协议版本号，携带其他元信息（状态码、权限、缓存、内容编码）
+ 扩展传输内容格式（图片、音视频、二进制等都可以传输）
## HTTP/1.1
### 长连接
HTTP/1.1 支持长连接和管道化连接，在一个 TCP 连接上可以传送多个 HTTP 请求，避免了因为多次建立 TCP 连接的时间消耗和延时(Connection:keep-alive)
### 缓存处理
HTTP/1.1 新增了 ETag、If-Modified-Since、If-Match 、If-None-Match 等新的请求头来控制缓存
### 带宽优化以及网络连接的使用
HTTP/1.1 在请求头中引入了 range，支持断点续传的功能
### Host 头处理
在 HTTP/1.0 中认为每台服务器都有唯一的 IP 地址，但随着虚拟主机技术的发展，多个主机共享一个 IP 地址越发普遍，HTTP/1.1 的请求消息和响应消息都应该支持 Host 头域，且请求消息中如果没有 Host 头域会报 400 错误
