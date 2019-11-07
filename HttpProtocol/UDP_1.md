## 一个简单客户端  
![UDP通信](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/simple1.jpg)



### 解释  
1、socket创建一个套接字；  
2、然后使用send发送数据，在后面绑定ip与端口，同时指定编码格式；

### 效果
可以在虚拟机看到主机发出的数据；
![send](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/send1.jpg)

### 说明  
UDP基本不用，互联网开始起步，设备差，UDP用的多。  

### 知识点
windows里默认编码是GBK；  
Linux里默认编码是utf-8;  

私有ip：  
10.0.0.0  ~  10.255.255.255  
172.16.0.0  ~  172.31.255.255  
192.168.0.0  ~ 192.168.255.255


### 接下来
-[TCP客户端与服务端](https://github.com/KissMyLady/Web-of-Python/blob/master/HttpProtocol/TCP_1.md)
