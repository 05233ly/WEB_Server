# HTTP协议总览
网络架构图
# ![网络架构图](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/HTTP_img.jpg)


## 浏览器给服务器的内容
# ![请求头](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/header1.jpg)
首先看看http请求消息（就是浏览器丢给服务器的）：

爬虫根据这个信息可以伪装成浏览器；  
一个http请求代表客户端浏览器向服务器发送的数据。一个完整的http请求消息，包含一个请求行，若干个消息头（请求头），换行，实体内容  
请求行：描述客户端的请求方式、请求资源的名称、http协议的版本号。 例如： GET/BOOK/JAVA.HTML HTTP/1.1  
请求头（消息头）包含（客户机请求的服务器主机名，客户机的环境信息等）：  
Accept：用于告诉服务器，客户机支持的数据类型  （例如：Accept:text/html,image/*）  
Accept-Charset：用于告诉服务器，客户机采用的编码格式  
Accept-Encoding：用于告诉服务器，客户机支持的数据压缩格式  
Accept-Language：客户机语言环境  
Host:客户机通过这个服务器，想访问的主机名  
If-Modified-Since：客户机通过这个头告诉服务器，资源的缓存时间  
Referer：客户机通过这个头告诉服务器，它（客户端）是从哪个资源来访问服务器的（防盗链）  
User-Agent：浏览器型号
Cookie：保存在本地
Connection：告诉服务器，请求完成后，是否保持连接  
Date：告诉服务器，当前请求的时间  

## 再看看HTTP响应消息（服务器返回给浏览器的）：  
# ![body](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/body2.jpg)
一个http响应代表服务器端向客户端回送的数据，它包括：  
一个状态行，若干个消息头，以及实体内容  

响应头(消息头)包含:  
Location：这个头配合302状态吗，用于告诉客户端找谁  
Server：服务器通过这个头，告诉浏览器服务器的类型  
Content-Encoding：告诉浏览器，服务器的数据压缩格式  
Content-Length：回送数据的长度  
Content-Type：回送数据的类型  
Last-Modified：当前资源缓存时间  
Refresh：隔多长时间刷新  
Content-Disposition：告诉浏览器以下载的方式打开数据。例如： context.Response.AddHeader("Content-Disposition","attachment:filename=xxx.jpg");                                          context.Response.WriteFile("aa.jpg");
Transfer-Encoding：告诉浏览器，传送数据的编码格式  
ETag：缓存相关的头（可以做到实时更新）  
Expries：告诉浏览器回送的资源缓存多长时间。如果是-1或者0，表示不缓存  
Cache-Control：控制浏览器不要缓存数据   no-cache  
Pragma：控制浏览器不要缓存数据          no-cache  
Connection：响应完成后，是否断开连接。  close/Keep-Alive  
Date：告诉浏览器，服务器响应时间  

  





