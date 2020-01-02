# 浏览器与服务器传输方式
网络架构图
# ![网络架构图](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/HTTP_img.jpg)


## 浏览器给服务器的内容
# ![请求头](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/header1.jpg)
首先看看http请求消息（就是浏览器丢给服务器的）：

爬虫根据这个信息可以伪装成浏览器；  
一个http请求代表客户端浏览器向服务器发送的数据。一个完整的http请求消息，包含一个请求行，若干个消息头（请求头），换行，实体内容  
请求行：描述客户端的请求方式、请求资源的名称、http协议的版本号。 例如： GET/BOOK/JAVA.HTML HTTP/1.1  
请求头（消息头）包含（客户机请求的服务器主机名，客户机的环境信息等）：  
```
Accept： 用于告诉服务器，客户机支持的数据类型  （例如：Accept:text/html,image/*）  
Accept-Charset：   用于告诉服务器，客户机采用的编码格式  
Accept-Encoding：  用于告诉服务器，客户机支持的数据压缩格式  
Accept-Language：  客户机语言环境  
Host:     客户机通过这个服务器，想访问的主机名  
If-Modified-Since：客户机通过这个头告诉服务器，资源的缓存时间  
Referer： 客户机通过这个头告诉服务器，它（客户端）是从哪个资源来访问服务器的（防盗链）  
User-Agent：浏览器型号
Cookie：    保存在本地
Connection：告诉服务器，请求完成后，是否保持连接  
Date：      告诉服务器，当前请求的时间  
```

## 再看看HTTP响应消息（服务器返回给浏览器的）：  
# ![body](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/body2.jpg)
一个http响应代表服务器端向客户端回送的数据，它包括：  
一个状态行，若干个消息头，以及实体内容  

响应头(消息头)包含:  
```
Location：这个头配合302状态吗，用于告诉客户端找谁  
Server：  服务器通过这个头，告诉浏览器服务器的类型  
Content-Encoding：告诉浏览器，服务器的数据压缩格式  
Content-Length：  回送数据的长度  
Content-Type：    回送数据的类型  
Last-Modified：   当前资源缓存时间  
Refresh：  隔多长时间刷新  
Content-Disposition：告诉浏览器以下载的方式打开数据。
例如： context.Response.AddHeader("Content-Disposition","attachment:filename=xxx.jpg");                                               
      context.Response.WriteFile("aa.jpg");  
Transfer-Encoding：  告诉浏览器，传送数据的编码格式  
ETag：    缓存相关的头（可以做到实时更新）  
Expries： 告诉浏览器回送的资源缓存多长时间。如果是-1或者0，表示不缓存  
Cache-Control：控制浏览器不要缓存数据   no-cache  
Pragma：       控制浏览器不要缓存数据   no-cache  
Connection：响应完成后，是否断开连接。  close/Keep-Alive  
Date：      告诉浏览器，服务器响应时间  
```

