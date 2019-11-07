### TCP客户端
TCP客户端很简单，把socket里的套接字type改成SOCK_STREAM就可以了  
![TCP客户端](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/tcp_client3.jpg)
### TCP服务端  
一图抵万语。通过买个手机，类比TCP服务器的创建；  
1、socket创建一个套接字；             买个手机；  
2、bind绑定ip和port；                插上手机卡；  
3、listent使用套接字变为可以被动连接； 设计手机能够响铃;  
4、recv/send收发数据；               等待别人打过来;  
![TCP客户端](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/tcp_server.jpg)  

### 接下来
-[文件下载器](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/Data_down.md)  