HTTP 响应状态码参考： 
====  
## 1开头
```
1xx:信息
100 Continue: 服务器仅接收到部分请求，但是一旦服务器并没有拒绝该请求，客户端应该继续发送其余的请求。
101 Switching Protocols: 服务器转换协议：服务器将遵从客户的请求转换到另外一种协议。
```
## 2开头   
```
2xx:成功
200 OK 请求成功（其后是对 GET 和 POST 请求的应答文档）
201 Created 请求被创建完成，同时新的资源被创建。
202 Accepted 供处理的请求已被接受，但是处理未完成。
203 Non-authoritative Information
文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝。
204 No Content
没有新文档。浏览器应该继续显示原来的文档。如果用户定期地刷新页面，而 Servlet 可以确定用户文档足够新，
这个状态代码是很有用的。
205 Reset Content 没有新文档。但浏览器应该重置它所显示的内容。用来强制浏览器清除表单输入内容。
206 Partial Content 客户发送了一个带有 Range 头的 GET 请求，服务器完成了它。
```
## 3开头   
```
3xx:重定向
300 Multiple Choices 多重选择。链接列表。用户可以选择某链接到达目的地。最多允许五个地址。
301 Moved Permanently 所请求的页面已经转移至新的 url  
302 Moved Temporarily 所请求的页面已经临时转移至新的 url  
303 See Other 所请求的页面可在别的 url 下被找到  
304 Not Modified
未按预期修改文档。客户端有缓冲的文档并发出了一个条件性的请求（一般是提供 If-Modified-Since 头表示客
户只想比指定日期更新的文档）。服务器告诉客户，原来缓冲的文档还可以继续使用。
305 Use Proxy 客户请求的文档应该通过 Location 头所指明的代理服务器提取。
306 Unused 此代码被用于前一版本。目前已不再使用，但是代码依然被保留。
307 Temporary Redirect 被请求的页面已经临时移至新的 url。
```
## 4开头   
```
4xx:客户端错误
400 Bad Request 服务器未能理解请求。
401 Unauthorized 被请求的页面需要用户名和密码。
401.1 登录失败。
401.2 服务器配置导致登录失败。
401.3 由于 ACL 对资源的限制而未获得授权。
401.4 筛选器授权失败。
401.5 ISAPI/CGI 应用程序授权失败。
401.7 访问被 Web 服务器上的 URL 授权策略拒绝。这个错误代码为 IIS 6.0 所专用。
402 Payment Required 此代码尚无法使用。
403 Forbidden 对被请求页面的访问被禁止。
403.1 执行访问被禁止。
403.2 读访问被禁止。
403.3 写访问被禁止。
403.4 要求 SSL。
403.5 要求 SSL 128。
403.6 IP 地址被拒绝。
403.7 要求客户端证书。
403.8 站点访问被拒绝。
403.9 用户数过多。
403.10 配置无效。
403.11 密码更改。
403.12 拒绝访问映射表。
403.13 客户端证书被吊销。
403.14 拒绝目录列表。
403.15 超出客户端访问许可。
403.16 客户端证书不受信任或无效。
403.17 客户端证书已过期或尚未生效。
403.18 在当前的应用程序池中不能执行所请求的 URL。这个错误代码为 IIS 6.0 所专用。
403.19 不能为这个应用程序池中的客户端执行 CGI。这个错误代码为 IIS 6.0 所专用。
403.20 Passport 登录失败。这个错误代码为 IIS 6.0 所专用。
404 Not Found 服务器无法找到被请求的页面。
404.0 没有找到文件或目录。
404.1 无法在所请求的端口上访问 Web 站点。
404.2 Web 服务扩展锁定策略阻止本请求。
404.3 MIME 映射策略阻止本请求。
405 Method Not Allowed 请求中指定的方法不被允许。
406 Not Acceptable 服务器生成的响应无法被客户端所接受。
407 Proxy Authentication Required 用户必须首先使用代理服务器进行验证，这样请求才会被处理。
408 Request Timeout 请求超出了服务器的等待时间。
409 Conflict 由于冲突，请求无法被完成。
410 Gone 被请求的页面不可用。
411 Length Required "Content-Length" 未被定义。如果无此内容，服务器不会接受请求。
412 Precondition Failed 请求中的前提条件被服务器评估为失败。
413 Request Entity Too Large 由于所请求的实体的太大，服务器不会接受请求。
414 Request-url Too Long 由于 url 太长，服务器不会接受请求。当 post 请求被转换为带有很长的查询信息
的 get 请求时，就会发生这种情况。
415 Unsupported Media Type 由于媒介类型不被支持，服务器不会接受请求。
416 Requested Range Not Satisfiable 服务器不能满足客户在请求中指定的 Range 头。
417 Expectation Failed 执行失败。
423 锁定的错误。
```
## 5开头   
```
5xx:服务器错误
500 Internal Server Error 请求未完成。服务器遇到不可预知的情况。
500.12 应用程序正忙于在 Web 服务器上重新启动。
500.13 Web 服务器太忙。
500.15 不允许直接请求 Global.asa。
500.16 UNC 授权凭据不正确。这个错误代码为 IIS 6.0 所专用。
500.18 URL 授权存储不能打开。这个错误代码为 IIS 6.0 所专用。
500.100 内部 ASP 错误。
501 Not Implemented 请求未完成。服务器不支持所请求的功能。
502 Bad Gateway 请求未完成。服务器从上游服务器收到一个无效的响应。
502.1 CGI 应用程序超时。 ·
502.2 CGI 应用程序出错。
503 Service Unavailable 请求未完成。服务器临时过载或当机。
504 Gateway Timeout 网关超时。
505 HTTP Version Not Supported 服务器不支持请求中指明的 HTTP 协议版本
```




